---
tipo: agente
area: 05-Agentes
id: A10
camada: producao
fonte: PLANO FINAL §13; blueprint v1 §13, §25–§26, §39
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A10 — Analytics & Learning

**Absorve:** dados + análise + otimização + aprendizado. Plano Final Fase 12. Realimenta A0.

## Missão
Ser o cérebro de aprendizado da empresa: `dados → diagnóstico → insight → experimento → resultado → aprendizado → memória`. O lançamento é um processo acumulativo: lançamentos anteriores geram experiência, dados, lista maior e ideias.

## Entradas / Saídas
- **Entra:** all events, metrics, experiments, outcomes.
- **Sai:** dashboards, diagnoses, anomalies, experiments, learnings, recommendations.

## Workflows (13)
A10-01 Coletar eventos · A10-02 Consolidar métricas · A10-03 Dashboard · A10-04 Attribution · A10-05 Diagnóstico · A10-06 Detectar anomalias · A10-07 Comparar lançamentos · A10-08 Analisar testes · A10-09 Gerar recomendações · A10-10 Post-mortem · A10-11 Registrar aprendizado · A10-12 Atualizar playbooks · A10-13 Alimentar A0.
Construir (Fase 12): Metrics, Attribution, Experiments, Post-mortem, Learning, Recommendations, Strategy updates.
n8n: `11_ANALYTICS/`. Eventos: `metric.recorded`, `experiment.created/completed`, `learning.created`. Gates: G11 Analysis Complete, G12 Learning Recorded.

## Autonomia
Análise, relatórios, monitoramento: **AUTO**. IA não altera sozinha toda a estratégia ([[escopo-e-principios]]).

## Regra de ouro — pendências
Scopes · KPIs · prompt · ferramentas de analytics/atribuição · schemas · formato de playbook: **PENDENTE**.
