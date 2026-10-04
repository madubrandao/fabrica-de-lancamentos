---
tipo: fase
area: 04-Fases
fonte: DEC-0010 §7 (critérios canônicos); blueprint v1 §35–§39 (itens de checklist, realocados); starter docs/ROADMAP.md
status: nao-iniciado
atualizado: 2026-10-04
---
# Milestones M00–M06 (checklists)

Critério de cada marco: [[DEC-0010-decisoes-canonicas-fase-1-v1]] §7. Os itens vêm do Blueprint e do starter e foram **alocados por mim ao marco correspondente**; a alocação é inferência ([[pendencias-de-decisao]] P12). Nada foi iniciado: nenhum código ou repositório existe. Cada capability só está pronta quando cumprir [[definicao-de-pronto]].

Transversal: o check `BUDGET_AUTHORIZED = true` pode ser exigido em qualquer marco para ações com gasto ([[gates]]).

## M00 — Foundation
Valida: fundação documental, repositório, Maestri, Claude Code, infraestrutura de desenvolvimento.
- [ ] Início formal do M00 autorizado pela Madu (DEC-0010 §10)
- [ ] Repositório Git `launch-os` · [ ] `CLAUDE.md` raiz (+ por domínio)
- [ ] Maestri: workspace, terminais ENG-00…07, responsabilidades, 6 notas pinadas ([[DEC-0014-notas-pinadas-no-maestri]]); definir onde ficam (repo ou Maestri)
- [ ] Claude Code instalado e verificado; decisão WSL/tmux/local
- [ ] Guardrails oficiais ENG-00…07 aplicados às definições de agente ([[engenharia-eng-00-a-07]])
- [ ] Docker de desenvolvimento (Postgres, Redis, n8n com versão fixa) · `.env.example` · bootstrap
- [ ] Importação do starter conforme [[DEC-0013-estrutura-do-repositorio-e-fonte-unica]]: compose único em `infrastructure/compose/`, referências `docs/decisions`/`docs/knowledge` corrigidas, `.claude/rules` sem repetir guardrails ([[starter-zip-inventario]])
- [ ] Esqueleto de CI
- [ ] Documentação de arquitetura transferida/linkada ao repo

## M01 — Core Platform
Valida: PostgreSQL, Redis, migrations, eventos, audit log, Agent Registry, permissions, approvals e state machine.
- [ ] PostgreSQL + schema inicial + migrations reversíveis
- [ ] Redis
- [ ] Event model e event schema ([[catalogo-de-eventos]])
- [ ] Audit log
- [ ] Agent Registry · [ ] Permission model
- [ ] Approval gates G01–G12 + policy check `BUDGET_AUTHORIZED`
- [ ] Launch state machine (status x type; decisão de continuidade)
- [ ] API skeleton

## M02 — Agent Engine
Valida: Agent Contract, context assembly, tool registry, memory interfaces, execution lifecycle, policy enforcement.
- [ ] Agent definition schema · [ ] Prompt loading
- [ ] Tool registry (escopo explícito por ferramenta)
- [ ] Memory read/write (com origem e timestamp)
- [ ] Execution state com `execution_id` e audit trail · retries/timeouts
- [ ] Agent routing · approval hooks · policy enforcement

## M03 — First Business Agents
Valida: A0–A4. Objetivo: produzir um lançamento até `LAUNCH_READY`.
- [ ] A0 · [ ] A1 · [ ] A2 · [ ] A3 · [ ] A4
- Primeiro workflow integrado: Create Launch ([[n8n-estrutura]])

## M04 — Launch Execution
Valida: A5–A10 e execução do ciclo completo.
- [ ] A5 · [ ] A6 · [ ] A7 · [ ] A8 · [ ] A9 · [ ] A10
- Ordem de ativação interna: DEC-0008 (P10)

## M05 — Learning Loop
Valida: análise, experimentação e learning loop.
`DATA → ANALYSIS → EXPERIMENT → RESULT → LEARNING → MEMORY → A0`
- [ ] Post-mortem · [ ] experimentos · [ ] learnings · [ ] recomendações · [ ] cada lançamento alimenta o seguinte

## M06 — Operator Dashboard
Valida: painel operacional e aprovação humana.
- [ ] Launches, approvals, agents, contacts, products/offers, metrics, audit trail ([[observabilidade-e-testes]])
- Regra: não construir o dashboard completo antes de estabilizar os contratos do Core.

Voltar: [[roadmap]]
