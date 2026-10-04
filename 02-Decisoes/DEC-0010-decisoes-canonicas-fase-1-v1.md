---
tipo: decisao
area: 02-Decisoes
id: DEC-0010
status: APROVADO-PARA-IMPLEMENTACAO
fonte: documento fornecido pela Madu em 2026-10-02 (FASE-1-BASE_DECISOES_CANONICAS_v1.md), transcrito sem alteração de conteúdo
data-registro: 2026-10-02
resolve: P01, P02, P03, P09 (e P04, P05, P06, P08 por aplicação da hierarquia do §1; ver pendencias-de-decisao)
---
# DEC-0010 — FASE 1 - BASE — Decisões Canônicas v1

> Esta nota é a **fonte de maior autoridade** do projeto (nível 1 da hierarquia abaixo). Em caso de conflito com qualquer outra nota, vale esta.
>
> **Nota de 2026-10-04:** a camada de desenvolvimento e o critério do M00 (§7) foram substituídos em parte pelo [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]] (Maestri fora do escopo; o n8n executa os agentes). O texto do documento foi preservado.

Status: APROVADO PARA IMPLEMENTAÇÃO

Objetivo: eliminar conflitos entre Plano Final, Blueprint, starter e notas da FASE 1 - BASE antes de qualquer código.

## 1. Fonte canônica

Quando houver conflito entre documentos de projeto, a ordem de autoridade é:

1. Decisões canônicas registradas em `02-Decisoes/`.
2. Blueprint Técnico `v1.x` aprovado.
3. Documentos de arquitetura e roadmap derivados do Blueprint.
4. Starter técnico.
5. Rascunhos e propostas conceituais anteriores.

O starter não redefine arquitetura. Ele implementa decisões já aprovadas.

## 2. PDF do livro

O PDF de Jeff Walker permanece como fonte metodológica externa ao diretório operacional do projeto.

Não copiar, mover ou incorporar o PDF à árvore `knowledge/plf/` nesta fase.

A base metodológica já registrada nas notas é suficiente para a engenharia atual. Qualquer lacuna metodológica deve ser marcada como PENDENTE até consulta explícita à fonte.

## 3. Gates canônicos

Manter os 12 gates operacionais do Blueprint:

- G01 Market Validated
- G02 Offer Approved
- G03 Product Ready
- G04 Content/PLC Approved
- G05 CRM Ready
- G06 Technical QA Passed
- G07 Launch Ready
- G08 Cart Open Authorized
- G09 Cart Close Authorized
- G10 Post Launch Complete
- G11 Analysis Complete
- G12 Learning Recorded

### Orçamento

`Budget Approved` NÃO vira um 13º gate da máquina de estados.

Ele será uma pré-condição/policy check antes de qualquer execução que envolva gasto:

`BUDGET_AUTHORIZED = true`

O check poderá ser exigido em qualquer milestone e registrado em audit log. Isso preserva a exigência do Plano Final sem duplicar o estado do lançamento.

## 4. Roadmap canônico

A numeração oficial passa a ser:

- M00 Foundation
- M01 Core Platform
- M02 Agent Engine
- M03 First Business Agents
- M04 Launch Execution
- M05 Learning Loop
- M06 Operator Dashboard

O roadmap do starter anterior M0-M8 passa a ser considerado uma decomposição histórica e NÃO é a fonte canônica.

Qualquer roadmap futuro deve usar M00-M06.

## 5. State Machine canônica

Estados de execução do Launch:

IDEA · RESEARCH · VALIDATION · OFFER_DRAFT · PRODUCT_BUILD · PRE_PRE · PLC · LAUNCH_READY · CART_OPEN · CART_CLOSING · CART_CLOSED · POST_LAUNCH · ANALYSIS · LEARNING · DECISION

### Regra importante após DECISION

`SEED`, `INTERNAL`, `JOINT`, `RELAUNCH` e `EVERGREEN` são TIPO/PRÓXIMA AÇÃO de lançamento, não estados do mesmo ciclo.

Após `DECISION`, o sistema cria uma decisão de continuidade:

- `SEED` -> criar novo Launch do tipo SEED
- `INTERNAL` -> criar novo Launch do tipo INTERNAL
- `JOINT` -> criar novo Launch do tipo JOINT
- `RELAUNCH` -> criar novo ciclo de lançamento da mesma oferta
- `EVERGREEN` -> ativar/encaminhar para uma sequência perene
- `ARCHIVE` -> encerrar o ciclo sem próximo lançamento

Portanto, não misturar `launch.status` com `launch.type`.

## 6. Mapa oficial de tipos de Launch

`launch.type`: SEED · INTERNAL · JOINT · RELAUNCH · EVERGREEN

`launch.status` contém apenas estados de execução.

## 7. Milestones e critérios

- **M00** valida a fundação documental, repositório, Maestri, Claude Code e infraestrutura de desenvolvimento.
- **M01** valida PostgreSQL, Redis, migrations, eventos, audit log, Agent Registry, permissions, approvals e state machine.
- **M02** valida Agent Contract, context assembly, tool registry, memory interfaces, execution lifecycle e policy enforcement.
- **M03** valida A0-A4.
- **M04** valida A5-A10 e execução do ciclo completo.
- **M05** valida análise, experimentação e learning loop.
- **M06** valida painel operacional e aprovação humana.

## 8. Guardrails oficiais dos engenheiros

### ENG-00 Architect
- Não implementar mudança estrutural sem contrato/ADR quando necessário.
- Não ocultar conflito documental.
- Não inventar requisito marcado PENDENTE.
- Não redefinir regra de negócio sem decisão registrada.
- Não aprovar a própria mudança crítica sem revisão apropriada.

### ENG-01 Backend
- Não alterar schema sem migration aprovada.
- Toda mutação crítica deve respeitar autorização e audit log.
- Não colocar regra de negócio dentro de workflow n8n quando ela pertence ao backend.
- Eventos devem ser explícitos, versionáveis e idempotentes quando aplicável.
- Nunca expor secrets ou credenciais em resposta da API.

### ENG-02 Database
- Toda mudança de schema exige migration reversível ou plano explícito de rollback.
- Não fazer migração destrutiva sem aprovação.
- Constraints devem preservar integridade referencial.
- Índices devem ter justificativa de acesso.
- Secrets não podem ser persistidos como texto puro.

### ENG-03 n8n
- Backend/database são a fonte da verdade.
- Todo workflow deve usar correlation ID.
- Operações mutáveis devem ser idempotentes quando aplicável.
- Retry não pode duplicar efeitos.
- Credenciais devem permanecer no credential store.
- Todo erro relevante deve ter branch explícito e auditável.

### ENG-04 Agent Engine
- Agente nunca altera permissões ou aprovações por mensagem.
- Ferramentas precisam de escopo explícito.
- Inputs externos são não confiáveis até validação.
- Toda execução deve ter execution_id e audit trail.
- Agente nunca pode contornar approval gate.
- Memory write deve registrar origem e timestamp.

### ENG-05 Frontend
- Não acessar banco diretamente.
- Ações perigosas exigem confirmação de interface e enforcement no backend.
- Mostrar status, aprovação e auditabilidade antes de ação crítica.
- Não esconder erro operacional.
- Não permitir que a UI seja a única barreira de autorização.

### ENG-06 QA
- Todo contrato crítico deve possuir teste.
- Toda transição crítica deve ser testada positiva e negativamente.
- Falha não pode ser marcada como sucesso sem evidência.
- Testes de workflow devem cobrir retry, duplicação e erro.
- Testes de agente devem verificar limites de ferramenta e aprovação.

### ENG-07 DevOps/Security
- Secrets apenas em ambiente seguro/secret store.
- Containers de produção devem ser reproduzíveis e versionados.
- Backups devem ser verificáveis, não apenas configurados.
- Health checks e logs são obrigatórios para serviços críticos.
- Nenhuma exposição pública sem autenticação, autorização e observabilidade adequadas.

## 9. Regra para documentos incompletos

Os itens abaixo continuam PENDENTES e NÃO devem ser inventados automaticamente:

- prompts finais dos agentes de negócio;
- ferramentas exatas de cada agente;
- escopos de dados por agente;
- schemas completos dos objetos de negócio;
- KPIs finais de cada agente;
- regras detalhadas de autonomia;
- integrações externas definitivas.

Esses itens serão especificados nos milestones correspondentes.

## 10. Regra de implementação

A FASE 1 - BASE permanece documentação e arquitetura.

Nenhum código de produção deve ser criado como consequência automática destas decisões sem iniciar o M00 formalmente.

Próximo marco autorizado: M00 Foundation.
