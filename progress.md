# Progresso — Da Reunião ao Documento

Controle do trabalho de produzir o pacote de design docs da feature de Webhooks de Notificação de Pedidos,
a partir de `TRANSCRICAO.md` e do código da aplicação.

Cada item marcado abaixo corresponde a um commit. Cada bloco corresponde a um Pull Request.

---

## Regras do jogo

1. **Nada entra sem origem.** Toda afirmação nos documentos precisa vir da transcrição ou do código.
   O que não tem origem vira questão em aberto, não vira invenção.
2. **Os documentos não se contradizem.** Existe um contrato de fatos canônico com os números, nomes e
   caminhos oficiais. Nenhum documento pode divergir dele.
3. **Cada documento tem uma altura.** PRD responde *por que e o quê*; RFC, *como pretendemos resolver*;
   ADR, *por que decidimos assim*; FDD, *como construir*. Conteúdo repetido é sinal de coisa no lugar errado.
4. **O código é a fonte da verdade sobre o código.** Onde a transcrição diverge do repositório, o repositório
   ganha e a divergência é registrada.
5. **Não alterar `src/`, `prisma/`, `tests/` nem configuração.** A entrega é documental.

---

## Estado das decisões

A reunião fechou **50 decisões** (`F01`–`F50`), descartou **4 itens de escopo** (`X01`–`X04`),
deixou **13 alternativas** para trás (`A01`–`A13`) e **5 questões em aberto**.
O código forneceu **25 fatos verificados** (`C01`–`C25`).

Para o FDD ficar acionável, faltavam decisões de implementação que a reunião não tomou. Foram fechadas
**30 decisões complementares** (`H01`–`H30`), cada uma carimbada com sua procedência:

| Carimbo | Significado | Vai ao Tracker como |
|---|---|---|
| `MERCADO` | Padrão de indústria verificado (Stripe, GitHub, Standard Webhooks) | Decisão de implementação, com a referência citada |
| `CODIGO` | Derivado de padrão que já existe no repositório | Fonte `CODIGO` + caminho real |
| `PRODUTO` | Chamada de quem produziu a documentação | Decisão de implementação, nunca atribuída a participante |

**Nenhuma decisão complementar pode ser apresentada como fala da reunião.**

### As sete que mais mudam o desenho

| | Decisão | Procedência |
|---|---|---|
| `H02` | HMAC assina **somente o corpo cru** — o `X-Timestamp` fica fora. A lacuna vira `Q07` | `[09:22]`, literal |
| `H06` | Secret em texto claro + `redact` no Pino; criptografia at-rest vira `Q06` | `[09:46] Sofia` |
| `H11` | Toda falha consome as 5 tentativas — sem atalho de 4xx para a DLQ | `[09:15]` `[09:17]`, literal |
| `H13` | Worker serial dentro de cada `order_id`, paralelo entre pedidos, concorrência 10 | `[09:12]` + `[09:04]` |
| `H15` | Fan-out: N endpoints assinantes geram N linhas na outbox, com N `eventId` | `PRODUTO` |
| `H.4` | `customerId` no body do POST e query param na listagem — fecha a `Q03` | `[09:32]` + `CODIGO` |
| `H23` | O que a reunião decidiu vira constante citando o ADR; só tuning vira env | `PRODUTO` |

---

## Régua de qualidade

Alvo declarado antes de escrever, para que o corte seja critério e não impressão:

| Documento | Alvo | Teto onde o retorno vira negativo |
|---|---|---|
| PRD | 3.500–4.500 palavras · 16–18 RF · 8–10 riscos | acima de 6.000 palavras perde altitude |
| RFC | 1.800–2.100 palavras | acima de 2.300 vira FDD disfarçado |
| FDD | 12.000–14.000 palavras · 8+ endpoints · 12–14 códigos `WEBHOOK_*` | acima de 18.000 vira reexplicação |
| ADR | 10–14 KB cada, 8 no total | acima de 18 KB por ADR o custo/benefício desaba |
| Tracker | densidade importa mais que linhas: colunas de contexto > contagem | 500 linhas rasas perdem para 130 linhas ricas |
| README | 3.000–3.500 palavras · 4 prompts integrais · 6 iterações | — |

**Volume não vira nota; densidade vira.** O teste binário: se um parágrafo sobrevive à troca do nome da
feature, é enchimento. Se ele cita um arquivo real, um intervalo de linha, um número com fonte, uma coisa que
deliberadamente não vai ser feita ou uma opção descartada com quem a derrubou, é densidade.

### Modos de falha a vigiar

Lista fechada antes da redação. Cada item vira uma verificação no fim, e ter a lista antes muda o que se
procura — sem ela, a revisão só encontra o que já esperava encontrar.

- [ ] Item descartado na reunião aparecendo como requisito — a falha central do enunciado
- [ ] Mesmo item classificado de três formas em três documentos
- [ ] Contradição numérica entre documentos, do tipo 5 tentativas no PRD contra 6 chamadas HTTP no FDD
- [ ] Citar arquivo que não existe no repositório
- [ ] Tabela de auditoria com conferências falsas
- [ ] Métrica tipada como histograma ou gauge quando a infra só produz linha de log
- [ ] SLA ou percentil inventado ocupando o campo "meta"
- [ ] RFC ultrapassando o teto de 4 páginas
- [ ] Campo em payload de resposta que não existe no schema proposto
- [ ] Frase no futuro sobre artefato do próprio pacote
- [ ] Um único commit para o pacote inteiro — torna inauditável a narrativa de iteração

---

## Plano de entrega

Sete PRs empilhados: cada branch sai da anterior, e o README cresce a cada uma, mostrando a evolução.

### PR 1 · `docs/00-processo` — base do processo

- [x] `progress.md` com plano, decisões e régua de qualidade
- [x] `docs/processo/FATOS.md` — contrato de fatos canônico (`F`, `X`, `A`, `C`, `H`, `Q`), rotulado como artefato de processo, não entregável
- [x] README v1 — Sobre o desafio · Ferramentas de IA · Workflow adotado

### PR 2 · `docs/01-adrs` — as decisões

Oito ADRs em MADR. Cobrem as 6 decisões obrigatórias do enunciado, mais snapshot e controle de acesso.

- [x] `ADR-001-outbox-transacional-no-mysql.md`
- [x] `ADR-002-worker-separado-em-polling.md`
- [x] `ADR-003-retry-com-backoff-e-dlq.md`
- [x] `ADR-004-hmac-sha256-com-secret-por-endpoint.md`
- [x] `ADR-005-entrega-at-least-once-com-event-id.md`
- [x] `ADR-006-reuso-dos-padroes-do-projeto.md` — cita código real
- [x] `ADR-007-snapshot-do-payload-na-insercao.md`
- [x] `ADR-008-controle-de-acesso-dos-endpoints.md` — cita `requireRole` em `src/middlewares/auth.middleware.ts:49-61`
- [x] `docs/adrs/README.md` — índice, e a lista do que **deliberadamente não virou ADR** com a justificativa
- [x] README v2

### PR 3 · `docs/02-rfc` — a proposta

- [ ] `docs/RFC.md` — metadados, TL;DR, contexto, proposta, alternativas, questões em aberto, impacto, links para os ADRs
- [ ] Diagrama C4 nível 1 (contexto) e nível 2 (contêineres)
- [ ] README v3

### PR 4 · `docs/03-fdd` — a implementação

- [ ] `docs/FDD.md` — fluxos, contratos, erros, resiliência, observabilidade, critérios de aceite
- [ ] Seção "Integração com o sistema existente" com caminhos e intervalos de linha reais
- [ ] Modelos Prisma propostos para as 4 tabelas, com as convenções derivadas do código
- [ ] Diagrama C4 nível 3 + sequência do fluxo feliz + sequência de falha/retry/DLQ + máquina de estados + ER
- [ ] Matriz de erros `WEBHOOK_*`, incluindo o tratamento do código órfão `WEBHOOK_SECRET_REQUIRED`
- [ ] README v4

### PR 5 · `docs/04-prd` — o produto

- [ ] `docs/PRD.md` — resumo executivo, problema, público, objetivos com meta, escopo, requisitos, riscos, aceitação
- [ ] Tabela de fora de escopo com a classificação única de cada item
- [ ] README v5

### PR 6 · `docs/05-tracker` — a rastreabilidade

- [ ] `docs/TRACKER.md` — tabela no formato obrigatório, com colunas extras de seção-destino e estado
- [ ] Índice reverso: os 54 timestamps distintos da fita, com o que cobre cada um e o motivo de quem ficou fora
- [ ] Seção separada das decisões sem âncora na reunião (`H01`–`H30`), com a procedência de cada uma
- [ ] README v6

### PR 7 · `docs/06-readme` — o processo

- [ ] README final — prompts customizados, iterações e ajustes, como navegar a entrega
- [ ] Matriz de cobertura dos critérios de aceite do enunciado
- [ ] Passe de humanização na prosa (README e seções narrativas de PRD e RFC)
- [ ] Varredura final de consistência entre todos os documentos

---

## Verificação antes de fechar

- [ ] Todo caminho de arquivo citado nos documentos existe no repositório
- [ ] Todo timestamp `[hh:mm] Nome` citado existe em `TRANSCRICAO.md` (155 falas, 54 timestamps distintos, 128 pares únicos)
- [ ] Nenhum número diverge entre documentos
- [ ] Nenhum item descartado na reunião aparece como requisito
- [ ] Nenhum campo de payload de resposta está fora do schema proposto
- [ ] Nenhuma ferramenta de observabilidade nomeada — a reunião nunca citou nenhuma
- [ ] `src/`, `prisma/`, `tests/` e configuração intocados
