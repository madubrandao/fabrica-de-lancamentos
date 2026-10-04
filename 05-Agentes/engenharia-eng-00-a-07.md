---
tipo: agente
area: 05-Agentes
camada: desenvolvimento
fonte: DEC-0010 §8 (guardrails oficiais); blueprint v1 §4–§7, §40; starter .claude/agents/*.md (superado nos guardrails)
status: canonico (guardrails)
atualizado: 2026-10-04
---
# Agentes de engenharia — ENG-00 a ENG-07 (camada de desenvolvimento)

**Não são os agentes A0–A10.** São os Claude Codes que CONSTROEM a empresa, operados como subagentes do Claude Code ([[DEC-0002-duas-camadas-maestri-so-desenvolvimento]], substituída em parte pelo [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]).

Os **guardrails oficiais** abaixo vêm de [[DEC-0010-decisoes-canonicas-fase-1-v1]] §8 e **substituem** os do starter (que estavam vazios/incompletos). O starter implementa decisões; não as redefine.

## Responsabilidades
| ID | Terminal | Responsável por |
|---|---|---|
| ENG-00 | ARCHITECT | arquitetura, ADRs, contratos entre serviços e de eventos, interfaces, decisões técnicas, critérios de integração |
| ENG-01 | BACKEND | API, autenticação, autorização, regras de negócio, launch state machine, event handlers, agent runtime gateway, audit log |
| ENG-02 | DATABASE | PostgreSQL, schemas, migrations, índices, constraints, relações, event store, memória estruturada, retenção e auditoria |
| ENG-03 | N8N | workflows, triggers, webhooks, subworkflows, retries, tratamento de erro, integrações, scheduling, filas |
| ENG-04 | AGENT ENGINE | agent registry, system prompts, tool registry, context assembly, memory retrieval/write, routing, permissões, estado de execução, handoffs, approval hooks |
| ENG-05 | FRONTEND | dashboard, products, offers, launches, audience, approvals, metrics, agent status, audit logs, configuração |
| ENG-06 | QA | unit, integration, contract, E2E, testes n8n, agent evals, regressão, failure injection, release gates |
| ENG-07 | DEVOPS / SEC | Docker, deploy, ambientes, secrets, backup, monitoramento, logs, hardening, CI/CD, disaster recovery |

## Guardrails oficiais
**ENG-00 Architect**
- Não implementar mudança estrutural sem contrato/ADR quando necessário.
- Não ocultar conflito documental.
- Não inventar requisito marcado PENDENTE.
- Não redefinir regra de negócio sem decisão registrada.
- Não aprovar a própria mudança crítica sem revisão apropriada.

**ENG-01 Backend**
- Não alterar schema sem migration aprovada.
- Toda mutação crítica respeita autorização e audit log.
- Não colocar regra de negócio em workflow n8n quando pertence ao backend.
- Eventos explícitos, versionáveis e idempotentes quando aplicável.
- Nunca expor secrets ou credenciais em resposta da API.

**ENG-02 Database**
- Toda mudança de schema exige migration reversível ou plano explícito de rollback.
- Sem migração destrutiva sem aprovação.
- Constraints preservam integridade referencial.
- Índices com justificativa de acesso.
- Secrets nunca em texto puro.

**ENG-03 n8n**
- Backend/database são a fonte da verdade.
- Todo workflow usa correlation ID.
- Operações mutáveis idempotentes quando aplicável; retry não duplica efeitos.
- Credenciais no credential store.
- Todo erro relevante tem branch explícito e auditável.

**ENG-04 Agent Engine**
- Agente nunca altera permissões ou aprovações por mensagem.
- Ferramentas precisam de escopo explícito.
- Inputs externos são não confiáveis até validação.
- Toda execução tem `execution_id` e audit trail.
- Agente nunca contorna approval gate.
- Memory write registra origem e timestamp.

**ENG-05 Frontend**
- Sem acesso direto ao banco.
- Ações perigosas exigem confirmação de interface **e** enforcement no backend.
- Mostrar status, aprovação e auditabilidade antes de ação crítica.
- Não esconder erro operacional.
- A UI nunca é a única barreira de autorização.

**ENG-06 QA**
- Todo contrato crítico tem teste.
- Toda transição crítica testada positiva e negativamente.
- Falha não vira sucesso sem evidência.
- Testes de workflow cobrem retry, duplicação e erro.
- Testes de agente verificam limites de ferramenta e aprovação.

**ENG-07 DevOps/Security**
- Secrets apenas em ambiente seguro/secret store.
- Containers de produção reproduzíveis e versionados.
- Backups verificáveis, não apenas configurados.
- Health checks e logs obrigatórios para serviços críticos.
- Nenhuma exposição pública sem autenticação, autorização e observabilidade adequadas.

## Efeito do DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes
O runtime dos agentes A0-A10 passa para o n8n. Os guardrails oficiais acima **não mudam**. O escopo do ENG-04 e o papel do `agent runtime gateway` do ENG-01 ficam **PENDENTES** até o desenho do ENG-00 no M01.

## Onde ficam guardrails e regras ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] D7)
- Guardrails oficiais: **só** na seção `Guardrails` de cada `.claude/agents/ENG-xx.md` (fonte única).
- `.claude/rules/*.md`: só regras transversais por tema (architecture, n8n, security, testing), sem repetir guardrail de agente.
- `CLAUDE.md` raiz: referencia, não copia.
- `Responsibilities` nas definições: só responsabilidade, sem regras coladas.
- Dispensa de teste que falha: registrada em `docs/adr/`; não enfraquece o guardrail do ENG-06.
- Vale ao importar o starter no M00; o starter não foi modificado.

## Responsabilidades reutilizáveis (histórico; Maestri fora do escopo)
LEAD_ARCHITECT · BACKEND_ENGINEER · DATABASE_ENGINEER · N8N_ENGINEER · AGENT_ENGINEER · FRONTEND_ENGINEER · QA_ENGINEER · DEVOPS_ENGINEER · REVIEWER · TESTER.

## Conexões recomendadas no canvas (histórico; Maestri fora do escopo)
ARCHITECT → todos · BACKEND ⇄ DATABASE · BACKEND ⇄ AGENT ENGINE · BACKEND ⇄ N8N · AGENT ENGINE ⇄ N8N · FRONTEND ⇄ BACKEND · QA ⇄ BACKEND/N8N/AGENT ENGINE · DEVOPS ⇄ todos.

Ver [[starter-zip-inventario]].
