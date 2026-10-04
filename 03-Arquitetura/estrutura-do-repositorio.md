---
tipo: arquitetura
area: 03-Arquitetura
fonte: blueprint v1 §7–§9, §12, §40; PLANO FINAL §23; starter zip
status: canonico (DEC-0013 resolve P06 e P08)
atualizado: 2026-10-04
---
# Estrutura do repositório `launch-os` (a ser criado — ainda não existe)

> Este vault é a **memória do projeto**. O repositório de código `launch-os` é outro, ainda não criado. Não confundir.

## Estrutura (Blueprint §8 — vigente, [[DEC-0013-estrutura-do-repositorio-e-fonte-unica]])
```text
launch-os/
├── CLAUDE.md · README.md
├── apps/        api · dashboard · worker
├── packages/    core · agents · memory · events · schemas · integrations · observability
├── database/    migrations · seeds · schema
├── n8n/         workflows (pastas de domínio 00_SYSTEM … 99_ERRORS) · subworkflows · fixtures
├── knowledge/   plf · playbooks · policies · glossary
├── docs/        architecture · adr · agents · events · workflows · operations
├── tests/       unit · integration · e2e · evals
├── infrastructure/  docker (só Dockerfiles) · compose (docker-compose.dev.yml único) · deployment
└── scripts/
```
Árvore do Plano Final §23 (`agents/ workflows/ schemas/ prompts/ policies/ playbooks/ tests/ docs/ infra/`): **descartada** (DEC-0013 D2). `prompts/` não tem equivalente no Blueprint: **PENDENTE**.

## Fonte única por assunto (DEC-0013 D4, D6)
| Assunto | Fonte única |
|---|---|
| Contratos de API, evento e agente | `packages/schemas/` |
| Publicar/consumir eventos (código) | `packages/events/` |
| DDL | `database/schema/` |
| Decisões (ADR) | `docs/adr/` (`docs/status/DECISIONS.md` é só índice) |
| Conhecimento do livro | `knowledge/plf/` (sem o PDF) |
| Compose de desenvolvimento | `infrastructure/compose/docker-compose.dev.yml` |

`docs/events/` e `docs/agents/` só referenciam; não redefinem. `docs/decisions` e `docs/knowledge` do starter não existem na estrutura-alvo.

## CLAUDE.md por domínio (Blueprint §7)
Raiz + `apps/api` · `apps/dashboard` · `packages/agents` · `packages/memory` · `packages/events` · `n8n` · `database` · `infrastructure`.
Hierarquia: ROOT → DOMAIN → TASK CONTEXT. O contexto local nunca contradiz regra de segurança/arquitetura da raiz sem ADR explícito.
Conteúdo obrigatório do CLAUDE.md raiz: PROJECT PURPOSE · ARCHITECTURE · DOMAIN MODEL · TECH STACK · REPOSITORY RULES · DATABASE RULES · EVENT RULES · AGENT RULES · N8N RULES · SECURITY RULES · TESTING RULES · GIT RULES · DOCUMENTATION RULES · DEPLOYMENT RULES.

## Conhecimento PLF (`knowledge/plf/`, Blueprint §12)
19 arquivos: 01-core-principles · 02-list · 03-sideways-sales-letter · 04-mental-triggers · 05-pre-pre-launch · 06-pre-launch · 07-plc1 · 08-plc2 · 09-plc3 · 10-cart-open · 11-cart-close · 12-post-launch · 13-seed-launch · 14-internal-launch · 15-joint-launch · 16-business-launch-formula · 17-evergreen · 18-relaunch · 19-circle-of-awesome.
Cada arquivo separa: SOURCE PRINCIPLE · IMPLEMENTATION RULE · INPUT · OUTPUT · DECISION · EXAMPLE · DO NOT.
**O PDF não entra nesta árvore nesta fase** ([[DEC-0010-decisoes-canonicas-fase-1-v1]] §2): a pasta `knowledge/plf/` fica reservada. Arquivos só serão preenchidos com base nas notas já registradas; lacunas ficam PENDENTES até consulta explícita à fonte.

## Isolamento de trabalho (Blueprint §9)
Branches/worktrees por domínio: `feat/core-platform · database · n8n · agent-engine · frontend · qa · devops`. Cada agente altera só seu domínio, salvo autorização do Architect. Antes de alterar contrato compartilhado: registrar decisão → avisar dependentes → alterar schema/contrato → atualizar testes → executar integração.

## O que o starter zip traz
Ver [[starter-zip-inventario]] (em 08-Referencias).
