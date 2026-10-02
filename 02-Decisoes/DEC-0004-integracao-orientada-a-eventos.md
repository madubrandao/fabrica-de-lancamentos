---
tipo: decisao
area: 02-Decisoes
id: DEC-0004
status: aceita-nas-fontes
fonte: PLANO FINAL §18, §28; blueprint v1 §16–§17; starter ADR-0004
data-registro: 2026-10-02
---
# DEC-0004 — Integração orientada a eventos

**Decisão:** workflows e agentes não conversam diretamente; usam um **event bus central** com eventos imutáveis, payload versionado, `correlation_id` obrigatório e `causation_id` quando há evento pai. Eventos importantes são auditáveis.
**Alternativas:** encadear workflow A → B → C (rejeitada).
**Consequências:** [[catalogo-de-eventos]] é contrato central; n8n não guarda estado de negócio ([[DEC-0005-stack-de-producao]]).
