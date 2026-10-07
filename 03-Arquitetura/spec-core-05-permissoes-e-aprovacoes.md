---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0007; DEC-0010 §3, §8; DEC-0016 D8; DEC-0019; DEC-0020; blueprint v1 §13, §19–§21; permissoes-e-hitl; gates; engenharia-eng-00-a-07; M01-plano-de-execucao (F1, F4, F5)
status: decisões D5.1 a D5.10 aprovadas pela Madu em 2026-10-07 (recomendações do ENG-00); valores numéricos e escopos dos 11 agentes seguem PEND (M01, F1, bloco 5 de 6)
atualizado: 2026-10-07
---
# Especificação do Core, bloco 5: permissões, autenticação e aprovações

Parte da F1 do [[M01-plano-de-execucao]] ([[DEC-0022-inicio-formal-do-m01]]). **Proposta, sem código.** Fecha o que os blocos 1 a 4 deixaram para cá: autenticação do operador e do n8n, modelo e aplicação das permissões dos agentes, aprovações (incluindo prazo e `approval.expired`), o policy check `BUDGET_AUTHORIZED` e a auditoria de segurança. Marcas: **PROP** é proposta, **PEND** é o que os documentos não definem e não foi inventado. **Nenhum valor numérico foi inventado**: prazos e limites são configuração por política, definidos pela Madu ([[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] D8).

Blocos: 1 serviços e limites · 2 entidades e schema · 3 state machine e gates · 4 eventos · **5 permissões e aprovações** (este) · 6 ferramentas de código (ADRs). Blocos 1 a 4 com decisões aprovadas em 2026-10-07 ([[spec-core-01-servicos-e-limites]], [[spec-core-02-entidades-e-schema]], [[spec-core-03-state-machine-e-gates]], [[spec-core-04-eventos]]).

## 1. O que os documentos já fixam
| Regra | Fonte |
|---|---|
| Cada agente tem read scopes, write scopes, action scopes, approval requirements e budget limits; só o exemplo do A5 existe | [[permissoes-e-hitl]] (Blueprint §19) |
| Os escopos dos 11 agentes **não estão definidos** (Blueprint de Implementação v1, ainda não produzido) | [[permissoes-e-hitl]], [[A0-ceo-orquestrador]] |
| Níveis AUTO, SUPERVISED, HUMAN DECISION; lista de aprovação humana por padrão | [[permissoes-e-hitl]] |
| O agente nunca altera permissões ou aprovações por mensagem, nunca contorna approval gate; ferramentas precisam de escopo explícito; entrada externa é não confiável | guardrails do ENG-04 ([[engenharia-eng-00-a-07]]) |
| Mensagem de agente nunca concede permissão nem aprova gate humano pendente; privilégio mínimo; ações de segurança vão para o audit log | [[permissoes-e-hitl]] |
| Toda mutação crítica respeita autorização e audit; nenhum secret em resposta de API nem em texto puro | guardrails do ENG-01 e ENG-02 |
| A UI nunca é a única barreira; o backend impõe | guardrails do ENG-05 |
| `BUDGET_AUTHORIZED = true` é policy check registrado no audit log; limite de gasto acima do qual é HUMAN DECISION; valor do limite PENDENTE (antes do M04) | [[gates]], DEC-0017 C4 |
| Aprovação sem resposta vale como não aprovada; `approval.expired` e prazo PENDENTES | DEC-0016 D8, [[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]] |
| O A0 decide sequência, delegação e prioridade; exige humano para mudança estratégica crítica, contrato, grande alteração de orçamento, alteração de política e decisão legal | [[A0-ceo-orquestrador]] |
| Só a Madu opera e aprova | [[M01-plano-de-execucao]] |
| Só a API muda estado; aprovação só por canal humano autenticado; o gateway é a fachada da API | R1 a R8 e D1.5 do bloco 1 |
| `JWT_SECRET` reservado; serviços só em `127.0.0.1`; senha do Redis PENDENTE | [[M01-plano-de-execucao]], [[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]] |
| Quem concede cada gate: HUMANO ou VERIFICAÇÃO | bloco 3, D3.2 |

## 2. Atores e identidade (PROP)
| Ator | Como se identifica | `actor_type` no evento e no audit |
|---|---|---|
| **A Madu** (operador) | login no canal humano; token de sessão | `human`, `actor_id` = id do `user` |
| **Agente A0 a A10** (rodando no n8n) | uma credencial **por agente**, usada pelo workflow | `agent`, `actor_id` = código (`A0` a `A10`) |
| **Sistema** (agendador e entregador da API) | interno, sem credencial exposta | `system` |

Regra central: **a identidade vem da credencial, nunca do que o agente declara na mensagem ou no corpo do pedido.** Uma credencial só vale para o agente a que pertence. O n8n não escolhe quem é.

## 3. Autenticação (PROP)
### 3.1 Operador (canal humano autenticado)
- Login com e-mail e senha da Madu; a senha fica só como **hash forte** (`user.password_hash`, coluna nova), nunca em texto puro.
- A API devolve um **token de sessão de curta duração**, assinado com o `JWT_SECRET` reservado, com `actor_type = human`.
- Os endpoints de aprovação, de edição de permissões, de política e de reprocesso de entrega **só aceitam token humano**. Credencial de agente nunca os acessa (R8).
- A API escuta só em `127.0.0.1` no M01 (DEC-0019).
- Tentativas de login com falha e bloqueio por repetição: registradas no audit; limites numéricos são configuração (**PEND**).
- **PEND:** segundo fator e duração da sessão (valor da Madu); reautenticação para aprovação crítica.

### 3.2 n8n → API (agentes)
- **Credencial opaca por agente** (chave aleatória longa), guardada no *credential store* do n8n (guardrail do ENG-03). Na API fica só o **hash** (`agent_credential`, tabela nova de infraestrutura, com `agent_id`, `key_hash`, `created_at`, `revoked_at`, `last_used_at`).
- Cada pedido do n8n leva a credencial do agente que está agindo, mais `correlation_id`, `execution_id` e, nos pedidos mutadores, `Idempotency-Key` (bloco 4).
- **Credencial de serviço do n8n** (revisão independente): os workflows que não pertencem a um agente (retomada de aprovação, NOTIFY, claim e ack de evento) usam uma credencial de serviço, na mesma tabela `agent_credential` com `agent_id` nulo e `service_name` preenchido (`CHECK`: exatamente um dos dois). Escopo mínimo: claim e ack de evento, obter `correlation_id` e leitura para notificação. **Não pede transição nem aprovação.**
- Revogar e trocar a credencial é feito só pelo operador, sem parar os outros agentes. Agente desativado (`agent.is_active = false`) é negado mesmo com credencial válida.
- A chave é mostrada **uma vez**, na criação; nenhuma resposta da API a devolve depois.
- **Por que uma por agente e não uma do n8n inteiro:** com uma só, um workflow comprometido ou com erro poderia dizer "sou o A0". Com uma por agente, o dano fica limitado ao escopo daquele agente.

### 3.3 API → n8n (webhook)
- A API envia um **segredo no cabeçalho** do webhook, configurado como credencial no n8n. O segredo vive em ambiente seguro, não no banco e não no repositório.
- O n8n rejeita chamada sem o segredo. Evento reenviado não é problema (deduplicação, bloco 4).
- **Limite conhecido:** o segredo estático não tem carimbo de tempo nem assinatura contra repetição (replay). Aceito enquanto tudo roda em `127.0.0.1` (DEC-0019); revisitar quando o webhook for exposto.

### 3.4 Segredos (resumo)
Nenhum segredo em resposta de API, log, evento ou audit. Hash para o que a API só precisa comparar; ambiente seguro para o que ela precisa enviar. **Senha do Redis** e **banco e usuário do n8n separados** (D1.4) são tarefas da F2 e F6 (ENG-02 e ENG-07), listadas aqui para não se perderem.

## 4. Modelo de permissões (PROP)
Segue a D2.5 do bloco 2: `agent.permissions` em `jsonb`, versionado por `permissions_version`.

```json
{
  "read": ["offer", "metric"],
  "write": ["experiment"],
  "actions": ["pause_campaign", "create_test"],
  "requires_approval": ["increase_budget_above_policy_limit"],
  "budget_limits": {}
}
```

- **Negar por padrão:** o que não está listado é negado. Lista vazia significa nada permitido.
- **Vocabulário dos escopos:** o nome da entidade no banco (`launch`, `offer`, `campaign`...). O exemplo do Blueprint usa termos soltos (`audience`, `campaigns`, `offer`); a tradução para os nomes oficiais das 22 entidades vem do ENG-00 e fica no schema versionado de `packages/schemas/`.
- **`actions`:** nomes de ação. A lista de ações é um registro versionado no código; o M01 só registra as que ele próprio implementa (por exemplo, pedir transição e pedir aprovação). **Ações de ferramentas dos agentes são do M02** (tool registry está fora do escopo do M01).
- **`requires_approval`:** ações que o agente pode **pedir**, mas que só se executam depois de `approval` concedida pela Madu.
- **`budget_limits`:** valores definidos pela política (seção 7), não gravados no código. **PEND** até o C4 (antes do M04).
- **Escopos dos 11 agentes: PEND.** O M01 entrega o modelo, o armazenamento e a checagem. O registro semente cria os 11 agentes com **permissões mínimas**: só o necessário para as regras da seção 5. Ampliar o escopo de um agente é decisão da Madu, quando os agentes de M02 e M03 forem definidos.
- **Quem altera `permissions`:** só o operador, por token humano, com audit do antes e do depois. **Nenhum agente altera permissões**, nem as próprias, nem por mensagem (guardrail do ENG-04).

### 4.1 Checagem e aplicação
- **A API aplica sempre**, em todo endpoint mutador. É a barreira real (R1, R2).
- O n8n pode **consultar** antes de executar (fluxo (b) do bloco 1) por um endpoint de checagem, que devolve: permitido ou negado, se exige aprovação, motivo. A consulta serve para o workflow decidir o caminho e **não substitui** a checagem da API.
- Toda negação vai para o `audit_log` com resultado "negado" (bloco 2, 3.8).

## 5. Quem pode pedir o quê no M01 (PROP)
| Ação | Quem pode pedir | Quem decide |
|---|---|---|
| Criar launch | operador, ou A0 | API (regra do bloco 3, incluindo a de JOINT) |
| Pedir uma transição do launch | o `owner_agent` do **estado atual** (bloco 3, seção 4) ou o A0 | API: gates cumpridos + permissão |
| Pedir aprovação de gate | o `owner_agent` do estado atual ou o A0 | pedido só cria `approval` em `requested` |
| Conceder ou rejeitar gate HUMANO | **só a Madu** (token humano) | a Madu |
| Disparar a verificação de gate de VERIFICAÇÃO | o `owner_agent` | API, pela condição objetiva |
| Registrar a decisão de continuidade | A0 propõe; **a Madu confirma** (D3.4) | a Madu |
| Alterar permissões, credenciais, política, ativar ou desativar agente | só a Madu | a Madu |
| Reprocessar entrega `dead` | só a Madu | a Madu |

O A0 pode pedir qualquer transição porque é o orquestrador ("se pode avançar", Blueprint §13), mas **não concede gate** em nenhum caso.

## 6. Aprovações (PROP)
O modelo de dados é o do bloco 2, 3.6; os ajustes abaixo já foram consolidados lá em 2026-10-07.

### 6.1 Ciclo de vida
```text
requested ──(Madu aprova)──> granted ──(usada numa transição)──> consumida
    │
    ├──(Madu rejeita)──────> rejected
    └──(prazo vence)───────> expired        (nova solicitação é um novo pedido)
```
- **Sem resposta vale como não aprovada** (fail-closed): `requested` nunca libera transição.
- `rejected` e `expired` são finais. Para tentar de novo, abre-se **outro** `approval`; o histórico não é sobrescrito.
- Uma aprovação concedida é **de uso único**: fica ligada ao launch, ao gate e ao estado de origem, e é marcada como usada (`consumed_at`, coluna nova) na mesma transação da transição. Não vale para outra transição nem outro launch.
- **Quem concede:** gate HUMANO exige token humano de usuário ativo. Gate de VERIFICAÇÃO é concedido pela API quando a condição objetiva é verdadeira, e a linha registra `decided_by_type = system`. Rejeitar exige `reason` (texto).

### 6.2 Expiração e `approval.expired` (PROP)
- Novo status `expired` e novo evento `approval.expired`, no mesmo formato de `approval.rejected`, com `decided_by.type = system` e `reason` indicando vencimento. Resolve a pendência do [[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]].
- O **agendador dentro da API** (D1.3) marca como `expired` as aprovações `requested` com `expires_at` vencido, grava o evento e o audit. O workflow que esperava é acordado pela entrega do evento (bloco 4) e trata o desfecho.
- O **prazo é política** (`approval_expiry`), definido pela Madu. **Sem valor definido, a aprovação não expira, mas continua valendo como não aprovada.** Nenhum prazo padrão é inventado.

### 6.3 Aprovação de ação (não só de gate)
`approval.subject_type` passa a valer `gate` ou `action`. `action` cobre os itens de `requires_approval` do agente e o gasto acima do limite (seção 7). O `approval` guarda o que está sendo aprovado e o agente que pediu; ao executar a ação, a API confere que existe aprovação concedida e ainda não consumida para aquele pedido.

### 6.4 Canal da Madu até o dashboard (M06)
- No M01 **não há dashboard**. A Madu opera por endpoints autenticados e um **cliente mínimo de linha de comando** (a forma exata é do bloco 6), que lista aprovações pendentes e permite aprovar ou rejeitar. Nada disso passa por workflow (R8).
- **Aviso à Madu:** o `approval.requested` aciona um workflow de notificação (passo NOTIFY do [[n8n-estrutura]]). O aviso **não tem poder de aprovar**: só informa e aponta para o canal autenticado. O **canal de notificação** (e-mail ou mensageria) é **PEND**, escolha da Madu.
- A UI, quando existir, não é barreira: a API reimpõe tudo (guardrail do ENG-05).

### 6.5 Limite conhecido: aprovação de algo que depois muda (PEND)
Se o objeto aprovado mudar depois da aprovação (por exemplo, a oferta alterada depois do G02), a aprovação deveria perder o valor. Os documentos não definem a versão do objeto aprovado, e as entidades de produto e oferta só existem como esqueleto no M01. **Fica PEND para o M03**, quando existirem os objetos. No M01 a proteção é o uso único e o vínculo ao estado de origem.

## 7. Orçamento: `BUDGET_AUTHORIZED` (PROP)
É **policy check**, não gate (DEC-0010 §3). Funciona assim:
1. Toda ação marcada como "com gasto" passa pelo check antes de executar.
2. O check compara o valor com o limite da política (`budget_limits` do agente e a política geral).
3. Dentro do limite e com permissão: `BUDGET_AUTHORIZED = true`; acima: exige `approval` do tipo `action` concedido pela Madu (HUMAN DECISION); sem limite definido: **negado** (fail-closed).
4. O resultado e o motivo vão para o `audit_log` (`details`), com o valor, o limite e a aprovação, quando houver.
- Limite de orçamento e moeda: **PEND**, definidos pela Madu antes do M04 (DEC-0017 C4). Limite de custo de LLM: aberto (M02).
- No M01 nenhuma ação gasta dinheiro: a entrega é o mecanismo testado com limites de teste.

## 8. Política configurável (PROP)
Valores que mudam sem alterar código (prazo de aprovação, limites de orçamento e, no futuro, outros) ficam em `policy_setting` (tabela nova de infraestrutura: chave, valor jsonb, versão, quem alterou e quando).
- **Só o operador altera**, com audit do antes e do depois ("alterações de política" exigem humano, Blueprint §13).
- Sem valor definido, vale a regra de fail-closed de cada caso.
- Os números de infraestrutura do bloco 4 (tentativas, esperas, timeout, validade da chave de idempotência, limpeza das entregas) **não** vão para essa tabela: são configuração de ambiente, por serem técnicos e fixados com o deploy.

## 9. Auditoria de segurança (PROP)
O `audit_log` registra, aceito ou negado:
- login do operador (sucesso e falha), criação e revogação de credencial de agente;
- toda negação de permissão e toda tentativa de agente de conceder gate HUMANO ou alterar aprovação, permissão ou política;
- pedido, concessão, rejeição, expiração e uso de aprovação;
- alteração de `permissions`, de política, ativação ou desativação de agente (antes e depois);
- resultado do `BUDGET_AUTHORIZED`;
- reprocesso de entrega `dead`.

**Mudança de permissão não vira evento no M01**: fica só no audit (fecha a pendência do bloco 4). Os eventos de aprovação já existem (bloco 4).

## 10. Testes exigidos (F4, F5 e F7)
- **Fronteiras de permissão:** agente fora do escopo negado; agente desativado negado; credencial revogada negada; credencial de um agente não age como outro.
- **Aprovação:** token de agente não concede nem rejeita; chamada do n8n não concede; só token humano de usuário ativo concede; rejeição sem `reason` recusada.
- **Ciclo:** `requested` nunca libera; `expired` e `rejected` não liberam; aprovação usada uma vez não vale outra; aprovação de outro launch, outro gate ou outro estado de origem não vale.
- **Expiração:** o agendador expira no prazo, emite `approval.expired` e o workflow é acordado; sem prazo configurado, nada expira e nada é aprovado.
- **Quem pede:** só o `owner_agent` do estado atual e o A0 pedem transição; outros são negados.
- **Orçamento:** gasto sem limite definido negado; acima do limite exige aprovação; tudo no audit.
- **Escalada por mensagem:** texto livre de agente pedindo "me dê permissão" ou "aprove" não muda nada.
- **Segredos:** nenhuma resposta, log, evento, audit **nem linha de `idempotency_key`** contém chave, senha ou token; chave de agente aparece uma única vez, e repetir a criação não a revela de novo.
- **Completude do audit:** cada ação da seção 9 deixa linha de audit, inclusive as negadas.
- **Concorrência:** duas decisões simultâneas sobre o mesmo `approval` resultam em uma só.

## 11. Decisões (D5.1 a D5.10 aprovadas pela Madu em 2026-10-07: vale a recomendação do ENG-00 em todas; ajustes consolidados no bloco 2)
| # | Pergunta | Opções | Recomendação do ENG-00 |
|---|---|---|---|
| **D5.1** | Como a Madu se autentica | **A)** e-mail e senha com hash forte, token de sessão curto assinado com o `JWT_SECRET`, API só em `127.0.0.1`. **B)** chave de API fixa para o operador. **C)** provedor externo de login | **A.** Distingue com clareza o token humano do token de agente, o que sustenta o R8. B mistura com a credencial de serviço; C é peso desnecessário para uma pessoa só. Segundo fator e duração da sessão ficam PEND para você |
| **D5.2** | Como o n8n se autentica na API | **A)** uma credencial opaca por agente (A0 a A10), com hash na API (`agent_credential`), revogável; segredo no cabeçalho do webhook da API para o n8n. **B)** uma credencial única do n8n com o agente informado no pedido. **C)** JWT de serviço assinado com o `JWT_SECRET` | **A.** Limita o dano de um workflow comprometido ao escopo de um agente e impede "me passar por A0". B permite falsificar o agente; C faria o mesmo segredo que protege a sessão da Madu também emitir identidades de agente. Custo: 11 credenciais e uma tabela de infraestrutura |
| **D5.3** | Modelo de permissão | **A)** negar por padrão, `jsonb` (D2.5), vocabulário igual às entidades, registro semente dos 11 agentes com permissões mínimas, checagem sempre na API e endpoint de consulta para o n8n. **B)** liberar por padrão e bloquear exceções | **A.** Privilégio mínimo (DEC-0010 §8). Os escopos completos ficam PEND e entram por decisão sua quando os agentes forem definidos |
| **D5.4** | Quem pode pedir transição e aprovação | **A)** o `owner_agent` do estado atual e o A0; nenhum concede gate HUMANO. **B)** só o `owner_agent`. **C)** qualquer agente com a permissão | **A.** O A0 é o orquestrador que decide "se pode avançar"; limitar ao dono evita que agentes alheios movam o launch. Em nenhum caso o A0 concede gate |
| **D5.5** | Regras da aprovação | **A)** uso único, ligada a launch, gate e estado de origem; rejeição exige motivo; `rejected` e `expired` são finais (novo pedido = nova linha); gate de VERIFICAÇÃO concedido pela API com `decided_by_type = system`. **B)** aprovação reutilizável até o launch mudar de estado | **A.** Evita aprovação "esquecida" valendo depois. Exige as colunas `consumed_at` e `decided_by_type` em `approval` e o status `expired` (ajuste ao bloco 2, 3.6) |
| **D5.6** | Expiração (`approval.expired`) | **A)** status `expired`, evento `approval.expired` (formato de `rejected`, `decided_by` do sistema), agendador da API; prazo vindo da política e **definido por você**; sem prazo, não expira e continua não aprovada. **B)** nunca expirar. **C)** fixar um prazo agora | **A.** Evita workflow esperando para sempre sem eu inventar o número (D8 do DEC-0016). Quando você definir o prazo, ele entra em `policy_setting` |
| **D5.7** | `BUDGET_AUTHORIZED` e política | **A)** policy check (seção 7), fail-closed sem limite, acima do limite exige aprovação sua, registrado no audit; política em `policy_setting`, alterável só por você com audit. **B)** valores no código | **A.** Já é a regra do DEC-0010 §3 e do DEC-0016 D8 ("configuração por política, não valor fixo no código"). Custo: uma tabela de infraestrutura |
| **D5.8** | Como a Madu aprova até o dashboard (M06) | **A)** endpoints autenticados e um cliente mínimo de linha de comando; aviso por workflow de notificação sem poder de aprovar; canal de notificação escolhido por você. **B)** esperar o dashboard. **C)** aprovar por mensagem no canal de notificação | **A.** B trava os gates no M01; C violaria o R8 e a regra "mensagem não aprova gate". O canal de notificação fica PEND |
| **D5.9** | Auditoria de segurança e eventos | **A)** auditar tudo da seção 9; mudança de permissão só no audit, sem evento. **B)** também emitir evento de permissão | **A.** Nenhum consumidor precisa reagir a permissão no M01; o audit imutável cobre a rastreabilidade. Revisitar se o dashboard precisar |
| **D5.10** | Estruturas novas e ajustes ao bloco 2 | **A)** aceitar: tabelas `agent_credential` e `policy_setting` (infraestrutura, fora da lista de 22); em `user`, `password_hash`; em `approval`, `consumed_at`, `decided_by_type`, status `expired` (e `subject_type` = `gate` ou `action`). **B)** rejeitar | **A.** Cada item é consequência direta das decisões acima. Depois da sua aprovação, consolido tudo no bloco 2 com as tabelas do bloco 4 (`event_delivery`, `idempotency_key`) e a coluna `position` e a `dedupe_key` de `event` |

## 12. PENDENTE
- Escopos de leitura, escrita e ações dos 11 agentes (Blueprint de Implementação, M02 e M03).
- Valores numéricos: prazo de aprovação, duração da sessão, limites de tentativa de login, limites de orçamento e moeda (a Madu define).
- Segundo fator e reautenticação para aprovação crítica.
- Canal de notificação à Madu.
- Invalidação de aprovação quando o objeto aprovado muda (M03).
- Critérios objetivos dos gates de VERIFICAÇÃO (bloco 3).
- Registro de ações das ferramentas dos agentes e tool registry (M02).
- Bibliotecas de hash, de token e de agendamento (bloco 6).

## 13. Riscos
- **Credenciais de agente no n8n:** o *credential store* do n8n vira alvo. Mitigação: uma por agente, revogável, escopo mínimo, hash na API, nada em log. Segredos em ambiente seguro (ENG-07).
- **Sobrecarga de aprovação para a Madu:** com os critérios PEND, **11 dos 12 gates** ficam, na prática, como HUMANO; só o G12 tem critério proposto (bloco 3). Na prática, o caminho `decided_by_type = system` só é exercitado pelo G12. Aceito por ser o caminho de menor risco; cada critério objetivo que ela definir reduz a carga.
- **Aprovação "esquecida":** sem prazo definido, `requested` pode ficar parada. O efeito é só atraso (fail-closed), e o aviso à Madu é o remédio; definir o prazo resolve.
- **Aprovação de algo que muda depois** (seção 6.5), em aberto para o M03.
- **Tabelas de infraestrutura crescendo** (`agent_credential`, `policy_setting`, `event_delivery`, `idempotency_key`): cada uma entra na F2 com migration reversível e teste de constraints.
- **RAM da máquina** ao rodar API, PostgreSQL, Redis e n8n juntos ([[M01-plano-de-execucao]], riscos).

## 14. Fora deste bloco
Ferramentas de código (bloco 6), qualquer SQL, migration ou código, execução de agentes e tool registry (M02), dashboard (M06).

Ver [[permissoes-e-hitl]] · [[gates]] · [[catalogo-de-eventos]] · [[engenharia-eng-00-a-07]] · [[A0-ceo-orquestrador]].
