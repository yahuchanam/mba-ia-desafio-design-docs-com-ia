# Contrato de Fatos Canônicos

> **Artefato de processo, não entregável.** Este arquivo não faz parte do pacote de design docs
> exigido pelo desafio. Ele é a base de fatos que sustenta PRD, RFC, FDD, ADRs e Tracker, publicada
> aqui para que a rastreabilidade seja auditável por terceiros em vez de apenas afirmada.
> Os entregáveis são `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`, `docs/TRACKER.md` e `docs/adrs/`.

Fonte única de verdade. Nenhum número, nome ou caminho pode aparecer em PRD, RFC, FDD ou ADR
divergente do que está registrado aqui.

Regra de rigor: **ESTRITO** — todo item precisa de fala na transcrição ou evidência no código.
O que não tem origem vira questão em aberto, nunca invenção.

## Como ler os identificadores

| Prefixo | O que é | Fonte |
|---|---|---|
| `F01`–`F50` | Decisão fechada na reunião | `TRANSCRICAO.md` |
| `X01`–`X04` | Item descartado ou adiado | `TRANSCRICAO.md` |
| `A01`–`A13` | Alternativa considerada e recusada | `TRANSCRICAO.md` |
| `C01`–`C25` | Fato verificado no repositório | código |
| `D01`–`D02` | Divergência entre a fala e o código | ambos |
| `H01`–`H30` | Decisão complementar, tomada fora da reunião para o FDD ficar acionável | `MERCADO`, `CODIGO` ou `PRODUTO` |
| `Q01`–`Q07` | Questão em aberto, com dono | `TRANSCRICAO.md` |

---

## A. Decisões fechadas (fonte: TRANSCRICAO)

| # | Fato canônico | Valor exato | Localização |
|---|---|---|---|
| F01 | Padrão de entrega | Outbox no MySQL, insert na mesma transação SQL da mudança de status | `[09:06] Diego` |
| F02 | Garantia da outbox | Commit da transação ⇒ evento registrado; rollback ⇒ evento some junto | `[09:06] Diego` |
| F03 | Índices da outbox | Índice no campo de status e em `created_at` | `[09:08] Diego` |
| F04 | Estados da outbox | pendente, processando, falhou, entregue | `[09:08] Diego` |
| F05 | Leitura do worker | Lê só os pendentes, em batch pequeno, marca como entregue | `[09:08] Diego` |
| F06 | Modo do worker | Polling em loop, a cada **2 segundos**, eventos pendentes mais antigos | `[09:09] Diego` |
| F07 | Latência aceita | Pior caso 2s de latência mínima; requisito do cliente é **< 10 segundos** | `[09:10] Larissa` / `[09:02] Marcos` |
| F08 | Isolamento do worker | Processo separado, não na mesma instância da API | `[09:11] Diego` |
| F09 | Entry point | `src/worker.ts` + script `npm run worker`, análogo a `src/server.ts` | `[09:11] Larissa` |
| F10 | Prisma no worker | Mesmo banco, mesma `DATABASE_URL`, **PrismaClient separado** (é outro processo Node) | `[09:30] Bruno` |
| F11 | Ordering | Por `order_id`, implícita, ordem de `created_at`, **enquanto for single-worker** | `[09:12] Diego` / `[09:13] Larissa` |
| F12 | Retry — tentativas | **5 retentativas**, além da entrega inicial. Ver nota de desambiguação abaixo | `[09:15] Diego` / `[09:17] Larissa` |
| F13 | Retry — backoff | **1m / 5m / 30m / 2h / 12h** — cinco intervalos, somando **14h36min** entre a primeira falha e a última tentativa | `[09:17] Diego` |

> **Desambiguação obrigatória de `F12` e `F13`.** A reunião diz "5 tentativas" (`[09:15] Diego`, `[09:17] Larissa`,
> `[09:48] Larissa`), mas define **cinco intervalos** de backoff. As duas coisas só fecham se os cinco intervalos
> forem retentativas *posteriores* à entrega inicial — leitura confirmada pelo próprio Diego em `[09:17]`, que
> descreve "quase 15 horas entre primeira falha e última tentativa": `1m + 5m + 30m + 2h + 12h = 14h36min`, contado
> a partir da **primeira falha**, não da primeira chamada.
>
> **Número canônico: 5 retentativas · 5 intervalos · 6 chamadas HTTP no pior caso.**
>
> Onde qualquer documento deste pacote disser "5 tentativas", leia "5 retentativas". A aritmética completa vive
> em [ADR-003](../adrs/ADR-003-retry-com-backoff-e-dlq.md) e não deve ser reescrita por extenso em nenhum outro
> documento — os demais referenciam. Este é o ponto de contradição numérica mais comum neste desafio.
| F14 | DLQ | Tabela **separada** `webhook_dead_letter` com payload, motivo da falha e timestamp | `[09:18] Diego` |
| F15 | Replay de DLQ | `POST /admin/webhooks/dead-letter/:id/replay` — recoloca na outbox como pendente | `[09:18] Diego` / `[09:35] Diego` |
| F16 | Replay — autorização | Role **ADMIN** obrigatória, reusando o `requireRole` existente | `[09:36] Sofia` / `[09:36] Larissa` |
| F17 | Replay — auditoria | Endpoint tem que logar quem fez o replay | `[09:36] Sofia` |
| F18 | Assinatura | **HMAC-SHA256** sobre o corpo do request | `[09:20] Sofia` / `[09:22] Sofia` |
| F19 | Escopo do secret | **Uma secret única por endpoint**, não global da plataforma | `[09:21] Sofia` |
| F20 | Rotação de secret | Endpoint para o cliente pedir nova secret; antiga válida em paralelo por **24h**, depois morre | `[09:21] Sofia` |
| F21 | Origem do secret | Gerada por nós e devolvida na criação do webhook | `[09:31] Marcos` |
| F22 | TLS | URL do webhook tem que ser `https`; `http` é recusado com erro de validação (schema Zod) | `[09:23] Sofia` |
| F23 | Limite de payload | **64KB**; ultrapassou, **erra** (não trunca) | `[09:23] Sofia` / `[09:24] Diego` / `[09:24] Larissa` |
| F24 | Garantia de entrega | **at-least-once** (cliente pode receber duplicado) | `[09:24] Diego` |
| F25 | Idempotência | Header `X-Event-Id` com **UUID gerado quando o evento entra na outbox**; dedup do lado do cliente | `[09:25] Diego` |
| F26 | Headers do envio | `X-Event-Id`, `X-Signature`, `X-Timestamp`, `Content-Type: application/json` | `[09:44] Diego` |
| F27 | Header adicional | `X-Webhook-Id` com o id do endpoint cadastrado | `[09:44] Sofia` / `[09:45] Diego` |
| F28 | Timeout HTTP | **10 segundos**; sem resposta ⇒ falha ⇒ retry | `[09:42] Diego` |
| F29 | Payload do evento | JSON com `event_id`, `event_type` (`"order.status_changed"`), `timestamp` ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id`, `total_cents`. **Sem `items`** | `[09:43] Diego` |
| F30 | Snapshot | Payload é **renderizado na inserção** na outbox, não no envio | `[09:52] Larissa` / `[09:52] Diego` / `[09:52] Bruno` |
| F31 | Tipo de id | **UUID**, seguindo o padrão do projeto | `[09:51] Larissa` |
| F32 | Estrutura do módulo | `src/modules/webhooks` com controller, service, repository, routes e schemas | `[09:27] Bruno` |
| F33 | Arquivo do worker | Lógica de processamento no módulo: `webhook.worker.ts` ou `webhook.processor.ts` | `[09:28] Bruno` |
| F34 | Códigos de erro | Prefixo **`WEBHOOK_`** em tudo do módulo | `[09:28] Bruno` / `[09:29] Larissa` |
| F35 | Exemplos de código | `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED` | `[09:28] Bruno` |
| F36 | Reuso | `AppError`, Pino, error middleware centralizado, padrão de módulos, padrão de schemas Zod | `[09:29] Bruno` / `[09:30] Larissa` |
| F37 | Gancho no order.service | Função `publishWebhookEvent(tx, order, fromStatus, toStatus)` recebendo o `tx` da transação atual — função pura, sem injetar repository inteiro | `[09:41] Bruno` / `[09:41] Diego` |
| F38 | Atomicidade do gancho | Se o insert na outbox falhar, **rollback** da mudança de status | `[09:40] Bruno` / `[09:41] Diego` |
| F39 | Tabela de configuração | `url` + `secret` + `customer_id` + estado ativo | `[09:21] Bruno` |
| F40 | Filtro de eventos | Lista de status que o endpoint quer ouvir; filtro aplicado **na inserção** da outbox, não no envio | `[09:33] Marcos` / `[09:34] Bruno` / `[09:34] Diego` |
| F41 | CRUD de configuração | `POST` (cria), `PATCH` (edita), `DELETE` (remove), `GET` (lista por customer) | `[09:31] Marcos` / `[09:33] Bruno` |
| F42 | Origem do customer_id | **Body ou path — não vem do JWT** (o JWT é do usuário operador) | `[09:32] Larissa` |
| F43 | Histórico de entregas | `GET /webhooks/:id/deliveries` — últimos 100: sucesso/falha, payload, response, tempo de resposta | `[09:34] Marcos` |
| F44 | Autorização do CRUD | Qualquer role autenticada, **por enquanto** | `[09:36] Marcos` / `[09:37] Sofia` |
| F45 | Direção | **Outbound apenas** — só sai da gente pra eles | `[09:02] Marcos` / `[09:03] Sofia` |
| F46 | Prazo | **3 sprints**, incluindo a revisão de segurança no fim | `[09:46] Larissa` |
| F47 | Prazo de negócio | Atlas quer para **fim de novembro** | `[09:45] Marcos` |
| F48 | Revisão de segurança | Reservar **2 dias úteis** para a Sofia revisar HMAC e geração de secret antes do deploy | `[09:46] Sofia` |
| F49 | Clientes demandantes | Atlas Comercial, MaxDistribuição, Nova Cargo (3 clientes B2B) | `[09:00] Marcos` |
| F50 | Dor atual | Polling do cliente no `GET /orders`, integração lenta e cara; risco de churn da Atlas para o concorrente | `[09:00] Marcos` |

## B. Fora de escopo / adiado (→ PRD "Fora de escopo")

| # | Item | Decisão | Localização |
|---|---|---|---|
| X01 | Email de alerta ao cliente após falhas | "Email tá fora de escopo dessa fase. Talvez próxima fase, depois que a gente medir o impacto." | `[09:37] Larissa` |
| X02 | Dashboard / painel visual para o cliente | "Não, agora não. Só endpoints. Painel é projeto separado do time de frontend." | `[09:40] Larissa` |
| X03 | Webhooks inbound (cliente → nós) | "Só saindo da gente pra eles." | `[09:02] Marcos` |
| X04 | Arquivamento de linhas entregues (~30 dias) | "fora do escopo dessa feature" | `[09:08] Diego` |

## C. Questões em aberto (→ RFC "Questões em aberto")

| # | Questão | Estado | Localização |
|---|---|---|---|
| Q01 | Rate limiting de envio para o cliente (ex: 50 pedidos mudando em 1 minuto) | "observar e decidir depois"; registrado como ponto em aberto | `[09:38] Diego` / `[09:39] Larissa` |
| Q02 | Escala para múltiplos workers e ordering global | "problema do futuro"; opções levantadas: particionar por `order_id` ou lock pessimista | `[09:13] Diego` / `[09:13] Larissa` |
| Q03 | `customer_id` no body ou no path | Larissa fecha que não vem do JWT, mas **não define qual dos dois** | `[09:32] Larissa` |
| Q04 | Endurecer autorização do CRUD de configuração | "Por enquanto sim. Mais pra frente a gente pode endurecer." | `[09:37] Sofia` |

## D. Alternativas descartadas (→ RFC "Alternativas" + ADRs)

| # | Alternativa | Trade-off que motivou o descarte | Localização |
|---|---|---|---|
| A01 | Disparo HTTP síncrono dentro do `changeStatus` | Transação já é pesada; cliente lento trava mudança de status de outros pedidos; cliente fora do ar exigiria rollback da mudança de status | `[09:04] Bruno` / `[09:06] Diego` |
| A02 | Redis Streams / fila dedicada | Exigiria subir mais infra; time pequeno; "overengineering" — outbox no MySQL existente resolve | `[09:07] Larissa` / `[09:07] Diego` |
| A03 | Trigger de banco para notificar o worker | MySQL não tem `NOTIFY/LISTEN`; trigger só executa SQL, não notifica processo externo; improvisos ficam esquisitos | `[09:09] Bruno` / `[09:09] Diego` |
| A04 | DLQ como flag `failed` na própria outbox | Tabela separada deixa a leitura da outbox mais limpa e serve de evidência para debug e reprocessamento | `[09:17] Larissa` / `[09:18] Diego` |
| A05 | 3 tentativas de retry | Pouco: retentaria 3× em 30 minutos e mataria; já houve cliente com indisponibilidade de 2h em manutenção planejada | `[09:16] Bruno` / `[09:16] Diego` |
| A06 | Retry indefinido com backoff | Evento fica pendurado para sempre se o cliente sumiu | `[09:15] Diego` |
| A07 | Exactly-once | Exigiria coordenação dos dois lados, muito mais complexo; at-least-once + `event_id` resolve 99% dos casos (padrão Stripe/GitHub) | `[09:25] Diego` |
| A08 | Secret global da plataforma | "se vaza uma, vaza tudo" | `[09:21] Sofia` |
| A09 | Truncar payload acima do limite | Sofia é a favor de errar: "Se chegou nesse tamanho, tem algo errado" | `[09:23] Sofia` |
| A10 | Renderizar payload na hora do envio | Se o pedido mudar depois, o evento não refletiria o estado de quando o status mudou | `[09:52] Larissa` |
| A11 | Id auto incremental na outbox | UUID segue o padrão do resto do projeto | `[09:51] Larissa` |
| A12 | Filtrar eventos na hora do envio | Filtrar na inserção economiza linha na tabela | `[09:34] Bruno` / `[09:34] Diego` |
| A13 | Injetar o repository de webhook inteiro no OrderService | Função pura recebendo o `tx` é suficiente | `[09:41] Bruno` / `[09:41] Diego` |

## E. Fatos verificados no CÓDIGO (fonte: CODIGO)

| # | Fato | Evidência |
|---|---|---|
| C01 | Rotas montadas sob **`/api/v1`** ⇒ paths reais são `/api/v1/webhooks/...` e `/api/v1/admin/webhooks/...` | `src/app.ts:67`, `src/routes/index.ts:21-31` |
| C02 | `AppError` expõe `statusCode`, **`errorCode`** (não `code`), `details` | `src/shared/errors/app-error.ts:3-15` |
| C03 | Envelope de erro: `{ "error": { "code", "message", "details?" } }`; `details` omitido se `undefined` | `src/middlewares/error.middleware.ts:15-24` |
| C04 | Envelope de lista: `{ "data": [...], "pagination": { page, pageSize, total, totalPages } }`; recurso único é serializado cru | `src/shared/http/response.ts:8-11`, `src/modules/customers/customer.controller.ts:22` |
| C05 | `requireRole(...roles)` existe e funciona; hoje usado só em `GET /users/:id` | `src/middlewares/auth.middleware.ts:49-61`, `src/modules/users/user.routes.ts:15` |
| C06 | Roles existentes: exatamente **`ADMIN`** e **`OPERATOR`** | `prisma/schema.prisma:11-14`, `src/middlewares/auth.middleware.ts:6-10` |
| C07 | Padrão de PK: `String @id @default(uuid()) @db.Char(36)`; tabelas com `@@map("snake_case")`; colunas camelCase sem `@map` | `prisma/schema.prisma:25-138` |
| C08 | `changeStatus` roda tudo em `this.prisma.$transaction(async (tx) => {...})` | `src/modules/orders/order.service.ts:131-178` |
| C09 | Ponto de inserção do `publishWebhookEvent`: após `tx.orderStatusHistory.create(...)`, antes do re-fetch | `src/modules/orders/order.service.ts:159-169` |
| C10 | Estoque é debitado **só** em `PENDING → PAID` e reposto só em `PAID\|PROCESSING → CANCELLED` — não em toda mudança de status | `src/modules/orders/order.status.ts:29-37`, `order.service.ts:151-156` |
| C11 | Máquina de estados: 6 status, transições em `Readonly<Record<OrderStatus, ReadonlyArray<OrderStatus>>>` | `src/modules/orders/order.status.ts:3-10` |
| C12 | Pino configurado com `redact` — a lista **não cobre** `secret`/`signature` | `src/shared/logger/index.ts:4-11` |
| C13 | Correlation id existe: `req.id` via header `x-request-id`, devolvido em `X-Request-Id` | `src/middlewares/request-logger.middleware.ts:6-8` |
| C14 | **Nenhum cliente HTTP nas dependências** (sem axios/node-fetch/undici) ⇒ `fetch` nativo do Node 20 | `package.json:25-34`, `package.json:8` |
| C15 | `express.json({ limit: '1mb' })` — limite de entrada global | `src/app.ts:59` |
| C16 | Graceful shutdown existe mas **não aguarda** `server.close()` nem drena trabalho em background | `src/server.ts:13-21` |
| C17 | `tests/setup.ts` limpa tabelas em `beforeEach` na ordem de FK — tabelas novas precisam entrar ali | `tests/setup.ts:9-15` |
| C18 | Tipo `OrderWithRelations` é o payload natural do evento; `customer` já reduzido a `{id,name,email}` | `src/modules/orders/order.repository.ts:12-16` |
| C19 | Testes são de **integração real** contra MySQL: `fileParallelism: false`, `singleFork: true` | `vitest.config.ts:4-18` |
| C20 | `docker-compose.yml` sobe **só MySQL** — nenhuma fila, cache ou broker | `docker-compose.yml:2-24` |
| C21 | Módulo canônico = 5 arquivos: `*.schemas.ts`, `*.repository.ts`, `*.service.ts`, `*.controller.ts`, `*.routes.ts`; DI manual em `buildControllers` | `src/modules/customers/*`, `src/app.ts:26-53` |
| C22 | Padrão de erro específico a copiar: `InsufficientStockError extends UnprocessableEntityError` fixando código e `details`; precisa ser reexportado no barrel | `src/shared/errors/http-errors.ts:55-63`, `src/shared/errors/index.ts` |
| C23 | Env validada por Zod com `process.exit(1)` em falha; variáveis novas do worker entram aqui | `src/config/env.ts:4-27` |
| C24 | Não existe helper genérico de sucesso; só `paginated()` e `buildPagination()` | `src/shared/http/response.ts` |
| C25 | Zero código de webhook no repositório hoje — o vácuo é real | varredura em `src/` |

## F. Divergências entre transcrição e código (registrar, código prevalece)

| # | Fala | Realidade no código | Tratamento |
|---|---|---|---|
| D01 | `[09:04] Bruno`: a transação de mudança de status "decrementa `stock_quantity` dos produtos do pedido" | Verdadeiro só para `PENDING → PAID`; em `→ CANCELLED` vindo de `PAID`/`PROCESSING` o estoque é **reposto**; nas demais transições o estoque não é tocado | Documentar a versão precisa do código; a fala continua válida como motivação ("a transação já é pesada") |
| D02 | `[09:31] Marcos`: "Customer_id implícito do JWT" | JWT carrega `sub`/`email`/`role` do usuário operador — não há `customer_id` | Larissa corrige em `[09:32]`; vale como divergência resolvida na própria reunião |

---

## G. Não inventar (lista negra)

Nada disso tem origem na transcrição nem no código. **Não pode aparecer em nenhum documento:**

- SLA/uptime numérico (99,9% etc.)
- Volume de eventos por dia/mês, número de pedidos, número de clientes além dos 3 nomeados
- Custo, orçamento, headcount
- Ferramenta de observabilidade específica (Datadog, Grafana, Prometheus, OpenTelemetry) — a reunião nunca citou nenhuma
- Circuit breaker, rate limiting de saída como requisito (é questão em aberto, Q01)
- Fila externa, Kafka, RabbitMQ, SQS
- Nomes de pessoas fora dos 5 participantes
- Datas absolutas para a reunião (a transcrição diz apenas "quinta-feira, 09:00")

---

## H. Decisões complementares (resolvidas com o Marcus, fora da reunião)

A reunião fechou o **quê**; estes itens fecham o **como**, para o FDD ficar acionável.
Nenhum deles pode ser apresentado como decisão da reunião. Cada um carrega sua procedência:

- `MERCADO` — padrão de indústria verificado (Stripe, GitHub, Standard Webhooks). Citar a referência no FDD.
- `CODIGO` — derivado de um padrão que já existe no repositório. Vai ao Tracker com fonte `CODIGO`.
- `PRODUTO` — chamada do Marcus. Registrar como "decisão de implementação" no FDD, nunca como fala da reunião.

### H.1 Assinatura

| # | Decisão | Valor | Procedência |
|---|---|---|---|
| H01 | Valor de `X-Signature` | `v1=<hex>`; múltiplas assinaturas separadas por vírgula durante a rotação | `MERCADO` Stripe / GitHub |
| H02 | String canônica assinada | **Somente o corpo cru.** Literal a `[09:22] Sofia` ("HMAC-SHA256 sobre o corpo do request"). O `X-Timestamp` **não** entra na assinatura | `[09:22]` — assinar `{timestamp}.{corpo}`, ao padrão Stripe, foi cogitado e descartado por extrapolar a fala. A consequência está em `Q07` |
| H03 | Formato de `X-Timestamp` | Unix epoch em segundos (payload segue ISO 8601 conforme `[09:43]`) | `MERCADO` Stripe / Slack |
| H04 | Tolerância de recência | 5 minutos, **recomendada ao cliente na doc** — não validamos, somos o emissor. **Limitação a declarar:** como `X-Timestamp` fica fora da assinatura (`H02`), ele é adulterável; a defesa efetiva contra reprocessamento é a dedup por `X-Event-Id` (`F25`). Ver `Q07` | `MERCADO` Stripe + limitação derivada de `H02` |
| H05 | Geração do secret | 32 bytes de `crypto.randomBytes` em hex, prefixo `whsec_` | `MERCADO` Stripe + `CODIGO` (sem dependência nova, `node:crypto`) |
| H06 | Secret at-rest | Texto claro + `redact` no Pino. Criptografia **não decidida** → `Q06`, dono Sofia, prazo revisão pré-deploy `[09:46]` | `PRODUTO` |
| H07 | Exposição do secret | Retornado íntegro só em `POST /webhooks` e `POST /webhooks/:id/rotate-secret`; mascarado (`whsec_****cd12`) em todo GET | `MERCADO` + `[09:31]` |

### H.2 Worker

| # | Decisão | Valor | Procedência |
|---|---|---|---|
| H08 | Tamanho do batch | 50, via env com default | `PRODUTO` — `[09:08]` diz só "batch pequeno" |
| H09 | Claim de linhas | `SELECT … FOR UPDATE SKIP LOCKED` | `MERCADO` + `CODIGO` (MySQL 8.0 no `docker-compose.yml`) |
| H10 | Lease de evento travado | `processingStartedAt` + reclaim após 60s (6× o timeout de 10s) | `PRODUTO` — buraco que a reunião não enxergou |
| H11 | Classe de falha | **Toda** resposta não-2xx e toda falha de rede consomem as 5 tentativas. Sem atalho para 4xx | `[09:15]` `[09:17]` — literal, sem extrapolação |
| H12 | Jitter no backoff | **Não**. Curva exata de `[09:17]`. Jitter fica como melhoria futura | `[09:17]` |
| H13 | Paralelismo do batch | Serial dentro de cada `order_id`, paralelo entre `order_id`s distintos, concorrência 10 | `[09:12]` (ordering) + `[09:04]` (cliente lento não trava os outros) |
| H14 | Redirects 3xx | Não seguir; conta como falha | `MERCADO` Stripe |

### H.3 Dados — 4 tabelas

| Tabela | Origem do nome | Papel |
|---|---|---|
| `webhook_endpoints` | `PRODUTO` (a reunião diz "endpoint" em `[09:21]`, `[09:22]`, `[09:44]`, mas nunca nomeia a tabela) | Configuração: `customerId`, `url`, `secret`, `previousSecret`, `previousSecretExpiresAt`, `subscribedStatuses` (Json), `active` |
| `webhook_outbox` | `[09:06] Diego` | Uma linha por **(evento × endpoint)**. Campos: `eventId`, `webhookEndpointId`, `orderId`, `eventType`, `payload` (Json, snapshot), `status`, `attemptCount`, `nextAttemptAt`, `processingStartedAt` |
| `webhook_deliveries` | `PRODUTO` (deriva do path `/deliveries` de `[09:34]`) | Uma linha por **tentativa**: `attemptNumber`, `responseStatus`, `responseBody`, `durationMs`, `errorMessage`, `attemptedAt` |
| `webhook_dead_letter` | `[09:18] Diego` | `eventId`, `webhookEndpointId`, `payload`, `failureReason`, `attemptCount`, `movedAt`, `replayedAt` |

| # | Decisão | Valor | Procedência |
|---|---|---|---|
| H15 | Fan-out | Uma mudança de status com N endpoints assinantes gera **N linhas na outbox, com N `eventId` distintos** | `PRODUTO` — coerente com `X-Webhook-Id` de `[09:44]` |
| H16 | `subscribedStatuses` | Coluna `Json` | `CODIGO` — `customers.address` já é `Json` (`prisma/schema.prisma:44`) |
| H17 | Retenção de deliveries | Guarda todas as tentativas; o endpoint devolve as 100 mais recentes. Expurgo fora de escopo, mesmo argumento de `[09:08]` | `[09:34]` + `PRODUTO` |
| H18 | Secret antigo na rotação | Colunas na própria linha — a janela de 24h garante no máximo 2 secrets vivos | `[09:21]` + `PRODUTO` |
| H19 | Padrão de PK e mapeamento | `String @id @default(uuid()) @db.Char(36)`, `@@map("snake_case")`, colunas camelCase sem `@map` | `CODIGO` `prisma/schema.prisma` + `[09:51]` |

### H.4 Endpoints

Todos sob `/api/v1` (`CODIGO` — `src/app.ts:67`).

| Método | Path | Auth | Origem |
|---|---|---|---|
| `POST` | `/api/v1/webhooks` — `customerId` no body | autenticado | `[09:31]` + `[09:32]` + `PRODUTO` |
| `GET` | `/api/v1/webhooks?customerId=…&page=&pageSize=` | autenticado | `[09:33]` + `CODIGO` (`listCustomersQuerySchema`) |
| `GET` | `/api/v1/webhooks/:id` | autenticado | `CODIGO` (padrão de módulo) |
| `PATCH` | `/api/v1/webhooks/:id` | autenticado | `[09:33]` |
| `DELETE` | `/api/v1/webhooks/:id` → 204, hard delete | autenticado | `[09:33]` + `CODIGO` (`customer.controller.ts:45`) |
| `POST` | `/api/v1/webhooks/:id/rotate-secret` | autenticado | `[09:21]` (pede o endpoint, não dá o path) |
| `GET` | `/api/v1/webhooks/:id/deliveries` | autenticado | `[09:34]` |
| `GET` | `/api/v1/admin/webhooks/dead-letter` | **ADMIN** | `PRODUTO` — sem isso o replay de `[09:35]` não é usável |
| `POST` | `/api/v1/admin/webhooks/dead-letter/:id/replay` | **ADMIN** | `[09:35]` + `[09:36]` |

| # | Decisão | Valor | Procedência |
|---|---|---|---|
| H20 | Desativar sem apagar | `PATCH { active: false }`; `DELETE` é hard delete | `[09:21]` ("estado ativo") + `CODIGO` |
| H21 | Envelope de listagem | `{ data, pagination }`, `pageSize` default 20, teto 100 | `CODIGO` `src/shared/http/response.ts:8` |
| H22 | Envelope de erro | `{ error: { code, message, details? } }` — nada a mudar no middleware | `CODIGO` `src/middlewares/error.middleware.ts:15-24` |

### H.5 Operação

| # | Decisão | Valor | Procedência |
|---|---|---|---|
| H23 | Constantes congeladas | `POLL_INTERVAL_MS=2000`, `HTTP_TIMEOUT_MS=10000`, `MAX_ATTEMPTS=5`, `BACKOFF_MS=[60s,300s,1800s,7200s,43200s]`, `MAX_PAYLOAD_BYTES=65536`, `SECRET_GRACE_MS=86400000` — em `webhook.constants.ts`, cada uma com o ADR e o timestamp em comentário | `PRODUTO` — mudar exige PR e reabrir o ADR |
| H24 | Env de tuning | `WEBHOOK_BATCH_SIZE=50`, `WEBHOOK_CONCURRENCY=10`, `WEBHOOK_LEASE_TIMEOUT_MS=60000`, `WEBHOOK_WORKER_ENABLED=true` — validadas por Zod | `CODIGO` `src/config/env.ts:4-27` |
| H25 | Observabilidade sem vendor | Eventos Pino estruturados: `webhook_event_enqueued`, `webhook_delivery_attempted`, `webhook_delivery_succeeded`, `webhook_delivery_failed`, `webhook_moved_to_dlq`, `webhook_dlq_replayed`. Métricas derivadas desses logs. **Nenhuma ferramenta nomeada** | `CODIGO` `src/shared/logger/index.ts` |
| H26 | Tracing | Propagar o `requestId` da requisição de origem até a entrega, via coluna na outbox | `CODIGO` `src/middlewares/request-logger.middleware.ts:6-8` |
| H27 | Redação de segredos no log | Adicionar `*.secret`, `*.previousSecret`, `*.signature` ao `redact` | `CODIGO` `src/shared/logger/index.ts:4-11` (hoje não cobre) |
| H28 | Cliente HTTP | `fetch` nativo do Node 20 com `AbortSignal.timeout(10_000)`. Nenhuma dependência nova | `CODIGO` `package.json` (sem axios/undici) + `[09:42]` |
| H29 | Drenagem no shutdown | Worker precisa de hook de drenagem — o `shutdown` atual não aguarda `server.close()` | `CODIGO` `src/server.ts:13-21` |
| H30 | Limpeza nos testes | As 4 tabelas novas entram no `beforeEach`, respeitando ordem de FK | `CODIGO` `tests/setup.ts:9-15` |

### H.6 Questões em aberto — lista final para o RFC

| # | Questão | Dono | Fonte |
|---|---|---|---|
| Q01 | Rate limiting de envio para o cliente | Diego | `[09:38]` `[09:39]` |
| Q02 | Escala multi-worker e ordering global | Diego | `[09:13]` |
| Q03 | ~~`customer_id` no body ou no path~~ → **resolvida**: body no POST, query no GET | — | `[09:32]` + H.4 |
| Q04 | Endurecer autorização do CRUD de configuração | Sofia | `[09:37]` |
| Q05 | Arquivamento de linhas entregues e retenção de deliveries | Diego | `[09:08]` |
| Q06 | Secret at-rest: claro ou cifrado | Sofia | `[09:46]` |
| Q07 | Escopo da assinatura: `X-Timestamp` fica fora do HMAC (`H02`), logo é adulterável. A detecção de replay pedida em `[09:44]` recai sobre a dedup por `X-Event-Id`. Incluir o timestamp na string assinada, ao padrão Stripe, resolveria — mas extrapola `[09:22]` | Sofia | `[09:22]` + `[09:44]` + `[09:46]` |

---

## I. Não inventar (lista negra atualizada)

Nada disso tem origem na transcrição, no código ou nas decisões da seção H:

- SLA/uptime numérico (99,9% etc.)
- Volume de eventos por dia/mês, número de pedidos, número de clientes além dos 3 nomeados
- Custo, orçamento, headcount
- Ferramenta de observabilidade específica (Datadog, Grafana, Prometheus, OpenTelemetry)
- Circuit breaker; rate limiting de saída **como requisito** (é `Q01`)
- Fila externa, Kafka, RabbitMQ, SQS, Redis **como parte da solução** (Redis só aparece como alternativa descartada `A02`)
- Nomes de pessoas fora dos 5 participantes
- Datas absolutas para a reunião
- `event_type` além de `order.status_changed` — `[09:43]` cita só esse
- Alerta por email, dashboard, webhook inbound, auto-desativação de endpoint com falha — todos descartados em `X01`–`X04`
