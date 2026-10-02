---
tipo: contexto
area: 01-Contexto
fonte: PLANO FINAL DE ENGENHARIA.txt (§28, §29); blueprint v1 (§0, §12, §42–44); starter zip CLAUDE.md
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# Escopo e princípios

## Livro x engenharia nossa
Esta distinção **deve permanecer explícita** em toda a documentação (Blueprint §44).

**Derivado da fonte (livro):** pré-pré-lançamento; pré-lançamento; PLC; abertura e fechamento do carrinho; pós-lançamento; Seed Launch; Lançamento Interno; Lançamento Conjunto; múltiplas ofertas; relançamento; Lançamento Perene; Circle of Awesome; importância da lista e do relacionamento; acompanhamento de clientes e não compradores.

**Engenharia nossa:** 11 agentes de produção; 8 Claude Codes de engenharia; Maestri como cockpit; PostgreSQL/Redis; event bus; schemas; permissões; runtime de agentes; dashboard; state machine formal; gates; observabilidade; estrutura de repositório; Partituras do Maestri; workspaces; branches/worktrees.

Regras: não atribuir ao livro decisões de arquitetura; não converter o livro em promessa de resultado (o próprio livro diz que PLF não é "fácil ou automática").

## O que NÃO vamos fazer (Plano Final §28)
- começar com 20+ agentes;
- agente que cria outro agente;
- IA alterando sozinha toda a estratégia;
- workflow sem estado;
- memória apenas em prompt;
- workflow A → B → C sem camada central de eventos;
- automação que esconda suas próprias decisões.

## Regra de ouro do sistema (agentes de produção)
Todo agente deve responder: missão, input, output, processo, regras, dados acessíveis, o que pode decidir, o que precisa de aprovação, como é medido, quando aciona outro agente. **Se não der para responder, o agente não está pronto para ser construído.** Aplicado em [[05-Agentes/README|05-Agentes]].

## Regra de ouro para os Claude Codes
Toda tarefa deve ter: CONTEXT, OBJECTIVE, SCOPE, INPUTS, OUTPUTS, CONSTRAINTS, DEPENDENCIES, TESTS, DEFINITION OF DONE (nunca só "construa isso").

## Agente ≠ workflow
Um agente é dono de uma responsabilidade e pode ter vários workflows. Não criar um agente por workflow.

## Primeira versão operacional
Não são "todos os workflows". Ativa-se o **caminho crítico** A0→A1→A2→A3→A4→A6→A7→A8→A10→A0; depois A5 e A9 ganham capacidade de escala (Plano Final §25).

## Hierarquia de autoridade (canônica, [[DEC-0010-decisoes-canonicas-fase-1-v1]] §1)
1. Decisões canônicas registradas em `02-Decisoes/`.
2. Blueprint Técnico v1.x aprovado.
3. Documentos de arquitetura e roadmap derivados do Blueprint.
4. Starter técnico (implementa decisões; **não redefine arquitetura**).
5. Rascunhos e propostas conceituais anteriores.

Quando fontes conflitam: parar e expor o conflito, não reconciliar em silêncio ([[pendencias-de-decisao]]). Isso substitui a "hierarquia de verdade" do CLAUDE.md do starter (decisões humanas > código > arquitetura > metodologia > suposições), que passa a valer só como regra de trabalho dentro do repo de código, depois do M00.

## Regra para itens PENDENTES (DEC-0010 §9)
Não inventar automaticamente: prompts finais dos agentes, ferramentas exatas de cada agente, escopos de dados por agente, schemas completos dos objetos, KPIs finais, regras detalhadas de autonomia, integrações externas definitivas. Serão especificados nos marcos correspondentes.

## Regra de implementação (DEC-0010 §10)
A FASE 1 - BASE é documentação e arquitetura. Nenhum código de produção sem iniciar o **M00 formalmente**.
