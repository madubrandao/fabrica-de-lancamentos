---
tipo: decisao
area: 02-Decisoes
id: DEC-0016
status: aceita
fonte: decisão da Madu em 2026-10-04; DEC-0005; blueprint v1 §7.1, §18; starter (.env.example, docker-compose.yml, bootstrap.ps1, MAESTRI_SETUP); verificação do ambiente em 2026-10-04
data-registro: 2026-10-04
resolve: D1, D2, D3, D4, D5, D6, D7, D8 (bloco D de perguntas-em-aberto)
---
# DEC-0016 — Stack, ambiente de desenvolvimento e adiamentos técnicos

## Contexto
Os documentos não definiam linguagem, provedor de LLM, hospedagem, vector store, ambiente Windows, repositório, mapeamento estado→gate nem limites numéricos. O `CLAUDE.md` raiz exige a seção TECH STACK (Blueprint §7.1), então D1, D5 e D6 travavam o M00.

Ambiente verificado em 2026-10-04: Windows 11; Node v24.16, Git 2.54, Docker 29.5, Python 3.14 e Claude Code instalados; WSL só com a distro `docker-desktop` (parada), sem Ubuntu. O vault tem `origin` em `github.com/madubrandao/fabrica-de-lancamentos` (é o repositório da memória, não o do código).

## Decisões (todas as recomendadas aprovadas pela Madu)

### D1 — Linguagem e framework: TypeScript/Node
Backend e frontend em TypeScript sobre Node, com tipos compartilhados via `packages/schemas/` ([[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] D4).
- **PENDENTE:** framework de backend e de frontend. O ENG-00 propõe e a Madu aprova em ADR no M01.

### D2 — LLM: Anthropic primeiro, com camada de adaptador
Os agentes falam com uma interface de provedor (regra "adaptadores de infraestrutura atrás de interfaces", starter `architecture.md`); Anthropic é o primeiro provedor. A chave da OpenAI do `.env.example` fica fora até haver decisão nova.
- **PENDENTE:** modelo específico por agente (M02).

### D3 — Hospedagem: só dev agora
Desenvolvimento em Docker local. Teste e produção ficam **PENDENTES**, a decidir antes do M03, quando o primeiro ciclo simulado exigir ambiente de teste.

### D4 — Vector store: adiado
Nenhum vector store no M00. O M02 reserva só a interface de memória semântica. Decidir quando houver caso de uso concreto ([[DEC-0009-memoria-empresarial-em-tres-tipos]]). Registro: a imagem `postgres:17` do starter não traz pgvector; adotá-lo exigiria trocar a imagem.

### D5 — Ambiente de desenvolvimento: Windows nativo + Docker Desktop
PowerShell/Git Bash no Windows, com Docker Desktop para os serviços; compatível com o `bootstrap.ps1` do starter. WSL e tmux ficam fora por ora.
- **No M00:** criar `.gitattributes` fixando LF (o Git já avisou "LF will be replaced by CRLF").
- Reavaliar WSL2/Ubuntu se surgirem problemas de caminho ou quebra de linha.

### D6 — Repositório `launch-os`: novo, privado, fora de `.claude`
Repositório privado `launch-os` no GitHub, em pasta fora de `C:\Users\mmadu\.claude\projects` e **sem espaços no caminho** (por exemplo `C:\dev\launch-os`). Separado do vault. Privado porque `knowledge/plf/` deriva de material com direitos autorais (DEC-0010 §2 mantido). Criação só depois do **início formal do M00** (DEC-0010 §10).
- **PENDENTE:** caminho local e nome exato da organização/conta no GitHub, a confirmar na criação.

### D7 — Mapeamento estado → gate: adiado
Especificação do M01 pelo ENG-00, junto com "quem aprova" e os critérios de cada gate ([[gates]]). Registro: 15 estados e 12 gates; os gates G04 a G07 caem todos entre `PLC` e `CART_OPEN`, então a ligação não é automática.

### D8 — Limites numéricos: adiados, modelados como configuração
Orçamento, tempo de aprovação e SLA de agentes ficam **PENDENTES** para o M01, modelados como **configuração por política**, não como valor fixo no código. O limite de orçamento (C4) precisa estar definido antes do primeiro gasto do A5, no M04.
- **Regra de segurança:** aprovação sem resposta vale como **não aprovada** (fail-closed). Detalhes de `approval.expired` seguem PENDENTES ([[DEC-0011-eventos-de-aprovacao-requested-granted-rejected]]).

## Alternativas rejeitadas
- D1: Python no backend; decidir só no M01 (bloquearia o `CLAUDE.md` raiz).
- D2: Anthropic e OpenAI desde o início; Anthropic sem camada de adaptador.
- D3: definir os três ambientes agora; nuvem gerenciada.
- D4: pgvector no M00; vector DB dedicado.
- D5: WSL2 com Ubuntu agora; tmux (depende de Linux/WSL).
- D6: código dentro do repositório do vault; só local, sem remote.
- D7/D8: decidir agora sem requisitos; valores inventados.

## Consequências
- O `CLAUDE.md` raiz do `launch-os` pode registrar TECH STACK = TypeScript/Node, LLM via adaptador (Anthropic primeiro), Postgres, Redis, n8n e Docker.
- PENDENTES com marco-alvo: frameworks (M01), modelos por agente (M02), ambientes de teste e produção (antes do M03), vector store (sob demanda), estado→gate (M01), limites numéricos (M01; orçamento antes do M04).
- Nada foi implementado: nenhum repositório de código existe.

Ver [[perguntas-em-aberto]] · [[DEC-0005-stack-de-producao]] · [[milestones]] · [[blueprint-de-execucao]].
