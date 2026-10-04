---
tipo: fase
area: 04-Fases
fonte: PLANO FINAL §31; blueprint v1 §41; starter CLAUDE.md ("First milestone")
status: nao-iniciado
atualizado: 2026-10-02
---
# CORE v0.1 — primeira entrega de desenvolvimento

Itens (Plano Final §31):
1. PostgreSQL
2. Schema inicial
3. Company Memory
4. Event Bus
5. Agent Registry
6. Permission System
7. Launch State Machine
8. A0 CEO
9. A1 Intelligence
10. A2 Offer/Product
11. A3 Content/Copy
12. A4 CRM
13. Primeiro fluxo ponta a ponta de lançamento
14. Audit Log
15. Approval Gates

Depois conectam-se A5–A10.

**Versão do starter (mais restrita):** "LAUNCH OS CORE" = PostgreSQL + Redis + event model + agent registry + memory interfaces + approval gates + launch state machine + **A0 skeleton**. Regra: não construir o dashboard completo antes de os contratos do Core estarem estáveis.
Diferença entre as duas definições de escopo do Core (A0–A4 prontos x só esqueleto de A0): com o roadmap canônico M00–M06 ([[roadmap]]), A0–A4 pertencem ao M03 e a plataforma ao M01/M02. "CORE v0.1" é **nome histórico** e equivale a **M01+M02+M03** ([[DEC-0015-alocacao-do-roadmap-m01-m04]], P11 resolvida). Company Memory (item 3): interfaces no M02, tabelas no schema do M01. Primeiro fluxo ponta a ponta (item 13): Create Launch até `LAUNCH_READY` no M03, com A6 mínimo.

## Primeiro pedido ao Architect (Blueprint §41)
Ler o CLAUDE.md e toda a documentação de arquitetura. **Não implementar.** Produzir a especificação técnica do Core Platform com: arquitetura de serviços; entidades; state machine de Launch; event catalog; permission model; agent contract; memory contract; approval gates; estrutura de diretórios; ADRs iniciais. Não inventar requisitos de negócio; decisão indefinida vira "decisão pendente". Depois delegar: Database → schema; Backend → API/state machine; DevOps → Docker.
