---
tipo: indice
area: 03-Arquitetura
---
# Arquitetura

Arquitetura técnica, integrações, dados e infraestrutura. Tudo é **especificação de documentos**, sem código.

## Notas

- [[visao-geral]] — camadas dev/produção e papéis.
- [[modelo-de-entidades]]
- [[launch-state-machine]]
- [[gates]]
- [[catalogo-de-eventos]]
- [[memoria-empresarial]]
- [[permissoes-e-hitl]]
- [[n8n-estrutura]]
- [[estrutura-do-repositorio]] — repo `launch-os` (ainda não criado).
- [[observabilidade-e-testes]]
- [[spec-core-01-servicos-e-limites]] — **M01/F1, bloco 1** (decisões D1.x aprovadas em 2026-10-07): serviços e limites entre a API e o n8n.
- [[spec-core-02-entidades-e-schema]] — **M01/F1, bloco 2** (decisões D2.x aprovadas em 2026-10-07): colunas, chaves e constraints das 22 entidades.
- [[spec-core-03-state-machine-e-gates]] — **M01/F1, bloco 3** (decisões D3.x aprovadas em 2026-10-07): transições, mapeamento estado → gate, aprovadores, `owner_agent` e decisão de continuidade.
- [[spec-core-04-eventos]] — **M01/F1, bloco 4** (decisões D4.x aprovadas em 2026-10-07): payloads, entrega confiável, idempotência, retenção, eventos faltantes e papel do Redis.
- [[spec-core-05-permissoes-e-aprovacoes]] — **M01/F1, bloco 5** (decisões D5.x aprovadas em 2026-10-07): autenticação, permissões, aprovações (prazo e `approval.expired`), `BUDGET_AUTHORIZED` e auditoria de segurança.
- [[spec-core-06-ferramentas-de-codigo]] — **M01/F1, bloco 6** (decisões D6.x aprovadas em 2026-10-07; F1 em revisão independente): ADRs das ferramentas (Node, pnpm, TypeScript, Fastify, Zod, `pg`, migrations, testes, logs, segurança) e fechamento da F1.

Voltar: [[00-LEIA-PRIMEIRO]]
