---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0010 §8; DEC-0016 D2; DEC-0020; M02-plano-de-execucao (F1); spec-agent-01; spec-core-05 §4; .claude/agents/ENG-04-agent-engine.md; blueprint v1 §13, §19, §36
status: proposta do ENG-00, revisada após a revisão independente (spec-agent-07) — decisões D2.1 a D2.4 aguardam a Madu
atualizado: 2026-10-08
---
# Bloco 2 do Agent Engine: contrato e definição do agente

## 1. O que já está fixado
Agente = entidade do registry (identidade, permissões, escopos; M01). **Prompts, escopos finais, ferramentas e modelo de cada agente são PENDENTES** ([[A0-ceo-orquestrador]] e as fichas A1–A10). Acesso a LLM sempre por camada de adaptador, Anthropic primeiro (DEC-0016 D2). Guardrail do ENG-04: *agente nunca altera permissões por mensagem; toda execução tem `execution_id` e audit trail*.

## 2. O que é a "definição do agente" (PROP)
O **contrato** que o gateway carrega para executar um agente. Fica em **arquivos versionados no repositório**, revisados por pull request, validados por schema Zod ao subir a API. **Não há edição em tempo de execução** (mudar o comportamento de um agente é um PR e um deploy, como qualquer código). O banco guarda só o que muda em operação: **permissões** e **ativo/inativo** (já existem, M01).

### 2.1 Estrutura (PROP)
```text
agents/definitions/<código>/            (ex.: A0)
  agent.json                            manifesto (schema abaixo)
  prompt.md                             instruções do sistema (texto puro)
```
Manifesto (`agent.json`), campos (todos obrigatórios salvo indicação):
| Campo | Conteúdo |
|---|---|
| `code` | `A0` a `A10`, igual ao do registry |
| `version` | inteiro crescente; mudar o prompt ou qualquer campo exige subir a versão |
| `model` | identificador do modelo — **valor da Madu (PENDENTE)**; sem ele a definição **não carrega** |
| `prompt` | caminho relativo de `prompt.md` (somente dentro da pasta do agente) |
| `tools[]` | nomes de ferramentas do **registry** ([[spec-agent-03-ferramentas-politica-e-aprovacao]]); nome desconhecido recusa a definição |
| `reads[]`, `writes[]` | entidades (vocabulário oficial de `ENTITY_NAMES`) que o agente **declara precisar** |
| `handoff_targets[]` | agentes para os quais pode delegar (vazio por padrão) |
| `triggers[]` | tipos de evento que iniciam este agente (vazio por padrão); o mapa evento → agente vem **só daqui** (bloco 1, seção 4.1) |
| `limits` | `max_steps`, `max_tool_calls`, `max_output_tokens`, `max_context_bytes`, `deadline_seconds`, `max_handoff_depth`, `max_children` — **todos valores da Madu (PENDENTES)** |
| `output_schema` | nome de um schema Zod registrado para a saída final (opcional no M02; os agentes de negócio definem os seus no M03) |
Proibido na definição: segredos, URLs de webhook, chaves, qualquer credencial. Um teste varre as definições com um **guarda restrito a formatos de credencial** (chaves `sk-…`, JWT, `Bearer …`, blocos PEM). **Não** se usa o guarda de eventos no `prompt.md`: ele marca palavras como "senha" e "token" em texto livre e daria falso positivo em prompts de copy e suporte.

### 2.2 Hash e versão (PROP)
`definition_hash` = SHA-256 da forma canônica (`canonicalHash` do M01) do manifesto **mais o conteúdo do prompt com as quebras de linha normalizadas para LF** (o mesmo arquivo dá o mesmo hash no Windows e na CI em Linux). `agents/` e `tests/fixtures/agents/` ficam com `.gitattributes` `eol=lf`. Cada execução grava o `definition_hash` e a `version`: é possível reproduzir **com qual prompt e quais ferramentas** uma execução rodou.

### 2.3 Carga (PROP)
Na subida a API: lê todas as pastas, valida com o schema, resolve ferramentas contra o registry, calcula o hash e mantém tudo **em memória** (somente leitura). **Falha de qualquer definição = a API não sobe**, listando o que falta (mesma regra da configuração sem padrão). O arquivo `agent.json` de um agente sem `model` ou sem `limits` **não carrega**: nada é assumido.

## 3. Definição × permissões do banco (a regra que evita deriva) (PROP)
- **O banco manda.** `agent.permissions` (registry, M01) é a única autoridade em tempo de execução; a definição **declara o que o agente quer**.
- **Na hora de iniciar a execução**, a API confere: toda ação das `tools[]` e todo `reads[]`/`writes[]` da definição precisa estar **permitida no registry**; se não estiver, a execução é **recusada** (`definition_exceeds_permissions`), com audit. A Madu amplia o escopo pela CLI (`agents permissions`), com audit, e só então o agente roda.
- A semente de permissões mínimas **não muda**; os escopos reais dos agentes entram por decisão da Madu (M03).
- Teste de consistência obrigatório na CI: toda definição no repositório referencia só ferramentas existentes e entidades válidas.

## 4. Prompt e entrada não confiável (PROP)
- O **prompt do sistema** é o `prompt.md` da definição e **nunca** é concatenado com texto vindo de fora. Tudo que vem de fora (entrada da execução, resultado de ferramenta externa, conteúdo de memória gravado por terceiros, e-mail, página) entra como **dados marcados** (bloco 5, seção 3), nunca como instrução.
- O prompt **não vai para o n8n**: o n8n não recebe o prompt nem o contexto completo (bloco 1, seção 2), só a **diretiva do próximo passo**. Isso também mantém o texto sensível fora do histórico de execuções do n8n.

## 5. Decisões
| # | Pergunta | Opções | Recomendação |
|---|---|---|---|
| **D2.1** | Onde mora a definição | **A)** arquivos no repositório, schema Zod, PR e hash por execução. B) tabela no banco editável pela API/CLI | **A.** Revisão por PR, histórico no git, sem caminho para um agente (ou um bug) reescrever o próprio prompt; o banco continua só com o que muda em operação |
| **D2.2** | Formato do manifesto | **A)** JSON + `prompt.md`. B) YAML | **A.** Sem dependência nova (o Node lê JSON); o prompt em Markdown é revisável em PR |
| **D2.3** | Quem vence se definição e registry divergem | **A)** o banco (registry) manda; a definição que excede é recusada. B) a definição manda | **A.** A Madu controla o escopo; evita "agente amplia o próprio escopo por PR" sem passar por ela |
| **D2.4** | Valores: modelo por agente e `limits` | Valores da Madu | **Sem valor, a definição não carrega** (fail-closed). A F1 só define os campos |
| **D2.5** | Gatilho por evento | **A)** campo `triggers[]` na definição. B) mapa separado | **A:** um lugar só, revisado junto com o prompt |

## 6. PENDENTE
`model` e `limits` de cada agente; `output_schema` dos agentes de negócio (M03); onde guardar variações de prompt por experimento (M05).

## 7. Riscos
- **Prompt e permissão em lugares diferentes** (arquivo × banco): mitigado pela regra de recusa e pelo teste de consistência.
- **Definição grande demais:** o `max_context_bytes` limita o que é montado, não o arquivo; um teto de tamanho do prompt entra na F3.

## 8. Fora deste bloco
Ferramentas e política ([[spec-agent-03-ferramentas-politica-e-aprovacao]]), contexto e memória ([[spec-agent-05-contexto-e-memoria]]), os 144 workflows de cada agente (M03/M04).
