---
tipo: agente
area: 05-Agentes
id: A8
camada: producao
fonte: PLANO FINAL §11; blueprint v1 §13, §26
status: ficha-parcial (prompt, tools, schemas pendentes)
atualizado: 2026-10-02
---
# A8 — Customer Success

**Absorve:** atendimento + experiência + retenção. Plano Final Fase 10.

## Missão
Fechar o ciclo; não apenas responder suporte. O livro destaca atendimento, acompanhamento dos clientes, superação de expectativas e continuidade da relação após a venda.

## Entradas / Saídas
- **Entra:** order, customer, product, support events.
- **Sai:** onboarding, support, satisfaction, feedback, retention actions.

## Workflows (12)
A8-01 Onboarding · A8-02 Entrega de acesso · A8-03 Atendimento · A8-04 Classificação de tickets · A8-05 FAQ · A8-06 Escalonamento humano · A8-07 Pesquisa de satisfação · A8-08 Coleta de feedback · A8-09 Identificação de novas necessidades · A8-10 Retenção · A8-11 Recompra · A8-12 Indicação.
Construir (Fase 10): Onboarding, Support, Feedback, Satisfaction, Retention, Referral.
n8n: `09_CUSTOMER_SUCCESS/`. Eventos: `customer.created/onboarded/support_requested`.

## Autonomia
Classificação de tickets: AUTO. Escalonamento humano é workflow próprio (A8-06). Demais: **PENDENTE**.

## Regra de ouro — pendências
Scopes · KPIs · prompt · ferramenta de suporte e de entrega de acesso (não definidas) · schemas: **PENDENTE**.
