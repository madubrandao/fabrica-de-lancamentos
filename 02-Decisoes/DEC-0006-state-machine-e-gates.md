---
tipo: decisao
area: 02-Decisoes
id: DEC-0006
status: aceita; conflito resolvido por DEC-0010
fonte: PLANO FINAL §14–§15, §20; blueprint v1 §15, §21
data-registro: 2026-10-02
---
# DEC-0006 — Todo lançamento é uma state machine com gates

**Decisão:** todo lançamento tem estados formais e **não pula de estado sem cumprir o gate**. Estados e gates em [[launch-state-machine]] e [[gates]].
**Conflito (resolvido):** o Plano Final tinha 10 gates e o Blueprint 12. [[DEC-0010-decisoes-canonicas-fase-1-v1]] fixou **12 gates**; orçamento é policy check `BUDGET_AUTHORIZED = true`, não gate. Destinos pós-`DECISION`: tipo/próxima ação, não estados. Ver [[gates]] e [[launch-state-machine]].
**Tipos de lançamento:** SEED, INTERNAL, JOINT, RELAUNCH, EVERGREEN (campo `launch.type`, separado de `launch.status`).
**Regra do livro vira gate:** testar a oferta com lançamento interno antes de colocá-la diante das listas de parceiros (Blueprint §13/A9).
