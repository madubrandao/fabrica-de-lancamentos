---
tipo: decisao
area: 02-Decisoes
status: parcialmente-resolvido
fonte: comparação Plano Final x Blueprint v1 x starter zip (2026-10-02); resoluções por [[DEC-0010-decisoes-canonicas-fase-1-v1]]
atualizado: 2026-10-02
---
# Pendências de decisão (conflitos entre documentos)

Regra do projeto: quando fontes conflitam, **parar e expor**, não reconciliar em silêncio. A hierarquia de autoridade agora é a do [[DEC-0010-decisoes-canonicas-fase-1-v1]] §1: decisões em `02-Decisoes` > Blueprint v1.x > arquitetura/roadmap derivados > starter > rascunhos e propostas conceituais.

Legenda: **RESOLVIDO** = decidido explicitamente no DEC-0010. **RESOLVIDO (por hierarquia)** = o DEC-0010 não cita o item, mas a ordem de autoridade do §1 dá a resposta; é aplicação minha, **a confirmar pela Madu**. **ABERTO** = sem resposta.

| ID | Conflito | Situação | Resolução |
|---|---|---|---|
| P01 | Gates: 10 (Plano) x 12 (Blueprint) | **RESOLVIDO** (DEC-0010 §3) | 12 gates do Blueprint. "Budget Approved" **não** é gate: é pré-condição/policy check `BUDGET_AUTHORIZED = true` antes de qualquer gasto, registrado em audit log. Ver [[gates]]. |
| P02 | Destinos pós-`DECISION` | **RESOLVIDO** (§5–§6) | SEED/INTERNAL/JOINT/RELAUNCH/EVERGREEN são **tipo/próxima ação**, não estados. `DECISION` gera uma decisão de continuidade que cria novo Launch (ou ativa sequência perene, ou `ARCHIVE`). `launch.status` ≠ `launch.type`. Ver [[launch-state-machine]]. |
| P03 | Três numerações de roadmap | **RESOLVIDO** (§4, §7) | Oficial: **M00–M06**. Starter M0–M8 é histórico. Ver [[roadmap]], [[milestones]]. |
| P04 | Nomes/catálogo de eventos (Plano x Blueprint) | RESOLVIDO (por hierarquia) | Blueprint prevalece sobre o Plano Final. Ver [[catalogo-de-eventos]]. Sub-pendência **ABERTA**: padronizar `approval.completed` (§30) x `approval.granted/rejected` (§16). |
| P05 | Entidades (`Approval`, `AuditLog`, `AudienceSegment`) | RESOLVIDO (por hierarquia) | Lista do Blueprint. |
| P06 | `99_ERRORS` x `99_ERROR_RECOVERY`; estrutura de repo Plano x Blueprint | RESOLVIDO (por hierarquia) | `99_ERRORS` e a árvore de repositório do Blueprint §8. |
| P07 | Conjunto de notas pinadas no Maestri (Blueprint §3.1 = 5, §11 = 7; starter = 6 com `TASK_BOARD.md`) | **ABERTO** | Conflito dentro do próprio Blueprint; o DEC-0010 não decide. Decidir em M00. |
| P08 | Local do conhecimento do livro e dos ADRs (`knowledge/plf` x `docs/knowledge`; `docs/DECISIONS.md` x `docs/decisions`) | RESOLVIDO (por hierarquia) | Blueprint §8/§12. **Porém** DEC-0010 §2 proíbe incorporar o PDF a `knowledge/plf/` nesta fase: a pasta fica reservada, sem o PDF. |
| P09 | Guardrails do starter vazios/incompletos | **RESOLVIDO** (§8) | Guardrails oficiais de ENG-00…07 definidos no DEC-0010 §8; substituem os do starter. Ver [[engenharia-eng-00-a-07]]. Falta aplicar ao importar o starter (M00). Subitens **ABERTOS**: `n8n:latest` sem versão fixa; `docker-compose.yml` duplicado em `infra/`. |
| P10 | **Novo.** M03 valida A0–A4 e M04 valida A5–A10, mas DEC-0008 (caminho crítico) ativa A5 e A9 depois de A6/A7/A8/A10. | ABERTO (baixo risco) | Compatível se M04 for lido como "validar A5–A10 ao final", com a ordem de ativação interna definida em M04. Confirmar. |
| P11 | **Novo.** Escopo do Core: Plano Final `CORE v0.1` inclui A0–A4; starter diz "A0 skeleton". Com M00–M06, A0–A4 caem em M03, e M01 só tem a plataforma. | ABERTO (baixo risco) | Sugestão: tratar `CORE v0.1` como nome histórico = M01+M02+M03; confirmar. |
| P12 | **Novo.** Mapeamento antigo Blueprint M01 → novo: o "Core Online" antigo (repo, Maestri, Docker, Postgres…) foi dividido entre **M00** (repo, Maestri, Claude Code, infra de dev) e **M01** (Postgres, Redis, migrations, eventos, audit, registry, permissions, approvals, state machine). Itens "API skeleton" e "event schema" não estão citados nominalmente no §7. | ABERTO (baixo risco) | Alocados em M01 por inferência; confirmar. |

Depois de decidido, registrar como `DEC-00XX` e atualizar esta tabela.
