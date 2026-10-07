---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0004; DEC-0005; DEC-0011; DEC-0016 D8; DEC-0020; blueprint v1 §16, §17, §30–§31; catalogo-de-eventos; observabilidade-e-testes; M01-plano-de-execucao (F1, F3)
status: decisões D4.1 a D4.8 aprovadas pela Madu em 2026-10-07 (recomendações do ENG-00); valores numéricos seguem PEND (M01, F1, bloco 4 de 6)
atualizado: 2026-10-07
---
# Especificação do Core, bloco 4: eventos

Parte da F1 do [[M01-plano-de-execucao]] ([[DEC-0022-inicio-formal-do-m01]]). **Proposta, sem código.** Fecha o que os blocos anteriores deixaram para cá: payload dos eventos, entrega, idempotência, retenção, os eventos que faltam para as transições do bloco 3 e o papel do Redis. Tudo marcado **PROP** é proposta; o que os documentos não definem e não pode ser deduzido está marcado **PEND**. Valores numéricos **não foram inventados**: os de negócio (prazo de aprovação, limites) são política ([[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] D8) e os técnicos de entrega (tentativas, esperas, timeout, validade da chave) são configuração de ambiente, como o bloco 5, seção 8, separa. Todos a definir pela Madu.

Blocos: 1 serviços e limites · 2 entidades e schema · 3 state machine e gates · **4 eventos** (este) · 5 permissões e aprovações · 6 ferramentas de código (ADRs). Blocos 1 a 3 com decisões aprovadas em 2026-10-07 ([[spec-core-01-servicos-e-limites]], [[spec-core-02-entidades-e-schema]], [[spec-core-03-state-machine-e-gates]]).

## 1. O que os documentos já fixam
| Regra | Fonte |
|---|---|
| Workflows e agentes não conversam diretamente: usam um event bus central; eventos imutáveis; `correlation_id` obrigatório; `causation_id` quando há evento pai | [[DEC-0004-integracao-orientada-a-eventos]] |
| Contrato do evento com 12 campos; payload versionado; eventos importantes auditáveis | [[catalogo-de-eventos]] (Blueprint §17) |
| Catálogo mínimo de eventos de domínio (§16) | [[catalogo-de-eventos]] |
| Aprovação usa `approval.requested`, `approval.granted`, `approval.rejected`; `approval.completed` não existe; `approval.expired` foi aprovado depois (bloco 5, D5.6; adendo no DEC-0011) | [[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]] |
| Eventos de observabilidade (`agent.execution.*`, `workflow.*`, `api.*`, `llm.*`, `tool.execution`) | Blueprint §30 |
| Operações mutáveis são idempotentes; retry não duplica efeitos | guardrails do ENG-03 |
| Redis: cache, filas e controle de execução assíncrona; PostgreSQL é a fonte da verdade | [[DEC-0005-stack-de-producao]] |
| A API persiste os eventos; o n8n emite pedindo à API (R3); a API aciona o n8n por webhook (D1.2); agendador dentro da API (D1.3) | bloco 1 (aprovado) |
| Event store append-only (tabela `event` com as 12 colunas do contrato) | bloco 2, 3.7 (aprovado) |
| Aprovação sem resposta vale como não aprovada | DEC-0016 D8 |
| Testes exigidos: idempotência, evento duplicado, falha parcial, concorrência | [[observabilidade-e-testes]], F3 e F7 do plano do M01 |

## 2. Envelope e regras gerais (PROP)
O envelope é o do Blueprint §17, sem mudança. Regras do M01:
- **Quem escreve:** só a API grava em `event` (R3). O `event_id` e o `occurred_at` são gerados por ela.
- **`correlation_id`:** nasce na API no início da operação e é propagado em toda chamada, evento e linha de audit. Evento sem ele é recusado.
- **`causation_id`:** o `event_id` do evento que causou este. Numa transição que emite dois eventos (T6), o segundo aponta para o primeiro.
- **`actor_type` e `actor_id`:** `human` com o id do `user`; `agent` com o código (`A0` a `A10`); `system` para o agendador e a própria API.
- **Nome:** `<entidade>.<verbo_no_passado>`, em minúsculas, como o catálogo.
- **Payload sem segredos e sem dados pessoais:** guardrail de segurança. O payload leva identificadores e estado, não conteúdo sensível; o dado pesado fica na entidade.
- **Mesma transação:** mudança de estado, evento, audit e linhas de entrega gravam juntos ou nenhum grava (bloco 3, seção 2). **Exceção (revisão independente):** o audit de um pedido **negado** e o `error.detected` de uma falha são gravados em **transação própria**, para não sumirem junto com o rollback do pedido que falhou.
- **Persistir antes de entregar:** nada é entregue ao n8n sem estar gravado em `event`.

## 3. Eventos que o M01 emite e seus payloads (PROP)
O M01 só tem produtor real para os eventos de launch, aprovação, decisão e erro. Os eventos das entidades de domínio (`product`, `offer`, `lead`, `content`, `campaign`, `order`, `customer`, `partner`, `metric`, `experiment`, `learning`) continuam no catálogo, mas **seus payloads são PEND** e entram junto das entidades, a partir do M03. O M01 entrega o mecanismo (envelope, registro de tipos, entrega) para elas.

Todos os payloads têm `schema_version = 1` no envelope.

| Evento | Quando é emitido | Payload (campos) |
|---|---|---|
| `launch.created` | criação do launch (estado `IDEA`) | `launch_id`, `launch_type`, `product_id`, `offer_id`, `previous_launch_id` (ou nulo), `status`, `owner_agent` |
| `launch.*` de transição (lista na seção 4) | cada transição aceita (T1 a T14) | `launch_id`, `from_status`, `to_status`, `approval_ids[]` (aprovações concedidas que liberaram a transição; vazio sem gate), `gate_ids[]`, `owner_agent` (do novo estado) |
| `approval.requested` | pedido de aprovação criado | `approval_id`, `launch_id` (ou nulo), `gate_id` (ou nulo), `subject_type`, `subject_id` (ou nulo), `action_name` (ou nulo), `requested_by` (tipo e id), `expires_at` (ou nulo; PEND) |
| `approval.granted` | aprovação concedida (pela Madu, ou pela API em gate de VERIFICAÇÃO) | `approval_id`, `launch_id` (ou nulo), `gate_id` (ou nulo), `subject_type`, `action_name` (ou nulo), `decided_by` (tipo e id), `decided_at`, `reason` (ou nulo) |
| `approval.rejected` | aprovação rejeitada | os mesmos campos de `approval.granted` |
| `approval.expired` (novo, bloco 5, D5.6) | o agendador marca como vencida uma aprovação `requested` com `expires_at` vencido | os mesmos campos de `approval.rejected`, com `decided_by.type = system` e `reason` indicando vencimento |
| `decision.recorded` (novo, seção 4) | decisão de continuidade **confirmada** pela Madu (a proposta do A0 gera só `approval.requested`; bloco 3, seção 5) | `decision_id`, `launch_id`, `decision_type`, `chosen_option`, `next_launch_id` (ou nulo) |
| `launch.completed` | cria-se junto da decisão de continuidade (seção 4) | `launch_id`, `decision_id`, `chosen_option` |
| `error.detected` | falha tratada que exige atenção (entrega esgotada, transição inválida repetida, erro de validação grave) | `source` (`api`, `delivery`, `workflow`), `error_code`, `message` (sem segredo), `entity_type`, `entity_id` (ou nulo), `retryable` (booleano) |

Notas:
- O estado e os gates vêm do bloco 3; os campos acima só repetem o que a transição já grava.
- Aprovação negada por regra (agente tentando conceder gate HUMANO) **não gera evento de aprovação**: vai para o `audit_log` com resultado "negado" (bloco 2, 3.8). `error.detected` só se for relevante para operação.
- **Entidade dos eventos de aprovação:** `entity_type = approval` e `entity_id = approval_id`. A ordem de entrega deles é por aprovação, separada da ordem do launch; o launch recebe os efeitos pelos eventos de transição, que levam `approval_ids[]`.
- **`error.detected` sem entidade** (por exemplo, payload inválido na entrada): `entity_type = system` e `entity_id` = o `organization_id` (a coluna `entity_id` do `event` é obrigatória).
- **Sem laço:** se a entrega de um `error.detected` esgotar as tentativas, vira `dead` e gera **só audit**, nunca outro `error.detected`.
- `approval.expired` foi **aprovado** no bloco 5 (D5.6); o prazo continua PEND (valor da Madu).
- Mudança de permissões de agente e outras ações de segurança vão **só para o `audit_log`**, sem evento (bloco 5, seção 9, D5.9).

## 4. Eventos que faltam no catálogo (PROP)
O bloco 3 achou transições sem evento. O Blueprint chama o catálogo de "eventos mínimos", então acrescentar é permitido, desde que a Madu aprove:

| Transição (bloco 3) | Evento proposto | Observação |
|---|---|---|
| T2 `RESEARCH → VALIDATION` | `launch.validation_started` | par de `launch.validation_completed` (T3) |
| T4 `OFFER_DRAFT → PRODUCT_BUILD` | `launch.product_build_started` | segue o padrão do catálogo (estado de destino + `started`); `offer.approved` fica para o M03 |
| T13 `ANALYSIS → LEARNING` | `launch.learning_started` | `learning.created` fica para o M03 |
| T14 `LEARNING → DECISION` | `launch.decision_started` | |
| decisão de continuidade | `decision.recorded` | mutação crítica precisa de evento explícito |

Regras de ligação:
- **T6 emite dois eventos na mesma transação:** `launch.prepre_completed` e depois `launch.plc_started`, com `causation_id` encadeado.
- **`launch.completed`:** o Blueprint lista o evento, mas não diz quando. Proposta: emitido **junto com a decisão de continuidade**, em qualquer das 6 opções, porque nesse ponto o ciclo do launch termina e o próximo, se houver, é outro Launch. Ordem: `decision.recorded` (causa), `launch.completed` (efeito), e `launch.created` do novo launch quando houver.
- Os demais eventos de launch do catálogo seguem o mapeamento da seção 2 do bloco 3 (T1, T3, T5, T6 a T12).

Com isso, **toda transição T1 a T14 tem exatamente um evento de launch**, exceto T6 (dois), e nenhuma transição fica sem rastro. Somados ao `approval.expired`, os eventos novos são **6**.

## 5. Entrega confiável (PROP)
Fluxo, sem componente novo além do que o bloco 1 já definiu:

```text
1. API: numa transação grava estado + evento + audit + uma linha de entrega por assinante
2. Entregador (agendador dentro da API, D1.3): lê as linhas pendentes e chama o webhook do n8n (D1.2)
3. n8n responde 2xx assim que RECEBE o evento: a linha vira "delivered" (recebido). Qualquer outra resposta ou timeout: continua pendente, com nova tentativa
3b. O workflow, ao começar, pede à API o "claim" do evento (seção 6) e, ao terminar, confirma ("ack"): só então o tratamento está concluído
4. Esgotadas as tentativas: a linha vira "dead", a API emite error.detected (exceto se o evento for ele próprio um error.detected) e grava audit
5. Reprocessar uma linha "dead" só pelo operador (a Madu), por canal humano autenticado
```

Garantia: **pelo menos uma vez** (at-least-once). Duplicados são esperados e tratados pelo consumidor (seção 6). Não prometemos "exatamente uma vez".

**Estrutura nova (infraestrutura, não entidade de negócio):** a tabela `event_delivery`, para o controle mutável da entrega, já que `event` é imutável:

| Coluna | Tipo | Nota |
|---|---|---|
| `id` | uuid | PK |
| `event_id` | uuid | FK `event` |
| `subscriber` | text | nome do assinante (workflow ou grupo de workflows do n8n) |
| `status` | text | `CHECK` em `pending`, `delivered` (recebido pelo n8n), `dead` |
| `claimed_at` | timestamptz | preenchida **de forma atômica** quando o workflow pede o claim; impede dois tratamentos do mesmo evento |
| `processed_at` | timestamptz | preenchida no ack do workflow. **Claim com validade:** `claimed_at` sem `processed_at` além do tempo de configuração (**PEND**, valor da Madu) é um claim **vencido**: o entregador devolve a linha a `pending` (reenvio; `attempts` conta), gera alerta e `error.detected`; ao esgotar as tentativas, `dead`. Assim não há perda de evento: continua "pelo menos uma vez" |
| `attempts` | integer | |
| `next_attempt_at` | timestamptz | calculado com espera crescente (backoff) |
| `last_error` | text | sem segredo |
| `delivered_at` | timestamptz | |
| `created_at`, `updated_at` | timestamptz | |

Único por (`event_id`, `subscriber`). Não está na lista de 22 entidades do DEC-0012: é tabela de infraestrutura, como as do n8n, e **precisa de aprovação** (D4.1).

**Assinantes:** a lista de quem assina o quê é configuração versionada em `packages/events/` (bloco 6). Para o M01 precisam existir: (a) o workflow **Create Launch**; (b) o workflow que **retoma o processo que esperava a aprovação**, assinante de `approval.granted`, `approval.rejected` e `approval.expired`; (c) o workflow **NOTIFY** da Madu, assinante de `approval.requested` e `error.detected`. Os workflows (b) e (c) e o claim/ack **não pertencem a um agente**: usam a credencial de **serviço do n8n** (bloco 5, seção 3.2). Os assinantes dos demais workflows entram com M02 e M03 (**PEND**).

**Ordem:**
- Garantida **por entidade e por processamento** (`entity_type` + `entity_id`): o entregador só escolhe uma linha se **não existir** linha anterior da mesma (assinante, entidade) **sem `processed_at`** (`NOT EXISTS` sobre `position` menor). Isso cobre a linha pendente, em envio, recebida e ainda sem ack, e `dead`. Assim `SKIP LOCKED` não passa à frente do evento anterior, e o T8 só é entregue depois de o workflow do T7 confirmar (ack).
- **`dead` bloqueia a entidade** para aquele assinante até o reprocesso pelo operador (bloco 5, seção 5).
- **Assinantes ordenados e não ordenados:** a regra acima vale para os assinantes marcados como **ordenados** na configuração de `packages/events/` (o workflow Create Launch e o de retomada de aprovação). O assinante **NOTIFY** e os eventos `error.detected` são **não ordenados**: não bloqueiam nem são bloqueados, para que um `dead` de notificação não trave os seguintes.
- Mesmo assim, **todo workflow confere o estado atual do launch antes de agir**: a ordem é uma defesa a mais, não a única.
- **Sem ordem global** entre entidades diferentes.
- `event` ganha a coluna `position` (bigint, sequência crescente), fora do contrato do Blueprint. Ela só ordena eventos **da mesma entidade**, o que é seguro porque as escritas de uma entidade são serializadas pelo bloqueio da linha (bloco 3, riscos). **Não é cursor global**: a sequência é atribuída no INSERT, não no commit.

**Redis no M01 (PROP):** fora do caminho que garante a correção. Não há perda de evento se o Redis cair, porque a fila real é a tabela `event_delivery` no PostgreSQL. O Redis é validado no M01 como serviço (sobe, a API conecta, healthcheck verde, senha configurada) e pode servir a coisas não críticas, como cache. Usá-lo para acordar o entregador ou como fila de volume fica para quando houver necessidade (capacidade inicial é 1 lançamento por vez; o DEC-0017 C3 pede a arquitetura **preparada** para 5 simultâneos "com filas": o outbox no PostgreSQL com `event_delivery` já é a fila e comporta isso, e a troca por Redis fica para quando a medição mostrar necessidade).

**Falha parcial (casos a testar):** a API cai depois de gravar e antes de entregar (o entregador retoma da tabela); n8n responde depois do timeout (reenvio gera duplicado e o consumidor deduplica); n8n responde 2xx e a API cai antes de marcar `delivered` (reenvio duplicado, idem); workflow falha depois do 2xx (o claim vence, a linha volta a `pending` e o evento é reenviado; alerta dispara).

**Adendo da F3 (2026-10-07), achados da implementação e da revisão independente** (código em `packages/schemas` e `packages/events` do `launch-os`; decisão técnica no ADR-0018):
- **Claim trava a linha:** o claim faz `SELECT ... FOR UPDATE` antes de decidir, para esperar o entregador confirmar o recebimento. Um `UPDATE` direto com `status = 'delivered'` no `WHERE` devolveria `false` na hora, sobre a versão antiga da linha.
- **Recebido e nunca reivindicado** (acréscimo): o evento `delivered` que nenhum workflow reivindicou além da validade também é reaberto e reenviado (código de erro `not_claimed`). Sem isso, um workflow que respondeu 2xx e nunca chamou o claim deixaria o evento parado, e o "pelo menos uma vez" falharia. `claimValidityMs` passa a valer também como prazo para o workflow começar (e deve ser maior que o tempo limite da chamada).
- **Alerta que falha não derruba o `dead`:** o `error.detected` roda em um `SAVEPOINT`; a `dedupe_key` do alerta inclui o ciclo de reprocesso.
- **`last_error` seguro:** não repete a mensagem do `fetch` (pode citar a URL do webhook, que costuma levar token). A URL é validada (`https`, exceto localhost).
- **Premissa:** a ordem por entidade exige que o chamador serialize as escritas da mesma entidade (bloqueio da linha do launch), porque `position` é atribuída no INSERT.

## 6. Idempotência (PROP)
**Pedido mutador (R5):**
- Todo pedido que muda estado leva o cabeçalho `Idempotency-Key`.
- Tabela `idempotency_key` (infraestrutura, **precisa de aprovação** na D4.2): `organization_id`, `actor_id`, `key`, `request_hash`, `response_status`, `response_ref` (só o **identificador** do que foi criado, nunca o corpo), `created_at`. Único por (`organization_id`, `actor_id`, `key`).
- **Nunca se guarda o corpo da resposta**: ele pode conter segredo (chave de agente criada, token de login), o que contraria o DEC-0010 §8. Os endpoints que devolvem segredo (criação de credencial, login) **não usam resposta guardada**: repetir o pedido nunca revela o segredo de novo.
- Mesma chave e mesmo conteúdo: devolve o status guardado e o recurso referenciado (buscado de novo, sem o segredo), sem repetir efeito nem emitir evento.
- Mesma chave e conteúdo diferente: recusa com erro de conflito.
- A chave é gravada **na mesma transação** do efeito, para não existir efeito sem chave nem chave sem efeito.
- Por quanto tempo guardar a chave: **PEND** (configuração; valor da Madu).

**Evento:**
- `event_id` é único e gerado pela API.
- `event` ganha `dedupe_key` (texto, **único**), montada pela API com o que identifica o fato (por exemplo: tipo do evento + entidade + a transição ou o id da aprovação). Impede que uma repetição interna emita o mesmo fato duas vezes mesmo sem `Idempotency-Key`.
- **Consumidor (n8n):** deduplica por `event_id`. O padrão de workflow já tem VALIDATE no início ([[n8n-estrutura]]); o **claim** entra aí: o workflow pede à API o claim de (`event_id`, assinante); a API faz `UPDATE ... WHERE claimed_at IS NULL` (ou com o claim vencido) e responde se o pedido ganhou a vez ou se o evento já foi tratado. É atômico, sem o "consulta e depois grava" que tem corrida. Como o n8n não guarda estado de negócio (R6), o recibo fica em `event_delivery`, não no n8n.
- **Resposta do webhook:** o n8n responde 2xx ao receber (não ao terminar), para não segurar a conexão; o término é confirmado pelo ack. Validar esse comportamento no n8n 2.41.6 (bloco 1, riscos).

## 7. Versionamento e validação (PROP)
- O contrato de cada tipo de evento (payload e `schema_version`) vive em `packages/schemas/`, **fonte única**. A API valida o payload **antes** de gravar; payload inválido é recusado e vira `error.detected`.
- Evento gravado nunca é reescrito. Mudança que quebra o formato = nova `schema_version`; o produtor emite a versão nova e os consumidores declaram quais versões aceitam.
- Evento com versão que o consumidor não conhece não é descartado em silêncio: a entrega vai para `dead` e gera `error.detected`.
- Acrescentar campo opcional não quebra: mantém a versão (regra a confirmar na ADR de schemas, bloco 6).

## 8. Retenção (PROP)
| Dado | Proposta |
|---|---|
| `event` e `audit_log` | **não são apagados no M01.** São a fonte da verdade e do histórico; qualquer política de arquivamento depois exige decisão da Madu (inclusive o que fazer com dado pessoal no futuro) |
| `event_delivery` | linhas `delivered` podem ser limpas depois de um prazo de configuração (**PEND**, valor da Madu); linhas `dead` ficam até serem tratadas |
| `idempotency_key` | expiram por configuração (**PEND**, valor da Madu) |
| Telemetria (seção 9) | fora do event store; retenção **PEND** (M02) |

## 9. Eventos de observabilidade (PROP)
O Blueprint §30 lista `agent.execution.*`, `workflow.*`, `api.request/error`, `llm.request/error` e `tool.execution`. Eles são de alto volume e de uso diagnóstico, não fatos de negócio.
- **No M01:** não entram em `event`. A API registra `api.request` e `api.error` como logs estruturados com `correlation_id` e `execution_id`.
- **Em M02:** quando houver execução de agentes, a Madu decide onde guardar os demais (tabela própria, stack de logs ou o mesmo event store). O envelope do §17 já serve de formato.
- `error.detected` **fica no event store**, porque é fato que exige atenção e pode acionar workflow.

## 10. Testes e evidências exigidos (F3 e F7)
- **Contrato:** cada evento da seção 3 valida contra o schema; payload com campo faltando, a mais ou com segredo é recusado.
- **Duplicidade:** mesmo `Idempotency-Key` repetido, mesmo evento entregue duas vezes (consumidor deduplica), `dedupe_key` repetida.
- **Falha parcial:** queda entre gravar e entregar; timeout do n8n; n8n fora do ar e voltando; transação que falha no meio não deixa evento, audit ou entrega órfãos.
- **Ordem:** dois eventos da mesma entidade chegam em ordem mesmo com falha no primeiro; entidades diferentes não se bloqueiam.
- **Concorrência:** dois pedidos de transição ao mesmo tempo no mesmo launch geram um único evento (bloco 3, riscos).
- **Imutabilidade:** `UPDATE` e `DELETE` em `event` falham.
- **Encadeamento:** `correlation_id` presente em todo evento; `causation_id` correto em T6 e na decisão de continuidade.
- **Falha total de entrega:** esgota tentativas, vira `dead`, gera `error.detected` e audit; reprocesso só pelo operador. `dead` de um `error.detected` gera só audit (sem laço).
- **Ordem com falha:** `dead` do evento N bloqueia o N+1 da mesma entidade e não bloqueia outras entidades; `SKIP LOCKED` não entrega N+1 com N em voo.
- **Claim e ack:** dois claims simultâneos do mesmo evento, um ganha; claim sem ack vence, dispara alerta e o evento é reenviado (nenhum evento fica "tratado" sem ack).
- **Ordem por processamento:** T8 só é entregue ao assinante ordenado depois do ack de T7; assinante NOTIFY e `error.detected` não são bloqueados por `dead`.
- **Expiração:** `approval.expired` é emitido uma vez, com payload válido.
- **Segredos na idempotência:** a tabela `idempotency_key` nunca contém corpo de resposta, chave de agente nem token (varredura no teste); criação de credencial e login repetidos não revelam o segredo.
- **Transação própria:** pedido negado e `error.detected` deixam audit e evento mesmo quando o pedido original sofre rollback.
- **Redis fora do ar:** nenhuma perda de evento (a entrega não depende dele).

## 11. Decisões (D4.1 a D4.8 aprovadas pela Madu em 2026-10-07: vale a recomendação do ENG-00 em todas)
| # | Pergunta | Opções | Recomendação do ENG-00 |
|---|---|---|---|
| **D4.1** | Como entregar os eventos e qual o papel do Redis no M01 | **A)** outbox no PostgreSQL com tabela `event_delivery` e entregador na API; Redis fora do caminho crítico. **B)** fila no Redis como caminho principal. **C)** combinação (outbox e Redis acordando o entregador) | **A.** Persistência e entrega na mesma base e na mesma transação evitam perda de evento; Redis como caminho principal obrigaria a resolver consistência entre dois sistemas. Custo: uma tabela fora da lista de 22 (`event_delivery`), de infraestrutura. C só quando houver volume |
| **D4.2** | Idempotência | **A)** `Idempotency-Key` com tabela `idempotency_key`, mais `dedupe_key` única no evento e deduplicação por `event_id` no consumidor. **B)** só deduplicar no consumidor | **A.** Cobre pedido repetido, evento repetido e entrega repetida. B deixa o efeito (a transição) repetível. Custo: uma tabela de infraestrutura e uma coluna |
| **D4.3** | Ordem de entrega | **A)** por entidade, sem ordem global (coluna `position`). **B)** ordem global total | **A.** É a ordem que importa (estados de um launch) e é simples de garantir; B trava o sistema inteiro por um evento atrasado **Revisada pela segunda passada (2026-10-07):** a ordem vale por **processamento** (ack) para os assinantes ordenados, e `error.detected` e NOTIFY são não ordenados (seção 5) |
| **D4.4** | Eventos que faltam | **A)** acrescentar os 5 da seção 4 e fixar `launch.completed` junto da decisão de continuidade. **B)** usar um único `launch.status_changed` genérico. **C)** deixar T2, T4, T13 e T14 sem evento | **A.** Mantém o padrão de nomes do catálogo e cada transição com um evento próprio. B muda o catálogo inteiro; C contraria "todos os eventos importantes auditáveis" |
| **D4.5** | Retenção | **A)** `event` e `audit_log` não apagam no M01; limpeza só de `event_delivery` (entregues) e `idempotency_key`, com prazos definidos por você depois. **B)** definir prazos agora | **A.** Não invento número (D8 do DEC-0016); apagar histórico sem necessidade é o risco maior |
| **D4.6** | Telemetria (`api.*`, `llm.*`, `workflow.*`, `agent.execution.*`, `tool.execution`) | **A)** fora do event store no M01 (logs estruturados), decisão de armazenamento no M02. **B)** tudo no event store desde já | **A.** Alto volume e sem consumidor de negócio; só `error.detected` entra no store |
| **D4.7** | Falha de entrega | **A)** tentativas com espera crescente, depois `dead`, `error.detected` e audit; reprocesso só pelo operador. Números (tentativas, esperas, timeout) como configuração, com valores definidos por você. **B)** tentar para sempre | **A.** B esconde a falha e acumula carga. Os valores ficam como PENDENTES para você, sem número inventado |
| **D4.8** | Versionamento do payload | **A)** contrato em `packages/schemas/`; validação antes de gravar; quebra = nova `schema_version`; versão desconhecida vai para `dead` com `error.detected`. **B)** sem validação central | **A.** É a regra do Blueprint (payload versionado) e a fonte única do M00 |

## 12. PENDENTE
- Payload dos eventos de domínio de M03 em diante e o assinante de cada workflow.
- Valores numéricos: tentativas, espera entre tentativas, timeout, validade da chave de idempotência, prazo de limpeza das entregas (a Madu define).
- `approval.expired` está **aprovado** (D5.6); só o **valor do prazo** continua PEND (a Madu define).
- Mudança de permissão de agente: **só audit**, sem evento (D5.9, resolvido).
- Onde guardar a telemetria de agentes (M02).
- Autenticação do webhook do n8n e da chamada do n8n à API: **resolvida** no bloco 5 (D5.1 e D5.2); o replay do webhook é limite conhecido (bloco 5, 3.3).
- Biblioteca de fila, de validação de schema e de agendamento (bloco 6).

## 13. Riscos
- **Duplicação de entrega** é inevitável (at-least-once): o risco é o consumidor não ser idempotente. Mitigação: deduplicação por `event_id` obrigatória no passo VALIDATE de todo workflow, com teste de evento duplicado.
- **Entregador dentro da API** compete por recursos com a API. Aceitável na capacidade inicial (1 lançamento por vez); se crescer, vira `apps/worker` por nova decisão (D1.3 do bloco 1).
- **Linhas `dead` acumuladas sem alerta:** o `error.detected` e o painel de eventos (M06) precisam mostrá-las. Até o M06, o aviso é pelo canal do operador do bloco 1.
- **Tabelas de infraestrutura** (`event_delivery`, `idempotency_key`) estendem o schema do bloco 2; precisam entrar na F2 com migration reversível.
- **RAM da máquina** ao testar entrega com PostgreSQL, Redis, n8n e API juntos ([[M01-plano-de-execucao]], riscos).

## 14. Fora deste bloco
Permissões, autenticação e prazos de aprovação (bloco 5), ferramentas de código (bloco 6), qualquer SQL, migration ou código.

Ver [[catalogo-de-eventos]] · [[observabilidade-e-testes]] · [[launch-state-machine]] · [[gates]].
