---
tipo: fase
area: 04-Fases
fonte: DEC-0010 §7 (critérios canônicos); blueprint v1 §35–§39 (itens de checklist, realocados); starter docs/ROADMAP.md
status: nao-iniciado
atualizado: 2026-10-04
---
# Milestones M00–M06 (checklists)

Critério de cada marco: [[DEC-0010-decisoes-canonicas-fase-1-v1]] §7. Os itens vêm do Blueprint e do starter e foram **alocados por mim ao marco correspondente**; a alocação foi confirmada pela Madu em [[DEC-0015-alocacao-do-roadmap-m01-m04]] (P12). O M00 está concluído ([[DEC-0021-m00-foundation-concluido]]); o M01 está em andamento ([[DEC-0022-inicio-formal-do-m01]]) na fase F1 (especificação), e nenhum código de aplicação existe ainda. Cada capability só está pronta quando cumprir [[definicao-de-pronto]].

Transversal: o check `BUDGET_AUTHORIZED = true` pode ser exigido em qualquer marco para ações com gasto ([[gates]]).

## M00 — Foundation
**Status: concluído em 2026-10-04** ([[DEC-0021-m00-foundation-concluido]]; início formal em 2026-10-04, [[DEC-0018-inicio-formal-do-m00]]). Passo a passo em [[M00-plano-de-execucao]].
Valida: fundação documental, repositório, Claude Code, infraestrutura de desenvolvimento (sem Maestri, [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]).
- [x] Início formal do M00 autorizado pela Madu (DEC-0010 §10; [[DEC-0018-inicio-formal-do-m00]])
- [x] Repositório Git `launch-os` (privado, GitHub, fora de `.claude`, sem espaços no caminho; D6) · [x] `CLAUDE.md` raiz (+ por domínio), com TECH STACK = TypeScript/Node (D1)
- [x] ~~Maestri~~ removido do escopo ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]); 6 notas de estado em `docs/status/` no `launch-os`
- [x] Claude Code instalado e verificado (2.1.289); ambiente = Windows nativo + Docker Desktop ([[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] D5); `.gitattributes` fixando LF
- [x] Guardrails oficiais ENG-00…07 aplicados às definições de agente ([[engenharia-eng-00-a-07]])
- [x] Docker de desenvolvimento (Postgres, Redis, n8n com versão fixa) · `.env.example` · bootstrap ([[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]], validado em 2026-10-04)
- [x] Importação do starter conforme [[DEC-0013-estrutura-do-repositorio-e-fonte-unica]]: compose único em `infrastructure/compose/`, referências `docs/decisions`/`docs/knowledge` corrigidas, `.claude/rules` sem repetir guardrails ([[starter-zip-inventario]])
- [x] Esqueleto de CI (GitHub Actions, execução verde em `aeb58d4`)
- [x] Documentação de arquitetura transferida/linkada ao repo (referenciada, não copiada: `docs/architecture/README.md`)

## M01 — Core Platform
**Status: concluído em 2026-10-08** ([[DEC-0029-fechamento-do-m01-core-platform]]). Início formal ( em 2026-10-04, [[DEC-0022-inicio-formal-do-m01]]; plano em [[M01-plano-de-execucao]]). F0 e **F1 concluídas** (especificação aprovada pela Madu em 2026-10-07, portão F1: [[DEC-0023-aprovacao-do-portao-f1-especificacao-do-core]]). **F2 (banco), F6 (infra e CI) e F3 (eventos) fechadas em 2026-10-07** ([[DEC-0024-fechamento-da-f2-e-da-f6-do-m01]], [[DEC-0025-fechamento-da-f3-do-m01]]), mescladas na `main` com CI verde. **F4 (API fina, state machine e aprovações) fechada em 2026-10-07** ([[DEC-0026-fechamento-da-f4-do-m01]]), PR #4 mesclado. **F5 (DEC-0027) e F7 (DEC-0028) fechadas; F8 (fechamento) implementada em 2026-10-07 (PR #7), aguardando o DEC-0029 da Madu ([[DEC-0029-fechamento-do-m01-core-platform]]).** Critérios marcados com a aprovação do DEC-0029. **O M02 não está autorizado:** só com novo início formal da Madu; A5 (isolamento de credenciais no n8n) é a primeira tarefa dele. Código, schema e migrations só dentro da especificação e em branches `feat/*`.
Valida: PostgreSQL, Redis, migrations, eventos, audit log, Agent Registry, permissions, approvals e state machine.
- [x] PostgreSQL + schema inicial + migrations reversíveis
- [x] Redis (parcial, aceito: sobe saudável com senha; a API não o usa no M01)
- [x] Event model e event schema ([[catalogo-de-eventos]])
- [x] Audit log
- [x] Agent Registry · [x] Permission model
- [x] Approval gates G01–G12 + policy check `BUDGET_AUTHORIZED`
- [x] Launch state machine (status x type; decisão de continuidade)
- [x] API skeleton

## M02 — Agent Engine
**Status: implementado e verificado do zero em 2026-10-08; aguarda o aceite do [[DEC-0032-fechamento-do-m02-agent-engine]] pela Madu** (início formal em 2026-10-08, [[DEC-0030-inicio-formal-do-m02]]; plano em [[M02-plano-de-execucao]]; portão da F1: [[DEC-0031-portao-f1-especificacao-do-agent-engine]]). F0 a F7 concluídas; 1266 testes, fumaça 29/29 contra o n8n real, duas revisões completas sem achado crítico ou alto. **O M03 não está autorizado.** Lacunas abertas para antes do M03: L1 a L7 no DEC-0032.
Valida: Agent Contract, context assembly, tool registry, memory interfaces, execution lifecycle, policy enforcement.
- [x] Agent definition schema · [x] Prompt loading
- [x] Tool registry (escopo explícito por ferramenta)
- [x] Memory read/write (com origem e timestamp)
- [x] Execution state com `execution_id` e audit trail · retries/timeouts
- [x] Agent routing · approval hooks · policy enforcement
- [x] A5 resolvido (negativo, mitigado pelo token curto) · [x] n8n emite evento pela API · [x] `error.detected` de origem `api` e `workflow`

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
