---
tipo: agente
area: 05-Agentes
id: A5
camada: producao
fonte: PLANO FINAL §8; blueprint v1 §13, §19
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A5 — Acquisition & Growth

**Absorve:** tráfego + growth. Plano Final Fase 7. **Ativação adiada** para depois do caminho crítico ([[DEC-0008-caminho-critico-primeiro]]).

## Missão
Fazer a máquina crescer.

## Entradas / Saídas
- **Entra:** target audience, campaign objective, approved creative, budget policy.
- **Sai:** campaigns, audience configurations, optimization actions.

## Workflows (12)
A5-01 Planejar aquisição · A5-02 Criar campanha · A5-03 Criar público · A5-04 Criar teste · A5-05 Monitorar CPL · A5-06 Monitorar CAC · A5-07 Otimizar campanha · A5-08 Pausar campanha · A5-09 Escalar campanha · A5-10 Testar criativos · A5-11 Testar audiência · A5-12 Distribuição externa.
Construir (Fase 7): Campaign, Audience, Creative, Budget, Optimization, Tracking.
n8n: `06_ACQUISITION/`. Eventos: `campaign.created/started/updated/paused`.

## Limites
**Não pode aumentar orçamento indefinidamente sozinho:** há limites por política (valor **não definido**). Exemplo de permissão no Blueprint §19: lê audience/campaigns/metrics/offer; escreve campaigns/experiments; ações pause_campaign e create_test; aumento acima do limite exige aprovação.

Antes de qualquer execução com gasto: policy check `BUDGET_AUTHORIZED = true` ([[gates]]).

## Autonomia
Campanha: **SUPERVISED**; grandes alterações de orçamento e gastos acima do limite: aprovação humana.

## Regra de ouro — pendências
Limite de orçamento · plataformas de anúncio · KPIs (CPL, CAC citados nos workflows; metas não definidas) · prompt · schemas: **PENDENTE**.
