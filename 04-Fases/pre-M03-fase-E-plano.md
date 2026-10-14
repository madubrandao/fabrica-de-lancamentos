---
tipo: fase
area: 04-Fases
status: RASCUNHO em 2026-10-11 (não é o início formal do M03; a Madu ainda não autorizou o M03)
fonte: DEC-0034 (§2 perfil local, §3 políticas, §5 fase E); milestones.md (M03); BLOCKERS do launch-os
atualizado: 2026-10-11
---
# Fase E: início formal do M03 (rascunho)

**Este documento não inicia o M03.** O M03 só começa com um novo início formal da Madu, com plano único e as mesmas paradas dos marcos anteriores. Aqui fica a lista fechada do que o início formal precisa, para ela decidir de uma vez.

## O que o M03 valida (milestones.md)
A0 a A4 e o **A6 mínimo** (só o necessário para o Create Launch; definição **PENDENTE**). Objetivo: produzir um lançamento até `LAUNCH_READY`, no primeiro workflow integrado (Create Launch). Pré-requisito: produto fictício de teste (DEC-0017 C1). Integrações simuladas (DEC-0017 C5).

## Pré-condições (DEC-0034 §5)
| Fase | Situação em 2026-10-11 |
|---|---|
| A, B, C | Concluídas |
| D | D1 e D2 mesclados; **D3 e D4 não implementados** em 2026-10-14. Sequência e requisitos por modo (M03 local, M03 em staging, production): [[DEC-0037-sequencia-d4-d3-backup-e-retencao]] §6 (ACEITA em 2026-10-14); plano em [[pre-M03-d4-d3-plano-unificado]] |
| E | Este rascunho |
| F (hospedagem, TLS, imagem da API, rate limit, segundo fator, notificação) | Passou para **antes do M04**; ~~o M03 roda local~~ **[SUPERADO em 2026-10-14: o ambiente do M03 (local ou staging) será definido na autorização formal do M03; DEC-0037 §2]** |
| **Onde o M03 roda** | **Definido na autorização formal do M03:** o DEC-0034 dizia local; o DEC-0035 antecipou a hospedagem sem revogar nem confirmar essa frase, e o DEC-0037 §2 a considera superada. A autorização formal tem de dizer "local" ou "staging" |

## O que só a Madu fornece (nenhum valor é inventado)
| Item | O que falta | Referência |
|---|---|---|
| Chave Anthropic | Criar num workspace próprio do Console, com limite de gasto no Console se a conta permitir | DEC-0034 §3 |
| `llm_pricing` | Copiar da tabela oficial da Anthropic **no dia da configuração** (preço por milhão ÷ 1.000.000). Eu não preencho de memória | DEC-0034 §3 |
| Produto fictício de teste | Nome, promessa e público do produto usado no primeiro lançamento | DEC-0017 C1 (PEND) |
| Aceite dos valores já decididos | `llm_cost_limit` US$ 20 acumulados; modelo único `claude-sonnet-5-5`; `approval_expiry` 72 h; `budget_limit` 0 BRL | DEC-0034 §3 (aceitos em 2026-10-09; confirmar que seguem valendo) |
| Gates | Todos por aprovação humana da Madu. Critérios objetivos de G01 a G07 escritos por agente dentro do M03 | DEC-0034 §4 |

## O que eu preparo e ela aprova (proposta, nada implementado)
1. **Escopos e chaves de memória por agente** (A0 a A4 e A6 mínimo), com mínimo privilégio. Até lá `memory.write` não grava nada, por desenho (`packages/agents/src/memory.ts:39`).
2. **Definição do A6 mínimo** (hoje PENDENTE) e do conteúdo do `prompts/`.
3. **Critérios objetivos de G01 a G07**, um checklist por gate, com aprovação dela.
4. **Plano único do M03** no formato dos planos M01 e M02: fases, paradas, evidências esperadas, critério de fechamento (DEC próprio).
5. **Regra de custo:** sem chamada real a LLM antes do início formal; adaptador de teste até lá.

## Paradas
- O início formal é decisão da Madu (como DEC-0021, DEC-0029, DEC-0032 e DEC-0030 para o M02).
- Primeira chamada real a LLM só depois de chave, `llm_cost_limit`, `llm_pricing` e produto fictício estarem registrados.
- Cada PR: CI verde, revisão independente registrada e merge só com a autorização dela na hora.

## Lacunas conhecidas que este rascunho não resolve
- Texto do A6 mínimo e escopos reais dos agentes (dependem da especificação de cada agente).
- Política de orçamento completa e escopos de produção: antes do M04.
- Cobertura de testes: o piso de 85% de linhas e 80% de ramos está decidido, mas ainda não aplicado na CI (etapa 5 do plano de retomada).
