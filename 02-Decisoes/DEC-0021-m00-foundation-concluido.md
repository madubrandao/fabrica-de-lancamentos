---
tipo: decisao
area: 02-Decisoes
id: DEC-0021
status: aceita
fonte: decisão da Madu em 2026-10-04 (fechamento do M00); evidências do passo 11 em M00-plano-de-execucao; revisão independente completa
data-registro: 2026-10-04
---
# DEC-0021 — M00 Foundation concluído

## Contexto
O M00 foi iniciado formalmente no [[DEC-0018-inicio-formal-do-m00]] e teve o escopo ajustado pelo [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]] (sem Maestri). O critério de validação é o do DEC-0010 §7, sem o Maestri: fundação documental, repositório, Claude Code e infraestrutura de desenvolvimento.

## Decisão
A Madu aprovou, em 2026-10-04, o **fechamento do M00 Foundation**. **O M01 não está autorizado:** ele só começa com um novo início formal da Madu, como foi com o M00.

## Evidências do fechamento (2026-10-04)
| Critério | Evidência |
|---|---|
| Repositório | `madubrandao/launch-os`, privado, branch `main`, 86 arquivos, commit `44d9a16`, sincronizado com o GitHub |
| CI | execução `37244532793`, `success`, 14 verificações `ok` (39 arquivos obrigatórios) |
| Infraestrutura de desenvolvimento | postgres 17.11, redis 8.10.2 e n8n 2.41.6 `healthy`; `/healthz` 200; segredos obrigatórios; portas só em `127.0.0.1` ([[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]]) |
| Claude Code | 2.1.289 instalado e verificado; 8 subagentes `eng-00` a `eng-07` com os guardrails do DEC-0010 §8 |
| Documentação | `CLAUDE.md` com 14 seções e 8 por domínio; `docs/status/`, `docs/adr/`, índice de arquitetura; wikilinks do vault sem quebra |
| Segredos | 0 padrões de segredo no histórico dos dois repositórios; nenhum `.env`, `.pem` ou `.key` jamais versionado |
| Revisão independente | duas passadas em 2026-10-04: parcial (5 achados) e completa do repositório (47 de 47 arquivos lidos, 10 achados); todos corrigidos e testados |
| Disco | 22 GB livres em `C:` (critério de 20 GB para o M01 atendido na última medição) |

Definition of Done aplicada à fundação: critérios de código, evento, schema, workflow n8n e dashboard são N/A no M00; testes de código entram no M01 (a CI valida estrutura e configuração).

## O que o M00 NÃO entregou (por desenho)
Nenhum código de aplicação, schema, migration, evento, agente ou workflow. O repositório tem a fundação: estrutura de pastas do Blueprint §8, agentes e regras, Docker de desenvolvimento, CI estrutural, documentação e estado.

## Antes de autorizar o M01 (PENDENTES, a decidir pela Madu)
- **Número de humanos** que operam e aprovam os lançamentos (DEC-0017 C3).
- **Disco:** manter pelo menos 20 GB livres em `C:` e RAM disponível (parar os containers de outros projetos ao subir a pilha; comandos em `docs/status/BLOCKERS.md` do `launch-os`).
- **Framework de backend e de frontend** (ADR no M01) e a **ferramenta de orquestração do desenvolvimento** (DEC-0020).
- **Desenho do ENG-00 para o M01:** escopo do ENG-04, de `packages/agents` e do `agent runtime gateway` do ENG-01 com o runtime no n8n; adaptador de LLM x nós de IA do n8n.
- **Para o M01 e depois:** mapeamento estado → gate e critérios por gate (DEC-0016 D7); limites numéricos de aprovação e SLA (D8); `approval.expired`; senha do Redis; banco e usuário do n8n separados do banco de negócio.
- **Antes do M03:** produto fictício de teste, definição do A6 mínimo, ambientes de teste e produção. **Antes do M04:** política de orçamento.

## Consequências
- O estado do projeto passa de "M00 em andamento" para "M00 concluído; aguardando decisão sobre o M01".
- Qualquer código de aplicação, schema ou migration continua proibido até o início formal do M01 (DEC-0010 §10).

Ver [[M00-plano-de-execucao]] · [[roadmap]] · [[milestones]].
