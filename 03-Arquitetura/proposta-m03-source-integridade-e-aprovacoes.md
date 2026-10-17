---
tipo: arquitetura
area: 03-Arquitetura
id: proposta-m03-source-integridade-e-aprovacoes
status: PROPOSTA (rascunho de 2026-10-16 para revisão da Madu; nada implementado; schemas e runtime intactos)
fonte: launch-os main afa36c5 (packages/schemas/src/experimental/content-flow.ts; apps/api/src/gateway/step.ts; apps/api/src/gateway/tools.ts; database/migrations/0005 e 0010); DEC-0038 (proposta)
data-registro: 2026-10-16
---
# Proposta: `source`, validação preventiva de integridade e duas aprovações

**Só documento.** Não modifica schemas nem runtime. A implementação depende da aprovação da [[DEC-0038-preparacao-do-m03-em-prs]] e de autorização do PR.

## 1. Campo `source`
Campo obrigatório, **sem valor padrão** (ausente = rejeição), em `ObjectionMap` e `ContentPackage`; substituiria o literal `fictional: true` do módulo experimental.

| Valor | Significa | NÃO significa |
|---|---|---|
| `fictional` | Todo o conteúdo e os dados de origem são inventados para teste. Nada pode ser apresentado como fato. | — |
| `operator_approved` | A operadora **autorizou o uso** do material de origem. | Não significa afirmações verificadas, nem aprovação editorial do conteúdo final. |

**Âncora confiável:** a origem vem do `input` da execução raiz criada pela operadora (imutável depois do estado final, trigger `agent_execution_final_is_immutable`). O mapa e o pacote devem trazer o mesmo valor; ninguém o eleva no caminho (o A3 não promove `fictional` a `operator_approved`). Ausente ou divergente = rejeição. O `evidence` do A1 continua `hypothesis` nos dois casos.

**Como evitar fictício apresentado como fato:** (1) exibição com marcador visível "[FICTÍCIO]" na revisão; (2) a submissão registra `publishable: false` para `fictional`, e qualquer ferramenta futura de publicação deve recusá-lo; (3) sem valor padrão, o contrato falha fechado; (4) a triagem heurística (`reviewContentPackage`) continua, mas não substitui a revisão humana; (5) o texto de `operator_approved` na interface lembra que o uso foi autorizado, não que os fatos foram checados.

## 2. Validação preventiva de integridade (antes da aprovação)
**Diretriz aprovada:** a integridade é validada **obrigatoriamente antes** de a submissão chegar à aprovação.

**Fatos do código:** o pipeline de `packages/agents` é puro (sem banco). A aprovação nasce em `step.ts` (`createActionApproval`, por volta da linha 381), dentro da transação, com acesso ao banco. Execução em estado final é imutável por trigger (resultado e `input` não mudam depois), o que dá vínculo confiável com o que foi persistido.

**Ponto de imposição recomendado:** pré-checagem em `step.ts`, imediatamente **antes** de `createActionApproval`, só para a ferramenta `content.submit_for_approval` (gancho `precheck` por ferramenta, implementado em `tools.ts`), mais **reverificação no executor** da ferramenta depois da concessão (defesa em profundidade). A concessão não substitui a checagem.

**Vínculo com os originais persistidos:** a partir da execução do A0 que chama a ferramenta, localizar as filhas por `parent_execution_id`, `trigger_type='handoff'` e `trigger_ref` igual ao `call_id` de um `tool_call` `handoff.start` do próprio A0 com `target` A1 ou A3 e `status='succeeded'`. Exigir **exatamente uma** de cada (senão `missing_child` ou `ambiguous_child`).

**O que é verificado:**
1. Integridade A1 → A3: `verifyObjectionMapTransfer(resultado do A1, input do A3)`.
2. Integridade A3 → submissão: o pacote submetido é canonicamente igual ao `result.text` do A3.
3. `reviewContentPackage` sem problemas contra o mapa **original**.
4. `source` igual à da execução raiz.

**Rejeição segura:** negação `integrity_check_failed`, **fatal** na primeira versão (a execução falha; a operadora refaz com `executions retry`); auditoria `tool.denied` só com códigos estáveis, sem conteúdo do mapa ou do pacote; nenhuma aprovação criada; nenhum caminho concede aprovação automaticamente.

**Alternativas descartadas:** pipeline (puro, não vê o banco); `handoff.start` (preveniria a alteração antes de o A3 recebê-la, mas exige conhecer os contratos de cada agente; fica como melhoria futura); validar só no consumidor (tarde: a Madu já teria aprovado).

**Exige:** previsão expressa na DEC-0038 (§5) e ADR própria.

## 3. Duas aprovações
| | Aprovação técnica da chamada | Aprovação editorial do conteúdo |
|---|---|---|
| O que é | Mecanismo existente (`approval` da ferramenta) | Julgamento da operadora sobre qualidade, marca e promessas |
| Autoriza | **Registrar** a submissão com aqueles parâmetros (`paramsHash`, uso único, expira, invalida se a definição mudar) | Usar o conteúdo **fora** do sistema (publicação manual) |
| Registro | `approval` + `tool_call` | Decisão da operadora, com `package_hash`, resultado e nota |

**Menor solução, sem migration:**
- **Técnica:** `content.submit_for_approval` exige aprovação sempre. O texto exibido à operadora diz: "conceder **registra** a submissão; **não** é aprovação editorial e **não** autoriza publicar". O resultado após a concessão é `{ recorded: true, editorial_status: 'pending' }`. O sistema **não** tem estado "aprovado para publicação" nem ferramenta de publicação.
- **Editorial, nível 0 (PR1):** `approvals reject --reason` já exige motivo (serve como "ajustes/reprovado"). A decisão editorial é registrada em `07-Log/` com `package_hash`, resultado (aprovado para publicação manual / ajustes / reprovado) e nota. Modelo:
  `Pacote <hash> · source <valor> · decisão <aprovado|ajustes|reprovado> · nota <texto> · por <nome> em <data>`.
- **Editorial, nível 1 (opcional, PR posterior):** `launch-os content decide <approval_id> --outcome approved|changes_requested|rejected --note "..."`, endpoint só da operadora, gravando em `audit_log` (append-only, sem migration). A tabela `decision` só aceita o tipo `CONTINUITY` e não serve.
- **Teste:** nenhum campo ou evento chamado `published` ou `approved_for_publication` é produzido pelo fluxo.

**Custo para a operadora:** dois momentos (conceder o registro; depois decidir editorialmente). É o preço da separação; o nível 1 reduz o atrito depois, se a Madu quiser.

## 4. Pendências para a Madu
Aprovar ou ajustar: a semântica de `source`; o ponto de imposição e a rejeição fatal; o nível 0 ou 1 da decisão editorial; a DEC-0038.

Ver [[DEC-0038-preparacao-do-m03-em-prs]] · [[e1-inventario-de-dados-persistidos]] · [[permissoes-e-hitl]] · [[gates]]
