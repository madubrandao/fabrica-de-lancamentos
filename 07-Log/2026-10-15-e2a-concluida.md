---
tipo: log
area: 07-Log
fonte: sessão de 2026-10-15; launch-os PR #45 (main `6ccd6de`) e PR #46 (status, aguardando aprovação)
atualizado: 2026-10-15
---
# E1 e E2-A do plano D4 + D3 (2026-10-15)

## E1 (inventário, somente leitura)
Bancos `launch_os` e `n8n` no PostgreSQL 17.11; ferramentas `pg_dump`, `pg_restore` e `pg_dumpall` presentes na imagem (o Windows não precisa do PostgreSQL). O `n8n_data` guarda a `N8N_ENCRYPTION_KEY` em texto no arquivo `config`. `age` ausente no host: resolvido com imagem dedicada. Medições em pilha descartável, não nos dados reais.

## E2-A (mecanismo de backup local): CONCLUÍDA
- **Implementado e testado:** `pnpm ops:backup` (launch-os, PR #45 mesclado com autorização da Madu; main `6ccd6de`). `pg_dump`/`tar` por pipe para o `age` v1.3.2 em contêiner (só chaves públicas); manifesto com origem, conjunto esperado e hashes; pasta restrita por dono e SID; alvo conferido por inspeção do Docker; prazo, sinais e lock exclusivo **sem recuperação automática** (recuperação manual no runbook §9).
- **Evidências:** 1008 testes unitários (três execuções seguidas), autoteste Docker 49/49 com dados fictícios, CI 3/3. Duas revisões independentes; achados B1, A1 a A6, B1' e B2 corrigidos.
- **NÃO comprovado:** restaurabilidade (E3). O manifesto grava `restorable: false`. Nenhum backup do desenvolvimento foi feito; os volumes de desenvolvimento não foram iniciados nem tocados.

## Pendente para o primeiro backup do desenvolvimento
Par de chaves `age` definitivo gerado pela Madu (privada em entrada separada da `N8N_ENCRYPTION_KEY`, com cópia offline); variáveis `BACKUP_*` no `.env`; autorização para iniciar o PostgreSQL de desenvolvimento; `C:\LaunchOS-Backups` criada pelo comando no primeiro uso.

## Não autorizado
E2-B, E3, D3a e M03; chaves definitivas; dados reais; Railway e R2.

## Riscos residuais
`docs/operations/BACKUP-RUNBOOK.md` §6 e §8 (launch-os): restore e consistência entre artefatos; `verify` não autentica; encerramento forçado bloqueia os backups até a recuperação manual; `Ctrl+C` real não exercitado; imagem do age construída da fonte; sem `fsync` nem retenção.
