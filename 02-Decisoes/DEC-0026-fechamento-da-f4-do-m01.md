---
tipo: decisao
area: 02-Decisoes
id: DEC-0026
status: proposta
fonte: M01-plano-de-execucao (F4); plano único da F4 aprovado em 2026-10-07; spec-core-01 a 07; ADR-0019 e ADR-0020 no launch-os; evidências de 2026-10-07
data-registro: 2026-10-07
data-decisao: aguardando a Madu
---
# DEC-0026 — Fechamento da F4 (API fina, state machine e aprovações) do M01

> **PROPOSTA.** O PR #4 do `launch-os` foi mesclado na `main` (merge `a142863`) com a CI verde e a revisão independente registrada abaixo, como o plano autorizava. O fechamento da fase é decisão da Madu.

## Entrega
- **`apps/api` (Fastify 5):** autenticação (operador: argon2 + JWT HS256; agente e serviço do n8n: credencial opaca com hash SHA-256); `POST /launches`; transições com `FOR UPDATE`; aprovações e os 12 gates (G12 por verificação); decisão de continuidade (A0 propõe, Madu confirma); `BUDGET_AUTHORIZED` (nega sem limite); claim, ack e correlação para o n8n; agendador (vence aprovações, entrega, reabre claims); audit de toda mutação, aceita ou negada, com `correlation_id`.
- **`apps/cli`:** `bootstrap` (uma vez só, mesmo concorrente; segredos mostrados uma vez), `login`, `approvals list|grant|reject`.
- **Migration 0008:** `decision.proposed_product_id` e `proposed_offer_id` (lacuna da especificação achada na implementação; ADR-0020).
- ADR-0019 e ADR-0020.

## Evidências (2026-10-07)
- **402 testes** (57 unitários, 345 de integração) contra PostgreSQL 17.11: matriz das 14 transições, matriz dos 12 gates, segurança (JWT `none`/HS512/expirado, credencial revogada, R8), concorrência, idempotência, completude do audit, continuidade (6 opções), orçamento, CLI.
- **Provas negativas:** sem o `FOR UPDATE` da transição, o teste de concorrência falha; sem a checagem de token humano, os testes de R8 falham.
- Bug achado pelos testes do CLI e corrigido: data de sessão ilegível valia como não expirada.
- Lint, formatação, tipos e `check-repo.sh` verdes; CI do GitHub verde no PR #4.
- **Revisão independente** (agente diferente do autor, somente leitura): sem achado crítico; 2 altos, 6 médios e 8 baixos. **Tratados no PR:** o agente forjar a ação de continuidade e travar a varredura de vencimento e a entrega (A1; ação reservada, aprovação órfã sem efeito, etapas do agendador isoladas); a lista de aprovações não mostrava o que a Madu aprova na continuidade (A2; agora mostra opção, produto e oferta); 401/403 de qualquer rota no audit; falha do audit da negação no log; teto de tamanho no login; `launch_id` validado na organização; travas e `catch` do JOINT; `budget.check` sem Idempotency-Key. 7 testes de regressão; **352 testes de integração** no total.

## Itens PROP para a sua confirmação
1. Política `budget_limit` com valor `{"amount": N}`.
2. Prefixo `lo_` nas credenciais opacas (32 bytes em base64url).
3. Formato de erro padronizado `{error: {code, message, correlation_id}}`.
4. `approval_state = PENDING` quando o gate é exigido e ainda não pedido.
5. `allowBuilds: argon2: false` no pnpm (argon2 carrega por binários pré-compilados).
6. Migration 0008 e o fluxo de continuidade (ADR-0020), incluindo a reconferência da regra do JOINT na confirmação.
7. Gate de transição faltando responde 403 (`gate_not_cleared`).

## Pendências que a revisão achou e ficaram para a sua decisão (ADR-0019)
1. **Aprovação de ação genérica mostra só o hash** (ex.: o valor de um gasto): guardar os parâmetros legíveis exige a **migration 0009**, que o plano não autorizou. Recomendo fazer antes do M04.
2. **Dependências de gate da spec:** G07 não exige G05 e G06 antes; G08 não consulta orçamento (M04). Recomendo aceitar a ordem livre dentro da T7.
3. **Resposta perdida do `budget.check`** com aprovação já consumida: a Madu aprova de novo (uso único é intencional).
4. Login sem bloqueio por repetição; itens menores listados no ADR-0019.

## PENDENTE (Madu; sem padrão no código)
Duração da sessão, intervalo e lote do agendador, prazo de aprovação (`approval_expiry`), limite de orçamento e de tentativas de login. Sem eles: nada expira, gasto é negado e o agendador não liga.

## Decisão proposta
1. Aprovar o fechamento da F4 e confirmar os 7 itens PROP.
2. PR #4 mesclado na `main` (feito).
3. Próxima fase: **F5 (Agent Registry e permissões)**, com plano único.

Ver [[DEC-0025-fechamento-da-f3-do-m01]] · [[M01-plano-de-execucao]].
