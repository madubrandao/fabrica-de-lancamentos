---
tipo: agente
area: 05-Agentes
id: A4
camada: producao
fonte: PLANO FINAL §7; blueprint v1 §13
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A4 — Audience & CRM

**Absorve:** audiência + CRM + relacionamento. Plano Final Fase 6.

## Missão
Administrar o ativo central da metodologia: a lista e a relação com ela. Construir a lista e investir continuamente na relação é um dos princípios empresariais fundamentais.

## Entradas / Saídas
- **Entra:** contacts, events, segments, launch state.
- **Sai:** tags, segments, sequences, nurture actions, recovery actions.

## Workflows (15)
A4-01 Capturar lead · A4-02 Criar contato · A4-03 Enriquecer contato · A4-04 Segmentar · A4-05 Aplicar tags · A4-06 Nutrição · A4-07 Convite para lançamento · A4-08 Segmentação de lançamento · A4-09 Engajamento · A4-10 Reativação · A4-11 Não comprador · A4-12 Cliente · A4-13 Pós-lançamento · A4-14 Higienização · A4-15 Consentimento/preferências.
Construir (Fase 6): Lead, Contact, Segment, Tags, Sequences, Nurture, Engagement, Non-buyer recovery.
n8n: `05_AUDIENCE_CRM/`. Eventos: `lead.created/engaged/segmented`. Gate: G05 CRM Ready.

## Autonomia
Classificação, tagging, segmentação: **AUTO** (listas gerais de HITL).

## Regra de ouro — pendências
Scopes · KPIs · prompt · ferramenta de CRM/e-mail (não definida) · schemas · regras de consentimento (A4-15; base legal e política não definidas nos docs): **PENDENTE**.
