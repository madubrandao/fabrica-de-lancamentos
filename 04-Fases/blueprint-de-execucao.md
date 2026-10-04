---
tipo: fase
area: 04-Fases
fonte: blueprint v1 §3–§6, §10–§11, §33–§34, §40–§42; starter docs/FIRST_SESSION.md, docs/MAESTRI_SETUP.md
status: nao-iniciado
atualizado: 2026-10-02
---
# Blueprint de execução

Sequência operacional para sair do papel. **Nada abaixo foi executado.** Pré-requisito já em andamento: estrutura de memória (este vault).

## Os 15 passos (Blueprint §40)
| # | Passo | Responsável | Status |
|---|---|---|---|
| 1 | Criar o repositório `launch-os` | Madu / ENG-07 | [ ] |
| 2 | Abrir o repo em um Workspace do Maestri | Madu | [ ] |
| 3 | Criar terminais ENG-00…07 | Madu | [ ] |
| 4 | Criar `CLAUDE.md` raiz | ENG-00 | [ ] |
| 5 | Criar `DECISIONS.md` (só índice do Maestri para `docs/adr/`, [[DEC-0013-estrutura-do-repositorio-e-fonte-unica]]), `PROJECT_STATUS.md`, `CURRENT_TASK.md`, `DEFINITION_OF_DONE.md` | ENG-00 | [ ] |
| 6 | Architect produz `ARCHITECTURE.md`, `DOMAIN_MODEL.md`, `EVENT_CATALOG.md`, `AGENT_MODEL.md` | ENG-00 | [ ] |
| 7 | Database cria schema e migrations | ENG-02 | [ ] |
| 8 | Backend cria API e state machine | ENG-01 | [ ] |
| 9 | DevOps cria Docker local e ambientes | ENG-07 | [ ] |
| 10 | Agent Engine implementa registry, permissions e execution contract | ENG-04 | [ ] |
| 11 | N8N implementa o primeiro workflow de orquestração (Create Launch) | ENG-03 | [ ] |
| 12 | QA cria o primeiro conjunto de testes | ENG-06 | [ ] |
| 13 | Frontend cria painel mínimo de observação e aprovação | ENG-05 | [ ] |
| 14 | Construir A0 → A1 → A2 → A3 → A4 | ENG-04 + ENG-03 | [ ] |
| 15 | Executar o primeiro INTERNAL LAUNCH em ambiente de teste | todos | [ ] |

## Primeira sessão no Maestri (starter FIRST_SESSION)
1. Abrir `00-COMMAND-CENTER` apontando para a raiz do repo.
2. Iniciar ENG-00 ARCHITECT; `/context`; verificar que `CLAUDE.md` carregou e ler DECISIONS, DEFINITION_OF_DONE, ROADMAP.
3. Pedir ao Architect uma **auditoria** contra o CLAUDE.md (sem implementar): artefatos existentes, lacunas, riscos, dependências e ordem de execução do Milestone 1.
4. Só então abrir ENG-01 a ENG-07 conforme a tarefa.
5. Primeira mudança de código = fundação do Core (Postgres + Redis + migrations + event model + audit log + agent registry + approval gates + launch state machine).
6. Nenhum agente começa por telas bonitas, prompts gigantes ou centenas de workflows antes de os contratos do Core estarem estáveis.

## Pipeline por feature (Blueprint §33)
Humano → Architect → Spec/ADR → { Backend, Database, N8N } → Agent Engine → { Frontend, QA } → DevOps → Integração → Release.

## Formato obrigatório de tarefa para Claude Code (Blueprint §42)
`CONTEXT · OBJECTIVE · SCOPE · INPUTS · OUTPUTS · CONSTRAINTS · DEPENDENCIES · TESTS · DEFINITION OF DONE`.

## Maestri (referência, Blueprint §3–§6, §10–§11)
- **Workspaces:** 00-COMMAND-CENTER · 01-CORE-PLATFORM · 02-AGENT-ENGINE · 03-N8N · 04-LAUNCH-ENGINE · 05-DASHBOARD · 06-QA-SECURITY · 07-KNOWLEDGE-PLF.
- **Partituras:** 00-bootstrap (Architect, Backend, Database, DevOps) · 01-core-platform (+N8N, Agent Engine, QA) · 02-agent-engine · 03-n8n · 04-launch-engine · 05-full-stack (todos; montagem padrão diária) · 06-qa-release.
- **Notas compartilhadas:** PROJECT_STATUS · CURRENT_TASK · ARCHITECTURE · DECISIONS · BLOCKERS · AGENT_STATUS · RELEASE_NOTES (conjunto exato em aberto, P07).
- **Regra de comunicação:** o Architect coordena; especialistas podem conversar direto para reduzir dependências; decisões arquiteturais voltam ao Architect e são registradas em DECISIONS/ADR.
- **O humano** prioriza, aprova, interrompe, reorienta, resolve decisões e avalia riscos — não controla linha a linha.
- Windows: usar WSL ou local/tmux só depois de instalar e verificar Claude Code e toolchain; manter o repo em um único local canônico.

## Relação com M00–M06 (canônico)
Os 15 passos acima foram escritos pelo Blueprint com a numeração antiga. Pelo roadmap canônico ([[roadmap]]): passos 1–6 e 9 ≈ **M00**; passos 7–8 ≈ **M01**; passo 10 ≈ **M02**; passo 14 ≈ **M03**; passo 15 ≈ validação de M03/M04; passo 11–12 acompanham M01–M04; passo 13 ≈ **M06** (painel mínimo antes, completo depois). Alocação por inferência (P12).

## Antes do passo 1 (M00)
- **Início formal do M00** pela Madu: nenhum código/repositório é criado como consequência automática do DEC-0010 (§10).
- Decisões abertas que tocam o M00: P07 (notas pinadas no Maestri), P10–P12 em [[pendencias-de-decisao]].
- Ainda sem resposta em [[perguntas-em-aberto]]: stack de backend/frontend, provedor de LLM, hospedagem, ambiente Windows (WSL/tmux/local), local do repo.
- Guardrails oficiais de ENG-00…07 (DEC-0010 §8) entram nas definições de agente ao importar o starter.

Ver [[roadmap]] · [[milestones]] · [[CORE-v0.1]] · [[definicao-de-pronto]] · [[engenharia-eng-00-a-07]].
