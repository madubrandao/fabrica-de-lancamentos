---
tipo: fase
area: 04-Fases
fonte: DEC-0010 §7 (critérios canônicos); blueprint v1 §35–§39 (itens de checklist, realocados); starter docs/ROADMAP.md
status: nao-iniciado
atualizado: 2026-10-04
---
# Milestones M00–M06 (checklists)

Critério de cada marco: [[DEC-0010-decisoes-canonicas-fase-1-v1]] §7. Os itens vêm do Blueprint e do starter e foram **alocados por mim ao marco correspondente**; a alocação foi confirmada pela Madu em [[DEC-0015-alocacao-do-roadmap-m01-m04]] (P12). O M00 está em andamento; do M01 em diante nada foi iniciado e nenhum código existe. Cada capability só está pronta quando cumprir [[definicao-de-pronto]].

Transversal: o check `BUDGET_AUTHORIZED = true` pode ser exigido em qualquer marco para ações com gasto ([[gates]]).

## M00 — Foundation
**Status: em andamento** (início formal em 2026-10-04, [[DEC-0018-inicio-formal-do-m00]]). Passo a passo em [[M00-plano-de-execucao]].
Valida: fundação documental, repositório, Maestri, Claude Code, infraestrutura de desenvolvimento.
- [x] Início formal do M00 autorizado pela Madu (DEC-0010 §10; [[DEC-0018-inicio-formal-do-m00]])
- [ ] Repositório Git `launch-os` (privado, GitHub, fora de `.claude`, sem espaços no caminho; D6) · [ ] `CLAUDE.md` raiz (+ por domínio), com TECH STACK = TypeScript/Node (D1)
- [ ] Maestri: workspace, terminais ENG-00…07, responsabilidades, 6 notas pinadas ([[DEC-0014-notas-pinadas-no-maestri]]); definir onde ficam (repo ou Maestri)
- [ ] Claude Code instalado e verificado; ambiente = Windows nativo + Docker Desktop ([[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] D5); `.gitattributes` fixando LF
- [ ] Guardrails oficiais ENG-00…07 aplicados às definições de agente ([[engenharia-eng-00-a-07]])
- [x] Docker de desenvolvimento (Postgres, Redis, n8n com versão fixa) · `.env.example` · bootstrap ([[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]], validado em 2026-10-04)
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
- Pré-requisito: produto fictício de teste definido ([[DEC-0017-modelo-de-negocio-inicial]] C1)
- [ ] A0 · [ ] A1 · [ ] A2 · [ ] A3 · [ ] A4 · [ ] **A6 mínimo** (só o necessário para o Create Launch; definição PENDENTE; A6 completo no M04)
- Primeiro workflow integrado: Create Launch até `LAUNCH_READY` ([[n8n-estrutura]], [[DEC-0015-alocacao-do-roadmap-m01-m04]])

## M04 — Launch Execution
Valida: A5–A10 e execução do ciclo completo.
- [ ] A5 · [ ] A6 · [ ] A7 · [ ] A8 · [ ] A9 · [ ] A10
- Pré-requisito: política de orçamento definida antes de qualquer gasto do A5 ([[DEC-0017-modelo-de-negocio-inicial]] C4); integrações simuladas até o primeiro lançamento real (C5)
- Ordem de ativação interna: A6 (completo), A7, A8, A10, depois A5 e A9 ([[DEC-0008-caminho-critico-primeiro]], [[DEC-0015-alocacao-do-roadmap-m01-m04]]); o marco só fecha com A5–A10 validados

## M05 — Learning Loop
Valida: análise, experimentação e learning loop.
`DATA → ANALYSIS → EXPERIMENT → RESULT → LEARNING → MEMORY → A0`
- [ ] Post-mortem · [ ] experimentos · [ ] learnings · [ ] recomendações · [ ] cada lançamento alimenta o seguinte

## M06 — Operator Dashboard
Valida: painel operacional e aprovação humana.
- [ ] Launches, approvals, agents, contacts, products/offers, metrics, audit trail ([[observabilidade-e-testes]])
- Regra: não construir o dashboard completo antes de estabilizar os contratos do Core.

Voltar: [[roadmap]]
