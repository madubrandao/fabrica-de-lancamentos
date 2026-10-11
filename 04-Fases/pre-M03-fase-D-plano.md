---
tipo: fase
area: 04-Fases
status: PROPOSTO em 2026-10-11 (D1 e D2 já mesclados; D3 e D4 aguardam aprovação da Madu)
fonte: DEC-0033 (item 6); DEC-0034 (perfil de valores e fase D); PRs #35 e #36 do launch-os; leitura de `database/migrations/0006` e `docs/operations/M01-RUNBOOK.md` em 2026-10-11
atualizado: 2026-10-11
---
# Fase D antes do M03: operação local

Fonte da decisão: DEC-0034 (fase D = comando de importação, partição do `audit_log`, limpeza agendada, backup e restore testados com a `N8N_ENCRYPTION_KEY`). Este documento registra o que já foi feito e propõe o que falta. **Nada de D3 e D4 foi implementado.**

## Estado (2026-10-11)
| Item | Situação | Evidência |
|---|---|---|
| D1. Comando de importação dos workflows (`pnpm n8n:import`) | **Feito**, PR #35 mesclado com autorização da Madu | CI verde; revisão independente "aprovar com ressalvas", 4 achados tratados |
| D2. Limpeza agendada (`idempotency_key` 7 dias, `event_delivery` tratada 30 dias) | **Feito**, PR #36 mesclado com autorização da Madu | CI verde; revisão independente "aprovar com ressalvas", achado do teto do intervalo corrigido |
| D3. Partição mensal do `audit_log` | **Proposto** | abaixo |
| D4. Backup e restore testados | **Proposto** | abaixo |
| Fechamento da fase (docs, `CURRENT_TASK`, runbook) | Depois de D3 e D4 | |

`main` do launch-os em `0958649`. Fumaças M01 e M02 **não foram rodadas depois do merge** (Docker desligado em 2026-10-11).

## Evidências lidas no código (D3)
- `audit_log` (migration `0006`): `id uuid PRIMARY KEY`, `occurred_at timestamptz NOT NULL DEFAULT now()`, dois índices (`correlation_id`; `entity_type, entity_id, occurred_at` parcial), triggers `forbid_change` contra UPDATE, DELETE e TRUNCATE, e `REVOKE UPDATE, DELETE, TRUNCATE` do `launch_os_app`.
- **Nenhuma tabela referencia `audit_log`** (busca por `REFERENCES audit_log` em `database/`: zero resultados). A tabela tem FK para `organization` e `event`.
- A aplicação só grava por `INSERT` (`packages/events/src/db.ts:54-57`).
- O PostgreSQL exige a chave de partição na chave primária de tabela particionada. Logo `PRIMARY KEY (id)` precisa virar `(id, occurred_at)`.

## D3. Partição mensal do `audit_log`
Decisão já tomada (DEC-0034): partição mensal; 24 meses no banco; partição mais antiga exportada para backup antes de ser destacada; **nada é apagado sem decisão da Madu**.

**Proposta (PROP):**
1. Migration que converte `audit_log` em tabela particionada por `RANGE (occurred_at)`, com PK `(id, occurred_at)`, recriando índices e as duas travas (linha e TRUNCATE) em cada partição e o `REVOKE`. Preserva os dados existentes.
2. Criação automática das partições dos meses seguintes (a etapa de manutenção do D2 é o lugar natural), mais uma partição `DEFAULT` para nunca recusar uma gravação.
3. Comando manual (`pnpm`) que **exporta** a partição mais antiga que 24 meses e **só lista** o que poderia ser destacado. Destacar ou apagar fica fora do automático.

**PEND (decisão da Madu, sem padrão no código):**
- Se a partição `DEFAULT` entra ou se uma gravação sem partição deve falhar (mais seguro para a integridade, pior para a disponibilidade).
- Formato e destino da exportação (arquivo local em qual pasta).
- Quantos meses futuros criar de antemão.

**Riscos a tratar no plano de testes:** conversão com dados reais (tempo e bloqueio); trigger de imutabilidade em partição; `down` da migration; o teste de integração que prova que o `launch_os_app` não consegue apagar nada continua valendo.

## D4. Backup e restore
Estado hoje (`M01-RUNBOOK.md:76`): **não documentado nem testado**. Decisão do DEC-0034: `N8N_ENCRYPTION_KEY` guardada no gerenciador de senhas da Madu, com uma cópia offline; restore testado.

**Proposta (PROP):**
1. Script de backup do PostgreSQL (banco da plataforma e banco do n8n) e do volume do n8n necessário, com a lista exata do que entra.
2. Script de restore para uma pilha isolada e vazia, mais uma verificação objetiva (contagens por tabela, migrations aplicadas, workflow do n8n abre as credenciais com a chave).
3. Teste de ponta a ponta: backup, destruir a pilha isolada, restaurar, rodar a fumaça M02.
4. Runbook atualizado, com o passo manual da chave.

**PEND (decisão da Madu):** onde ficam os arquivos de backup (pasta e periodicidade); se o backup é automático ou só sob comando; retenção dos backups. O código não terá valor padrão.

**Limite honesto:** o restore só é "testado" quando eu o executar contra uma pilha isolada com o Docker ligado. A guarda da chave fora da máquina é ação da Madu.

## Ordem e paradas
D3 → D4 → fechamento. Um plano e uma aprovação para os dois; cada um com PR, CI verde e revisão independente registrada, e **merge só com a autorização da Madu na hora**. Antes de começar: ligar o Docker Desktop e rodar as fumaças M01 e M02 na `main`.

## Fora desta fase
- `promptHidden` da CLI (`apps/cli/src/main.ts:49-73`) tem a mesma falha de Ctrl+C e EOF corrigida no D1. PR próprio, com autorização.
- Hospedagem com TLS, imagem da API, rate limit, troca de senha, segundo fator e notificação: antes do M04 (DEC-0034).
- Fase E (início formal do M03): não autorizada.
