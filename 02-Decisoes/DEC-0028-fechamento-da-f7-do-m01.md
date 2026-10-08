---
tipo: decisao
area: 02-Decisoes
id: DEC-0028
status: proposta
fonte: M01-plano-de-execucao (F7); plano único da F7 aprovado em 2026-10-07; docs/status/QUALITY_EVIDENCE.md no launch-os; evidências de 2026-10-07
data-registro: 2026-10-07
data-decisao: aguardando a Madu
---
# DEC-0028 — Fechamento da F7 (Qualidade) do M01

> **PROPOSTA.** O PR #6 do `launch-os` foi mesclado na `main` (merge `d947eca`) com a CI verde e a revisão independente registrada abaixo, como o plano autorizava. O fechamento da fase é decisão da Madu.

## Entrega
- **Failure injection com o PostgreSQL real** (gatilhos que falham ou derrubam a conexão no meio): transição, concessão, continuidade, troca de permissões, claim e entrega ficam sem estado pela metade; resposta 500 padronizada; a repetição com a mesma `Idempotency-Key` conclui.
- **Inventário rotas × guardrails** a partir das rotas registradas no Fastify: 401 antes de validar, `Idempotency-Key`, replay, audit de negação; isenções em lista fechada e sem escrita de negócio.
- **Varredura de segredos** em tabelas, logs e respostas, com controle positivo; **contrato dos 23 eventos** por tipo; testes do agendador e da CLI.
- **17 provas negativas** e **relatório de cobertura** em `docs/status/QUALITY_EVIDENCE.md`.

## Evidências (2026-10-07)
- **800 testes** (293 unitários, 507 de integração); lint, formatação, tipos e `check-repo.sh` verdes; CI verde. Cobertura total 92,7% (relatório; sem limite mínimo).
- **Defeitos achados e corrigidos:** (1) queda de conexão no meio de uma transação gerava erro sem ouvinte (derrubaria o processo); (2) conexão ociosa perdida no pool, idem; (3) ordem de validação: anônimo recebia 400 em vez de 401. Os dois primeiros foram provados por injeção em teste, não por incidente.
- **Revisão independente** (agente diferente do autor, somente leitura): sem achado crítico; 3 altos, 3 médios e 4 baixos. Tratados: varredura que podia passar vazia (agora com percurso próprio e controle positivo); cobertura aparente do inventário nas rotas isentas e na idempotência (lista fechada, teste de não escrita e replay por rota); asserções frouxas; redação do relatório; ouvintes de erro com limpeza.

## Itens para a sua decisão
1. **Audit de 401 anônimo sem limite de taxa:** cada chamada sem credencial grava uma linha imutável em `audit_log`; um flood anônimo infla a tabela. Recomendo decidir antes de expor a API além de `127.0.0.1` (amostragem, limite por IP, ou auditar só token presente e inválido).
2. **Limite mínimo de cobertura:** valor numérico seu; hoje só relatório.
3. Mudanças de produção além do plano, aceitas por serem bug claro com teste: os dois ouvintes de erro, a autenticação em `onRequest` e o observador `onRoute` do `buildApp` (só para o inventário).

## Decisão proposta
1. Aprovar o fechamento da F7 e decidir os itens acima.
2. PR #6 mesclado na `main` (feito).
3. Próxima fase: **F8 (Fechamento do M01)**: verificação final, revisão independente completa, Definition of Done com evidência e DEC de fechamento do marco; plano único a aprovar.

Ver [[DEC-0027-fechamento-da-f5-do-m01]] · [[M01-plano-de-execucao]].
