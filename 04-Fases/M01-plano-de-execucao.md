---
tipo: fase
area: 04-Fases
fonte: DEC-0010 §7 e §10; DEC-0015; DEC-0017; DEC-0020; DEC-0021; blueprint v1 §14–§21, §41; plano aprovado pela Madu em 2026-10-04
status: em execução (início formal em 2026-10-04, DEC-0022); F0 e F1 concluídas (portão F1 aprovado em 2026-10-07, DEC-0023); F2 e F6 liberadas, ainda não iniciadas
atualizado: 2026-10-04
---
# M01 Core Platform — plano de execução (proposta)

> **O M01 foi autorizado pela Madu em 2026-10-04** ([[DEC-0022-inicio-formal-do-m01]]). Nenhum código de aplicação, schema ou migration é criado antes de a Madu aprovar a especificação (portão F1). Antecedente: [[DEC-0021-m00-foundation-concluido]].

## Escopo
Validar PostgreSQL, Redis, migrations reversíveis, eventos e event schema, audit log, Agent Registry, permissions, approvals (os 12 gates e o policy check `BUDGET_AUTHORIZED`), launch state machine (status x type, decisão de continuidade) e API skeleton ([[DEC-0010-decisoes-canonicas-fase-1-v1]] §7, [[DEC-0015-alocacao-do-roadmap-m01-m04]]). A **API é fina**: o n8n executa os agentes ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]).

**Fora do escopo:** execução de agentes, context assembly, tool registry e interfaces de memória (M02); agentes de negócio (M03 em diante); dashboard (M06); workflows do n8n além do necessário para testar a integração; adaptador de LLM; vector store; hospedagem de teste e produção (antes do M03).

## Decisões da Madu já dadas para este plano (2026-10-04)
- **Humanos que operam e aprovam: só a Madu (1 pessoa).** Modelo de aprovação sem papéis nem delegação; o `User` fica preparado para crescer. Uma única `Organization` ([[DEC-0017-modelo-de-negocio-inicial]]).
- **Ferramentas de código:** o `eng-00-architect` propõe e a Madu aprova em ADR; **nada é fixado antes**. A proposta inicial (Fastify, migrations SQL up/down, Zod, Vitest, pnpm) é só intenção, não decisão.

## Como corre: especificação primeiro, com portão de aprovação
Seguindo o Blueprint §41, o primeiro pedido ao Architect é produzir a especificação técnica **sem implementar**, registrando como PENDENTE o que não estiver definido. **Nenhum código de aplicação antes de a Madu aprovar a especificação** (portão F1).

## Fases
| Fase | Dono | Entrega | Verificação |
|---|---|---|---|
| **F0 Início formal** | Madu e ENG-00 | DEC de início do M01; pré-condições (disco ≥ 20 GB, RAM, containers de outros projetos parados); branches `feat/*` | `df -h /c`, CI verde, `docs/status/` atualizado |
| **F1 Especificação (portão)** | ENG-00 | Arquitetura de serviços e limites API x n8n; colunas das 22 entidades; transições da state machine com **mapeamento estado → gate**, quem aprova cada gate e `owner_agent` por estado; catálogo de eventos com payloads, idempotência e retenção; modelo de permissões; modelo de aprovação (1 aprovador; aprovação sem resposta = não aprovada); ADRs das ferramentas | A Madu revisa e aprova cada decisão; revisão independente da especificação |
| **F2 Banco e schema** | ENG-02 | Migrations reversíveis (up/down) das 22 entidades; event store append-only; `audit_log` imutável; aprovações; semente mínima (uma organização) | Round-trip up/down no CI com PostgreSQL; testes de constraints e de imutabilidade |
| **F3 Eventos** | ENG-00 e ENG-01 | Contrato de evento em `packages/schemas/` (fonte única); publicar e consumir em `packages/events/`; entrega confiável (ADR); idempotência; `correlation_id` obrigatório; papel do Redis | Testes de contrato; evento duplicado; falha parcial |
| **F4 API fina, state machine e aprovações** | ENG-01 | API skeleton; state machine que **só transiciona com o gate cumprido**; decisão de continuidade; gates G01-G12; `BUDGET_AUTHORIZED` como policy check no audit log; eventos `approval.requested/granted/rejected`; audit em toda mutação crítica | Testes positivos e negativos de **cada** transição e gate; completude do audit |
| **F5 Agent Registry e permissões** | ENG-04 e ENG-01 | Entidade e API do registry; armazenamento do modelo de permissões e endpoint de checagem que o n8n consultará; **sem executar agentes** | Testes de limites de escopo e de aprovação |
| **F6 Infra e CI** | ENG-07 | Jobs de lint, typecheck, unit, integration e migration round-trip com serviços PostgreSQL e Redis; versão do Node e gerenciador de pacotes fixados; segredos conforme o guardrail | CI verde em PR; `docker compose` com a API saudável |
| **F7 Qualidade** | ENG-06 | Testes de contrato de eventos, state machine, aprovações, audit, idempotência e failure injection | Cobertura dos guardrails; evidências |
| **F8 Fechamento** | Madu aprova | Verificação final, **revisão independente completa**, DoD com evidência, DEC de fechamento | Mesmo rigor do M00 |

**Ordem:** F0 → F1 (portão) → F2 e F6 em paralelo → F3 → F4 → F5 → F7 acompanha F2-F5 → F8. Cada engenheiro altera só o seu domínio, em branches `feat/*`, com revisão por outro agente antes do merge.

## Decisões que o M01 vai exigir da Madu (cada uma vira ADR ou DEC)
1. Ferramentas de código: framework do backend, migrations, validação de schemas, testes, versão do Node, gerenciador de pacotes.
2. Entrega de eventos: outbox transacional no PostgreSQL, Redis ou combinação; papel do Redis no M01.
3. Autenticação entre o n8n e a API e do operador (a Madu); uso do `JWT_SECRET` reservado.
4. Mapeamento estado → gate ([[gates]], [[launch-state-machine]]), quem aprova cada gate (a Madu, o A0 ou ambos) e critérios objetivos.
5. Prazo de expiração da aprovação e o evento `approval.expired`; limites numéricos e SLA (DEC-0016 D8).
6. Campos da entidade "decisão de continuidade".
7. Banco e usuário do n8n separados do banco de negócio.
8. Senha do Redis.
9. Escopo do ENG-04, de `packages/agents` e do `agent runtime gateway` do ENG-01 com o runtime no n8n (DEC-0020). **Tratamento na F1:** o gateway é a fachada da API (D1.5); `apps/worker` não existe no M01 (D1.3); `packages/agents` e o escopo do ENG-04 ficam PENDENTES para o M02.
10. Política de retenção do audit log e dos eventos ([[catalogo-de-eventos]]).

## Guardrails (DEC-0010 §8; fonte única em `.claude/agents/` do `launch-os`, [[engenharia-eng-00-a-07]])
Schema só com migration reversível ou plano de rollback, sem migração destrutiva sem aprovação; toda mutação crítica com autorização e audit; evento explícito, versionável e idempotente; nenhum secret em resposta de API ou em texto puro; agente nunca contorna gate nem altera permissões; backend e banco como fonte da verdade.

## Riscos
- **Disco e RAM:** 22 GB livres e ~0,8 GB de RAM livre medidos em 2026-10-04; o M01 sobe PostgreSQL, Redis, n8n e a API, e a CI usa serviços. Manter ≥ 20 GB livres e parar os containers de outros projetos ao trabalhar.
- **Regras de negócio não definidas** (gates, aprovadores, prazos): mitigado pelo portão F1; nada é inventado.
- **Runtime no n8n** (DEC-0020) pode mudar o escopo do Agent Registry e do gateway: tratar na F1.
- **Revisão independente** só vale se feita por outro agente com escopo completo, como no M00.
- **Proteção da branch `main` em repositório privado** pode exigir plano pago do GitHub: verificar antes de depender dela; sem ela, valem PRs e revisão por convenção.
- **Escopo crescer:** o M01 não inclui execução de agentes nem dashboard.

## Critérios de pronto do M01
Todas as capacidades cumprindo o [[definicao-de-pronto]] (código, testes passando, logs, erros tratados, permissões, documentação, evento definido, schema versionado, ADR atualizado, revisão por outro agente); CI verde com testes de integração e migration round-trip; os 12 gates testados positiva e negativamente, junto com as transições que os exigem ([[spec-core-03-state-machine-e-gates]], D3.8); completude do audit demonstrada; pilha Docker com a API saudável; disco ≥ 20 GB; revisão independente completa; DEC de fechamento aprovado pela Madu.

## F0 concluída (2026-10-04)
Pré-condições conferidas e registradas no [[DEC-0022-inicio-formal-do-m01]]: disco 22 GB livres, CI verde em `f00f597`, repositórios sincronizados, Claude Code 2.1.289, número de humanos respondido (só a Madu). O `docs/status/` do `launch-os` foi atualizado para refletir o M01 em andamento. **Achado:** a proteção de branch não existe neste repositório privado (exige GitHub Pro ou repositório público), então a revisão por outro agente antes do merge vale por convenção. As branches `feat/*` são criadas quando cada fase começa; a F1 começa em `feat/core-platform`.

## F1 concluída (2026-10-07)
Especificação em seis blocos mais fechamento, com as decisões D1.1 a D7.7 aprovadas pela Madu ([[spec-core-01-servicos-e-limites]] a [[spec-core-07-revisao-independente-f1]]); revisão independente em duas passadas (0 críticos e 0 altos no final); portão aprovado no [[DEC-0023-aprovacao-do-portao-f1-especificacao-do-core]]. Os ADRs do bloco 6 foram gravados no `launch-os` (ADR-0005 a ADR-0017, commit local, sem push). **Próximo:** F2 (banco e schema) e F6 (infra e CI), cada uma com a confirmação da Madu para começar.

Ver [[milestones]] · [[roadmap]] · [[M00-plano-de-execucao]].
