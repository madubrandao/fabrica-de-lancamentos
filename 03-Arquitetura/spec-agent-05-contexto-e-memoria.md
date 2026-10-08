---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0009; DEC-0016 D4; DEC-0020; memoria-empresarial; spec-core-02; spec-core-05 §4; spec-agent-01 a 04; ENG-04; blueprint v1 §18, §36
status: proposta do ENG-00, revisada após a revisão independente (spec-agent-07) — decisões D5.1 a D5.6 aguardam a Madu
atualizado: 2026-10-08
---
# Bloco 5 do Agent Engine: contexto e memória

## 1. O que já está fixado
Memória empresarial em **três tipos**: estruturada (PostgreSQL), semântica (embeddings, **só se agregar valor**; vector store **adiado**, o M02 só reserva a interface) e operacional (estruturas transacionais; o estado atual nunca depende só da memória semântica) ([[memoria-empresarial]], [[DEC-0009-memoria-empresarial-em-tres-tipos]]). Guardrail do ENG-04: *memory write registra origem e timestamp*. O modelo de dados `facts/entities/relationships`, a retenção e a auditoria são **PENDENTES**; as tabelas da memória estruturada ficaram "no schema do M01" (DEC-0015), mas o M01 entregou só os esqueletos das 22 entidades.

## 2. Montagem de contexto (PROP)
Função **pura e determinística** no pacote de agentes: `assembleContext(definição, permissões, fontes) → { mensagens, hash, bytes, omitidos }`. Fontes e **ordem fixa**:
1. **Sistema:** `prompt.md` da definição (nunca misturado com texto externo).
2. **Catálogo de ferramentas permitidas:** só as da definição **e** permitidas no registry (o modelo nunca vê o que não pode usar).
3. **Tarefa:** a entrada da execução, como dado marcado.
4. **Estado do launch** (se `launch_id`): linha resumida do launch (estado, dono, `approval_state`, gates pendentes) **somente se `launch` estiver em `permissions.read`**.
5. **Histórico da execução:** resultados de ferramentas e passos anteriores, como dados marcados. **Memória não é injetada automaticamente no M02**: o que o agente lê entra por `memory.read`, ou seja, pelo histórico (a injeção automática, com consulta, ordem e limite, é decisão do M03).
**Teto:** `max_context_bytes` (valor da Madu; a relação com a janela em **tokens** do modelo e com `max_output_tokens` é validada na carga da definição). Se estourar, o corte é **determinístico e por item inteiro**: sai primeiro o **estado do launch**, depois os **itens mais antigos do histórico**, **nunca o histórico mais recente** (o laço precisa dele) e **nunca** sistema, catálogo e tarefa; cada corte deixa um marcador `omitido` registrado. Se **sistema + catálogo + tarefa** já passam do teto, o passo falha com **`context_too_large`** (não há corte silencioso). Mesma entrada = mesmo contexto = mesmo `hash` (gravado no `llm_call`).

## 3. Entrada não confiável (PROP)
Tudo que não é o `prompt.md` entra em **blocos delimitados com origem** (`<dado origem="ferramenta:test.echo" confianca="externa"> … </dado>`), com os delimitadores **neutralizados por escape** (`<`, `>` e `&` dentro do dado viram entidades; o conteúdo não consegue fechar o bloco), cobertos por fixtures. O sistema instrui o modelo a tratá-los como dados. Isso é **mitigação, não garantia**: a garantia está no pipeline do bloco 3 (o modelo só *pede*; quem decide é o banco). Testes com fixtures de injeção desde a F3.

## 4. Escopos de leitura (PROP)
O que um agente consegue ler é a interseção de: `permissions.read` (registry) e o que a definição declara. **Passa a ser aplicado** (antes informativo): `memory.read`, o estado do launch e qualquer fonte de contexto passam pelo filtro. Pedido fora do escopo = item omitido e audit `denied` (sem revelar que existe).

## 5. Memória estruturada mínima (PROP)
Uma tabela **append-only** `memory_item` (modelo no estilo do `event`/`audit_log`: sem UPDATE nem DELETE para o papel da aplicação; correção = novo item que **substitui** o anterior):
| Coluna | Conteúdo |
|---|---|
| `id`, `organization_id`, `created_at` | padrão |
| `kind` | `fact` \| `preference` \| `note` (lista fechada; ampliar é decisão) |
| `subject_type`, `subject_id` | entidade a que se refere (opcional). **Restrito às entidades de negócio**: `audit_log`, `event`, `approval`, `user` e `agent` não são assunto de memória. **Dado pessoal (`contact`, `customer`, `conversation`) é proibido no M02** (ver D5.7) |
| `version` | inteiro por cadeia (`organization_id`, `subject_type`, `subject_id`, `key`); o valor vigente é **sempre a maior `version`** (não se usa "o mais recente por `supersedes_id`") |
| `key`, `value` | chave curta e valor (jsonb, tamanho limitado) |
| `source_type`, `source_id`, `source_execution_id` | **origem obrigatória**: `agent` + código + execução, `human` + usuário, ou `import` |
| `supersedes_id` | item que este corrige (nulo no primeiro) |
Integridade: `UNIQUE NULLS NOT DISTINCT (organization_id, subject_type, subject_id, key, version)` (PostgreSQL 17; **assunto nulo também fica protegido**), `UNIQUE (supersedes_id)` (sem **fork**), `CHECK (version = 1 ⇔ supersedes_id IS NULL)` e o item substituído tem de ser da **mesma cadeia** e `version − 1` (FK composta ou gatilho no padrão da 0006); `memory.write` envia `expected_version` e usa `pg_advisory_xact_lock` por (organização, assunto, chave), como o `policy.ts` do M01 (conflito → 409); `CHECK` de origem (`agent` exige `source_id` e `source_execution_id`; `human` exige o usuário e não tem execução; `import` exige `source_id`); **`REVOKE UPDATE, DELETE` explícito** para o papel da aplicação (o `ALTER DEFAULT PRIVILEGES` do M01 concede `UPDATE`) mais gatilhos no padrão da migration 0006; guarda de segredos e de tamanho no `value`.
O "valor vigente" de uma chave é o item mais recente da cadeia de `supersedes_id`. **Escopos de memória:** `memory` entra como **23º nome** no vocabulário de entidades (`ENTITY_NAMES`), para `read: ['memory']` e `write: ['memory']`. `memory` **não** pode ser `subject_type` (autorreferência). Item **sem assunto** só é lido por agente com `read: ['memory']` e gravado por agente com `write: ['memory']`. Exigir `write` da entidade do assunto sobrecarrega o significado de `write`: **risco registrado**. `objection` e `partner` ficam na lista de assuntos; a Madu confirma se carregam dado pessoal. Escrita por agente exige `write: ['memory']`, grava `source_*` (origem e `execution_id`) e `created_at` e gera audit. **Leitura de memória gravada por terceiros volta como dado marcado** (bloco 5 seção 3).
**O modelo rico** (`facts/entities/relationships` com tipos, confiança, expiração) **fica PENDENTE para o M03**, quando existir um agente que precise; o mínimo acima não o impede (é um superconjunto de compatível).

## 6. Memória semântica (PROP)
Só a **interface** (`SemanticMemory { add(item), search(query, filtros) }`) e uma implementação `NotConfigured` que devolve `not_configured`. **Nenhum vector store, nenhuma extensão, nenhuma imagem de banco nova** no M02 (a imagem `postgres:17` não traz `pgvector`, DEC-0016 D4). Quando houver caso de uso concreto, a Madu decide a ferramenta.

## 7. Auditoria e retenção (PROP)
Toda escrita e toda leitura **negada** de memória vai ao `audit_log` com `execution_id`; leitura permitida fica em `tool_call` (sem duplicar o conteúdo no audit). **Retenção: PENDENTE (Madu).** Como a tabela é append-only, o crescimento é monitorado; um prazo de arquivamento exige decisão e migration própria.

## 8. Decisões
| # | Pergunta | Opções | Recomendação |
|---|---|---|---|
| **D5.1** | Montagem de contexto | **A)** função pura na API, ordem e corte determinísticos, filtrada por escopos de leitura. B) o workflow monta | **A.** Reprodutível, testável e o n8n não vê o contexto |
| **D5.2** | Entrada externa no contexto | **A)** sempre como dado marcado e delimitado, mais o pipeline como garantia. B) texto livre | **A** |
| **D5.3** | Memória estruturada | **A)** `memory_item` append-only mínima, origem obrigatória, `memory` como escopo. B) modelo rico `facts/entities/relationships` já | **A.** O modelo rico espera um caso de uso real (M03) |
| **D5.4** | Memória semântica | **A)** só interface + `NotConfigured`. B) adotar `pgvector` agora | **A** (DEC-0016 D4) |
| **D5.5** | Escopos `read`/`write` | **A)** aplicados a partir do M02, inclusive o **escopo dinâmico** pelo `subject_type`. B) continuam informativos | **A** (mesma decisão D3.2) |
| **D5.6** | Visibilidade da memória entre agentes | **A)** da organização inteira, filtrada pelo escopo da entidade-assunto (um agente só lê assuntos de entidades que pode ler). B) cada agente só lê o que ele gravou | **A:** memória é "da empresa" (DEC-0009). **Risco anotado:** um agente grava e outro lê; a leitura volta como dado marcado |
| **D5.8** | Texto livre na memória | **A)** no M02, `memory.write` só aceita `key` de uma **lista fechada** configurada (vazia em produção; de teste nos testes) e `kind` fixo: sem texto livre, logo sem caminho para e-mail ou telefone. B) texto livre com risco residual aceito | **A:** o M02 entrega o mecanismo; a memória de negócio é do M03 |
| **D5.7** | Dado pessoal na memória | **A)** proibido no M02 (assuntos pessoais recusados no schema); o caminho de apagamento/anonimização (LGPD) é decidido junto com o M03. B) permitir já | **A:** a tabela é só de inserção e não suporta pedido de apagamento sem um desenho próprio |

## 9. PENDENTE
`max_context_bytes`; limite de tamanho de `value` e de itens por leitura; retenção; modelo rico da memória e do vector store (M03+); política de expiração.

## 10. Riscos
- **Memória como vetor de injeção** (um agente grava texto malicioso que outro lê): mitigado por origem obrigatória, dado marcado e pipeline; resta um risco residual, a revisar no M03.
- **Corte de contexto que remove o que importa:** a prioridade fixa é documentada e testada; o `omitidos` fica no registro.

## 11. Fora deste bloco
Execução ([[spec-agent-04-execucao-estado-e-telemetria]]), código e testes ([[spec-agent-06-ferramentas-de-codigo-e-testes]]).
