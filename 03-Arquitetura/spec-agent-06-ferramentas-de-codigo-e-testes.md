---
tipo: arquitetura
area: 03-Arquitetura
fonte: spec-core-06; DEC-0013; DEC-0016 D2; DEC-0029; spec-agent-01 a 05; M02-plano-de-execucao (F2 a F7); packages/*/CLAUDE.md do launch-os
status: proposta do ENG-00, revisada após a revisão independente (spec-agent-07) — decisões D6.1 a D6.5 aguardam a Madu
atualizado: 2026-10-08
---
# Bloco 6 do Agent Engine: ferramentas de código, migrations, testes e ADRs

Continua o [[spec-core-06-ferramentas-de-codigo]]. **Nada aqui é implementado antes de a Madu aprovar o portão.**

## 1. Pacotes e onde cada coisa mora (PROP)
| Onde | Conteúdo | Dono |
|---|---|---|
| `packages/agents` | schema da definição, carga e hash; tool registry; pipeline de policy (**puro**); montagem de contexto (**pura**); máquina de estados da execução (**pura**); fixtures de injeção | ENG-04 |
| `packages/memory` | **fica reservado e vazio no M02** (D6.1): a interface `Memory`/`SemanticMemory` e a implementação PostgreSQL (SQL explícito, padrão do `packages/events`) moram em `packages/agents` até haver um segundo consumidor | — |
| `packages/integrations` (pasta já existente) | `LlmAdapter` (interface), `AnthropicAdapter` (**`fetch`**, sem SDK novo), `FakeLlmAdapter` (testes) | ENG-04 |
| `apps/api/src/gateway/*` | rotas `/agent-gateway/…`, ator `execution`, token, despachante, passos; reaproveita `mutate`, `audit`, `requirePermission` | ENG-01 |
| `apps/cli` | `executions list|show|start|cancel|retry`, `dispatches list` (**sem** `dispatches reprocess`: refazer é nova execução, D4.5) | ENG-01 |
| `agents/definitions/` | definições de produção (`agent.json` + `prompt.md`). **No M02 fica vazia**: os agentes reais (A0–A10) são do M03. As definições de **referência** do M02 ficam em `tests/fixtures/agents/` e usam um código existente do registry (`A9`) **só no banco de teste** | ENG-04 |
| `n8n/workflows/00_SYSTEM` e `n8n/subworkflows` | `execute-agent` e o agente de referência; `saveData*Execution: none` | ENG-03 |
Regra de dependência: `packages/agents` e `packages/memory` **não importam** a API nem o `pg` da API; a API os usa. O adaptador de LLM nunca é importado por `packages/agents`.

## 2. Dependências novas (PROP: **nenhuma** obrigatória)
`fetch` do Node 24 para o LLM; JSON para o manifesto; `node:crypto` para o token (HS256, já usado) e `@fastify/jwt` já instalado cobre a verificação (um segundo "namespace" de segredo). **Zero dependência nova** é a meta; qualquer exceção volta à Madu antes de instalar (versão exata, como no M01).

## 3. Migrations previstas (F2; todas reversíveis, com round-trip)
| # | Conteúdo |
|---|---|
| 0009 | `approval.action_params` (jsonb, parâmetros legíveis da aprovação de ação) — já decidida (DEC-0029) |
| 0010 | `agent_execution` (+ `token_epoch`, `lease_epoch`, `lease_run_id`, `renew_from_ep`, `renew_exp`, `renew_used`, `deadline_at`, `UNIQUE (organization_id, agent_id, trigger_type, trigger_ref)`, `CHECK` de `root/parent/depth`, gatilho de estado final imutável, FKs `RESTRICT`, índices pelos acessos reais) |
| 0011 | `agent_step`, `tool_call` (`UNIQUE (execution_id, call_id)` e `(execution_id, seq)`) e `llm_call` |
| 0012 | `agent_dispatch` (fila de despacho: execução, tipo `start`/`resume`, status, tentativas, próxima tentativa, erro sanitizado; a época vive em `agent_execution`) |
| 0013 | `memory_item` **append-only** (`version`, `UNIQUE` da cadeia e de `supersedes_id`, `CHECK` de origem, **`REVOKE UPDATE, DELETE` explícito** além dos gatilhos no estilo de `event`/`audit_log`) |
Privilégios: o papel da aplicação ganha `SELECT/INSERT/UPDATE` (padrão) nas tabelas de execução e **só `SELECT/INSERT`** em `memory_item`; **DELETE** só na fila de despacho já tratada, se a Madu definir retenção. Semente: nenhuma linha de execução; `ENTITY_NAMES` ganha `memory`; `AGENT_ACTIONS` **aceita `tool.<nome>` por formato** (a conferência contra o registry é na carga e no `PUT permissions`; sem dependência circular).

## 4. Configuração (sem valor padrão no código)
`EXECUTION_TOKEN_SECRET` (obrigatória; **PROP, aguardando a Madu: mínimo de 32 caracteres e recusa de `CHANGE_ME`**, valendo também para o `JWT_SECRET`; diferente do `JWT_SECRET`) · `EXECUTION_TOKEN_TTL_SECONDS` · `AGENT_DISPATCH_*` (tentativas, esperas, tempo limite, lote) · `N8N_WEBHOOK_URL_AGENT` (base; o caminho é `agent/<código>`) · `LLM_PROVIDER` e `ANTHROPIC_API_KEY` (a chave só da Madu; **sem chave, o adaptador real não liga**) · políticas `llm_cost_limit` e `llm_pricing` em `policy_setting` (valores da Madu; **sem limite ou sem preço, a chamada ao LLM é negada**; mudança no `policy.ts`) · `NODE_ENV` (as ferramentas `test.*` só carregam com `test`; o boot recusa o contrário). Carga que falha listando o que falta, como no M01.

## 5. Testes (Vitest, PostgreSQL real; mesmo padrão do M01)
- **Unitários (F3):** schema da definição (válidos e inválidos), registry × definição, **matriz do pipeline** (cada passo com negativo), máquina de estados (transição legal e ilegal), montagem de contexto (determinismo, corte, escopos, delimitadores), **fixtures de injeção** (escalada por mensagem).
- **Integração (F4):** token (assinatura, `alg`, expiração, agente errado, execução cancelada, agente desativado, repetição), cada rota do gateway (aceita e negada, com audit), despacho (sucesso, falha, retry, `dead`), retomada pós-aprovação (mesmo hash; hash alterado não executa), handoff (profundidade, ciclo), memória (origem obrigatória, escopos, append-only), limite de custo (sem limite nega).
- **Inventário de rotas** estendido (cada rota nova na tabela) e **replay por Idempotency-Key**.
- **n8n (F5):** em pilha isolada, com **LLM de teste**: caminho feliz, retry, timeout, evento duplicado, falha parcial, falha da API, entrada inválida, rollback (Blueprint §31), mais conferência do `saveData*: none`.
- **Qualidade (F6):** failure injection (execução, despacho, ferramenta), varredura de segredos (token, chave do LLM, prompts, contexto), provas negativas por proteção, cobertura.
- **Matriz acrescentada pela revisão:** token T1 depois de novo despacho (401); `start` e `step` concorrentes (lease, 409); cancelamento durante a chamada ao LLM; `exp` no meio da execução e `renew`; `x-execution-id` forjado; `step` repetido devolve a diretiva; diretiva de ferramenta externa não idempotente emitida no máximo uma vez; `definition_changed` na retomada; mesma chave de idempotência em duas execuções do mesmo agente; **cada rota do M01 chamada com token de execução (403)**; fork de `supersedes_id`; evento duplicado criando duas execuções; `dispatched` que nunca inicia; `saveData` em workflow **e** subworkflow; token ausente de log, audit, evento e `idempotency_key` (teste próprio: o guarda de eventos só cobre eventos); `test.*` ligado fora de `NODE_ENV=test` recusa o boot; permissão com `tool.x` de ferramenta removida não derruba o agente.
- **Matriz acrescentada pela segunda leitura:** `renew` com resposta perdida (retry dentro da janela; fora dela); `start` repetido com o mesmo `run_id` e com `run_id` diferente; assumir o lease com época maior (W1 morre, W2 assume, W1 recebe 401); rejeição e vencimento **sem** `resume`; `awaiting_*` → `failed` por esgotamento do `resume`; `step_in_progress`; assunto nulo e `UNIQUE NULLS NOT DISTINCT`; API sem limites obrigatórios **não sobe**; `GET /launches/:id` e `POST /permissions/check` com token de execução (403); aprovação órfã; `complete`/`fail` repetidos com o mesmo `seq`; fase 2 do `step` com a época mudada durante o LLM (grava o `llm_call`, devolve 401); o tratador próprio do `step` audita 403/409.
- **Fumaça do M02:** estende `scripts/smoke` com uma execução ponta a ponta (despacho, passo, ferramenta interna, ferramenta externa, aprovação no meio, retomada, conclusão, audit).

## 5-bis. Contrato do `LlmAdapter` (PROP)
`complete({ model, system, messages, tools, max_output_tokens }) → { calls[], text?, usage, stop_reason }`: mensagens em papéis (`user`/`assistant`/`tool_result`), definição nativa de ferramentas (nome, descrição, schema de entrada), **uso de ferramenta sem paralelismo** (`disable_parallel_tool_use`), `usage` com tokens de entrada e saída, erros tipados (`rate_limited`, `timeout`, `provider_error`, `invalid_request`) **sem texto do provedor que cite chave ou URL**. Se vier mais de uma chamada, o `step` falha com `multiple_tool_calls`. `FakeLlmAdapter` roteirizável nos testes; `AnthropicAdapter` com `fetch`, **só liga com chave e limite de custo da Madu**.

## 5-ter. Mudanças no código do M01 que o M02 exige (lista explícita, para não haver surpresa)
| Onde | Mudança |
|---|---|
| `apps/api/src/app.ts` (rotas do M01) | `GET /launches/:id` e `POST /permissions/check` passam a lista de **permitidos** (hoje por exclusão); tratador próprio do `step`; logger recriado depois do `authenticate()`; `x-correlation-id` ignorado para o ator `execution` |
| `apps/api/src/approvals.ts` (ordem de travas) | ordem global de travas **execução → agente → aprovação → launch**; em `grant`/`reject` lê a aprovação **sem trava** e, só se `action_name` começa com `tool.` e há `tool_call` (por `approval_id`), trava a execução, depois `lockApproval`, e **reconfere `status = 'requested'`**; em `expire` seleciona sem trava (com `JOIN` em `tool_call`) e trava execução e aprovação nessa ordem; **gates e continuidade ficam no caminho antigo, sem mudança (invariante, com teste)**; teste de impasse entre `grant`/`expire` e `step`; só `action_name` com `tool.` e `tool_call` ligado muda de caminho (gate e continuidade intactos, com teste de injeção de falha no `grant`); extração de `evaluateBudget` e `createActionApproval` |
| `packages/events` (`loadWebhookTargets`) | assinantes internos isentos de `N8N_WEBHOOK_URL_<NOME>` |
| `apps/api/src/context.ts` | `Actor` ganha `execution`; `actorRef` (→ `agent` + código), `actorKey` (→ `execution:<exe>`); `execution_id` e `correlation_id` do contexto vêm do token e da linha, não de cabeçalho |
| `apps/api/src/agents.ts` | `requirePermission` exaustivo (nega tipo desconhecido); `PUT permissions` confere `tool.<nome>` contra o registry |
| `apps/api/src/app.ts` | `authenticate()` escolhe a chave pelo `typ` do cabeçalho e verifica `typ`; cabeçalho `x-execution-id` só como UUID não verificado; rotas do gateway |
| `apps/api/src/approvals.ts` | `grantApproval`, `rejectApproval`, `expireDueApprovals` inserem `agent_dispatch(resume)` na mesma transação; `requestActionApproval` grava os parâmetros legíveis (0009) |
| `apps/api/src/policy.ts` | chaves `llm_cost_limit` e `llm_pricing` |
| `apps/api/src/config.ts` | `EXECUTION_TOKEN_SECRET` (≠ `JWT_SECRET`, piso e sem `CHANGE_ME`, **também para o `JWT_SECRET`**) |
| `packages/schemas` | `ENTITY_NAMES` ganha `memory`; `AGENT_ACTIONS` aceita `tool.<nome>` por formato; o teste `schemas-permissions` (22 entidades, 7 ações) é atualizado |
| `packages/events` | assinante interno `agent-trigger` (transporte em processo); o assinante (b) "retoma aprovação" deixa de ser usado por execuções de agente |
| `apps/cli` | `executions list|show|start|cancel|retry`, `dispatches list`; `approvals list` mostra os parâmetros legíveis com escape |

## 6. CI (sem mudança estrutural)
Os mesmos 3 jobs; o job de integração ganha as migrations novas e os testes novos. O teste do n8n (F5) **roda localmente** na pilha isolada e a CI roda só o que não precisa do n8n (decisão de CI para o n8n é da hospedagem, antes do M03).

## 7. ADRs (escritos no repositório **depois do portão**, ADR-0023 em diante)
Gateway e identidade por token de execução · definição do agente em arquivo · tool registry e pipeline de política · ciclo de vida da execução e telemetria · memória estruturada mínima · adaptador de LLM e custo.

## 8. Decisões
| # | Pergunta | Opções | Recomendação |
|---|---|---|---|
| **D6.1** | Pacotes | **A)** núcleo puro e implementação com `pg` em `packages/agents` (memória incluída), adaptador em `packages/integrations`, `packages/memory` reservado. B) tudo em `apps/api` | **A.** Testável sem HTTP; um pacote a menos até haver segundo consumidor |
| **D6.2** | Cliente do LLM | **A)** `fetch` com adaptador próprio. B) SDK oficial da Anthropic | **A.** Zero dependência nova; o contrato é pequeno. B continua possível por ADR |
| **D6.3** | Migrations | **A)** 0009 a 0013 como na seção 3. B) menos tabelas (execução em uma só) | **A.** Cada tabela tem um acesso próprio e um papel de privilégio |
| **D6.4** | Dependências novas | **A)** nenhuma; exceção volta à Madu. B) livre | **A** |
| **D6.5** | Teste do n8n na CI | **A)** local no M02; CI depois da hospedagem. B) subir o n8n na CI agora | **A.** Evita CI pesada e instável antes da decisão de infraestrutura |

## 9. PENDENTE
Valores de configuração (seção 4); chave do LLM e limite de custo; hospedagem do n8n (CI do n8n); versão final do modelo por agente.

## 10. Riscos
- **A API cresce** (gateway, despachante, adaptador): mitigado por módulos separados, núcleo puro nos pacotes e o inventário de rotas.
- **Teste do n8n frágil** (versão, tempo de subida): isolado na F5, não bloqueia a CI.
- **Custo do LLM em testes:** só adaptador de teste; chamada real só com chave e limite da Madu.

## 11. Fora deste bloco
Os agentes de negócio (M03), a hospedagem e o CD, o dashboard (M06).
