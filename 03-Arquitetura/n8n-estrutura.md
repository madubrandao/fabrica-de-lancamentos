---
tipo: arquitetura
area: 03-Arquitetura
fonte: blueprint v1 §22–§26; PLANO FINAL §22; starter .claude/rules/n8n.md
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# n8n — estrutura de workflows

## Pastas por domínio
```text
n8n/
00_SYSTEM · 01_CEO · 02_INTELLIGENCE · 03_OFFER_PRODUCT · 04_CONTENT_COPY · 05_AUDIENCE_CRM
06_ACQUISITION · 07_LAUNCH · 08_SALES · 09_CUSTOMER_SUCCESS · 10_PARTNERSHIPS · 11_ANALYTICS
99_ERRORS   (Plano Final: 99_ERROR_RECOVERY — ver P06)
```
Exemplo no Plano Final: `01_CEO/` contém `create_objective`, `delegate_task`, `approval_gate`, `launch_decision`, `cycle_close`.

## Padrão
`TRIGGER → VALIDATE → LOAD CONTEXT → EXECUTE → WRITE RESULT → EMIT EVENT → LOG → NOTIFY IF NEEDED`

## Regras (starter)
- Estado de negócio no backend/banco, não escondido no workflow.
- Workflows idempotentes onde houver replay.
- `correlation_id` em toda execução ligada a um lançamento.
- Retry com backoff limitado e caminhos de falha explícitos.
- Separar subworkflows reutilizáveis da orquestração específica do lançamento.
- Nunca colocar segredos no JSON exportado.
- Lógica de domínio independente de detalhes do n8n.

## Workflows de negócio descritos no Blueprint
1. **Create Launch** (primeiro workflow integrado): Webhook/API → Validate → Create Launch → A0 → A1 → A2 → A3 → A4 → A6 → Approval Gate → `LAUNCH_READY`. Entrada: `product_id`, `launch_type`, `objective`.
2. **Open Cart:** Launch Ready → QA técnico (sales page, links, order form, checkout, thank-you, confirmação, CRM, tracking) → Approval → Cart Open → emite `launch.cart_opened`.
3. **Monitoring:** tráfego, cliques, visitas ao order form, pedidos, conversão, reembolsos, suporte, engajamento de CRM → consumidos por A10 (dashboards e alertas).
4. **Post-Launch:** Cart Closed → clientes→A8; não compradores→A4/A7; métricas→A10; dados de parceiros→A9 → Post Mortem → Learning → A0 Decision.

## Testes de n8n (Blueprint §31)
happy path · retry · timeout · evento duplicado · falha parcial · falha de API · input inválido · rollback.

## Pendente
Os 144 workflows (A0-01…A10-13) estão só **listados por nome** ([[05-Agentes/README|05-Agentes]]); entradas, saídas e eventos de cada um ainda não existem.
