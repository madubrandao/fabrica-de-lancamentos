---
tipo: log
area: 07-Log
data: 2026-10-08
---
# 2026-10-08 — F5 a F7 do M02 e fechamento proposto

- **F5 (n8n):** 5a (workflows e contrato, PR #19) e 5b (fumaça no n8n real, PR #20). A revisão do 5b pediu cenários que só entraram na F6.
- **F6 (qualidade, PR #22):** `approval-resume` ignora `tool.*`; varredura de segredos do Agent Engine; failure injection do gateway; S13 a S16 na fumaça (29/29); 11 provas negativas por mutação (P18 a P28). A revisão independente do PR achou controles positivos fracos e um S13 sujeito a token expirado: corrigidos antes do merge.
- **F7 (fechamento, PR #23):** verificação do zero (clone limpo, volumes novos), duas revisões completas (conformidade e segurança/operação; 0 críticos, 0 altos). Corrigidos: `renew_used` em execução final (500), `seq` int32 (500), piso do `N8N_WEBHOOK_SECRET`, varredura × `LLM_TIMEOUT_MS`, ordem varredores/despachante. Lacunas L1 a L7 registradas no [[DEC-0032-fechamento-do-m02-agent-engine]].
- **Erros meus no caminho (corrigidos):** restaurei um arquivo com `git checkout` e perdi uma edição ainda sem stage (reaplicada; o teste novo a protege); o PR #20 disse "achados tratados" quando três cenários da revisão ainda faltavam (entraram na F6).
- **Estado:** M02 implementado; aguarda o aceite do DEC-0032 pela Madu. M03 não autorizado. Pilha isolada derrubada.
