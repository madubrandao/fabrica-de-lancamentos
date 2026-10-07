---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0010 §3, §5–§6; DEC-0016 D7, D8; DEC-0011; blueprint v1 §13, §15–§16, §19–§21; launch-state-machine; gates; permissoes-e-hitl; M01-plano-de-execucao (F1)
status: decisões D3.1 a D3.8 aprovadas pela Madu em 2026-10-07 (recomendações do ENG-00); critérios objetivos dos gates seguem PEND (M01, F1, bloco 3 de 6)
atualizado: 2026-10-07
---
# Especificação do Core, bloco 3: state machine e gates

Parte da F1 do [[M01-plano-de-execucao]] ([[DEC-0022-inicio-formal-do-m01]]). **Proposta, sem código.** Este bloco resolve a pendência que o [[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] (D7) deixou para o ENG-00: o mapeamento estado → gate, quem aprova cada gate, `owner_agent` por estado e a decisão de continuidade. **Os documentos não ligam gates a transições nem definem aprovadores**: tudo que está marcado **PROP** é proposta e só vale depois da aprovação da Madu. O que continua sem definição está marcado **PEND**.

Blocos: 1 serviços e limites · 2 entidades e schema · **3 state machine e gates** (este) · 4 eventos · 5 permissões e aprovações · 6 ferramentas de código (ADRs). Os blocos 1 e 2 têm as decisões D1.x e D2.x aprovadas em 2026-10-07 ([[spec-core-01-servicos-e-limites]], [[spec-core-02-entidades-e-schema]]).

## 1. O que os documentos já fixam
| Regra | Fonte |
|---|---|
| 15 estados em sequência linear, de `IDEA` a `DECISION` | [[launch-state-machine]] (DEC-0010 §5) |
| `launch.status` tem só estados de execução; `launch.type` (SEED, INTERNAL, JOINT, RELAUNCH, EVERGREEN) é outra coluna | DEC-0010 §5–§6 |
| Depois de `DECISION` há uma decisão de continuidade com 6 opções; cada ciclo é um Launch próprio | DEC-0010 §5 |
| 12 gates G01 a G12; "nenhuma transição crítica ocorre sem o gate correspondente" | [[gates]] (DEC-0010 §3) |
| `BUDGET_AUTHORIZED = true` é policy check, registrado no audit log, **não** é o 13º gate | DEC-0010 §3 |
| Os gates G04 a G07 caem todos entre `PLC` e `CART_OPEN`: a ligação não é automática | DEC-0016 D7 |
| Joint Launch só depois de um Internal Launch ter testado oferta e sequência; "vira um gate do nosso sistema" | Blueprint §13 (A9) |
| Aprovação sem resposta vale como **não aprovada** (fail-closed) | DEC-0016 D8 |
| Só a API transiciona; nenhum agente contorna um gate; aprovação só por canal humano autenticado | R7 e R8 do bloco 1 |
| Níveis AUTO, SUPERVISED, HUMAN DECISION e a lista de aprovação humana por padrão (go/no-go de lançamento, preço, garantia, oferta, orçamento) | [[permissoes-e-hitl]] |
| Só a Madu opera e aprova | [[M01-plano-de-execucao]] |

## 2. Transições: tabela proposta (PROP)
São 14 transições, todas para frente, entre estados vizinhos. A coluna "Gate" é a proposta de mapeamento (D3.1). "Evento" é o que o [[catalogo-de-eventos]] já tem; onde não existe evento para a transição, está marcado **sem evento no catálogo** e a decisão fica para o bloco 4 (não criei evento novo).

| # | De → Para | Gate exigido | Evento já existente no catálogo |
|---|---|---|---|
| T1 | `IDEA` → `RESEARCH` | nenhum | `launch.research_started` |
| T2 | `RESEARCH` → `VALIDATION` | nenhum | sem evento no catálogo |
| T3 | `VALIDATION` → `OFFER_DRAFT` | **G01** Market Validated | `launch.validation_completed` |
| T4 | `OFFER_DRAFT` → `PRODUCT_BUILD` | **G02** Offer Approved | sem evento de launch (existe `offer.approved`) |
| T5 | `PRODUCT_BUILD` → `PRE_PRE` | **G03** Product Ready | `launch.prepre_started` (existe `product.approved`) |
| T6 | `PRE_PRE` → `PLC` | **G04** Content/PLC Approved | `launch.prepre_completed` e `launch.plc_started` |
| T7 | `PLC` → `LAUNCH_READY` | **G05** CRM Ready + **G06** Technical QA Passed + **G07** Launch Ready | `launch.launch_ready` |
| T8 | `LAUNCH_READY` → `CART_OPEN` | **G08** Cart Open Authorized | `launch.cart_opened` |
| T9 | `CART_OPEN` → `CART_CLOSING` | nenhum | `launch.cart_closing` |
| T10 | `CART_CLOSING` → `CART_CLOSED` | **G09** Cart Close Authorized | `launch.cart_closed` |
| T11 | `CART_CLOSED` → `POST_LAUNCH` | nenhum | `launch.postlaunch_started` |
| T12 | `POST_LAUNCH` → `ANALYSIS` | **G10** Post Launch Complete | `launch.analysis_started` |
| T13 | `ANALYSIS` → `LEARNING` | **G11** Analysis Complete | sem evento de launch (existe `learning.created`) |
| T14 | `LEARNING` → `DECISION` | **G12** Learning Recorded | sem evento no catálogo (`launch.completed` fica para o bloco 4) |

Regras da máquina (PROP):
- **Só avança para o estado seguinte.** Sem pular estado e sem voltar. Qualquer outra transição é recusada e vai para o audit (resultado "negado").
- **Gate rejeitado ou sem resposta não libera**: o launch fica no estado atual (fail-closed).
- Uma transição com mais de um gate (T7) só ocorre com **todos** os gates concedidos.
- A transição, o evento e o audit são gravados **na mesma transação** da API; se um falhar, nenhum vale (R2 e R3 do bloco 1).
- Repetir o pedido com a mesma chave de idempotência (R5) não repete a transição nem os eventos.
- Quem pede a transição é o agente dono do estado, via API; quem decide é a API (R7).

## 3. Gates (PROP)
Cada gate tem um **tipo**:
- **HUMANO:** só a Madu concede, por canal humano autenticado (R8).
- **VERIFICAÇÃO:** o agente dono pede, e a API concede sozinha **se** a condição objetiva estiver verdadeira no banco. Agente não aprova nada: ele só dispara a checagem.

| Gate | Nome | Tipo proposto | Critério (o que a API checa ou o que a Madu avalia) | Por quê |
|---|---|---|---|---|
| G01 | Market Validated | HUMANO | critério **PEND** (resultado de pesquisa do A1) | "estratégia central" é HUMAN DECISION |
| G02 | Offer Approved | HUMANO | critério **PEND** (oferta, preço, garantia do A2) | oferta, preço e garantia estão na lista de aprovação humana |
| G03 | Product Ready | HUMANO | critério **PEND** | "mudança crítica de produto" é HUMAN DECISION; com um só humano o custo é baixo |
| G04 | Content/PLC Approved | HUMANO | critério **PEND** (copy principal e PLC do A3) | copy principal e PLC são SUPERVISED: o agente prepara, o humano aprova |
| G05 | CRM Ready | VERIFICAÇÃO | critério **PEND** (configuração do A4/A6); sem critério definido o gate fica HUMANO | **PROP:** etapa operacional; nenhum documento classifica o G05 como AUTO ou de baixo risco, e **não há critério objetivo definido** |
| G06 | Technical QA Passed | VERIFICAÇÃO | QA de sales page, links, order form, checkout, thank-you, confirmação, CRM e tracking ([[n8n-estrutura]]); resultado do A6 registrado. Itens objetivos **PEND** | idem |
| G07 | Launch Ready | HUMANO | G05 e G06 concedidos + critério **PEND** | fecha o preparo do lançamento |
| G08 | Cart Open Authorized | HUMANO | G07 concedido (conta como concedido **mesmo já usado** na transição T7: a checagem aceita `granted` com `consumed_at` preenchido, para o mesmo launch); `BUDGET_AUTHORIZED = true` quando houver gasto | "go/no-go de lançamento" é aprovação humana por padrão |
| G09 | Cart Close Authorized | HUMANO | critério **PEND** | **PROP:** decisão de encerrar a venda; nenhum documento define o aprovador |
| G10 | Post Launch Complete | VERIFICAÇÃO | critério **PEND** (entregas pós-lançamento do A8) | etapa operacional |
| G11 | Analysis Complete | VERIFICAÇÃO | critério **PEND** (análise do A10 registrada) | etapa AUTO |
| G12 | Learning Recorded | VERIFICAÇÃO | **PROP:** existe ao menos 1 registro `learning` ligado ao launch | o nome do gate já é a condição |

Notas:
- **Enquanto o critério objetivo de um gate de VERIFICAÇÃO estiver PEND, o gate é tratado como HUMANO.** Assim nenhum gate fica sem controle por falta de definição. É a escolha de menor risco.
- **O A0 não concede gate.** A saída do A0 no Blueprint §13 inclui "approvals" e "next state", mas a lista "pode decidir" fala de sequência, delegação e prioridade, e os gates HUMANO são decisão da Madu. É uma **escolha aprovada** (D3.2 e D5.4), não um fato dos documentos.
- Os critérios objetivos de cada gate **não existem nos documentos**. A Madu os define, e cada um vira ADR ou DEC. Nada foi inventado.
- Gate é regra do sistema, não checklist de documentação ([[gates]]).

## 4. `owner_agent` por estado (PROP)
Derivado das entradas e saídas dos agentes no Blueprint §13. É o agente responsável pelo estado e que pede a transição de saída dele.

| Estado | `owner_agent` | Base |
|---|---|---|
| `IDEA` | A0 | A0 decide sequência e inicia processos não críticos |
| `RESEARCH` | A1 | pesquisa de mercado, avatar, VOC |
| `VALIDATION` | A1 | mapa de oportunidades e objeções |
| `OFFER_DRAFT` | A2 | spec de produto e oferta, preço, garantia |
| `PRODUCT_BUILD` | A2 | spec de produto |
| `PRE_PRE` | A3 | mensagens e conteúdo; apoio do A4 **PEND** |
| `PLC` | A3 | PLC1 a PLC3, sequência de e-mails, sales page |
| `LAUNCH_READY` | A6 | cronograma, QA, alertas operacionais |
| `CART_OPEN` | A6 | transições de estado e alertas; apoio do A7 **PEND** |
| `CART_CLOSING` | A6 | idem |
| `CART_CLOSED` | A6 | idem |
| `POST_LAUNCH` | A8 | onboarding, suporte, feedback |
| `ANALYSIS` | A10 | diagnósticos e anomalias |
| `LEARNING` | A10 | learnings |
| `DECISION` | A0 | decisões, prioridades e próximo estado |

O A5 (campanhas) e o A9 (parceiros) não são donos de estado: atuam dentro de estados (A5 depois do caminho crítico, A9 a partir de JOINT). Isso fica **PEND** para o bloco 5.

## 5. Decisão de continuidade (PROP)
Modelo de dados já definido no bloco 2: `decision` com `decision_type = CONTINUITY`, `chosen_option`, `next_launch_id`.

Fluxo, depois de T14 (`DECISION`):
1. O A0 prepara a decisão de continuidade (A0 sugere; a opção é uma decisão de negócio). **Quem confirma é PEND**: ver D3.4.
2. A API grava a `decision`, o evento e o audit **na mesma transação** que cria o efeito:

| Opção | Efeito (DEC-0010 §5) | Dados |
|---|---|---|
| `SEED` | novo Launch tipo SEED | `next_launch_id` preenchido; `previous_launch_id` no novo launch |
| `INTERNAL` | novo Launch tipo INTERNAL | idem |
| `JOINT` | novo Launch tipo JOINT | idem; só com a regra da seção 6 |
| `RELAUNCH` | novo ciclo da mesma oferta | idem; mesma `offer_id` |
| `EVERGREEN` | encaminha a uma sequência perene | `next_launch_id` nulo; detalhes **PEND** (entidade `sequence` é esqueleto) |
| `ARCHIVE` | encerra o ciclo | `next_launch_id` nulo; o launch fica encerrado |

3. O novo Launch nasce em `IDEA`, com os 15 estados e os 12 gates outra vez.
4. Um Launch com decisão registrada **não recebe uma segunda decisão de continuidade** (`CHECK` ou unicidade por launch, PROP).

Como "fechado" aparece no banco depois de `ARCHIVE` ou de uma decisão que cria o próximo ciclo é **PEND**: o `launch.status` não tem estado fora dos 15, e não criei um.

## 6. Regra do Joint Launch (PROP)
O Blueprint pede que "Joint só após Internal ter testado oferta e sequência" **vire um gate**, mas a lista tem 12 gates e o DEC-0010 fechou em 12. Proposta: tratar como **policy check de criação** (como o `BUDGET_AUTHORIZED`), sem 13º gate:
- Criar um Launch `JOINT` exige que exista, para a mesma oferta, um Launch `INTERNAL` que chegou a `DECISION`.
- A negativa vai para o audit.
- O critério de "testou oferta e sequência" além disso é **PEND** (depende de métricas, M05).

## 7. `approval_state` do launch (PROP)
O bloco 2 deixou os valores em aberto. Proposta, derivados do `approval` mais recente do gate exigido pela transição seguinte:

| Valor | Quando |
|---|---|
| `NOT_REQUIRED` | a próxima transição não tem gate, ou o launch está em `DECISION` (não há transição seguinte) |
| `PENDING` | existe `approval` em `requested` para a transição seguinte |
| `GRANTED` | todos os gates da transição seguinte estão concedidos |
| `REJECTED` | o último pedido de gate foi rejeitado |
| `EXPIRED` | o último pedido de gate venceu sem resposta (bloco 5, D5.6) |

É coluna **derivada** das linhas de `approval` (bloco 2, 3.6), que são a fonte da verdade. A coluna é persistida em `launch`, então ela é atualizada **na mesma transação** de qualquer mudança em `approval` daquele launch, e há teste de consistência entre a coluna e as linhas (revisão independente).

## 8. Auditoria e testes que esta especificação exige
- **Audit em toda transição**, aceita ou negada: ator, estado de origem e destino, gates verificados, resultado, `correlation_id`, `execution_id`.
- Eventos de aprovação (`approval.requested/granted/rejected`) ligados ao gate e ao launch ([[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]]).
- Testes da F4 e F7, **positivos e negativos de cada transição e de cada gate**:
  - Transição sem gate: ok. Transição com o gate pendente, rejeitado ou ausente: recusada. T7 com só 2 dos 3 gates: recusada.
  - Pular estado, voltar estado e transição repetida: recusadas; repetida com a mesma chave de idempotência: sem efeito duplicado.
  - Agente tentando conceder gate HUMANO ou alterar aprovação: recusado (R8).
  - Decisão de continuidade duplicada: recusada. JOINT sem INTERNAL anterior: recusado.
  - Atomicidade: falha entre mudar o estado e gravar evento/audit não deixa estado parcial (failure injection, F7).

## 9. Decisões (D3.1 a D3.8 aprovadas pela Madu em 2026-10-07: vale a recomendação do ENG-00 em todas)
| # | Pergunta | Opções | Recomendação do ENG-00 |
|---|---|---|---|
| **D3.1** | Mapeamento estado → gate | **A)** a tabela da seção 2: G01 em T3, G02 em T4, G03 em T5, G04 em T6, G05 + G06 + G07 em T7, G08 em T8, G09 em T10, G10 a G12 em T12 a T14; T1, T2, T9 e T11 sem gate. **B)** um gate em cada transição possível, mudando a lista | **A.** Respeita as ligações de nome (G08 é "Cart Open", G09 é "Cart Close") e coloca G04 a G07 entre `PLC` e `CART_OPEN`, como o DEC-0016 pede. Só há 3 transições entre `PRE_PRE` e `CART_OPEN` para 4 gates, então G05, G06 e G07 dividem T7 |
| **D3.2** | Tipo de cada gate (quem concede) | **A)** HUMANO (a Madu) em G01 a G04, G07 a G09; VERIFICAÇÃO em G05, G06, G10, G11, G12, com a regra de que, sem critério objetivo definido, o gate é HUMANO. **B)** todos HUMANO. **C)** todos VERIFICAÇÃO | **A.** Respeita os níveis do documento de HITL sem automatizar o que não tem critério. Com uma só humana, B sobrecarrega com 12 aprovações por ciclo; C contraria a lista de aprovação humana por padrão |
| **D3.3** | `owner_agent` por estado | **A)** a tabela da seção 4. **B)** só A0 em todos | **A.** Segue as saídas de cada agente no Blueprint. Na prática o dono só importa para quem pode pedir a transição de saída (bloco 5) |
| **D3.4** | Quem confirma a decisão de continuidade | **A)** só a Madu. **B)** A0 propõe e a Madu confirma. **C)** A0 decide | **B.** O A0 sugere com base em métricas e learnings; a escolha (relançar, parceria, arquivar) é estratégica e está na lista de HUMAN DECISION. C contraria o Blueprint |
| **D3.5** | Transições para trás e abandono antes de `DECISION` | **A)** não existem no M01: só avança, e um launch parado fica parado. **B)** criar estado `ABORTED` ou transição de retorno | **A.** Nenhum documento define estado fora dos 15; acrescentar um muda o DEC-0010. Registro como **PEND** e a Madu decide quando surgir a necessidade |
| **D3.6** | Regra do Joint Launch | **A)** policy check de criação (seção 6). **B)** 13º gate | **A.** O DEC-0010 fechou em 12 gates e já tem o precedente do `BUDGET_AUTHORIZED` como policy check |
| **D3.7** | `approval_state` | **A)** os 4 valores da seção 7, derivados. **B)** coluna independente | **A.** Uma única fonte da verdade (`approval`) evita estado duplicado |
| **D3.8** | Texto do critério de pronto do M01: "as 12 transições com gate testadas" | **A)** reescrever para "os 12 gates e as transições que os exigem". **B)** manter | **A.** Com a tabela da seção 2 há **10 transições com gate** cobrindo **12 gates**, porque G05, G06 e G07 dividem T7. O texto atual não bate com isso e precisa ser corrigido para o critério ser verificável |

## 10. PENDENTE para os outros blocos
- Critérios objetivos de cada gate (a Madu define; cada um vira ADR ou DEC).
- Eventos faltantes para T2, T4, T13 e T14 e o momento de `launch.completed` (bloco 4).
- Prazo de expiração da aprovação e `approval.expired` (bloco 5, DEC-0016 D8).
- Quais agentes podem pedir cada transição e o apoio de A4, A5, A7 e A9 dentro dos estados (bloco 5).
- Como o launch fica "encerrado" depois de `ARCHIVE`; `EVERGREEN` na prática (M03 em diante).
- Critério de "Internal testou oferta e sequência" (M05).

## 11. Riscos
- **Mapeamento é escolha de projeto**, não fato dos documentos. Se a Madu mudar a D3.1, mudam T-table, testes e aprovadores. Por isso a tabela vive como constante versionada em `packages/schemas/` (fonte única), não espalhada no código (ADR do bloco 6).
- **Gate sem critério objetivo** vira aprovação manual: custo de tempo da Madu, aceito por ser fail-closed.
- **Eventos faltando** no catálogo para quatro transições: o bloco 4 precisa decidir sem inventar mais do que o necessário.
- **Concorrência:** dois pedidos de transição ao mesmo tempo no mesmo launch. Mitigação: bloqueio da linha do launch na transação e checagem do estado de origem (detalhe no plano da F4).

## 12. Fora deste bloco
Payload e entrega dos eventos (bloco 4), autenticação, permissões e prazos de aprovação (bloco 5), ferramentas de código (bloco 6), qualquer SQL ou código.

Ver [[launch-state-machine]] · [[gates]] · [[permissoes-e-hitl]] · [[catalogo-de-eventos]] · [[modelo-de-entidades]].
