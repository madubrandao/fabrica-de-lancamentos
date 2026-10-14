---
tipo: fase
area: 04-Fases
status: ACEITO em 2026-10-14 (DEC-0037 ACEITA; E0 concluída; E1 a E13 NÃO autorizadas: cada etapa depende de autorização própria da Madu)
fonte: DEC-0037; DEC-0034 §5 (fase D); DEC-0035 §3.6, §3.7, §8; DEC-0036 §2.3; pre-M03-fase-D-plano.md (2026-10-11); launch-os `main` em `5ce7c63`
atualizado: 2026-10-14
---
# Plano unificado D4 + D3 (backup e restore, depois retenção do `audit_log`)

**Substitui, para fins de execução, as seções "D3" e "D4" de [[pre-M03-fase-D-plano]]** (que permanece como histórico). A decisão e as condições estão no [[DEC-0037-sequencia-d4-d3-backup-e-retencao]]. **Nada aqui está implementado.** Cada etapa: branch própria, `check-repo.sh` depois do `git add`, CI verde, revisão independente registrada e merge só com a autorização da Madu na hora. Complexidade: S pequena, M média, L grande (relativa, sem prazo prometido).

## Ordem
D4 local (E1 a E4) → D3a partição (E5, E6) → trilha Railway (E7 a E13). D4 vem antes de D3 porque a conversão do `audit_log` só pode rodar depois de existir um backup restaurado e testado (E3). **D3a (E5, E6) não depende do destino da exportação e pode avançar quando seus pré-requisitos estiverem atendidos; D3b (E11) permanece bloqueada até a E7 validar o destino da exportação** (DEC-0037 §2, item 5).

## Trilha D4 e D3a (local)
| Etapa | Objetivo | Arquivos e componentes | Depende de | Riscos | Testes e aceite | Compl. |
|---|---|---|---|---|---|---|
| **E0** (autorizada) | Formalizar a sequência | DEC-0037, este plano, ponteiros nos DEC-0034/35/36, fases D e E, milestones, log | Aceite da Madu no PR | Reescrever decisão aceita sem registro | Verificações documentais; nenhum DEC antigo alterado além de um ponteiro; Madu aceita o PR | S |
| **E1** | Inventário e linha de base, **somente leitura** | Doc de evidência: tamanho dos bancos, **conteúdo real do `n8n_data`**, dados binários e modo fila do n8n 2.41.6, variáveis e segredos por ambiente e o que se regenera, estado do Redis, disponibilidade do `age` | E0 | O volume do n8n guardar algo que o dump não cobre | Lista fechada do que entra em cada backup, com justificativa e fonte; itens não medidos continuam marcados | S |
| **E2** | Backup local | `scripts/ops/backup.ts` e biblioteca (padrão TS do repositório); testes unitários | E1; instalação do `age` aprovada pela Madu | Segredo em argv; dump inconsistente; alvo errado | `pg_dump` dos dois bancos, manifesto (SHA-256, versão, última migration, contagens) e cifra; hashes conferem; recusa contexto Railway; sem segredo em argv; testes verdes | M |
| **E3** | Restore local e drill | `scripts/ops/restore.ts`, verificação objetiva, projeto compose isolado de restore | E2 | Perder `REVOKE` e owners no restore; derrubar a pilha errada; restaurar sobre banco não vazio | Os 7 passos do DEC-0037 §5, com **controles negativos** (app sem UPDATE/DELETE/TRUNCATE; n8n com chave errada falha); API sobe no banco restaurado; recusa banco não vazio; evidência e tempo medido (insumo do RTO) | L |
| **E4** | Runbook local e custódia das chaves | `docs/operations/BACKUP-RUNBOOK.md` (launch-os) | E3 | Procedimento só no papel | A Madu executa o procedimento e **confirma a guarda** da chave privada `age` e da `N8N_ENCRYPTION_KEY` em entradas separadas (ação dela; o Claude Code não verifica) | S |
| **E5** | Conversão do `audit_log` em tabela particionada | Migration `0014` (+ `down`), função de criação de partições, chamada pela manutenção da API; testes de integração | E3 (backup verificado), D-9 revisado, investigação obrigatória concluída. **Não depende do destino da exportação** | Ver DEC-0037 §4.4: PK `(id, occurred_at)`; FKs; **TRUNCATE por partição**; privilégios de partição nova; lock; dados existentes | **Relatório da investigação obrigatória antes da migration** (DEC-0037 §4.4): desempenho com `EXPLAIN`, particionamento, consultas históricas e destacamento sobre `triggers.ts:37-42` e `deliver.ts:132-138`; **migração não presumida segura**; correção de consulta, se necessária, é decisão separada; **ensaio em cópia restaurada antes**; contagem e soma de verificação iguais; tabela antiga mantida como `audit_log_pre_particao` **sem apagar**; imutabilidade, privilégios e FKs provados **em cada partição**; gravação sem partição **falha de forma explícita**; `down` funciona; CI de integração verde | L |
| **E6** | Folga de partições e alertas | Comando `launch-os ops audit-partitions` (leitura), evento de falha | E5, D-7 | Parada silenciosa sem canal | Folga em meses calculada (normal 2; aviso < 2; crítico < 1); falha por partição ausente gera evento; testes dos limiares. **Alerta automático e independente é exigido para ambiente hospedado sem supervisão; monitoramento manual só no desenvolvimento local, sem exceção para o staging** | M |

## Trilha Railway (cada etapa só depois da autorização da etapa 4 da DEC-0036 e, quando houver recurso externo, com confirmação da Madu a cada passo)
| Etapa | Objetivo | Componentes | Depende de | Riscos | Aceite | Compl. |
|---|---|---|---|---|---|---|
| **E7** | Validar e preparar o R2 (a Madu cria; o Claude Code entrega o checklist; o bucket de desenvolvimento é opcional) | Buckets por ambiente, tokens por função, lock, 2FA | D-2, D-3 | Token amplo; lock removível por administrador; jurisdição inadequada | **Checklist da D-2:** localização/jurisdição, quem remove o lock e se fica registro, escopo dos tokens, 2FA, bucket sem acesso público; teste de papéis (quem grava não lê nem apaga; quem lê não grava). **Reprovou: avaliar S3 `sa-east-1` ou B2** | S |
| **E8** | Serviço de backup agendado (staging sob demanda; production agendado) | `infrastructure/railway/backup.Dockerfile` com digest, papel `launch_os_backup`, cron, envio com checksum, alerta de backup atrasado | E2, E7, D-9, etapa 4 | Credenciais do job vazarem; execução sobreposta; backup que falha em silêncio | Backup chega cifrado ao R2 com manifesto; job sem chave privada; alerta se o último backup passar de 36 h; **sem lifecycle de exclusão ativo** (D-4) | L |
| **E9** | Restore no staging (DEC-0035 §3.7) | Serviço temporário com `psql` e `pg_restore`, par `age` de staging | E8 | Chave privada no serviço temporário | Evidência da §3.7 (comando, hora, contagens antes e depois, migrations, credenciais do n8n abertas com a chave); serviço removido e registrado; tempo medido contra o RTO | M |
| **E10** | Redis | Resultado do teste 3 da etapa 5 do DEC-0035 | etapa de modo fila | Fila perdida em silêncio | Perda do Redis em execução recuperada **ou marcada explicitamente como perdida**; só então vale D-8 | S |
| **E11** | **D3b (BLOQUEADA até a E7 validar o destino da exportação):** exportação e `DETACH` guiado | Comando de exportação NDJSON gzip, manifesto, hash, cifra; bucket `audit-export` | E5, E7 | Destacar antes de verificar; exportação incompleta | Com partições sintéticas: exportar, subir, baixar de volta, conferir hash e contagem **igual** à da partição; `DETACH` só com aprovação registrada da Madu; **nada é apagado**; o `DROP` é decisão separada | L |
| **E12** | Primeiro backup de production restaurado | Ativação em production; drill em ambiente isolado (D-10) | E8, E9, deploy de production | Restaurar dados reais fora de ambiente próprio | Restore isolado com evidência e RTO medido; sem dados de production no staging | M |
| **E13** | Proteção adicional independente (D-11) | Segunda cópia em outro provedor e conta; cópia offline cifrada | D-11 decidida, E8 | Duas cópias com o mesmo administrador não são independentes | Segunda cópia verificada por hash; credenciais e administrador distintos; cópia offline reproduzida e conferida pela Madu | M |

## Liberação
O critério por modo (M03 local, M03 em staging, production) é a matriz do [[DEC-0037-sequencia-d4-d3-backup-e-retencao]] §6. O M03 só começa com o início formal da fase E, decisão da Madu, **depois** dessa matriz cumprida e com a escolha de onde ele roda.

## Fora deste plano
Hospedagem com TLS, rate limit, troca de senha, segundo fator e notificação geral (antes do M04, DEC-0034); `promptHidden` da CLI (PR próprio, já registrado na fase D).
