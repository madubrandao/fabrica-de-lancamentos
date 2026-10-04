---
tipo: decisao
area: 02-Decisoes
status: parcialmente-resolvido
fonte: comparação Plano Final x Blueprint v1 x starter zip (2026-10-02); resoluções por [[DEC-0010-decisoes-canonicas-fase-1-v1]]
atualizado: 2026-10-04
---
# Pendências de decisão (conflitos entre documentos)

Regra do projeto: quando fontes conflitam, **parar e expor**, não reconciliar em silêncio. A hierarquia de autoridade agora é a do [[DEC-0010-decisoes-canonicas-fase-1-v1]] §1: decisões em `02-Decisoes` > Blueprint v1.x > arquitetura/roadmap derivados > starter > rascunhos e propostas conceituais.

Legenda: **RESOLVIDO** = decidido explicitamente no DEC-0010. **RESOLVIDO (por hierarquia)** = o DEC-0010 não cita o item, mas a ordem de autoridade do §1 dá a resposta; é aplicação minha, **a confirmar pela Madu**. **ABERTO** = sem resposta.

| ID | Conflito | Situação | Resolução |
|---|---|---|---|
| P01 | Gates: 10 (Plano) x 12 (Blueprint) | **RESOLVIDO** (DEC-0010 §3) | 12 gates do Blueprint. "Budget Approved" **não** é gate: é pré-condição/policy check `BUDGET_AUTHORIZED = true` antes de qualquer gasto, registrado em audit log. Ver [[gates]]. |
| P02 | Destinos pós-`DECISION` | **RESOLVIDO** (§5–§6) | SEED/INTERNAL/JOINT/RELAUNCH/EVERGREEN são **tipo/próxima ação**, não estados. `DECISION` gera uma decisão de continuidade que cria novo Launch (ou ativa sequência perene, ou `ARCHIVE`). `launch.status` ≠ `launch.type`. Ver [[launch-state-machine]]. |
| P03 | Três numerações de roadmap | **RESOLVIDO** (§4, §7) | Oficial: **M00–M06**. Starter M0–M8 é histórico. Ver [[roadmap]], [[milestones]]. |
| P04 | Nomes/catálogo de eventos (Plano x Blueprint) | **RESOLVIDO** ([[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]]) | Hierarquia confirmada pela Madu em 2026-10-04: Blueprint prevalece sobre o Plano Final. Padrão único `approval.requested/granted/rejected`; `approval.completed` removido. Ver [[catalogo-de-eventos]]. Novo PENDENTE: `approval.expired` (D8). |
| P05 | Entidades (`Approval`, `AuditLog`, `AudienceSegment`) | **RESOLVIDO** ([[DEC-0012-lista-de-entidades-do-blueprint]]) | Confirmado pela Madu em 2026-10-04: lista do Blueprint §14, 22 entidades; `audience` é nome histórico de `AudienceSegment`. |
| P06 | `99_ERRORS` x `99_ERROR_RECOVERY`; estrutura de repo Plano x Blueprint | **RESOLVIDO** ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]]) | Confirmado pela Madu em 2026-10-04: `99_ERRORS`; árvore do Blueprint §8 (a do Plano Final descartada); fonte única por assunto (D1–D5, D7). Novo PENDENTE: `prompts/` não tem equivalente no Blueprint. |
| P07 | Conjunto de notas pinadas no Maestri (Blueprint §3.1 = 5, §11 = 7; starter = 6 com `TASK_BOARD.md`) | **ABERTO** | Conflito dentro do próprio Blueprint; o DEC-0010 não decide. Decidir em M00. |
| P08 | Local do conhecimento do livro e dos ADRs (`knowledge/plf` x `docs/knowledge`; `docs/DECISIONS.md` x `docs/decisions`) | **RESOLVIDO** ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] D6) | Confirmado em 2026-10-04: `docs/adr/` é a fonte única das decisões (`DECISIONS.md` vira só índice do Maestri); conhecimento em `knowledge/plf/`. DEC-0010 §2 mantido: a pasta fica reservada, sem o PDF. |
| P09 | Guardrails do starter vazios/incompletos | **RESOLVIDO** (§8) | Guardrails oficiais de ENG-00…07 definidos no DEC-0010 §8; substituem os do starter. Ver [[engenharia-eng-00-a-07]]. Falta aplicar ao importar o starter (M00). Subitem `docker-compose.yml` duplicado: **RESOLVIDO** (DEC-0013 D5, um só em `infrastructure/compose/`). Subitem **ABERTO**: `n8n:latest` sem versão fixa (definir no M00). |
| P10 | **Novo.** M03 valida A0–A4 e M04 valida A5–A10, mas DEC-0008 (caminho crítico) ativa A5 e A9 depois de A6/A7/A8/A10. | ABERTO (baixo risco) | Compatível se M04 for lido como "validar A5–A10 ao final", com a ordem de ativação interna definida em M04. Confirmar. |
| P11 | **Novo.** Escopo do Core: Plano Final `CORE v0.1` inclui A0–A4; starter diz "A0 skeleton". Com M00–M06, A0–A4 caem em M03, e M01 só tem a plataforma. | ABERTO (baixo risco) | Sugestão: tratar `CORE v0.1` como nome histórico = M01+M02+M03; confirmar. |
| P12 | **Novo.** Mapeamento antigo Blueprint M01 → novo: o "Core Online" antigo (repo, Maestri, Docker, Postgres…) foi dividido entre **M00** (repo, Maestri, Claude Code, infra de dev) e **M01** (Postgres, Redis, migrations, eventos, audit, registry, permissions, approvals, state machine). Itens "API skeleton" e "event schema" não estão citados nominalmente no §7. | ABERTO (baixo risco) | Alocados em M01 por inferência; confirmar. |

Depois de decidido, registrar como `DEC-00XX` e atualizar esta tabela.
