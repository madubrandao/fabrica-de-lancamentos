---
tipo: decisao
area: 02-Decisoes
id: DEC-0025
status: aceita
fonte: M01-plano-de-execucao (F3); plano único da F3 aprovado em 2026-10-07; spec-core-04-eventos; ADR-0018 no launch-os; evidências de 2026-10-07
data-registro: 2026-10-07
data-decisao: 2026-10-07 (a Madu aprovou seguindo as recomendações do ENG-00)
---
# DEC-0025 — Fechamento da F3 (eventos) do M01

> **Aprovado pela Madu em 2026-10-07.** O PR #3 do `launch-os` foi mesclado na `main` (merge `997c1a5`) com a CI verde e a revisão independente registrada; este DEC é o fechamento da fase.

## Entrega
`packages/schemas` e `packages/events` no `launch-os`, conforme o [[spec-core-04-eventos]] aprovado:
- **Contrato** (Zod 4): envelope de 12 campos, estrito; payloads dos **23 tipos** de evento do M01; validação com coerência (gates e aprovações da transição, dono do estado, aprovação de gate ou de ação); guarda de segredos; `dedupe_key`; forma canônica e hash; constantes de estados, gates, transições e donos.
- **Publicação** na transação do chamador; **idempotência** de pedido sem corpo de resposta; **entregador** com `FOR UPDATE SKIP LOCKED` e ordem por entidade e por processamento; nova tentativa com espera crescente; `dead` com audit e `error.detected` sem laço; **claim e ack** atômicos; claim vencido e recebido sem claim reabertos; reprocesso; limpeza.
- **Configuração de entrega sem valor padrão:** a carga falha listando o que falta.

## Evidências (2026-10-07)
- **44 testes unitários e 112 de integração** passando contra PostgreSQL 17.11; lint, tipos, formatação e `check-repo.sh` aprovados; CI do GitHub verde no PR #3.
- **Provas negativas:** sem o `NOT EXISTS` da ordem, 4 testes falham; sem o `FOR UPDATE` do claim, o teste de espera falha.
- **Revisão independente** (agente diferente do autor, somente leitura): sem achado crítico; 2 altos (alerta perdido no segundo ciclo de `dead`; alerta que falha derrubaria o `dead`), 5 médios e 3 baixos, **todos tratados** no commit `048d6f6`.
- **A CI achou uma coisa que meu `check-repo.sh` local não achou** (rodei antes de `git add`, então os arquivos novos ainda não estavam no índice): uma credencial falsa literal num teste. Corrigido; regra daqui em diante: rodar o `check-repo.sh` depois de `git add`.

## O que mudou em relação à especificação (precisa da sua confirmação)
1. **Recebido e nunca reivindicado é reaberto e reenviado** (código `not_claimed`). A especificação só previa o claim vencido sem ack. Sem o acréscimo, um workflow que respondeu 2xx e nunca pediu o claim deixaria o evento parado, quebrando o "pelo menos uma vez". O reenvio é seguro porque o claim deduplica. Consequência: `claimValidityMs` vale também como prazo para o workflow **começar**.
2. **O claim trava a linha** (`SELECT ... FOR UPDATE`) antes de decidir, para esperar o entregador confirmar o recebimento.
3. **Coerência do payload** mais estrita que o esboço: o evento de transição leva exatamente os gates da transição, uma aprovação por gate e o dono do novo estado; `launch.created` nasce em `IDEA` com o `A0`; `expired` é do sistema e `rejected` é humano, ambos com motivo.
4. **Itens PROP:** `batchSize` como sexta configuração; cabeçalhos `x-correlation-id` e `x-event-id`; códigos `delivery_exhausted`, `claim_expired` e `not_claimed`; assinantes `create-launch`, `approval-resume` e `notify` e o mapeamento mínimo evento-assinante.

## Decisão
1. **Aprovado o fechamento da F3** com a entrega acima e **confirmados** os 4 itens da seção anterior (evento recebido e nunca reivindicado reaberto, claim que trava a linha, coerência mais estrita dos payloads e os itens PROP).
2. **Mesclado** o PR #3 na `main` (feito).
3. A próxima fase é a **F4 (API fina, state machine e aprovações)**, com plano único e as mesmas paradas; ela só começa depois de o plano ser aprovado.

## PENDENTE (não bloqueia o fechamento)
- **Valores definidos pela Madu:** tentativas, esperas, tempo limite, validade do claim e tamanho do lote da entrega; prazos de limpeza das entregas e das chaves de idempotência.
- **Consumidor com versão de payload desconhecida** indo para `dead` com `error.detected` (bloco 4, seção 7): hoje só a publicação recusa; o consumo é do n8n, da F4 e do M02.
- **Premissa para a F4:** a ordem por entidade exige que a API serialize as escritas da mesma entidade (bloqueio da linha do launch na transação); deve ser cumprida e testada na F4.
- Validar no n8n 2.41.6 que o webhook responde 2xx ao receber, quando existirem os workflows (M02 e M03); URLs e segredo dos webhooks.
- Payload dos eventos de domínio (M03 em diante).

## Consequências
- A F4 passa a ser a próxima fase. [[milestones]], [[roadmap]] e [[M01-plano-de-execucao]] foram atualizados em 2026-10-07.

Ver [[DEC-0024-fechamento-da-f2-e-da-f6-do-m01]] · [[spec-core-04-eventos]] · [[M01-plano-de-execucao]].
