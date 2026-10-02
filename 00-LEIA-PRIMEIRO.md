---
tipo: indice
projeto: fabrica-de-lancamentos
criado: 2026-10-02
atualizado: 2026-10-02
---
# Fábrica de Lançamentos — Memória do Projeto

Vault Obsidian com a memória do projeto. Estrutura adaptada do padrão BRAVY SCOOL: pastas numeradas por área, README em cada nível e este índice na raiz.

**O projeto em uma frase:** LAUNCH OS — uma empresa de lançamentos autônoma baseada na metodologia de *A Fórmula do Lançamento*, operada por 11 agentes de IA (A0–A10) sobre backend + n8n + PostgreSQL + Redis, construída por Claude Codes coordenados no Maestri, com humano nas decisões críticas. Detalhe em [[visao-do-projeto]].

**Estado:** FASE 1 - BASE (documentação e arquitetura), sem código. Decisões canônicas v1 aprovadas. Próximo marco autorizado: **M00 Foundation** (início formal pendente). Ver [[2026-10-02-leitura-fase-1]].

## Áreas

- [[01-Contexto/README|01-Contexto]] — Visão do projeto, objetivo, público, regras de negócio e glossário.
- [[02-Decisoes/README|02-Decisoes]] — Registro de decisões (ADRs): contexto, escolha, alternativas e consequências.
- [[03-Arquitetura/README|03-Arquitetura]] — Arquitetura técnica, integrações, dados e infraestrutura.
- [[04-Fases/README|04-Fases]] — Plano e status por fase do projeto (a FASE 1 - BASE já existe na raiz).
- [[05-Agentes/README|05-Agentes]] — Agentes do projeto: função, entradas, saídas e limites.
- [[06-Skills/README|06-Skills]] — Skills do projeto: uma pasta numerada por skill, com SKILL.md.
- [[07-Log/README|07-Log]] — Diário de sessões: o que foi feito, evidências, pendências.
- [[08-Referencias/README|08-Referencias]] — Links, materiais externos e fontes de consulta.

- `FASE  1 - BASE/` — material base original (blueprint, plano de engenharia, mapa de extração, starter zip). **Fonte da verdade; não editar.** Resumo em [[fontes-da-fase-1]].

## Comece por aqui
0. **[[DEC-0010-decisoes-canonicas-fase-1-v1]]** — fonte canônica (maior autoridade). Em conflito, vale ela.
1. [[visao-do-projeto]] → [[escopo-e-principios]]
2. [[visao-geral]] (arquitetura) → [[launch-state-machine]] → [[05-Agentes/README|agentes]]
3. [[blueprint-de-execucao]] (os 15 passos) → [[roadmap]]
4. **Pendências que precisam da Madu:** [[pendencias-de-decisao]] e [[perguntas-em-aberto]]

## Regras

- Fluxo: Diagnóstico → Evidências → Planejamento → Implementação → Testes → Validação → Documentação.
- Nada é assumido sem evidência; mudanças são confirmadas com a Madu antes.
- Toda decisão relevante vira nota em 02-Decisoes; toda sessão vira nota em 07-Log.
- Nunca versionar segredos (tokens, senhas, .env).
- Tudo que os documentos não definem fica marcado **PENDENTE**; nada é inventado (DEC-0010 §9).
- Hierarquia: decisões em `02-Decisoes` > Blueprint > arquitetura/roadmap derivados > starter > rascunhos.
- Nenhum código sem iniciar o M00 formalmente (DEC-0010 §10).
- Distinguir sempre: **derivado do livro** x **engenharia nossa**; **agentes A0–A10 (produção)** x **ENG-00…07 (desenvolvimento)**.
