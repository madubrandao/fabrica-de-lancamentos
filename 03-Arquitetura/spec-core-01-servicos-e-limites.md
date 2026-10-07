---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0004; DEC-0005; DEC-0007; DEC-0009; DEC-0020; blueprint v1 §1.2, §16–§18, §22–§23, §30, §41; PLANO FINAL §18
status: decisões D1.1 a D1.5 aprovadas pela Madu em 2026-10-07 (recomendações do ENG-00); R1 a R8 seguem como proposta até a revisão final da especificação (M01, F1, bloco 1 de 6)
atualizado: 2026-10-07
---
# Especificação do Core, bloco 1: serviços e limites entre a API e o n8n

Parte da F1 do [[M01-plano-de-execucao]] ([[DEC-0022-inicio-formal-do-m01]]). **Sem código.** As decisões D1.1 a D1.5 foram aprovadas pela Madu em 2026-10-07 (seção 5); as regras R1 a R8 seguem como proposta até a revisão final. O que os documentos não definem está marcado **PENDENTE**.

Blocos da especificação: **1 serviços e limites** (este) · 2 entidades e schema · 3 state machine e gates · 4 eventos · 5 permissões e aprovações · 6 ferramentas de código (ADRs).

## 1. O que os documentos já fixam
| Regra | Fonte |
|---|---|
| Workflows e agentes não conversam diretamente: usam um event bus central, com eventos imutáveis, `correlation_id` obrigatório e `causation_id` quando há evento pai | [[DEC-0004-integracao-orientada-a-eventos]] |
| Estado de negócio fica no backend e no banco, nunca escondido no n8n | [[DEC-0005-stack-de-producao]] |
| O n8n orquestra e **executa** os agentes A0-A10; a API é fina (estado, gates, aprovações, audit, eventos) | [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]] |
| Agente nunca altera permissões nem aprovações por mensagem e nunca contorna gate; mensagem de agente não aprova gate humano | [[DEC-0007-human-in-the-loop-tres-niveis]], guardrails em [[engenharia-eng-00-a-07]] |
| Todo workflow usa correlation ID; operações mutáveis são idempotentes; retry não duplica efeitos | guardrails do ENG-03 |
| O estado atual nunca depende só da memória semântica | [[DEC-0009-memoria-empresarial-em-tres-tipos]] |
| Padrão de workflow: TRIGGER, VALIDATE, LOAD CONTEXT, EXECUTE, WRITE RESULT, EMIT EVENT, LOG, NOTIFY | [[n8n-estrutura]] |

## 2. Componentes e responsabilidades (proposta)
| Componente | Responsável por | Não é responsável por |
|---|---|---|
| **Operador** (a Madu) | decisões HUMAN DECISION, aprovações de gates e de gasto | executar workflows |
| **API fina** (`apps/api`) | estado do negócio, state machine e gates, aprovações, permissões, audit log, persistência e emissão de eventos, idempotência | rodar agentes, orquestrar workflows, falar com LLM |
| **n8n** | orquestração e execução dos agentes A0-A10 e das integrações, retries, agendamento dos workflows | guardar estado de negócio, decidir gates, alterar permissões |
| **PostgreSQL** | fonte da verdade do estado estruturado, event store, audit log | lógica de aplicação |
| **Redis** | cache, filas e controle de execução assíncrona | fonte da verdade de qualquer estado de negócio |
| **Provedores de LLM** | geração, sempre atrás da camada de adaptador ([[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]]) | estado |

## 3. Fluxos principais (proposta)
```text
(a) Criar um launch
Operador → API: valida, grava estado + evento launch.created + audit, gera correlation_id
         → API aciona o workflow do n8n, passando launch_id e correlation_id

(b) Um agente executa uma etapa (workflow no n8n)
n8n → API: pede o contexto do launch e a checagem de permissão do agente
n8n executa (LLM, ferramentas)
n8n → API: "solicito a transição / registro o resultado", com chave de idempotência
API valida permissão e gate; se ok, muda o estado, grava audit e emite o evento

(c) Aprovação humana
API: cria approval.requested (evento + audit) e avisa o operador
Operador → API (canal humano autenticado): aprova ou rejeita → approval.granted / approval.rejected
API → n8n: o workflow que esperava a aprovação continua (ou é abortado)
Aprovação sem resposta vale como NÃO aprovada

(d) Efeito com gasto
n8n → API: pede autorização; a API checa BUDGET_AUTHORIZED e a política; registra no audit
```
Regra comum: **só a API muda o estado de negócio**; o n8n pede, e a API decide.

## 4. Regras de fronteira (proposta)
- **R1.** O n8n lê e escreve o estado de negócio **só pela API**, nunca direto nas tabelas (ver decisão D1.1).
- **R2.** Toda mutação crítica passa pela API, com autorização e audit.
- **R3.** Os eventos são persistidos pela API; o n8n emite eventos **pedindo à API**, nunca escrevendo no event store.
- **R4.** O `correlation_id` nasce na API e é propagado pelo n8n em toda chamada; `causation_id` liga o evento ao anterior.
- **R5.** Toda chamada mutadora do n8n leva uma **chave de idempotência**; repetir a chamada não repete o efeito.
- **R6.** O n8n guarda apenas o estado de **execução** dele (execuções do workflow); nenhum estado de negócio.
- **R7.** Só a API transiciona a state machine; nenhum agente contorna um gate.
- **R8.** Aprovações só mudam por canal **humano autenticado**, nunca por chamada de workflow ou de agente.

## 5. Decisões (D1.1 a D1.5 aprovadas pela Madu em 2026-10-07: vale a recomendação do ENG-00 em todas)
| # | Pergunta | Opções | Recomendação do ENG-00 |
|---|---|---|---|
| **D1.1** | Como o n8n acessa o estado de negócio? | **A)** só pela API (R1). **B)** leitura direta no banco e escrita pela API. **C)** acesso direto (contraria os guardrails do ENG-01 e do ENG-03) | **A.** Uma única porta de entrada: toda regra, permissão e audit passam pela API. Custo: um endpoint por leitura que o n8n precisar. B reduz endpoints, mas espalha o conhecimento do schema para dentro dos workflows |
| **D1.2** | Como a API aciona o n8n? | **A)** a API chama um webhook do n8n. **B)** a API põe um item numa fila Redis que o n8n consome (o suporte a isso na versão fixada precisa ser verificado). **C)** o n8n consulta a API de tempos em tempos | **A.** Mais simples e fácil de testar; exige o n8n sempre disponível, e a API trata falha e reenvio com idempotência. B fica para quando houver necessidade de volume |
| **D1.3** | Quem mede o **tempo** (expiração de aprovação, timeout de etapa)? Os documentos não dizem; precisa de um relógio | **A)** agendador dentro do processo da API. **B)** workflow de agenda no n8n chamando a API. **C)** um processo separado, `apps/worker` | **A para o M01.** Não cria processo novo e mantém a regra "sem resposta = não aprovada" no dono do estado. `apps/worker` só se aparecer carga que justifique (hoje aparece na árvore do Blueprint sem descrição) |
| **D1.4** | Banco do n8n | **A)** mesma instância PostgreSQL, **banco e usuário separados**. **B)** instância separada. **C)** como está hoje: mesmo banco e mesmo superusuário | **A.** Dá privilégio mínimo sem pesar na máquina (RAM e disco limitados). B isola mais, mas custa mais recursos. C contraria o privilégio mínimo |
| **D1.5** | O que é o "agent runtime gateway" do ENG-01 (pendência do [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]])? | **A)** a **fachada da API** que os workflows de agentes usam: checagem de permissão, pedido de transição, registro de resultado e emissão de evento. **B)** um serviço separado entre o n8n e a API | **A.** Evita um serviço a mais; define o gateway como o conjunto de endpoints do R1 a R5 |

**Resolvido nos blocos seguintes:** autenticação do n8n e do operador (bloco 5), entrega confiável dos eventos (bloco 4) e a lista de endpoints (seção 5-bis; os contratos exatos saem do schema versionado nas fases F3 e F4). **Continua PENDENTE:** onde a montagem de contexto roda e o que é lido de cada agente (M02, junto da memória).

## 5-bis. Catálogo mínimo de endpoints (PROP, revisão independente)
Só a **lista de intenções**; os contratos exatos (corpo, validação, códigos) saem do schema versionado em `packages/schemas/` nas fases F3 e F4. Cabeçalhos comuns (nomes PROP): `Authorization`, `X-Correlation-Id`, `X-Execution-Id` e, nos pedidos mutadores, `Idempotency-Key`.

| Grupo | Intenção | Quem chama | Idempotente |
|---|---|---|---|
| Launch | criar launch; ler launch e contexto; pedir transição | operador ou A0 (criar); agente com permissão (ler); `owner_agent` ou A0 (transição) | criar e transição: sim |
| Aprovação | pedir aprovação de gate ou de ação; listar pendentes; conceder ou rejeitar | agente (pedir); operador (listar, conceder, rejeitar) | pedir: sim |
| Decisão | propor e registrar a decisão de continuidade | A0 (propor); operador (confirmar) | sim |
| Evento | registrar resultado e emitir evento a pedido do n8n; claim e ack de evento | agente (emitir); serviço do n8n (claim e ack) | sim |
| Permissão | consultar se o agente pode agir | agente | leitura |
| Correlação | obter um `correlation_id` novo para fluxo que nasce fora da API (por exemplo, agendado no n8n) | serviço do n8n | não |
| Registry | listar e ler agentes; editar permissões; ativar e desativar; criar e revogar credencial | operador | sim |
| Política | ler e alterar `policy_setting` | operador | sim |
| Entrega | listar entregas `dead` e reprocessar | operador | sim |
| Saúde | healthcheck | local | leitura |

Erros padronizados (códigos HTTP e corpo sem segredo, com `correlation_id`): validação, não autenticado, negado (permissão ou gate), não encontrado, conflito (estado ou chave de idempotência) e falha interna.

**Bootstrap das identidades:** não passa pela API; o comando do `apps/cli` fala direto com o banco (bloco 5, 3.5), então não há endpoint de bootstrap na lista.

**Fora do M01 (revisão independente):** `apps/worker` não existe (D1.3 aprovada: o agendador fica na API); `packages/agents` e o escopo do ENG-04 ficam **PENDENTES** para o M02; o gateway é a fachada da API (D1.5).

## 6. Riscos
- **API como ponto único de falha:** se a API cair, nenhum agente muda estado. Mitigação: healthcheck, retries com backoff no n8n e idempotência; é o preço de ter um único dono do estado.
- **Muitos endpoints pequenos:** mitigado por definir só o que um workflow do caminho crítico (Create Launch) realmente usa.
- **Acoplamento ao n8n 2.41.6** (versão fixada): validar o webhook, a espera por aprovação e a idempotência na versão fixada antes de depender deles.
- **RAM da máquina** ao subir API, PostgreSQL, Redis e n8n juntos: parar os containers de outros projetos (`docs/status/BLOCKERS.md` do `launch-os`).

## 7. Fora deste bloco
Colunas das entidades (bloco 2), transições e gates (bloco 3), payload dos eventos e entrega (bloco 4), permissões e aprovações (bloco 5), ferramentas de código (bloco 6).

Ver [[visao-geral]] · [[catalogo-de-eventos]] · [[permissoes-e-hitl]] · [[observabilidade-e-testes]].
