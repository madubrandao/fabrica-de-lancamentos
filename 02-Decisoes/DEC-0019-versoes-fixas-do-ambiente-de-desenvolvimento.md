---
tipo: decisao
area: 02-Decisoes
id: DEC-0019
status: aceita
fonte: decisão da Madu em 2026-10-04 (passo 8 do M00, "siga com 8"); validação em Docker em 2026-10-04; Docker Hub; DEC-0013 D5; DEC-0016 D3
data-registro: 2026-10-04
---
# DEC-0019 — Versões fixas e portas do ambiente de desenvolvimento

## Contexto
O compose do starter usava `postgres:17`, `redis:8` e `n8nio/n8n:latest`, sem versão exata. A regra de produção do starter pede artefatos fixados, e o DEC-0013 D5 deixou o pin do `n8n` PENDENTE para o M00. Durante a validação apareceram dois achados: a porta 5678 já estava em uso por outro projeto da máquina (e a 5679 também), e a variável `N8N_BASIC_AUTH_ACTIVE` não existe mais no n8n 2.x.

## Decisão
1. **Imagens fixas** em `infrastructure/compose/docker-compose.dev.yml`:
   - `postgres:17.11`
   - `redis:8.10.2`
   - `n8nio/n8n:2.41.6` (a tag `latest` apontava para a 2.41.6 em 2026-10-02; as 2.42.x são outro canal)
2. **Porta do n8n no host: 5680** (padrão do compose e do `.env.example`). Postgres 5432 e Redis 6379 mantidos.
3. **`N8N_BASIC_AUTH_ACTIVE` removida** do compose e do `.env.example`: a variável não aparece no n8n 2.41.6 e era ignorada.
4. **Healthcheck do n8n** adicionado (`wget` em `/healthz`); os três serviços têm healthcheck.
5. **`.env` local com segredos aleatórios**, nunca versionado; o `.env.example` mantém `CHANGE_ME`.
6. As versões só mudam por decisão registrada (ADR em `docs/adr/` ou nova DEC).

## Validação (2026-10-04)
- `docker compose up -d`: postgres, redis e n8n `healthy`.
- Versões conferidas dentro dos containers: PostgreSQL 17.11, Redis 8.10.2, n8n 2.41.6; `PONG` no Redis; `/healthz` do n8n com HTTP 200.
- Os 4 containers de outros projetos da máquina continuaram intactos.
- `scripts/bootstrap.ps1` executa no PowerShell 7 e no Windows PowerShell 5.1 e agora falha com mensagem clara se o daemon do Docker estiver parado.

## Alternativas rejeitadas
- Manter `latest` ou tags de major (`17`, `8`): o resultado muda sem aviso.
- Usar a porta 5678: conflita com outro projeto da máquina.
- Fixar por digest: mais rígido do que a fase pede; reavaliar para produção.

## Consequências e PENDENTES
- O n8n marca o modo de task runner interno como obsoleto (`N8N_RUNNERS_MODE`): avaliar o runner externo quando houver workflows com código.
- O disco `C:` ficou com cerca de 12 GB livres (95% usado) depois de baixar as imagens: acompanhar.
- Atualização de versões, estratégia de backup dos volumes e ambientes de teste e produção seguem PENDENTES (DEC-0016 D3).

Ver [[M00-plano-de-execucao]] · [[DEC-0013-estrutura-do-repositorio-e-fonte-unica]] · [[DEC-0005-stack-de-producao]].
