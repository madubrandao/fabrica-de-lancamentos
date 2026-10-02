---
tipo: agente
area: 05-Agentes
id: A1
camada: producao
fonte: PLANO FINAL §4; blueprint v1 §13
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A1 — Market Intelligence

**Absorve:** pesquisa de mercado. **Alimenta:** A2 (e A3). Plano Final Fase 3.

## Missão
Concentrar a inteligência sobre mercado. O pré-pré-lançamento tem papel central: testar receptividade, descobrir objeções, coletar linguagem e usar isso para refinar a oferta.

## Entradas / Saídas
- **Entra:** pesquisas, comentários, e-mails, mensagens, dados de CRM, dados de lançamento, feedback, dados de clientes (Blueprint: survey, comments, CRM data, support data, launch data).
- **Sai:** Market Intelligence Report · Avatar Profile · Objection Map · VOC Dataset · Opportunity Map.

## Workflows (12)
A1-01 Mapear mercado · A1-02 Mapear avatar · A1-03 Mapear dores · A1-04 Mapear desejos · A1-05 Mapear objeções · A1-06 Mapear linguagem · A1-07 Analisar concorrência · A1-08 Executar pré-pré-lançamento · A1-09 Coletar respostas · A1-10 Classificar feedback · A1-11 Atualizar mapa de objeções · A1-12 Gerar oportunidades de produto.
n8n: `02_INTELLIGENCE/`. Eventos: `launch.research_started`, `launch.validation_completed`, `launch.prepre_started/completed`. Gate: G01 Market Validated.

## Autonomia
Classificação, resumos e análise são AUTO segundo as listas gerais de HITL (derivado; confirmar). Escopos reais: **PENDENTE**.

## Regra de ouro — pendências
Scopes de leitura/escrita · KPIs · prompt · ferramentas (fontes de pesquisa) · schemas: **PENDENTE**.
