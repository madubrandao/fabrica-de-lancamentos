---
tipo: proposta-conceitual
area: 01-Contexto
status: PROPOSTA NÃO APROVADA
atualizado: 2026-10-16
origem: visão expressa pela operadora nas conversas de 2026-10-16; não constava do vault original
relacionados: "[[visao-do-projeto]], [[A0-ceo-orquestrador]], [[permissoes-e-hitl]], [[proposta-afropilot-studio-producao-criativa]]"
---
# PROPOSTA — AfroPilot, assistente executivo permanente

> **NÃO IMPLEMENTADO / NÃO APROVADO COMO ARQUITETURA DE PRODUÇÃO.** Formaliza a visão da operadora para revisão; não altera DEC-0001 a DEC-0038, marcos, permissões, orçamento nem a autorização do M03.

## 1. Propósito

O **AfroPilot** é o assistente executivo permanente da operadora para organizar prioridades, acompanhar projetos, propor próximos passos, delegar tarefas a sistemas autorizados e reunir entregas para decisão humana. Sua função existe **também fora de um lançamento**. A Sol é o primeiro caso de uso comercial; a operação da Afro Ads e outros produtos podem ser incorporados depois, com escopos próprios.

**Exemplo de resultado desejado:** a operadora informa que dispõe de duas horas para avançar no lançamento da Sol; o AfroPilot apresenta uma prioridade concreta, propõe tarefas, encaminha as ações autorizadas ao LAUNCH OS, acompanha os estados e reúne materiais para revisão.

## 2. Separação de responsabilidades (não criar agentes duplicados)

| Componente | Responsabilidade | O que NÃO é |
|---|---|---|
| **Operadora (Madu)** | Aprova estratégia, conteúdo editorial, gastos, publicação e ações críticas. | Não é substituída por automação. |
| **AfroPilot** | Porta de entrada conversacional e coordenação pessoal de projetos, tarefas e pendências; adapta a apresentação do trabalho à operadora. | Não é o agente A0 e não concede permissões às ferramentas. |
| **LAUNCH OS** | Sistema de execução da empresa de lançamentos: estado, regras, gates, auditoria, n8n, API e agentes A0–A10. | Não é uma interface pessoal para todo e qualquer projeto. |
| **A0** | Agente de negócio do LAUNCH OS: orquestra objetivos, dependências e agentes **dentro de um lançamento**. | Não é o assistente permanente fora do LAUNCH OS. |
| **A1, A2, A3...** | Responsabilidades especializadas documentadas em `05-Agentes/`. | Não são novos agentes do AfroPilot. |
| **AfroPilot Studio** | Módulo de produção de arquivos visuais e audiovisuais a partir de briefings autorizados. | Não decide oferta, promessa ou estratégia editorial. |
| **Claude Code / ENG-00…07** | Desenvolvimento, testes e manutenção do software. | Não deve ser a dependência permanente de execução da camada de produção. |

**Direção de integração proposta:** `Operadora → AfroPilot → API / gates do LAUNCH OS → A0 → agentes → entregas → AfroPilot → decisão da operadora`. O Studio pode receber um `CreativeBrief` de A3 (ou da operadora em trabalhos recorrentes fora de lançamentos), mas não deve contornar as regras de autorização.

## 3. Capacidades pretendidas

1. **Planejamento diário assistido:** priorizar uma tarefa executável dentro do tempo disponível; reduzir sobrecarga de opções, sem alterar automaticamente compromissos.
2. **Quadro de execução:** distinguir `planejado`, `aguardando decisão`, `em execução`, `bloqueado` e `concluído`, com próxima ação explícita.
3. **Coordenação de projetos:** distinguir Sol (produto), Afro Ads (operação) e cada lançamento ou campanha; não misturar dados de projetos.
4. **Delegação governada:** emitir apenas comandos expostos pelo LAUNCH OS e autorizados para o contexto; sem privilégio implícito decorrente de mensagem de chat.
5. **Central de revisão:** apresentar entregas, proveniência, limitações e custo antes de ações irreversíveis.
6. **Acompanhamento e memória:** consultar fontes versionadas e estado operacional; registrar decisões aprovadas, sem tratar mensagens livres como fonte canônica.

## 4. Modos de uso propostos

- **Assistido (primeira experiência):** conversa com a operadora e preparação de materiais usando Claude Code interativo, com ações executadas manualmente sob supervisão. **Não é integração de produção nem execução autônoma.**
- **Integrado (futuro):** interface conversa/CLI/painel envia tarefas pela API do LAUNCH OS, acompanha `execution_id`, consulta estados e apresenta aprovações. Depende de desenho, autenticação, autorização, auditoria e dos gates aplicáveis.
- **Proativo (futuro):** resumos recorrentes e alertas apenas com canal, frequência, consentimento e políticas aprovados. Sem trabalho em segundo plano presumido.

## 5. Fluxo mínimo de valor (critério de aceitação proposto)

**Pedido:** "Prepare um conteúdo educativo da Sol sobre evitar dupla marcação".

1. AfroPilot registra objetivo, produto, canal e limites da solicitação.
2. LAUNCH OS encaminha, quando autorizado, tarefas de pesquisa e conteúdo ao caminho A0 → A1 → A3.
3. A3 devolve o conteúdo textual e um `CreativeBrief` para produção visual (contrato ainda proposto).
4. Studio cria uma peça estática em modo assistido, com ferramentas permitidas e ativos com proveniência.
5. AfroPilot reúne texto, visual, checklist de qualidade e pendências para revisão da operadora.
6. A operadora decide explicitamente o resultado editorial. **Nenhuma publicação ou gasto automático.**

**Primeira entrega mensurável:** um pacote de conteúdo e uma peça visual revisável, com fonte identificada, sem envio externo. Não exige dashboard completo nem 11 agentes executando simultaneamente.

## 6. Governança e estado atual

- A hierarquia de autoridade continua em [[escopo-e-principios]]; decisões aceitas prevalecem sobre esta proposta.
- Aplicam-se AUTO / SUPERVISED / HUMAN DECISION da [[DEC-0007-human-in-the-loop-tres-niveis]] e as regras da [[permissoes-e-hitl]].
- Aprovação de ferramenta **não equivale** a aprovação editorial; aprovação editorial **não equivale** a permissão de publicar.
- DEC-0037 continua bloqueando o início formal do M03 até cumprimento e decisão explícita. DEC-0038 continua **proposta** no arquivo recebido.
- Nesta data, **não há implementação documentada** do AfroPilot no código do LAUNCH OS; trata-se de uma visão a especificar, sem criar um 12º agente de negócio por padrão.

## 7. Pendências para futura decisão

- Identidade/autenticação e permissões da interface do AfroPilot.
- Contrato `TaskRequest`, estados, isolamento de projetos e forma de armazenamento do histórico.
- Como receber eventos e aprovações do LAUNCH OS sem duplicar sua state machine.
- Que ações poderão ocorrer sem confirmação explícita e como revogá-las.
- Canal de interface inicial definitivo (CLI, conversa ou painel) e mecanismos de operação proativa.

Ver [[proposta-afropilot-studio-producao-criativa]] · [[proposta-afropilot-studio-especificacao-de-imagens]] · [[A0-ceo-orquestrador]].
