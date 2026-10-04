---
tipo: indice
area: 02-Decisoes
---
# Decisoes

Registro de decisões (ADRs): contexto, escolha, alternativas e consequências.

## Notas

- [[DEC-0001-arquitetura-escalavel-desde-o-inicio]]
- [[DEC-0002-duas-camadas-maestri-so-desenvolvimento]]
- [[DEC-0003-16-funcoes-em-11-agentes]]
- [[DEC-0004-integracao-orientada-a-eventos]]
- [[DEC-0005-stack-de-producao]]
- [[DEC-0006-state-machine-e-gates]]
- [[DEC-0007-human-in-the-loop-tres-niveis]]
- [[DEC-0008-caminho-critico-primeiro]]
- [[DEC-0009-memoria-empresarial-em-tres-tipos]]
- [[DEC-0010-decisoes-canonicas-fase-1-v1]] — **canônico, nível 1 de autoridade.** Resolve gates, roadmap M00–M06, state machine, guardrails de ENG-00…07 e a regra do PDF.
- [[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]] — padrão único `approval.requested/granted/rejected`; resolve P04.
- [[DEC-0012-lista-de-entidades-do-blueprint]] — lista oficial de 22 entidades (Blueprint §14); resolve P05.
- [[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] — árvore do Blueprint §8, `99_ERRORS`, fonte única por assunto, guardrails só nos agentes; resolve P06 e P08.
- [[DEC-0014-notas-pinadas-no-maestri]] — 6 notas pinadas no `00-COMMAND-CENTER`; resolve P07.
- [[DEC-0015-alocacao-do-roadmap-m01-m04]] — M04 valida A5–A10 ao final, A6 mínimo no M03, `CORE v0.1` = M01+M02+M03, API skeleton e event schema no M01; resolve P10, P11 e P12.
- [[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] — TypeScript/Node, LLM via adaptador (Anthropic primeiro), dev em Docker local, Windows nativo, repo privado; adia vector store, estado→gate e limites numéricos.
- [[DEC-0017-modelo-de-negocio-inicial]] — produto fictício de teste, só lançamentos próprios, capacidade de 1 lançamento por vez, orçamento antes do M04, plataformas simuladas; resolve o bloco C.
- [[pendencias-de-decisao]] — conflitos entre documentos: **P01–P12 todos resolvidos** (P09 mantém o pin do `n8n` aberto).

Status "aceita-nas-fontes" = a decisão consta nos documentos da FASE 1; DEC-0010 prevalece em qualquer conflito.

Voltar: [[00-LEIA-PRIMEIRO]]
