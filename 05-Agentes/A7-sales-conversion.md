---
tipo: agente
area: 05-Agentes
id: A7
camada: producao
fonte: PLANO FINAL §10; blueprint v1 §13, §25
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A7 — Sales & Conversion

**Absorve:** vendas + conversão + recuperação comercial. Plano Final Fase 9.

## Missão
Transformar comportamento em conversão. Na abertura, o livro dá atenção explícita a tráfego, cliques, formulário de pedidos e vendas nas primeiras horas.

## Entradas / Saídas
- **Entra:** traffic, page events, checkout events, orders, objections.
- **Sai:** diagnosis, recovery actions, test recommendations.

## Workflows (10)
A7-01 Monitorar conversão · A7-02 Analisar página · A7-03 Analisar checkout · A7-04 Identificar objeções comerciais · A7-05 Gerar recomendações · A7-06 Recuperar abandono · A7-07 Recuperar não compradores · A7-08 Comunicação de fechamento · A7-09 Analisar oferta · A7-10 Recomendar testes.
Construir (Fase 9): Conversion monitor, Checkout tracking, Abandonment, Objection recovery, Closing.
n8n: `08_SALES/`. Eventos: `order.created/paid/refunded`.

## Autonomia
Monitoramento e análise: AUTO. Alterações de oferta/preço: via A2/humano.

## Regra de ouro — pendências
Scopes · KPIs · prompt · ferramentas · schemas: **PENDENTE**.
