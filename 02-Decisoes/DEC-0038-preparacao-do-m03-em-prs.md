---
tipo: decisao
area: 02-Decisoes
id: DEC-0038
status: PROPOSTA NÃO APROVADA (rascunho de 2026-10-16 para revisão da Madu; não vale como decisão até o aceite expresso)
fonte: DEC-0034 (§5, cabeçalho); DEC-0037 (§6, §8); docs/status/CURRENT_TASK.md do launch-os; 04-Fases/pre-M03-fase-E-plano.md; 03-Arquitetura/proposta-m03-source-integridade-e-aprovacoes.md
data-registro: 2026-10-16
data-decisao: (em branco: aguarda aprovação)
emenda: DEC-0034 (§5, fase E), DEC-0037 (§6), sem revogar nenhuma
---
# DEC-0038: preparação do M03 em branches e PRs (PROPOSTA NÃO APROVADA)

> **PROPOSTA NÃO APROVADA.** Este texto não altera código, banco, configuração, workflows, Railway nem recursos externos, e **não autoriza o M03**. Só passa a valer depois de aceite expresso da Madu, registrado na linha de aprovação ao final.

## 1. Contexto
- O DEC-0034 §5 diz: "Sem chamada real a LLM antes da fase E" e exige, para cada fase, "branch própria, `check-repo.sh` depois do `git add`, CI verde e revisão independente registrada no PR antes do merge". O cabeçalho diz que o M03 "continua **não autorizado** até o início formal (fase E)".
- O plano da fase E prevê que o Claude prepare **propostas** ("nada implementado"). O `CURRENT_TASK` exige "novo início formal dela, com plano único e as mesmas paradas". O DEC-0037 §6 trata os requisitos como "necessário, não suficiente" e não autoriza o M03.
- **Lacuna:** as decisões não dizem se é permitido preparar **código e testes** do M03 antes do início formal. Esta proposta fecha a lacuna sem presumir permissão.

## 2. Decisão proposta
Permitir, **depois do aceite desta DEC**, a preparação de PRs de código e testes do M03 com **dados fictícios**, sem liberar a operação do M03:
- **PR1 (funcional):** contratos promovidos, definições reais dos agentes, ferramenta de submissão para aprovação e validação preventiva de integridade (seção 5).
- **PR2 (n8n):** somente os workflows de A0, A1 e A3.
- Cada PR em branch própria, com `check-repo.sh`, CI verde e revisão independente registrada. **Cada PR depende de autorização separada da Madu, inclusive o merge.**
- Definições com `model` e `limits` ainda não aceitos pela Madu entram **marcadas como proposta** e não são mescladas antes do aceite.

## 3. Proibido expressamente antes da liberação formal do M03
1. Chamadas a LLM real (a chave Anthropic, o `llm_cost_limit` de US$ 20, o `llm_pricing` e o modelo continuam **não aprovados**).
2. Uso de credenciais operacionais.
3. Execução dos agentes em ambiente real.
4. Qualquer modificação do banco de desenvolvimento (migrations incluídas no banco real, seeds, permissões por CLI).
5. Importação operacional de workflows n8n (os arquivos podem existir no repositório; importá-los em um n8n em execução não).
6. Publicação ou envio de conteúdo.
7. Merge automático.
Também seguem proibidos, sem autorização própria: backup, restore, E5/E6, Railway, R2, instalação de skills.

## 4. O que permanece, sem alteração
- Todos os requisitos de segurança do DEC-0037 §6 para o M03 local (E1 a E6, fumaças M01 e M02 verdes, guarda das chaves confirmada pela Madu) e a regra "necessário, não suficiente".
- As restrições do DEC-0034 sobre LLM real, valores e gates humanos.
- A decisão sobre **onde** o M03 roda (local ou staging) continua para a autorização formal; o piloto inicial indicado pela Madu é local controlado.

## 5. Mudança prevista no núcleo (única, com ADR)
A preparação pode alterar **um** ponto do núcleo do gateway: a **validação obrigatória de integridade antes da submissão para aprovação**, imposta em `apps/api/src/gateway/step.ts` imediatamente antes de criar a aprovação, apenas para a ferramenta de submissão, com reverificação no executor da ferramenta. Detalhe em [[proposta-m03-source-integridade-e-aprovacoes]]. A mudança exige **ADR própria** que a justifique e testes de integração na CI. Qualquer outra alteração em `apps/api/src/gateway/*` ou `packages/agents` exige nova autorização.

## 6. Vigência
Vale do aceite até o início formal do M03 (quando o plano único do M03 a substitui) ou até revogação pela Madu. PRs não mesclados continuam sujeitos a estas regras.

## 7. Riscos
| Risco | Mitigação |
|---|---|
| Código mesclado antes do M03 ser lido como início do M03 | Esta DEC diz que não é; nada é ligado sem permissões no banco, LLM configurado e início formal |
| Valores de teste tomados por decisão | `model`, `limits` e preços marcados como proposta; merge só após aceite |
| Deriva de escopo | Dois PRs definidos; cada um com autorização separada |
| Validação no gateway virar mudança ampla do núcleo | Limitada a um ponto, com ADR e teste |
| Os requisitos da DEC-0037 serem vistos como dispensados | Seção 4: preservados integralmente |

## 8. Aprovação
Aprovada por: ______________________  em: ____/____/______   (**em branco: proposta não aprovada**)

Ver [[DEC-0034-perfil-de-valores-e-plano-ate-o-m03]] · [[DEC-0037-sequencia-d4-d3-backup-e-retencao]] · [[pre-M03-fase-E-plano]] · [[e1-inventario-de-dados-persistidos]]
