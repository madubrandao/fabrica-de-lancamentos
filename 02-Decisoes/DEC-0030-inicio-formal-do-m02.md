---
tipo: decisao
area: 02-Decisoes
id: DEC-0030
status: proposta
fonte: M02-plano-de-execucao; plano aprovado pela Madu em 2026-10-08; DEC-0029 (M02 não autorizado até novo início formal)
data-registro: 2026-10-08
data-decisao: aguardando a Madu ("iniciar M02")
---
# DEC-0030 — Início formal do M02 Agent Engine

> **PROPOSTA.** O M02 está **planejado, não iniciado**. Este DEC só vale quando a Madu disser "iniciar M02". Até lá: nenhum código, nenhuma branch, nenhum Docker.

## Contexto
O M01 foi fechado em 2026-10-08 ([[DEC-0029-fechamento-do-m01-core-platform]]), que deixou claro que o M02 só começa com um novo início formal, como o M01 ([[DEC-0022-inicio-formal-do-m01]]) e o M00 ([[DEC-0018-inicio-formal-do-m00]]). O plano está em [[M02-plano-de-execucao]].

## O que a Madu autoriza ao dizer "iniciar M02"
1. A **F0** (início formal) e a **F1** (especificação, com portão): spike do A5 no n8n 2.41.6 em pilha isolada e os blocos da especificação com as decisões D1–D11. **Nenhum código de aplicação antes de ela aprovar a especificação.**
2. Cada fase seguinte (F2 a F7) com **um plano único por fase**, as mesmas paradas do M01 e revisão independente antes de cada merge.
3. O escopo é o do [[M02-plano-de-execucao]]: o motor de execução e os agentes de **referência**, não os agentes de negócio (M03).

## Pré-condições (a Madu cuida)
- Docker ligado e **containers dos outros projetos parados** (eu só mexo no `launch-os` e na pilha isolada); pelo menos **20 GB livres** em `C:`.
- **Chamada real a um LLM** só com a **chave Anthropic dela e um limite de custo definido por ela**; sem isso, o adaptador de teste.
- Valores (limite de custo de LLM, modelo por agente, timeouts, retries, limites de contexto) e escopos reais dos agentes: **PENDENTES**, sem padrão no código.

## Antes do M03 (não são do M02, mas travam o M03 e a produção)
Rate limit e bloqueio de login, retenção do `audit_log`, troca de senha, backup e restore, hospedagem de teste e produção, imagem Docker da API (A4), produto fictício de teste e definição do A6 mínimo.

## Decisão
Aguardando a Madu: dizer **"iniciar M02"** (com ou sem ajustes ao plano).

Ver [[M02-plano-de-execucao]] · [[milestones]] · [[roadmap]].
