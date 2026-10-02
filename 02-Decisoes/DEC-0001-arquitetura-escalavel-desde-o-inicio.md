---
tipo: decisao
area: 02-Decisoes
id: DEC-0001
status: aceita-nas-fontes
fonte: blueprint v1 §0; starter docs/DECISIONS.md (ADR-0001)
data-registro: 2026-10-02
---
# DEC-0001 — Arquitetura escalável desde o início

**Contexto:** a empresa deve operar muitos lançamentos; reescrever um MVP descartável custaria caro.
**Decisão:** construir a arquitetura-alvo completa e **ativar a capacidade gradualmente**. Não construir MVP descartável.
**Alternativas:** MVP enxuto e reescrita depois (rejeitada).
**Consequências:** contratos (schemas, eventos, permissões) vêm antes de telas, prompts grandes ou centenas de workflows. Primeira versão ativa só o caminho crítico ([[DEC-0008-caminho-critico-primeiro]]).
**Responsável:** Madu (dono humano). Registrado nas fontes como decisão tomada.
