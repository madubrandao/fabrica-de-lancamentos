---
tipo: decisao
area: 02-Decisoes
id: DEC-0024
status: aceita
fonte: M01-plano-de-execucao (F2 e F6); DEC-0023; especificação do Core (spec-core-01 a 07); ADR-0005 a ADR-0017 e adendos no launch-os; evidências de 2026-10-07
data-registro: 2026-10-07
data-decisao: 2026-10-07 (a Madu aprovou seguindo as recomendações do ENG-00)
---
# DEC-0024 — Fechamento da F2 (banco e schema) e da F6 (infra e CI) do M01

> **Aprovado pela Madu em 2026-10-07**, incluindo o push das branches `feat/devops` e `feat/database` e a abertura dos pull requests. O fechamento vale com a condição "CI verde no GitHub": se a CI acusar algo, a correção entra antes do merge (ver "Resultado da CI" ao final, quando houver).

## Contexto
O [[DEC-0023-aprovacao-do-portao-f1-especificacao-do-core]] liberou a F2 e a F6. A Madu aprovou em 2026-10-07 um plano único para as duas ([[M01-plano-de-execucao]]), com paradas só em: subir o Docker, atualizar o Node local, push e pull request, qualquer coisa fora da especificação e este fechamento.

## O que foi entregue (repositório `launch-os`, só local)
| Fase | Branch | Commits | Entrega |
|---|---|---|---|
| F6 | `feat/devops` | `5994369` | monorepo pnpm; Node, pnpm, TypeScript 6.0, ESLint 10, Prettier e Vitest 5 com versões exatas (ADR-0005 a ADR-0007, 0012, 0013); compose com **senha no Redis** e **n8n em banco e usuário próprios**; provisionamento idempotente dos papéis; `bootstrap.ps1` que acrescenta as senhas novas ao `.env` sem tocar nas existentes; CI com jobs de qualidade e de integração; teste de pares up/down |
| F2 | `feat/database` (sobre a F6) | `ce775c7`, `664fb84`, `e58d4ff`, `e0b90e8` | 7 migrations em SQL up/down: **22 tabelas** (8 do núcleo, 14 de domínio mínimas, D7.3) e **4 de infraestrutura**; `event` e `audit_log` imutáveis (privilégios, trigger por linha e por comando); semente de 1 organização e 11 agentes, sem segredo; ferramentas `migrate` e `seed`; 52 testes de integração |

## Evidências (2026-10-07)
- **52 testes de integração e 4 unitários passando**, contra PostgreSQL 17.11 em Docker. Lint, tipos, formatação e `check-repo.sh` aprovados.
- **Round-trip** das 7 migrations: sobe, desce tudo e sobe de novo deixam o schema idêntico (colunas, constraints, índices, triggers, funções, ACLs de tabela e de coluna). Também funcionou pelo CLI no banco de negócio.
- **Provas negativas:** sem o `REVOKE`, sem o trigger de `TRUNCATE` ou sem um `:?` do compose, os testes e o `check-repo.sh` falham.
- **Provisionamento:** funcionou em volume novo e é idempotente; falha de verdade (status 3) com senha vazia ou ausente. O healthcheck do Postgres por TCP só passa no servidor definitivo (verificado: papéis e bancos já existem quando fica saudável).
- **Compose:** o n8n subiu saudável no banco `n8n` (142 tabelas dele, nenhuma no banco de negócio); o Redis exige senha e roda como usuário `redis` (uid 999).
- **Revisão independente** (agente diferente do autor, somente leitura): sem achados críticos nem altos; 3 médios e 12 baixos. Todos tratados em `e58d4ff` e `e0b90e8`, exceto os itens listados em "Condições abertas".
- **Mudanças em relação ao esboço:** a revisão apontou que o `DELETE` concedido por padrão à aplicação contrariava a D2.1 (sem DELETE físico). Corrigido: só `event_delivery` e `idempotency_key` (limpeza do bloco 4) têm `DELETE`. Também: a aplicação não consegue forjar `event.position`, e o Redis voltou a rodar sem root.

## Decisão
1. **Aprovado o fechamento** da F2 e da F6 do M01 com as entregas acima.
2. **Autorizado o push** das branches `feat/devops` e `feat/database` e a abertura do(s) pull request(s) no `madubrandao/launch-os`, para a CI rodar no GitHub. O merge na `main` só com a CI verde e a revisão registrada (DEC-0022, regra 6).
3. **Confirmadas as constraints propostas** (PROP) listadas no cabeçalho de `database/migrations/0001_base.up.sql`.
4. A próxima fase é a **F3 (eventos)**. Ela só começa depois de um plano único aprovado pela Madu, com as mesmas paradas.

## Condições abertas (não bloqueiam o fechamento, mas ficam registradas)
- **CI no GitHub não executada.** No primeiro push, conferir: `psql` 15+ no runner (usa `\getenv`), `corepack` com `pnpm@11.28.2` e o cache do pnpm, `pnpm audit --audit-level=high` (pode bloquear por aviso novo em dependência de desenvolvimento; avaliar `continue-on-error`), download do Node 24.21.0, Redis com senha no job de integração.
- **Node local** continua na 24.16.0; a CI usa a 24.21.0. Atualizar é decisão da Madu.
- **Major do pnpm:** fixada a 11 (11.28.2); a 12 (`latest` 12.9.1) não foi avaliada (adendo no ADR-0006).
- **Snapshot em `database/schema/`:** PENDENTE.
- **`CREATEDB` do papel migrador** é necessário aos testes; retirar em produção (hospedagem antes do M03).
- **Falta o teste do DOWN com dados** nas tabelas e o teste de descer uma migration por vez; o round-trip hoje desce tudo de uma vez com as tabelas vazias.
- O banco de desenvolvimento do M00 foi **recriado** (`down -v`) com a autorização da Madu; os dados eram só de validação do M00.

## Consequências
- A F3 passa a ser a próxima fase. [[milestones]], [[roadmap]] e [[M01-plano-de-execucao]] foram atualizados em 2026-10-07.
- Mudar qualquer decisão técnica desta entrega exige ADR ou DEC (guardrail do ENG-00).

Ver [[DEC-0023-aprovacao-do-portao-f1-especificacao-do-core]] · [[spec-core-02-entidades-e-schema]] · [[spec-core-04-eventos]] · [[spec-core-05-permissoes-e-aprovacoes]].
