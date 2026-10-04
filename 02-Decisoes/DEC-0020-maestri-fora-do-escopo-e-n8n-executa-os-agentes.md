---
tipo: decisao
area: 02-Decisoes
id: DEC-0020
status: aceita
fonte: decisão da Madu em 2026-10-04 ("n8n total, Maestri sairá do escopo"); pesquisa sobre o Maestri em M00-plano-de-execucao
data-registro: 2026-10-04
substitui: DEC-0002 (parcial), DEC-0014, DEC-0005 (runtime dos agentes), DEC-0010 §7 (critério do M00) e a camada de desenvolvimento do DEC-0010
---
# DEC-0020 — Maestri fora do escopo; n8n orquestra e executa os agentes

## Contexto
O Blueprint previa o Maestri como cockpit de desenvolvimento (workspaces, terminais, notas pinadas, conexões). A pesquisa mostrou que o plano gratuito tem 1 workspace, e o desenho pedia 8; a versão Windows existe segundo o site oficial, mas a instalação não foi feita. A Madu decidiu tirar o Maestri do escopo e concentrar a produção no n8n.

## Decisões (todas confirmadas pela Madu em 2026-10-04)

### 1. Maestri fora do escopo
O Maestri deixa o projeto, no desenvolvimento e na produção. Workspaces, terminais, conexões, Partituras e notas pinadas do Maestri não são mais entregáveis. O passo 6 e o passo 7 do M00 deixam de existir como foram escritos.

### 2. Desenvolvimento: Claude Code puro com subagentes
Os 8 engenheiros ENG-00 a ENG-07 são subagentes do Claude Code, definidos em `.claude/agents/`. Ferramenta de orquestração de desenvolvimento: **PENDENTE**, decidir depois; não bloqueia o M00.

### 3. Produção: o n8n orquestra e executa os agentes A0-A10
Os agentes de negócio são executados em workflows do n8n (com nós de IA). PostgreSQL e Redis permanecem. Uma **API fina** do backend mantém estado, gates, aprovações, audit log e eventos. Backend e banco continuam sendo a fonte da verdade.

### 4. Redação da regra "agente x workflow" (aprovada pela Madu)
> O agente continua uma **entidade do registry** (identidade, permissões, escopos); a **execução** acontece em workflows do n8n. Um agente pode ter vários workflows; um workflow não é um agente.

### 5. As 6 notas do DEC-0014 viram arquivos do repositório
Local: `docs/status/` no `launch-os`: `PROJECT_STATUS.md`, `CURRENT_MILESTONE.md`, `CURRENT_TASK.md`, `DECISIONS.md` (só índice para `docs/adr/`), `BLOCKERS.md` e `DEFINITION_OF_DONE.md`. Para manter a fonte única, `docs/status/DEFINITION_OF_DONE.md` é só um ponteiro para `docs/DEFINITION_OF_DONE.md`.

### 6. Critério do M00
O M00 valida a fundação documental, o repositório, o Claude Code e a infraestrutura de desenvolvimento (sem Maestri). Isso substitui o texto do DEC-0010 §7 nesse ponto.

## O que muda nas decisões anteriores
| Decisão | Efeito |
|---|---|
| DEC-0002 | Substituída em parte: continua a separação entre desenvolvimento e produção; sai o Maestri |
| DEC-0005 | O runtime dos agentes passa do backend para o n8n |
| DEC-0010 §7 e camada de desenvolvimento | M00 sem Maestri; dev com Claude Code |
| DEC-0014 | Substituída: notas em `docs/status/` |
| DEC-0013 D6, DEC-0018 | `DECISIONS.md` é índice em `docs/status/`; escopo do M00 sem Maestri |

## O que NÃO muda
Guardrails oficiais do DEC-0010 §8, os 12 gates, a state machine (status x type), as 22 entidades, o contrato de eventos, o policy check `BUDGET_AUTHORIZED`, o roadmap M00-M06 e a hierarquia de autoridade.

## PENDENTES
- **Escopo do ENG-04 e de `packages/agents`:** com o runtime no n8n, o que fica no código (registry, definições, permissões, montagem de contexto, adaptador de LLM) precisa de desenho do ENG-00.
- **Papel do `agent runtime gateway` do ENG-01.**
- **Camada de adaptador de LLM (DEC-0016 D2):** como convive com os nós de IA do n8n.
- **Hospedagem do n8n em produção** (modo fila usa o Redis): junto dos ambientes de teste e produção.
- **Ferramenta de orquestração de desenvolvimento.**
- **Impacto no M02:** o escopo é mantido; "onde roda" fica PENDENTE.

## Alternativas rejeitadas
- n8n como plataforma inteira, sem backend próprio: guardrails de estado e aprovação mais difíceis de garantir e de testar.
- Manter o Maestri com o plano gratuito de 1 workspace.
- Escolher outra ferramenta de orquestração agora.

## Fontes não editadas
O Blueprint e os demais originais em `FASE  1 - BASE/` ainda descrevem o Maestri; são fonte original e não são alterados. Pela hierarquia, este DEC prevalece. Os logs históricos também não são editados.

Ver [[M00-plano-de-execucao]] · [[roadmap]] · [[engenharia-eng-00-a-07]].
