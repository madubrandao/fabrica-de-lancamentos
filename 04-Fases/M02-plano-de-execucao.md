---
tipo: fase
area: 04-Fases
id: M02-plano-de-execucao
status: em execução (início formal em 2026-10-08, DEC-0030)
fonte: plano aprovado pela Madu em 2026-10-08 ("PODE PLANEJAR M02"); milestones; DEC-0020; DEC-0029; blueprint v1 §36
data-registro: 2026-10-08
---
# M02 — Agent Engine: plano de execução

> **Iniciado formalmente em 2026-10-08** ([[DEC-0030-inicio-formal-do-m02]]), como o M01 ([[DEC-0022-inicio-formal-do-m01]]). O M01 está fechado ([[DEC-0029-fechamento-do-m01-core-platform]]). Cada fase tem um plano único aprovado pela Madu; nenhum código antes de ela aprovar a especificação (F1).

## O que o M02 é
O motor que **executa agentes**: Agent Contract, definição e carga de prompts, tool registry com escopo explícito, montagem de contexto, interfaces de memória (origem e timestamp), ciclo de vida da execução (`execution_id`, estado, audit, retry, timeout), roteamento/handoff, approval hooks e policy enforcement. **Não cria os agentes de negócio** (A0–A10 são do M03 em diante). Por que há portão de especificação: o [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]] deixou PENDENTES o que fica no código e o que fica no n8n, o papel do *agent runtime gateway* e a convivência do adaptador de LLM com os nós de IA.

**Fora de escopo:** agentes de negócio e os 144 workflows (M03/M04), dashboard (M06), vector store (só a interface), integrações externas reais, hospedagem de produção, aprendizado (M05).

## Pendências herdadas do M01
A5 (isolamento de credenciais por workflow no n8n 2.41.6, D7.7; **primeira tarefa da F1**, em pilha isolada) · M1 (o n8n pede à API que emita evento e registre resultado; F4) · M2 (`error.detected` de origem `api` e `workflow`; F4) · M3 (`next_launch_id`; só se houver consumidor) · M5 (papel do Redis; F1 decide) · `JWT_SECRET` com tamanho mínimo e sem `CHANGE_ME` (F4) · migration 0009, parâmetros legíveis da aprovação de ação (F2). **Não entram** (obrigatórias antes de expor a API): rate limit, retenção do `audit_log`, troca de senha, backup.

## Decisões que a F1 propõe (recomendação do ENG-00; a Madu aprova)
| # | Pergunta | Recomendação |
|---|---|---|
| D1 | Onde rodam registry, contexto, ferramentas, memória e estado | No **gateway da API**; o n8n nunca toca o banco e chama o gateway com a credencial do agente |
| D2 | Quem chama o LLM | **Adaptador na API** (Anthropic primeiro): a chave fica fora do credential store do n8n, o limite de custo é duro, o audit é central. Alternativa: nós de IA do n8n |
| D3 | Onde mora a definição do agente | **Arquivos versionados no repositório** com schema Zod, revisados por PR; hash da definição gravado em cada execução |
| D4 | Tool registry | Nome, schemas, entidades lidas/escritas, efeito externo, `requires_approval`, custo; internas na API, externas no n8n; ações no registro `AGENT_ACTIONS`; M02 entrega o mecanismo + ferramentas de teste |
| D5 | Estado da execução | Tabelas próprias (execução, chamada de ferramenta, chamada ao LLM) fora do event store; estados: criada, em execução, aguardando aprovação, concluída, falhou, expirou, cancelada |
| D6 | Contexto | Determinístico, na API, filtrado pelos escopos de leitura, com teto de tamanho, entrada externa marcada não confiável, hash gravado |
| D7 | Memória | Interface de leitura/escrita com origem e timestamp; mínimo em PostgreSQL; semântica só a interface |
| D8 | Aprovação no meio | Ferramenta com `requires_approval` cria aprovação de ação (hash dos parâmetros); execução em `aguardando aprovação`; retoma pelo *approval-resume*; só com o mesmo hash |
| D9 | Handoff | Execução filha (`parent_execution_id`), sem entidade de negócio nova |
| D10 | Custo de LLM e modelo por agente | **Valores da Madu**; sem limite, a chamada ao LLM é negada (fail-closed) |
| D11 | Injeção de prompt | Entrada externa nunca muda permissão, aprovação nem ferramenta; testes de escalada por mensagem desde a F3 |

## Fases (uma aprovação de plano por fase; ordem F0 → F1 → F2 ∥ F3 → F4 → F5 → F6 → F7)
| Fase | Dono | Entrega | Verificação |
|---|---|---|---|
| F0 Início formal | Madu e ENG-00 | DEC-0030; pré-condições (disco ≥ 20 GB, RAM, Docker, containers de outros projetos parados) | `docs/status/` atualizado |
| F1 Especificação (portão) | ENG-00 | Spike A5; blocos 1 a 6 da especificação; decisões D1–D11 | Madu aprova; revisão independente da especificação |
| F2 Banco e schema | ENG-02 | Migrations reversíveis (0009, execução, ferramentas, LLM, memória) | Round-trip, constraints, privilégios |
| F3 Contratos e núcleo | ENG-04 | `packages/agents`, `packages/memory`, adaptador de LLM e implementação de teste; tudo puro e testável | Unitários; ferramentas × escopos; escalada por mensagem |
| F4 Gateway na API | ENG-01 e ENG-04 | Rotas de execução, ferramenta, memória, aprovação no meio, `error.detected`, emissão de evento | Inventário de rotas; matriz de integração; completude do audit |
| F5 n8n | ENG-03 | Subworkflow `execute-agent` e um agente de referência sem negócio | Blueprint §31: caminho feliz, retry, timeout, duplicado, falha parcial, falha da API, entrada inválida, rollback |
| F6 Qualidade | ENG-06 | Failure injection, varredura de segredos (chave do LLM, prompts, contexto), provas negativas, cobertura | Evidências |
| F7 Fechamento | Madu aprova | Verificação do zero, fumaça do M02, revisão completa, DoD, runbook, DEC de fechamento (o M03 não fica autorizado) | Mesmo rigor do M01 |

## Paradas
1. Docker e RAM. 2. **Chamada real a um LLM só com a chave Anthropic e um limite de custo definidos pela Madu**; sem isso, adaptador de teste. 3. Valores (custo, modelo por agente, timeouts, retries, limites de contexto) e escopos reais dos agentes: PENDENTES, sem padrão no código. 4. Se o A5 mostrar que o isolamento é impossível, o ENG-00 volta com alternativa. 5. Aprovação de D1–D11 (F1) e do fechamento (F7).

## Riscos
A5 negativo · onde roda o laço do agente (API: controle e custo duro, mas API maior; n8n: mais simples, enforcement por confiança) · deriva entre definição em arquivo e registry no banco (hash e teste de consistência) · injeção de prompt e custo (D10, D11, F6) · escopo inflando para negócio.

## Critérios de saída (propostos)
Agent definition schema · Prompt loading · Tool registry com escopo explícito · Memory read/write com origem e timestamp · Execution state com `execution_id` e audit trail, retries e timeouts · Agent routing, approval hooks e policy enforcement · **mais** A5 resolvido, o n8n emitindo evento pela API e `error.detected` de origem `api` e `workflow`.

Ver [[milestones]] · [[roadmap]] · [[M01-plano-de-execucao]] · [[DEC-0030-inicio-formal-do-m02]].


## Andamento
- **F0** concluída em 2026-10-08: DEC-0030 aceito; `docs/status` do `launch-os` sincronizados (PR #8 mesclado).
- **F1, spike A5 (2026-10-08): NEGATIVO.** O n8n 2.41.6 Community não isola credencial por workflow (projetos e compartilhamento exigem licença; o workflow do agente B usou a credencial do agente A). Evidências S1 a S8 em [[2026-10-08-spike-a5-n8n]]. **Parada nº 4 do plano:** o ENG-00 propõe a execução iniciada pela API com token curto por execução e **aguarda a decisão da Madu** antes de escrever os blocos da especificação.

- **F1 concluída em 2026-10-08:** especificação (blocos 1 a 7) aprovada pela Madu ([[DEC-0031-portao-f1-especificacao-do-agent-engine]]); quatro revisões independentes; decisões D1–D6 conforme o bloco 7. Ajustes ao plano: D8 (retomada na transação da decisão), D5 (estados e época), D1 (token por execução), F3 (`packages/memory` vazio). **Próximo:** PR de ADRs no `launch-os` e plano único da F2 e da F3.
