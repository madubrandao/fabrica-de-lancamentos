---
tipo: decisao
area: 02-Decisoes
id: DEC-0002
status: substituida-parcialmente (DEC-0020)
fonte: blueprint v1 §1–§2; starter docs/DECISIONS.md (ADR-0002); .claude/rules/architecture.md
data-registro: 2026-10-02
---
# DEC-0002 — Duas camadas: desenvolvimento x produção; Maestri só desenvolvimento

> **Substituída em parte pelo [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]] (2026-10-04):** continua a separação entre desenvolvimento e produção; o Maestri saiu do escopo e o desenvolvimento usa Claude Code com subagentes. O texto abaixo é o registro histórico.

**Contexto:** Maestri e Claude Code organizam o time que constrói; a empresa em produção não pode depender deles.
**Decisão:**
- Camada de **desenvolvimento**: Maestri (cockpit) + Claude Codes ENG-00…07 + GitHub/Git como fonte da verdade do código.
- Camada de **produção**: dashboard/API → backend → agent runtime + n8n → LLMs/tools/APIs → PostgreSQL (+ Redis, memória, eventos).
- Maestri e Claude Code podem ser desligados sem derrubar a produção.
- Maestri **não** é: banco, runtime de agentes, event bus, CRM, executor de workflows de lançamento, fonte de regras de negócio.
- Identidades separadas: A0–A10 (negócio) x ENG-00…07 (engenharia). Comunicação dev via conexões do Maestri; comunicação runtime via event bus de produção. Não acoplar os dois.
**Alternativas:** Maestri como parte do runtime (rejeitada: ponto único de falha).
**Consequências:** ver [[visao-geral]] e [[engenharia-eng-00-a-07]].
