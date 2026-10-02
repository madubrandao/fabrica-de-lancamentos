---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0010 §5–§6 (canônico); blueprint v1 §15, §27, §28; PLANO FINAL §14–§15, §27
status: canonico
atualizado: 2026-10-02
---
# Launch State Machine

Canônico: [[DEC-0010-decisoes-canonicas-fase-1-v1]] §5–§6. O lançamento **não pula de estado sem cumprir o gate** ([[gates]]).

## `launch.status` — somente estados de execução
```text
IDEA → RESEARCH → VALIDATION → OFFER_DRAFT → PRODUCT_BUILD → PRE_PRE → PLC
→ LAUNCH_READY → CART_OPEN → CART_CLOSING → CART_CLOSED → POST_LAUNCH
→ ANALYSIS → LEARNING → DECISION
```

## `launch.type` — tipo do lançamento
`SEED · INTERNAL · JOINT · RELAUNCH · EVERGREEN`

**Não misturar `launch.status` com `launch.type`.** Seed/Internal/Joint/Relaunch/Evergreen são tipo ou próxima ação, nunca estados do mesmo ciclo.

## Depois de `DECISION`: decisão de continuidade
O sistema cria uma decisão de continuidade:
| Decisão | Efeito |
|---|---|
| `SEED` | criar novo Launch do tipo SEED |
| `INTERNAL` | criar novo Launch do tipo INTERNAL |
| `JOINT` | criar novo Launch do tipo JOINT |
| `RELAUNCH` | criar novo ciclo de lançamento da mesma oferta |
| `EVERGREEN` | ativar/encaminhar para uma sequência perene |
| `ARCHIVE` | encerrar o ciclo sem próximo lançamento |

Cada ciclo é, portanto, um Launch próprio que percorre os 15 estados; a continuidade liga um Launch ao seguinte.

## Significado dos tipos (nos docs)
SEED: começar sem lista/produto e testar a ideia · INTERNAL: validação · JOINT: expansão (listas de parceiros) · RELAUNCH · EVERGREEN: sequência acionada conforme novos leads entram.

## Circle of Awesome (capability nativa)
`IDEA → SEED → PRODUCT → INTERNAL → DATA/PROOF → JOINT → NEW IDEAS → SEED` — realizado pela cadeia de decisões de continuidade acima, não por estados extras.

## Evergreen (Blueprint §28)
`lead.created → segment → eligibility → evergreen_sequence → content progression → offer → conversion → analytics`

## Pendente (especificar em M01)
- Tabela de transições permitidas e eventos emitidos por transição.
- Mapeamento estado → gate.
- `owner_agent` por estado.
- Modelo da entidade "decisão de continuidade" (campos, vínculo com `Decision` e `Launch`). **Não inventado aqui.**
