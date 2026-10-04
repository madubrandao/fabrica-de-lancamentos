---
tipo: fase
area: 04-Fases
fonte: DEC-0018; DEC-0010 §7; DEC-0013; DEC-0014; DEC-0016; blueprint v1 §3, §7, §10–§11, §40; starter (lido no scratchpad)
status: proposto (aguardando confirmação da Madu para executar)
atualizado: 2026-10-04
---
# M00 Foundation — plano de execução

Início formal: [[DEC-0018-inicio-formal-do-m00]]. Nada abaixo foi executado, exceto a verificação de ambiente.

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
| Maestri | **não verificado**: não encontrei instalação em `AppData\Local` |

## Passos
| # | Passo | Dono | Status |
|---|---|---|---|
| 1 | Iniciar o Docker Desktop e confirmar o daemon | Madu | [ ] |
| 2 | Criar a pasta `C:\dev\launch-os` e `git init` com `.gitattributes` (LF) e `.gitignore` do starter | Claude | [x] feito em 2026-10-04 (commit local inicial; sem remote) |
| 3 | Criar o repositório privado `madubrandao/launch-os` no GitHub e ligar o `origin` (**ação externa**) | Claude, com confirmação | [x] feito em 2026-10-04: https://github.com/madubrandao/launch-os (PRIVATE, branch `main`, commit `3e0b786` enviado) |
| 4 | Importar o starter aplicando as correções do [[starter-zip-inventario]] (compose único em `infrastructure/compose/`, `docs/adr`, guardrails nos agentes, rules sem repetição) | Claude | [x] feito em 2026-10-04 (commit local no `launch-os`; ainda sem push). Desvios: `FILE_INDEX.txt` não importado (índice estático desatualiza); ADR-0001 a 0004 do starter não duplicados (já são DEC-0001 a 0004 do vault); `docs/ROADMAP.md` reescrito com M00-M06; chave OpenAI removida do `.env.example` (DEC-0016 D2); `name: launch-os` adicionado ao compose; textos do repo em português; `CLAUDE.md` com correções mínimas (substituído no passo 5) |
| 5 | `CLAUDE.md` raiz com as 14 seções do Blueprint §7.1 (TECH STACK = TypeScript/Node) e os 8 `CLAUDE.md` por domínio | Claude (ENG-00) | [ ] |
| 6 | Notas pinadas do Maestri: 6 notas do [[DEC-0014-notas-pinadas-no-maestri]] | Madu + Claude | [ ] |
| 7 | Maestri: workspace, terminais ENG-00…07, conexões | **Madu** (interface do Maestri) | [ ] |
| 8 | Docker de desenvolvimento: compose com **versões fixas** (inclui o pin do `n8n`), `.env.example`, `bootstrap.ps1`; `docker compose up` com healthchecks verdes | Claude (ENG-07) | [ ] |
| 9 | Esqueleto de CI (GitHub Actions, sem lógica de negócio) | Claude (ENG-07) | [ ] |
| 10 | Documentação de arquitetura **referenciada** no repo, não copiada (fonte única, DEC-0013 D4) | Claude (ENG-00) | [ ] |
| 11 | Verificação final contra [[definicao-de-pronto]] e fechamento do M00 | Madu aprova | [ ] |

## Pendências do M00
- Caminho local (`C:\dev\launch-os`) e nome do repositório: confirmar.
- Maestri instalado? Verificar com a Madu.
- Versão fixa do `n8n`: definir ao montar o compose.
- Onde ficam as notas do Maestri (repo ou Maestri), [[DEC-0014-notas-pinadas-no-maestri]].
- **Nova:** relação entre as decisões do vault (`DEC-00XX`) e os ADRs do repo (`docs/adr/`). Duplicar cria duas fontes. Proposta a decidir: o vault guarda as decisões do projeto; `docs/adr/` guarda só ADRs técnicos do código, referenciando o vault quando preciso.
- Bloco B de [[perguntas-em-aberto]] (material faltando), não bloqueia o M00.

## Observações do passo 4
- O compose agora fica em `infrastructure/compose/`; o `.env` fica na raiz. Por isso o comando exige `--env-file .env` (documentado no `README` do repo). `docker compose config` valida a sintaxe, mas **o `up` só será testado no passo 8**, com o Docker rodando.
- `N8N_BASIC_AUTH_ACTIVE` foi mantida como veio do starter; verificar no passo 8 se a versão fixada do n8n ainda usa essa variável.
- `docs/DEFINITION_OF_DONE.md` do repo é cópia operacional da nota `definicao-de-pronto` do vault; em caso de divergência vale o vault até decisão contrária.

## Riscos conhecidos
- Docker parado: o passo 8 não passa sem o daemon.
- Com `core.autocrlf=true` no sistema, sem `.gitattributes` os scripts e o JSON do n8n podem quebrar por CRLF (mitigado no passo 2; o repo usa `core.autocrlf=false` local).
- O `bootstrap.ps1` do starter tem texto com acento e pode ler errado no Windows PowerShell 5.1 se estiver sem BOM: verificar no passo 8.
- Pouco espaço em disco: acompanhar o tamanho das imagens e volumes Docker.
- Criar o repositório no GitHub e fazer push são ações externas: só com confirmação.
- O `docker-compose.yml` do starter usa `n8nio/n8n:latest` e `CHANGE_ME` como senhas padrão: não usar assim fora do ambiente local.

Ver [[milestones]] · [[blueprint-de-execucao]] · [[roadmap]].
