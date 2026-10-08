---
tipo: decisao
area: 02-Decisoes
id: DEC-0029
status: aceita
fonte: M01-plano-de-execucao (F8); plano único da F8 aprovado em 2026-10-07; docs/status/M01_DOD.md, QUALITY_EVIDENCE.md e docs/operations/M01-RUNBOOK.md no launch-os; revisões independentes completas de 2026-10-07
data-registro: 2026-10-08
data-decisao: 2026-10-08 (a Madu aprovou seguindo as recomendações do ENG-00)
---
# DEC-0029 — Fechamento do M01 Core Platform

> **Aprovado pela Madu em 2026-10-08.** O PR #7 do `launch-os` foi mesclado na `main` (merge `bb3b654`) com a CI verde e as revisões independentes registradas abaixo. **M01 fechado. O M02 NÃO está autorizado:** ele só começa com um novo início formal dela, como foi com o M00 ([[DEC-0021-m00-foundation-concluido]]).

## Evidências do fechamento (2026-10-07)
| Critério | Evidência |
|---|---|
| Repositório | `madubrandao/launch-os`, privado, `main` em `bb3b654`; 8 PRs de fase mesclados (#2 a #7) preservando os commits |
| Verificação do zero | Clone limpo da `main`, pilha isolada (`launch-os-smoke`, volumes próprios, derrubada sem sobra): PostgreSQL, Redis e n8n saudáveis; `pnpm install --frozen-lockfile`; migrations up, down (tudo) e up; sementes duas vezes (idempotente); lint, formatação, tipos, unitários, integração e `check-repo.sh`, todos verdes |
| Fumaça ponta a ponta | `scripts/smoke/m01-smoke.ts`: API e CLI de verdade contra banco descartável, **49 de 49**: bootstrap (12 credenciais; segundo recusado), login, as **14 transições** e os **12 gates** (G12 por verificação), decisão de continuidade (o agente não confirma a própria proposta; a Madu confirma pela CLI; o próximo launch nasce em IDEA), orçamento (sem limite nega; dentro autoriza; acima exige aprovação por valor), política/credenciais/entregas `dead` pela CLI, entrega ao webhook com claim e ack na ordem, audit e varredura de segredos |
| Testes | **843** (296 unitários, 547 de integração) contra PostgreSQL 17.11; 17 provas negativas; failure injection; inventário de rotas × guardrails; varredura de segredos |
| Qualidade | `docs/status/QUALITY_EVIDENCE.md`; cobertura total 92,7% (relatório, sem limite mínimo) |
| Segurança | `pnpm audit` sem vulnerabilidades; **zero** segredos no histórico dos dois repositórios; nenhum `.env`, chave ou dump jamais versionado; imutabilidade de `event` e `audit_log` em três camadas; HS256 fixo; R8 por teste; portas só em 127.0.0.1 |
| Revisões independentes | Uma por fase (F3, F4, F5, F7) mais, na F8: **conformidade** (98 exigências: 65 cumpridas, 14 com desvio registrado, 5 desvios não registrados, 7 lacunas, 7 pendentes legítimos), **segurança e operação** (sem crítico nem alto) e uma terceira, sobre o **código novo da F8** (1 alto e 3 médios, todos corrigidos). Nenhum achado crítico em nenhuma |
| Definition of Done | `docs/status/M01_DOD.md`, item por item, com workflow n8n e dashboard como N/A |
| Disco | 27 GB livres em `C:` na última medição |

## Critérios de saída do M01 (`milestones.md`)
| Critério | Situação |
|---|---|
| PostgreSQL + schema + migrations reversíveis | Cumprido (snapshot de `database/schema` e `down` com dados: pendentes da DEC-0024) |
| Redis | **Parcial:** sobe saudável, com senha, como usuário `redis`, mas a API não o usa (a spec previa "a API conecta" e um teste de "Redis fora do ar") |
| Event model e event schema | Cumprido (23 tipos, contrato estrito) |
| Audit log | Cumprido |
| Agent Registry + Permission model | Cumprido |
| Gates G01–G12 + `BUDGET_AUTHORIZED` | Cumprido, com os desvios já aceitos (G07 sem G05/G06 antes; G08 sem orçamento, M04) |
| Launch state machine (+ decisão de continuidade) | Cumprido |
| API skeleton | Cumprido, **mas sem imagem Docker da API** (ver abaixo) |

## O que mudou na F8 por causa da revisão (você escolheu "implementar agora")
A revisão de conformidade achou três canais da operadora que a especificação previa e faltavam: **política**, **credenciais** (criar, listar, revogar; a chave aparece uma vez) e **entregas `dead`** (listar e reprocessar). Implementados na API e na CLI, só com token humano e audit, com testes e na fumaça ([[spec-core-05-permissoes-e-aprovacoes]]; ADR-0022). Também entraram: logs com `correlation_id` e `execution_id`, teste de consistência de `approval_state` com oráculo independente, teto técnico (10 anos) em `approval_expiry.seconds` e criação concorrente de política sem 500.

## O que o M01 NÃO entrega (registrado, com recomendação)
| # | Lacuna | Recomendação do ENG-00 |
|---|---|---|
| A4 | **Imagem Docker da API / `docker compose` com a API saudável** (critério do plano). Hoje a API roda no host (`node apps/api/src/server.ts`), verificada pela fumaça | **Alterar o critério** para "API roda e é verificada pela fumaça"; fazer a imagem junto com a decisão de hospedagem (antes do M03) |
| A5 | **Isolamento das credenciais por workflow no n8n (D7.7)**: nunca verificado no n8n 2.41.6 | **Verificar no início do M02**, antes de guardar qualquer credencial de agente no n8n; se for impossível, o ENG-00 volta a você com alternativa |
| M1 | O n8n não consegue pedir à API que emita evento nem registrar resultado (R3/D1.5) | M02 (os payloads de domínio são pendentes) |
| M2 | `error.detected` com origem `api` ou `workflow` não tem produtor (só o da entrega) | M02 |
| M3 | `decision.recorded` leva `next_launch_id: null` (o vínculo está em `launch.created.previous_launch_id`) | Aceitar como está, ou corrigir no M02 se um consumidor precisar |
| M5 | Redis não é usado pela API | Aceitar que, no M01, "validar o Redis" significa subir com senha; decidir o papel real dele no M02 |
| — | OpenAPI não gerado (spec: PEND) | M02 ou quando houver cliente externo |

## PENDENTES que são seus (sem valor no código; a API falha ou nega, nunca assume)
- **Valores:** duração da sessão (`AUTH_SESSION_TTL_SECONDS`); intervalo e lote do agendador; prazo de aprovação (`approval_expiry`); limite de orçamento e moeda (`budget_limit`); limite de tentativas de login; os 6 valores da entrega e a validade do claim; validade das chaves de idempotência e prazo de limpeza das entregas; limite mínimo de cobertura.
- **Definições:** escopos reais dos 11 agentes (M02/M03); critérios objetivos dos gates G01 a G11 (só o G12 tem); canal de notificação; segundo fator.
- **Migration 0009** (parâmetros legíveis da aprovação de ação), **antes do M04**.

## Pendências de segurança e operação (obrigatórias antes de expor a API além de `127.0.0.1`, ou seja, antes do M03)
1. **Rate limit e bloqueio de login** (cada tentativa gasta um argon2) e **limite de taxa do audit de 401 anônimo** (o `audit_log` é imutável; um flood o infla sem como podar) — decisão registrada no DEC-0028.
2. **Retenção do `audit_log`** (partição ou arquivamento).
3. **Troca de senha da operadora e `logout`** (hoje só por SQL; o token vale até expirar).
4. **Backup e restore testados** (inclui guardar a `N8N_ENCRYPTION_KEY` fora da máquina), rotação de `JWT_SECRET` e das senhas dos papéis, rotação de logs.
5. Retirar `CREATEDB` do migrador em produção; fixar imagens por digest; escopo do `.env` do processo da API; `claimEvent`/`ackEvent` sem filtro de organização (multi-organização).
6. **Barato, recomendo já no início do M02:** exigir tamanho mínimo e recusar `CHANGE_ME` em `JWT_SECRET` e nas senhas.
7. Itens menores da revisão: validação de `http://` fora de loopback na CLI, `requireAgent` sem uso, funções de limpeza ainda fora do agendador (os prazos são seus).

## Decisão
1. **Aprovado o fechamento do M01** com as lacunas acima registradas, e **marcar os critérios de saída** do M01 no `milestones.md` (Redis como parcial, aceito).
2. **Alterar o critério A4** (API saudável no compose) como recomendado, e registrar A5 como **primeira tarefa do M02**.
3. **O M02 não está autorizado.** Ele só começa com um novo início formal seu, com plano único e as mesmas paradas.

Ver [[DEC-0028-fechamento-da-f7-do-m01]] · [[M01-plano-de-execucao]] · [[milestones]].
