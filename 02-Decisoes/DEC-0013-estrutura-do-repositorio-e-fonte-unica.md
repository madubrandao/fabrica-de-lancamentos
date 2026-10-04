---
tipo: decisao
area: 02-Decisoes
id: DEC-0013
status: aceita
fonte: decisão da Madu em 2026-10-04; blueprint v1 §7–§12, §22; PLANO FINAL §22–§23; starter (lido no scratchpad, não modificado)
data-registro: 2026-10-04
resolve: P06, P08
---
# DEC-0013 — Estrutura do repositório `launch-os` e fonte única por assunto

## Contexto
A leitura de todo o planejamento achou 7 duplicidades ou conflitos na estrutura do repositório, nas decisões, no compose e nos guardrails. Critério da Madu: a estrutura mais limpa quanto a duplicidade, estrutura e guardrails. Este DEC confirma e aplica a hierarquia do [[DEC-0010-decisoes-canonicas-fase-1-v1]] §1 e fecha P06 e P08.

## Decisões

### D1: pasta de erros do n8n
`99_ERRORS` (Blueprint §22). `99_ERROR_RECOVERY` (Plano Final §22) é descartado.

### D2: árvore do repositório
Vale a árvore do Blueprint §8 por inteiro; a do Plano Final §23 é descartada.

| Plano Final §23 (descartado) | Blueprint §8 (vigente) |
|---|---|
| `agents/` | `packages/agents/` |
| `workflows/` | `n8n/workflows/` |
| `schemas/*.json` | `packages/schemas/` |
| `playbooks/`, `policies/` | `knowledge/playbooks/`, `knowledge/policies/` |
| `infra/` | `infrastructure/` |
| `prompts/` | **sem equivalente: PENDENTE** |

`prompts/` não será criada agora: os prompts finais dos agentes ainda não existem (DEC-0010) e não se sabe onde moram.

### D3: `n8n/`
As pastas de domínio (`00_SYSTEM` … `99_ERRORS`) ficam dentro de `n8n/workflows/`. `n8n/subworkflows/` guarda só o reutilizável; `n8n/fixtures/` guarda os dados de teste. (Blueprint §8 e §22 descreviam `n8n/` de dois jeitos incompatíveis.)

### D4: fonte única de definição
| Assunto | Fonte única | Observação |
|---|---|---|
| Contratos de API, evento e agente | `packages/schemas/` | |
| Publicar/consumir eventos | `packages/events/` | só código, sem redefinir o contrato |
| DDL | `database/schema/` | só DDL |
| `docs/events/`, `docs/agents/` | referenciam | não redefinem |

### D5: compose
Um único compose de desenvolvimento: `infrastructure/compose/docker-compose.dev.yml`. O `docker-compose.yml` da raiz do starter sai: o `diff` mostrou que os dois arquivos são idênticos. `infrastructure/docker/` guarda só Dockerfiles. A versão fixa do `n8n` (hoje `latest`) fica **PENDENTE** para o M00.

### D6: decisões e conhecimento (fecha a P08)
- `docs/adr/` é a fonte única das decisões, um arquivo por ADR.
- `DECISIONS.md` fica só como nota-índice, apontando para `docs/adr/`, sem texto próprio (sem Maestri, o índice fica em `docs/status/DECISIONS.md`, [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]).
- `docs/decisions`, `docs/DECISIONS.md` (como fonte) e `docs/knowledge` do starter somem.
- Conhecimento do livro: `knowledge/plf/`, **sem o PDF** (DEC-0010 §2).

### D7: guardrails e regras
1. Os guardrails oficiais (DEC-0010 §8) ficam **só** na seção `Guardrails` de cada `.claude/agents/ENG-xx.md`.
2. `.claude/rules/*.md` mantém só regras transversais por tema (architecture, n8n, security, testing), sem repetir guardrail de agente.
3. O `CLAUDE.md` raiz referencia, não copia.
4. `Responsibilities` fica só com responsabilidade.
5. A dispensa de teste que falha (permitida pelo `testing.md` do starter) é registrada em `docs/adr/`. Isso não enfraquece o guardrail do ENG-06: falha não vira sucesso sem evidência.

## Alternativas rejeitadas
- Manter as duas árvores (Plano e Blueprint) lado a lado: duplicidade de pastas e de definições.
- Manter `docs/DECISIONS.md` e `docs/adr/` como fontes paralelas: risco de divergência.
- Guardrail repetido em agents, rules e `CLAUDE.md`: três cópias que divergem.

## Consequências
- Nada do starter foi modificado. Tudo vale **ao importar o starter no M00** ([[starter-zip-inventario]], [[milestones]]).
- PENDENTES: `prompts/` (D2) e versão fixa do `n8n` (D5).
- Se uma fonte única (D4) precisar mudar, registrar nova decisão antes de alterar o contrato.

Ver [[pendencias-de-decisao]] (P06, P08, P09) · [[estrutura-do-repositorio]] · [[n8n-estrutura]] · [[engenharia-eng-00-a-07]].
