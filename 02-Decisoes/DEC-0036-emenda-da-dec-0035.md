---
tipo: decisao
area: 02-Decisoes
id: DEC-0036
status: ACEITA (texto final aprovado pela Madu em 2026-10-11, com 2 ajustes, a decisão pela opção C, o veredito sobre P2 a P6 e a adição do limite explícito da verificação reduzida)
fonte: DEC-0035 (§3.2, §3.3, §3.4, §8, §10); PR #38 (revisão independente, achado 1); apps/api/src/config.ts:215-223; packages/events/src/transport.ts:78-82; scripts/n8n/import.ts:73-160; scripts/n8n/import-cli.ts; scripts/smoke/m02-smoke.ts:112-135, 162-165, 257-296; apps/api/src/gateway/step.ts:724; infrastructure/railway/api.Dockerfile; .dockerignore
data-registro: 2026-10-11
emenda: DEC-0035
---
# DEC-0036: emenda à DEC-0035 (exceção nos dois pontos do código, reordenação das etapas e verificação reduzida no staging)

> **Aceita pela Madu em 2026-10-11.** A DEC-0035 continua válida em tudo que esta DEC não altera. O M03 continua **não autorizado**. Nada é criado no Railway sem a confirmação da Madu a cada passo. Sem chamada real a LLM.

## 1. Contexto
O PR #38 (etapa 2 da DEC-0035) foi mesclado (`cb66543`). A revisão independente e a leitura do código encontraram pontos que a DEC-0035 não cobria: (a) a regra "`http` só para localhost" existe em **dois** lugares; (b) a ordem das etapas deixaria a API no ar entre as etapas 3 e 5 com URLs de webhook provisórias; (c) a fumaça atual (M01 e M02) **não consegue** rodar do notebook contra o Railway: roda a API em processo local, cria o banco de teste direto no PostgreSQL (privado no Railway) e usa o CLI do Docker.

## 2. Decisões

### 2.1 A exceção `.railway.internal` vale nos DOIS pontos do código
A exceção nomeada do §3.3 da DEC-0035 se aplica a **ambos** os locais: `apps/api/src/config.ts` (`N8N_WEBHOOK_URL_AGENT`) e `packages/events/src/transport.ts` (`N8N_WEBHOOK_URL_CREATE_LAUNCH`, `_APPROVAL_RESUME` e `_NOTIFY`). **A mesma regra, de um rótulo só e sem diferenciar maiúsculas de minúsculas, vale nos dois.**
- **Regra:** `http` é aceito se, e somente se, o `hostname` (de `new URL(valor).hostname`, convertido para minúsculas) for `<serviço>.railway.internal`, com **exatamente um** rótulo antes do sufixo. Recusados: dois ou mais rótulos (`x.y.railway.internal`), ponto final, userinfo (`x.railway.internal@evil.com`), sufixo em host externo (`railway.internal.evil.com`, `evil.com/.railway.internal`).
- **Implementação única:** `packages/events/src/url-policy.ts`, exportada como `isAllowedWebhookUrl(url)`, usada pelos dois pontos para a regra não divergir. As mensagens de erro atuais (por exemplo, "precisa ser https") são preservadas. **Direção da dependência:** `apps/api` já depende de `@launch-os/events` (o `server.ts` importa dele) e `transport.ts` já mora no pacote; colocar a regra em `apps/api` inverteria a direção. **ENG-00 confirma ou corrige a localização; não é bloqueio.**
- **Testes obrigatórios (ENG-06), para os dois pontos:** aceita `http://n8n-editor.railway.internal:5678/...`; aceita maiúsculas e mistura (`N8N.RAILWAY.INTERNAL`, `N8n.Railway.Internal`) pelo mesmo motivo que o minúsculo; recusa `http://x.y.railway.internal`, `http://x.railway.internal.`, `http://x.railway.internal@evil.com`, `http://railway.internal.evil.com`, `http://evil.com/.railway.internal`, `http://evil.com.RAILWAY.INTERNAL.evil.com` e qualquer `http` público; `https` e `http` de loopback continuam aceitos.
- Nenhuma flag liga ou desliga a exceção. Registrada como **exceção nomeada**, não como abertura da regra.

### 2.2 URLs provisórias de webhook: PENDENTE de verificação, não bloqueio
**PENDENTE (verificação, não bloqueio):** o primeiro deploy da API no staging (etapa 4) deve usar as URLs privadas finais. Após o deploy, o ENG-07 confirma no painel do Railway que as 4 variáveis (`N8N_WEBHOOK_URL_AGENT`, `_CREATE_LAUNCH`, `_APPROVAL_RESUME`, `_NOTIFY`) contêm `http://<serviço>.railway.internal:...` e **não** o loopback provisório. Se contiverem loopback, corrigir e redeployar **antes da etapa 6**.
Com a nova ordem (§2.3), a exceção é mesclada antes da conexão do Railway; o valor provisório descrito no `RAILWAY-RUNBOOK.md` (§3.2 e §6, passo 3) só seria necessário se a API fosse implantada antes disso, e o runbook será atualizado na etapa 3a.

### 2.3 Reordenação das etapas
A exceção sobe para antes da conexão do Railway, para não haver deploy intermediário quebrado. A numeração da DEC-0035 §8 passa a ser:

| Etapa | Conteúdo | Responsáveis | Parada |
|---|---|---|---|
| 1 | DEC-0035 e ADR | ENG-00 | Concluída |
| 2 | PR "Dockerfile e Railway" | ENG-07, ENG-02, ENG-01 | **Concluída** (PR #38, `cb66543`) |
| 3a | PR "Exceção `.railway.internal`" (§2.1) | ENG-06, ENG-00 aprova | CI verde, revisão independente, merge só com a Madu |
| 3b | PR "Trava das fumaças e importação remota" (§2.4), depois da 3a | ENG-06, ENG-03, ENG-00 aprova | Idem |
| 4 | Madu conecta o Railway (projeto, dois ambientes, segredos; domínio só do editor, e só depois da 4.1) | **Madu** | Cada ação externa confirmada |
| 4.1 | Conta de dono do n8n antes do domínio público; rate limit do login definido | ENG-03 + Madu | Bloqueia a etapa 5 |
| 5 | PR "Modo fila" (editor, worker, Redis; testes de perda de worker e de Redis; persistência do Redis) | ENG-03, ENG-00, ENG-06, ENG-07 | Idem |
| 6 | **Verificação reduzida no staging** (§2.4) | ENG-06 | Falha em qualquer um dos 7 itens: não avança para production |
| 7 | D4: restore no staging | ENG-07, ENG-06 | — |
| 8 | D3: **bloqueada** até a Madu decidir o destino da exportação | ENG-02 | Decisão da Madu |
| 9 | Documentação, `PROJECT_STATUS.md`, relatório de custo, DEC de fechamento | ENG-00 | Aceite da Madu |
| 10 | Deploy em production, só com tudo validado no staging, os webhooks fora do domínio do editor e o 2FA (ou justificativa registrada) | ENG-07, ENG-03 | Autorização da Madu |

### 2.4 Fumaças: opção C (decisão da Madu)
**O fato:** a fumaça M02 (e a M01) roda a API dentro do processo do notebook (`m02-smoke.ts:257-274`), cria o banco de teste direto no PostgreSQL (`m02-smoke.ts:162-165`) e usa o CLI do Docker (`import.ts:93-160`). Nada disso alcança o staging, cujo Postgres é privado e cujo n8n não enxerga o notebook.

**Decisão: opção C.** As fumaças completas **M01 (49) e M02 (31) continuam rodando LOCALMENTE**, com os **mesmos digests** de `infrastructure/image-digests.json`, como **critério de aceite do código**. No staging roda uma **verificação reduzida e explícita**. Motivo: as fumaças testam o comportamento interno do motor de agentes (retries, timeouts, `resume_lost`, injeção de falha), que **não muda** entre local e Railway; o que muda no Railway é infraestrutura (rede privada, DNS interno, variáveis, TLS do editor), e isso as fumaças completas não testam.

**Verificação reduzida da etapa 6, no mínimo, no staging:**
1. O healthcheck da API responde 200.
2. As migrations estão aplicadas (contagem esperada: 13).
3. A importação dos workflows via `railway ssh` funciona.
4. O gatilho chega ao n8n, o n8n chama o gateway, o passo termina com falha controlada `llm_not_configured`, registrada na execução e no audit, **sem nenhuma chamada a LLM**.
5. Pelo menos 1 evento chega ao destino (valida a exceção `.railway.internal` em `transport.ts`).
6. Um gate de aprovação humana (por exemplo, G01) é concedido (ver P3 sobre o usuário operador).
7. O log de auditoria registra os eventos acima.

**Se qualquer um dos 7 itens falhar, o deploy não avança para production.** Se a Madu quiser cobertura total das fumaças no Railway no futuro, abre-se uma **DEC nova**; não agora.

**Limite explícito:** esta verificação reduzida **NÃO** cobre o comportamento do agente com LLM real (produção de saída correta, retries em cima de falha do modelo, custo real). Essa cobertura fica para o M03, quando a chave Anthropic da Madu entrar e o `llm_cost_limit` estiver ativo. A verificação da etapa 6 prova apenas a cadeia de infraestrutura, não a qualidade do agente.

**Efeito na DEC-0035:** a linha da etapa 6 ("Fumaças M01 (49) e M02 (31) contra o staging") e o teste 1 do §3.2 ("fumaça M02 inteira em staging") passam a ser a verificação de 7 itens. Os testes 2 e 3 do §3.2 (derrubar o worker e derrubar o Redis no meio de uma execução) **continuam** na etapa 5 e **não podem virar generalidade**: o procedimento concreto deles é a pendência P6.

**Efeito na trava (etapa 3b), escopo reduzido:**
- **Allowlist do staging: não é necessária** (nenhuma fumaça vai ao Railway). Substitui o item 2 do §3.4 da DEC-0035.
- **Blocklist do production: continua**, como proteção contra acidente; vale para fumaças. Os nomes do production só existem depois da etapa 4; **até lá, qualquer alvo remoto continua recusado** (comportamento atual, falha fechada).
- **Importação de workflows no production** é um passo legítimo de deploy (etapa 10), diferente de uma fumaça, e exige **confirmação explícita própria** (além do `--confirm <container>` de hoje). Forma exata: P4.
- **Adaptação do `import-cli.ts` para `railway ssh`: continua necessária** (item 3 da verificação). Reaproveita a lógica do comando da PR #35; muda só o meio de execução. `railway run` executa localmente e não serve.
- A trava passa a abranger **os três pontos** onde ela existe: `assertLocalTarget` (`import.ts:73`), `main()` da fumaça M02 (container `launch-os-smoke*`, URL local, banco local) e o `import-cli.ts` (`N8N_CONTAINER`, `N8N_URL`).

### 2.5 Pontos de implementação (veredito da Madu)
- **P2, o "resultado" da verificação 4: APROVADO.** O staging não tem chave Anthropic nem ferramentas `test.*` (só carregam com `NODE_ENV=test`); sem LLM, o passo falha com `llm_not_configured` (`step.ts:724`). O critério é o do item 4 da verificação: o gatilho chega ao n8n, o n8n chama o gateway, o passo termina com falha controlada `llm_not_configured`, registrada na execução e no audit, sem nenhuma chamada a LLM.
- **P3, o usuário operador da verificação 6: DELEGADO a ENG-07 e ENG-06, com restrição.** O `bootstrap` usa o papel migrador direto no banco, e a imagem da API **não contém** `apps/cli/src` (excluído no PR #38). **Incluir `apps/cli/src` na imagem CONTRADIZ a escolha do PR #38:** se for o caminho, o ENG-07 deve justificar a superfície adicional. **Alternativa preferida (registrada):** criar o operador por **SQL direto no banco**, pelo serviço temporário da D4 (`psql` na mesma rede privada); é configuração única (one-time setup), não runtime. O ENG-07 decide, com essa preferência registrada.
- **P4, a blocklist e a importação no production: APROVADO.** A blocklist vale para fumaças; a importação em production é passo legítimo de deploy, com confirmação explícita própria. Forma exata: ENG-06 e ENG-00.
- **P5, a testabilidade da 3b: APROVADO.** Testes unitários localmente na 3b; a prova real da importação remota é o item 3 da etapa 6.
- **P6, testes de perda de worker e de Redis: APROVADO.** Procedimento concreto na etapa 5 (ENG-06). Não pode virar generalidade.

## 3. O que NÃO muda
Tudo da DEC-0035 não citado aqui: dois ambientes, API só em rede privada, editor do n8n como único serviço público, modo fila, D3 bloqueada, D4, digests, segredos, custo como estimativa a medir, `HOST`, gates e arquitetura dos agentes. `knowledge/plf/` e `FASE  1 - BASE/` não são tocados.

## 4. Pendentes
| PENDENTE | Responsável | Bloqueia |
|---|---|---|
| Confirmar a localização `packages/events/src/url-policy.ts` e o nome `isAllowedWebhookUrl` | ENG-00 | Não bloqueia (etapa 3a) |
| Verificação: as 4 URLs de webhook no painel contêm `http://<serviço>.railway.internal:...` e não o loopback | ENG-07 | Etapa 6 (não bloqueia o deploy da etapa 4) |
| Atualizar o `RAILWAY-RUNBOOK.md` (§3.2 e §6, passo 3) para a nova ordem | ENG-07 | Etapa 3a |
| P3: meio de criar o usuário operador no staging (preferência: SQL pelo serviço temporário da D4; incluir a CLI na imagem exige justificativa) | ENG-07 + ENG-06 | Etapa 6 |
| P4: forma da blocklist e da confirmação da importação no production | ENG-06 + ENG-00 | Etapa 3b |
| P5: prova real da importação remota (item 3 da verificação) | ENG-06 | Etapa 6 |
| P6: procedimento concreto dos testes de perda de worker e de Redis | ENG-06 | Etapa 5 |
| Demais pendentes da DEC-0035 §10 | os mesmos | os mesmos, com a numeração do §2.3 |

## 5. Adendo de 2026-10-11 (aprovado pela Madu durante a etapa 3a, PR #39)

### 5.1 Nova condição de rejeição: credenciais na URL
A política das URLs de webhook (§2.1) ganha **uma condição adicional de rejeição**: a URL **não pode trazer usuário nem senha**, em nenhum protocolo e para nenhum host (`https://usuario:senha@host`, `http://usuario@127.0.0.1`).
- **Como se decide:** depois do parser, por `url.username !== ''` ou `url.password !== ''`. **Nunca por heurística sobre o texto cru** (procurar `@` na string): um `@` na query, no fragmento ou no caminho não é credencial e não é recusado; userinfo vazio (`https://@host`) o parser descarta e é aceito.
- **Ordem das verificações:** primeiro o protocolo e o host real (inseguro), depois as credenciais. `http://x.railway.internal@evil.com` continua recusado como host inseguro, porque o host real é `evil.com`.
- **Motivo:** o `fetch` do Node recusa credenciais na URL na hora da chamada (verificado na revisão do PR #39), então sem esta regra a API subiria e todo despacho falharia; o segredo do webhook vai em cabeçalho, nunca na URL; evita segredo em log; e mantém `config.ts` e `transport.ts` idênticos.
- **A exceção `.railway.internal` do §2.1 não muda.** A mensagem é fixa ("não pode conter usuário nem senha na URL") e não repete a URL nem o segredo.
- Vale nos dois pontos (`apps/api/src/config.ts` e `packages/events/src/transport.ts`), pela mesma função `packages/events/src/url-policy.ts`, com testes.

### 5.2 Comentário obrigatório e comportamento fixado em teste
O código da política documenta, e os testes fixam: a política avalia o **hostname normalizado pelo parser**, não o texto cru. Codificações que normalizam para o **mesmo** host privado (`n8n%2erailway.internal`, ponto ideográfico, porta padrão, IDN em punycode) são **aceitas por desenho**, porque o destino do `fetch` é o host privado; armadilhas que normalizam para **outro** host (`:80@evil.com`, barra invertida, `#`, `?`, `%00@`) são **recusadas**.

### 5.3 Terceiro ponto com regra de `https`: `scripts/n8n/markers.ts` vai para a etapa 3b
`checkApiBaseUrl` (`scripts/n8n/markers.ts`) valida a URL pela qual o **n8n chama a API** (carrega o token de execução) e hoje aceita `http` só para `localhost`, `127.0.0.1`, `[::1]` e `host.docker.internal`; com a API só em rede privada, `http://api.railway.internal:3000` seria recusado na importação.
- **Decisão:** tratar na **etapa 3b**, reaproveitando `isPrivateRailwayHost` (a mesma regra de um rótulo só, sem diferenciar maiúsculas de minúsculas). **Não se mistura** com a política das URLs de webhook: é outro consumidor e outra semântica (a URL da API, não a dos webhooks do n8n).
- Se a 3b também recusar credenciais nessa URL é decisão do PR da 3b; fica como item de revisão.

### 5.4 Pendentes acrescentados
| PENDENTE | Responsável | Bloqueia |
|---|---|---|
| Tratar `checkApiBaseUrl` com `isPrivateRailwayHost` (§5.3) | ENG-06 | Etapa 3b |
| Nomes de serviço do Railway com `_` ou outros caracteres fora da regra do rótulo (a API recusa subir; falha visível) | ENG-07 | Primeiro deploy do staging |

Ver [[DEC-0035-migracao-para-o-railway]] · [[DEC-0034-perfil-de-valores-e-plano-ate-o-m03]].
