---
tipo: agente
area: 05-Agentes
id: A0
camada: producao
fonte: PLANO FINAL §3; blueprint v1 §13
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A0 — CEO / Orquestrador

**Absorve:** CEO + estratégia. **Caminho crítico:** início e fim do ciclo (A10 → A0).

## Missão
Transformar objetivos empresariais em decisões, prioridades e delegações.

## Decide
O QUE fazer · POR QUE · QUANDO · QUAL prioridade · QUAL orçamento · QUAL agente executa · SE pode avançar · SE precisa de aprovação humana.

## Entradas / Saídas
- **Entra:** objectives, company state, metrics, alerts, learnings.
- **Sai:** decisions, tasks, approvals, priorities, next state.
- **Saída central:** `{ decision_id, objective, priority, owner_agent, deadline, required_approval }`.

## Workflows (12)
A0-01 Criar objetivo estratégico · A0-02 Criar nova iniciativa · A0-03 Avaliar oportunidade · A0-04 Autorizar novo produto · A0-05 Autorizar novo lançamento · A0-06 Delegar tarefa · A0-07 Controlar dependências · A0-08 Revisar status geral · A0-09 Detectar bloqueios · A0-10 Aprovar decisão crítica · A0-11 Encerrar ciclo · A0-12 Iniciar próximo ciclo.
n8n: `01_CEO/` (create_objective, delegate_task, approval_gate, launch_decision, cycle_close).

## Autonomia ([[permissoes-e-hitl]])
- **Pode decidir:** sequência operacional, delegação, prioridade, iniciar processos não críticos, solicitar análises.
- **Requer humano:** mudanças estratégicas críticas, contratos relevantes, grandes alterações de orçamento, alterações de política, decisões legais, mudança de posicionamento central.

## Aciona
Todos os agentes. Exemplo de ciclo: A1 detecta objeção → A0 pede revisão a A2 → A2 → A0 pede atualização a A3 → A10 recomenda teste → A0 executa experimento → A10 registra aprendizado → A0 atualiza estratégia.

## Construção
Plano Final Fase 2: Objective Engine, Task Engine, Decision Engine, Approval Engine, Agent Router, State Machine. Primeiro workflow integrado: Create Launch ([[n8n-estrutura]]).

## Regra de ouro — pendências
Dados que pode acessar (read/write scopes) · KPIs · system prompt · ferramentas · schemas de entrada/saída · política de erro e auditoria: **PENDENTE** (Blueprint de Implementação v1).
