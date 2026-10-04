---
tipo: decisao
area: 02-Decisoes
id: DEC-0014
status: substituida (DEC-0020)
fonte: decisão da Madu em 2026-10-04; blueprint v1 §3.1, §11, §40 (passo 5), §42; starter docs/MAESTRI_SETUP.md
data-registro: 2026-10-04
resolve: P07
---
# DEC-0014 — Notas pinadas no Maestri (`00-COMMAND-CENTER`)

> **Substituída pelo [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]] (2026-10-04):** sem Maestri, as 6 notas viram arquivos em `docs/status/` do repositório `launch-os`. O texto abaixo é o registro histórico.

## Contexto
As fontes divergiam sobre o conjunto de notas: Blueprint §3.1 = 5, Blueprint §11 = 7, starter = 6 (com `TASK_BOARD.md`). Ao todo eram 10 nomes diferentes. O passo 5 do Blueprint (§40) manda criar `CURRENT_TASK`, e o formato de tarefa do §42 pressupõe uma nota de tarefa corrente. `ARCHITECTURE` já existe como arquivo do repositório (`docs/architecture/`), e `DECISIONS` já é só índice para `docs/adr/` ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] D6).

## Decisão
Seis notas pinadas no M00, sem duplicidade:

| Nota | Função |
|---|---|
| `PROJECT_STATUS.md` | estado geral do projeto |
| `CURRENT_MILESTONE.md` | marco atual (M00–M06, [[roadmap]]) |
| `CURRENT_TASK.md` | tarefa corrente, no formato do Blueprint §11/§42 |
| `DECISIONS.md` | **só índice** que aponta para `docs/adr/` |
| `BLOCKERS.md` | bloqueios |
| `DEFINITION_OF_DONE.md` | critérios de pronto ([[definicao-de-pronto]]) |

**Fora do conjunto inicial:**
- `TASK_BOARD.md`: sobrepõe `CURRENT_TASK` e só existe no starter (nível 4 da hierarquia).
- `ARCHITECTURE.md`: a fonte é `docs/architecture/`; uma nota pinada duplicaria.
- `AGENT_STATUS.md` e `RELEASE_NOTES.md`: ainda não há agentes rodando nem releases; entram depois, por ADR.

## Alternativas rejeitadas
- A (5 do §3.1): deixa `CURRENT_TASK` sem nota fixa, apesar do passo 5.
- B (6 do starter): traz `TASK_BOARD` e não traz `CURRENT_TASK`.
- C (7 do §11): sem `CURRENT_MILESTONE` e `DEFINITION_OF_DONE`; `ARCHITECTURE` duplica o repo; `AGENT_STATUS` e `RELEASE_NOTES` prematuras.
- E (as 10): contra o critério de estrutura limpa.

## Consequências
- **PENDENTE (M00):** onde as notas ficam, dentro do repositório ou na pasta do Maestri. O Blueprint diz que são arquivos Markdown reais, mas não diz onde. Isso decide se `DEFINITION_OF_DONE` e `DECISIONS` podem ser o mesmo arquivo de `docs/` ou se viram cópias. Verificar com o Maestri instalado; não assumir.
- Adicionar `AGENT_STATUS`, `RELEASE_NOTES` ou outra nota exige novo ADR.

Ver [[pendencias-de-decisao]] (P07) · [[blueprint-de-execucao]] · [[milestones]].
