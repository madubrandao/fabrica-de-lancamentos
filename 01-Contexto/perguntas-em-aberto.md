---
tipo: contexto
area: 01-Contexto
fonte: análise da leitura da FASE 1 - BASE (2026-10-02)
status: aberto
atualizado: 2026-10-04
---
# Perguntas em aberto

Itens que os documentos **não respondem**. Nada aqui foi assumido. Conflitos entre documentos estão em [[pendencias-de-decisao]].

## Material faltando ou inconsistente
- [ ] `a forma como vamos implementar o projeto.txt` está **vazio** (0 bytes). Existe conteúdo?
- [ ] `MAPA DE EXTRAÇÃO…AQUI ESTAO AS PERGUNTAS…txt` **não contém perguntas**; é a proposta conceitual inicial. Onde estão as perguntas reais?
- [ ] `RESPOSTAS DAS FASES DE EXTRAÇÃO…txt` é praticamente idêntico a `PLANO FINAL DE ENGENHARIA.txt` (diferença de 2 bytes). Onde estão as respostas reais?
- [x] ~~PDF do livro entra no projeto?~~ **Resolvido (DEC-0010 §2):** o PDF fica como fonte externa ao diretório operacional; não copiar nem incorporar a `knowledge/plf/`. Lacunas metodológicas ficam PENDENTES até consulta explícita à fonte.
- [ ] O Blueprint de Implementação v1 por agente (system prompt, tools, permissões, schemas, regras, gatilhos, workflows, critérios de aprovação) — pedido pelo Plano Final — **não existe ainda**.

## Negócio (decidido no [[DEC-0017-modelo-de-negocio-inicial]], 2026-10-04)
- [x] ~~Primeiro produto/oferta de teste~~ **produto fictício de teste** (C1). Pendente: definir o produto fictício, antes do M03.
- [x] ~~Modelo de operação~~ **só lançamentos próprios, uma única organização**; revisitar antes do M04 (C2).
- [x] ~~Dimensionamento~~ **capacidade inicial de 1 lançamento por vez**, arquitetura preparada para 5 (C3). Pendente: número de humanos que operam e aprovam, antes do M01.
- [x] ~~Orçamento e limites de gasto~~ **adiado para antes do M04** (C4). Aberto: limite de custo de LLM (reavaliar no M02).
- [x] ~~Plataformas~~ **adaptadores simulados agora; plataformas reais por ADR antes do primeiro lançamento real** (C5).

## Técnico (decidido no [[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]], 2026-10-04)
- [x] ~~Linguagem/framework~~ **TypeScript/Node** (D1). Pendente: framework de backend e de frontend (ADR no M01).
- [x] ~~Provedores de LLM~~ **Anthropic primeiro, com camada de adaptador** (D2). Pendente: modelo por agente (M02).
- [x] ~~Hospedagem/ambientes~~ **só dev em Docker local** (D3). Pendente: teste e produção, antes do M03.
- [x] ~~Vector store~~ **adiado**; M02 só reserva a interface (D4).
- [x] ~~Ambiente de dev no Windows~~ **Windows nativo + Docker Desktop**, com `.gitattributes` LF no M00 (D5).
- [x] ~~Repositório `launch-os`~~ **privado no GitHub, fora de `.claude`, sem espaços no caminho**, criado após o início formal do M00 (D6). Pendente: caminho e conta exatos.

## Processo
- [ ] Mapeamento estado da state machine → gate: **adiado para a especificação do M01 (ENG-00)** (D7).
- [ ] Limites numéricos: orçamento, tempo de aprovação, SLA de agentes: **adiados para o M01**, como configuração por política; orçamento antes do M04 (D8).
