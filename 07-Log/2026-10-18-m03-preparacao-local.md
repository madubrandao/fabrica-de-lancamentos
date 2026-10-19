---
tipo: log
area: 07-Log
fonte: sessão de 2026-10-18; launch-os main `77386c5` (merge do PR #56, CI verde run 39142064978); [[DEC-0039-liberacao-do-primeiro-teste-real-do-m03]] (aprovada pela Madu com condições em 2026-10-18)
atualizado: 2026-10-18
---
# M03: preparação local do primeiro teste real (2026-10-18)

## Autorizado pela Madu
- **Merge do PR #56:** merge commit `77386c5`, preso ao HEAD `9054510`, com a árvore idêntica à testada. CI da `main` verde.
- **DEC-0039 aprovada com condições:**
  - um teste local, com até 3 tentativas e só dados fictícios;
  - produto "serviço de tranças da Sol", `source: fictional`;
  - A0 só com `tool.handoff.start` e `tool.content.submit_for_approval`;
  - **proibido** conceder `request_action_approval`;
  - teto de US$ 20 na sessão, sem ultrapassar limites menores;
  - `claude-sonnet-5-5` condicionado a disponibilidade, identificação exata e preço oficial no dia, sem troca automática de modelo.
- **E4:** o aceite operacional depende do backup com `n8n_data` e do restore verificado, com a `N8N_ENCRYPTION_KEY` decifrando as credenciais.
- **Anthropic:** chamada paga **não autorizada** ainda.

## Feito (DEC-0039 §5)
1. **Recursos:** 39,9 GB livres. Os volumes existentes foram preservados, e o n8n ganhou um volume novo.
2. **`.env`:**
   - o `bootstrap.ps1` acrescentou só o `EXECUTION_TOKEN_SECRET`;
   - entrou o perfil local da DEC-0034 §2 e um `N8N_WEBHOOK_SECRET` aleatório;
   - nenhum valor foi lido nem exibido; o arquivo foi de 1449 para 2571 bytes;
   - ficaram de fora `MAINTENANCE_*`, `LLM_PROVIDER` e a chave Anthropic, que está vazia.
3. **Banco do desenvolvimento:** migrations 0001 a 0014 e semente com os 11 agentes.
4. **Identidades:** bootstrap com 1 operadora, 11 credenciais de agente e 1 de serviço do n8n. A senha foi gerada e, junto com as 12 credenciais, gravada **só** em `%USERPROFILE%\launch-os-segredos\credenciais-dev-2026-10-18.txt`. A ACL do arquivo é restrita ao usuário e ao SYSTEM, sem herança. Nada foi impresso.
5. **API e D-7:** a API subiu; `ops:audit-partitions` deu saída 0, com folga de 2 meses (outubro a dezembro de 2026). Depois a API foi encerrada pelo Claude Code por falta de RAM, sem defeito.
6. **n8n do desenvolvimento:**
   - 6 workflows publicados: A9, A0, A1, A3, `execute-agent` e `tool-test-external-noop`;
   - 2 credenciais cifradas;
   - reinício confirmado;
   - os webhooks de A9, A0, A1 e A3 prontos entre 19,3 e 19,9 s, depois de 10 respostas 404 cada (o mesmo padrão da H7).

## Próximo checkpoint
1. A Madu move as credenciais para o Bitwarden e apaga o arquivo.
2. Backup com `n8n_data` (`BACKUP_INCLUDE_N8N_DATA=true`) e restore isolado verificado, com o n8n do alvo decifrando a credencial (E4-3). Só com RAM suficiente e sem outro processo pesado rodando.
3. Religar a API, fazer o login da operadora e gravar as políticas e as permissões do A0 pela CLI.
4. Procedimento da chave Anthropic; aguardar a autorização final.
