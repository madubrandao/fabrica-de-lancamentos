---
tipo: decisao
area: 02-Decisoes
id: DEC-0011
status: aceita
fonte: decisão da Madu em 2026-10-04; blueprint v1 §16 e §30
data-registro: 2026-10-04
resolve: P04
---
# DEC-0011 — Eventos de aprovação: `requested` / `granted` / `rejected`

## Contexto
O Blueprint v1 usava dois conjuntos de nomes para o domínio de aprovação:
- §16 (eventos de negócio): `approval.requested`, `approval.granted`, `approval.rejected`.
- §30 (observabilidade): `approval.requested`, `approval.completed`.

Eventos são imutáveis e têm payload versionado ([[catalogo-de-eventos]]), então o nome precisa ser único antes de qualquer código.

## Decisão
1. **Hierarquia confirmada:** o catálogo e os nomes de eventos do Blueprint prevalecem sobre o Plano Final (P04).
2. **Padrão único:** `approval.requested`, `approval.granted`, `approval.rejected`.
3. **`approval.completed` não existe.** Fica removido do §30 do catálogo.
4. O §16 é o catálogo de domínio; o §30 é só telemetria e não define evento de aprovação próprio.

## Alternativas rejeitadas
- `requested` / `completed` com `payload.decision`: todo consumidor teria de abrir o payload para saber o desfecho, e o §16 também precisaria mudar.
- Manter os dois nomes: duplicidade e risco de divergência, contra a regra de eventos imutáveis e auditáveis.

## Consequências
- Consumidores (gates, state machine) assinam `approval.granted` ou `approval.rejected` diretamente.
- **PENDENTE:** `approval.expired` (aprovação vencida sem resposta) não consta em nenhum documento. Não foi assumido. Ligado ao D8 (tempo de aprovação) em [[perguntas-em-aberto]].

Ver [[pendencias-de-decisao]] (P04) · [[permissoes-e-hitl]] · [[gates]].
