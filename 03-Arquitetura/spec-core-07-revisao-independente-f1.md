---
tipo: arquitetura
area: 03-Arquitetura
fonte: revisão independente de 2026-10-07 (agente revisor, somente leitura) sobre os blocos 1 a 6; DEC-0010; DEC-0016 D7; M01-plano-de-execucao (F1)
status: correções de texto aplicadas em 2026-10-07; decisões D7.1 a D7.7 aguardando aprovação da Madu (M01, F1, fechamento)
atualizado: 2026-10-07
---
# Revisão independente da especificação do Core (blocos 1 a 6)

Fecha a F1 do [[M01-plano-de-execucao]] conforme a D6.10: um **agente revisor independente**, com acesso só de leitura, leu os 6 blocos ([[spec-core-01-servicos-e-limites]], [[spec-core-02-entidades-e-schema]], [[spec-core-03-state-machine-e-gates]], [[spec-core-04-eventos]], [[spec-core-05-permissoes-e-aprovacoes]], [[spec-core-06-ferramentas-de-codigo]]) e conferiu contra as fontes canônicas. Resultado: **0 críticos, 6 altos, 13 médios, 7 baixos**. Veredito do revisor: **pronta com correções, não pronta para o portão como estava.**

A Madu mandou seguir com as recomendadas (2026-10-07). Neste documento:
- a **seção 1** lista cada achado e onde foi tratado;
- a **seção 2** traz as 7 decisões novas (D7.1 a D7.7) que **mudam ou acrescentam** decisões já aprovadas, e por isso **só valem depois da aprovação da Madu**;
- a **seção 3** registra um número que o revisor errou e que conferi.

## 1. Tratamento dos achados
Numeração do relatório do revisor. **Aplicado** = correção de texto ou schema sem mudar decisão aprovada, já feita em 2026-10-07.

| # | Sev. | Achado (resumo) | Tratamento |
|---|---|---|---|
| 1 | alto | Aprovação de `action` sem vínculo com a ação e o valor | **D7.2** |
| 2 | alto | `idempotency_key.response` podia guardar segredo | **Aplicado:** bloco 4, seção 6 (`response_status` e `response_ref`, nunca o corpo; endpoints com segredo não usam resposta guardada); teste no bloco 4, seção 10 e no bloco 5 |
| 3 | alto | Ordem por entidade com `position` e `SKIP LOCKED` | **Aplicado:** bloco 4, seção 5 (`NOT EXISTS` do anterior, `dead` bloqueia a entidade, `position` não é cursor global); bloco 6, ADR-12 |
| 4 | alto | Faltam esqueletos e semente para a F2 | **D7.3** |
| 5 | alto | Bootstrap de identidades não definido | **D7.4** |
| 6 | alto | G04 antes de `PLC`, contra o DEC-0016 D7 | **D7.1** |
| 7 | médio | Aprovação sem estado de origem e sem unicidade | **Aplicado:** bloco 2, 3.6 (`from_status`, índice único parcial) |
| 8 | médio | G08 depende da G07 já consumida | **Aplicado:** bloco 3, seção 3 (G08); bloco 2, 3.6 |
| 9 | médio | Deduplicação do consumidor sem tabela e sem atomicidade | **Aplicado:** bloco 4, seções 5 e 6 (`claimed_at`, `processed_at`, claim atômico, ack, 2xx ao receber) |
| 10 | médio | Assinantes e identidade dos workflows | **Aplicado:** bloco 4, seção 5 (3 assinantes do M01); bloco 5, 3.2 (credencial de serviço do n8n) |
| 11 | médio | Dono do estado não casa com quem executa os gates | **D7.5** |
| 12 | médio | Decisão de continuidade sem modelo de proposta e confirmação | **D7.6** |
| 13 | médio | Audit de negação e `error.detected` dentro de transação com rollback; `entity_id` obrigatório; laço | **Aplicado:** bloco 4, seções 2 e 3 (transação própria, `entity_type = system`, sem laço) |
| 14 | médio | Nome do evento de T4 trocado | **Aplicado:** bloco 4, seção 4 (`launch.product_build_started`) |
| 15 | médio | `approval.expired` pela metade | **Aplicado:** bloco 4, seções 3 e 10; bloco 3, seção 7 (`EXPIRED`); catálogo e DEC-0011 (adendo) |
| 16 | médio | `approval_state` derivado mas persistido; `owner_agent` e `result` | **Aplicado:** bloco 2, 3.4 (CHECK com os 5 valores, atualização na mesma transação, teste); bloco 3, seção 7 |
| 17 | médio | Catálogo de endpoints ausente; `execution_id`; `correlation_id` fora da API | **Aplicado:** bloco 1, seção 5-bis (lista de intenções, cabeçalhos, erros, endpoint de correlação) |
| 18 | médio | Isolamento de credenciais no n8n; replay do webhook | **D7.7** (isolamento); **Aplicado:** limite do replay registrado no bloco 5, 3.3 |
| 19 | médio | Imutabilidade só com trigger por linha | **Aplicado:** bloco 2, seção 6 (`TRUNCATE`, dois papéis no banco, testes) |
| 20 | baixo | Justificativas de G05 e G09 e do A0 apresentadas como fato; "todos os 12 gates HUMANO" | **Aplicado:** bloco 3, seção 3 (PROP e citação do Blueprint §13); bloco 5, seção 13 |
| 21 | baixo | Plano ainda diz "as 12 transições com gate" | **Aplicado:** plano do M01, critérios de pronto |
| 22 | baixo | Item 9 do plano só em parte coberto | **Aplicado:** plano do M01 (item 9) e bloco 1, 5-bis |
| 23 | baixo | Metadados desatualizados e texto contraditório entre blocos 4 e 5 | **Aplicado:** blocos 1, 2, 4 e 5 |
| 24 | baixo | DEC-0017 C3 ("5 simultâneos") não citado ao deixar o Redis fora | **Aplicado:** bloco 4, seção 5 |
| 25 | baixo | Valores numéricos sem marca PROP | **Aplicado:** bloco 3 (G12), bloco 6 (ADR-11, 32 bytes) |
| 26 | baixo | `launch_id` e `gate_id` sem "ou nulo"; entidade dos eventos de aprovação | **Aplicado:** bloco 4, seção 3 |
| — | — | Bloco 6: ESLint `engines`, peers do adaptador Zod, riscos do `node-pg-migrate`, pnpm 12 | **Aplicado:** bloco 6 (ADR-1, 2, 5 e 7). A major do pnpm (11 ou 12) fica **PEND** para a F6 |

Todos os links internos já estavam resolvendo, segundo o revisor.

## 2. Decisões novas para a Madu (D7.x)
Cada uma muda ou acrescenta algo já aprovado. Se aprovar tudo como recomendado, é só dizer "siga com as recomendadas"; depois disso eu aplico nos blocos e registro.

| # | Pergunta | Opções | Recomendação do ENG-00 |
|---|---|---|---|
| **D7.1** | Onde fica o G04 (Content/PLC Approved) | **A)** mover para a transição `PLC → LAUNCH_READY` (T7), que passa a exigir G04, G05, G06 e G07; T6 (`PRE_PRE → PLC`) fica **sem gate**. **B)** manter em T6 e registrar a divergência com o DEC-0016 D7 para você aceitar | **A.** O DEC-0016 D7 diz que G04 a G07 caem entre `PLC` e `CART_OPEN`, e o conteúdo que o G04 aprova (PLC1 a PLC3, e-mails, sales page) é produzido pelo A3 **durante** `PLC`. Em T6 o gate aprovaria algo que ainda não existe. **Efeito:** muda a D3.1 (aprovada), a tabela do bloco 3 e a D3.8: passam a ser **9 transições com gate** cobrindo os 12 gates (ver seção 3) |
| **D7.2** | Vínculo da aprovação de `action` com a ação e o valor | **A)** `approval` ganha `action_name` e `action_params_hash` (hash dos parâmetros em forma canônica, incluindo o valor); a API compara os dois ao consumir a aprovação, e `subject_id` fica sem uso até o M03. **B)** deixar como está | **A.** Sem isso, o uso único não impede usar uma aprovação dada para um gasto em outro gasto maior (violaria o `BUDGET_AUTHORIZED`). Duas colunas novas em `approval` |
| **D7.3** | Quais tabelas a F2 cria e como nasce a semente de teste | **A)** criar as **22 tabelas**: as 8 do núcleo completas e as 14 de domínio **só com `id`, `organization_id` e carimbos**, mais as relações **já documentadas** (`offer.product_id`, `launch.product_id` e `launch.offer_id`, `learning.launch_id`); as colunas de negócio entram por migration aditiva no M03. A semente de produção traz só a organização e os 11 agentes; `product` e `offer` de teste vivem **só nos testes** (fixtures), nunca em `database/seeds`. **B)** criar só as tabelas necessárias ao M01 (núcleo, `product`, `offer`, `audience_segment`, `learning`) | **A.** Cumpre a F2 do plano ("22 entidades") e o critério do M01 sem inventar coluna de negócio; acrescentar colunas depois é aditivo e reversível. B é mais enxuto, mas deixa 10 entidades da lista oficial sem tabela e exige nova migration estrutural no M03 |
| **D7.4** | Como nascem a senha da Madu e as 11 credenciais | **A)** um comando de **bootstrap interativo**, rodado por você, fora do repositório: pede a senha (guarda só o hash), gera as credenciais dos agentes e do serviço do n8n e as **mostra uma vez** para você colar no *credential store* do n8n; tudo vai para o audit. As seeds só criam a organização e os agentes, **sem nenhum segredo**. Rodar de novo recusa se o usuário já existe; troca de credencial é pelo cliente de linha de comando. **B)** senha e chaves em arquivo de ambiente ou seed | **A.** B colocaria segredo em arquivo que pode ser versionado (guardrail do DEC-0010 §8). O comando fica no `apps/cli` já aprovado (D6.9) |
| **D7.5** | Quem pode pedir aprovação e disparar verificação de cada gate | **A)** por **dono do gate**, e não só do estado: G01 A1; G02 e G03 A2; G04 A3; G05 A4; G06, G07, G08 e G09 A6; G10 A8; G11 e G12 A10 (**PROP**, derivado das saídas dos agentes). Podem pedir: o dono do gate, o `owner_agent` do estado ou o A0. O resultado do QA do G06 **não tem entidade nem escopo definidos**: fica **PEND até o M03** e, no M01, o G06 é HUMANO. **B)** manter só o `owner_agent` do estado (D5.4) | **A.** Com B, o A6 não consegue disparar G05 e G06 durante `PLC`, cujo dono é o A3. A tabela de donos de gate é constante versionada em `packages/schemas/`, como a de transições |
| **D7.6** | Proposta e confirmação da decisão de continuidade | **A)** o A0 **propõe**: a API grava a `decision` com `status = proposed` e abre um `approval` do tipo `action` (`record_continuity_decision`, com a opção escolhida e os ids de produto e oferta do novo launch no hash da D7.2); quando você concede, a `decision` vira `confirmed`, a API cria o próximo launch e emite `decision.recorded` e `launch.completed`; se rejeitar ou vencer, a `decision` fica `rejected` ou `expired`. A proposta pede `product_id` e `offer_id` **explícitos** para o novo launch (SEED, INTERNAL, JOINT e RELAUNCH); os padrões (por exemplo, "mesma oferta") ficam **PEND** para a definição do A0. A regra do JOINT passa a dizer "existe, para a mesma oferta, um launch INTERNAL com decisão de continuidade **confirmada**". **B)** deixar o modelo atual | **A.** Reaproveita a máquina de aprovação já aprovada e deixa a proposta rastreável. `decision` ganha `status`, `proposed_by_type` e `proposed_by_id` (no bloco 2, 3.5) |
| **D7.7** | Isolamento das credenciais de agente no n8n | **A)** exigir que cada credencial de agente seja usada **só nos workflows daquele agente** (projetos do n8n ou compartilhamento restrito), com **teste na F6** de que um workflow não acessa a credencial de outro; entra como requisito do ENG-03 e do ENG-07. **B)** não exigir | **A.** A defesa do bloco 5 ("uma credencial por agente") só funciona se o n8n impedir o uso cruzado, e os workflows de agente processam entrada externa e LLM (risco de injeção de prompt). A viabilidade no n8n 2.41.6 é **verificada na F6**; se não for possível, volto a você com alternativa |

## 3. Número que o revisor errou (conferido)
O revisor disse que, ao mover o G04 para T7, "continuam 10 transições com gate". **Não continuam.** Os gates ficam em T3 (G01), T4 (G02), T5 (G03), T7 (G04 a G07), T8 (G08), T10 (G09), T12 (G10), T13 (G11) e T14 (G12): **9 transições com gate**, cobrindo os 12 gates. T6 passaria a ficar sem gate (junto de T1, T2, T9 e T11). Isso entra na D7.1 e, se for aprovada, o texto da D3.8 passa de "10" para "9".

## 4. O que fica para depois
- **Segunda passada** do revisor, só nos pontos alterados, antes do DEC-0023 (aprovação do portão F1).
- **Dono do resultado do QA** do G06 (M03) e critérios objetivos dos gates de VERIFICAÇÃO (a Madu define).
- **Major do pnpm** (11 ou 12) e versões exatas (F6).
- Gravar os ADRs em `docs/adr/` do `launch-os` e atualizar o `docs/status/` de lá (ação externa, confirmo antes).

Ver [[M01-plano-de-execucao]] · [[gates]] · [[launch-state-machine]] · [[permissoes-e-hitl]].
