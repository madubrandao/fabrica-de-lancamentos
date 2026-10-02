---
tipo: arquitetura
area: 03-Arquitetura
fonte: blueprint v1 §18; PLANO FINAL §16, §24 (Fase 1)
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# Memória empresarial (Company Memory)

Decisão: [[DEC-0009-memoria-empresarial-em-tres-tipos]]. Não é uma "base de prompts".

## Tipos
| Tipo | Onde | Conteúdo |
|---|---|---|
| Estruturada | PostgreSQL | facts, entities, relationships, preferences, segments, objections, metrics, experiments, learnings, decisions |
| Semântica | embeddings (somente se agregar valor) | feedback, conversations, support, research, content, launch reports, post-mortems |
| Operacional | estruturas transacionais | estado atual; nunca depende só da memória semântica |

## Escopo por domínio
Market · Audience · Products · Offers · Launches · Content · Customers · Partners · Metrics · Experiments · Decisions · Learnings.

## Fase de construção (Plano Final, Fase 1 — Memory + Database)
PostgreSQL · Schema · Entities · Relations · Event log · Vector/semantic memory · Document storage.

## Aprendizado
`dados → diagnóstico → insight → experimento → resultado → aprendizado → memória` (A10). Cada lançamento alimenta o seguinte.

## Pendente
Modelo de dados de `facts/entities/relationships`, política de retenção e auditoria, escolha de vector store, contratos de leitura/escrita por agente (ver [[permissoes-e-hitl]]).
