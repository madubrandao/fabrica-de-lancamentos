---
tipo: decisao
area: 02-Decisoes
id: DEC-0018
status: aceita
fonte: decisão da Madu em 2026-10-04; DEC-0010 §7 e §10
data-registro: 2026-10-04
---
# DEC-0018 — Início formal do M00 Foundation

> **Marco concluído em 2026-10-04:** [[DEC-0021-m00-foundation-concluido]].

## Contexto
O [[DEC-0010-decisoes-canonicas-fase-1-v1]] §10 proíbe qualquer código ou repositório de código sem o início formal do M00. Em 2026-10-04 todas as pendências entre documentos (P01–P12) e os blocos técnico e de negócio estavam resolvidos ([[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]] a [[DEC-0017-modelo-de-negocio-inicial]]). Restava o bloco B de [[perguntas-em-aberto]] (material faltando), que não trava o M00.

## Decisão
A Madu autorizou, em 2026-10-04, o **início formal do M00 Foundation**.

**Escopo autorizado (DEC-0010 §7):** fundação documental, repositório, Claude Code e infraestrutura de desenvolvimento (o Maestri saiu do escopo em [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]). Itens em [[milestones]] e [[M00-plano-de-execucao]].

**Fora do escopo:** qualquer código de aplicação, schema, migration, evento, agente ou workflow. Isso é M01 em diante. O M00 não autoriza o M01.

## Regras do M00
- Cada ação de execução é confirmada com a Madu antes de rodar (fluxo: Diagnóstico → Evidências → Planejamento → Implementação → Testes → Validação → Documentação).
- Ações externas ou difíceis de desfazer (criar repositório no GitHub, push) só com confirmação explícita.
- Nenhum segredo é versionado.
- O M00 só fecha com o critério de [[definicao-de-pronto]] cumprido e evidência de verificação.

## Consequências
- O primeiro passo de execução é o passo 1 do [[blueprint-de-execucao]]: criar o repositório `launch-os`, conforme DEC-0016 D6.
- O estado do projeto passa de "FASE 1 - BASE (sem código)" para "M00 em andamento".
- **PENDENTES** do M00 listados em [[M00-plano-de-execucao]].

Ver [[roadmap]] · [[milestones]].
