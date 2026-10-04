---
tipo: decisao
area: 02-Decisoes
id: DEC-0015
status: aceita
fonte: decisão da Madu em 2026-10-04; DEC-0010 §7; DEC-0008; blueprint v1 §23, §35–§39; PLANO FINAL §25, §31
data-registro: 2026-10-04
resolve: P10, P11, P12
---
# DEC-0015 — Alocação do roadmap: M01, M03 e M04

## Contexto
Três pontos do roadmap M00–M06 ([[roadmap]]) dependiam de confirmação. Na revisão apareceu uma quarta questão: o Blueprint §37 diz que o M03 (A0–A4) produz um lançamento até `LAUNCH_READY`, mas o workflow Create Launch (§23) passa por **A6 Launch Operations** antes do Approval Gate e do `LAUNCH_READY`, e o A6 pertence ao M04.

## Decisões

### P10: M03/M04 e o caminho crítico
O M04 valida A5–A10 **ao final**. A ordem de ativação do [[DEC-0008-caminho-critico-primeiro]] vale **dentro do M04**: A6, A7, A8, A10, depois A5 e A9. A numeração M00–M06 do DEC-0010 não muda; não há submarcos.

### A6 no M03
Um **A6 mínimo entra no M03**, só o necessário para o Create Launch chegar ao `LAUNCH_READY`. O A6 completo continua no M04. O objetivo do M03 do Blueprint §37 é preservado.
- **PENDENTE:** o que exatamente compõe o "A6 mínimo" (quais workflows e saídas do A6). Definir no Blueprint de Implementação por agente, não aqui.

### P11: `CORE v0.1`
Nome histórico para **M01+M02+M03**. Mapeamento dos itens do Plano Final §31:

| Itens | Marco |
|---|---|
| PostgreSQL, schema inicial, Event Bus, Agent Registry, Permission System, Launch State Machine, Audit Log, Approval Gates (1, 2, 4–7, 14, 15) | M01 |
| Company Memory (3): interfaces de memória | M02; as tabelas da memória estruturada ficam no schema do M01 |
| A0–A4 (8–12) | M03 |
| Primeiro fluxo ponta a ponta (13) | Create Launch até `LAUNCH_READY`, no M03, com o A6 mínimo |

O fluxo ponta a ponta do ciclo completo (lançamento simulado até a análise pós-lançamento) é validado no M04.

### P12: "API skeleton" e "event schema"
Ficam no **M01**. O event schema tem como fonte única `packages/schemas/` ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] D4). O M00 não contém código de aplicação.

## Alternativas rejeitadas
- Dividir o M04 em submarcos: muda a numeração canônica fixada no DEC-0010.
- Abandonar o termo `CORE v0.1`: o Plano Final e o starter continuam citando-o.
- API skeleton no M00: criaria código de aplicação antes do M01, contra a regra de não escrever código sem iniciar o M00 formalmente (DEC-0010 §10).
- A6 completo no M03 ou M03 sem A6: o primeiro contraria o DEC-0010 §7; o segundo tira o `LAUNCH_READY` do M03.

## Consequências
- O critério de pronto do M03 inclui o Create Launch chegando ao `LAUNCH_READY` com o A6 mínimo.
- **PENDENTE:** definição do A6 mínimo (acima).
- Docker/compose (infraestrutura) é M00; os serviços PostgreSQL e Redis são validados no M01.

Ver [[pendencias-de-decisao]] (P10, P11, P12) · [[milestones]] · [[CORE-v0.1]] · [[n8n-estrutura]].
