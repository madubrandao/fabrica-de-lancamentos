---
tipo: proposta-especificacao
area: 03-Arquitetura
status: PROPOSTA NÃO APROVADA
atualizado: 2026-10-16
origem: requisitos criativos expressos pela operadora; dimensões sugeridas para primeiros formatos
relacionados: "[[proposta-afropilot-studio-producao-criativa]], [[A3-content-copy]]"
---
# PROPOSTA — Padrão de criação de imagens do AfroPilot Studio

> **Especificação inicial, não implementada.** Define como solicitar e avaliar imagens. Não define modelos de IA, permissões de produção, identidade de marca aprovada nem o contrato definitivo do runtime.

## 1. Princípio de criação

**Clareza → Personalidade → Conexão → Autoridade → Estética.** O resultado deve comunicar bem antes de impressionar visualmente. Evitar imagens genéricas, personagens desconectados do público, promessas inventadas e textos ilógicos. Respeitar a representação de afroempreendedoras sem caricaturas ou estereótipos.

Separar **geração de elementos visuais** de **diagramação da peça final**. Textos, títulos e CTAs são aplicados por composição programática editável (SVG/HTML/CSS ou motor escolhido), não confiados ao gerador de imagens. Isso permite corrigir texto sem gerar toda a arte de novo.

## 2. Tipos de peça e formatos iniciais

| Tipo | Exportação de referência | Observações |
|---|---|---|
| Post para feed | 1080 × 1350 px (4:5) | Primeiro formato do MVP. |
| Carrossel | 1080 × 1350 px por slide (4:5) | Um arquivo individual por slide; identidade consistente entre slides. |
| Story | 1080 × 1920 px (9:16) | Margens para elementos de interface; zonas seguras definidas por template. |
| Capa de Reel | 1080 × 1920 px (9:16) | Verificar cortes nas prévias do perfil. |
| Post quadrado | 1080 × 1080 px (1:1) | Alternativa quando solicitada. |
| Hero de site | Dimensões definidas por layout responsivo | Arquivos desktop e mobile separados se a composição precisar mudar. |

Formatos são **pontos de partida**, sujeitos às exigências vigentes das plataformas na data da implementação. Sem upscale artificial como substituto de resolução real.

## 3. Identidade visual por marca

Nunca usar o mesmo tema indiscriminadamente para Sol, Afro Ads e outros negócios. Cada job recebe `brand_id` e `brand_version` aprovados, contendo: paleta, fontes com licença, logos, ilustrações, contraste, tom visual, tipografia, uso de rostos e elementos proibidos.

**Referências provisórias fornecidas pela operadora em conversas, NÃO extraídas deste ZIP e NÃO incorporadas como tokens oficiais:**

- **Afro Ads institucional:** café `#1E130A`, dourado `#C39B6B`, champanhe `#E4BB8B`, marfim `#F2E8D8`, terracota `#A56A4B`; identidade premium e acolhedora.
- **Linha editorial social da Sol / Afro Ads:** teal profundo `#123A38`, laranja queimado `#E0611C`, halo solar, acentos dourados, estética editorial afrofuturista. **É uma direção de conteúdo, não uma substituição da identidade oficial do produto ou do dashboard.**

Esses perfis exigem confirmação em BrandProfile antes de uso automático.

## 4. Contrato de briefing proposto (`VisualBrief`, NÃO schema em código)

Campos mínimos e interpretação:

| Campo | Obrigatório? | Finalidade |
|---|---|---|
| `project_id`, `brand_id`, `brand_version` | Sim | Isolar projeto, cliente e tokens visuais. |
| `source_ref`, `source_kind` | Sim | Apontar para texto/brief autorizado; distinguir ficção de material autorizado. |
| `channel`, `asset_type`, `width`, `height` | Sim | Destino e tamanho do artefato. |
| `objective`, `audience`, `key_message` | Sim | Uma ideia central clara por peça. |
| `approved_copy` | Sim para arte com texto | Texto literal a aplicar na composição. |
| `visual_direction`, `reference_assets` | Sim (lista pode ser vazia) | Estilo, enquadramento, material de apoio e proveniência. |
| `required_elements`, `forbidden_elements` | Sim | O que deve entrar e o que deve ser evitado. |
| `consent_and_licensing` | Sim | Direitos de uso de fotos, modelos, fontes e terceiros. |
| `output_format`, `editable_source_required` | Sim | PNG/WebP/JPEG + arquivo-fonte quando cabível. |
| `human_review_required` | Sim, `true` no MVP | Estado obrigatório antes de qualquer publicação. |

**Exemplo ilustrativo de brief, não um objeto já aceito pela API:**

```yaml
project_id: sol-launch-demo
brand_id: sol-editorial-provisorio
brand_version: proposta-v0
source_ref: brief-ficticio-001
source_kind: fictional
channel: instagram-feed
asset_type: static_post
width: 1080
height: 1350
objective: educar trancistas sobre conflitos de agenda
audience: trancistas autonomas
key_message: cada servico ocupa uma duracao diferente
approved_copy:
  headline: 'Já marcou duas clientes no mesmo horário?'
  supporting: 'Uma agenda precisa considerar o tempo de cada serviço.'
visual_direction: 'composição editorial, contraste alto, folhagens não obrigatórias'
reference_assets: []
required_elements: ['hierarquia legivel', 'espaco negativo para o texto']
forbidden_elements: ['depoimento inventado', 'logo não autorizado', 'promessa de funcionalidade não validada']
consent_and_licensing: 'somente ativos próprios ou licenciados; nenhuma imagem de cliente real'
output_format: png
editable_source_required: true
human_review_required: true
```

## 5. Instruções para o motor de geração de elementos (não texto)

Prompts devem descrever: **assunto, composição, estilo, iluminação, textura, enquadramento, paleta, espaço vazio, restrições e formato**. Incluir referência permitida quando houver. **Não inserir credenciais ou dados pessoais no prompt.**

**Exemplo de prompt visual para o cenário fictício da Sol:**

> Ilustração editorial premium para conteúdo educativo de uma empresa de tecnologia para profissionais de tranças. Composição vertical 4:5, retrato artístico de uma empreendedora negra representada com naturalidade e respeito, iluminação solar suave, profundidade e elementos abstratos relacionados a organização e tempo. Estética contemporânea e autoral, fundo teal profundo com pequenos detalhes em laranja queimado, espaço negativo na metade superior esquerda para título aplicado posteriormente. Sem palavras, letras, interface falsa, logotipos, números, marcas ou aparência de testemunho de cliente real.

**Variações permitidas:** enquadramento, luminosidade, textura, distribuição de elementos. **Invariantes:** texto aprovado, proporção, mensagem central, tokens de marca e restrições de direitos.

## 6. Render, qualidade e aprovação

- **Pré-render:** conferir texto autorizado, ordem de leitura, idioma pt-BR, contraste, margens, legibilidade móvel, continuidade entre slides e fidelidade ao estilo.
- **Qualidade do arquivo:** largura/altura corretas, formato sem perda excessiva, nenhum corte de rosto, braços, cabelos ou elementos essenciais; transparência preservada quando necessária.
- **Conformidade:** direitos de imagem e modelos, consistência da marca, ausência de promessas comerciais não aprovadas, identificação interna do uso de IA quando aplicável.
- **Entrega:** imagem final + fonte editável + prévia + manifest (versão de marca, IDs de entrada, ferramentas/modelos, parâmetros não sensíveis, hashes, licença/proveniência, resultado do QA).
- **Revisão:** resultado `needs_changes`, `approved_for_manual_use` ou `rejected` registrado pela operadora. Aprovação editorial não aciona publicação e não modifica gates do LAUNCH OS.

**Critério de pronto do MVP:** produzir **um PNG 1080×1350 e uma fonte editável**, com texto legível, arte coerente, direitos verificados, manifest e aprovação/rejeição manual; refazer a mesma peça a partir dos insumos versionados sem alterar indevidamente a mensagem.

## 7. Não fazer no primeiro ciclo

- Gerar as 7 proporções responsivas em lote antes de comprovar um post 4:5.
- Criar faces de clientes ou depoimentos fictícios para simular evidência real.
- Prometer recursos da Sol cuja disponibilidade ainda não tenha sido confirmada.
- Instalar automaticamente modelos pesados, requisitar GPU remota ou usar serviços pagos.
- Publicar diretamente em redes sociais ou autorizar gastos sem decisão da operadora.

## 8. Pendências de validação

BrandProfiles oficiais; famílias de fontes e licenças; modelo de geração opcional; critérios quantitativos de QA; versionamento de fontes editáveis; local de armazenamento; políticas de retenção; integração técnica efetiva com as aprovações do LAUNCH OS. Nenhuma destas escolhas está implementada por esta proposta.

Ver [[proposta-afropilot-studio-producao-criativa]] · [[A3-content-copy]].
