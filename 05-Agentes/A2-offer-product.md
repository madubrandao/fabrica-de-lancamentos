---
tipo: agente
area: 05-Agentes
id: A2
camada: producao
fonte: PLANO FINAL §5; blueprint v1 §13
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A2 — Offer & Product

**Absorve:** oferta + produto + qualidade. Plano Final Fase 4. Gera o objeto que A3 e A6 usam.

## Missão
Oferta e produto conversam diretamente (fusão deliberada).

## Entradas / Saídas
- **Entra:** market intelligence, opportunity, customer feedback.
- **Sai:** Product Spec · Offer Spec · Transformation Map · Price Structure · Bonus Structure · Guarantee Structure · Product Version.

## Workflows (14)
A2-01 Criar conceito de produto · A2-02 Validar problema · A2-03 Definir transformação · A2-04 Criar mecanismo · A2-05 Estruturar oferta · A2-06 Definir bônus · A2-07 Estruturar garantia · A2-08 Definir preço · A2-09 Criar produto · A2-10 Criar módulos · A2-11 Criar materiais · A2-12 Fazer QA · A2-13 Atualizar produto · A2-14 Criar nova versão.
n8n: `03_OFFER_PRODUCT/`. Eventos: `product.*`, `offer.*`. Gates: G02 Offer Approved, G03 Product Ready.

## Autonomia (derivado das listas gerais de HITL; confirmar)
- Oferta: **SUPERVISED**.
- Preço estratégico, garantia, mudança crítica de produto, novo produto estratégico: **HUMAN DECISION**.

## Regra de ouro — pendências
Scopes · KPIs · prompt · ferramentas · schemas: **PENDENTE**.
