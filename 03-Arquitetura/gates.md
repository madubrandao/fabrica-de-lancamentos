---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0010 §3 (canônico); blueprint v1 §21; PLANO FINAL §20 (histórico)
status: canonico
atualizado: 2026-10-02
---
# Gates de aprovação

Canônico: [[DEC-0010-decisoes-canonicas-fase-1-v1]] §3. Regra: nenhuma transição crítica ocorre sem o gate correspondente. Gates são regras do sistema, não checklist de documentação.

## 12 gates operacionais
| ID | Gate |
|---|---|
| G01 | Market Validated |
| G02 | Offer Approved |
| G03 | Product Ready |
| G04 | Content/PLC Approved |
| G05 | CRM Ready |
| G06 | Technical QA Passed |
| G07 | Launch Ready |
| G08 | Cart Open Authorized |
| G09 | Cart Close Authorized |
| G10 | Post Launch Complete |
| G11 | Analysis Complete |
| G12 | Learning Recorded |

## Orçamento NÃO é gate
`Budget Approved` não é um 13º gate. É uma **pré-condição / policy check** antes de qualquer execução que envolva gasto:

`BUDGET_AUTHORIZED = true`

- Pode ser exigido em qualquer milestone.
- É registrado em audit log.
- Preserva a exigência do Plano Final sem duplicar o estado do lançamento.
- Gasto acima do limite continua sendo HUMAN DECISION ([[permissoes-e-hitl]]); o valor do limite segue **PENDENTE** ([[perguntas-em-aberto]]).

## Regras adicionais
- Agente nunca contorna approval gate nem altera aprovações por mensagem (guardrail ENG-04).
- Antes de abrir o carrinho: QA de sales page, links, order form, checkout, thank-you page, confirmação, CRM e tracking ([[n8n-estrutura]]).
- Regra do livro como gate: Joint Launch só após Internal Launch ter testado oferta e sequência.

## Pendente
Quem aprova cada gate (humano, A0 ou ambos), critérios objetivos de aprovação e o mapeamento estado → gate (os docs listam os dois, não os ligam). A especificar em M01.

(A lista de 10 gates do Plano Final está superada.)
