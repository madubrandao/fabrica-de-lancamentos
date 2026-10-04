---
tipo: contexto
area: 01-Contexto
fonte: FASE  1 - BASE/blueprint_empresa_lancamentos_autonoma_maestri_v1.md (§0, §46); PLANO FINAL DE ENGENHARIA.txt (§1, §30); MAPA DE EXTRAÇÃO…txt
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# Visão do projeto

## O que é
**Empresa de Lançamentos Autônoma**, também chamada de **LAUNCH OS** (Launch Operating System): uma empresa de lançamentos digitais cuja operação de negócio é executada por agentes de IA, workflows e automação, com supervisão humana nas decisões críticas.

A metodologia de lançamento vem do livro *A Fórmula do Lançamento / Product Launch Formula* (Jeff Walker). A **arquitetura de software é engenharia nossa** sobre essa metodologia. Ver [[escopo-e-principios]] (seção "Livro x engenharia nossa").

## Como a ideia foi construída (cadeia)
`Livro → Metodologia → Processos → Funções → Profissionais → Agentes de IA → Workflows → Empresa`
Primeiro se descobrem os trabalhos necessários, depois se decide quantos agentes existem (não o contrário).

## Decisão central
Construir **já com arquitetura de escala**, sem MVP descartável. A arquitetura-alvo é construída completa e a capacidade é ativada gradualmente ([[DEC-0001-arquitetura-escalavel-desde-o-inicio]]).

## Arquitetura em uma frase
16 funções humanas de referência → **11 agentes** (A0–A10) → workflows por agente → memória empresarial central → eventos → n8n → APIs/LLMs → ciclo contínuo de lançamento e aprendizado.

## Duas camadas (não confundir)
| Camada | Quem | Papel |
|---|---|---|
| Desenvolvimento | Claude Code com 8 subagentes (ENG-00…07) | CONSTROEM a empresa |
| Produção | n8n (orquestra e executa os agentes) + API fina + PostgreSQL + Redis + LLMs; agentes A0–A10 | OPERAM a empresa |

O desenvolvimento (Claude Code) pode ser desligado sem derrubar a produção. O Maestri saiu do escopo ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]). Ver [[visao-geral]].

## Primeiro grande objetivo técnico
> Cadastrar um produto e conseguir gerar e executar um lançamento inteiro através do sistema.

Critério de sucesso da v1: [[criterio-de-sucesso-v1]].

## Autoridade
O humano (Madu) é a autoridade máxima nas decisões críticas ([[permissoes-e-hitl]]).

## O que este documento NÃO define
Produto real e nicho (o primeiro ciclo usa um produto fictício de teste, [[DEC-0017-modelo-de-negocio-inicial]]), valores de orçamento (antes do M04) e as plataformas reais de integração. Provedor de LLM, stack e hospedagem de dev estão no [[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]]. Ver [[perguntas-em-aberto]].
