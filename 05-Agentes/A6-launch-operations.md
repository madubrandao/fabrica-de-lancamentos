---
tipo: agente
area: 05-Agentes
id: A6
camada: producao
fonte: PLANO FINAL §9; blueprint v1 §13, §23–§26
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A6 — Launch Operations

**Absorve:** operações + execução do lançamento. Plano Final Fase 8: conecta Research → Offer → Product → Content → CRM → Acquisition → Launch (primeiro ponto de execução ponta a ponta).

## Missão
Ser o gerente operacional digital do lançamento. A metodologia exige tarefas técnicas antes da abertura: páginas, links, pedido, confirmação, agradecimento e atendimento.

## Entradas / Saídas
- **Entra:** approved product, approved offer, approved communication, CRM configuration, campaign status.
- **Sai:** launch schedule, QA result, state transitions, operational alerts.

## Workflows (14)
A6-01 Criar launch · A6-02 Criar cronograma · A6-03 Criar checklist · A6-04 Verificar dependências · A6-05 QA técnico · A6-06 QA de conteúdo · A6-07 QA de checkout · A6-08 Pré-lançamento · A6-09 Abrir carrinho · A6-10 Monitorar abertura · A6-11 Fechar carrinho · A6-12 Pós-lançamento · A6-13 Incident Management · A6-14 Rollback.
n8n: `07_LAUNCH/`. Eventos: `launch.*` (cart_opened, cart_closing, cart_closed, postlaunch_started…). Gates: G06 Technical QA, G07 Launch Ready, G08 Cart Open, G09 Cart Close, G10 Post Launch Complete.

## Checklist antes de abrir o carrinho (Blueprint §24)
Sales Page · Links · Order Form · Checkout · Thank You Page · Confirmation · CRM · Tracking → Approval → Cart Open → `launch.cart_opened`.

## Autonomia
Go/no-go do lançamento e destrutivas em produção: aprovação humana (starter). Demais: **PENDENTE**.

## Regra de ouro — pendências
Scopes · KPIs · prompt · ferramentas (checkout, páginas) · schemas · política de rollback: **PENDENTE**.
