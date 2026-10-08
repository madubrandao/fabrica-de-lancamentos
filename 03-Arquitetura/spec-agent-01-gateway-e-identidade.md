---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0020; DEC-0029; DEC-0030; M02-plano-de-execucao (F1); spike A5 (2026-10-08-spike-a5-n8n); spec-core-01; spec-core-04 §5; spec-core-05; spec-agent-07 (revisão); blueprint v1 §22, §36
status: proposta do ENG-00, revisada após a revisão independente (spec-agent-07) — decisões D1.1 a D1.7 aguardam a Madu (A5 resolvido pela Madu em 2026-10-08: token curto por execução)
atualizado: 2026-10-08
---
# Bloco 1 do Agent Engine: gateway, identidade e limites

Continua o [[spec-core-01-servicos-e-limites]]. Resolve o PENDENTE do [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]: **o que roda na API e o que roda no n8n**, e o papel do *agent runtime gateway*.

## 1. O que já está fixado
- O agente é uma **entidade do registry**; a **execução** acontece em workflows do n8n (DEC-0020 §4). Regras R1–R8 do M01 valem.
- **Spike A5 (negativo):** o n8n 2.41.6 Community **não isola credenciais por workflow**; o n8n é um **único domínio de confiança**; a fronteira de segurança fica **na API** ([[2026-10-08-spike-a5-n8n]]).
- A Madu escolheu em 2026-10-08: **token curto por execução**.

## 2. Divisão de responsabilidades (PROP)
| Peça | Onde | Por quê |
|---|---|---|
| Registry, permissões, definições, prompts, ferramentas, escopos | **API** (gateway) | fonte da verdade; o n8n nunca vê o prompt |
| Contexto, política, memória, estado da execução, audit | **API** | R1–R8; testável com banco real |
| Chamada ao **LLM** | **API** (adaptador) | chave fora do credential store do n8n; custo duro; audit central |
| Laço de passos, ferramentas **externas**, integrações, notificações | **n8n** | é o papel do n8n (DEC-0020) |
| **Iniciar** uma execução | **só a API** | um workflow não se passa por outro agente |
O **gateway** é uma fachada da própria API (D1.5 do M01): rotas `/agent-gateway/…` no mesmo processo, com `mutate`, audit e idempotência. Não é serviço novo.

## 3. Identidade: token curto por execução (decisão da Madu)
### 3.1 O que acontece
A API **cria a execução** (`execution_id`), emite um **token assinado, curto, preso ao agente e à execução**, e **chama o webhook do agente no n8n** passando o token. O workflow só fala com o gateway **com esse token**. **Nenhuma credencial de longa duração de agente** vive no n8n: só o segredo do webhook API→n8n e a credencial de **serviço** (claim/ack, já existente).

### 3.2 O token (PROP)
JWT HS256 com **segredo próprio** (`EXECUTION_TOKEN_SECRET`), claims `typ=execution`, `sub=<código do agente>`, `exe=<execution_id>`, `org`, **`ep=<época>`**, `exp`, `jti`. **Algoritmo fixo**; `typ` obrigatório. O `typ` existe em **dois lugares com papéis diferentes**: no **cabeçalho do JWT** (não confiável: só escolhe qual segredo tentar) e no **claim** (verificado depois da assinatura). O caminho de sessão da Madu **rejeita** token com claim `typ=execution`, e o gateway rejeita token sem ele. O **primeiro** token de uma execução já nasce com `exp = min(agora + duração, deadline_at)`, e o despachante não envia nada se o `deadline_at` já passou. **Duração: PENDENTE (valor da Madu).** Piso de segurança do segredo (mínimo de 32 caracteres e recusa de `CHANGE_ME`): **PROP aguardando a Madu**, aplicado também ao `JWT_SECRET` (pendência herdada do M01). A carga verifica que `EXECUTION_TOKEN_SECRET ≠ JWT_SECRET`.
**Como o `authenticate()` escolhe a chave:** lê o `typ` do cabeçalho **sem confiar nele**, só para escolher o segredo; depois verifica com algoritmo fixo e **exige** o claim `typ=execution`. Um token de execução apresentado a uma rota do M01 é recusado (403); um token de sessão apresentado ao gateway é recusado (401).

### 3.3 Época: um token por vez (corrige o risco de tokens antigos)
`agent_execution.token_epoch` (inteiro) **sobe a cada despacho** (`start` ou `resume`) **e a cada `renew`**. A revalidação exige `ep == token_epoch`: **o token anterior morre no ato**, mesmo antes do `exp`. Teste obrigatório: T1 usado depois do despacho que gerou T2 → 401.

### 3.4 Revalidação a cada chamada
Além da época: a execução existe, é do agente do token, está em estado que aceita **aquela rota** (matriz estado × rota no bloco 4), não passou do `deadline_at`, e o agente está ativo. **Cancelar a execução ou desativar o agente invalida o token na hora.**

### 3.5 Renovação (D1.6)
Execução mais longa que o `exp` precisa renovar: `POST renew`, autenticada pelo token **ainda válido**, emite novo token com `ep+1` e `exp = min(agora + duração, deadline_at)`; o antigo morre pela época. **O `exp` nunca passa do `deadline_at`.**
**Resposta perdida do `renew`:** a identificação do `renew` vem do **estado da linha**, derivada de `(execution_id, ep_atual)` (não da Idempotency-Key). Repetido com o token da época **anterior**, depois de a época já ter subido por esse mesmo `renew` e **antes de qualquer chamada com o token novo**, ele **reemite o token da época atual** (jti determinístico) **sem subir a época de novo**. O token novo **nunca** é guardado em `idempotency_key` nem em tabela. **Estado que torna isto implementável:** `agent_execution.renew_from_ep`, `renew_exp` (o `exp` da reemissão, **congelado no primeiro valor**) e `renew_used` (marcado pela **primeira chamada com o token novo**). O retry com o token da época anterior só é aceito se `renew_used = false`, nenhum despacho subiu a época desde o `renew` e ainda dentro do `deadline_at`, com **tolerância de `exp` apenas nessa rota**; um despacho (a época sobe sem `renew`) **fecha a janela**. (Alternativa B: duração do token = `deadline`, mais simples, mas a janela de um token comprometido é a execução inteira.)

### 3.6 O novo ator `execution` no código do M01 (corrige o furo crítico)
- `Actor` ganha `{ type: 'execution', code, executionId, organizationId }`.
- **`requirePermission` passa a tratar o ator de forma exaustiva** (`switch` sem caso permissivo por omissão): `human` é isento (como hoje); `execution` é avaliado por `decideForAgent(code)`; `agent` idem; `service` não executa ações de agente; **qualquer tipo novo nega**.
- `actorRef(execution)` devolve `{ type: 'agent', id: <código> }` (o banco só aceita `agent`/`human`/`system` em audit e eventos) e o `execution_id` viaja no **contexto**, vindo **só do claim `exe`**.
- `actorKey(execution)` = `execution:<exe>` (idempotência por execução; a chave do `step` é derivada de `execution_id` + `seq`, estável entre retries do nó HTTP).
- **As rotas do M01 passam a usar lista de permitidos (`human`, `agent` ou `service`, conforme a rota), nunca lista de negados.** Hoje duas rotas funcionam por exclusão e aceitariam o ator novo: `GET /launches/:id` (só bloqueia `service`) e `POST /permissions/check` (cai no ramo "não-agente"); **as duas mudam** (ver o bloco 6, seção 5-ter). O ator `execution` só alcança as rotas do gateway. Teste de inventário: **cada rota do M01 chamada com token de execução devolve 403.**
- **`execution_id` e `correlation_id` do audit e do log não vêm de cabeçalho do chamador** para o ator `execution`: o `execution_id` vem do token e o `correlation_id` de `agent_execution.correlation_id`. Se o cabeçalho `x-execution-id` vier e divergir → **403 e audit** (o `mutate` audita 403). Para os demais atores o cabeçalho passa a ser aceito só com formato UUID e marcado como **não verificado**. **Log:** o logger filho criado no hook `onRequest` (antes de autenticar) é **recriado depois do `authenticate()`** com o `execution_id` do token, e o cabeçalho `x-correlation-id` do chamador é **ignorado** para o ator `execution` (o `reqId` passa a ser o `correlation_id` da linha).

## 4. Como uma execução começa e retoma (PROP)
1. **Gatilhos**, todos criados pela API: (a) **evento** (assinante interno, ver 4.1); (b) **operadora** (`executions start`); (c) **herança** (a ferramenta `handoff.start` de outra execução, bloco 4). Agendamento é do M03.
2. A API grava a execução (`created`) e uma linha em **`agent_dispatch`** (fila no padrão do outbox, mesma transação).
3. O **despachante** (laço próprio, com limite de concorrência e `FOR UPDATE SKIP LOCKED`, com disparo imediato depois do commit) **só incrementa a época com `UPDATE … WHERE status IN (…)` sob `FOR UPDATE` da execução** (`created`/`dispatched` para `start`; `awaiting_approval`/`awaiting_child` para `resume`; **`running` para `redispatch`**, que o varredor (f) do bloco 4 cria como `agent_dispatch(kind=start)` quando um workflow cai depois do `start`); se o estado já saiu dessa lista, marca o despacho como `done` **sem reenviar**. No `start` de época maior sobre execução `running`, a época já foi subida pelo despachante sob `FOR UPDATE`; um passo `open` com `llm_call requested` segue `step_in_progress`/`step_interrupted` (bloco 4 §3). Teste: W1 morre depois do `start`, o (f) redespacha, W2 assume o lease, W1 com o token velho recebe 401. Monta o token **na hora do envio** e faz `POST` no webhook do agente com `x-webhook-secret`. **2xx = recebido** (S2). Entrega **pelo menos uma vez**; o `start` do gateway trata a repetição (bloco 4).
4. **Retomada** (aprovação **concedida**, filha terminada): a API insere `agent_dispatch(kind=resume)` **na mesma transação** que a causa (seção 4.2); o despachante manda **token novo**. Rejeição e vencimento **não** geram `resume` (seção 4.2).
5. Falha de despacho além do limite (**valor da Madu**): execução `failed` (final) + `error.detected` (origem `api`); **reiniciar = criar outra execução** (`executions retry`), nunca ressuscitar a mesma (D4.5).

### 4.1 Gatilho por evento (D1.7)
- O mapa **evento → agente** vem da definição do agente (`triggers[]`, bloco 2). Os eventos usam o **mesmo outbox do M01** com um assinante **interno** (`agent-trigger`) cujo "transporte" é uma função em processo (o `Transport` já é injetável), em vez do webhook HTTP: herda ordem, retry e `dead`.
- `UNIQUE (organization_id, agent_id, trigger_type, trigger_ref)` com `trigger_ref = event_id`: entrega duplicada **não** cria segunda execução.
- `agent-trigger` é **não ordenado** (`ordered=false`): um erro numa execução não trava os eventos seguintes da mesma entidade. **Recusa legítima** (agente inativo, `definition_exceeds_permissions`) **não vira retry até `dead`**: o assinante registra `skipped` com audit e confirma (ack). Assinantes **internos são isentos** da exigência de `N8N_WEBHOOK_URL_<NOME>` (muda o `loadWebhookTargets`).
- `trigger_ref` por tipo: `event` = `event_id`; `operator` = a Idempotency-Key; `retry` = `execution_id` da falhada; `handoff` = o `call_id` do `handoff.start` na mãe. Execução gerada por **evento emitido por outra execução** herda `root_execution_id` e `depth` e conta em `max_executions_per_root` (corta o ciclo "A emite E que dispara A").
- Recomendado sobre a alternativa (um workflow no n8n que chama a API para iniciar), porque mantém a regra "só a API inicia".

### 4.2 Retomada dentro da transação (muda o `approvals.ts` do M01)
Só quando a aprovação é de uma **ferramenta de uma execução** (`action_name` começa com `tool.` e há `tool_call` ligado; gates e continuidade **não mudam**): `grantApproval` **insere o `agent_dispatch(resume)` na própria transação** (não depende de um consumidor de `approval.granted`); `rejectApproval` e `expireDueApprovals` marcam o `tool_call` como `denied(approval_rejected|approval_expired)` e levam a execução a **`failed` sem `resume`**. **Ordem canônica de travas: execução antes de aprovação** em `grant`, `reject`, `expire` e no `step` de retomada (evita impasse). Se a execução já é final, a concessão tardia é aceita **sem `resume`** e auditada com `execution_final`; a CLI marca essas aprovações como **órfãs**. Exige índice em `tool_call(approval_id)`. **O assinante (b) do M01** ("workflow que retoma o processo que esperava a aprovação") **deixa de ser usado para execuções de agente**; o assinante `notify` continua.

## 5. Superfície do gateway (enxuta; corrige o excesso)
**Para o workflow (token de execução):** `start`, `step`, `tool-calls/:id/result` (resultado de ferramenta externa; exige `tool_call.execution_id` igual ao `exe` do token), `renew`, `complete`, `fail`. **Só isso.** `complete` e `fail` **não levam corpo livre**: referenciam o `seq` cuja diretiva gravada é `completed`/`failed`, e o resultado vem de `agent_step.directive`. **Toda rota do gateway revalida dentro da transação, com `FOR UPDATE` da execução, e converte violação do gatilho de estado final em 409.** O `step` usa **tratador próprio** (duas fases, bloco 4) e **não** o `mutate` de transação única. **`start`, `renew`, `complete` e `fail` também não podem usar o `Idempotency-Key` do `mutate`** (uma repetição devolveria só `{replayed, id}` e o workflow ficaria sem lease nem token): usam `mutate` com `idempotent:false` ou tratador próprio, e a idempotência vem do **estado da linha** (`run_id`/`lease_epoch` no `start`; `(execution_id, ep)` no `renew`; `seq` no `complete`/`fail`), com o corpo **sempre recomputado a partir da linha**. `complete`/`fail` repetidos com o mesmo `seq` sobre execução já final devolvem **200 com o resultado gravado**, não 409. As rotas de leitura e da operadora passam por `mutate`. **Exceção declarada à exigência de `Idempotency-Key` do M01 (spec-core-04 e spec-core-05):** `start`, `renew`, `complete` e `fail` ficam **fora do replay por chave** e são testados por estado da linha (o `step` idem, por `seq`). Memória, handoff e emissão de evento **não são rotas**: são **ferramentas internas** que o LLM pede e o `step` executa pelo pipeline (bloco 3). Assim o workflow não consegue chamar uma ferramenta interna com entrada arbitrária.
**Para a operadora (token humano):** `executions` (listar, ver, iniciar, cancelar, `retry`), `dispatches` (listar).
**Para o serviço (credencial do n8n):** claim/ack (M01) e **`fail`** restrito (seção 6).
Todas entram no **inventário de rotas** e levam `execution_id` (do token) no audit; a negação **da rota** (época, estado, agente) vira 401/403 **sem escrita de estado**; a negação 403/409 é **auditada fora da transação**, como no `mutate`, **inclusive no tratador próprio do `step`**; 401 não audita. Teste de inventário: o tratador próprio também audita.

## 6. O que a credencial de serviço alcança (risco aceito, explicitado)
Claim/ack de qualquer evento, `POST /correlations`, `permissions/check` de qualquer agente (M01) e, agora, `fail` de execução. Para o `fail` do serviço: só em `dispatched` e `running`, com **código de erro de uma lista fechada**, mensagem truncada e sanitizada, ator `system:n8n` no audit. **Mesmo assim, um workflow comprometido pode derrubar execuções de outros agentes** (negação de serviço) e injetar texto limitado em `error.detected`: registrado como risco aceito no M02.

## 7. Papel do Redis
Sem papel no M02 (a fila de despacho usa o PostgreSQL). Fica para o modo fila do n8n na hospedagem (antes do M03). Encerra o M5 do DEC-0029.

## 8. Decisões (recomendação do ENG-00)
| # | Pergunta | Opções | Recomendação |
|---|---|---|---|
| D1.1 | Onde ficam registry, contexto, ferramentas, memória, estado e audit | **A)** gateway na API. B) parte no n8n | **A** |
| D1.2 | Identidade dos agentes no n8n | **Token curto por execução, só a API inicia — decidida pela Madu em 2026-10-08** | — |
| D1.3 | Quem chama o LLM | **A)** adaptador na API. B) nós de IA do n8n (aceitam URL base própria, S6) | **A:** a chave não entra no credential store (S7), custo duro, prompt fora do histórico do n8n |
| D1.6 | Renovação do token | **A)** `renew` com época, `exp ≤ deadline_at`. B) duração do token = `deadline` | **A:** janela curta mesmo em execução longa |
| D1.7 | Como a API vira o gatilho por evento | **A)** assinante interno no outbox + `UNIQUE` por `trigger_ref`. B) workflow no n8n chama a API | **A:** só a API inicia execuções |

## 9. PENDENTE
Duração do token; limites e esperas do despacho; `max_concurrent_executions` por agente e global — **configuração obrigatória: sem esses valores a API não sobe (ou o `start` de execução é negado)**, nunca "só o custo trava" (fail-closed); `EXECUTION_TOKEN_SECRET` real; URL-base dos webhooks (a base e o caminho `agent/<código>` existem no M02 para o agente de referência; as URLs de produção, no M03).

## 10. Riscos
- A API vira o disparador dos agentes: mitigado por reaproveitar `mutate`, audit, o inventário e pelo núcleo puro.
- O workflow ainda vê entradas e o que passa por ele: mitigado por histórico `none` (bloco 4) e por não devolver prompt nem contexto.
- Token vazado: limitado ao agente e à execução, morre por época, cancelamento ou `exp`.
- Credencial de serviço ampla: seção 6.

## 11. Fora deste bloco
[[spec-agent-02-contrato-e-definicao-do-agente]] · [[spec-agent-03-ferramentas-politica-e-aprovacao]] · [[spec-agent-04-execucao-estado-e-telemetria]] · [[spec-agent-05-contexto-e-memoria]] · [[spec-agent-06-ferramentas-de-codigo-e-testes]] · [[spec-agent-07-revisao-independente]].
