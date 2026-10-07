---
tipo: arquitetura
area: 03-Arquitetura
fonte: blueprint v1 §16–§17, §30; PLANO FINAL §18
status: Blueprint prevalece (P04 resolvido; approval.* padronizado pelo DEC-0011)
atualizado: 2026-10-04
---
# Catálogo de eventos e contrato de evento

## Contrato-base (Blueprint §17)
```json
{
  "event_id": "uuid",
  "event_type": "launch.cart_opened",
  "occurred_at": "ISO-8601",
  "organization_id": "uuid",
  "entity_type": "launch",
  "entity_id": "uuid",
  "actor_type": "agent|human|system",
  "actor_id": "string",
  "correlation_id": "uuid",
  "causation_id": "uuid|null",
  "payload": {},
  "schema_version": 1
}
```
**Regras:** eventos imutáveis; payload versionado; `correlation_id` obrigatório; `causation_id` quando houver evento pai; eventos importantes auditáveis.

## Eventos mínimos (Blueprint §16)
- **product:** created · updated · approved
- **offer:** created · updated · approved
- **launch:** created · research_started · validation_completed · prepre_started · prepre_completed · plc_started · launch_ready · cart_opened · cart_closing · cart_closed · postlaunch_started · analysis_started · completed
- **lead:** created · engaged · segmented
- **content:** created · approved · published
- **campaign:** created · started · updated · paused
- **order:** created · paid · refunded
- **customer:** created · onboarded · support_requested
- **partner:** created · qualified · activated · sale_attributed
- **metric.recorded** · **experiment:** created · completed · **learning.created**
- **approval:** requested · granted · rejected
- **error.detected**

(Plano Final §18 usa `launch.ready` e não lista os itens em negrito acima que o Blueprint acrescenta; ver P04.)

## Eventos de observabilidade (Blueprint §30)
`agent.execution.started/completed/failed` · `workflow.started/completed/failed` · `api.request` · `api.error` · `llm.request` · `llm.error` · `tool.execution`
Observação: os eventos de aprovação foram padronizados pelo [[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]]. `approval.completed` (§30) **foi removido**; vale `approval.requested/granted/rejected` do §16. `approval.expired` foi **aprovado** (D5.6, [[spec-core-05-permissoes-e-aprovacoes]]); o prazo de aprovação segue PENDENTE (D8).

## Pendente
Payload de cada evento, produtor/consumidor por evento, política de idempotência e retenção. Responsável: ENG-00 + ENG-02 ([[engenharia-eng-00-a-07]]). Decisões D4.x aprovadas pela Madu em 2026-10-07 em [[spec-core-04-eventos]] (payloads, entrega, idempotência, retenção, 6 eventos novos, incluindo `approval.expired`); valores numéricos seguem PEND.
