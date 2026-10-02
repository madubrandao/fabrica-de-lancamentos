---
tipo: arquitetura
area: 03-Arquitetura
fonte: blueprint v1 §29–§32; starter .claude/rules/testing.md
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# Observabilidade, testes e painel

## Observabilidade (Blueprint §30)
Registrar: `agent.execution.*` · `workflow.*` · `api.request/error` · `llm.request/error` · `tool.execution` · `approval.*`.
Cada execução deve responder: quem executou · o que recebeu · o que decidiu · quais ferramentas usou · o que mudou · qual evento gerou · qual foi o resultado.

## Testes (Blueprint §31 + starter)
- **Código:** unit · integration · contract · E2E.
- **Agentes:** prompt evals · tool-use evals · decision evals · policy evals · regression evals.
- **n8n:** happy path · retry · timeout · duplicate event · partial failure · API failure · invalid input · rollback.
- **Sistema:** idempotency · concurrency · race conditions · permission boundaries · audit completeness.
- Regras do starter: transições de estado exigem unit test; contratos entre módulos exigem integration test; caminhos críticos exigem E2E; prompts de agentes exigem contract tests determinísticos (campos e políticas); teste falhando é bloqueio, salvo dispensa registrada pelo dono humano em DECISIONS.

## Painel humano (Blueprint §29)
Company: Overview · Products · Offers · Launches · Audience · Campaigns · Customers · Partners · Metrics · Experiments · Learnings · Agents · Approvals · Events · Audit Log.
Launch Dashboard: Launch State · Current Gate · Current Owner · Tasks · Dependencies · PLC Status · CRM Status · Traffic Status · Orders · Conversion · Alerts · Approvals · Timeline.
Regra do starter: não construir o dashboard completo antes de estabilizar os contratos do Core; frontend sem acesso direto ao banco; ações perigosas exigem confirmação e enforcement no backend.

## Definição de pronto
Ver [[definicao-de-pronto]].
