---
tipo: fase
area: 04-Fases
fonte: DEC-0018; DEC-0010 §7; DEC-0013; DEC-0014; DEC-0016; blueprint v1 §3, §7, §10–§11, §40; starter (lido no scratchpad)
status: concluido em 2026-10-04 (DEC-0021)
atualizado: 2026-10-04
---
# M00 Foundation — plano de execução

Início formal: [[DEC-0018-inicio-formal-do-m00]]. **Fechamento: [[DEC-0021-m00-foundation-concluido]] (2026-10-04).** **Atualização:** o Maestri saiu do escopo ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]); os passos 6 e 7 foram ajustados. Nada abaixo foi executado, exceto a verificação de ambiente.

## Verificação de ambiente (2026-10-04, somente leitura)
| Item | Resultado |
|---|---|
| Node / Git | v24.16 / 2.54: ok |
| Claude Code | 2.1.289: ok |
| GitHub CLI | 2.98 autenticado como `madubrandao`, escopos `repo`, `read:org`, `gist` |
| Docker | CLI 29.5 instalado, mas **o daemon não está rodando** (Docker Desktop parado) |
| `core.autocrlf` | **`true` no gitconfig do sistema** (padrão do Git for Windows; correção: a primeira leitura indicou "não configurado" por ter consultado só o nível global). Converte LF para CRLF no checkout, por isso o `.gitattributes` com LF é necessário (DEC-0016 D5) |
| Disco `C:` | 14 GB livres de 222 GB (94% usado); as imagens Docker (Postgres, Redis, n8n) ocupam alguns GB |
| `C:\dev` | não existe |
| Maestri | fora do escopo ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]) |

## Passos
| # | Passo | Dono | Status |
|---|---|---|---|
| 1 | Iniciar o Docker Desktop e confirmar o daemon | Madu | [x] feito em 2026-10-04 (daemon 29.5.2) |
| 2 | Criar a pasta `C:\dev\launch-os` e `git init` com `.gitattributes` (LF) e `.gitignore` do starter | Claude | [x] feito em 2026-10-04 (commit local inicial; sem remote) |
| 3 | Criar o repositório privado `madubrandao/launch-os` no GitHub e ligar o `origin` (**ação externa**) | Claude, com confirmação | [x] feito em 2026-10-04: https://github.com/madubrandao/launch-os (PRIVATE, branch `main`, commit `3e0b786` enviado) |
| 4 | Importar o starter aplicando as correções do [[starter-zip-inventario]] (compose único em `infrastructure/compose/`, `docs/adr`, guardrails nos agentes, rules sem repetição) | Claude | [x] feito em 2026-10-04 (commit local no `launch-os`; ainda sem push). Desvios: `FILE_INDEX.txt` não importado (índice estático desatualiza); ADR-0001 a 0004 do starter não duplicados (já são DEC-0001 a 0004 do vault); `docs/ROADMAP.md` reescrito com M00-M06; chave OpenAI removida do `.env.example` (DEC-0016 D2); `name: launch-os` adicionado ao compose; textos do repo em português; `CLAUDE.md` com correções mínimas (substituído no passo 5) |
| 5 | `CLAUDE.md` raiz com as 14 seções do Blueprint §7.1 (TECH STACK = TypeScript/Node) e os 8 `CLAUDE.md` por domínio | Claude (ENG-00) | [x] feito em 2026-10-04 (raiz com 108 linhas, em português; domínios de 9-10 linhas; referenciam os guardrails em vez de copiá-los; commit local no `launch-os`, sem push) |
| 6 | ~~Notas pinadas do Maestri~~ → 6 notas de estado em `docs/status/` no `launch-os` ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]) | Claude | [x] feito em 2026-10-04 (commit local `aeb58d4`) |
| 7 | ~~Maestri: workspace, terminais ENG-00…07, conexões~~ | — | removido do escopo ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]) |
| 8 | Docker de desenvolvimento: compose com **versões fixas** (inclui o pin do `n8n`), `.env.example`, `bootstrap.ps1`; `docker compose up` com healthchecks verdes | Claude (ENG-07) | [x] feito em 2026-10-04: postgres 17.11, redis 8.10.2, n8n 2.41.6; 3 serviços healthy, `/healthz` 200; n8n na porta 5680 por conflito; `N8N_BASIC_AUTH_ACTIVE` removida; `.env` local com segredos aleatórios; pilha parada com `down` (volumes preservados); commit local no `launch-os` ([[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]]) |
| 9 | Esqueleto de CI (GitHub Actions, sem lógica de negócio) | Claude (ENG-07) | [x] implementado e testado localmente em 2026-10-04 (commit local `effbd86` no `launch-os`); **validado no GitHub Actions**: execução `37238422875` no commit `effbd86`, `success`, as 10 verificações `ok` e `RESULTADO: aprovado` no log; push feito |
| 10 | Documentação de arquitetura **referenciada** no repo, não copiada (fonte única, DEC-0013 D4) | Claude (ENG-00) | [x] feito em 2026-10-04 (commit local no `launch-os`; ainda sem push): `docs/architecture/README.md` indexa o vault sem copiar, mais ponteiros em `docs/agents`, `docs/events` e `docs/workflows`; CI exige os 4 arquivos |
| 11 | Verificação final contra [[definicao-de-pronto]] e fechamento do M00 | Madu aprova | [x] verificação e revisão independente completa feitas; **M00 fechado pela Madu em 2026-10-04** ([[DEC-0021-m00-foundation-concluido]]) |

## Pendências do M00
- Caminho local (`C:\dev\launch-os`) e nome do repositório: confirmar.
- ~~Maestri instalado?~~ Maestri fora do escopo.
- ~~Versão fixa do `n8n`~~: resolvida no [[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]] (2.41.6).
- ~~Onde ficam as notas do Maestri~~ → `docs/status/` ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]).
- **Nova:** relação entre as decisões do vault (`DEC-00XX`) e os ADRs do repo (`docs/adr/`). Duplicar cria duas fontes. Proposta a decidir: o vault guarda as decisões do projeto; `docs/adr/` guarda só ADRs técnicos do código, referenciando o vault quando preciso.
- Bloco B de [[perguntas-em-aberto]] (material faltando), não bloqueia o M00.

## Observações do passo 4
- O compose agora fica em `infrastructure/compose/`; o `.env` fica na raiz. Por isso o comando exige `--env-file .env` (documentado no `README` do repo). `docker compose config` valida a sintaxe, mas **o `up` só será testado no passo 8**, com o Docker rodando.
- `N8N_BASIC_AUTH_ACTIVE` foi mantida como veio do starter; verificar no passo 8 se a versão fixada do n8n ainda usa essa variável.
- `docs/DEFINITION_OF_DONE.md` do repo é cópia operacional da nota `definicao-de-pronto` do vault; em caso de divergência vale o vault até decisão contrária.

## Observações do passo 8
- A máquina tem outros 4 containers de outros projetos (dois n8n e dois Postgres), que não foram tocados. Portas 5678 e 5679 ocupadas, por isso o n8n do LAUNCH OS usa a 5680.
- Disco `C:` com cerca de 12 GB livres (95% usado) depois do download das imagens.
- n8n avisa que o task runner interno é obsoleto (`N8N_RUNNERS_MODE`): avaliar o runner externo quando houver workflows com código.

## Observações do passo 9
- `scripts/ci/check-repo.sh` valida: 20 arquivos obrigatórios, 14 seções do `CLAUDE.md`, 8 agentes com Guardrails, ausência de `.env`/chaves/padrões de segredo, sem CRLF no índice, imagens do compose e `N8N_IMAGE` com versão exata, `docker compose config`, e referências descartadas (DEC-0013). Roda também localmente: `bash scripts/ci/check-repo.sh`.
- `.github/workflows/ci.yml` roda o script em push na `main` e em pull requests, com `permissions: contents: read` e `actions/checkout` v7.0.1 fixado pelo SHA do commit.
- Testado: aprovado no estado atual; reprovado com 6 violações injetadas em uma cópia.
- Ainda não há testes de código nem lint: não existe código de aplicação (entram a partir do M01).

## Incidente durante o passo 9 (2026-10-04)
O primeiro teste negativo rodou no repositório real, porque o `cp` falhou numa cadeia `&&` e os comandos seguintes executaram fora da cópia. Efeitos e reversão:
- Arquivos rastreados alterados (guardrails do ENG-03, tag do Redis no compose) e `x.txt` com token falso: revertidos com `git reset` e `git checkout`; nada foi commitado nem enviado; o repositório ficou igual ao `HEAD`.
- O `.env` local (ignorado pelo Git) foi sobrescrito e **os segredos anteriores se perderam**; foi regenerado com novos valores aleatórios.
- **Consequência resolvida:** os 3 volumes `launch-os_*` tinham a senha antiga do Postgres; a Madu autorizou e eles foram removidos (nenhum container os usava; volumes dos outros projetos intactos). A pilha foi subida de novo com o `.env` regenerado: 3 serviços `healthy`, `/healthz` 200; depois `down`, com os volumes novos preservados.
- O teste foi refeito com trava que aborta fora da cópia.

## Pesquisa sobre o Maestri (2026-10-04; histórico: o Maestri saiu do escopo no [[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]])
- **Fontes de terceiros** (Setapp, AlternativeTo, Product Hunt e resenhas) dizem que o Maestri é um app nativo de macOS, sem versão para Windows ou Linux. **O site oficial** (`themaestri.app`) tem páginas próprias para Windows e Linux e, na página do Windows, informa **versão disponível agora para Windows 11 x64**, com instalador e versão portátil no CDN oficial (`cdn.maestri.dev/win/x64/`). Como o site oficial é a fonte mais recente e primária, vale ele; a divergência indica que as resenhas estão desatualizadas. Não confirmei a versão nem testei o instalador.
- **Preço (site oficial):** gratuito permanente com agentes ilimitados, canvas infinito, conexões entre agentes e **1 workspace**; Pro com pagamento único de R$95, workspaces ilimitados e uso em até 2 dispositivos (Mac ou Windows), com 7 dias de teste.
- **Impacto no plano:** o Blueprint §3 e o `MAESTRI_SETUP.md` pedem **8 workspaces**. Com o plano gratuito (1 workspace) não dá para criá-los todos: é preciso o Pro (decisão e custo da Madu) ou adaptar o desenho a 1 workspace. **PENDENTE.**
- **Instalação:** o download do instalador foi bloqueado pelo classificador de segurança do Claude Code (executável externo). Nada foi baixado nem instalado. A decisão de como proceder (a Madu baixar, ou liberar a permissão) está com a Madu.

## Passo 11: verificação final (2026-10-04)
Evidências coletadas depois do push de `aeb58d4` (`launch-os`) e `7246f9f` (vault):
- **CI:** execução `37241151698`, `success`, commit `aeb58d4`; 10 verificações `ok`, 32 arquivos obrigatórios.
- **Repositórios:** `launch-os` privado, branch `main`, 86 arquivos rastreados, 7 commits, sincronizado com o GitHub; vault privado, 70 arquivos, sincronizado.
- **Segredos:** 0 padrões de segredo no histórico dos dois repositórios; nenhum `.env`, `.pem` ou `.key` jamais versionado; `.env` local ignorado e com segredos aleatórios.
- **Docker:** postgres 17.11, redis 8.10.2 e n8n 2.41.6 `healthy`; `/healthz` do n8n 200; Redis `PONG`; Postgres aceitando conexões; pilha encerrada com `down`; containers dos outros projetos intactos.
- **Estrutura:** as 36 pastas do Blueprint §8 presentes, 13 domínios de n8n, nenhum PDF em `knowledge/plf/`.
- **Agentes e CLAUDE.md:** 8 agentes `eng-xx` com guardrails (5 em cada, 6 no ENG-04, como no DEC-0010 §8); `CLAUDE.md` raiz com 14 seções e 8 por domínio.
- **Documentação:** wikilinks de todo o vault conferidos, sem links quebrados; textos desatualizados corrigidos nesta verificação.

### Definition of Done aplicada à fundação
| Critério | Situação |
|---|---|
| código implementado | N/A: o M00 não tem código de aplicação (só configuração e scripts) |
| testes escritos e passando | parcial: a CI valida estrutura e configuração; testes de código entram no M01 |
| logs implementados, erros tratados | N/A (sem código); o `bootstrap.ps1` falha com mensagem clara |
| permissões verificadas | CI com `contents: read`; sem segredos versionados |
| documentação atualizada | atendido |
| evento definido, schema versionado | N/A no M00 (M01) |
| workflow n8n conectado | N/A no M00 (M03) |
| dashboard mínimo | N/A no M00 (M06) |
| ADR atualizado se houve decisão arquitetural | atendido: DEC-0011 a DEC-0020 no vault; `docs/adr/` pronto, sem ADR técnico ainda |
| revisão por outro agente | **atendida em 2026-10-04**: 1ª passada parcial (`docs/status/`) e 2ª passada completa do repositório (seção abaixo) |
| contratos explícitos, impacto de segurança, sem estado local commitado, deploy entendido, evidência de verificação | atendido (esta seção) |

### Riscos e pendências para a Madu ao fechar
- **Disco `C:` com 8,9 GB livres (97% usado)**, caindo a cada imagem Docker baixada; precisa de limpeza antes do M01.
- Pendências herdadas, nenhuma bloqueia o fechamento: ver `docs/status/BLOCKERS.md` no `launch-os`.

## Revisão independente e limpeza de recursos (2026-10-04)
**Revisão independente (parcial):** rodada com `/code-review` (nível high) sobre `C:\dev\launch-os`. Achou 5 inconsistências, todas em `docs/status/` (o que o último commit alterou): verificação marcada como feita com a revisão ainda aberta; `BLOCKERS.md` contraditório ("nenhum bloqueio" com itens pendentes); número de disco volátil sem critério; `CURRENT_TASK.md` desalinhado; três níveis sem definição. Nenhum defeito em código ou configuração foi apontado, mas **não há evidência de cobertura do repositório inteiro**: a revisão completa segue como decisão em aberto da Madu. As 5 correções foram aplicadas em `docs/status/`.

**Disco (autorizado pela Madu):** `npm cache clean --force` (cache de 13 GB para 969 MB) e `pnpm store prune` (721 MB, 273 pacotes). `C:` passou de **8,8 GB para 22 GB livres** (97% para 91% usado), acima do critério de 20 GB definido pela Madu para autorizar o M01. Não foram tocados: Clipchamp (16 GB), app Claude (9,5 GB), Chrome (10 GB), imagens e volumes do Docker de outros projetos, nem o VHDX do Docker (compactação não autorizada).

**RAM:** 7,7 GB no total, ~0,8 GB livre (90% em uso), com 4 containers de outros projetos rodando (`qg-madu-n8n`, `n8n-n8n-1`, `app-postgres_app-1`, `n8n-postgres_n8n-1`, todos `unless-stopped`). A Madu autorizou pará-los enquanto se trabalha no LAUNCH OS; **ainda não foram parados**, porque a pilha do LAUNCH OS está desligada. Comandos de parar e religar em `docs/status/BLOCKERS.md` do `launch-os`.

## Revisão independente completa (2026-10-04)
Um revisor independente (subagente com contexto novo, somente leitura) leu **47 de 47 arquivos não vazios** do `launch-os` (os 39 `.gitkeep` têm 0 byte; total de 86 versionados) e cruzou o repositório com os DEC-0010, 0012, 0013, 0015, 0016, 0019 e 0020. Não leu DEC-0001 a 0009, 0011, 0014 e 0017, as fichas A0-A10 e algumas notas de arquitetura e fases do vault: isso é do vault, fora do repositório de código. Resultado: **nenhum achado de gravidade alta, nenhum segredo real versionado**, guardrails dos 8 agentes fiéis ao DEC-0010 §8, roadmap, entidades, gates e eventos coerentes. **10 achados** (2 de gravidade média, 8 baixa ou média-baixa), todos confirmados por leitura e corrigidos:
1. Segredos com fallback `CHANGE_ME` e portas em todas as interfaces: segredos obrigatórios (`${VAR:?}`) e portas em `127.0.0.1`.
2. Redis sem senha: com a porta local o risco cai; senha do Redis **PENDENTE para o M01** (decisão da Madu).
3. n8n e banco de negócio compartilham banco e superusuário, sem decisão registrada: registrado como PENDENTE (M01), não decidido.
4. Guardrails copiados em 7 `CLAUDE.md` e 3 `rules/`: substituídos por referência ao `ENG-xx`; a CI agora detecta cópia exata.
5. Versões e porta repetidas em 5+ lugares: docs apontam para compose e `.env.example`; a CI compara compose e `.env.example`.
6. `check-repo.sh` com falsos negativos: 14 títulos na ordem, `.env.*`, `github_pat_`, `sk-`, `xox`, fim de linha misto, caminhos com espaço, tags de imagem, arquivos de CI na lista obrigatória.
7. `.gitignore`: acrescentados `.mcp.json`, `.claude/settings.local.json`, dumps e IDE. **Desvio do relatório:** `*.sql` **não** foi ignorado, porque as migrations de `database/migrations/` serão `.sql`.
8. `bootstrap.ps1`: roda de qualquer pasta, gera segredos aleatórios, leitura em UTF-8, falha clara com Docker parado.
9. `ci.yml`: `persist-credentials: false`, runner `ubuntu-24.04`, cancelamento só em pull request.
10. Textos: CI/CD, backup, logs e atualização de versões como PENDENTES; `JWT_SECRET` e variáveis sem uso marcados como reservados; nome da pessoa removido da regra de push.

**Testes das correções:** `check-repo.sh` aprovado no estado atual; em uma cópia isolada com 10 violações injetadas reprovou (saída 1) e apontou todas; `bootstrap.ps1` testado no PowerShell 7 e no Windows PowerShell 5.1, chamado de outra pasta (`.env` na raiz, 0 `CHANGE_ME`, sem BOM, acentos, LF, segunda execução não sobrescreve) e com o Docker "parado" simulado por um `docker` falso (mensagem clara, sem `NativeCommandError`); pilha real subida com o compose novo (3 serviços `healthy`, `/healthz` 200, `netstat` mostra `127.0.0.1:5432`, `5680` e `6379`) e o compose sem `.env` falha com "required variable POSTGRES_PASSWORD is missing". Para o teste, os 4 containers de outros projetos foram parados e religados no fim, e voltaram como estavam.

## Riscos conhecidos
- Docker parado: o passo 8 não passa sem o daemon.
- Com `core.autocrlf=true` no sistema, sem `.gitattributes` os scripts e o JSON do n8n podem quebrar por CRLF (mitigado no passo 2; o repo usa `core.autocrlf=false` local).
- O `bootstrap.ps1` do starter tem texto com acento e pode ler errado no Windows PowerShell 5.1 se estiver sem BOM: verificar no passo 8.
- Pouco espaço em disco: acompanhar o tamanho das imagens e volumes Docker.
- Criar o repositório no GitHub e fazer push são ações externas: só com confirmação.
- O `docker-compose.yml` do starter usa `n8nio/n8n:latest` e `CHANGE_ME` como senhas padrão: não usar assim fora do ambiente local.

Ver [[milestones]] · [[blueprint-de-execucao]] · [[roadmap]].
