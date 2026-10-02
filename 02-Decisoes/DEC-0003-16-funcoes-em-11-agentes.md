---
tipo: decisao
area: 02-Decisoes
id: DEC-0003
status: aceita-nas-fontes
fonte: PLANO FINAL DE ENGENHARIA.txt §1–§2, §28; blueprint v1 §0; starter ADR-0003
data-registro: 2026-10-02
---
# DEC-0003 — 16 funções humanas → 11 agentes; agente é dono de responsabilidade, workflow implementa processo

**Contexto:** criar um agente por tarefa geraria dezenas de miniagentes desconectados.
**Decisão:** partir de 16 funções humanas de referência e fundi-las em **11 agentes** (A0–A10). Um agente é dono de uma responsabilidade e pode ter vários workflows. Não criar 16 agentes, nem 20+, nem agente que cria agente.
**Alternativas:** um agente por função humana; um agente por workflow (rejeitadas).
**Consequências:** 144 workflows catalogados nas fontes distribuídos entre A0–A10 (A0:12, A1:12, A2:14, A3:18, A4:15, A5:12, A6:14, A7:10, A8:12, A9:12, A10:13). Detalhe em [[05-Agentes/README|05-Agentes]].
**Nota:** o conjunto das 16 funções humanas não está listado nominalmente nos documentos lidos; só a tabela de absorção em 11 agentes. Ver [[perguntas-em-aberto]].
