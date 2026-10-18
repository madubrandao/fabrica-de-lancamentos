---
tipo: decisao
area: 02-Decisoes
id: DEC-0039
status: PROPOSTA (não aprovada). Uma decisão única para liberar o primeiro teste real A0 → A1 → A3
fonte: DEC-0034 (§2, §3); DEC-0037 (§6, D-7); DEC-0038 (§3, registro da decisão final); decisões da Madu de 2026-10-17 (E5/E6, itens 1 a 5) e 2026-10-18 (D6, D10, D15); launch-os PR #56 (fumaças M01 49/49, M02 31/31, fluxo 22/22 duas vezes)
data-registro: 2026-10-18
---
# DEC-0039: liberação do primeiro teste real do M03 (PROPOSTA)

> **PROPOSTA, não aprovada.** Com o aceite, este texto passa a ser o **plano único** do M03 que a DEC-0038 §6 previa. Ele substitui as proibições da DEC-0038 §3 **só no escopo abaixo**. Nada do que já foi aprovado é reaberto.

## 1. O que se pede
Uma única decisão, com cinco itens: o aceite da E4, a liberação do M03 para um teste, o produto fictício, as permissões do A0 e os limites de custo e de modelo.

## 2. Já aprovado e preservado (não se decide de novo)
- **DEC-0034 §2:** o perfil local do `.env`.
- **DEC-0034 §3:** `approval_expiry` 259200 s e `budget_limit` 0.
- **DEC-0038, decisão final:** os `limits` das definições.
- **D-7:** `pnpm ops:audit-partitions` antes de cada sessão; falha é bloqueio.
- **Decisões 1 a 5 da E5/E6:** `MAINTENANCE_*` desligado e nenhuma limpeza nem retenção. As partições são criadas na **subida da API** (`server.ts`), sem a manutenção.
- **D6, D10 e D15:** aprovadas em 2026-10-18.
- **DEC-0037 §6:** "necessário, não suficiente".

## 3. Itens para aprovação

| Item | Proposta |
|---|---|
| **E4 concluída (D1)** | A Madu declara a E4 concluída com a evidência já registrada: a `age` no Bitwarden, com recuperação validada e cópia offline, e a `N8N_ENCRYPTION_KEY` numa nota separada, comparada com o `.env` (17/10). Detalhe no Registro de execução de `pre-M03-d4-d3-plano-unificado`. |
| **M03, liberação (D13)** | Local, na máquina da Madu, supervisionado e só com dados fictícios. Escopo: **um** fluxo de conteúdo A0 → A1 → A3, até a submissão para revisão humana e a decisão editorial (`content decide`). **Não inclui** `LAUNCH_READY`, outros agentes nem publicação. |
| **O M03 autoriza** | No banco **de desenvolvimento**: migrations 0001 a 0014, semente e bootstrap das identidades. Também: políticas e permissões pela API; criar o n8n do desenvolvimento e importar com `--include-content-flow`; `BACKUP_INCLUDE_N8N_DATA=true`, com backup e restore verificado em alvo descartável. A chave Anthropic entra no `.env` e `LLM_PROVIDER=anthropic` fica ligado **só durante a sessão do teste**. Até **3 execuções** do A0 por sessão, e cada nova tentativa só com o OK da Madu. |
| **Continua proibido** | Publicar ou enviar conteúdo; dados ou clientes reais; Railway e produção; `MAINTENANCE_*`, limpeza ou retenção; skills; outros agentes; alterar volumes de outros projetos. |
| **O teste termina quando** | O A0 conclui com a submissão registrada e a Madu grava a decisão editorial; ou falha, é cancelado ou esgota as 3 tentativas. Depois disso o `LLM_PROVIDER` é desligado e a chave sai do `.env` (ou é revogada no Console). |
| **Produto fictício (D9)** | "Serviço de tranças da Sol, fictício". **Promessa:** conteúdo educativo sobre considerar a duração do serviço para não haver conflito na agenda. **Público:** trancistas autônomas. Sem clientes, preços, números ou depoimentos. **Entrada do A0:** `{"objective": "Criar conteúdo educativo para atrair trancistas interessadas na Sol", "source": "fictional"}`. |
| **Permissões do A0 (D14)** | `{"read": [], "write": [], "actions": ["tool.handoff.start", "tool.content.submit_for_approval"], "requires_approval": [], "budget_limits": {}}`. São só as duas ferramentas da definição, testadas assim na fumaça. A1 e A3 ficam com a semente (sem ferramentas). Escopo de memória: nenhum, porque o fluxo não usa memória. |
| **Custo e modelo (D7)** | `llm_cost_limit` US$ 20 acumulados. `llm_pricing` copiado da tabela oficial **no dia da ativação** e conferido antes de gravar; o registrado em 17/10 é US$ 2 por milhão de tokens de entrada e US$ 10 de saída. Modelo `claude-sonnet-5-5`. Chave num workspace próprio, com limite de gasto no Console se a conta permitir. Reserva medida de US$ 0,71 por fluxo e pior caso de US$ 2,12; com 3 tentativas, no máximo cerca de US$ 6,36. |

## 4. Conflitos documentais e resolução proposta
1. **Valores do LLM:**
   - A DEC-0034 §3 diz "aceita" o teto de US$ 20, o preço do dia, o `claude-sonnet-5-5` e a chave em workspace próprio.
   - A DEC-0038 §3.1 diz que eles "continuam não aprovados".
   - **Resolução:** o aceite desta DEC aprova esses valores para este teste, e a contradição deixa de existir.
2. **E4-2, "evidências versionadas":**
   - A auditoria pedia commit no vault mais um PR de docs no launch-os, e a Madu vetou PRs só de documentação.
   - **Resolução:** o versionamento oficial é o vault, quando a Madu mesclar o PR #3. O `CURRENT_TASK` e o `BLOCKERS` do launch-os, hoje desatualizados, são atualizados dentro do próximo PR de código, e não bloqueiam o teste.
3. **Vigência da DEC-0038:** ela vale "até o início formal do M03". Com este aceite, as proibições dela ficam substituídas, só no escopo da seção 3.
4. **ADR-0030 com status "proposta"**, embora mesclada com o aceite da Madu em 17/10: é só desatualização documental; corrigir no próximo PR de código.
5. **Folga das partições:** a migration cria o mês atual e mais 2, e a subida da API recria o que faltar. Não há conflito com `MAINTENANCE_*` desligado.

## 5. Sequência depois do aceite (cada passo só se o anterior passar)
1. **Pré-checagem:**
   - PR #56 mesclado e `main` verde;
   - RAM livre e pelo menos 20 GB no `C:`;
   - nenhum contêiner de outro projeto rodando.
2. **`.env` completo pelo `bootstrap.ps1`,** que só acrescenta o que falta, com o perfil da DEC-0034 §2. Antes, registrar o tamanho do arquivo; nunca ler os valores. Sem `MAINTENANCE_*` e sem a chave.
3. **Banco do desenvolvimento:**
   - migrations 0001 a 0014, semente e bootstrap;
   - a operadora faz login.
4. **API no ar.** `pnpm ops:audit-partitions` tem de retornar 0 (D-7), com registro.
5. **Configuração pela API:** políticas da seção 3 (o `llm_pricing` só no dia) e permissões do A0.
6. **n8n do desenvolvimento:**
   - criar e importar com `--include-content-flow`;
   - os 4 webhooks têm de responder.
7. **Backup e restore (E4-3):**
   - backup com `n8n_data`;
   - restore verificado em alvo descartável, com a credencial do n8n decifrando.
8. **A Madu cria a chave.** Ela entra no `.env`, o `LLM_PROVIDER` é ligado e a API reinicia.
9. **Teste:**
   - `ops:audit-partitions` de novo;
   - `executions start A0` com a entrada da seção 3;
   - a Madu concede pela CLI e, depois, `content decide`.
10. **Encerramento:** desligar o LLM, registrar o custo real e o resultado no vault.

## 6. Aprovação
Aprovada por: ________ em: ___/___/2026.

Ver [[DEC-0034-perfil-de-valores-e-plano-ate-o-m03]] · [[DEC-0037-sequencia-d4-d3-backup-e-retencao]] · [[DEC-0038-preparacao-do-m03-em-prs]] · [[pre-M03-d4-d3-plano-unificado]]
