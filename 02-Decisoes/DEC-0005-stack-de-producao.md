---
tipo: decisao
area: 02-Decisoes
id: DEC-0005
status: aceita-nas-fontes-parcial
fonte: blueprint v1 (cabeçalho, §1.2, §46, §48); starter CLAUDE.md, docker-compose.yml, .env.example
data-registro: 2026-10-02
---
# DEC-0005 — Stack de produção

**Decidido nas fontes:**
| Componente | Papel |
|---|---|
| Backend | regras de negócio, APIs, event processing, runtime dos agentes |
| n8n | orquestração de workflows de negócio e integrações |
| PostgreSQL | estado transacional e memória estruturada (fonte de verdade do estado estruturado) |
| Redis | cache, filas e controle de execução assíncrona |
| Git/GitHub | fonte da verdade do código, revisão, branches, CI/CD |
| Docker | desenvolvimento local (starter: postgres 17, redis 8, n8n) |

**Regras:** estado de negócio fica no backend/banco, não escondido no n8n; estado operacional nunca depende só de memória semântica.

**Decidido depois ([[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]], 2026-10-04):** TypeScript/Node; LLM via adaptador com Anthropic primeiro; ambiente de dev em Docker local; vector store adiado.
**Ainda NÃO decidido (ver [[perguntas-em-aberto]]):** framework de backend e frontend; modelo por agente; hospedagem de teste e produção.
**Observação:** o `docker-compose.yml` do starter fixa `n8nio/n8n:latest`; a regra de produção do starter pede artefatos pinados após o início do release. Registrar versão ao iniciar a camada de código. Compose único em `infrastructure/compose/docker-compose.dev.yml` ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] D5).
