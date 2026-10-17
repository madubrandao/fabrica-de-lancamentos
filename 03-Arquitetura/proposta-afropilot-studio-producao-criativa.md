---
tipo: proposta-arquitetura
area: 03-Arquitetura
status: PROPOSTA NÃO APROVADA
atualizado: 2026-10-16
origem: visão expressa pela operadora; opções técnicas candidatas, ainda sem validação de licença/benchmark neste vault
relacionados: "[[proposta-afropilot-assistente-executivo]], [[A3-content-copy]], [[proposta-afropilot-studio-especificacao-de-imagens]]"
---
# PROPOSTA — AfroPilot Studio: produção de conteúdo visual e audiovisual

> **Módulo futuro, não implementado.** Nenhuma ferramenta é autorizada para instalação, execução, compra ou publicação por este documento. O Studio NÃO altera os marcos M00–M06 nem acrescenta automaticamente um agente A11.

## 1. Objetivo e limite

Converter briefs e conteúdo aprovados em **arquivos criativos editáveis e exportáveis**: posts estáticos, imagens para carrossel, Stories, thumbnails e, numa etapa posterior, vídeos curtos com animação, narração e legendas. Servir tanto ao A3 durante lançamentos quanto à produção recorrente da Afro Ads fora de lançamentos.

**A3 define o que comunicar. Studio define como materializar visualmente.** O Studio não cria promessas comerciais, pesquisa fictícia apresentada como fato ou decisões estratégicas sem supervisão.

## 2. Pipeline proposto

```text
Operadora / A3 (ContentPackage + CreativeBrief + fontes)
          ↓
Validação de entrada, source, direitos de uso e marca
          ↓
Plano visual (layout, hierarquia, cenas/ativos, dimensões)
          ↓
Motor de composição / geração opcional de elementos
          ↓
Renderização de prévia → QA técnico e editorial automatizável
          ↓
Revisão editorial HUMANA (aprovar, pedir ajuste, rejeitar)
          ↓
Exportação local controlada (publicação é ação separada)
```

**Regras:** não montar peça final a partir de um `ContentPackage` que já foi rejeitado; preservar o vínculo com o briefing original; não alterar textos aprovados sem nova revisão; decisões editoriais separadas de aprovação técnica da execução.

## 3. Componentes propostos (ainda sem código)

| Componente | Faz | Entrada/saída principal |
|---|---|---|
| **BrandProfile** | Regras por marca, paleta, tipografia licenciada, logo, referências e zonas seguras. | `brand_id` → tokens versionados. |
| **CreativeBrief** | Objetivo, canal, audiência, texto, formato, estilo e restrições. | Brief A3/operadora → plano de criação. |
| **AssetRegistry** | Referências e direitos de imagens, áudio, fontes e logos, inclusive ativos gerados. | ativos → metadados de origem, licença e hash. |
| **ImageComposer** | Montagem determinística: texto, imagem, hierarquia, margens, overlays e exportação. | plano + ativos → PNG/WebP e fonte editável. |
| **ImageGenerator** (opcional) | Gera elementos visuais; não é responsável por escrever o texto da arte. | prompt sem dados sensíveis → imagem + parâmetros. |
| **VideoRenderer** (posterior) | Cenas, motion, narração e legendas. | storyboard + mídia → MP4 + manifest. |
| **QualityGate** | Verificações de dimensão, legibilidade, direitos, metadados e consistência. | arquivo → relatório de problemas. |
| **ReviewDelivery** | Reúne arquivos, versões, prévias e status para decisão da operadora. | pacote → aprovação/rejeição registrada. |

Esses nomes são **vocabulário de projeto**, não APIs, classes ou tabelas existentes.

## 4. Ferramentas abertas candidatas (não são decisões de stack)

| Uso | Candidatos | Condição antes de escolher |
|---|---|---|
| Arte estática determinística | SVG/HTML/CSS + Sharp para rasterizar/compor | Confirmar stack, tipografia e fidelidade visual em teste pequeno. |
| Geração opcional de imagens | ComfyUI com modelos compatíveis | Licença do **modelo** e de seus pesos, uso comercial, memória e custo de GPU. |
| Edição/ativos 3D | Blender; Krita | Avaliar necessidade real; evitar tornar o primeiro MVP dependente de 3D. |
| Vídeo, cortes e áudio | FFmpeg | Conferir build e licenças dos codecs em distribuição. |
| Motion programático | Revideo (MIT); Remotion (licença comercial condicional) | Fazer avaliação jurídica/operacional antes de adotar; não presumir todos como software livre irrestrito. |
| Voz sintética | TTS open source **ou** serviço gerenciado | Medir naturalidade pt-BR; conferir licença de modelo/voz e consentimento. |

**Não há decisão aprovada de ferramenta neste documento.** Software aberto não torna gratuitos a GPU, o armazenamento, as licenças de modelos ou as APIs. O notebook de referência tem 8 GB de RAM; o primeiro ensaio deve evitar modelos pesados locais, usando templates e renderização leve. Nuvem/GPU remota exigem autorização específica.

## 5. Entregas e fases propostas — sem multiplicar PRs

**V0 — Prova de valor assistida, sem acesso a produção:** a partir de um texto e brief fornecidos, gerar **um post 4:5** com fonte editável, PNG de prévia, manifesto de origem e checklist. Uma única tarefa consolidada; sem LLM real ou GPU obrigatória.

**V1 — Imagens em lote com revisão:** 3–5 variações de post/carrossel + identidade versionada + aprovação editorial manual + re-renderização controlada.

**V2 — Vídeo curto:** storyboard → cenas animadas → voz/licenças → legendas → MP4. Prova de custo/memória antes de decidir render local, CPU efêmera ou GPU.

**V3 — Integração com AfroPilot/LAUNCH OS:** entradas e saídas autenticadas, rastreabilidade por execução, estados, limites e aprovações. Só após marcos e decisões correspondentes; nenhuma automação externa presumida.

## 6. Direitos, segurança e controle editorial

- Imagem de cliente ou pessoa identificável: origem, direitos e autorização documentados; não gerar depoimento fotográfico fictício como se fosse cliente real.
- Todo ativo externo é entrada não confiável; metadados, URLs, arquivos e prompts devem ser validados na futura implementação.
- Não publicar, enviar mensagens ou contratar recursos sem aprovação expressa, orçamento e credenciais devidamente autorizados.
- Não usar marcas, logos, fontes comerciais ou obras de terceiros sem licença apropriada.
- Identificar internamente ativos sintéticos e preservar a proveniência; evitar alegações ou resultados da Sol não comprovados.
- O sistema não deve compartilhar segredos, imagens pessoais ou dados de clientes com provedores externos sem autorização e base de tratamento adequadas.

## 7. Relação com decisões existentes

- Preserva o mapa dos **11 agentes A0–A10** ([[DEC-0003-16-funcoes-em-11-agentes]]); Studio é **serviço de produção criativa**, não novo agente de negócio.
- A3 continua dono da estratégia de mensagem e do conteúdo ([[A3-content-copy]]). A6 pode coordenar entregas do lançamento, sem assumir renderização.
- Afropilot se comunica com LAUNCH OS pelas interfaces autorizadas; nunca escreve diretamente gates, approvals ou registros imutáveis.
- O início de execução operacional em LAUNCH OS permanece bloqueado pela [[DEC-0037-sequencia-d4-d3-backup-e-retencao]]; [[DEC-0038-preparacao-do-m03-em-prs]] está não aprovada no ZIP.
- A PoC local independente **não equivale** a iniciar o M03; seu escopo e autorização devem ser confirmados em tarefa própria antes de executá-la.

## 8. Decisões pendentes (não preencher implicitamente)

1. Ferramentas e licenças comerciais definitivas.
2. Formato e armazenamento de `CreativeBrief`, `BrandProfile`, jobs e revisões.
3. Separação entre serviço Studio autônomo e módulos no monorepo, com justificativa.
4. Onde o Studio receberá tarefas e como reportará estados/falhas/custos.
5. Política de orçamento, autorização da operadora e limites de recursos.
6. Identidade visual oficial de cada marca (ver exemplos provisórios na especificação de imagens).

Ver [[proposta-afropilot-studio-especificacao-de-imagens]] · [[proposta-afropilot-assistente-executivo]].
