---
tipo: arquitetura
area: 03-Arquitetura
fonte: revisão independente de 2026-10-08 (agente revisor, somente leitura) sobre os blocos 1 a 6 do Agent Engine; spike A5; M02-plano-de-execucao (F1); código do M01 em C:\dev\launch-os
status: duas revisões independentes tratadas; terceira leitura curta pendente; pacote de decisões para o portão da F1 abaixo (aguarda a Madu)
atualizado: 2026-10-08
---
# Bloco 7 do Agent Engine: revisão independente da especificação e pacote de decisões

## 1. Resultado da primeira revisão
Revisor **diferente do autor**, somente leitura, que também conferiu as afirmações da especificação contra o código do M01 (audit com `execution_id`, `AGENT_ACTIONS`, `mutate`, `requirePermission`, `policy.ts`, privilégios padrão). Veredicto: **a especificação não estava pronta para a Madu aprovar**. O desenho central (token por execução, fronteira na API, pipeline em ordem fixa, aprovação com hash de uso único, telemetria em tabelas próprias) foi considerado coerente; os furos eram de detalhe, mas alguns de segurança. **3 críticos, 13 altos, 12 médios e vários baixos.**

## 2. O que foi achado e onde foi corrigido
| Achado | Resumo | Corrigido em |
|---|---|---|
| **C1** | `requirePermission` do M01 ignora quem não é `agent` (`return`): o ator `execution` passaria sem avaliação | Bloco 1 §3.6 (exaustivo; tipo novo nega; `actorRef`/`actorKey`; rotas do M01 recusam o token) |
| **C2** | Tokens antigos voltavam a valer depois de um novo despacho | Bloco 1 §3.3 (`token_epoch`, claim `ep`) |
| **C3** | O token curto expirava no meio de qualquer execução longa | Bloco 1 §3.5 e D1.6 (`renew`, `exp ≤ deadline_at`) |
| A1 | `execution_id` do audit vinha de cabeçalho do chamador | Bloco 1 §3.6 (vem do token; `correlation_id` da linha) |
| A2 | Idempotência do M01 não devolve corpo; `step` sem tabela, LLM dentro de transação | Bloco 4 §3 (`agent_step`, duas fases, `seq`, diretiva gravada) |
| A3 | Dois workflows podiam rodar a mesma execução | Bloco 4 §2.2 (lease por época, `409 already_started`) |
| A4 | Corrida `created→dispatched` e execução que nunca começa; varredores não declarados | Bloco 4 §2.1 e §4 |
| A5 | Contradição entre "reprocesso humano" e "estado final não muda" | Bloco 1 §4.5 e D4.5 (`failed` final; `executions retry` cria outra) |
| A6 | Gatilho por evento sem mecanismo; duplicidade; retomada dependente de evento | Bloco 1 §4.1 e §4.2, Bloco 2 `triggers[]`, `UNIQUE` por `trigger_ref` |
| A7 | Aprovação de ferramenta externa e retomada: reemissão, TOCTOU, mudança de definição | Bloco 3 §4 (pipeline reexecutado na retomada, `definition_changed`, diretiva no máximo uma vez) |
| A8 | Escopo de entidade estático; memória sobre `contact`/`customer` | Bloco 3 §2 (`scope(input)`), Bloco 5 D5.6/D5.7 |
| A9 | Cadeia `supersedes_id` com fork; `UPDATE` concedido por padrão; apagamento (LGPD) | Bloco 5 §5 e D5.7 |
| A10 | Custo do LLM sem desenho; `policy.ts` sem a chave; `max_output_tokens` ausente; duas aprovações de gasto | Bloco 4 §5, Bloco 3 passo 6, Bloco 2 `limits` |
| A11 | Handoff sem estado de espera, `handoff.result` fora do registry, sem limites de filhas | Bloco 4 §6 (`awaiting_child`, `max_children`, ciclo) |
| A12 | Superfície do gateway maior que o necessário; rota `events` vazia | Bloco 1 §5 (só `start`/`step`/`result`/`renew`/`complete`/`fail`), Bloco 3 (`event.emit`), D4.8 |
| A13 | Hash dependente de fim de linha; teste do histórico do n8n só em `workflows/` | Bloco 2 §2.2, Bloco 4 §8 |
| M1–M12 | passo 0 do pipeline, transação do TOCTOU, `fail` do serviço, `authenticate()` por `typ`, chave de idempotência, estados faltantes, `usage`, corte de contexto, exibição das aprovações, despachante, dependência circular `schemas↔agents`, guarda do prompt, matriz de testes | Blocos 1, 3, 4, 5, 6 (seções citadas em cada um) |
| Baixos | decisões sem alternativa real reduzidas; base de URL de webhook no M02; `test.*` só com `NODE_ENV=test`; contrato do `LlmAdapter`; `error.detected` com `entity=system`; `packages/memory` reservado; controle de `--decrypted` é processual | Bloco 1 §8–9, Bloco 3 §2, Bloco 4 §7–8, Bloco 6 §1 e §5-bis |

## 3. Segunda leitura
Uma **segunda revisão independente** conferiu se os críticos e altos foram tratados e se as correções criaram contradição nova. Resultado na seção 6.

## 4. Pacote de decisões para o portão (o que a Madu realmente escolhe)
Respondível em bloco com **"siga com as recomendadas"**. Itens já decididos por ela estão marcados.
| # | Decisão | Recomendação | Onde |
|---|---|---|---|
| D1.1 | Registry, contexto, ferramentas, memória, estado e audit **na API**; n8n não toca o banco | Sim | Bloco 1 |
| D1.2 | Identidade por **token curto por execução**, só a API inicia | **Já decidida pela Madu em 2026-10-08** | Bloco 1 |
| D1.3 | **LLM chamado pela API** (adaptador), não por nós de IA do n8n | Sim | Bloco 1 |
| D1.6 | Renovação do token: `renew` com época e `exp ≤ deadline_at` | Sim | Bloco 1 |
| D1.7 | Gatilho por evento: assinante **interno** no outbox, `UNIQUE` por `trigger_ref` | Sim | Bloco 1 |
| D2.2–D2.5 | Manifesto JSON + `prompt.md`; o registry manda se divergir; `model` e `limits` sem valor impedem a definição de carregar; gatilho por evento no campo `triggers[]` | Sim (decididas por consequência de D2.1; listadas aqui para o "siga com as recomendadas" valer) | Bloco 2 |
| D2.1 | Definição do agente em **arquivo no repositório** (`agent.json` + `prompt.md`), hash por execução, **o banco (registry) manda** se divergir | Sim | Bloco 2 |
| D3.5/D3.6 | `test.*` e `event.emit` de teste só com `NODE_ENV=test` (boot recusa o contrário); retomada com definição alterada **falha** com `definition_changed` (custo operacional: um deploy que mude a definição falha as execuções que esperam aprovação) | Sim | Bloco 3 |
| D3.1/D3.2 | Tool registry no código; **escopos `read`/`write` passam a ser aplicados** (inclusive dinâmicos) | Sim | Bloco 3 |
| D3.3/D3.4 | Approval hook com retomada **na mesma transação da decisão**, pipeline reexecutado; efeito externo não idempotente emitido **no máximo uma vez** | Sim | Bloco 3 |
| D4.1 | Estado da execução em **tabelas próprias** (não no event store) | Sim | Bloco 4 |
| D4.2 | Laço **passo a passo em duas fases** (API: LLM e ferramentas internas; n8n: externas) | Sim | Bloco 4 |
| D4.3 | Handoff por **execução filha** com `awaiting_child` | Sim | Bloco 4 |
| D4.5 | Despacho morto = `failed` final; **refazer = nova execução** | Sim | Bloco 4 |
| D4.6/D4.7 | Histórico do n8n **`none`** (workflow e ambiente); `deadline_at` único inclui a espera | Sim | Bloco 4 |
| D4.8 | Critério "n8n emitindo evento pela API" satisfeito por **mecanismo + teste** (sem tipo de negócio real) | Sim | Bloco 4 |
| D5.1/D5.2/D5.5 | Contexto montado na API, determinístico, filtrado por escopo; entrada externa sempre como **dado marcado**; escopos aplicados (inclusive dinâmicos) | Sim | Bloco 5 |
| D5.3/D5.4 | Memória mínima **só de inserção**, com origem; semântica só interface | Sim | Bloco 5 |
| D5.8 | **Sem texto livre** na memória no M02 (`key` de lista fechada configurada, `kind` fixo) | Sim | Bloco 5 |
| D5.6 | Memória visível à organização, **filtrada pelo escopo da entidade-assunto** | Sim | Bloco 5 |
| D5.7 | **Dado pessoal proibido na memória no M02** (apagamento/LGPD decidido no M03) | Sim | Bloco 5 |
| D6.1/D6.2/D6.4 | `packages/agents` com núcleo puro + pg; **LLM por `fetch`**; **nenhuma dependência nova** | Sim | Bloco 6 |
| D6.3 | Migrations 0009 a 0013 como no bloco 6 (inclui `agent_step`, `token_epoch`/`lease_*`, `memory_item` com `version`) | Sim | Bloco 6 |
| D6.5 | Teste do n8n **local** no M02; CI do n8n depois da hospedagem | Sim | Bloco 6 |

## 4-bis. Riscos aceitos e mudanças no M01 (a Madu confirma junto com o pacote)
**Riscos aceitos, explícitos:** (1) a credencial de **serviço** do n8n pode derrubar execuções de outros agentes e injetar texto limitado em `error.detected` (bloco 1 §6); (2) ferramenta **externa** não é recolhida por um cancelamento posterior (bloco 3 §3); (3) delegação confusa entre mãe e filha (bloco 4 §6); (4) leitura de memória gravada por terceiros volta como dado marcado, com risco residual (bloco 5 §10); (5) exigir `write` da entidade do assunto sobrecarrega o significado de `write` (bloco 5).
**Confirmações pedidas à Madu:** (a) num `llm_call` que o varredor marca `failed`, a reserva de custo **continua contando**; (b) `objection` e `partner` carregam dado pessoal? (se sim, saem da lista de assuntos da memória).
**Mudanças no código do M01 que a implementação exige:** lista completa no bloco 6, seção 5-ter (`context.ts`, `agents.ts`, `app.ts` — inclusive `GET /launches/:id` e `POST /permissions/check` —, `approvals.ts`, `policy.ts`, `config.ts`, `packages/schemas`, `packages/events`, CLI), cada uma com teste.

## 5. PENDENTES que são da Madu (nenhum ganha padrão no código)
- **Valores (acréscimos da segunda leitura):** prazo do `running` sem `step`; prazo do `llm_call`/`agent_step` aberto; truncamento das mensagens de erro; tamanho máximo de `input`, `result`, `value` e saída de ferramenta; itens por leitura de memória; teto de tamanho do `prompt.md`; retenção dos novos registros; `max_concurrent_executions` e `AGENT_DISPATCH_*` são **obrigatórios** (sem eles a API não sobe).
- **Valores:** duração do token; `deadline_seconds`, `max_steps`, `max_tool_calls`, `max_output_tokens`, `max_context_bytes`, `max_handoff_depth`, `max_children`, `max_executions_per_root`, `max_concurrent_executions` (por agente e global); tentativas e esperas do despacho; prazo "despachada e não iniciada"; timeout de cada ferramenta; limite de exibição das aprovações; **piso de segurança dos segredos** (proposto: 32 caracteres e sem `CHANGE_ME`).
- **LLM:** a **chave Anthropic**, o **limite de custo** (moeda, unidade, a quem se aplica), o **preço por token de cada modelo** e o **modelo de cada agente**. Sem eles, só o adaptador de teste.
- **Retenção** de execução, passos, ferramentas, chamadas ao LLM e memória; apagamento de dado pessoal.
- **Definições:** escopos reais dos 11 agentes e seus prompts (M03); ferramentas de negócio (M03/M04); `approval_expiry` (sem ele a aprovação nunca vence).
- **Infraestrutura (antes do M03):** hospedagem do n8n e URLs reais dos webhooks, Redis, imagem Docker da API, rate limit, retenção do `audit_log`, troca de senha, backup.

## 6. Segunda leitura (resultado e correções)
Veredicto: os 3 críticos estavam tratados; **sem crítico novo**; **7 altos novos** (4 deles deixavam uma execução legítima encalhada até o prazo), 14 médios e baixos. Tudo tratado nos blocos 1, 3, 4, 5 e 6:
| Achado | Resumo | Corrigido em |
|---|---|---|
| N1 | `start`/redespacho podiam encalhar a execução (resposta perdida; 2xx perdido) | Bloco 4 §2 e §2.2 (`run_id`, `lease_epoch`, assume o lease com época maior), Bloco 1 §4.3 (época só sobe com `UPDATE … WHERE status`), Bloco 4 §4(f) |
| N2 | `renew` com resposta perdida matava a execução | Bloco 1 §3.5 (chave por `(execution_id, ep)`, reemite o token da época atual) |
| N3 | negação dentro da transação apagava o `llm_call` pago; passo aberto; `mutate` não serve ao `step` | Bloco 3 §3 (negação é desfecho, sem rollback), Bloco 4 §3 (3 tipos de passo, `step_in_progress`, `abandoned`, tratador próprio) |
| N4 | `GET /launches/:id` e `POST /permissions/check` aceitariam o ator `execution` | Bloco 1 §3.6 e Bloco 6 §5-ter (lista de permitidos) |
| N5 | rejeição/vencimento geravam `resume`; `resume` sem fim | Bloco 1 §4.2 (só concessão gera `resume`; rejeição/vencimento → `failed`), Bloco 4 §2.1 e §4 |
| N6 | `UNIQUE` da memória não valia com assunto nulo | Bloco 5 §5 (`NULLS NOT DISTINCT`, `CHECK` de versão, mesma cadeia) |
| N7 | limites sem valor falhavam aberto | Bloco 1 §9 (configuração obrigatória; sem ela a API não sobe) |
| M-a…M-n | `complete`/`fail` sem corpo livre; revalidação dentro da transação; aprovação órfã, ordem de travas e cascata de filhas; extração de `evaluateBudget`/`createActionApproval`; `AGENT_ACTIONS` por formato; CLI sem `dispatches reprocess`; fórmula da reserva de custo; ciclos de evento, `ordered=false`, `skipped`, `trigger_ref` por tipo; memória (vigente = maior `version`, assunto nulo, `memory` não é assunto, texto livre); `error.detected` com `entity_type=agent_execution`; log reatrelado após `authenticate()`; PROP do piso dos segredos; `definition_changed` como custo operacional | Blocos 1, 3, 4, 5, 6 |
| Baixos | `typ` cabeçalho × claim; primeiro token com `exp` limitado; `tool-calls/:id/result` só da própria execução; `handoff.result` removido (a retomada do `handoff.start` entrega o resultado); fumaça com perfil de teste; `.gitattributes` também nas fixtures | Blocos 1, 3, 4, 6 |

## 6-ter. Terceira leitura (resultado e correções)
Sem crítico; **2 altos** e 9 médios, todos de redação ou lacuna que reabriam os cenários N1 e N2, tratados: **A1** (o redespacho de uma execução `running` pelo varredor (f) tem mecanismo: despacho `redispatch`, bloco 1 §4.3); **A2** (`start`, `renew`, `complete` e `fail` **não** usam a Idempotency-Key do `mutate`; a idempotência vem do estado da linha, bloco 1 §5); **M1** (estado do `renew`: `renew_from_ep`, `renew_exp`, `renew_used`); **M2–M4** (contradições de redação: rejeição/vencimento sem `resume`, `handoff.result`, "valor vigente"); **M5** (ordem de travas e passos do `grant`/`reject`/`expire`, gates e continuidade intactos); **M6** (a fase 2 sempre grava o `llm_call`); **M7** (schema do `value` por `key`); **M8** (matriz de testes); **M9** (auditoria da negação no tratador próprio).

## 6-quater. Quarta leitura
Veredicto: **pronto**, sem crítico nem alto. 4 médios e 3 baixos, tratados: passo `abandoned` e retomada em `seq+1` (o que o workflow faz com `step_in_progress`); exceção à Idempotency-Key declarada (bloco 1 §5 e bloco 6 §5-ter); replay de `complete`/`fail` na matriz de estados; ordem de travas ancestral → descendente; redação de "concedida"; identificação do `renew` pelo estado da linha; limite de tentativas do varredor (f).

## 6-bis. Mudança no plano do M02
Esta especificação altera decisões do [[M02-plano-de-execucao]]: **D8** (a retomada deixa de ser o assinante `approval-resume` e passa a ser inserida na transação da decisão); **D5** (estados: acrescenta `dispatched`, `awaiting_child`, `timed_out` e a época); **D1** ("credencial do agente" vira token por execução); **F3** (`packages/memory` fica vazio e reservado). O plano será atualizado após o portão.

## 7. O que acontece depois do portão
Com o "aprovado" da Madu: **(1)** PR de **ADRs** no `launch-os` (docs only, ADR-0023 em diante: gateway e identidade, definição em arquivo, tool registry e pipeline, ciclo de vida e custo, memória mínima, adaptador de LLM); **(2)** plano único da **F2 (banco)** e da **F3 (contratos e núcleo)**, que podem andar em paralelo; **(3)** nada de código antes disso.

Ver [[spec-agent-01-gateway-e-identidade]] · [[spec-agent-02-contrato-e-definicao-do-agente]] · [[spec-agent-03-ferramentas-politica-e-aprovacao]] · [[spec-agent-04-execucao-estado-e-telemetria]] · [[spec-agent-05-contexto-e-memoria]] · [[spec-agent-06-ferramentas-de-codigo-e-testes]] · [[M02-plano-de-execucao]].
