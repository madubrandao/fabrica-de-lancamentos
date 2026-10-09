---
tipo: fase
area: 04-Fases
status: proposto (aguarda a aprovação da Madu)
fonte: DEC-0032 (L3, L6, L5 e emenda R4); DEC-0034 (fase B); leitura do código do launch-os em 2026-10-09
atualizado: 2026-10-09
---
# Fase B antes do M03: L3, L6 e L5 (plano único, PROPOSTO)

Ordem aprovada no DEC-0032: L3 → L6 → L5. **Uma aprovação só para os três.** Sem LLM real, sem agente de negócio, sem dependência nova, sem migration.

## Evidências lidas no código
- **Gatilho por evento** (`apps/api/src/gateway/triggers.ts`): a raiz só é herdada se existir audit `event.emit` com o `execution_id` (`emitterOf`, linhas 31-40). Os outros eventos viram raiz nova e não contam no teto por raiz.
- **Quem causou o evento:** todo evento causado por agente grava `actor_type = 'agent'` e o código do agente, tanto pela credencial `lo_` quanto pelo token de execução (`apps/api/src/context.ts:41-43`). O `execution_id` só fica no audit (`common.ts:107`).
- **Eventos que o próprio gateway produz:** `error.detected` (`executions.ts:270`) e `approval.requested`, `approval.granted`, `approval.rejected` e `approval.expired` das aprovações de ferramenta (`approvals.ts`).
- **PUT de permissões** (`apps/api/src/agents.ts:85-117`): grava `tool.<nome>` sem conferir o registry. A ferramenta desconhecida nunca executa (falha fechada), mas o erro de digitação passa calado.
- **`fail` do serviço** (`apps/api/src/gateway/routes.ts:357-394`): aceita `dispatched` e `running` sem conferir o `lease_run_id`. O workflow já manda `run_id: $execution.id` no `start` (`scripts/n8n/build.ts:229`), mas não manda no nó "Falha do serviço" (`build.ts:390-395`).

## B1. L3: ciclo de eventos (com o R4)
**Recomendação: duas travas, as duas fail-closed.**
1. **Na subida:** recusar definição com `triggers[]` contendo evento que o gateway produz (`error.detected`, `approval.*`), com mensagem listando agente e evento. É a correção do DEC-0032.
2. **No gatilho (R4):** se o evento tem `actor_type = 'agent'`, procurar a execução causadora no audit da mesma organização (`event_id` do evento com `execution_id` preenchido, não só `event.emit`). Achou: herda a raiz e conta no teto, como hoje. **Não achou** (por exemplo, evento criado com a credencial `lo_` de um agente, fora de execução): não dispara, grava audit `denied` com `untraced_agent_event` e confirma o evento. Agente chamando agente continua possível pelo caminho oficial, o handoff.
- **Tarefa de verificação dentro da B1:** garantir que toda emissão feita com contexto de execução grave o audit com `event_id` e `execution_id`. Onde não gravar, corrigir, com teste.
- **Alternativa (não recomendada):** só a trava 1. É mais simples, mas deixa o R4 aberto, e o único freio fica sendo o custo e a concorrência.

Testes: na subida (cada evento do gateway recusado); no gatilho (evento de execução herda a raiz; evento de credencial `lo_` é negado e confirmado; evento humano continua criando raiz nova); teto por raiz alcançado pelo novo caminho.

## B2. L6: PUT de permissões confere o registry
- O `PUT /agents/:código/permissions` recusa com 400 `unknown_tool` (listando os nomes) qualquer `tool.<nome>` que o registry não conhece. Não sobe a versão e grava audit `denied`.
- Na subida, aviso no log (sem segredo) para permissões já gravadas no banco com ferramenta desconhecida. Só aviso: a execução já falha fechada.

Testes: ferramenta conhecida aceita; desconhecida recusada sem mudar a versão; aviso na subida.

## B3. L5: `fail` do serviço confere o `run_id`
- O nó "Falha do serviço" passa a enviar `run_id: $execution.id`, igual ao `start`.
- A API exige `run_id` no `fail` do serviço e responde 409 `lease_mismatch` se for diferente do `lease_run_id`. Execução sem lease também é recusada, porque o workflow nunca chama `fail` antes do `start` (`build.ts:402-403`).
- Os JSONs são regenerados por `scripts/n8n/build.ts` (nunca editados à mão) e o contrato unitário é atualizado.

Testes: integração (run_id certo falha a execução; run_id de época antiga recebe 409 e a execução segue; sem lease recebe 409); contrato do JSON (o nó envia `run_id`).

## Entrega
- Três PRs pequenos, na ordem: `feat/m03-pre-b1-l3`, `feat/m03-pre-b2-l6`, `feat/m03-pre-b3-l5`.
- Em cada PR: tipos, lint, formatação, `check-repo.sh` depois do `git add`, CI verde e revisão independente registrada antes do merge.
- **Merge:** feito por você, ou por mim com a sua autorização explícita naquele momento (o classificador do modo automático exige isso).
- **Fumaça do M02 contra o n8n real:** uma vez, no fim da B3, porque o workflow muda. Exige subir a pilha isolada, então você fecha os programas pesados antes. Os testes de integração rodam na CI.
- No fim da fase: `docs/status`, runbook e vault atualizados.

## Paradas
1. Se a trava 2 da L3 quebrar algum teste ou cenário existente que dependa de evento de agente fora de execução, volto a você antes de seguir.
2. Memória insuficiente para a fumaça: paro e peço uma janela.
3. Qualquer divergência das especificações ou dos ADRs, ou necessidade de migration ou dependência nova.

Ver [[DEC-0032-fechamento-do-m02-agent-engine]] · [[DEC-0034-perfil-de-valores-e-plano-ate-o-m03]].
