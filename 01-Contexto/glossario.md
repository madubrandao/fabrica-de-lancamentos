---
tipo: contexto
area: 01-Contexto
fonte: PLANO FINAL DE ENGENHARIA.txt; blueprint v1; MAPA DE EXTRAÇÃO…txt
status: rascunho-para-validacao
atualizado: 2026-10-02
---
# Glossário

Só termos que aparecem nos documentos. O PDF do livro é fonte externa e não faz parte do projeto ([[DEC-0010-decisoes-canonicas-fase-1-v1]] §2); definições metodológicas além do que está nos docs ficam PENDENTES até consulta explícita à fonte.

| Termo | Significado nos docs |
|---|---|
| PLF | Product Launch Formula / A Fórmula do Lançamento (Jeff Walker). |
| PLC | Product Launch Content: sequência de conteúdo do pré-lançamento (PLC 1, 2, 3). Lógica: oportunidade → transformação → experiência da propriedade. |
| Pré-pré-lançamento | Etapa que testa receptividade, descobre objeções, coleta linguagem e refina a oferta. Alimenta A1. |
| Seed Launch | Caminho para começar sem lista ou produto e testar a ideia antes de investir na criação. |
| Internal Launch | Lançamento interno: nosso estágio de **validação**; testa oferta e sequência antes de pedir a parceiros. |
| Joint Launch | Lançamento conjunto: usa listas de parceiros; nosso estágio de **expansão**. |
| Relaunch / Evergreen | Relançamento; lançamento perene (sequência acionada conforme novos contatos entram na lista). |
| Circle of Awesome | Ciclo ideia → Seed → Internal → Joint → novas ideias → novo Seed. |
| Cart open / closing / closed | Estados de abertura, fechamento iminente e fechamento do carrinho. |
| Launch Operating System (LAUNCH OS) | O produto que construímos: conhecimento → políticas → agentes → eventos → workflows → state machine → dados → analytics → aprendizado → próxima decisão. |
| Agente (A0–A10) | Agente de **negócio** da empresa em produção. |
| ENG-00…07 | Claude Codes de **engenharia** (camada de desenvolvimento). Não confundir com A0–A10. |
| Workflow | Processo implementado (ex.: no n8n) e pertencente a um agente. |
| Event bus | Camada central de eventos pela qual workflows/agentes se comunicam. |
| Gate | Regra do sistema: o lançamento não muda de estado sem cumprir o gate. |
| HITL | Human-in-the-loop: níveis AUTO, SUPERVISED, HUMAN DECISION. |
| Company Memory | Memória empresarial estruturada (PostgreSQL) + semântica (embeddings, quando valer) + operacional (estado transacional). |
| Maestri | Cockpit/canvas para orquestrar Claude Codes. **Fora do escopo** do projeto desde 2026-10-04 ([[DEC-0020-maestri-fora-do-escopo-e-n8n-executa-os-agentes]]). |
| Partitura | Configuração de canvas salva do Maestri (histórico; fora do escopo). |
| ADR | Architecture Decision Record. |
| VOC | Voice of Customer (dataset de linguagem do cliente, saída de A1). |
| CORE v0.1 | Primeira entrega de desenvolvimento ([[CORE-v0.1]]). |
