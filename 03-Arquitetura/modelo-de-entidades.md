---
tipo: arquitetura
area: 03-Arquitetura
fonte: blueprint v1 §14; PLANO FINAL §16–§17
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# Modelo de entidades

## Entidades (Blueprint §14 — lista vigente, ver P05 em [[pendencias-de-decisao]])
Organization · User · Agent · Product · Offer · Launch · Contact · AudienceSegment · Campaign · Content · Sequence · Partner · Customer · Conversation · Objection · Experiment · Metric · Event · Decision · Learning · Approval · AuditLog

(O Plano Final lista as mesmas, com `audience` no lugar de `AudienceSegment` e sem `Approval`/`AuditLog`.)

## Entidade central `launch` (Plano Final §17)
`launch_id · organization_id · product_id · offer_id · launch_type · objective · audience_segment · start_date · cart_open · cart_close · status · owner_agent · approval_state · result`

`launch_type`: SEED · INTERNAL · JOINT · RELAUNCH · EVERGREEN.

## Relações
```text
PRODUCT → OFFER → LAUNCH → { CONTENT, CAMPAIGN, PARTNERS } → AUDIENCE → { LEADS, CUSTOMERS } → DATA → LEARNING
```

## Pendente
Colunas, tipos, chaves e constraints de cada entidade **não estão definidos** nos documentos. Serão produzidos pelo ENG-02 DATABASE a partir da especificação do ENG-00 ARCHITECT (Blueprint §41). Nada aqui deve ser lido como schema final.
