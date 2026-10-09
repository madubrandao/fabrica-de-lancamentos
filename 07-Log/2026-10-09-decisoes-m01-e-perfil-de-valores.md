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
- **Próximo:** fase A (saneamento) no `launch-os`.
