---
tipo: decisao
area: 02-Decisoes
id: DEC-0022
status: aceita
fonte: decisão da Madu em 2026-10-04; DEC-0010 §7 e §10; DEC-0021; plano aprovado em M01-plano-de-execucao
data-registro: 2026-10-04
---
# DEC-0022 — Início formal do M01 Core Platform

## Contexto
O M00 foi concluído no [[DEC-0021-m00-foundation-concluido]]. O [[DEC-0010-decisoes-canonicas-fase-1-v1]] §10 exige o início formal de cada marco antes de qualquer código. O plano do M01 foi aprovado pela Madu em [[M01-plano-de-execucao]].

## Decisão
A Madu autorizou, em 2026-10-04, o **início formal do M01 Core Platform**, com o escopo do [[M01-plano-de-execucao]] (fases F0 a F8): PostgreSQL, Redis, migrations reversíveis, eventos e event schema, audit log, Agent Registry, permissions, approvals (12 gates e `BUDGET_AUTHORIZED`), launch state machine e API skeleton, com a API fina e o n8n executando os agentes ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]). **M02 e seguintes não estão autorizados.**

## Regras do M01
1. **Especificação primeiro:** a F1 é uma especificação pelo `eng-00-architect`, sem código. **Nenhum código de aplicação, schema ou migration é criado antes de a Madu aprovar a especificação** (portão F1). A aprovação da especificação será registrada em DEC próprio.
2. **Ferramentas de código** (framework, migrations, validação, testes, versão do Node, gerenciador de pacotes): o ENG-00 propõe e a Madu aprova em ADR. Nada é fixado antes.
3. **Humanos:** só a Madu opera e aprova (1 pessoa), uma única `Organization`.
4. Cada ação de execução é confirmada com a Madu; ações externas ou difíceis de desfazer (criar recursos, push) só com confirmação explícita.
5. Nenhum segredo é versionado. Os guardrails oficiais (DEC-0010 §8) não mudam.
6. **Revisão independente por convenção:** a proteção de branch **não está disponível** neste repositório privado (a API do GitHub respondeu que exige GitHub Pro ou repositório público; verificado em 2026-10-04). Por isso o trabalho vai em branches `feat/*` com pull request, e o merge só ocorre depois da revisão por outro agente e da CI verde. Não há trava técnica; vale a regra.
7. **Branches por fase:** cada branch `feat/*` é criada quando a fase correspondente começa, não antes. A F1 começa em `feat/core-platform`.

## Evidências da F0 (2026-10-04)
| Pré-condição | Resultado |
|---|---|
| Disco | 22 GB livres em `C:` (mínimo 20 GB, atendido) |
| `launch-os` | sincronizado com o GitHub; CI `success` em `f00f597`; `check-repo.sh` aprovado |
| Vault | sincronizado |
| Claude Code | 2.1.289 |
| RAM | cerca de 0,9 GB livres; a F1 não sobe a pilha; ao subi-la (F2 em diante), parar os 4 containers de outros projetos (`docs/status/BLOCKERS.md` do `launch-os`) |
| Humanos | respondido: só a Madu |

## PENDENTES que o M01 resolverá (cada um por ADR ou DEC, com aprovação da Madu)
Ferramentas de código; entrega de eventos e papel do Redis; autenticação entre o n8n e a API e do operador; mapeamento estado → gate, aprovador e critérios de cada gate; expiração da aprovação e `approval.expired`; campos da decisão de continuidade; banco e usuário do n8n separados; senha do Redis; escopo do ENG-04, de `packages/agents` e do gateway do ENG-01; retenção do audit log e dos eventos.

## Consequências
- O estado do projeto passa de "M00 concluído; M01 não autorizado" para "M01 em andamento, na F1 (especificação)".
- Continua proibido código de aplicação, schema e migration até a aprovação da especificação.

Ver [[M01-plano-de-execucao]] · [[milestones]] · [[roadmap]].
