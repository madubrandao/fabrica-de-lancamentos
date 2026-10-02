---
tipo: fase
area: 04-Fases
fonte: blueprint v1 §32; starter docs/DEFINITION_OF_DONE.md
status: definido-nas-fontes
atualizado: 2026-10-02
---
# Definição de pronto (Definition of Done)

Uma capability só está pronta quando (Blueprint §32):
- [ ] código implementado
- [ ] testes escritos e passando
- [ ] logs implementados
- [ ] erros tratados
- [ ] permissões verificadas
- [ ] documentação atualizada
- [ ] evento definido
- [ ] schema versionado
- [ ] workflow n8n conectado quando aplicável
- [ ] dashboard mínimo disponível quando necessário
- [ ] ADR atualizado se houve decisão arquitetural
- [ ] revisão por outro agente concluída

Reforços do starter: contratos de entrada/saída explícitos; revisão de impacto de segurança; nenhum segredo ou estado local commitado; implicações de deploy entendidas; responsável reporta evidência de verificação.

**Protocolo de entrega por tarefa (starter CLAUDE.md):** inspecionar código/docs antes de editar → declarar suposições → menor mudança coerente → testes direcionados e depois a suíte relevante → atualizar docs se arquitetura/contratos mudarem → reportar arquivos alterados, testes, riscos e próxima dependência → **nunca declarar concluído sem verificação**.
