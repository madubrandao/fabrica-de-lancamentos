---
tipo: decisao
area: 02-Decisoes
id: DEC-0012
status: aceita
fonte: decisão da Madu em 2026-10-04; blueprint v1 §14; PLANO FINAL §16–§17
data-registro: 2026-10-04
resolve: P05
---
# DEC-0012 — Lista oficial de entidades: Blueprint §14 (22 entidades)

## Contexto
Plano Final (§16) e Blueprint (§14) listam entidades quase idênticas. Há 3 diferenças:

| Entidade | Plano Final | Blueprint §14 |
|---|---|---|
| Audiência | `audience` | `AudienceSegment` |
| Aprovação | não consta | `Approval` |
| Auditoria | não consta | `AuditLog` |

Evidências: o Plano Final já lista "Approval Engine", "Audit Log" e "Approval Gates" como componentes e tem `approval_state` em `launch` (§17); usa `audience_segment` na estrutura do repositório e como campo de `launch`. Os eventos `approval.*` do [[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]] pressupõem a entidade `Approval`.

## Decisão
1. **Lista oficial = Blueprint §14, 22 entidades:** Organization · User · Agent · Product · Offer · Launch · Contact · AudienceSegment · Campaign · Content · Sequence · Partner · Customer · Conversation · Objection · Experiment · Metric · Event · Decision · Learning · Approval · AuditLog.
2. `audience` (Plano Final) é **nome histórico** de `AudienceSegment`. Usar `AudienceSegment`.
3. `Approval` e `AuditLog` são entidades próprias, com tabela.

## Consequências
- Colunas, tipos, chaves e constraints continuam **PENDENTES**: ENG-02 DATABASE, a partir da especificação do ENG-00 ARCHITECT. Esta decisão fixa só a lista e os nomes.
- A entidade central `launch` (Plano Final §17) mantém os campos listados em [[modelo-de-entidades]].

Ver [[pendencias-de-decisao]] (P05) · [[modelo-de-entidades]].
