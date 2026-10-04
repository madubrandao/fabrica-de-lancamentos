---
tipo: decisao
area: 02-Decisoes
id: DEC-0017
status: aceita
fonte: decisão da Madu em 2026-10-04; PLANO FINAL §26–§27; MAPA (dimensionamento); blueprint v1 §24, §40 (passo 15), §47; DEC-0010 §3
data-registro: 2026-10-04
resolve: C1, C2, C3, C4, C5 (bloco C de perguntas-em-aberto)
---
# DEC-0017 — Modelo de negócio inicial

## Contexto
Nenhum documento definia produto, modelo de operação, dimensionamento, orçamento nem plataformas. O primeiro lançamento previsto é **simulado**, em ambiente de teste (Blueprint §40 passo 15; [[criterio-de-sucesso-v1]]), então nada deste bloco trava o M00. Prazos de decisão: C2 e C3 antes do M01; C1 antes do M03; C4 e C5 antes do M04.

## Decisões (todas as recomendadas aprovadas pela Madu)

### C1 — Primeiro produto: fictício, de teste
O primeiro ciclo usa um **produto fictício de teste**. Valida o pipeline sem depender de mercado real, dados externos nem gasto. O produto real entra num ciclo posterior, por decisão da Madu.
- **PENDENTE:** definição do produto fictício (nome, oferta, público de teste), antes do M03.

### C2 — Modelo de operação: só lançamentos próprios
Uma única `Organization`. O `organization_id` permanece no modelo de dados e nos eventos ([[DEC-0012-lista-de-entidades-do-blueprint]]), mas **sem** multi-cliente, cobrança nem painel por cliente. Revisitar antes do M04.

### C3 — Dimensionamento: capacidade inicial de 1 lançamento por vez
Ativa o caminho crítico ([[DEC-0008-caminho-critico-primeiro]]) com a arquitetura preparada para 5 lançamentos simultâneos (filas). O termo "MVP" do MAPA vira "capacidade inicial" (o DEC-0001 rejeita MVP descartável). Os 11 agentes já estão fechados (DEC-0003).
- **PENDENTE:** número de humanos que operam e aprovam os lançamentos. O vault só registra a Madu como autoridade máxima; não assumido.

### C4 — Orçamento: adiado para antes do M04
O lançamento simulado não gasta. Antes do M04 a Madu define: moeda, limite por campanha, por lançamento e por período, e quem aprova acima do limite. Os valores são configuração por política ([[DEC-0016-stack-ambiente-e-adiamentos-tecnicos]] D8), não valor fixo no código. O policy check `BUDGET_AUTHORIZED = true` continua valendo ([[gates]]).
- **ABERTO (não decidido):** limite de custo de uso de LLM. Nenhum documento o cobre; a Madu não o incluiu agora. Reavaliar no M02, quando houver execução de agentes.

### C5 — Plataformas: adaptadores simulados agora, escolha real por ADR
Nos M03 e M04 as integrações têm **implementação simulada** atrás de interfaces (`packages/integrations/`). As plataformas reais são escolhidas por categoria, em ADR, **antes do primeiro lançamento real**. Categorias vindas dos documentos: páginas (sales page, order form, thank-you), checkout e pagamento, CRM e e-mail, tracking, anúncios e publicação de conteúdo.

## Alternativas rejeitadas
- C1: produto real agora (exigiria fontes de dados externos para o A1, ainda inexistentes); produto de cliente (depende da C2).
- C2: serviço para clientes desde o início (isolamento entre clientes, autenticação por organização e cobrança; aumento de escopo e de risco).
- C3: 5 lançamentos simultâneos já na primeira versão; 20+ (fora do escopo razoável antes do Core).
- C4: definir valores agora (a Madu define os números; não são inventados).
- C5: escolher as plataformas agora, antes de haver o que integrar.

## Consequências
- O M03 precisa do produto fictício definido; o schema do M01 mantém `organization_id` com uma única organização.
- PENDENTES: produto fictício (antes do M03), número de humanos (antes do M01), política de orçamento (antes do M04), limite de custo de LLM (M02), plataformas reais (antes do primeiro lançamento real).
- Nada foi implementado.

Ver [[perguntas-em-aberto]] · [[visao-do-projeto]] · [[A5-acquisition-growth]] · [[milestones]].
