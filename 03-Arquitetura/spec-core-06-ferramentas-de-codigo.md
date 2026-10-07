---
tipo: arquitetura
area: 03-Arquitetura
fonte: DEC-0013; DEC-0016 D1, D5; DEC-0019; DEC-0020; M01-plano-de-execucao (F1, F6); blocos 1 a 5 desta especificação; consulta ao registro do npm e ao nodejs.org em 2026-10-07
status: decisões D6.1 a D6.10 aprovadas pela Madu em 2026-10-07 (recomendações do ENG-00); versões exatas fixadas na F6; ADRs ainda não gravados no launch-os (M01, F1, bloco 6 de 6)
atualizado: 2026-10-07
---
# Especificação do Core, bloco 6: ferramentas de código (ADRs)

Último bloco da F1 do [[M01-plano-de-execucao]] ([[DEC-0022-inicio-formal-do-m01]]). **Proposta, sem código e sem instalar nada.** O plano diz que "o ENG-00 propõe e a Madu aprova em ADR; **nada é fixado antes**", e que a lista inicial (Fastify, migrations SQL up/down, Zod, Vitest, pnpm) é só intenção. Aqui cada escolha vem com **evidência medida em 2026-10-07** (versões e compatibilidades consultadas no registro do npm e no nodejs.org, e o ambiente desta máquina) e com as alternativas. Marcas: **PROP** é proposta; **PEND** é o que não pode ser decidido agora.

Blocos: 1 serviços e limites · 2 entidades e schema · 3 state machine e gates · 4 eventos · 5 permissões e aprovações · **6 ferramentas de código** (este). Blocos 1 a 5 com decisões aprovadas em 2026-10-07.

## 1. O que já está fixado
| Fato | Fonte |
|---|---|
| Backend e frontend em **TypeScript sobre Node**, com tipos compartilhados em `packages/schemas/` | [[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] D1 |
| Framework de backend, migrations, validação, testes, Node e gerenciador de pacotes **não estavam fixados** | [[M01-plano-de-execucao]] |
| PostgreSQL 17.11, Redis 8.10.2, n8n 2.41.6, imagens fixas; portas só em `127.0.0.1`; versões só mudam por decisão registrada | [[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]] |
| Estrutura `launch-os`: `apps/api`, `packages/schemas` (contratos, fonte única), `packages/events`, `database/migrations`, `database/schema` (DDL), `docs/adr` (ADRs), `tests/` | [[estrutura-do-repositorio]] (DEC-0013) |
| Desenvolvimento em Windows nativo + Docker Desktop, com LF fixado | DEC-0016 D5 |
| Repositório de código: `madubrandao/launch-os`, privado | [[M00-plano-de-execucao]] |
| API fina; sem ORM imposto; entrega por outbox no PostgreSQL; agendador na API; Redis fora do caminho crítico | blocos 1 e 4 (aprovados) |
| Guardrails: migration reversível, sem secret em texto puro, evento versionável | DEC-0010 §8 |

## 2. Ambiente medido nesta máquina em 2026-10-07
| Item | Medido |
|---|---|
| Node | v24.16.0 instalado |
| pnpm | 11.5.1 instalado |
| npm | 11.13.0 |
| Docker | 29.5.2 |
| Git | 2.54.0 |
| Disco `C:` | **22,2 GB livres** (o plano exige ≥ 20 GB: margem pequena) |
| Node no nodejs.org | **24.x é LTS "Krypton"**; última 24.21.0 (2026-09-07). A linha 26.x (26.11.0) ainda **não** é LTS |
| pnpm no registro | última 11.x consultada: 11.28.5 |

## 3. ADRs propostos
Cada ADR vira um arquivo em `docs/adr/` do `launch-os` **depois da aprovação** (ver seção 6). As versões abaixo são as do registro em 2026-10-07; a versão **exata** de cada pacote é fixada pelo arquivo de bloqueio (lockfile) na F6 e só muda por decisão registrada.

### ADR-1 Node.js: 24 LTS
- **Evidência:** 24.x é LTS (nodejs.org); 26.x não é. Todas as bibliotecas escolhidas abaixo aceitam Node 24 (Vitest 5: `^22.12 || ^24 || >=26`; ESLint 10: `^20.19 || ^22.13 || >=24`; node-pg-migrate 9: `>=20.11`).
- **Proposta:** fixar a linha **24 LTS** com versão exata (arquivo `.node-version`, campo `engines` e imagem de container quando houver). Na máquina há 24.16.0 e a LTS atual é 24.21.0: na F6 fixamos o patch e **a Madu decide se atualiza o Node local** (ação na máquina dela).
- **Alternativas rejeitadas:** Node 26 (não é LTS); Node 22 (LTS mais antiga, sem ganho).

### ADR-2 Gerenciador de pacotes: pnpm com workspaces
- **Evidência:** o repositório é um monorepo (`apps/`, `packages/`); pnpm 11.5.1 já está instalado; npm 11.13.0 também tem workspaces.
- **Proposta:** **pnpm workspaces**, versão exata fixada no campo `packageManager` do `package.json` (instalado: 11.5.1; no registro a última 11.x é a 11.28.5 e o `latest` já é a **12.9.1**). **PEND:** a F6 escolhe a major (11 ou 12) e a versão exata, depois de verificar a compatibilidade com Node 24 e o lockfile.
- **Alternativa rejeitada:** npm workspaces (menos controle de dependências entre pacotes do monorepo e instalação mais lenta). É a opção de menor mudança se a Madu preferir; custo é aceitável.

### ADR-3 Linguagem: TypeScript 6.0 (não 7.0)
- **Evidência (achado importante):** a última versão do TypeScript no registro é a **7.0.2**, mas o `typescript-eslint` 8.71.1 declara compatibilidade com `typescript >=4.8.4 <6.1.0`. **TypeScript 7 não funciona com o lint**.
- **Proposta:** **TypeScript 6.0.x** (última: 6.0.3), modo estrito, fixado. Reavaliar o 7 quando o `typescript-eslint` suportar, por nova decisão.
- **Alternativas rejeitadas:** TS 7.0.2 (sem lint compatível); TS 5.9 (linha anterior, sem ganho).

### ADR-4 Framework da API: Fastify 5
- **Evidência:** Fastify 5.12.5, atualizado em 2026-10-04; ecossistema oficial (`@fastify/jwt` 10.2.2, atualizado em 2026-10-04); logger Pino integrado; ganchos por ciclo de vida (úteis para `correlation_id`, idempotência e audit).
- **Proposta:** **Fastify 5.x**, API em TypeScript, plugins por domínio, validação por esquema na entrada.
- **Alternativas rejeitadas:** Express 5.2.1 (menos recursos nativos de validação e de ciclo de vida, o que empurra para bibliotecas extras); Hono 4.13 (voltado a edge e ambientes leves, sem ganho aqui).

### ADR-5 Validação e contratos: Zod 4 em `packages/schemas/`
- **Evidência:** Zod 4.6.5 (atualizado em 2026-10-02). O adaptador `fastify-type-provider-zod` 7.0.0 exige `fastify ^5.5`, `zod >=4.1.5`, `@fastify/swagger >=9.5.1` e `openapi-types ^12.1.3` (último ajuste do adaptador em 2026-06, mais antigo que o resto).
- **Proposta:** **Zod 4** como **fonte única** dos contratos (API, eventos, permissões) em `packages/schemas/`, de onde saem os tipos TypeScript. O adaptador é **opcional**: se ficar atrás do Fastify ou do Zod, a API valida com `parse` explícito no início do handler, sem perder o contrato. Gerar a documentação OpenAPI é **PEND** (F4).
- **Alternativa rejeitada:** TypeBox 1.3 (gera JSON Schema nativo, mas o time e o Blueprint já pensam em Zod, e a regra de versionamento de evento precisa de validações programáveis).

### ADR-6 Acesso ao PostgreSQL: `pg`, SQL explícito, sem ORM
- **Evidência:** `pg` 8.23.1 (atualizado em 2026-09-30). O outbox e a imutabilidade (blocos 2 e 4) dependem de transações, `SELECT ... FOR UPDATE SKIP LOCKED` e triggers, que um ORM esconde. A DDL é fonte única em `database/schema/` (DEC-0013).
- **Proposta:** **`pg`** com consultas SQL parametrizadas, repositórios finos por domínio, transações explícitas. Sem ORM e sem construtor de consultas no M01.
- **Alternativas rejeitadas:** Drizzle 0.45.3 e Kysely 0.29.6 (o schema já é especificado em SQL; as duas camadas extras competem com `database/schema/` como fonte da verdade); `postgres` 3.4.9 (menos adotado; sem ganho decisivo).

### ADR-7 Migrations: `node-pg-migrate` com arquivos SQL up/down
- **Evidência:** `node-pg-migrate` 9.0.0 (atualizado em 2026-10-05). Conferi o código do pacote: ele carrega migrations **em SQL com arquivos pareados `.up.sql` e `.down.sql`** e recusa pares duplicados ou misturados.
- **Proposta:** migrations em **SQL puro, um par up/down por mudança**, em `database/migrations/`; nenhuma migração destrutiva sem aprovação (DEC-0010 §8). A CI roda o **round-trip** (subir tudo, descer tudo, subir de novo) e compara o resultado com o snapshot em `database/schema/` (detalhe do snapshot: PEND, F2). **Atenção (revisão independente):** o carregador de SQL do pacote fica em `dist/legacy/`, o pacote exige `pg` abaixo da versão 9, e um arquivo SQL único sem o par `.down.sql` vira migration **irreversível**; por isso a CI deve falhar se existir `.up.sql` sem `.down.sql`.
- **Alternativas rejeitadas:** executor próprio de SQL (mais código para manter e testar); ferramentas fora do ecossistema Node, como dbmate (mais um binário na máquina e na CI). Não avaliei dbmate no registro do npm por não ser pacote Node; se a Madu quiser comparar, vira nova decisão.

### ADR-8 Testes: Vitest 5, banco real
- **Evidência:** Vitest 5.0.3 (atualizado em 2026-09-30), aceita Node 24 e `@types/node >=24`. `testcontainers` 12.2.0 exige `node >= 22.22` e Docker por teste.
- **Proposta:** **Vitest** para unitários, contrato e integração, com cobertura (`@vitest/coverage-v8`, mesma versão). Integração contra **PostgreSQL real**: o do compose na máquina (banco `launch_os_test` criado e descartado por execução) e **containers de serviço** na CI. Sem testcontainers no M01: o disco tem 22,2 GB livres e a RAM é curta; subir containers por teste piora os dois. Injeção de falha (F7) por pontos de falha controlados no código de teste.
- **Alternativas rejeitadas:** Jest (mais configuração para TypeScript e ESM); testcontainers (peso na máquina).

### ADR-9 Qualidade de código: ESLint 10 + typescript-eslint + Prettier
- **Evidência:** ESLint 10.12.0 aceita Node 24; `typescript-eslint` 8.71.1 aceita ESLint 10 e TypeScript <6.1; Prettier 3.9.9.
- **Proposta:** ESLint (configuração plana) com `typescript-eslint` e Prettier para formatação; `.gitattributes` com LF (D5 do DEC-0016); verificação de tipos como etapa própria da CI.
- **Alternativa rejeitada:** Biome (menos aderente à regra de tipos do `typescript-eslint` que o plano de testes usa).

### ADR-10 Logs: Pino
- **Evidência:** Pino 10.4.0 (atualizado em 2026-10-02), é o logger nativo do Fastify.
- **Proposta:** **Pino** com logs estruturados (`correlation_id`, `execution_id`), **redação obrigatória** de campos secretos (senha, token, chave de agente, cabeçalho de autorização). Reforça o guardrail "nenhum secret em log" do bloco 5.

### ADR-11 Segurança: argon2, JWT e credenciais de agente
- **Senha da Madu:** **`argon2`** 0.45.1. O pacote traz binários pré-compilados (verifiquei os `prebuilds/` no pacote), então não exige compilador no Windows. **A instalação no Windows e na CI é validada na F2**; se falhar, a alternativa é `@node-rs/argon2` 2.2.2 (também com binários por plataforma).
- **Token de sessão:** **`@fastify/jwt`** 10.2.2, algoritmo **fixado em HS256** (sem aceitar outro), vida curta, assinado com o `JWT_SECRET` (bloco 5, D5.1). Alternativa: `jose` 6.2.12 (sem integração com Fastify).
- **Credencial de agente:** chave aleatória (tamanho de 32 bytes: **PROP**) gerada com o módulo nativo `node:crypto`; no banco, só o **hash SHA-256** (a chave já tem entropia alta, então não precisa de hash lento); comparação em tempo constante. Sem biblioteca extra (bloco 5, D5.2).

### ADR-12 Agendador e entregador: dentro da API, sem biblioteca de fila
- **Evidência:** o bloco 4 aprovou entrega por outbox no PostgreSQL e o bloco 1 aprovou o agendador dentro da API.
- **Proposta:** laços temporizadores no processo da API, que pegam trabalho com `SELECT ... FOR UPDATE SKIP LOCKED` (evita duas instâncias pegarem a mesma linha) **e só escolhem linha sem evento anterior pendente da mesma (assinante, entidade)** (bloco 4, seção 5; o `SKIP LOCKED` sozinho não garante a ordem). Sem biblioteca de fila e sem Redis no caminho crítico. Cobre: entrega de eventos, expiração de aprovações, limpeza.
- **Alternativa rejeitada:** fila sobre Redis (decisão D4.1: fora do caminho crítico no M01).

### ADR-13 Cliente de linha de comando da Madu (bloco 5, D5.8)
- **Proposta:** um pacote pequeno em TypeScript no monorepo (`apps/cli`, nome a confirmar), que usa o login da seção 3.1 do bloco 5 e chama os endpoints de aprovação. Também traz o comando de **bootstrap** das identidades (bloco 5, 3.5, D7.4), que fala **direto com o banco** com a credencial do operador, sem endpoint na API. Sem interface gráfica; o dashboard é do M06. Arquivo de sessão local com permissão restrita, **fora do repositório**. Biblioteca de comandos: **PEND** (escolha na F4; não é decisão de arquitetura).
- **Fora da lista de pastas do DEC-0013:** `apps/cli` não existe na estrutura. É extensão pequena e **precisa da sua aprovação** (D6.9).

## 4. O que **não** muda e o que fica PEND
- **Não muda:** PostgreSQL 17.11, Redis 8.10.2, n8n 2.41.6 (DEC-0019).
- **PEND:** CI exata (jobs e versão das Actions, F6); pacote de documentação OpenAPI (F4); biblioteca de linha de comando (F4); snapshot de `database/schema/` (F2); senha do Redis e banco e usuário do n8n (F2 e F6, bloco 5); hospedagem de teste e produção (antes do M03); modelo de LLM e adaptador (M02).

## 5. Riscos
- **Pacotes novos e de uma pessoa só:** várias bibliotecas foram atualizadas esta semana. O lockfile e a regra "versões só mudam por decisão" evitam surpresas; cada atualização passa por CI.
- **Cadeia de suprimentos:** instalar pacotes executa código de terceiros. Mitigação: poucas dependências, lockfile com integridade, `pnpm audit` na CI (F6), sem pacote desnecessário. Nada é instalado antes da sua aprovação.
- **TypeScript 7 e ESLint:** o TS 7 é novo e ainda não tem lint compatível; se a Madu quiser o 7, o preço é abrir mão do `typescript-eslint` hoje.
- **Disco:** 22,2 GB livres (o plano exige ≥ 20 GB). `node_modules`, imagens e volumes consomem; acompanhar na F2 e na F6.
- **Windows:** binários nativos (argon2) e quebras de linha; coberto pela validação na F2 e pelo `.gitattributes`.
- **Fastify e Zod:** o adaptador de tipos pode atrasar; o plano de contingência é o `parse` explícito (ADR-5).

## 6. Fechamento da F1 (depois da aprovação do bloco 6)
Segundo o plano do M01, a F1 só encerra com **revisão independente da especificação**. Proposta de sequência:
1. Registrar as decisões D6.x aprovadas.
2. **Revisão independente da especificação inteira** (blocos 1 a 6), por **outro agente** com escopo completo, como no M00. Os achados e as correções ficam registrados.
3. **DEC de aprovação do portão F1** (próximo número: DEC-0023), com a Madu aprovando a especificação revista. Só então há permissão para criar código, schema e migrations.
4. Os ADRs do bloco 6 viram arquivos em `docs/adr/` no `launch-os` (repositório separado: **ação externa, confirmo com você antes**), e o `docs/status/` do `launch-os` é atualizado.
5. Início da F2 (banco e schema) e da F6 (infra e CI) em paralelo, em `feat/*`.

## 7. Decisões (D6.1 a D6.10 aprovadas pela Madu em 2026-10-07: vale a recomendação do ENG-00 em todas)
| # | Pergunta | Opções | Recomendação do ENG-00 |
|---|---|---|---|
| **D6.1** | Versão do Node | **A)** 24 LTS, patch exato fixado na F6. **B)** Node 26. **C)** Node 22 | **A.** É a LTS vigente; 26 não é LTS. Você decide na F6 se atualiza o 24.16.0 local para o patch fixado |
| **D6.2** | Gerenciador de pacotes | **A)** pnpm workspaces, versão exata no `packageManager`. **B)** npm workspaces | **A.** Monorepo com vários pacotes e pnpm já instalado. B só se você preferir uma ferramenta a menos |
| **D6.3** | TypeScript | **A)** 6.0.x, estrito, fixado. **B)** 7.0.2 | **A.** O TS 7 não funciona com o `typescript-eslint` hoje (suporta `<6.1.0`). Reavaliar quando houver suporte |
| **D6.4** | Framework da API | **A)** Fastify 5. **B)** Express 5. **C)** Hono | **A.** Validação, ciclo de vida e logs nativos; fica coerente com os ganchos de `correlation_id`, idempotência e audit |
| **D6.5** | Contratos e validação | **A)** Zod 4 em `packages/schemas/`, adaptador opcional com `parse` explícito de contingência. **B)** TypeBox | **A.** Fonte única de contratos, com validações programáveis para o versionamento de evento |
| **D6.6** | Acesso ao banco | **A)** `pg` com SQL explícito, sem ORM. **B)** Drizzle ou Kysely | **A.** O schema é especificado em SQL; outbox, `SKIP LOCKED` e triggers ficam visíveis e testáveis |
| **D6.7** | Migrations | **A)** `node-pg-migrate` com `.up.sql` e `.down.sql` pareados e round-trip na CI. **B)** executor próprio | **A.** Confirmei no pacote o suporte a SQL com up/down; evita código próprio para algo crítico |
| **D6.8** | Testes, qualidade, logs, segurança e agendador (ADR-8 a ADR-12) | **A)** Vitest 5 com banco real e sem testcontainers; ESLint 10 + typescript-eslint + Prettier; Pino com redação de segredos; argon2, `@fastify/jwt` fixo em HS256, credencial de agente com `node:crypto`; agendador na API com `SKIP LOCKED`. **B)** variar algum item | **A.** Cada item tem a evidência e a alternativa na seção 3. Se algum não agradar, diga qual e eu ajusto só esse |
| **D6.9** | Cliente de linha de comando (`apps/cli`) | **A)** criar `apps/cli`, fora da lista do DEC-0013, com a biblioteca de comandos na F4. **B)** só `curl` e scripts | **A.** Sem dashboard até o M06, é o canal humano de aprovação (D5.8). B funciona, mas é mais frágil para aprovar com segurança |
| **D6.10** | Fechamento da F1 | **A)** a sequência da seção 6 (revisão independente por outro agente, depois DEC-0023 de aprovação do portão). **B)** aprovar o portão direto, sem revisão independente | **A.** O plano do M01 exige a revisão e o M00 só fechou depois dela |

Ver [[estrutura-do-repositorio]] · [[M01-plano-de-execucao]] · [[DEC-0019-versoes-fixas-do-ambiente-de-desenvolvimento]] · [[spec-core-04-eventos]] · [[spec-core-05-permissoes-e-aprovacoes]].
