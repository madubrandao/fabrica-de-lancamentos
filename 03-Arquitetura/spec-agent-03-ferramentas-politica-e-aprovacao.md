---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0010 §8; DEC-0020; spec-core-05 §4–§7; spec-agent-01; spec-agent-02; spec-agent-07 (revisão); DEC-0029; .claude/agents/ENG-04-agent-engine.md; blueprint v1 §19–§21, §36
status: proposta do ENG-00, revisada após a revisão independente — decisões D3.1 a D3.5 aguardam a Madu
atualizado: 2026-10-08
---
# Bloco 3 do Agent Engine: ferramentas, policy enforcement e approval hooks

## 1. O que já está fixado
*Ferramentas precisam de escopo explícito; agente nunca contorna approval gate; agente nunca altera permissões ou aprovações por mensagem* (ENG-04). O M01 entregou o modelo de permissões (nega por padrão), `requirePermission`, aprovações de **ação** com hash dos parâmetros (D7.2), `BUDGET_AUTHORIZED` e `AGENT_ACTIONS`. **Ações das ferramentas e o tool registry são do M02** ([[spec-core-05-permissoes-e-aprovacoes]], seção 4).

## 2. Tool registry (PROP)
Registro **no código** (`packages/agents`), versionado e revisado por PR. Cada ferramenta declara:
| Campo | Conteúdo |
|---|---|
| `name` | `dominio.verbo` (ex.: `memory.read`); único |
| `kind` | `internal` (a API executa) ou `external` (o n8n executa sob contrato) |
| `input`, `output` | schemas Zod estritos; entrada já **normalizada** pelo schema é a que vai ao hash |
| `scope(input)` | função que devolve `{ reads[], writes[] }` de entidades **para aquela entrada** (escopo **dinâmico**: em `memory.*`, o `subject_type` da entrada exige também `read`/`write` da entidade citada) |
| `effect` | `none`, `internal_write` ou `external_effect` |
| `requires_approval` | `never`, `always` ou regra nomeada (`above_budget`) |
| `cost_class` | `none` ou `spend`; se `spend`, `amount_path` aponta o campo da entrada que traz o valor |
| `idempotent` | repetir a mesma chamada é seguro? |
| `timeout_seconds` | **valor da Madu** (PENDENTE) |
A **ação de permissão** de cada ferramenta é `tool.<name>`. O schema de permissões **aceita `tool.<nome>` por formato** (regex) — evita dependência circular `schemas ↔ agents` —; a **conferência contra o registry** acontece na **carga** da API e no `PUT /agents/:code/permissions`. Uma permissão gravada com `tool.x` de ferramenta que saiu do registry **não invalida o agente inteiro**: a ação é ignorada e a carga registra o aviso. `requires_approval` do modelo de permissões **passa a valer**: ferramenta marcada ali exige aprovação **para aquele agente**, além da regra da ferramenta.

**Ferramentas entregues no M02 (só o mecanismo):** `memory.read`, `memory.write`, `handoff.start`, `event.emit` (internas; o resultado da filha volta pela retomada do próprio `handoff.start`, sem `handoff.result` separado); `test.echo`, `test.external_noop` (externa, para provar o laço), `test.risky_write` (interna, `always`, para provar o hook). As `test.*` e o `event.emit` com tipo de teste **só carregam com `NODE_ENV=test`; o boot recusa subir se estiverem ligadas fora disso**. `AGENT_EMITTABLE_EVENTS` (tipos que `event.emit` pode publicar) é **vazia em produção**; no teste tem um tipo fictício, o que demonstra o critério "o n8n/o agente emitindo evento pela API" (M1 do DEC-0029) por mecanismo + teste. Ferramentas de negócio são do M03/M04.

## 3. Pipeline de policy enforcement (PROP; ordem fixa; cada passo tem teste negativo)
Para cada chamada de ferramenta, no `step`:
0. **Repetição:** se `call_id` já existe na execução, **devolve o resultado gravado antes de contar** contra qualquer limite.
1. **Execução válida** — época, estado, `deadline_at`, `max_steps`, `max_tool_calls`; **a execução (`FOR UPDATE`) e o agente (`FOR SHARE`) são travados — ordem global de travas: **execução, agente, aprovação, launch** — na mesma transação que contém a ação do passo 8**: cancelamento ou desativação não escapam entre a validação e a execução de ferramenta **interna**.
2. **Agente ativo** e ferramenta **listada na definição**.
3. **Permissão de ação** — `decideForAgent(agente, tool.<name>)` no banco.
4. **Entrada válida** — schema Zod estrito (entrada externa é não confiável até validar); **guarda de segredos e de tamanho** sobre a entrada.
5. **Escopos de entidade** — `scope(input)` ⊆ `permissions.read`/`write`. **Passa a ser aplicado** (em M01 eram informativos).
6. **Orçamento** — se `cost_class = spend`: `valor = input[amount_path]`; sem `budget_limit` definido → **nega**; `valor ≤ limite` → segue; `valor > limite` → exige aprovação (passo 7, **a mesma aprovação**, com o valor no hash). (Função `evaluateBudget` reaproveitada do `checkBudget`, **sem** criar uma segunda aprovação.) O custo do **LLM** tem pipeline próprio (bloco 4).
7. **Aprovação** — se `requires_approval`: cria a aprovação de **ação**; a execução vai a `awaiting_approval`. (As funções do M01 `requestActionApproval` e `checkBudget` exigem ator `agent` e as permissões `request_action_approval`/`check_budget`, que a semente não concede: **extrai-se** `evaluateBudget` e `createActionApproval(tx, ctx, …)` **sem checagem de tipo de ator**; quem decide, no pipeline, é a permissão `tool.<nome>`. `request_action_approval` **não** é exigida via execução.)
8. **Execução e registro** — interna: executa e grava; externa: **autoriza** o n8n (diretiva). Grava `tool_call` e audit com o `execution_id`.
Negação em qualquer passo: `tool_call` `denied` com motivo e audit `denied`. **A negação de ferramenta é um desfecho de negócio, não uma exceção**: a fase 2 do `step` **grava** o `tool_call denied` e o `llm_call` (o LLM já foi pago) e persiste a diretiva `failed {code}` ou `continue` (com a negação no histórico); **não há rollback**. Só a negação **da rota** (época, estado, agente) vira 401/403 sem escrita. **Nenhum texto do LLM muda uma decisão** (decide o banco e o registry).
**Risco aceito, explicitado:** numa ferramenta **externa**, o efeito acontece **depois** da autorização e o cancelamento posterior **não o recolhe**; o `tool_call` fica `failed`/`outcome_unknown` e o resultado de execução já finalizada é recusado.

## 4. Approval hook (PROP)
1. Ferramenta que exige aprovação → aprovação de **ação** com `action_name = tool.<name>` e `action_params_hash` sobre `{ agente, tool, entrada canônica **já normalizada**, execution_id, call_id }`. A execução vai a **`awaiting_approval`**; o `step` devolve `awaiting_approval` e o workflow **termina**.
2. A Madu vê **o que está aprovando**: os **parâmetros legíveis** (**migration 0009**, já decidida no DEC-0029). A CLI mostra a **forma canônica completa** (o hash cobre tudo), com **escape de caracteres de controle** e limite de tamanho de exibição (**valor da Madu**); o texto vem do LLM e pode ser longo ou enganoso. Sem `approval_expiry` definido, **a aprovação nunca vence**: é o estado padrão até a Madu definir.
3. **Concedida** → a **mesma transação** insere o `agent_dispatch(resume)` (bloco 1, seção 4.2). **Rejeitada ou vencida** → o `tool_call` vira `denied(approval_rejected|approval_expired)` e a execução `failed`, **sem `resume`**.
4. **Na retomada**, o `step` reapresenta **o mesmo `tool_call` pendente** (mesmo `call_id`, **mesma entrada**) e **reexecuta o pipeline inteiro (passos 1 a 7) numa só transação**, com a execução travada, **antes** de consumir a aprovação. Se entre o pedido e a retomada mudou a permissão, o agente foi desativado, a execução foi cancelada ou o **`definition_hash` carregado difere** do da execução → falha com `definition_changed` (ou o motivo correspondente). **Consequência operacional, explícita:** um deploy que altere a definição **falha as execuções que esperam aprovação** (pode ser dias). É seguro; a alternativa (executar com a definição fixada pelo hash guardado) exige guardar a definição e fica como opção para a Madu (D3.6). O modelo **não** é consultado de novo para reescrever a entrada.
5. A aprovação só vale com **o mesmo hash**, **concedida e não consumida**; é **consumida** na autorização (uso único).
6. **Ferramenta externa com `external_effect` e `idempotent=false`:** a diretiva é emitida **no máximo uma vez** (estado `running` não reemite); sem resultado, o `tool_call` vira `failed` com `outcome_unknown` e vai ao operador. A diretiva só é reemitida se `idempotent=true`.
7. O agente **não concede** aprovação (R8): só token humano.

## 5. Ferramentas externas (PROP)
O gateway **autoriza** (pipeline) e devolve a diretiva `run_external_tool { call_id, tool, input }`. O workflow executa a integração (credencial **do serviço de integração**, no credential store do n8n — nunca uma credencial "do agente") e responde com `tool-calls/:id/result`. O gateway valida o resultado pelo schema de saída, aplica o **guarda de segredos e tamanho**, **marca o conteúdo como dado não confiável**, grava e audita. Repetir o mesmo `call_id` devolve o resultado gravado. Timeout/falha do n8n → `tool_call failed` + `error.detected` (origem `workflow`).

## 6. Injeção de prompt (PROP)
Entrada externa e resultado de ferramenta são **dados**, nunca instruções; o LLM só *pede* ferramenta, **quem decide é o pipeline**. Texto que diga "aprove", "me dê permissão" ou "ignore as regras" não altera permissão, aprovação, escopo nem a lista de ferramentas. Testes desde a F3 e na F6: escalada por mensagem, ferramenta fora do escopo pedida por texto, aprovação "forjada" no texto, argumento que tenta sair do escopo de entidade (escopo **dinâmico**).

## 7. Decisões
| # | Pergunta | Opções | Recomendação |
|---|---|---|---|
| D3.1 | Tool registry | **A)** no código, ações `tool.<nome>` por formato + conferência na carga. B) tabela no banco | **A** |
| D3.2 | Ordem do pipeline e escopos `read`/`write` aplicados | **A)** a ordem da seção 3, escopo dinâmico. B) manter informativos | **A** |
| D3.3 | Approval hook | **A)** aprovação de ação com hash + `awaiting_approval` + retomada na mesma transação da decisão + pipeline reexecutado. B) bloquear esperando | **A** |
| D3.4 | Ferramenta externa com efeito não idempotente | **A)** diretiva no máximo uma vez + `outcome_unknown`. B) reemitir sempre | **A** |
| D3.5 | `test.*` e `event.emit` de teste | **A)** só `NODE_ENV=test`, boot recusa o contrário; a fumaça do M02 roda com perfil de teste e agente de referência no banco descartável | **A** |
| D3.6 | Retomada com definição alterada | **A)** falha com `definition_changed`. B) executa com a definição fixada pelo hash (guardar a definição) | **A:** simples e seguro; B se a espera por aprovação for longa na prática |

## 8. PENDENTE
`timeout_seconds` de cada ferramenta; limite de exibição das aprovações; ferramentas de negócio e schemas (M03/M04); regras nomeadas além de `never`/`always`/`above_budget`; limite e moeda do orçamento (antes do M04).

## 9. Riscos
Volume de aprovações para a Madu · aprovação de objeto que depois muda (PENDENTE do M03) · passo esquecido numa rota nova (inventário + matriz) · efeito externo após cancelamento (seção 3).

## 10. Fora deste bloco
[[spec-agent-04-execucao-estado-e-telemetria]] · [[spec-agent-05-contexto-e-memoria]] · ferramentas de negócio.
