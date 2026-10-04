---
tipo: referencia
area: 08-Referencias
fonte: FASE  1 - BASE/launch-os-starter-claude-maestri-v1.zip (lido via extração no scratchpad, nada extraído para o projeto)
status: inventario
atualizado: 2026-10-04
---
# Inventário do `launch-os-starter-claude-maestri-v1.zip`

Starter do repositório `launch-os`. **Não foi extraído para o projeto**; ao decidir criar o repo ([[blueprint-de-execucao]] passo 1), ele é a base.

| Arquivo | Conteúdo | Observação |
|---|---|---|
| `CLAUDE.md` | missão, arquitetura, agentes, objetos, state machine, 10 regras de engenharia, aprovação humana, protocolo de entrega, git, hierarquia de verdade, primeiro milestone | usa `docs/decisions` e `docs/knowledge` (P08) |
| `README.md` | visão dev/produção e passos de início | |
| `.claude/rules/` | `architecture.md`, `n8n.md`, `security.md`, `testing.md` | incorporadas em [[permissoes-e-hitl]], [[n8n-estrutura]], [[observabilidade-e-testes]] |
| `.claude/agents/` | 8 definições ENG-xx (architect, backend, database, n8n, agentengine, frontend, qa, devops) | seção `Guardrails` **vazia**; regras coladas em uma linha. **Substituídos** pelos guardrails oficiais do DEC-0010 §8 (P09 resolvido); aplicar ao importar no M00 |
| `docs/DECISIONS.md` | ADR-0001…0004 (aceitos) | refletidos em DEC-0001…0004 |
| `docs/DEFINITION_OF_DONE.md` | 10 critérios | em [[definicao-de-pronto]] |
| `docs/ROADMAP.md` | Milestones 0–8 + teste de aceite | em [[roadmap]] |
| `docs/MAESTRI_SETUP.md` | workspaces, canvas, conexões, notas, regra de operação, recomendação Windows | em [[blueprint-de-execucao]] |
| `docs/FIRST_SESSION.md` | primeira sessão no Maestri (em português) | em [[blueprint-de-execucao]] |
| `docker-compose.yml` e `infra/docker-compose.dev.yml` | postgres:17, redis:8 (appendonly), n8n:latest com Postgres | duplicados; `latest` sem pin |
| `.env.example` | variáveis de app, DB, Redis, n8n, ANTHROPIC/OPENAI (vazias), JWT | valores `CHANGE_ME` |
| `.mcp.json.example` | placeholder de MCP | |
| `.gitignore` | ignora `.env*` (exceto example), `CLAUDE.local.md`, node_modules, dist, secrets/ etc. | |
| `scripts/bootstrap.ps1` | exige git, docker e claude; cria `.env` a partir do example; mostra versões | |
| `FILE_INDEX.txt` | lista dos arquivos | |
| pastas vazias | `apps/ packages/ database/ n8n/ tests/ scripts/` | |

## Status canônico do starter
Pelo [[DEC-0010-decisoes-canonicas-fase-1-v1]] §1, o starter está no nível 4: **implementa** decisões, não redefine arquitetura. Seu `docs/ROADMAP.md` (M0–M8) é histórico; vale M00–M06 ([[roadmap]]).

## Ao importar no M00 ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]])
O starter **não foi modificado**. Na importação:
1. Remover o `docker-compose.yml` da raiz (idêntico ao de `infra/`, confirmado por `diff`); manter um só em `infrastructure/compose/docker-compose.dev.yml`; `infra/` vira `infrastructure/`. Fixar a versão do `n8n` (PENDENTE).
2. `CLAUDE.md`: trocar `docs/decisions` por `docs/adr` e `docs/knowledge` por `knowledge/plf`; `docs/DECISIONS.md` deixa de ser fonte e vira índice do Maestri para `docs/adr/` (README, FIRST_SESSION e `testing.md` também o citam).
3. `.claude/agents/*.md`: preencher `Guardrails` com o DEC-0010 §8 e deixar `Responsibilities` só com responsabilidade.
4. `.claude/rules/*.md`: remover o que repete guardrail de agente.
5. Atualizar o README do starter (aponta para `infra/docker-compose.dev.yml`) e o `FILE_INDEX.txt`.

## Diferenças em relação ao Blueprint
Starter não traz `knowledge/`, `docs/adr`, `infrastructure/` (usa `infra/`), `workflows` nem `schemas`. Estrutura-alvo completa está em [[estrutura-do-repositorio]].
