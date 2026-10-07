---
tipo: decisao
area: 02-Decisoes
id: DEC-0023
status: PROPOSTA — aguardando a decisão da Madu (não vale até ela aprovar)
fonte: M01-plano-de-execucao (F1 e portão F1); DEC-0022 regra 1; blocos 1 a 6 e fechamento em 03-Arquitetura (spec-core-01 a 07); revisão independente em duas passadas (2026-10-07)
data-registro: 2026-10-07
---
# DEC-0023 — Aprovação do portão F1: especificação técnica do Core (M01)

> **Este é um rascunho para a Madu decidir.** Enquanto o status for PROPOSTA, **continua proibido** criar código de aplicação, schema ou migration (DEC-0022, regra 1). Ao aprovar, o status passa a `aceita` e a data da decisão é registrada.

## Contexto
O [[DEC-0022-inicio-formal-do-m01]] determina que nenhum código, schema ou migration seja criado antes de a Madu aprovar a especificação (portão F1). A F1 do [[M01-plano-de-execucao]] produziu a especificação em seis blocos mais um documento de fechamento, todos em `03-Arquitetura/`:

| Bloco | Documento | Decisões |
|---|---|---|
| 1 | [[spec-core-01-servicos-e-limites]] | D1.1 a D1.5, aprovadas em 2026-10-07 |
| 2 | [[spec-core-02-entidades-e-schema]] | D2.1 a D2.8, aprovadas em 2026-10-07 |
| 3 | [[spec-core-03-state-machine-e-gates]] | D3.1 a D3.8, aprovadas em 2026-10-07 |
| 4 | [[spec-core-04-eventos]] | D4.1 a D4.8, aprovadas em 2026-10-07 |
| 5 | [[spec-core-05-permissoes-e-aprovacoes]] | D5.1 a D5.10, aprovadas em 2026-10-07 |
| 6 | [[spec-core-06-ferramentas-de-codigo]] | D6.1 a D6.10, aprovadas em 2026-10-07 |
| Fechamento | [[spec-core-07-revisao-independente-f1]] | D7.1 a D7.7 e os ajustes da segunda passada, aprovados em 2026-10-07 |

## Evidências de verificação
- **Revisão independente em duas passadas** por um agente revisor diferente do autor, em modo de leitura, sobre os blocos 1 a 6 (2026-10-07). Primeira passada: 0 críticos, 6 altos, 13 médios, 7 baixos. Segunda passada, no commit `f45dc08`: 0 críticos e 0 altos; os 6 altos e os 13 médios estavam resolvidos; as 10 observações restantes (4 médias) foram aplicadas depois. Detalhe em [[spec-core-07-revisao-independente-f1]].
- Links internos conferidos por script em todos os blocos (nenhum quebrado).
- Versões das ferramentas conferidas no registro do npm e no nodejs.org em 2026-10-07 (bloco 6); uma incompatibilidade real encontrada (TypeScript 7 sem suporte no `typescript-eslint`) e tratada na decisão.
- **Limite da verificação:** a revisão foi sobre texto. Nenhum código, schema ou migration existe, então nada foi executado. A correção da especificação será exercitada pelos testes da F2 a F7.

## Decisão proposta
1. **Aprovar** a especificação técnica do Core dos blocos 1 a 6 e do fechamento (documentos da tabela acima) como base do M01, **com as decisões D1.1 a D7.7 já aprovadas**.
2. **Liberar**, a partir da aprovação, a criação de código, schema e migrations do M01, **só** dentro do que a especificação define, **em branches `feat/*`** e **pelas fases do plano**: F2 (banco e schema) e F6 (infra e CI) em paralelo, depois F3, F4, F5, F7 e F8, cada uma com revisão por outro agente antes do merge.
3. **Continuam fora do M01** (não autorizados): execução de agentes, tool registry e montagem de contexto (M02), agentes de negócio (M03 em diante), dashboard (M06), adaptador de LLM, vector store e hospedagem de teste e produção.
4. **Registrar os ADRs** do bloco 6 em `docs/adr/` do repositório `launch-os` e atualizar o `docs/status/` de lá. É **ação externa em outro repositório**: só com a confirmação explícita da Madu no momento.
5. **Nada é instalado antes de cada fase começar.** Em especial, a versão exata do Node e do pnpm só é fixada na F6, e a decisão de atualizar o Node local (24.16.0 hoje; a LTS atual é a 24.21.0) é da Madu.

## O que continua PENDENTE (não bloqueia o início da F2 e da F6, mas bloqueia o que depende deles)
- **Valores definidos pela Madu:** prazo de aprovação, validade do claim, tentativas, esperas e timeout de entrega, validade da chave de idempotência, prazo de limpeza das entregas, duração da sessão e limites de login, limite de orçamento e moeda (antes do M04).
- **Critérios objetivos dos gates** e o resultado do QA do G06 (M03). Enquanto isso, os gates sem critério são aprovação da Madu.
- **Escopos de leitura, escrita e ações dos 11 agentes** (M02 e M03).
- **Canal de notificação** à Madu (e-mail ou mensageria).
- **Segundo fator** e reautenticação para aprovação crítica.
- **Versão major do pnpm (11 ou 12)** e versões exatas das bibliotecas (F6).
- Colunas de negócio das 14 entidades de domínio (a partir do M03) e a definição do produto fictício (antes do M03).

## Consequências, se aprovado
- O estado do projeto passa de "M01 na F1 (especificação)" para "M01 liberado para a F2 e a F6".
- Os documentos [[milestones]], [[roadmap]] e [[M01-plano-de-execucao]] são atualizados para refletir isso **depois** da aprovação.
- Mudar qualquer decisão da especificação depois disso exige decisão registrada (ADR ou DEC), e o ENG-00 não aprova a própria mudança crítica sem revisão apropriada (guardrail do DEC-0010 §8).

## Alternativas
- **Aprovar com ressalvas:** a Madu lista o que quer mudar; o ENG-00 ajusta, o revisor confere os pontos alterados e o DEC é aprovado depois.
- **Não aprovar:** o código continua proibido e a F1 segue aberta.

Ver [[DEC-0022-inicio-formal-do-m01]] · [[M01-plano-de-execucao]] · [[spec-core-07-revisao-independente-f1]].
