---
tipo: fase
area: 04-Fases
fonte: DEC-0010 §4, §7 (canônico); blueprint v1 §35–§39; PLANO FINAL §24–§25; starter docs/ROADMAP.md (histórico)
status: canonico (M00–M06)
atualizado: 2026-10-04
---
# Roadmap canônico — M00 a M06

Numeração oficial ([[DEC-0010-decisoes-canonicas-fase-1-v1]] §4). Qualquer roadmap futuro **deve** usar M00–M06. O M0–M8 do starter é decomposição histórica, não fonte canônica.

| Marco | Nome | O que valida (DEC-0010 §7) |
|---|---|---|
| **M00** | Foundation | fundação documental, repositório, Maestri, Claude Code e infraestrutura de desenvolvimento |
| **M01** | Core Platform | PostgreSQL, Redis, migrations, eventos, audit log, Agent Registry, permissions, approvals e state machine |
| **M02** | Agent Engine | Agent Contract, context assembly, tool registry, memory interfaces, execution lifecycle e policy enforcement |
| **M03** | First Business Agents | A0–A4 |
| **M04** | Launch Execution | A5–A10 e execução do ciclo completo |
| **M05** | Learning Loop | análise, experimentação e learning loop |
| **M06** | Operator Dashboard | painel operacional e aprovação humana |

**M00 Foundation: iniciado formalmente em 2026-10-04** ([[DEC-0018-inicio-formal-do-m00]]; plano em [[M00-plano-de-execucao]]). O escopo é só o do M00; o M01 não está autorizado. A FASE 1 - BASE (documentação e arquitetura) está concluída.

## Correspondência com as numerações antigas (só referência histórica)
| Conteúdo | Plano Final (Fases) | Blueprint (antigo) | Starter | **Canônico** |
|---|---|---|---|---|
| Repo, CLAUDE.md, Maestri, Claude Code, Docker de dev, docs | Fase 0 (parte) | M01 Core Online (parte) | M0 | **M00** |
| Postgres, Redis, migrations, eventos, audit, registry, permissions, approvals, state machine | Fases 0–2 | M01 / M02 (parte) | M1 | **M01** |
| Agent contract, contexto, tools, memória, execução, policies | — | M02 | M2 | **M02** |
| A0, A1, A2, A3, A4 | Fases 2–6 | M03 | M3–M4 | **M03** |
| A5, A6, A7, A8, A9, A10 | Fases 7–12 | M04 | M5–M7 | **M04** |
| Learning loop (análise, experimentos, aprendizado) | Fase 12 | M05 | M7 | **M05** |
| Dashboard | — | passo 13 do §40 | M8 | **M06** |

API skeleton e event schema ficam no M01 ([[DEC-0015-alocacao-do-roadmap-m01-m04]], P12 resolvida).

## Ativação (DEC-0008)
Caminho crítico: A0→A1→A2→A3→A4→A6→A7→A8→A10→A0; A5 e A9 depois. O M04 valida A5–A10 ao final e a ordem de ativação acima vale dentro dele. **A6 mínimo entra no M03** para o Create Launch chegar ao `LAUNCH_READY` ([[DEC-0015-alocacao-do-roadmap-m01-m04]]).

## Teste de aceite ponta a ponta (starter, mantido)
Criar produto, gerar oferta, rodar pesquisa, gerar plano de lançamento, criar assets de PLC, passar pelos gates, executar lançamento **simulado**, registrar pedidos/eventos, fechar carrinho, rodar análise pós-lançamento e persistir aprendizados.

Ver [[milestones]] · [[blueprint-de-execucao]] · [[CORE-v0.1]] · [[criterio-de-sucesso-v1]].
