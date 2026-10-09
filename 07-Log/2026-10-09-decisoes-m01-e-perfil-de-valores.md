---
tipo: log
area: 07-Log
data: 2026-10-09
---
# 2026-10-09 — Decisões pendentes do M01, perfil de valores e plano até o M03

- **DEC-0033 (aceita):** A4 com critério alterado (API verificada pela fumaça; imagem Docker junto da hospedagem), OpenAPI adiado, Node 24.21.0, retenção do `audit_log` por partição, notificação e segundo fator adiados, M3 aceito como está.
- **Erro meu (corrigido antes do commit):** o rascunho do DEC-0033 dizia que o M3 tinha sido fechado no M02; o `CURRENT_MILESTONE.md` lista só M1, M2 e M5.
- **DEC-0034 (aceita, "Siga com as recomendaras"):** perfil de valores do desenvolvimento local, políticas (aprovação em 72 h, orçamento 0, teto de LLM de US$ 20, um modelo), gates por aprovação humana, L7 (exceto a importação) movida para antes do M04 e plano A a F até o M03.
- **Emenda do DEC-0032:** pontos médios da revisão completa registrados (R1 a R5); o `memory.write` sem chaves é por desenho.
- **Fase A concluída:** PR #24 (documentação) e PR #25 (fumaça do M01 de volta, 49/49; teste da subida real do `server.ts`; fumaça do M02 29/29) mesclados pela Madu, `main` em `6f56da1` com CI verde. As duas revisões independentes pediram ajustes, tratados antes do merge. A CI usa `POSTGRES_HOST_AUTH_METHOD=trust`, então não prova a senha dos papéis do banco.
- **Node 24.21.0:** não instalado. A espera foi cortada duas vezes por falta de memória; o instalador oficial, verificado por SHA-256, aguarda a Madu.
- **Maestri:** a Madu confirmou que continua fora do planejamento (DEC-0020).
- **Fase B concluída:** o plano foi revisado em modo de planejamento e aprovado pela Madu ([[pre-M03-fase-B-plano]]).
  - **L3 (PR #26):**
    - gatilhos proibidos na subida;
    - evento de agente só dispara via `event.emit`;
    - `approval.granted` e `approval.rejected` só disparam com ator humano (caso do G12, achado da revisão).
  - **L6 (PR #27):** PUT de permissões recusa ferramenta desconhecida, e a subida avisa.
  - **L5 (PR #28):** `fail` do serviço confere `run_id` e época; o `renew` leva o lease junto (defeito meu, achado antes da revisão, que confirmou). Fumaça do M02: 29/29.
- **CI:** o Docker Hub bloqueou os downloads anônimos dos runners. A Madu criou o token e os segredos, e a CI passou a baixar com login (PR #29).
- **Merges:** todos autorizados pela Madu. O classificador do modo automático exige essa autorização no momento do merge.
- **Próximo:** plano único da fase C (L1 com R1, L2, L4).
