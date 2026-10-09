---
tipo: decisao
area: 02-Decisoes
id: DEC-0034
status: aceita
fonte: DEC-0033 (lista de decisões exclusivas da Madu); apps/api/src/config.ts; apps/api/src/policy.ts; packages/events/src/config.ts; packages/agents/src/memory.ts; scripts/smoke/m01-smoke.ts e m02-smoke.ts (valores de teste já exercitados)
data-registro: 2026-10-09
data-decisao: 2026-10-09 (a Madu aceitou em bloco: "Siga com as recomendaras")
---
# DEC-0034 — Perfil de valores e plano até o M03

> **Aceita pela Madu em 2026-10-09**, depois de pedir "traga a melhor opção". O perfil local vale para o desenvolvimento; os valores de produção saem no plano da hospedagem. O M03 continua **não autorizado** até o início formal (fase E). Os valores **não entram no código** como padrão (DEC-0016 D8): vão no `.env` local (pelo bootstrap) e nas políticas, gravadas pela API com audit.

## 1. Princípio
- **Para continuar a implementação (L1 a L7), nenhum valor da Madu é necessário:** os testes e as fumaças usam valores de teste.
- Os valores passam a ser necessários quando a API rodar de verdade com o n8n e o LLM, no M03. Por isso a proposta tem dois perfis: **desenvolvimento local** (aceitar agora) e **produção** (decidir no plano da hospedagem).

## 2. Perfil de desenvolvimento local (proposto)
Critério: menor risco. A máquina tem pouca RAM (cerca de 90% em uso), há uma única operadora e só se acessa `127.0.0.1`.

| Variável | Valor | Por quê |
|---|---|---|
| `AUTH_SESSION_TTL_SECONDS` | 3600 (1 h) | Ainda não há `logout` nem troca de senha: token curto limita o dano. Já testado nas fumaças |
| `EXECUTION_TOKEN_TTL_SECONDS` | 300 (5 min) | Testado na fumaça do M02. Execução longa depende da L1 (`/renew`), que vem antes do M03 |
| `SCHEDULER_INTERVAL_MS` / `SCHEDULER_BATCH_SIZE` | 5000 / 50 | 5 s de atraso é aceitável para vencer aprovação e entregar evento; lote testado |
| `DELIVERY_MAX_ATTEMPTS` | 5 | Cobre queda curta do n8n |
| `DELIVERY_BACKOFF_BASE_MS` / `_MAX_MS` | 2000 / 60000 | Espera crescente até 1 min |
| `DELIVERY_REQUEST_TIMEOUT_MS` | 10000 | O webhook responde ao receber |
| `DELIVERY_CLAIM_VALIDITY_MS` | 60000 | Maior que o tempo limite (regra do código) e dá tempo ao workflow começar |
| `DELIVERY_BATCH_SIZE` | 20 | Testado |
| `N8N_WEBHOOK_URL_AGENT` | `http://127.0.0.1:5680/webhook` | Porta do n8n local (`.env.example`); http só é aceito em localhost |
| `AGENT_DISPATCH_BATCH_SIZE` / `_MAX_ATTEMPTS` | 10 / 3 | Testado |
| `AGENT_DISPATCH_RETRY_DELAY_SECONDS` / `_TIMEOUT_MS` | 30 / 10000 | O webhook responde ao receber; 30 s evita martelar o n8n parado |
| `AGENT_START_GRACE_SECONDS` | 60 | Despachada e não iniciada em 1 min: redespacha |
| `LLM_TIMEOUT_MS` | 120000 (2 min) | Respostas longas de conteúdo |
| `AGENT_STEP_STALE_SECONDS` / `AGENT_LLM_CALL_STALE_SECONDS` | 300 / 300 | Têm de ser maiores que `LLM_TIMEOUT_MS` (o código recusa subir se não forem) |
| `AGENT_MAX_CONCURRENT_EXECUTIONS` / `_PER_AGENT` | 3 / 2 | RAM da máquina |
| `AGENT_MAX_PAYLOAD_BYTES` | 65536 (64 KiB) | Os testes usaram 4 a 8 KiB, pouco para texto de lançamento |
| `AGENT_MAX_EXECUTIONS_PER_ROOT` | 10 | Testado; corta ciclos |
| Marcadores do n8n: `__API_BASE_URL__` | `http://host.docker.internal:3000` | O n8n roda no Docker; a API, na máquina |
| `__HTTP_TIMEOUT_MS__` / `__HTTP_MAX_TRIES__` / `__HTTP_WAIT_MS__` / `__STEP_WAIT_S__` | 150000 / 3 / 2000 / 5 | O tempo limite precisa passar o do LLM, porque pela leitura de `step.ts` o LLM é chamado dentro do `step` (a confirmar por teste na fase A). `__STEP_WAIT_S__` ≤ 60 |

## 3. Políticas (gravadas pela API, com audit)
| Política | Valor | Por quê |
|---|---|---|
| `approval_expiry` | 259200 s (72 h) | Uma só aprovadora; cobre um fim de semana |
| `budget_limit` | `amount: 0`, moeda BRL (convenção registrada; o schema não tem campo de moeda) | Todo gasto exige aprovação da Madu até a política de orçamento do M04 (DEC-0017 C4) |
| `llm_cost_limit` | US$ 20 **acumulados** da organização (o código trata como teto total, reservas incluídas) | Basta para o M03 com o produto fictício; subir só por decisão |
| `llm_pricing` | Preço por token do modelo escolhido, copiado da tabela oficial da Anthropic no dia da configuração (preço por milhão ÷ 1.000.000) | Não inventar preço |
| Modelo | Um só para os agentes do M03: `claude-sonnet-5-5` | Um preço só; trocar por agente depois de medir custo e qualidade |
| Chave Anthropic | A Madu cria num workspace próprio do Console, com limite de gasto no próprio Console se a conta permitir | Segunda barreira, além do `llm_cost_limit` |

## 4. Demais decisões
| Pendência | Recomendação |
|---|---|
| Gates G01 a G12 | Todos por **aprovação humana da Madu**. Critérios objetivos escritos por agente dentro do M03 (G01 a G07, até `LAUNCH_READY`); G08 a G12 no M04/M05. O agente entrega um checklist e o gate exige todos os itens e a aprovação dela |
| Escopos dos agentes e chaves de memória | Definidos na especificação de cada agente do M03 (mínimo privilégio); eu proponho, a Madu aprova. Até lá, `memory.write` não grava nada, por desenho (`packages/agents/src/memory.ts:39`) |
| Limite de login | 5 falhas em 15 min bloqueiam por 15 min, com audit |
| Segundo fator | TOTP (aplicativo autenticador), sem dependência externa |
| Canal de notificação | E-mail pelo n8n (assinante `N8N_WEBHOOK_URL_NOTIFY`) para aprovação pendente e erro |
| Retenção do `audit_log` | Partição mensal; 24 meses no banco; partição mais antiga exportada para backup antes de ser destacada; nada é apagado sem decisão |
| Limpeza | Chaves de idempotência: 7 dias. Entregas tratadas: 30 dias. As funções existem (`packages/events/src/maintenance.ts`), mas o agendador ainda não as chama |
| Meta de cobertura | Piso de 85% de linhas e 80% de ramos na CI, abaixo do medido (93%/89%, número a reproduzir na `main`): falha só com regressão real |
| `N8N_ENCRYPTION_KEY` | Guardada no gerenciador de senhas da Madu, com uma cópia offline; restore testado |
| Hospedagem do n8n, TLS e URL | **Mudança proposta:** passar a L7 (hospedagem, TLS, imagem Docker da API, segundo fator, rate limit, notificação) de "antes do M03" para **"antes do M04"**. O M03 roda local, com integrações simuladas (DEC-0017 C5). A decisão de hospedagem sai no plano da L7, com 2 ou 3 opções orçadas |
| Emenda do DEC-0032 | Registrar os 5 pontos médios da revisão. O ponto 3 (`memory.write`) deixa de ser defeito: é desenho, e as chaves vão para o M03 |
| Node 24.21.0 | Eu rodo o `winget` com a versão exata e a Madu aceita o pedido de elevação do Windows; alternativa: instalador oficial |

## 5. Plano até o início do M03
| Fase | Conteúdo | Depende de |
|---|---|---|
| A. Saneamento (1 PR) | Fumaça do M01 com o ambiente completo; teste da subida do `server.ts`; confirmar que o LLM roda dentro do `step`; comentário errado em `step.ts:410`; `.env.example` com o perfil local comentado; fumaças M01 (49) e M02 (29) de novo na pilha isolada; Node 24.21.0 | Aceite deste DEC; Docker |
| B. L3, L6, L5 (1 plano) | Ciclo de eventos do gateway; PUT de permissões validando `tool.*`; conferir época no `fail` do serviço | Plano já descrito |
| C. L1, L2, L4 | Workflow chama `/renew`; `outcome_unknown` em ferramenta externa não idempotente; liberar reserva em `invalid_request` e `rate_limited` | B |
| D. Operação local | Comando de importação dos workflows (parte da L7 que fica antes do M03); partição do `audit_log`; limpeza agendada; backup e restore testados (com a `N8N_ENCRYPTION_KEY`) | C |
| E. Início formal do M03 | Chave, `llm_cost_limit`, `llm_pricing`, produto fictício; A0 a A4 e A6 mínimo, com escopos, memória e critérios dos gates G01 a G07 | Madu |
| F. Antes do M04 (L7) | Hospedagem com TLS, imagem da API, digest, `CREATEDB`, rate limit, `logout` e troca de senha, segundo fator, notificação | Decisão de hospedagem |

Cada fase: branch própria, `check-repo.sh` depois do `git add`, CI verde e revisão independente registrada no PR antes do merge. Sem chamada real a LLM antes da fase E.

Ver [[DEC-0033-decisoes-pendentes-do-m01]] · [[DEC-0032-fechamento-do-m02-agent-engine]] · [[DEC-0017-modelo-de-negocio-inicial]] · [[gates]].
