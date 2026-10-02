---
tipo: contexto
area: 01-Contexto
fonte: análise da leitura da FASE 1 - BASE (2026-10-02)
status: aberto
atualizado: 2026-10-02
---
# Perguntas em aberto

Itens que os documentos **não respondem**. Nada aqui foi assumido. Conflitos entre documentos estão em [[pendencias-de-decisao]].

## Material faltando ou inconsistente
- [ ] `a forma como vamos implementar o projeto.txt` está **vazio** (0 bytes). Existe conteúdo?
- [ ] `MAPA DE EXTRAÇÃO…AQUI ESTAO AS PERGUNTAS…txt` **não contém perguntas**; é a proposta conceitual inicial. Onde estão as perguntas reais?
- [ ] `RESPOSTAS DAS FASES DE EXTRAÇÃO…txt` é praticamente idêntico a `PLANO FINAL DE ENGENHARIA.txt` (diferença de 2 bytes). Onde estão as respostas reais?
- [x] ~~PDF do livro entra no projeto?~~ **Resolvido (DEC-0010 §2):** o PDF fica como fonte externa ao diretório operacional; não copiar nem incorporar a `knowledge/plf/`. Lacunas metodológicas ficam PENDENTES até consulta explícita à fonte.
- [ ] O Blueprint de Implementação v1 por agente (system prompt, tools, permissões, schemas, regras, gatilhos, workflows, critérios de aprovação) — pedido pelo Plano Final — **não existe ainda**.

## Negócio (nenhum doc define)
- [ ] Qual é o primeiro produto/oferta de teste? Nicho/mercado?
- [ ] Modelo de operação: só lançamentos próprios, ou serviço para clientes? (o MAPA cita cenários de 1, 5 e 20+ lançamentos/clientes, sem valores)
- [ ] Dimensionamento: quadro mínimo / profissional / escala (a tabela do MAPA tem "X" por preencher).
- [ ] Orçamento e limites de gasto por política (A5 "não aumenta orçamento indefinidamente"; o limite não está definido).
- [ ] Quais plataformas de checkout, e-mail/CRM, anúncios e páginas serão usadas?

## Técnico (nenhum doc define)
- [ ] Linguagem/framework do backend e do frontend.
- [ ] Provedores de LLM (o `.env.example` do starter lista Anthropic e OpenAI como opções, sem decisão).
- [ ] Hospedagem/ambientes (dev, teste, produção).
- [ ] Vector store para memória semântica (Blueprint: "somente quando trouxer valor"; Plano: "Vector/semantic memory" na Fase 1).
- [ ] Ambiente de dev no Windows: WSL, tmux ou local (starter recomenda decidir depois de instalar e verificar Claude Code e toolchain).
- [ ] Repositório `launch-os` (Git/GitHub): quando criar e onde.

## Processo
- [ ] Mapeamento estado da state machine → gate (os docs listam os dois, não os ligam).
- [ ] Limites numéricos: orçamento, tempo de aprovação, SLA de agentes.
