---
tipo: decisao
area: 02-Decisoes
id: DEC-0031
status: aceita
fonte: M02-plano-de-execucao (F1); spec-agent-01 a 07; spike A5 (2026-10-08-spike-a5-n8n); quatro revisões independentes (spec-agent-07); resposta da Madu em 2026-10-08 ("siga com as recomendadas")
data-registro: 2026-10-08
data-decisao: 2026-10-08 (a Madu aprovou seguindo as recomendações do ENG-00)
---
# DEC-0031 — Portão da F1 do M02: especificação do Agent Engine aprovada

> **Aprovado pela Madu em 2026-10-08.** Termina a F1. Passam a valer os blocos [[spec-agent-01-gateway-e-identidade]] a [[spec-agent-07-revisao-independente]]. **Nenhum código foi escrito**; a F2 e a F3 só começam com plano único aprovado.

## Evidências da F1
- **Spike A5 (negativo)** no n8n 2.41.6 Community, em pilha isolada derrubada ao fim: não há isolamento de credencial por workflow; S2 a S8 documentados em [[2026-10-08-spike-a5-n8n]].
- **Seis blocos de especificação** e **quatro revisões independentes** (agentes diferentes do autor, somente leitura): 3 críticos e 13 altos na primeira; 7 altos novos na segunda; 2 altos na terceira; **nenhum crítico nem alto na quarta** ("pronto"). Todos os achados tratados nos blocos; mapa achado → correção no [[spec-agent-07-revisao-independente]].
- PR #8 do `launch-os` mesclado (docs/status sincronizados com o fim do M01 e o início do M02).

## O que a Madu aprovou (todas as recomendações do ENG-00)
1. **Identidade (D1.2, dela, e D1.1/D1.3/D1.6/D1.7):** o gateway fica na API; o n8n não toca o banco; **token curto por execução** preso ao agente e à execução, com **época** (um token válido por vez), `renew` com `exp ≤ deadline_at`; só a **API inicia** execuções (despacho por fila própria, gatilho por evento via assinante interno no outbox); o **LLM é chamado pela API** (adaptador).
2. **Definição do agente em arquivo no repositório** (`agent.json` + `prompt.md`), hash por execução, **o registry no banco manda** se divergir; sem `model` ou `limits` a definição não carrega.
3. **Tool registry no código**; pipeline de política em ordem fixa; **escopos `read`/`write` passam a ser aplicados** (inclusive dinâmicos); approval hook com **retomada na mesma transação da decisão**, pipeline reexecutado, `definition_changed` como custo operacional aceito; efeito externo não idempotente emitido **no máximo uma vez**.
4. **Execução:** tabelas próprias; laço **passo a passo em duas fases**; lease por `run_id`; handoff por execução filha com `awaiting_child`; despacho morto = `failed` final (refazer = nova execução); histórico do n8n em `none`; `deadline_at` único; critério "n8n emitindo evento" por mecanismo (`event.emit`) + teste.
5. **Contexto e memória:** montagem determinística na API; entrada externa sempre como dado marcado; memória **só de inserção**, com origem; **sem texto livre** (chaves de lista fechada com schema do valor); **sem dado pessoal** no M02; semântica só a interface; **`objection` e `partner` fora da lista de assuntos** até a Madu confirmar que não carregam dado pessoal.
6. **Código:** `packages/agents` (núcleo puro + pg), adaptador de LLM por `fetch`, **nenhuma dependência nova**, migrations **0009 a 0013**, teste do n8n **local** no M02.
7. **Confirmado:** num `llm_call` que o varredor marca `failed`, a **reserva de custo continua contando**.
8. **Riscos aceitos:** credencial de serviço do n8n pode derrubar execuções alheias; efeito externo não é recolhido por cancelamento posterior; delegação confusa mãe→filha; leitura de memória gravada por terceiros; `write` da entidade-assunto sobrecarrega o significado de `write`.
9. **Mudanças no código do M01** listadas no bloco 6, seção 5-ter (`context.ts`, `agents.ts`, `app.ts`, `approvals.ts`, `policy.ts`, `config.ts`, `packages/schemas`, `packages/events`, CLI), cada uma com teste.

## Mudança no plano do M02
O [[M02-plano-de-execucao]] fica ajustado: **D8** (retomada na transação da decisão, não pelo assinante `approval-resume`), **D5** (estados com `dispatched`, `awaiting_child`, `timed_out` e a época), **D1** ("credencial do agente" vira token por execução), **F3** (`packages/memory` reservado e vazio).

## PENDENTES (da Madu; nenhum ganha padrão no código)
Valores de token, prazos, limites, concorrência (obrigatória: sem ela a API não sobe), timeouts, tamanhos e retenção; **chave Anthropic, limite de custo (moeda, unidade, escopo), preço por token e modelo por agente**; `approval_expiry`; escopos e prompts reais dos agentes (M03); infraestrutura antes do M03 (hospedagem do n8n, Redis, imagem Docker da API, rate limit, retenção do `audit_log`, troca de senha, backup). Lista completa: [[spec-agent-07-revisao-independente]], seção 5.

## Próximos passos
1. **PR de ADRs no `launch-os`** (só documentação, ADR-0023 em diante) e sincronização de `docs/status`.
2. **Plano único da F2 (banco) e da F3 (contratos e núcleo)**, que podem andar em paralelo; nada de código antes dele.

Ver [[DEC-0030-inicio-formal-do-m02]] · [[M02-plano-de-execucao]] · [[milestones]].
