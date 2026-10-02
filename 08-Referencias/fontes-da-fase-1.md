---
tipo: referencia
area: 08-Referencias
fonte: pasta FASE  1 - BASE
status: inventario
atualizado: 2026-10-02
---
# Fontes da FASE 1 - BASE

Os originais ficam em `FASE  1 - BASE/` (**fonte da verdade, intactos**). As notas do vault são derivadas e citam o arquivo de origem no frontmatter.

| Arquivo original | Tamanho | O que realmente contém | Usado em |
|---|---|---|---|
| `blueprint_empresa_lancamentos_autonoma_maestri_v1.md` | 37 KB | Blueprint Técnico v1.0 (status: "aprovado para início da implementação") | quase todas as notas |
| `PLANO FINAL DE ENGENHARIA.txt` | 28,8 KB | Arquitetura-alvo, workflows por agente, fases 0–12, CORE v0.1 | agentes, roadmap, arquitetura |
| `RESPOSTAS DAS FASES DE EXTRAÇÃO - …txt` | 28,8 KB | cópia praticamente idêntica do Plano Final (dif. 2 bytes) | — (duplicado) |
| `MAPA DE EXTRAÇÃO DE FUNCIONALIDADES … PERGUNTAS….txt` | 10 KB | proposta conceitual inicial (livro → processos → funções → agentes; ficha técnica; dimensionamento; Mapa A x Mapa B) | [[visao-do-projeto]], [[perguntas-em-aberto]] |
| `a forma como vamos implementar o projeto.txt` | 0 B | vazio | — |
| `launch-os-starter-claude-maestri-v1.zip` | — | ver [[starter-zip-inventario]] | rules, DoD, roadmap, setup |

## Documento adicional (fornecido pela Madu em 2026-10-02)
`FASE-1-BASE_DECISOES_CANONICAS_v1.md` — registrado integralmente como [[DEC-0010-decisoes-canonicas-fase-1-v1]]. **Nível 1 de autoridade**: prevalece sobre Blueprint, Plano Final e starter. O arquivo original foi enviado por upload e **não foi copiado para `FASE  1 - BASE/`**; a transcrição vive em `02-Decisoes/`.

## Ideias do MAPA ainda **não** desdobradas em outras notas (candidatas a trabalho futuro)
- Ficha técnica por agente com **nível de autonomia em %** e KPI (exemplo: Estrategista 80%) — hoje só há AUTO/SUPERVISED/HUMAN.
- Classificação de cada função: Agente autônomo / Agente + supervisão / Humano.
- Dimensionamento por carga: MVP (1 lançamento) · Profissional (5) · Escala (20+), com X agentes e X humanos **não calculados**.
- Dois mapas: **A** (empresa com humanos) e **B** (empresa com agentes), para medir o grau de automação.
- Hierarquia alternativa com Diretores (Estratégia, Marketing, Operações, Vendas, Dados) e especialistas — **superada** pela decisão dos 11 agentes ([[DEC-0003-16-funcoes-em-11-agentes]]).
- As 8 fases de trabalho do MAPA (engenharia do livro → … → blueprint técnico) já foram percorridas até o Plano Final.

## Fontes externas citadas (não consultadas nesta sessão)
- PDF `A_formula_do_lancamento_Jeff_Walker.pdf` — **fora do projeto por decisão** ([[DEC-0010-decisoes-canonicas-fase-1-v1]] §2: fonte metodológica externa, não incorporar a `knowledge/plf/`); o Blueprint cita linhas 4197–4217 (checklist de abertura), 4214–4232 (monitoramento), 4493–4505 (pós-lançamento), 5430–5468 (internal antes de joint), 6049–6088 (Circle of Awesome), 6088–6106 (perene).
- Documentação do Maestri (URLs do Blueprint §48): `https://www.themaestri.app/pt-br/docs/` + `intro`, `terminals`, `connections`, `workspaces`, `partituras`, `file-tree`, `notes`. As afirmações sobre o Maestri nos docs são "fonte externa" e **não foram revalidadas**.
