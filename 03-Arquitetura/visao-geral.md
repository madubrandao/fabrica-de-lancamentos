---
tipo: arquitetura
area: 03-Arquitetura
fonte: blueprint v1 §1, §45–§46; PLANO FINAL §30
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# Visão geral da arquitetura

## Camada de produção
```text
USUÁRIO → DASHBOARD / API FINA (estado, gates, aprovações, audit)
                              └── N8N (orquestra e executa os agentes A0-A10) → LLMs / TOOLS / APIs / WEBHOOKS
                                   ↓
                              POSTGRESQL ── REDIS ── MEMORY ── EVENTS
```

## Camada de desenvolvimento
```text
HUMANO → Claude Code (subagentes ENG-00…07) → GITHUB/GIT → CÓDIGO + DOCS + TESTES
```
Ver [[DEC-0002-duas-camadas-maestri-so-desenvolvimento]] e [[engenharia-eng-00-a-07]].

## Papéis
| Peça | Papel |
|---|---|
| ~~Maestri~~ | fora do escopo ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]) |
| Claude Code | executor de engenharia |
| n8n | orquestrador de processos de negócio e executor dos agentes A0-A10 |
| Backend (API fina) | estado, regras críticas, gates, aprovações, audit e eventos; escopo final PENDENTE |
| PostgreSQL | fonte de verdade do estado estruturado |
| Redis | cache, filas, execução assíncrona |
| A0–A10 | força de trabalho cognitiva da empresa |
| Humano | autoridade máxima nas decisões críticas |

## Camadas transversais
- **Company Memory** ([[memoria-empresarial]]): Market, Audience, Products, Offers, Launches, Content, Customers, Partners, Metrics, Experiments, Decisions, Learnings.
- **Event Bus / Orquestração** ([[catalogo-de-eventos]]): n8n sobre APIs, LLMs e banco.

## Launch Operating System
`KNOWLEDGE → POLICIES → AGENTS → EVENTS → WORKFLOWS → LAUNCH STATE MACHINE → DATA → ANALYTICS → LEARNING → NEXT DECISION`

## Fluxo de agentes (hierarquia)
Humano → A0 (CEO/Orquestrador) → A1 (inteligência), A2 (oferta/produto), A4/A5 (crescimento) → A3 (comunicação) → A6 (operações) → A7 (vendas) e A8 (clientes) → A9 (parcerias) → A10 (analytics) → de volta a A0. Detalhe em [[05-Agentes/README|05-Agentes]].

## Padrão de workflow n8n
`TRIGGER → VALIDATE → LOAD CONTEXT → EXECUTE → WRITE RESULT → EMIT EVENT → LOG → NOTIFY IF NEEDED` ([[n8n-estrutura]]).

## Documentos relacionados
[[modelo-de-entidades]] · [[launch-state-machine]] · [[gates]] · [[permissoes-e-hitl]] · [[estrutura-do-repositorio]] · [[observabilidade-e-testes]]
