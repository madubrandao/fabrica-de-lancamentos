---
tipo: checkpoint
area: raiz
atualizado: 2026-10-18
fonte: verificação direta em 2026-10-18 (git, gh, docker, psql somente leitura); 07-Log/2026-10-18-m03-preparacao-local.md; memória do Claude Code
---
# RETOMADA: primeiro teste real do M03 (checkpoint de 2026-10-18)

> Leia isto primeiro na próxima sessão. "**Confirmado**" foi conferido em 2026-10-18 sem reiniciar nenhum serviço; "**registrado**" vem de evidência anterior.

## 1. Objetivo
O primeiro teste **real** e supervisionado A0 → A1 → A3, com Anthropic, local, dados fictícios e aprovação humana. Hoje ele está **preparado pela metade**: a chamada paga continua **não autorizada**.

## 2. Concluído, com evidência
- **launch-os** (`C:\dev\launch-os`, branch `main`, HEAD `77386c5`, igual a `origin/main`), **confirmado**:
  - PR #55 (E5/E6): MESCLADO em `6a7f397`.
  - PR #56 (Lote 3): MESCLADO em `77386c5` (HEAD da branch: `9054510`).
  - CI da `main` em `77386c5`: verde (run 39142064978).
- **Lote 3** (PR #56), **registrado:**
  - ADR-0031: a aprovação pendente vence quando a execução termina, e a concessão tardia responde 409.
  - ADR-0032: `content decide|list` e o marcador `[FICTÍCIO]`.
  - A importação do n8n espera o webhook de cada agente.
  - Evidências:
    - unitários 1208/1208; integração com 41 arquivos e 841 testes;
    - fumaças M01 49/49, M02 31/31 e fluxo A0 → A1 → A3 com n8n real 22/22 duas vezes seguidas;
    - CI 3/3 verde.
  - Revisão Codex (só leitura): 2 achados reais **não bloqueantes no local**, registrados como limites (ADR-0032 e M02-RUNBOOK §3).
  - H7: 9 medidas sem falha; os webhooks ficam prontos entre 18 e 21 s depois do restart.
- **Fases:**
  - M01 e M02: concluídos (DEC-0032).
  - E1 confirmada (§8); E2-B e E3 com o conjunto real feitas (17/10).
  - E4: guarda das chaves confirmada (17/10), mas o **aceite operacional está pendente** (ver §5).
  - E5/E6: entrega de código concluída (PR #55); a partição está aplicada no dev, com folga de 2 meses.
  - **M03: liberado só para 1 teste (DEC-0039) e ainda NÃO executado.**
- **Decisões:**
  - DEC-0037: ACEITA (14/10).
  - DEC-0038: ACEITA (17/10). As proibições dela foram substituídas, só no escopo do teste, pela DEC-0039.
  - DEC-0039: **APROVADA com condições (18/10)** e em execução (§3 e §4).
- **Vault** (`C:\Users\mmadu\.claude\projects\fabrica_de_lancamentos`): branch `docs/dec-0038-proposta-e1`, PR #3 **ABERTO e não mesclado** (o merge é da Madu). Só um item não versionado, `FASE  1 - BASE/`, que não deve ser tocado.

## 3. Infraestrutura (desenvolvimento local, notebook de 8 GB)
**Confirmado agora:**
- **Contêineres:** `launch-os-postgres-1` e `launch-os-n8n-1` rodando e saudáveis. Nenhum outro contêiner rodando.
- **Volumes:** `launch-os_postgres_data`, `launch-os_n8n_data` e `launch-os_redis_data`, todos preservados.
- **Banco `launch_os`:**
  - 14 migrations, 11 agentes, 1 usuária, 12 credenciais;
  - 3 partições do `audit_log` (outubro a dezembro de 2026);
  - **0 políticas**, **0 execuções**;
  - o A0 ainda tem as permissões **da semente**: as aprovadas ainda não foram gravadas.
- **n8n:** 6 workflows ativos (A9, A0, A1, A3, `execute-agent` e `tool-test-external-noop`).
- **API:** desligada (sem resposta em `:3000`). O Claude Code a encerrou por falta de RAM, sem defeito.
- **Máquina:** 568 MB de RAM livres e 38,4 GB livres no `C:`.

**Registrado:** o `.env` tem o perfil da DEC-0034 §2 e o `EXECUTION_TOKEN_SECRET`, **sem** `MAINTENANCE_*`, `LLM_PROVIDER` e chave Anthropic (vazia). `BACKUP_INCLUDE_N8N_DATA` ainda está `false`.

## 4. Decisões
**Aprovadas** (não reabrir):
- **DEC-0039:**
  - um teste com até 3 tentativas;
  - produto "serviço de tranças da Sol", `source: fictional`;
  - A0 só com `tool.handoff.start` e `tool.content.submit_for_approval`;
  - **proibido conceder `request_action_approval`**.
- **Custo e modelo:**
  - teto de US$ 20 na sessão, sem ultrapassar limites menores;
  - `claude-sonnet-5-5` só depois de confirmar a disponibilidade, o ID exato e o preço oficial no dia, **sem trocar de modelo automaticamente**.
- **Políticas:** `approval_expiry` 259200 s e `budget_limit` 0.
- **Proibido:** publicação, dados reais, Railway e produção, limpeza e retenção.

**Pendentes:**
- aceite operacional da E4, depois da E4-3;
- autorização final da chamada paga à Anthropic;
- merge do PR #3 do vault;
- infraestrutura em nuvem (§9).

## 5. Bloqueios reais, em ordem
1. **RAM:** com 8 GB e cerca de 0,5 GB livre, não cabe com segurança o restore isolado (uma segunda pilha) nem a API junto com o n8n.
2. **E4-3:**
   - colocar `BACKUP_INCLUDE_N8N_DATA=true`;
   - fazer o backup;
   - fazer o restore numa pilha descartável, com o n8n do alvo decifrando a credencial.

   Sem isso, a E4 não é aceita operacionalmente.
3. **API e operadora:**
   - religar a API e fazer o login da operadora (a senha está no arquivo protegido, §6);
   - gravar as 4 políticas e as permissões do A0 pela CLI.
4. **Chave Anthropic:** a Madu cria a chave e o limite no Console e a coloca no `.env` sem passar pelo chat. Depois vem a autorização final, `LLM_PROVIDER=anthropic` só durante a sessão e o `llm_pricing` do dia.

## 6. Segredos (só a localização; nunca os valores)
- **Credenciais do bootstrap:** a senha da operadora `brandaomadu28@gmail.com` e as 12 chaves `lo_`.
  - Estão em `C:\Users\mmadu\launch-os-segredos\credenciais-dev-2026-10-18.txt`.
  - A ACL é só do usuário e do SYSTEM.
  - **Pendente:** a Madu passa o conteúdo para o Bitwarden. O arquivo **só é apagado depois da confirmação dela**.
- **Outros segredos:**
  - `.env` em `C:\dev\launch-os\.env`, não versionado;
  - chave `age` no Bitwarden, com cópia offline;
  - `N8N_ENCRYPTION_KEY` numa nota segura do Bitwarden.
- **Temporários de chave antigos,** com remoção a decidir pela Madu: `C:\LaunchOS-Verif\` e o alvo da E3 no scratchpad da sessão anterior.

## 7. Próxima ação mínima
Liberar RAM fechando programas pesados que não estejam em uso. Depois, a E4-3: backup com `n8n_data` e restore isolado verificado, **um processo pesado por vez**, registrando no vault.

## 8. Retomada: comandos e caminhos
- **Conferir sem mudar nada:**
  - `cd C:\dev\launch-os; git status -sb; docker ps`
  - `docker exec launch-os-postgres-1 psql -U launch_os -d launch_os -tAc "SELECT count(*) FROM pgmigrations"`
- **Runbooks:**
  - backup e restore: `C:\dev\launch-os\docs\operations\BACKUP-RUNBOOK.md` (§7.3 n8n) e `RESTORE-RUNBOOK.md` (§6 n8n);
  - M02 e fumaças: `docs/operations/M02-RUNBOOK.md`.
- **Registros:**
  - sequência e condições: `02-Decisoes/DEC-0039-liberacao-do-primeiro-teste-real-do-m03.md`;
  - log: `07-Log/2026-10-18-m03-preparacao-local.md`.
- **Memória do Claude Code:** `C:\Users\mmadu\.claude\projects\C--Users-mmadu--claude-projects-fabrica-de-lancamentos\memory\lote3-pr56-e-dec-0039.md`.

## 9. Infraestrutura em nuvem: EM AVALIAÇÃO (nada autorizado)
- **Fatos:**
  - o notebook tem 8 GB de RAM e ficou sem memória (a API foi encerrada);
  - a Madu já tem conta **Railway Pro**, onde a Sol roda;
  - a Madu já tem um domínio na **Hostinger**;
  - os ambientes Railway do LAUNCH OS (DEC-0035/0036) existem vazios.
- **Em avaliação:** mover a infraestrutura **de desenvolvimento** do LAUNCH OS para a nuvem, de preferência na conta Railway, **isolada da Sol**.
- **Próxima decisão:** comparar o custo incremental do Railway com uma VPS barata e definir como o Claude Code executa comandos e testes remotamente.
- **Não autorizado:** migração, contratação, alteração de DNS e deploy.

## 10. Restrições e o que NÃO refazer
- **NÃO refazer:**
  - migrations, semente ou bootstrap: o bootstrap recusa uma segunda execução;
  - a importação do n8n;
  - as fumaças M01, M02 e do fluxo, salvo mudança de código;
  - a H7, salvo evidência de falha;
  - auditorias gerais;
  - PRs só de documentação.
- **NÃO:**
  - chamar a Anthropic paga sem a autorização final;
  - conceder `request_action_approval`;
  - ligar `MAINTENANCE_*`;
  - tocar Railway, produção ou a Sol;
  - apagar ou alterar volumes e backups existentes;
  - imprimir ou versionar segredos.
- **Rodar um serviço pesado por vez.** Não religar a API sem pedido.
