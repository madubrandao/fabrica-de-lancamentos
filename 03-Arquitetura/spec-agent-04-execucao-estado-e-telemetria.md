---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0010 §8; DEC-0020; DEC-0029; spec-core-04 §3, §7, §9; spec-agent-01 a 03; spec-agent-07 (revisão); blueprint v1 §30–§31, §36; 2026-10-08-spike-a5-n8n
status: proposta do ENG-00, revisada após a revisão independente — decisões D4.1 a D4.8 aguardam a Madu
atualizado: 2026-10-08
---
# Bloco 4 do Agent Engine: ciclo de vida da execução, estado, custo do LLM e telemetria

## 1. O que já está fixado
*Toda execução tem `execution_id` e audit trail* (ENG-04). O `audit_log` já tem a coluna `execution_id` (**texto**; o join com `agent_execution.id` uuid exige cast). A telemetria dos agentes ficou **fora do event store** (D4.6 do Core) para o M02 decidir. O M02 herda M1 (n8n emitir evento pela API) e M2 (`error.detected` de origem `api` e `workflow`).

## 2. A execução (PROP)
Tabela `agent_execution` (`id` = `execution_id`): `organization_id`, `agent_id`, `definition_hash`, `definition_version`, `trigger_type` (`event` | `operator` | `handoff` | `retry`), `trigger_ref`, `parent_execution_id`, `root_execution_id`, `depth`, `correlation_id` (herdado do gatilho), `launch_id` (opcional), `input` (jsonb limitado), `status`, **`token_epoch`**, `lease_epoch`, `lease_run_id`, `deadline_at` (**calculado na criação**: `created_at + deadline_seconds` da definição), `started_at`, `finished_at`, `result` (jsonb limitado) ou `error_code` + `error_message` (**sanitizados**), `cancel_reason`, `cancelled_by`. **Sem contador de uso na linha**: o uso agregado sai de `llm_call` (evita contenção na linha travada). Restrições: `UNIQUE (organization_id, agent_id, trigger_type, trigger_ref)`; `CHECK` de consistência `root/parent/depth`; **gatilho de estado final imutável** (depois de `succeeded`/`failed`/`timed_out`/`cancelled` nada muda).

### 2.1 Estados e transições (tabela completa)
| Origem | Destino | Quem faz | Por quê |
|---|---|---|---|
| `created` | `dispatched` | despachante (após o 2xx, `UPDATE … WHERE status='created'`) | webhook recebeu |
| `created` ou `dispatched` | `running` | `start` do workflow | primeiro `start` da época vence; o 2xx pode chegar depois do `start` |
| `running` | `running` (assume o lease) | `start` com **época maior** que `lease_epoch` | retomada após queda do workflow: continua do último `seq` fechado |
| `awaiting_approval` ou `awaiting_child` | `failed` | despachante (esgotou o despacho `resume`) | `error.detected` de origem `api` |
| `created` ou `dispatched` | `failed` | despachante (esgotou tentativas) ou serviço (`fail` restrito) | despacho morto |
| `running` | `awaiting_approval` | `step` (ferramenta com aprovação) | pipeline passo 7 |
| `running` | `awaiting_child` | `step` (`handoff.start`) | espera a filha |
| `awaiting_approval` ou `awaiting_child` | `running` | `start` de um despacho `resume` | aprovação decidida ou filha terminou |
| `awaiting_approval` | `failed` | decisão (rejeitada/vencida) ou política | sem aprovação |
| `running` | `succeeded` | `complete` | fim |
| `running` | `failed` | `step`/`fail`/despacho/política | erro, limite |
| qualquer não final | `timed_out` | varredor de `deadline_at` | prazo |
| qualquer não final | `cancelled` | operadora | cancelamento |
Transição fora desta tabela = 409 e audit. `deadline_at` conta o tempo corrido **inclusive** o de espera por aprovação ou filha (prazo único; se a Madu aprovar depois do prazo, a retomada é recusada porque a execução já é final). **Matriz estado × rota** (qual rota aceita qual estado) é gerada desta tabela e testada.

### 2.2 `start` com lease e despacho duplicado
Cada despacho sobe a **época**. O `start` recebe um **`run_id`** (o identificador da execução do n8n) e grava `lease_epoch` e `lease_run_id` **na linha** (o lease nunca se reconstrói a partir do corpo guardado da idempotência, que o M01 não guarda): (1) **mesmo `run_id` na mesma época** → devolve o **mesmo lease** (cobre a resposta perdida e o retry do nó HTTP); (2) **`run_id` diferente na mesma época** → **409 `already_started`** e aquele workflow termina; (3) **época maior** que `lease_epoch` com a execução já `running` → **assume o lease** e continua do último `seq` fechado (cobre o despacho repetido depois de queda). Passos concorrentes são **serializados por `FOR UPDATE`**. Testes: dois `start`, dois `step`, `start` repetido com resposta perdida, despacho duplicado depois de `start`.

## 3. O passo (`step`) em duas fases (corrige idempotência e transação longa)
Tabelas: **`agent_step`** (`execution_id`, `seq`, `status` = `open` | `done` | `failed`, `directive` limitada **sem prompt**, tempos), `tool_call` (`execution_id`, `seq`, **`call_id` único por execução**, `tool_name`, `input`/`input_hash`, `status` = `requested` | `denied` | `awaiting_approval` | `running` | `succeeded` | `failed` | `outcome_unknown` | `cancelled` (o `handoff.start` fica `running` enquanto a filha trabalha e vira `succeeded`/`failed` com o resultado da filha na retomada), `approval_id`, `output` limitada, motivo sanitizado, tempos) e `llm_call` (`execution_id`, `step_seq`, `model`, `input_tokens`, `output_tokens`, `cost`, `latency_ms`, `status` = `requested` | `done` | `failed`).
0. **Três tipos de passo**, cada um com `seq` próprio: `llm` (chama o modelo), `resume_tool` e `resume_child` (retomadas: **não** chamam o modelo; reapresentam o `tool_call` pendente / entregam o resultado da filha). `agent_step.status` = `open` | `done` | `failed` | `abandoned`.
1. O workflow envia o **`seq` esperado**. **Fase 1** (transação curta): valida (pipeline passos 0–2), trava a execução, grava `agent_step(seq, open)`, **reserva** `llm_call(requested)` — **o custo reservado já conta no limite** — e confere o limite de custo (seção 5).
2. **Chamada ao LLM fora de transação** (segundos), com `max_output_tokens` da definição.
3. **Fase 2** (transação): registra o uso (`llm_call done`), roda o pipeline contra o que o modelo pediu e persiste a **diretiva** em `agent_step.directive`: `run_external_tool {call_id,tool,input}` | `awaiting_approval {approval_id}` | `awaiting_child {execution_id}` | `completed {result}` | `failed {code}` | `continue`.
4. **`step` repetido** (retry do nó HTTP, mesma chave derivada de `execution_id`+`seq`) devolve a **diretiva gravada**; `seq` fora de ordem → 409. **Passo `open` com `llm_call requested` e outra requisição ainda viva → 409 `step_in_progress`.** Passo `open` cujo `llm_call` o varredor marcou `failed` → a execução falha com `step_interrupted` e o passo vira `abandoned` (o custo reservado continua contando). `UNIQUE (execution_id, seq)` garante a unicidade; o `step` usa **tratador próprio**, não o `mutate`. Se a API cair entre as fases, o `llm_call` fica `requested` e um varredor o marca `failed` (o custo reservado continua contando).
5. O **LLM executa uma ferramenta por passo**: o adaptador pede uso de ferramenta **sem paralelismo**; se vier mais de uma chamada, o `step` falha com `multiple_tool_calls` (contrato do adaptador no bloco 6).

## 4. Varredores do agendador (declarados)
(a) `deadline_at` vencido → `timed_out` (token morre, ferramentas em curso `failed`); (b) **despachada e não iniciada** além do prazo (**valor da Madu**) → redespacha e conta tentativa; esgotado → `failed` + `error.detected` (origem `api`); (c) `llm_call` `requested` velho → `failed`; (d) aprovação vencida de execução → `failed` (via a mesma transação de `expireDueApprovals`); (e) despacho `dead` (vale também para `resume`, medido desde a inserção do despacho); (f) **`running` sem `step` novo além do prazo** (valor da Madu) → redespachado com época nova. **Finalizar uma execução** (cancelar, expirar, falhar) cancela as **filhas ativas**, marca o `tool_call` pendente como `failed(execution_final)` e **omite** o `resume` de uma mãe já final. O despachante é **um laço próprio** (limite de concorrência, `FOR UPDATE SKIP LOCKED`, disparo imediato após o commit), não o tick sequencial atual, para não atrasar a entrega de eventos e o vencimento de aprovações. **Limites de concorrência** (`max_concurrent_executions` por agente e global): valores da Madu; sem eles, só o custo trava uma rajada.

## 5. Custo do LLM (PROP; valores da Madu)
- **Pipeline do LLM** (antes de chamar o provedor): execução válida → agente ativo → **limite de custo** → chamada. Sem `llm_cost_limit` na política, **nega** (fail-closed, como o orçamento).
- **Reserva de custo (regra, sem constante inventada):** `reserva = max_output_tokens × preço de saída + (limite superior de tokens de entrada) × preço de entrada`, com o limite superior de entrada = `max_context_bytes` (cada token cobre no mínimo 1 byte: cota rigorosa e conservadora). No `done`, o custo real substitui a reserva; num `failed` por varredura, **a reserva fica contando** (a Madu confirma).
- **Política nova** `llm_cost_limit` em `policy_setting` (mudança no `policy.ts`: o `z.enum` de chaves ganha uma; **listada nas mudanças do M01**). **Valores PENDENTES (Madu):** moeda e unidade; **a quem o limite se aplica** (por chamada, por execução, por dia ou total); e **o preço por token de cada modelo** (outra política `llm_pricing`, valor dela; sem preço, o custo não se calcula e a chamada é negada).
- **Limite acumulado sob concorrência:** a verificação e a **reserva** acontecem sob trava (`pg_advisory_xact_lock` por organização), para dois passos simultâneos não estourarem juntos.
- `limits` da definição ganha `max_output_tokens` (a API da Anthropic exige `max_tokens`; nenhum valor é inventado).
- Gasto de **negócio** (ferramenta `spend`) continua no pipeline de ferramentas (bloco 3, passo 6).

## 6. Handoff (PROP)
A ferramenta interna `handoff.start` cria uma **execução filha** (`parent_execution_id`, `root_execution_id`, `depth+1`) para um agente em `handoff_targets` da **definição da mãe** e permitido no registry; a mãe passa a **`awaiting_child`** e **não consome passos** enquanto espera. Quando a filha chega a estado final, a **mesma transação** do `complete`/`fail`/`timed_out`/`cancel` da filha insere `agent_dispatch(resume)` para a mãe; o resultado volta pela ferramenta interna `handoff.result`. Regras: **profundidade** (`max_handoff_depth` da definição da mãe), **ciclo** (agente já presente nos ancestrais é negado), **`max_children` por mãe** e **`max_executions_per_root`** (valores da Madu), a filha herda `correlation_id`, `launch_id` e o **menor** `deadline_at`; o limite de **custo é da organização** no M02 (não há por execução). A entrada que a mãe passa à filha é **dado não confiável** e a filha exige a **própria** aprovação (delegação confusa aceita e registrada).

## 7. Telemetria, eventos e logs
- **Estado e uso nas tabelas próprias**, não no event store. Entram no event store só os eventos de aprovação existentes, `error.detected` e os tipos de `AGENT_EMITTABLE_EVENTS` (vazia em produção).
- **`error.detected`** ganha produtores de origem **`api`** (despacho morto, limite, `timed_out`/`failed` por erro do sistema) e **`workflow`** (`fail` reportado pelo n8n). Sem laço (regra do M01). O payload de `error.detected` do M01 já tem `entity_type` (texto) e `entity_id` (uuid, opcional): usa-se **`entity_type = agent_execution` e `entity_id = <execution_id>`**, e o envelope vai com `entity = system` e `entity_id` = a organização. **O contrato atual comporta; não há versão nova.**
- **Eventos `agent.execution.*` de negócio**: adiados para o M03 (sem consumidor).
- Logs com `correlation_id` e `execution_id` (já feitos); audit de toda mutação do gateway com `execution_id` do **token**.
- **Prompt e resposta do LLM não são gravados** (só hashes do contexto e do resultado); gravar é opção explícita por ambiente, com retenção da Madu.

## 8. Histórico do n8n (S7/S8) (PROP)
`saveDataSuccessExecution`, `saveDataErrorExecution` e `saveExecutionProgress` em `none`/`false` **nos workflows de agente e nos subworkflows** (`n8n/workflows/` **e** `n8n/subworkflows/`), conferido por teste. **Reforço no ambiente** (`EXECUTIONS_DATA_SAVE_ON_SUCCESS=none`, `…_ERROR=none`, `…_PROGRESS=false`): custo = nenhum histórico no n8n para workflows que não sejam de agente (no M02 só existem os de agente). O workflow de erro recebe **só `execution_id` e um código**, sanitizados. A proibição de `export:credentials --decrypted` é **controle processual** (runbook), não técnico.

## 9. Decisões
| # | Pergunta | Opções | Recomendação |
|---|---|---|---|
| D4.1 | Estado da execução | **A)** tabelas próprias, máquina da seção 2. B) event store | **A** |
| D4.2 | Laço LLM→ferramenta | **A)** passo a passo em duas fases (API: LLM + internas; n8n: externas). B) tudo no n8n. C) tudo na API | **A** |
| D4.3 | Handoff | **A)** execução filha com estado `awaiting_child`. B) entidade "tarefa" | **A** |
| D4.5 | Despacho morto | **A)** `failed` final; refazer = nova execução (`executions retry`); sem `dispatches reprocess`. B) deixar a execução reprocessável | **A:** estado final nunca muda |
| D4.6 | Histórico do n8n | **A)** `none` nos workflows (e no ambiente). B) só por workflow | **A:** no M02 só há workflows de agente |
| D4.7 | Prazo único (`deadline_at`) inclui a espera por aprovação | **A)** sim. B) pausa durante a espera | **A:** simples; a Madu escolhe os valores |
| D4.8 | Critério "o n8n emitindo evento pela API" | **A)** satisfeito por mecanismo (`event.emit`) + teste, sem tipo real em produção. B) exigir um tipo de negócio real agora | **A:** os payloads de domínio são do M03 |

## 10. PENDENTE
Todos os valores de limites, prazos, concorrência, custo e preço; retenção de `agent_execution`/`tool_call`/`llm_call`/`agent_step`/texto do LLM; tamanho máximo de `input`/`result`; eventos `agent.execution.*` (M03); versão do contrato de `error.detected` se for preciso (F3).

## 11. Riscos
Tabelas crescem sem retenção (diferente do `audit_log`, podem ser limpas por prazo) · token que expira no meio (resolvido por `renew`) · despacho duplicado (lease) · efeito externo após cancelamento (bloco 3).

## 12. Fora deste bloco
[[spec-agent-05-contexto-e-memoria]] · [[spec-agent-06-ferramentas-de-codigo-e-testes]].
