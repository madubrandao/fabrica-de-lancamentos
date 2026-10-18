---
tipo: fase
area: 04-Fases
id: E1
status: CONFIRMADA pela Madu em 2026-10-17 (§8); elaborada em 2026-10-16, somente leitura
fonte: DEC-0037 (§4 item 3, §6, §7); 04-Fases/pre-M03-d4-d3-plano-unificado.md (E1); 07-Log/2026-10-15-e2a-concluida.md; docs/operations/BACKUP-RUNBOOK.md e docker-compose.dev.yml do launch-os (main afa36c5)
data-registro: 2026-10-16
---
# E1: inventário e linha de base dos dados persistidos (para confirmação)

**Estado:** este documento **não** é a confirmação. Ele reúne o que foi medido e o que não foi, para a Madu confirmar (ou corrigir) a E1.

## 1. Método e limites
Somente leitura em 2026-10-16: `docker volume ls`, `docker ps -a`, `docker system df -v`, `docker images`, leitura de `infrastructure/compose/docker-compose.dev.yml`, `database/provision/provision.sql`, runbooks e do log de 2026-10-15; `df -h /c`. **Nenhum contêiner foi iniciado, nenhum segredo foi lido** (do `.env` só se conferiu existência e tamanho), nenhum backup ou restore foi feito. O escopo é só o que pertence ao LAUNCH OS (projeto Compose `launch-os`).

## 2. Inventário: o que persiste e o que entra no backup
| Item | Evidência | No backup? | Observação |
|---|---|---|---|
| Volume `launch-os_postgres_data` | 89,5 MB; contêiner `launch-os-postgres-1` parado há 8 dias | **Entra** | Dumps (`pg_dump -Fc`) dos bancos `launch_os` e `n8n` e dos papéis (`BACKUP_DATABASES`, `BACKUP_INCLUDE_GLOBALS`; BACKUP-RUNBOOK §1). O `provision.sql` cria os dois bancos. |
| Volume `launch-os_redis_data` | 292 B (`appendonly yes`); `launch-os-redis-1` parado | **Não entra** | BACKUP-RUNBOOK: Redis fora do escopo. Papel do Redis: "sem papel no M02" (BLOCKERS). A perda do Redis é teste de staging (D-8). |
| `n8n_data` (`/home/node/.n8n`) | **Não existe** volume `launch-os_n8n_data` nem contêiner `launch-os-n8n-1` no Docker; o compose declara o serviço e o volume, nunca criados no dev | **Não entra agora** (`BACKUP_INCLUDE_N8N_DATA=false`) | O arquivo `config` guarda a `N8N_ENCRYPTION_KEY` em texto (E1 de 2026-10-15). Cobertura do n8n e da chave: **verificação própria antes da primeira operação que dependa dele**. |
| `.env` | Existe (968 bytes); conteúdo **não lido** | **Não entra** (custódia da Madu) | Regeneráveis: `JWT_SECRET`, `N8N_WEBHOOK_SECRET` (com reimportação). **Não** regeneráveis: `N8N_ENCRYPTION_KEY` e a privada `age` (DEC-0037 §4 item 3). |
| Workflows, definições, código | Git | **Não entra** (fonte de verdade) | Reimportáveis a partir do repositório. |
| Pasta de backups `C:\LaunchOS-Backups` | **Não existe** | n/a | O comando cria o último nível no primeiro uso e a restringe ao usuário. |
| Imagens | `launch-os-age:1.3.2` (23 MB), `postgres:17.11`, `redis:8.10.2`, `n8nio/n8n:2.41.6` presentes | **Não entram** | Reconstruíveis; a imagem do `age` é construída da fonte (BACKUP-RUNBOOK §6). |

## 3. Exclusões expressas (outros projetos no Docker Desktop)
Existem volumes e contêineres de projetos **que não são do LAUNCH OS** e que **não entram** em nenhum backup nem inventário: `n8n_n8n_data`, `n8n_n8n_db_data`, `qg_madu_n8n_data`, `app_app_db_data`, `projetomadu_postgres_data`, `projetomadu_redis_data`, `repositorio_sol_postgres_data`, `repositorio_sol_redis_data` e volumes anônimos. O comando de backup confere o rótulo Compose, o serviço, a imagem e o volume por inspeção (`BACKUP_COMPOSE_PROJECT=launch-os`) e recusa alvo diferente.

## 4. Espaço em disco (bloqueio)
`C:` tem 222 GB, **5,3 GB livres (98% usado)** em 2026-10-16 (eram 15,7 GB em 2026-10-15). A regra do BLOCKERS pede **≥ 20 GB livres em `C:`** antes de fases que sobem a pilha. **Este item bloqueia a E2-B.** Há imagens grandes no Docker (por exemplo `n8nio/n8n` 1,75 GB, `docker.n8n.io/n8nio/n8n` 1,81 GB e 2,42 GB) que podem pertencer a outros projetos: a escolha do que liberar é **da Madu**; nada será removido pelo Claude.

## 5. Itens NÃO medidos ou NÃO verificados
- Tamanho lógico e conteúdo dos bancos `launch_os` e `n8n` (o Postgres está parado e não será ligado sem autorização).
- Se o banco `n8n` do dev contém dados (n8n nunca rodou no projeto `launch-os`).
- Dados binários e modo fila do n8n 2.41.6 (confirmar na documentação do n8n).
- Papel efetivo e conteúdo do Redis.
- Guarda da chave privada `age` (Bitwarden e cópia em papel) e da `N8N_ENCRYPTION_KEY`: **ação da Madu; o Claude Code não consegue verificar**.
- Disponibilidade do `age` no host Windows: resolvida por imagem dedicada (não é necessária no host).

## 6. Divergência documental da E1
| Fonte | O que diz |
|---|---|
| Log de 2026-10-15 e cabeçalho do plano unificado (commit `cb92be7`) | "E1 concluída", em um parágrafo, com medições "em pilha descartável, não nos dados reais" |
| Memória do projeto | Inventário realizado; **nenhuma aprovação formal encontrada** |
| DEC-0037 §7 | Ainda marca "NÃO VERIFICADO (E1)" para o `n8n_data`/modo fila e para o `age` |
| Critério do plano (E1) | "Lista fechada do que entra em cada backup, com justificativa e fonte; itens não medidos continuam marcados" |
Este documento é a lista fechada pedida. **A DEC-0037 não foi alterada**; a atualização da tabela §7 será uma proposta separada, depois da confirmação.

## 7. Evidências
`docker volume ls`; `docker ps -a`; `docker system df -v` (tamanhos: `launch-os_postgres_data` 89,5 MB, `launch-os_redis_data` 292 B); `df -h /c`; `ls -la .env` (apenas tamanho); `docker-compose.dev.yml` (volumes `postgres_data`, `redis_data`, `n8n_data`; serviço `n8n` com `N8N_ENCRYPTION_KEY` obrigatória); `.env.example` (variáveis `BACKUP_*`); BACKUP-RUNBOOK §1, §6, §7; RESTORE-RUNBOOK §6.

## 8. Confirmação da Madu
- [x] Confirmo a E1 como concluída, com estas inclusões, exclusões e itens não medidos.
- [ ] Corrijo: (nenhuma correção)

Confirmada por: **Madu**, em **17/10/2026** (autorização explícita na sessão do Claude Code; registro feito pelo Claude a pedido dela).

**Escopo confirmado:**
- **Entram:** bancos `launch_os` e `n8n` e papéis do PostgreSQL (`BACKUP_DATABASES=launch_os,n8n`, `BACKUP_INCLUDE_GLOBALS=true`).
- **Ficam fora:** Redis; `.env` (custódia da Madu); `n8n_data` (inexistente no desenvolvimento, `BACKUP_INCLUDE_N8N_DATA=false`); workflows (versionados no Git).

**Limitações que continuam NÃO verificadas:**
- **Recuperação das credenciais do n8n:** depende da custódia separada da `N8N_ENCRYPTION_KEY`, que não entra no backup. Sem ela, o banco `n8n` restaurado não decifra as credenciais (RESTORE-RUNBOOK, limite A4).
- Tamanho lógico e conteúdo dos bancos `launch_os` e `n8n`, incluindo se o banco `n8n` do desenvolvimento tem dados.
- Dados binários e modo fila do n8n 2.41.6.
- Papel efetivo e conteúdo do Redis.

**Atualizações desde a elaboração (2026-10-16):** o bloqueio do §4 caiu (C: com 44,5 GB livres em 17/10/2026). A recuperação independente da chave `age` pela cópia do Bitwarden foi validada criptograficamente em 17/10/2026 (`age` v1.3.2 oficial, procedência verificada): mesma pública da original (`age18wvggt9ujk83gsc35fdl5a6cjp64exyjv58llnu2t2mxcyf3fvfqef3sl4`) e texto sintético decifrado só com a cópia, hashes iguais. A confirmação da E1 **não** comprova restaurabilidade: só a E3 aplicada ao conjunto real comprova.

Ver [[DEC-0037-sequencia-d4-d3-backup-e-retencao]] · [[pre-M03-d4-d3-plano-unificado]] · [[DEC-0038-preparacao-do-m03-em-prs]]
