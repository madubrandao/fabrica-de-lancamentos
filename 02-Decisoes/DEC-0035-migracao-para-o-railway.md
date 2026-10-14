---
tipo: decisao
area: 02-Decisoes
id: DEC-0035
status: ACEITA (conteúdo aprovado pela Madu em 2026-10-11, com as 4 notas e as 3 observações incorporadas; nada foi criado no Railway)
fonte: decisões da Madu em 2026-10-11; DEC-0033; DEC-0034 (§4 e §5); DEC-0017; M02-RUNBOOK; apps/api/src/server.ts:132 e config.ts:222; scripts/n8n/import.ts:73; documentação pública do Railway (CLI e private networking)
data-registro: 2026-10-11
emenda: DEC-0034 (muda "o M03 roda local" e antecipa parte da hospedagem), DEC-0033 (A4: imagem Docker da API)
---
# DEC-0035: migração da pilha para o Railway (staging e production)

> **Aceita pela Madu em 2026-10-11** (aprovou o conteúdo e autorizou a gravação como ACEITA). O M03 continua **não autorizado**. Nenhuma chamada real a LLM. Nenhum recurso é criado no Railway sem a confirmação da Madu a cada passo; o Claude Code não tem credencial do Railway.
>
> **Emenda posterior (PROPOSTA, 2026-10-14):** [[DEC-0037-sequencia-d4-d3-backup-e-retencao]] reorganiza a sequência de D3 e D4 e o destino da exportação; em especial, **D3a fica liberada do bloqueio do §8 (etapa 8) e do §10, que passa a valer só para D3b (exportação)**, e os alertas do §3.6 mudam conforme a D-7. O texto desta DEC não foi alterado; vale o que o DEC-0037 disser **somente depois de aceito** (hoje é PROPOSTA).

## 1. Contexto
- O ambiente local (7,7 GB de RAM, 17 GB livres em `C:`, mínimo exigido 20 GB) parou o projeto várias vezes. A pilha ocupa pouco em dados (~170 MB), mas exige PostgreSQL, Redis e n8n ligados, e o notebook não tem folga.
- A Madu já usa o Railway em outros projetos e informa que o **plano Pro está contratado** (premissa dela, não verificada pelo Claude Code).
- O DEC-0034 deixava a hospedagem para antes do M04 e dizia que o M03 roda local. Esta DEC **antecipa** a hospedagem.

## 2. Decisão
1. **O Railway passa a ser o ambiente de execução da pilha**, com **dois ambientes** no mesmo projeto: `launch-os-staging` e `launch-os-production`.
2. **Motivo:** custo coberto, em princípio, pelo plano Pro já contratado (ver §7, estimativa a medir) e fim da dependência da memória e do disco do notebook.
3. **Serviços por ambiente:** `api`, `n8n-editor`, `n8n-worker`, `postgres` (17) e `redis` (8). Imagens por **digest** (§3.8).
4. O desenvolvimento continua no notebook (editor, Git, Claude Code e testes unitários).

### 2.1 Exposição
| Serviço | Público? | Acesso |
|---|---|---|
| `postgres` | **Não** | Só rede privada. Sem domínio, sem TCP proxy |
| `redis` | **Não** | Só rede privada. Sem domínio, sem TCP proxy |
| `api` | **Não** | Só por `${{api.RAILWAY_PRIVATE_DOMAIN}}`. **Nenhum domínio público gerado** |
| `n8n-worker` | **Não** | Só rede privada |
| `n8n-editor` | **Sim**, único serviço público | Autenticação própria do n8n (ver §6, R2 e R8) |

Cada ambiente do Railway tem rede privada **isolada**; staging e production não se enxergam.

### 2.2 Dois ambientes
- **`launch-os-staging`:** recebe fumaças, testes e restores. Pode ser derrubado e recriado. Pode ficar **desligado** fora das janelas de teste, para reduzir custo. Só dados fictícios.
- **`launch-os-production`:** só recebe deploy **depois de validado no staging**. **Nunca recebe fumaças.** As fumaças criam usuários e dados.
- Cada ambiente tem seus próprios segredos. `N8N_ENCRYPTION_KEY` é **diferente por ambiente** e **idêntica entre editor e worker do mesmo ambiente**.

## 3. Decisões técnicas

### 3.1 `HOST` configurável (ENG-01, no PR do Dockerfile)
- `apps/api/src/server.ts:132` passa a ler `HOST` do ambiente. Padrão `127.0.0.1` em desenvolvimento. No Railway o valor é definido pela configuração do serviço.
- **PENDENTE:** `0.0.0.0` ou `::` no Railway. Ambientes antigos resolvem `railway.internal` só por IPv6; os criados a partir de 16/10/2025 resolvem IPv4 e IPv6. Verificar no primeiro deploy do staging e registrar a evidência.
- O ENG-01 só confirma o `/healthz` (`app.ts:472`) depois; não há trabalho novo.
- Teste: sem `HOST`, escuta em `127.0.0.1`; com `HOST`, escuta no valor dado. Valor inválido recusa subir.

### 3.2 Modo fila do n8n (PR separado do Dockerfile)
- **Desenho:** `n8n-editor` (UI e webhooks, `EXECUTIONS_MODE=queue`) + `n8n-worker` (executa) + Redis como fila, com a **mesma `N8N_ENCRYPTION_KEY`** nos dois. Banco PostgreSQL próprio do n8n, como hoje.
- **Quem faz:** ENG-03 configura; ENG-00 valida o desenho. Um ADR (ou seção desta DEC) detalha as variáveis; os nomes exatos são **confirmados contra a documentação do n8n 2.41.6** antes de codar.
- **Relação com o M02:** o despacho de agentes continua pela fila no PostgreSQL (o Redis não tinha papel no M02). Aqui o Redis serve à fila de execuções do n8n.
- **Testes obrigatórios da etapa 4:**
  1. fumaça M02 inteira em staging com editor e worker separados;
  2. **derrubar o worker** no meio de uma execução e confirmar que o `resume_lost`/varredores recuperam como hoje (R1);
  3. **derrubar o Redis no meio de uma execução:** o que estava em fila precisa ser **recuperado ou explicitamente marcado como perdido**, nunca sumir em silêncio; o resultado e a evidência ficam registrados.
- **Persistência do Redis:** o Redis agora carrega a fila de execuções do n8n, e o D4 (§3.7) cobre só o PostgreSQL. **PENDENTE (ENG-07, etapa 4):** volume persistente ou efêmero. Proposta a validar: **production com volume persistente (AOF)** e staging pode ser efêmero; a decisão final sai do teste 3 acima.

### 3.3 Exceção de `http` na rede privada (`config.ts:222`)
- A regra "N8N_WEBHOOK_URL_AGENT precisa ser https (http só para localhost)" ganha **uma exceção nomeada e restrita**: aceita `http` **se, e somente se, o hostname terminar em `.railway.internal`**.
- **Determinística:** sem flag para ligar ou desligar. O hostname vem de `new URL(valor).hostname` (nunca de comparação de texto no valor cru). `https` continua valendo para qualquer outro host.
- **Justificativa:** `.railway.internal` só resolve dentro da rede privada do Railway (tráfego em túneis WireGuard entre serviços do mesmo ambiente). Não é alcançável de fora.
- **Testes obrigatórios (ENG-06):** aceita `http://n8n.railway.internal:5678/...`; recusa `http://railway.internal.evil.com`, `http://evil.com/.railway.internal`, `http://x.railway.internal@evil.com`, `http://x.railway.internal.` (ponto final) e qualquer `http` público.
- **Maiúsculas e minúsculas:** `http://N8N.RAILWAY.INTERNAL:5678` deve ser tratado como `n8n.railway.internal`. A comparação usa o `hostname` já normalizado e **converte para minúsculas antes de comparar**, de modo que o maiúsculo **seja aceito pelo mesmo motivo** que o minúsculo, e não "por engano". Há teste com maiúsculas, com mistura (`N8n.Railway.Internal`) e com o sufixo em maiúsculas sobre um host externo (`http://evil.com.RAILWAY.INTERNAL.evil.com`, recusado).
- **Subdomínio encadeado:** `http://x.y.railway.internal`. **Regra proposta: RECUSADO.** Só é aceito o formato com **exatamente um rótulo** antes de `.railway.internal` (`<serviço>.railway.internal`), que é o que a DEC usa (`n8n.railway.internal`, `api.railway.internal`). Motivo: menor superfície e determinismo. **Reavaliar apenas se** o ENG-07 comprovar, no primeiro deploy do staging, que o Railway gera nomes com mais de um rótulo para o serviço que o n8n precisa; nesse caso, uma nova DEC de emenda autoriza o formato exato, nunca um curinga. **Esta regra precisa da confirmação da Madu.**
- Registrada como **exceção nomeada**, não como "abertura da regra".

### 3.4 Trava das fumaças (ENG-06 implementa, ENG-00 aprova)
- `assertLocalTarget` (`scripts/n8n/import.ts:73`) hoje só aceita `127.0.0.1`, `localhost` e `[::1]`. Passa a aceitar alvo remoto **somente** se:
  1. o alvo for declarado por variável explícita (`SMOKE_TARGET_URL`);
  2. o host estiver numa **allowlist** versionada (sem segredos) que contém **apenas o staging**;
  3. o host **não** estiver na lista de bloqueio, que contém o production, mesmo com override.
- Fora da allowlist: **recusa**. Existe um override explícito (`SMOKE_ALLOW_UNLISTED=…`) que exige confirmação em linha de comando e registra no log; **nunca** vale para o host de production.
- Mantém as travas de hoje: nome do container validado, execução só contra pilha de fumaça, `down -v` só com `-p`.
- Testes: aceita staging da allowlist; recusa production, host desconhecido, `127.0.0.1` com variável ausente (comportamento antigo preservado), URL malformada.

### 3.5 Importação dos workflows no Railway
- Reaproveita o comando de importação do PR #35 (`pnpm n8n:import`); **muda só o meio de execução**, de `docker exec` local para execução **dentro do container do n8n** no Railway.
- **Ajuste técnico:** o comando que executa dentro do container é **`railway ssh`**. `railway run` executa **localmente** com as variáveis do Railway injetadas e **não** serve para o CLI interno do n8n.
- A importação **reinicia o n8n** (como hoje): só em janela de manutenção.
- **PENDENTE:** a adaptação do `importWorkflows` para o meio remoto é ENG-03 + ENG-06; o ADR descreve o contrato.

### 3.6 D3: partição mensal do `audit_log`
- **Política** (DEC-0034): partição mensal, 24 meses no banco, partição mais antiga exportada antes de ser destacada, nada apagado sem decisão da Madu.
- **Sem partição `DEFAULT`:** se a partição do mês não existir, a gravação **falha de forma explícita**. O `audit_log` é imutável; falha silenciosa é inaceitável.
- **2 meses criados à frente.**
- **Alertas obrigatórios** (sem eles trocamos "buraco silencioso" por "parada silenciosa da operação"):
  1. o job de criação de partição não executou no prazo esperado;
  2. uma gravação falhou por partição ausente.
- **PENDENTE (Madu): destino da exportação** (object storage). **O D3 fica BLOQUEADO até essa decisão.** Nada será inventado.
- **PENDENTE:** canal dos alertas (o DEC-0034 prevê e-mail pelo n8n, adiado para antes do M04; definir o que vale já nesta fase), prazo esperado do job e responsável.
- Responsável: ENG-02. Migração reversível, com teste de imutabilidade em cada partição.

### 3.7 D4: restore testado
- **Serviço temporário** no Railway (staging), na mesma rede privada, com `psql` e `pg_restore`. Nada de acesso público ao banco.
- **Evidência obrigatória:** comando executado, horário, e contagem de linhas nas tabelas-chave (`event`, `audit_log`, `approval`, `agent_execution` e as demais que o ENG-02 listar) antes e depois; migrations aplicadas; abrir as credenciais do n8n com a chave.
- O serviço temporário é **derrubado** ao final. A DEC exige o registro da remoção.
- **PENDENTE:** onde ficam os arquivos de backup e por quanto tempo (ligado ao destino do object storage do D3).

### 3.8 Imagens por digest
- O PR da etapa 2 **registra os digests exatos** de `n8nio/n8n:2.41.6`, `postgres:17`, `redis:8` e da **imagem da API** em um **arquivo versionado** (sem segredos), e não só no `railway.json`. O `railway.json` e o Dockerfile **leem** esse arquivo; ele é a fonte única. Nome e local do arquivo: **PENDENTE** (ENG-07; proposta `infrastructure/image-digests.json`).
- Os digests **não são inventados**: são obtidos do registro no momento do PR e anexados como evidência (comando e saída).
- **Política de atualização:** trocar um digest só **por PR**, com a **evidência do changelog** da versão nova, CI verde e revisão independente. Vale para staging e production; production só recebe o digest depois de validado no staging.

## 4. O que NÃO muda
Arquitetura (API fina + n8n + PostgreSQL + Redis); agentes A0 a A10 e seus contratos; workflows de negócio e o padrão TRIGGER → VALIDATE → LOAD CONTEXT → EXECUTE → WRITE RESULT → EMIT EVENT → LOG → NOTIFY; permissões e gates G01 a G12; gates por aprovação humana da Madu; perfil de valores do DEC-0034; ausência de chamada real a LLM; `knowledge/plf/` não é publicado; `FASE  1 - BASE/` não é tocada.

## 5. Segredos
- Nunca versionados. Ficam em variáveis do Railway marcadas como segredo, por ambiente.
- **`N8N_ENCRYPTION_KEY` é segredo crítico:** idêntica entre editor e worker do mesmo ambiente; guardada no gerenciador de senhas da Madu, com uma cópia offline (DEC-0034). Perdê-la torna as credenciais do n8n irrecuperáveis.
- `JWT_SECRET`, senhas dos papéis do banco, `REDIS_PASSWORD`, senha do n8n: valores aleatórios por ambiente, gerados por quem conecta o Railway.
- A chave Anthropic **não entra** nesta fase.

## 6. Riscos
- **R1: custo acima do estimado** (ver §7).
- **R2: editor do n8n público.** O primeiro visitante de um n8n novo pode criar a conta de dono. **Mitigação obrigatória:** não gerar o domínio público antes de a conta de dono estar criada (etapa 3.1) e desabilitar o cadastro aberto. O editor público também serve os webhooks no mesmo domínio; a proteção deles é o cabeçalho `x-webhook-secret` já existente, e isso **não basta** para production.
  **Pré-requisito do deploy em production:** ou separar o domínio dos webhooks do domínio do editor, ou restringir os webhooks ao tráfego privado, já que a API os chama por `${{api.RAILWAY_PRIVATE_DOMAIN}}`. **Não deployar production com webhooks expostos no domínio do editor.** No staging o pré-requisito é avaliado e registrado, mas não bloqueia as fumaças.
- **R3: partição ausente derruba escritas** (§3.6). Mitigado por 2 meses à frente e pelos 2 alertas.
- **R4: sem proteção de branch** no GitHub; a revisão independente vale por convenção (DEC-0022).
- **R5: divergência entre staging e production.** Mesmas imagens por digest e mesmo Dockerfile; só mudam variáveis.
- **R6: dependência de um provedor** (Railway) e de internet. Plano de saída: o compose local continua funcionando.
- **R7: exceção `.railway.internal`** depende de o serviço de DNS interno do Railway se comportar como documentado; testes cobrem o parser, não o DNS.
- **R8: único serviço público sem rate limit nem 2FA definidos.** Mitigação: rate limit antes do domínio público (etapa 3.1) e 2FA, ou justificativa da ausência, antes de production (etapa 10); itens no §10.
- **R9: perda do Redis em execução:** coberta pelo teste obrigatório da etapa 4 (§3.2).

## 7. Custo e limite do M03 (ESTIMATIVA A MEDIR, não é fato)
| Item | Valor | Origem |
|---|---|---|
| Crédito incluído no plano Pro | US$ 20/mês | **Informado pela Madu**; não verificado pelo Claude Code |
| Custo estimado da stack | US$ 15 a 30/mês | Estimativa da Madu |
| Custo adicional provável | US$ 0 a 10/mês | Estimativa da Madu |
| Referência de mercado (terceiros) | US$ 10 por GB de RAM/mês; US$ 20 por vCPU/mês; US$ 0,15 por GB/mês de volume | Fontes públicas, a confirmar no painel |
- **Como medir:** relatório de uso do Railway por serviço depois de 7 dias com o staging ligado só nas janelas de teste e o production ligado, e registro dos números na DEC de fechamento.
- **Limite do M03 (inalterado, DEC-0034):** US$ 20 acumulados de uso da API Anthropic (`llm_cost_limit`), um único modelo (`claude-sonnet-5-5`), `llm_pricing` copiado da tabela oficial no dia. **Separado do custo do Railway.**

## 8. Plano de execução (cada passo é um PR ou uma etapa com parada)
| # | Etapa | Responsáveis | Parada |
|---|---|---|---|
| 1 | **Esta DEC e o ADR do modo fila** | ENG-00 | Aceite da Madu (dado) |
| 2 | **PR "Dockerfile e Railway":** Dockerfile da API (imagem por digest, healthcheck, start command explícitos, pre-deploy de migração), `railway.json` ou `railway.toml`, **arquivo versionado com os digests exatos e a política de atualização (§3.8)**, `HOST` configurável, docs de operação (sem segredos) | ENG-07, ENG-02 (migração), ENG-01 (`HOST`) | CI verde (3 jobs), revisão independente, merge só com a Madu |
| 3 | **Madu conecta o Railway:** cria o projeto e os dois ambientes e os segredos. O Claude Code confirma cada passo. **Domínio público só do editor, e só depois da etapa 3.1** | **Madu** | Cada ação externa é confirmada |
| 3.1 | **Conta de dono do n8n:** a Madu (ou agente autorizado por ela) cria a conta de dono do n8n **antes** de gerar o domínio público. Método definido previamente (ver §10). **Rate limit do login definido antes do domínio público** | ENG-03 + Madu | **Bloqueia a etapa 4** |
| 4 | **PR "Modo fila":** editor, worker, Redis; `N8N_ENCRYPTION_KEY` compartilhada; **testes de perda de worker e de Redis em execução (§3.2)**; decisão de persistência do Redis | ENG-03, ENG-00 valida, ENG-06 (testes), ENG-07 (Redis) | Idem. Só depois da etapa 3.1 |
| 5 | **PR "Trava das fumaças e exceção `.railway.internal`":** §3.3 e §3.4 | ENG-06, ENG-00 aprova | Idem |
| 6 | **Fumaças M01 (49) e M02 (31) contra o staging**, com evidência | ENG-06 | Falha: paro e reporto |
| 7 | **D4:** restore no staging, evidência, serviço temporário derrubado | ENG-07, ENG-06 | — |
| 8 | **D3:** **bloqueado** até a Madu decidir o destino da exportação | ENG-02 | **Decisão da Madu** |
| 9 | Documentação, `PROJECT_STATUS.md`, relatório de custo e DEC de fechamento | ENG-00 | Aceite da Madu |
| 10 | Deploy do production **somente** depois de tudo validado no staging **e** do pré-requisito de webhooks (§6, R2) **e** do 2FA do editor (ou justificativa registrada) | ENG-07, ENG-03 | **Autorização da Madu.** Bloqueado se os webhooks estiverem no domínio do editor |

## 9. Alternativas consideradas
- **Claude Code na nuvem** (VM de 16 GB com Docker): serve para verificação, mas **não persiste** e não é um ambiente de produção.
- **VPS pago** (Hostinger KVM 2, Hetzner CX33): viável; foi preterido por a Madu já ter o Railway e o plano Pro.
- **Ampliar a RAM e manter local:** não resolve o armazenamento nem a disponibilidade.
- **Clerk:** só autenticação, não resolve infraestrutura; fica para avaliação no M04.

## 10. Pendentes, responsáveis e o que bloqueiam
| PENDENTE | Responsável | Bloqueia |
|---|---|---|
| Destino da exportação do `audit_log` (object storage) | Madu | D3 (etapa 8) |
| Canal dos alertas do D3 | Madu | D3 (etapa 8) |
| Prazo esperado do job de partição e quem responde ao alerta | Madu + ENG-02 | D3 (etapa 8) |
| Local e retenção dos backups | Madu | D4 (etapa 7) |
| `0.0.0.0` ou `::` no Railway | ENG-07, no primeiro deploy do staging | Etapa 2 |
| Formato de nome com mais de um rótulo em `.railway.internal` (§3.3) | ENG-07, no primeiro deploy do staging | Etapa 5 |
| Confirmar a regra "um rótulo só" do §3.3 | Madu | Etapa 5 |
| Método de criar a conta de dono do n8n | ENG-03 + Madu | Etapa 3.1 |
| **Rate limit no login do n8n-editor** | Madu + ENG-07 | Etapa 3.1 (antes do domínio público) |
| **2FA no n8n-editor** (ou justificar a ausência) | Madu | Deploy em production (etapa 10) |
| Variáveis exatas do modo fila (n8n 2.41.6) | ENG-03 | Etapa 4 |
| **Persistência do Redis** (volume ou efêmero) | ENG-07 | Etapa 4 |
| **Teste de perda de Redis em execução** | ENG-06 | Etapa 4 |
| Separar o domínio dos webhooks ou restringi-los ao tráfego privado | ENG-03 + ENG-07 | Etapa 10 (deploy em production) |
| **Nome e local do arquivo de digests; digests exatos** de n8n 2.41.6, PostgreSQL 17, Redis 8 e da imagem da API | ENG-07 | Etapa 2 |
| Aceite da conexão do Railway (projeto, ambientes, segredos) | Madu | Etapas 2 e 3 |
| Custo real medido | Madu + ENG-07 | Etapa 9 |

**A atualizar depois do aceite:** `PROJECT_STATUS.md`, `BLOCKERS.md`, runbooks e o `CLAUDE.md` do repositório (§4 e §14 ainda dizem "ambientes PENDENTES" e o marco "M01 em F1").

Ver [[DEC-0034-perfil-de-valores-e-plano-ate-o-m03]] · [[DEC-0033-decisoes-pendentes-do-m01]] · [[DEC-0017-modelo-de-negocio-inicial]] · [[pre-M03-fase-D-plano]].
