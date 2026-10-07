---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0010; DEC-0012; DEC-0016; DEC-0017; blueprint v1 §14–§19, §21; PLANO FINAL §16–§17; M01-plano-de-execucao (F1)
status: decisões D2.1 a D2.8 aprovadas pela Madu em 2026-10-07 (recomendações do ENG-00); documento do bloco 2 de 6 ainda a revisar como um todo
atualizado: 2026-10-07
---
# Especificação do Core, bloco 2: entidades e schema

Parte da F1 do [[M01-plano-de-execucao]] ([[DEC-0022-inicio-formal-do-m01]]). **Sem código, sem migration.** As decisões D2.1 a D2.8 foram aprovadas pela Madu em 2026-10-07 com as recomendações do ENG-00 (seção 7). Colunas marcadas PROP seguem como proposta até a revisão final do bloco; nada aqui é schema final. O que os documentos não definem está marcado **PENDENTE**, e as escolhas que dependem dela estão em "Decisões para a Madu". Continua o [[spec-core-01-servicos-e-limites]] (bloco 1, ainda aguardando aprovação).

Blocos da especificação: 1 serviços e limites · **2 entidades e schema** (este) · 3 state machine e gates · 4 eventos · 5 permissões e aprovações · 6 ferramentas de código (ADRs).

**Marcas de origem** usadas nas colunas:
- **DOC**: o campo ou valor está escrito em um documento.
- **DER**: consequência direta de uma regra documentada, sem escolha nova.
- **PROP**: proposta do ENG-00, que a Madu pode recusar.
- **PEND**: os documentos não definem; não foi inventado.

## 1. O que os documentos já fixam
| Regra | Fonte |
|---|---|
| Lista oficial de 22 entidades; `audience` é nome histórico de `AudienceSegment`; `Approval` e `AuditLog` têm tabela própria | [[DEC-0012-lista-de-entidades-do-blueprint]] |
| Uma única `Organization`, mas `organization_id` permanece no modelo e nos eventos | [[DEC-0017-modelo-de-negocio-inicial]] C2 |
| Só a Madu opera e aprova; o `User` fica preparado para crescer | [[M01-plano-de-execucao]] |
| Campos da `launch`: `launch_id`, `organization_id`, `product_id`, `offer_id`, `launch_type`, `objective`, `audience_segment`, `start_date`, `cart_open`, `cart_close`, `status`, `owner_agent`, `approval_state`, `result` | [[modelo-de-entidades]] (Plano Final §17) |
| `launch.status` tem 15 estados; `launch.type` tem 5 valores; os dois nunca se misturam | [[launch-state-machine]] |
| Depois de `DECISION` existe uma decisão de continuidade (SEED, INTERNAL, JOINT, RELAUNCH, EVERGREEN, ARCHIVE); cada ciclo é um Launch próprio e a continuidade liga um ao seguinte | [[launch-state-machine]] |
| Contrato do evento com 12 campos; eventos imutáveis; `correlation_id` obrigatório | [[catalogo-de-eventos]] (Blueprint §17) |
| Estado operacional em estruturas transacionais, nunca só em memória semântica | Blueprint §18, [[DEC-0009-memoria-empresarial-em-tres-tipos]] |
| Permissão do agente: read, write, action scopes, approval requirements, budget limits | [[permissoes-e-hitl]] (Blueprint §19) |
| 12 gates G01-G12; `BUDGET_AUTHORIZED` é policy check registrado no audit log, não é gate | [[gates]] |
| Event store append-only e `audit_log` imutável; migrations reversíveis | [[M01-plano-de-execucao]] F2, guardrails do DEC-0010 §8 |

## 2. Convenções comuns (PROP)
| Regra | Detalhe |
|---|---|
| Chave primária | `id uuid`, sem sequência numérica |
| `organization_id` | `uuid NOT NULL`, FK para `organization`, em toda entidade de negócio (DER de DEC-0012/0017). Hoje há uma linha em `organization` |
| Carimbos | `created_at` e `updated_at`, `timestamptz NOT NULL`. `event` e `audit_log` só têm `occurred_at`, sem `updated_at` |
| Listas fechadas dos documentos | `CHECK` sobre texto (ver D2.2) |
| Exclusão | sem `DELETE` físico em entidade de negócio no M01; ver D2.1 |
| Migrations | reversíveis (up e down); sem migração destrutiva sem aprovação |
| Nomes | tabelas e colunas em inglês e `snake_case`, como no contrato do evento e no `launch` do Plano Final |

## 3. Núcleo detalhado
São as entidades que o M01 precisa de fato para validar eventos, audit, approvals, permissions e state machine.

### 3.1 `organization`
| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `id` | uuid | não | DER | PK |
| `name` | text | não | PROP | |
| `created_at`, `updated_at` | timestamptz | não | PROP | |

Semente do M01: uma linha (DEC-0017 C2).

### 3.2 `user`
| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `id` | uuid | não | DER | PK |
| `organization_id` | uuid | não | DER | FK |
| `name` | text | não | PROP | |
| `email` | text | não | PROP | único por organização |
| `is_active` | boolean | não | PROP | |
| `password_hash` | text | não | DER (bloco 5, D5.1) | só o hash forte da senha; nunca texto puro nem devolvido por resposta de API |
| `created_at`, `updated_at` | timestamptz | não | PROP | |

Autenticação do operador: [[spec-core-05-permissoes-e-aprovacoes]] (D5.1 aprovada). Com um só humano, não há papel nem delegação ([[M01-plano-de-execucao]]). **PEND:** segundo fator e duração da sessão (valores da Madu).

### 3.3 `agent`
| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `id` | uuid | não | DER | PK |
| `organization_id` | uuid | não | DER | FK |
| `code` | text | não | DER | `A0` a `A10` (11 agentes, DEC-0003); único por organização |
| `name` | text | não | PROP | |
| `is_active` | boolean | não | PROP | |
| `permissions` | jsonb | não | PROP | read, write, actions, requires_approval, budget_limits (Blueprint §19); ver D2.5 |
| `permissions_version` | integer | não | PROP | versiona o modelo |
| `created_at`, `updated_at` | timestamptz | não | PROP | |

**PEND:** os escopos dos 11 agentes (só o A5 tem exemplo no Blueprint); o nível de autonomia (AUTO, SUPERVISED, HUMAN DECISION) é por tipo de ação, não por agente, e fica para o bloco 5.

### 3.4 `launch`
| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `id` | uuid | não | DOC (`launch_id`) | PK |
| `organization_id` | uuid | não | DOC | FK |
| `product_id` | uuid | não | DOC | FK `product` |
| `offer_id` | uuid | não | DOC | FK `offer` |
| `launch_type` | text | não | DOC | `CHECK` em SEED, INTERNAL, JOINT, RELAUNCH, EVERGREEN |
| `objective` | text | sim | DOC | texto livre; obrigatoriedade PEND |
| `audience_segment_id` | uuid | sim | DOC (`audience_segment`) | FK `audience_segment`; ver D2.6 |
| `start_date` | timestamptz | sim | DOC | tipo PROP |
| `cart_open` | timestamptz | sim | DOC | tipo PROP |
| `cart_close` | timestamptz | sim | DOC | tipo PROP; `CHECK (cart_close > cart_open)` quando ambos existem (PROP) |
| `status` | text | não | DOC | `CHECK` nos 15 estados; inicial `IDEA` (DER). Só a API altera (R7 do bloco 1) |
| `owner_agent_id` | uuid | sim | DOC (`owner_agent`) | FK `agent`; regra de `owner_agent` por estado é PEND (bloco 3) |
| `approval_state` | text | não | DOC | **valores PEND** (bloco 3 ou 5) |
| `result` | jsonb | sim | DOC | **estrutura PEND** |
| `previous_launch_id` | uuid | sim | DER | FK `launch`; liga o ciclo ao anterior (a continuidade liga um Launch ao seguinte) |
| `created_at`, `updated_at` | timestamptz | não | PROP | |

`status` e `launch_type` são colunas distintas e nenhuma restringe a outra (regra de [[launch-state-machine]]).

### 3.5 `decision` (inclui a decisão de continuidade)
A lista oficial não tem entidade separada para a continuidade, e o plano do M01 lista o modelo dela entre as decisões da Madu (item 6). Proposta: usar `Decision` tipada (ver D2.4).

| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `id` | uuid | não | DER | PK |
| `organization_id` | uuid | não | DER | FK |
| `launch_id` | uuid | sim | DER | FK `launch` de origem |
| `decision_type` | text | não | PROP | `CHECK`; valores iniciais: `CONTINUITY`. Outros tipos PEND |
| `chosen_option` | text | não | DER | para `CONTINUITY`: SEED, INTERNAL, JOINT, RELAUNCH, EVERGREEN, ARCHIVE |
| `next_launch_id` | uuid | sim | DER | FK `launch` criado pela decisão; nulo em EVERGREEN e ARCHIVE |
| `decided_by_type` | text | não | PROP | `human` ou `agent` |
| `decided_by_id` | text | não | PROP | |
| `rationale` | text | sim | PROP | |
| `decided_at` | timestamptz | não | PROP | |
| `created_at` | timestamptz | não | PROP | |

**PEND:** quem decide a continuidade (humano, A0 ou ambos) e os campos de negócio das demais decisões da memória estruturada ([[memoria-empresarial]]).

### 3.6 `approval`
| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `id` | uuid | não | DER | PK |
| `organization_id` | uuid | não | DER | FK |
| `launch_id` | uuid | sim | DER | FK `launch` |
| `gate_id` | text | sim | DER | `CHECK` em G01 a G12; nulo quando o pedido não for de gate |
| `subject_type` | text | não | DER (bloco 5, D5.5) | `CHECK` em `gate` ou `action` |
| `subject_id` | uuid | sim | PROP | |
| `status` | text | não | DER (bloco 5, D5.6) | `CHECK` em `requested`, `granted`, `rejected`, `expired` |
| `requested_by_type`, `requested_by_id` | text | não | DER | quem pediu (agente ou sistema) |
| `requested_at` | timestamptz | não | DER | |
| `decided_by_type` | text | sim | DER (bloco 5, D5.5) | `CHECK` em `human` ou `system`; `system` em gate de VERIFICAÇÃO concedido pela API e em `expired` |
| `decided_by_user_id` | uuid | sim | DER | FK `user`; preenchido quando `decided_by_type = human` |
| `decided_at` | timestamptz | sim | DER | |
| `consumed_at` | timestamptz | sim | DER (bloco 5, D5.5) | marcada na mesma transação em que a aprovação concedida é usada; uso único |
| `reason` | text | sim | PROP | obrigatório ao rejeitar |
| `expires_at` | timestamptz | sim | DER (bloco 5, D5.6) | calculado da política `approval_expiry`; nulo se a política não existir. Valor do prazo: PEND (Madu) |
| `correlation_id` | uuid | não | DER | |
| `created_at`, `updated_at` | timestamptz | não | PROP | |

Constraints propostas:
- `status IN ('granted','rejected')` exige `decided_by_type` e `decided_at`; com `decided_by_type = human` exige também `decided_by_user_id` (R8 do bloco 1: gate HUMANO só por canal humano autenticado). `decided_by_type = system` só vale para gate de VERIFICAÇÃO (bloco 3) e para `expired`.
- `consumed_at` só pode ser preenchido quando `status = granted`.
- `rejected` e `expired` são finais; novo pedido é nova linha (bloco 5, 6.1).
- Sem resposta, `status` permanece `requested` e nada vale como aprovado (regra do DEC-0010).
- `BUDGET_AUTHORIZED` é policy check e **não** vira linha de `approval`; o registro dele fica no `audit_log` ([[gates]]). Gasto acima do limite exige `approval` com `subject_type = action` (bloco 5, seção 7).

### 3.7 `event` (event store, append-only)
As 12 colunas vêm do contrato do Blueprint §17.

| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `event_id` | uuid | não | DOC | PK |
| `event_type` | text | não | DOC | ex.: `launch.cart_opened` |
| `occurred_at` | timestamptz | não | DOC | |
| `organization_id` | uuid | não | DOC | FK |
| `entity_type` | text | não | DOC | |
| `entity_id` | uuid | não | DOC | |
| `actor_type` | text | não | DOC | `CHECK` em `agent`, `human`, `system` |
| `actor_id` | text | não | DOC | `string` no contrato |
| `correlation_id` | uuid | não | DOC | obrigatório |
| `causation_id` | uuid | sim | DOC | FK lógica para outro `event_id` |
| `payload` | jsonb | não | DOC | versionado |
| `schema_version` | integer | não | DOC | |
| `position` | bigint | não | DER (bloco 4, D4.3) | sequência crescente, fora do contrato do Blueprint; ordena a entrega por entidade e serve de cursor |
| `dedupe_key` | text | não | DER (bloco 4, D4.2) | **única**; montada pela API a partir do fato; impede emitir o mesmo fato duas vezes |

Decisões do [[spec-core-04-eventos]] (aprovadas): entrega por outbox no PostgreSQL (tabela `event_delivery`), retenção (`event` não é apagado no M01), ordem por entidade. **PEND:** particionamento e os valores numéricos de entrega.

### 3.8 `audit_log` (imutável)
Os documentos exigem audit em toda mutação crítica e `execution_id` com audit trail, mas **não definem colunas**. Proposta:

| Coluna | Tipo | Nulo | Origem | Nota |
|---|---|---|---|---|
| `id` | uuid | não | DER | PK |
| `organization_id` | uuid | não | DER | FK |
| `occurred_at` | timestamptz | não | DER | |
| `actor_type`, `actor_id` | text | não | DER | quem agiu |
| `action` | text | não | PROP | ex.: transição, aprovação, policy check |
| `entity_type`, `entity_id` | text, uuid | sim | DER | alvo |
| `correlation_id` | uuid | não | DER | |
| `execution_id` | text | sim | DER | guardrail: toda execução tem `execution_id` |
| `event_id` | uuid | sim | PROP | ver D2.7 |
| `outcome` | text | não | PROP | permitido, negado, erro |
| `details` | jsonb | sim | PROP | ex.: `BUDGET_AUTHORIZED = true` e o motivo |

**Constraint:** só `INSERT`; `UPDATE` e `DELETE` bloqueados (ver seção 6). **PEND:** retenção (item 10 das decisões do M01).

### 3.9 Tabelas de infraestrutura (blocos 4 e 5, aprovadas)
Não são entidades de negócio e **não alteram a lista de 22** do DEC-0012. Cada uma entra na F2 com migration reversível e teste de constraints. As colunas estão nos blocos de origem:

| Tabela | Para quê | Onde está definida |
|---|---|---|
| `event_delivery` | controle mutável da entrega de eventos (o `event` é imutável) | [[spec-core-04-eventos]], seção 5 (D4.1) |
| `idempotency_key` | chave de idempotência dos pedidos mutadores | [[spec-core-04-eventos]], seção 6 (D4.2) |
| `agent_credential` | hash da credencial de cada agente, com revogação | [[spec-core-05-permissoes-e-aprovacoes]], seção 3.2 (D5.2) |
| `policy_setting` | política configurável (prazo de aprovação, limites de orçamento), alterável só pela Madu com audit | [[spec-core-05-permissoes-e-aprovacoes]], seção 8 (D5.7) |

## 4. Entidades de domínio (esqueleto)
Valem a partir do M03 em diante. O M01 só reserva tabela, chaves e relações; **as colunas de negócio não estão em nenhum documento e não foram inventadas**. Todas têm `id`, `organization_id`, `created_at` e `updated_at`.

| Entidade | Relações propostas (PROP) | Observação |
|---|---|---|
| `product` | pai de `offer` | colunas de negócio PEND; produto fictício definido só antes do M03 (DEC-0017 C1) |
| `offer` | `product_id` | colunas PEND; eventos `offer.*` existem |
| `audience_segment` | `organization_id`; ligação a `launch` por D2.6 | colunas PEND |
| `contact` | `organization_id` | ver D2.8 sobre `lead` |
| `customer` | pode referenciar `contact_id` | colunas PEND |
| `campaign` | `launch_id` | colunas PEND |
| `content` | `launch_id` | colunas PEND |
| `sequence` | pai PEND (launch, offer ou evergreen) | colunas PEND |
| `partner` | `organization_id`; ligação a `launch` PEND | relação com Launch é N:N? PEND |
| `conversation` | `contact_id` ou `customer_id` | colunas PEND |
| `objection` | pai PEND (offer ou launch) | colunas PEND |
| `experiment` | pai PEND (launch ou campaign) | colunas PEND |
| `metric` | alvo genérico (`entity_type`, `entity_id`) | colunas PEND |
| `learning` | `launch_id` | colunas PEND; liga ao gate G12 |

## 5. Relações (texto)
Cadeia documentada: `PRODUCT → OFFER → LAUNCH → { CONTENT, CAMPAIGN, PARTNERS } → AUDIENCE → { LEADS, CUSTOMERS } → DATA → LEARNING`.

```text
organization 1──N user, agent, product, launch, event, audit_log, approval, decision, ...
product      1──N offer
offer        1──N launch              (um launch pertence a uma oferta)
product      1──N launch
launch       1──N content, campaign, approval, decision, learning   (propostas)
launch       0..1──1 launch           (previous_launch_id: ciclo anterior)
decision     N──1 launch              (origem)   · 0..1──1 launch (next_launch_id)
approval     N──1 launch · N──0..1 user (decided_by_user_id)
agent        1──N launch              (owner_agent_id)
event        N──1 organization · causation_id → event (lógico)
```

Entidades documentadas sem posição fixa nesta cadeia (PEND): `sequence`, `objection`, `experiment`, `metric`, `conversation`, `partner`.

## 6. Imutabilidade e integridade (proposta)
- `event` e `audit_log`: privilégio de `INSERT` e `SELECT` apenas para o usuário da aplicação, mais trigger que lança erro em `UPDATE` e `DELETE`. Teste de imutabilidade na F2 ([[M01-plano-de-execucao]]).
- Usuário do banco da API separado do usuário do n8n (D1.4 do bloco 1).
- Chaves estrangeiras `ON DELETE RESTRICT` por padrão.
- `launch.status` só muda por transição validada pela API; a tabela de transições permitidas é do bloco 3. O banco pode reforçar com `CHECK` na lista de estados, mas **não** codifica a ordem das transições neste bloco.
- Idempotência de eventos e de mutações: bloco 4.

## 7. Decisões (D2.1 a D2.8 aprovadas pela Madu em 2026-10-07: vale a recomendação do ENG-00 em todas)
| # | Pergunta | Opções | Recomendação do ENG-00 |
|---|---|---|---|
| **D2.1** | Exclusão de dados de negócio | **A)** sem `DELETE` físico; encerramento por status (há `ARCHIVE`). **B)** `deleted_at` (soft delete). **C)** `DELETE` físico permitido | **A.** O fluxo já tem encerramento por `ARCHIVE`, e `event` e `audit_log` nunca apagam. B acrescenta filtro em toda consulta. Se houver exigência legal de apagar dados pessoais (contatos), tratar em ADR própria |
| **D2.2** | Listas fechadas (status, tipos, gates) | **A)** `CHECK` sobre texto. **B)** tabelas de domínio com FK. **C)** tipo `ENUM` do PostgreSQL | **A.** Migration reversível simples. `ENUM` do PostgreSQL é pior de reverter. B só se a lista precisar de metadados (ex.: gate com aprovador, que o bloco 3 pode exigir; então revisar) |
| **D2.3** | `approval`: um registro por pedido ou histórico por linha | **A)** uma linha por pedido, atualizada na decisão, com histórico em `event` e `audit_log`. **B)** linhas append-only (pedido e resposta separados) | **A.** Mais simples de consultar o estado atual; o histórico já fica nas duas tabelas imutáveis |
| **D2.4** | Modelo da decisão de continuidade | **A)** `decision` tipada (`CONTINUITY`). **B)** tabela própria | **A.** A lista oficial tem 22 entidades (DEC-0012); tabela própria seria a 23ª e exigiria nova decisão |
| **D2.5** | Permissões do agente | **A)** `jsonb` versionado em `agent`. **B)** tabelas normalizadas de escopos | **A para o M01.** O Blueprint §19 já descreve o modelo como documento. A estrutura final e a validação ficam no bloco 5 e em `packages/schemas/`. Se for preciso consultar "quem pode X" com frequência, migrar para B |
| **D2.6** | `launch.audience_segment` | **A)** FK `audience_segment_id`. **B)** texto livre | **A.** O campo aponta para uma entidade que existe |
| **D2.7** | Relação entre `event` e `audit_log` | **A)** tabelas independentes, ligadas por `correlation_id`, com `event_id` opcional no audit. **B)** audit sempre referencia um evento | **A.** Há ações de audit sem evento (negação de permissão, policy check) |
| **D2.8** | `lead` e `order` nos eventos, sem entidade na lista de 22 | **A)** `lead` = `contact` com estágio; `order` fica fora do M01 e entra com M04. **B)** adicionar entidades `Lead` e `Order` (altera o DEC-0012) | **A.** Os eventos `lead.*` e `order.*` são do Blueprint §16, mas a lista oficial não tem essas entidades. Nada precisa ser decidido para o M01 além de não bloquear |

## 8. PENDENTE para os blocos seguintes
- Tabela de transições, mapeamento estado → gate, `owner_agent` por estado, valores de `approval_state` (bloco 3).
- Payload por evento, idempotência, retenção e entrega confiável (bloco 4).
- Autenticação, escopos dos 11 agentes, aprovador por gate, prazo de expiração da aprovação (bloco 5).
- Tipos exatos, ferramenta de migration e convenção de nomes finais (bloco 6).
- Colunas de negócio das entidades de domínio (a partir do M03, junto da definição do produto fictício).

## 9. Riscos
- **Esqueleto demais ou de menos:** o M01 precisa só do núcleo; as demais tabelas podem ser criadas na F2 apenas como esqueleto ou adiadas para o M03. Isso é uma escolha da Madu e o bloco não a fecha.
- **`approval_state` e `result` sem estrutura:** são campos do `launch` documentados, mas sem valores. Ficam como `PEND` até o bloco 3.
- **Contradição possível com o bloco 3:** se o mapeamento estado → gate exigir metadados de gate, a recomendação D2.2 muda para tabela de domínio.
- **RAM e disco da máquina** ao testar migrations com PostgreSQL ([[M01-plano-de-execucao]] riscos).

## 10. Fora deste bloco
Transições e gates (bloco 3), eventos e entrega (bloco 4), permissões e aprovações em detalhe (bloco 5), ferramentas de código (bloco 6), qualquer SQL ou migration.

Ver [[visao-geral]] · [[modelo-de-entidades]] · [[catalogo-de-eventos]] · [[permissoes-e-hitl]] · [[observabilidade-e-testes]].
