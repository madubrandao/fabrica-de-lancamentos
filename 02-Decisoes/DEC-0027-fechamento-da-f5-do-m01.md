---
tipo: decisao
area: 02-Decisoes
id: DEC-0027
status: proposta
fonte: M01-plano-de-execucao (F5); plano único da F5 aprovado em 2026-10-07; spec-core-05 (seções 4, 5, 9, 10); ADR-0021 no launch-os; evidências de 2026-10-07
data-registro: 2026-10-07
data-decisao: aguardando a Madu
---
# DEC-0027 — Fechamento da F5 (Agent Registry e permissões) do M01

> **PROPOSTA.** O PR #5 do `launch-os` foi mesclado na `main` (merge `511d750`) com a CI verde e a revisão independente registrada abaixo, como o plano autorizava. O fechamento da fase é decisão da Madu.

## Entrega
- **Modelo de permissões** (`packages/schemas`): nega por padrão; 22 entidades; registro versionado de 7 ações; `requires_approval` contido em `actions`; permissões malformadas negam tudo.
- **Registry na API:** só o operador lista, consulta, troca permissões (com `expected_version` opcional) e ativa/desativa; audit do antes e do depois ligado à entidade `agent`; sem evento (D5.9). Agente, serviço e anônimo são negados e auditados.
- **`POST /permissions/check`** para o n8n (informativo; a API reaplica em cada rota).
- **Aplicação:** `requirePermission` somada às regras de dono em toda rota mutadora de agente.
- **Semente 0002:** permissões mínimas, nunca sobrescreve alteração da Madu. **CLI:** `agents list|show|permissions|activate|deactivate`. ADR-0021. Sem migration e sem dependência nova.

## Evidências (2026-10-07)
- **72 testes unitários e 379 de integração** passando contra PostgreSQL 17.11; lint, formatação, tipos e `check-repo.sh` verdes; CI do GitHub verde.
- **Prova negativa:** sem a checagem de permissão, 10 testes falham.
- **Revisão independente** (agente diferente do autor, somente leitura): sem achado crítico nem alto; 3 médios e 8 baixos. Tratados no PR: controle otimista de versão (`expected_version`), audit por entidade, 400 para agente ausente na consulta, restauração da semente nos testes, teste de concorrência mais forte, CLI sem repetir arquivo inválido. **Registrados como limite conhecido no ADR-0021:** `read`/`write`/`requires_approval`/`budget_limits` são informativos no M01; janela de corrida na revogação; audit de `/permissions/check` sem limite de taxa.

## Itens PROP para a sua confirmação
1. Ação `request_action_approval` acrescentada ao registro (pedido de aprovação de ação, necessário para gasto).
2. `claim_event` e `ack_event` fora do registro (são da credencial de serviço do n8n).
3. Semente: A0 com 5 ações; A1 a A10 com `request_transition`, `request_approval` e `verify_gate` (as regras de dono limitam quem pode de fato); ninguém com `check_budget` nem `request_action_approval` até você conceder.
4. `read` e `write` ficam informativos até o M02.

## PENDENTE (Madu)
Escopos reais dos 11 agentes (`read`, `write`, ações de ferramentas, orçamento por agente): M02 e M03. Mais os valores já pendentes da F4 (sessão, agendador, prazo de aprovação, orçamento, login) e a migration 0009 (parâmetros legíveis da aprovação de ação) antes do M04.

## Decisão proposta
1. Aprovar o fechamento da F5 e confirmar os 4 itens PROP.
2. PR #5 mesclado na `main` (feito).
3. Próxima fase: **F7 (Qualidade)**, que acompanha as demais, e depois **F8**; a Madu escolhe a ordem e aprova o plano único.

Ver [[DEC-0026-fechamento-da-f4-do-m01]] · [[M01-plano-de-execucao]].
