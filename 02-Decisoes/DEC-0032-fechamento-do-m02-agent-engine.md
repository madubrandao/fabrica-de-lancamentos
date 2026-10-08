---
tipo: decisao
area: 02-Decisoes
id: DEC-0032
status: aceita
fonte: M02-plano-de-execucao (F6 e F7); plano único da F6 e da F7 aprovado em 2026-10-08 ("Plano aprovado, finalize a m02"); docs/status/M02_DOD.md, M02_QUALITY_EVIDENCE.md e docs/operations/M02-RUNBOOK.md no launch-os; revisões independentes completas de 2026-10-08; decisões do item 1 aprovadas pela Madu em 2026-10-08 ("Concordo com as recomendações do item 1, siga")
data-registro: 2026-10-08
data-decisao: 2026-10-08
---
# DEC-0032 — Fechamento do M02 Agent Engine

> **ACEITA pela Madu em 2026-10-08.** O M02 está **implementado, verificado do zero e fechado formalmente**, com as lacunas L1 a L7 e as BAIXAS abaixo registradas. **O M03 NÃO está autorizado:** só com novo início formal, plano único e as mesmas paradas. Antes dele, as lacunas são tratadas na ordem da seção "Decisão" (com um plano único por fase).

## Evidências do fechamento (2026-10-08)
| Critério | Evidência |
|---|---|
| Repositório | `madubrandao/launch-os`, privado, `main` em `2ef4f72`; PRs do M02 mesclados preservando os commits: #8 e #12 (status), #9 (ADRs), #10 (banco), #11 (núcleo), #13 a #17 (gateway), #18 (status), #19 e #20 (n8n), #21 (status), #22 (qualidade), #23 (fechamento) |
| Verificação do zero | Clone limpo da `main` (`9db7145`), pilha isolada recriada com volumes novos (`launch-os-smoke`): `pnpm install --frozen-lockfile`, tipos, lint, formatação, `check-repo.sh`, `pnpm audit --prod` (sem vulnerabilidades), migrations `up`/`down`/`up` no `global-setup`, **1255 testes** e **fumaça 29 de 29**. Depois das correções do fechamento: **1266 testes** (515 unitários, 751 de integração) e fumaça 29/29. Pilha derrubada sem sobra |
| Fumaça ponta a ponta | `scripts/smoke/m02-smoke.ts` contra o **n8n 2.41.6 real**: caminho feliz, ferramenta interna e externa, aprovação no meio, retry, timeout, duplicado, falha parcial, falha da API, entrada inválida (com controle positivo), rollback, modelo lento, época morta, cancelada com aprovação pendente, rejeitada e vencida; o n8n não guarda histórico nem token; token ausente de logs e de 9 tabelas |
| Qualidade | `docs/status/M02_QUALITY_EVIDENCE.md`: varredura de segredos do Agent Engine (chave do LLM, prompt, contexto; 30 testes), failure injection do gateway, **11 provas negativas por mutação (P18 a P28)** mais a P29 do defeito achado no fechamento; cobertura 93,4% instruções e 89,5% ramos (relatório, sem limite mínimo) |
| Revisões independentes | Uma por PR de fase e, no fechamento, **duas revisões completas** sobre o clone limpo: **conformidade** (cerca de 133 exigências das especificações 01 a 06 e do plano: 0 altas, 5 médias, 8 baixas) e **segurança e operação** (0 críticos, 0 altos, 6 médios, 9 baixos). Nenhum achado crítico ou alto em nenhuma |
| Definition of Done | `docs/status/M02_DOD.md`, item por item, com dashboard como N/A |
| Disco | Pilha isolada derrubada; sem volumes `smoke` |

## Critérios de saída do M02 (plano)
| Critério | Situação |
|---|---|
| Agent definition schema e carga de prompt | Cumprido (`agent.json` + `prompt.md`, hash na execução, falha de carga impede a subida) |
| Tool registry com escopo explícito | Cumprido (pipeline de política em 8 passos) |
| Memory read/write com origem e timestamp | Cumprido na gravação (`memory_item`, só inserção); a leitura devolve versão e valor (a origem e o timestamp ficam na gravação, BAIXA registrada) |
| Execution state com `execution_id`, audit, retries e timeouts | Cumprido |
| Agent routing, approval hooks, policy enforcement | Cumprido (handoff por execução filha, gatilho por evento, aprovação com hash e retomada na transação da decisão) |
| A5 resolvido | **Negativo e mitigado:** o n8n Community não isola credencial por workflow; a fronteira é a API (token curto por execução, época, pipeline). Aprovado no DEC-0031 |
| n8n emitindo evento pela API | Cumprido (`event.emit`, só tipos da lista permitida; vazia em produção) |
| `error.detected` de origem `api` e `workflow` | Cumprido |
| Papel do Redis (M5 do M01) | **Sem papel no M02** (a fila usa o PostgreSQL); volta com o modo fila do n8n na hospedagem |
| `next_launch_id` (M3 do M01) | **Fora do M02:** não há consumidor; o vínculo continua em `launch.created.previous_launch_id` |

## O que mudou no fechamento por causa das revisões (corrigido, com teste)
`renew_used` só em execução não final (era 500, agora 409); `seq` limitado a int32 (era 500); `N8N_WEBHOOK_SECRET` com piso de 32 caracteres, ASCII visível e sem `CHANGE_ME`; a subida recusa `AGENT_STEP_STALE_SECONDS` ou `AGENT_LLM_CALL_STALE_SECONDS` menores que `LLM_TIMEOUT_MS` com o LLM ligado; varredores rodam antes do despachante. Também na F6: `approval-resume` do M01 passou a ignorar aprovações `tool.*`.

## O que o M02 NÃO entrega (registrado, com recomendação)
**Antes do primeiro agente real ou de qualquer ferramenta com efeito (M03):**
| # | Lacuna (achado) | Recomendação do ENG-00 |
|---|---|---|
| L1 | O workflow do n8n **não chama `/renew`** (conformidade, MÉDIA). Execução mais longa que o TTL do token morre com 401 | Implementar o nó de renew no `execute-agent` (usa `token_exp` do webhook, margem, resposta perdida) com cenário na fumaça. Alternativa barata e provisória: manter TTL ≥ `deadline_seconds` (já no runbook) |
| L2 | **Ferramenta externa não idempotente:** `outcome_unknown` existe no banco mas nunca é gravado; um `step` repetido reemite a diretiva (conformidade, MÉDIA; vira ALTA com a primeira ferramenta com efeito) | Implementar antes da primeira ferramenta externa com efeito: marcar `outcome_unknown` na retomada e não reemitir, com teste |
| L3 | **Ciclo de eventos do próprio gateway:** um agente com `triggers:["error.detected"]` (ou `approval.*`) gera execução que falha e emite o mesmo evento, sem passar pelo teto por raiz (segurança, MÉDIA) | Recusar na subida `triggers[]` com eventos que o gateway produz, até haver herança de raiz para eles |
| L4 | **Reserva de custo nunca é liberada** quando o provedor falha sem cobrar (chave revogada, 429): o teto acumulado se esgota (segurança, MÉDIA) | Liberar a reserva em `invalid_request` e `rate_limited`; manter em `timeout` e `provider_error` (podem ter cobrado) e na varredura. **Decisão sua**, porque mexe na regra fail-closed do custo |
| L5 | O `fail` da credencial de serviço **não confere época nem `run_id`**: um workflow de época antiga que vê um 5xx pode falhar a execução que a época nova conduz (segurança, MÉDIA) | O workflow enviar o `run_id` e a API recusar (409) se diferente do `lease_run_id` |
| L6 | `PUT /agents/:código/permissions` não confere `tool.<nome>` contra o registry (conformidade, MÉDIA; falha fechada) | Validar contra o registry no PUT e logar aviso na carga, junto dos escopos reais do M03 |
| L7 | Infraestrutura: **comando de importação** dos workflows (hoje só a fumaça chama), hospedagem do n8n com **TLS**, imagem Docker da API, rate limit, retenção do `audit_log`, troca de senha, backup | Junto da decisão de hospedagem, antes do M03 |

**BAIXAS das duas revisões (backlog, sem urgência):** disparo imediato do despachante depois do commit (hoje só polling); "401 não audita" da spec contra o M01 (que audita 401/403); teto de tamanho do prompt (valor seu); estado do launch no contexto como linha completa; lista `omitted` do contexto não persistida; `kind` fixo por chave de memória e origem/timestamp na leitura; teste de `tool.removida`; teste de esgotamento do despacho `resume`; marcação de `x-execution-id` não verificado no audit; `trigger_ref` do handoff com o `call_id` do modelo (usar `execution.id:callId`); write-once de `action_params_hash` e `status` da aprovação; `costUsed` soma a vida inteira (janela por período); `launch_id` sem validação na criação; negações anônimas 401/403 sem limite de taxa (já no DEC-0028); `NODE_ENV=test` liga as ferramentas `test.*` (controle processual); descrição de `event.emit` cita `AGENT_EMITTABLE_EVENTS`, que não existe.

## PENDENTES que são seus (sem valor no código; a API falha ou nega, nunca assume)
- **Valores:** duração do token de execução; despacho (lote, tentativas, espera, timeout); `AGENT_START_GRACE_SECONDS`, `AGENT_STEP_STALE_SECONDS`, `AGENT_LLM_CALL_STALE_SECONDS`; concorrência (global e por agente); tamanho máximo de payload; execuções por raiz; entrega de eventos (`DELIVERY_*`); `approval_expiry`; **limites de cada agente** (`deadline_seconds`, passos, ferramentas, saída, contexto, profundidade de handoff, filhas); timeout de cada ferramenta; **meta mínima de cobertura**; retenção de execução, passos, chamadas e memória.
- **LLM:** a chave Anthropic, `LLM_TIMEOUT_MS`, o limite de custo (`llm_cost_limit`: teto acumulado da organização), o preço por token (`llm_pricing`) e o modelo de cada agente. Sem eles, só o adaptador de teste.
- **Definições:** escopos reais dos 11 agentes e seus prompts; ferramentas de negócio (M03/M04).
- **Antes do M03:** a hospedagem do n8n e as URLs reais dos webhooks (itens L7).

## Decisão (aceita pela Madu em 2026-10-08)
1. **Fechamento do M02 aceito**, com as lacunas L1 a L7 e as BAIXAS acima registradas. Critérios de saída marcados no `milestones.md`.
2. **Registrado:** A5 negativo e mitigado pelo token curto; `next_launch_id` fora do M02; Redis sem papel no M02.
3. **L4 decidida:** liberar a reserva de custo em `invalid_request` e `rate_limited` (erro sem cobrança); manter a reserva em `timeout` e `provider_error` (podem ter cobrado) e na varredura.
4. **Ordem das lacunas antes do M03 aprovada:**
   1. L3: recusar na subida `triggers[]` com eventos que o gateway produz.
   2. L6: validar `tool.<nome>` contra o registry no PUT de permissões.
   3. L5: o workflow envia `run_id`; a API responde 409 se for diferente de `lease_run_id`.
   4. L1: nó de renew no `execute-agent`, com cenário na fumaça (margem é valor da Madu; marcador até lá).
   5. L2: marcar `outcome_unknown` na retomada e não reemitir a diretiva; obrigatória antes da primeira ferramenta externa com efeito.
   6. L4: implementar a liberação de reserva conforme a decisão do item 3.
   7. L7: comando de importação dos workflows (pode sair antes); TLS, imagem da API, rate limit, retenção, troca de senha e backup junto da hospedagem.
   Cada item vira um plano único aprovado pela Madu, com PR, CI verde e revisão independente registrada.
5. **O M03 não está autorizado.** Ele só começa com novo início formal seu, com plano único e as mesmas paradas (chamada real a LLM só com a chave e o limite de custo definidos por você).
6. **Não autorizado nesta decisão:** push do commit do vault `04b2753` e qualquer alteração no `launch-os`.
