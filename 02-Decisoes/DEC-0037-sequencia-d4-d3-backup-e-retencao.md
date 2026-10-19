---
tipo: decisao
area: 02-Decisoes
id: DEC-0037
status: ACEITA (texto final aprovado pela Madu em 2026-10-14, depois da revisão dos PRs #1 e #44; as decisões D-1 a D-10 e os 6 ajustes obrigatórios foram aprovados com condições na mesma data). A aceitação **não implementa nada**: só a E0 (documentação) foi autorizada e concluída; E1 a E13 e o M03 continuam **NÃO autorizados**, cada etapa depende de autorização própria.
fonte: DEC-0034 (§2, §4, §5); DEC-0035 (§3.2, §3.6, §3.7, §8, §10); DEC-0036 (§2.3); 04-Fases/pre-M03-fase-D-plano.md e pre-M03-fase-E-plano.md; launch-os na main `5ce7c63` (migration 0006, packages/events/src/db.ts:54-64, packages/events/src/maintenance.ts, infrastructure/compose/docker-compose.dev.yml, docs/operations/M01-RUNBOOK.md:76); pesquisa pública de 2026-10-14 (preços e bucket locks do Cloudflare R2; cron e backups do Railway)
data-registro: 2026-10-14
data-decisao: 2026-10-14 (aceite formal da Madu, por autorização explícita, depois da revisão dos PRs)
emenda: DEC-0034 (§5, fase D), DEC-0035 (§3.6, §8 etapas 7 e 8, §10), DEC-0036 (§2.3)
---
# DEC-0037: sequência e escopo de D4 (backup e restore) e D3 (retenção do `audit_log`)

> **ACEITA pela Madu em 2026-10-14.** Não altera código, banco, configuração nem recursos externos. O M03 continua **não autorizado**. Nada é criado no Railway nem em nuvem. Esta DEC **não reescreve** o DEC-0034, o DEC-0035 nem o DEC-0036: ela registra o que muda, o que permanece e quem decide o que falta. Os três ganham só uma linha de ponteiro para cá.

## 1. Contexto: o conflito de sequência (exposto, não reconciliado em silêncio)
| Fonte | O que diz | Consequência |
|---|---|---|
| DEC-0034 §5, fase D | "Operação local": comando de importação, partição do `audit_log`, limpeza agendada, **backup e restore testados** (com a `N8N_ENCRYPTION_KEY`) são pré-condição do M03; o M03 roda local | D3 e D4 inteiros antes do M03 |
| DEC-0035 §1 e §2 | Antecipa a hospedagem: o Railway passa a ser o ambiente de execução da pilha | A frase "o M03 roda local" do DEC-0034 fica sem dono: ela não foi revogada nem confirmada |
| DEC-0035 §8 etapas 7 e 8 (renumeradas no DEC-0036 §2.3) | **D4 = restore no staging** (serviço temporário) e **D3 bloqueado** até o destino da exportação | D4 passa a depender do Railway, que o DEC-0034 não previa |
| `pre-M03-fase-D-plano.md` (2026-10-11) | D3 e D4 locais, antes do M03; a partição `DEFAULT` aparece como pergunta aberta | Diverge do DEC-0035 §3.6, que já decidiu **sem** partição `DEFAULT` |
| `pre-M03-fase-E-plano.md` (2026-10-11) | Lista a fase D como "em andamento" e o M03 como local | Desatualizado em relação ao DEC-0035 |
| Estado real (launch-os, `5ce7c63`) | `audit_log` é tabela comum com PK `(id)`; não existe script, comando nem runbook de backup (`M01-RUNBOOK.md:76`: "NÃO DOCUMENTADO nem testado"); a limpeza agendada existe (D2) e nunca toca `event` nem `audit_log` | D3 e D4 **não foram implementados** |

## 2. O que permanece e o que muda
**Permanece, sem alteração:** a política do `audit_log` (partição mensal, 24 meses no banco, exportar antes de destacar, **nada é apagado sem decisão**); a ausência de partição `DEFAULT` e os 2 meses criados à frente (DEC-0035 §3.6); a `N8N_ENCRYPTION_KEY` guardada no gerenciador de senhas da Madu com uma cópia offline (DEC-0034 §3); chave diferente por ambiente (DEC-0035 §2.2); o estado do Redis como parte do teste de perda do DEC-0035 §3.2; as demais pré-condições da fase E (chave Anthropic, `llm_cost_limit`, `llm_pricing`, produto fictício).

**Muda (somente por esta DEC, depois do aceite):**
1. D3 e D4 deixam de ser um bloco único "local, antes do M03" e passam a ter **parte local** e **parte Railway** (D-1).
2. O destino da exportação deixa de ser "pendente" e passa a ser **Cloudflare R2, condicionado** (D-2).
3. Alertas automáticos e independentes passam a ser exigidos antes de qualquer ambiente hospedado sem supervisão (D-7). **O monitoramento manual é permitido apenas no desenvolvimento local; não há exceção para o staging.** Isso flexibiliza, só para o local, os "alertas obrigatórios" do DEC-0035 §3.6.
4. A frase "o M03 roda local" **deixa de ser pressuposto e é considerada superada**: o ambiente do M03 (local ou staging) será definido **na autorização formal do M03**. Esta DEC não decide isso (§6).
5. **D3a e D3b ficam separadas.** **D3a** (E5 conversão e E6 folga de partições) **pode avançar quando seus pré-requisitos estiverem atendidos** (E3 concluída, D-9 revisado e a investigação obrigatória do §4.4 concluída); ela **não depende do destino da exportação**. **D3b** (E11, exportação e `DETACH`) **permanece bloqueada até a E7 validar o destino da exportação**. Assim, o "D3 bloqueado até o destino da exportação" do DEC-0035 §8 (etapa 8) e §10 passa a valer **somente para D3b**; o restante do D3 segue o §6 desta DEC.

## 3. Decisões (D-1 a D-10) e as condições impostas pela Madu
| # | Decisão | Condição ou limite |
|---|---|---|
| **D-1** | Dividir D3 e D4 em parte local e parte Railway, conforme a matriz do §6 | Formalizada por esta DEC |
| **D-2** | **Cloudflare R2** como opção inicial de storage privado compatível com S3, com buckets separados por ambiente e finalidade (`-staging`, `-production` e `launch-os-audit-export-production`; o bucket de desenvolvimento `launch-os-backups-dev` é **opcional** e não é requisito do M03 local: o backup local é gravado em pasta local) | **Condicionada à validação** (checklist da etapa E7): políticas de segurança, **localização/jurisdição dos dados**, **proteção contra exclusão**, escopo dos tokens, 2FA da conta, registro de ações administrativas. Se não passar, alternativas: AWS S3 `sa-east-1` com Object Lock, Backblaze B2 |
| **D-3** | Criptografia no cliente com **`age`**, um par de chaves por ambiente; o job de backup só tem a chave **pública** | **Custódia separada:** a chave privada de cada ambiente e a `N8N_ENCRYPTION_KEY` ficam em entradas distintas do gerenciador de senhas, cada uma com cópia offline. Perder a privada inutiliza os backups daquele ambiente |
| **D-4** | Retenção proposta: production 7 diários, 4 semanais e 6 mensais; staging 3 mais recentes; local 5 manuais | **Provisória.** **Nenhuma exclusão automática (lifecycle de expiração) é ativada** antes de nova revisão e confirmação da Madu |
| **D-5** | Metas: RPO 24 h e RTO 4 h | **São metas a comprovar** no drill (E9/E12), **não garantias** |
| **D-6** | `audit_log` mensal; fronteiras em UTC; sem `DEFAULT`; 2 meses à frente; exportação NDJSON gzip + manifesto + hash, cifrada; só `DETACH`, nunca `DROP`; tabela pré-partição mantida até aprovação | **Sujeita a testes** de integridade, privilégios, referências e imutabilidade (ver §5.4) |
| **D-7** | Canal de alertas: monitoramento manual **só no desenvolvimento local** | **Alertas automáticos e independentes** antes de operar ambientes hospedados sem supervisão. "Independente" = não depende do processo monitorado (a API) para avisar. **Sem exceção para o staging** |
| **D-8** | Redis sem backup | **Somente depois** do teste que comprove recuperação ou detecção explícita das execuções perdidas (DEC-0035 §3.2, teste 3). Sem a prova, a decisão não vale |
| **D-9** | Papel `launch_os_backup` (somente leitura) e função `SECURITY DEFINER` restrita a criar partições do `audit_log` | **Sujeitos à revisão** de privilégio mínimo, `search_path` fixo e **testes negativos** (o papel não escreve, não cria outra coisa, não é chamável por quem não deve) |
| **D-10** | Todo restore ocorre em ambiente **isolado**; **proibido** usar dados reais de production no staging | Só com **autorização específica** e controles adequados |

## 4. Ajustes obrigatórios (resposta da Madu) e como ficam registrados
1. **Bucket lock não é garantia absoluta.** Pela documentação pública do R2, uma regra de lock impede apagar ou sobrescrever objetos por prazo ou indefinidamente e prevalece sobre a expiração do lifecycle; **também pela documentação, um bucket com regras de lock não pode ser esvaziado antes de remover as regras**, ou seja, quem administra o bucket pode remover a regra. Por isso o lock é tratado como **barreira contra erro e contra token comprometido, não contra administrador privilegiado**. *Não verificado:* quem exatamente pode remover a regra e se a ação fica registrada. Entra no checklist da E7.
2. **Proteção adicional independente (nova decisão D-11, pendente).** Para os backups críticos de production e as exportações do `audit_log`: (a) uma **segunda cópia em outro provedor e outra conta**, com credenciais e administrador próprios, preferencialmente com modo de retenção que nem o administrador consiga encurtar (por exemplo, AWS S3 Object Lock em modo *compliance*, a validar); (b) uma **cópia offline cifrada** periódica (mensal) sob sua guarda. Os dois são requisito de production (§6); a escolha do provedor da segunda cópia é sua.
3. **Cobertura dos dados do n8n.** Os dois bancos PostgreSQL **não bastam**. O compose local também monta o volume `n8n_data` em `/home/node/.n8n`, e o n8n usa Redis no modo fila do Railway. A etapa **E1 produz o inventário** do que cada ambiente persiste: bancos `launch_os` e `n8n`; conteúdo real do volume `n8n_data` (a confirmar: arquivo de configuração, nós, dados binários, logs); estado do Redis; variáveis e segredos por ambiente (o que se regenera: `JWT_SECRET`, `N8N_WEBHOOK_SECRET` com nova importação; o que não: `N8N_ENCRYPTION_KEY` e a chave privada `age`); workflows no repositório (fonte de verdade, reimportáveis). *Fato do código:* o compose já desliga o histórico de execuções do n8n (`EXECUTIONS_DATA_SAVE_*: none`), então execuções não são dado a proteger; `import:credentials` recria hoje as duas credenciais do n8n, mas credenciais futuras de integrações reais podem não ser recriáveis. Nada disso foi medido ainda.
4. **Conversão da chave primária do `audit_log` (D3).** Evidência lida em 2026-10-14 no launch-os (`main`): **nenhuma tabela referencia `audit_log`** (busca por `REFERENCES audit_log` em `database/`: zero); a única escrita é `INSERT` sem `ON CONFLICT` e sem `RETURNING` (`packages/events/src/db.ts:54-64`); os testes e o helper de teste leem ordenando por `occurred_at` e `id`; **o código de produção tem duas leituras que NÃO filtram por `occurred_at`:** `apps/api/src/gateway/triggers.ts:37-42` (`SELECT execution_id FROM audit_log WHERE action = 'event.emit' AND entity_type = 'event' AND entity_id = $1 AND organization_id = $2 LIMIT 1`) e `packages/events/src/deliver.ts:132-138` (`count(*)` por `entity_type`, `entity_id`, `action` e `details->>'delivery_id'`); os testes de imutabilidade fazem `UPDATE` e `DELETE ... WHERE id` (`tests/integration/immutability.test.ts:71-72`); as outras migrations só citam a tabela em comentários. **Riscos a provar, não presumidos:** (i) o PostgreSQL exige a chave de partição na PK, então a PK vira `(id, occurred_at)` e **o banco deixa de garantir unicidade global de `id`** (aceito por `gen_random_uuid()`, a registrar); (ii) as FKs **de** `audit_log` para `organization` e `event` precisam sobreviver à tabela particionada; (iii) os triggers de linha propagam para as partições, mas **o trigger de TRUNCATE por partição precisa ser provado**, e partições novas não podem nascer com privilégios para `launch_os_app`; (iv) o dono das tabelas continua podendo `TRUNCATE`, `DETACH` e `DROP` (risco C6 já aceito, não piora); (v) conversão com dados reais (tempo e lock). **Procedimento mínimo:** backup verificado e **ensaio em cópia restaurada** antes da migration; copiar para a tabela nova, conferir contagem e soma de verificação, trocar os nomes numa transação, manter a antiga como `audit_log_pre_particao` **sem apagar**, `down` reversível; testes de imutabilidade em **cada** partição.
   **Investigação obrigatória na E5, antes de qualquer migration (relatório próprio; a migração NÃO é presumida segura antes dela).** Cobre, para as duas leituras acima e para qualquer outra que uma busca ampliada achar: (a) **desempenho**: planos de execução (`EXPLAIN`) com várias partições, já que sem `occurred_at` não há poda de partição e a consulta visita todas, usando o índice parcial `(entity_type, entity_id, occurred_at)` de cada partição; (b) **particionamento**: efeito das mudanças de índice e de PK sobre essas consultas; (c) **consultas históricas**: `reprocessCycle` conta o histórico inteiro e `emitterOf` procura por evento sem limite de data; (d) **destacamento** (*hipóteses a provar, não conclusões*): depois de um `DETACH`, as linhas antigas somem dessas consultas; `emitterOf` pode passar a devolver "sem rastro" para eventos antigos e `reprocessCycle` pode contar menos ciclos do que ocorreram. Se a investigação mostrar impacto, a correção (filtrar por data, mudar a consulta ou outra) é **decisão separada, com autorização da Madu**, e não está feita nem autorizada aqui.
5. **Requisitos por modo.** Matriz do §6.
6. **Riscos não verificados.** Lista do §7; nenhum é declarado resolvido.

## 5. Arquitetura (resumo; detalhe em `04-Fases/pre-M03-d4-d3-plano-unificado.md`)
Quatro trilhas separadas: **A** local; **B** staging; **C** production; **D** exportação do `audit_log`. Controles comuns: tokens por bucket e por função (quem grava não lê nem apaga; o leitor só existe durante o drill; o administrador fica fora de qualquer ambiente); manifesto com SHA-256, versão do PostgreSQL, última migration e contagem por tabela; restore recusa arquivo com hash divergente e **nunca escreve sobre banco não vazio**; backup e restore com alvo explícito e confirmação por nome (production com confirmação extra), como a importação da etapa 3b-2; imagem do serviço de backup com digest fixado. **Restore testável** (o mesmo em todas as trilhas): pilha isolada e vazia; hash; decifrar; `pg_restore` preservando owners e privilégios; conferir migrations e contagens; **controles negativos** (`launch_os_app` não faz UPDATE/DELETE/TRUNCATE em `audit_log` e `event`; n8n com chave errada falha); API sobe contra o banco restaurado; pilha derrubada com evidência. Cron do Railway: roda em UTC, mínimo de 5 minutos, e pula a execução se a anterior ainda não terminou; os backups de volume do Railway são camada extra e **não** a estratégia, porque não sobrevivem à exclusão do volume ou do projeto.

## 6. Requisitos obrigatórios por modo
Legenda: **O** = obrigatório antes de o modo operar; **S** = obrigatório apenas se o ambiente operar sem supervisão; **—** = não se aplica; **D** = pode vir depois, mas bloqueia o modo seguinte. As etapas E0 a E13 estão no plano unificado.
| Requisito | M03 local | M03 em staging | Production |
|---|---|---|---|
| E0: DEC-0037 aceito | O | O | O |
| E1: inventário completo dos dados persistidos (inclui `n8n_data`) | O | O | O |
| E2 a E4: backup e restore local, com drill e controles negativos; guarda das chaves confirmada pela Madu | O | O (base dos scripts do staging) | O |
| E5 e E6: partição mensal do `audit_log` e folga de partições | O | O | O |
| Alertas automáticos e independentes (D-7) | — (monitoramento manual permitido **só aqui**) | **S**: obrigatório se operar sem supervisão; monitoramento manual **não** é aceito no staging | O |
| Fumaças M01 (49) e M02 (31) verdes; intermitência registrada | O | O, mais verificação reduzida de 7 itens e **registro dos webhooks na primeira importação** | O, validadas antes no staging |
| E7: buckets, tokens, localização e proteção contra exclusão validados | — | O | O |
| E8: backup agendado | — (sob comando) | sob demanda antes de cada janela | O |
| E9: restore testado no staging (DEC-0035 §3.7) | — | O | O (herdado) |
| E10: teste de perda do Redis concluído (D-8) | — | O | O |
| E11: exportação do `audit_log` e `DETACH` guiado, provados com partições sintéticas | — | D | O |
| E12: primeiro backup de production restaurado em ambiente isolado (D-10) | — | — | O |
| E13 / D-11: proteção adicional independente (segunda cópia e cópia offline) | — | — | O |
| Restore com dados reais de production no staging | proibido | proibido | proibido sem autorização específica |

**Lacuna registrada, não resolvida:** operar o staging **com** a supervisão presencial da Madu, mas **sem** alerta automático, não está coberto pela D-7 como aprovada (manual só no local). Quem quiser operar assim precisa de confirmação expressa dela, que não existe hoje.

**Critério objetivo para liberar o M03 (necessário, não suficiente; o início formal é decisão da Madu):** todas as células **O** das colunas "M03 local" ou "M03 em staging" (a que ela escolher) com evidência registrada, mais o início formal da fase E. As colunas "staging" e "production" são **cumulativas** com a anterior.

## 7. Riscos e itens não verificados (nenhum está resolvido)
| Item | Estado |
|---|---|
| Tamanho real dos dumps e custo do R2 no M03 | **NÃO MEDIDO** (R2 público: US$ 0,015/GB-mês, 10 GB-mês grátis; confirmar no painel) |
| Localização/jurisdição dos dados no R2; quem pode remover um lock; registro de ações administrativas | **NÃO VERIFICADO** (E7) |
| `age` disponível para Windows e para a imagem do serviço de backup | **NÃO VERIFICADO** (E1) |
| Conteúdo real do volume `n8n_data` e uso de dados binários no modo fila | **NÃO VERIFICADO** (E1; confirmar na documentação do n8n 2.41.6) |
| Trigger de TRUNCATE por partição; privilégios de partições novas; custo de lock na conversão | **NÃO VERIFICADO** (E5) |
| Impacto da partição e do `DETACH` nas leituras de produção `triggers.ts:37-42` e `deliver.ts:132-138` (sem filtro de `occurred_at`): desempenho, histórico e resultado depois de destacar | **NÃO INVESTIGADO**; a migração **não** é presumida segura (E5) |
| Backups de volume do Railway no plano da Madu | **NÃO VERIFICADO**; só camada extra |
| Cron do Railway em execução real, com a imagem de backup | **NÃO VERIFICADO** (E8) |
| RPO 24 h e RTO 4 h | **METAS**, a comprovar no drill |
| Perda do Redis em execução | **NÃO TESTADO** (DEC-0035 §3.2); D-8 não vale antes disso |
| Guarda das chaves privadas e da `N8N_ENCRYPTION_KEY` | **Ação da Madu**; o Claude Code não consegue verificar |
| Intermitência da fumaça M02 (webhook não registrado em 120 s) | **ABERTA, causa não identificada** (`M02_QUALITY_EVIDENCE.md`) |
| Provas contra o Railway real (`railway ssh`, `restart`, `/proc/1/stat`, `-p` com nome) | **PENDENTES** (etapa 6) |
| Testes de integração rodam só na CI | Sem execução local |

## 8. Pendências de decisão da Madu e paradas
- **Onde o M03 roda** (local ou staging): definido **na autorização formal do M03** (fase E). A frase "o M03 roda local" está superada.
- **Aceite formal desta DEC: concluído em 2026-10-14**, por autorização explícita da Madu; o status passou de PROPOSTA para ACEITA. A aceitação não implementa nada e não autoriza E1 a E13 nem o M03.
- **Referência operacional do vault:** `feat/core-platform` é a referência **provisória**. A `main` do vault está 43 commits atrás e não tem os DEC-0032 a DEC-0036. **Atividade futura, não feita agora:** promover `feat/core-platform` para a `main`, com **auditoria própria** dos 43 commits de diferença. Enquanto isso, esta DEC só é referência em `feat/core-platform`.
- **D-11:** provedor da segunda cópia independente e frequência da cópia offline.
- **Revisão da retenção D-4** antes de qualquer exclusão automática.
- **Destino definitivo da exportação** depois do checklist da E7 (se o R2 reprovar).
- **Canal dos alertas automáticos** (conta SMTP ou monitor externo) antes de qualquer ambiente sem supervisão; **prazo esperado do job de partição e quem responde ao alerta**.
- **Autorizar cada etapa de implementação** (E1 em diante), uma por vez, com PR, CI verde, revisão independente e merge só com a autorização dela.
- **Registro posterior (ponteiro, 2026-10-17):** decisões da Madu sobre a E5/E6 (aceite do relatório da §4.4, D11 sem filtro de data até a E11, critério de conclusão da entrega de código, `MAINTENANCE_*` opcional sem limpeza autorizada, rotina manual da D-7 no local) em [[pre-M03-d4-d3-plano-unificado]] (Registro de execução) e [[2026-10-17-e5-e6-entrega-de-codigo]]. Esta DEC não foi alterada além deste ponteiro.

## 9. O que esta DEC NÃO faz
Não implementa E1 a E13; não cria bucket, token, chave ou recurso externo; não altera banco, código, configuração, workflows nem o Railway; não exclui dados; não autoriza o M03; não revoga nenhuma decisão anterior.

Ver [[DEC-0034-perfil-de-valores-e-plano-ate-o-m03]] · [[DEC-0035-migracao-para-o-railway]] · [[DEC-0036-emenda-da-dec-0035]] · [[pre-M03-fase-D-plano]] · [[pre-M03-fase-E-plano]] · [[pre-M03-d4-d3-plano-unificado]]
