---
tipo: decisao
area: 02-Decisoes
id: DEC-0033
status: aceita
fonte: DEC-0029 (pendências do fechamento do M01); DEC-0032 (revisão do M02, achados médios de 2026-10-08)
data-registro: 2026-10-09
data-decisao: 2026-10-09 (a Madu aprovou seguindo as recomendações: "Pode seguir com os recomendados e o que for decisão exclusiva minha")
---
# DEC-0033 — Decisões pendentes do M01 (recomendações aprovadas)

> **Aceita pela Madu em 2026-10-09.** Fecha as decisões do DEC-0029 que cabiam a ela com recomendação. Não muda o M03, que segue **não autorizado**.

## Decisões
| # | Pendência | Decisão | Efeito |
|---|---|---|---|
| 1 | A4: imagem Docker da API | Critério do M01 alterado para "API roda e é verificada pela fumaça". A imagem Docker sai com a hospedagem, antes do M03 | `milestones.md` e `BLOCKERS` do `launch-os` atualizados |
| 2 | OpenAPI não gerado | Adiado até haver cliente externo | Pendência registrada, sem bloquear o M03 |
| 3 | Node local (24.16.0) contra o 24.21.0 da CI | Atualizar a máquina para 24.21.0 | **Ação pendente:** a instalação exige elevação de privilégio e não foi executada nesta data |
| 4 | Critérios objetivos dos gates G01 a G11 | Definir antes do M03 | Conteúdo é decisão exclusiva da Madu (ver lista abaixo) |
| 5 | Canal de notificação e segundo fator | Adiados; definir antes de expor a API fora de `127.0.0.1` | Pendência registrada |
| 6 | Retenção do `audit_log` | Partição, em vez de arquivamento | Implementação pré-M03, em plano próprio |

## Pendências que são exclusivamente da Madu
- **Valores (sem padrão no código):** duração da sessão (`AUTH_SESSION_TTL_SECONDS`); intervalo e lote do agendador (`SCHEDULER_*`); prazo de aprovação (`approval_expiry`); limite de orçamento e moeda (`budget_limit`); limite de tentativas de login; os seis valores de entrega (`DELIVERY_*`) e a validade do claim; validade das chaves de idempotência e prazo de limpeza das entregas; limite mínimo de cobertura.
- **Conteúdo dos gates G01 a G11:** os critérios objetivos (item 4).
- **Canal de notificação e segundo fator** (item 5).
- **Escopos reais dos 11 agentes** (M02/M03, já registrados).

## Já fechado (verificado no código)
- `JWT_SECRET` com tamanho mínimo e recusa de `CHANGE_ME` (`apps/api/src/config.ts`).
- Migration 0009 (parâmetros legíveis da aprovação de ação), criada na F2 do M02.
- M1 (n8n emite evento pela API), M2 (`error.detected` de origem `api` e `workflow`) e M5 (papel do Redis, sem papel no M02): fechados no M02, conforme `docs/status/CURRENT_MILESTONE.md` (linha 18).
- M3 (`next_launch_id: null` em `decision.recorded`): não consta como fechado no M02. Recomendação do DEC-0029 era aceitar como está, sem consumidor identificado; registrado como aceito nesta decisão, sujeito à confirmação da Madu.

## Pendências de segurança antes de expor a API (antes do M03, sem mudança)
Rate limit e bloqueio de login; troca de senha e `logout`; backup e restore testados, com `N8N_ENCRYPTION_KEY` guardada fora da máquina; rotação de `JWT_SECRET` e das senhas dos papéis; retirar `CREATEDB` do migrador; fixar imagens por digest; escopo do `.env` da API; retenção do `audit_log` (item 6); `claimEvent` e `ackEvent` com filtro de organização, só se houver mais de uma organização.

Ver [[DEC-0029-fechamento-do-m01-core-platform]] · [[DEC-0032-fechamento-do-m02-agent-engine]] · [[milestones]].
