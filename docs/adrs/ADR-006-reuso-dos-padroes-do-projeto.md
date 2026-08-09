# ADR-006 — Reuso dos padrões existentes do projeto no módulo de webhooks

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:48] Larissa` (resumo) · `[09:49]` Diego, Bruno e Sofia confirmam           |
| **Relacionados** | [ADR-001](ADR-001-outbox-transacional-no-mysql.md) · [ADR-002](ADR-002-worker-separado-em-polling.md) · [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md) · [ADR-008](ADR-008-controle-de-acesso-dos-endpoints.md) |

## Contexto

A feature de webhooks acrescenta o primeiro módulo novo desde que a codebase se estabilizou, e acrescenta também o primeiro processo Node que não é a API. Isso abriu três perguntas encadeadas no bloco de estrutura de código: **(1)** o módulo segue o formato dos módulos existentes ou ganha desenho próprio? **(2)** onde mora a lógica do worker, já que ele não é um controller? **(3)** como o `order.service` chama o módulo de webhooks sem que os dois times fiquem amarrados um no outro? A terceira depende da primeira: se o módulo é convencional, o ponto de entrada dele para dentro da transação de pedidos precisa ser explicitado, porque a convenção do projeto injeta dependências por construtor e o gancho não cabe nesse formato.

Bruno abriu o bloco com o inventário do que já existe: *"A gente tem um padrão claro na codebase. Cada domínio é um módulo em src/modules com controller, service, repository, routes e schemas. Webhook vai seguir igual. Vou propor uma pasta src/modules/webhooks com toda a estrutura. Faz sentido?"* — `[09:27] Bruno`. Diego respondeu *"Faz."* — `[09:28] Diego`. A verificação no repositório confirma o padrão: `src/modules/customers/` tem exatamente cinco arquivos e a montagem é DI manual em `buildControllers` (`src/app.ts:26-53`), sem container nem framework de injeção.

O peso da decisão está menos no que se ganha e mais no que se evita gastar. O middleware de erro já resolve `AppError`, Zod e Prisma (`src/middlewares/error.middleware.ts:14-54`), e Bruno registrou a consequência direta disso: *"E o logger, que é Pino, já tá no projeto inteiro. Não vamos botar nada novo. O middleware de erro centralizado já trata AppError, Zod e Prisma. Vai pegar nossos erros sem precisar mudar nada."* — `[09:29] Bruno`. Larissa fechou o bloco em `[09:30]`: *"Decisão: reuso máximo do que já existe. AppError, Pino, error middleware, padrão de módulos, padrão de schemas Zod, padrão de códigos de erro. Webhook fica como módulo igual aos outros."*

A leitura do código, porém, mostra que "reuso máximo" não é reuso total. Duas restrições reais aparecem só quando se abre `src/shared/errors/http-errors.ts`, e estão registradas abaixo como sub-decisão. O gancho no `order.service` também precisou de forma própria, porque a transação de `changeStatus` (`src/modules/orders/order.service.ts:131-178`) entrega um `tx` que nenhum repository instanciado com o `PrismaClient` raiz consegue enxergar.

## Decisão

O módulo `src/modules/webhooks` herda os padrões existentes do projeto, um a um, sem exceção não documentada.

| Padrão existente | Onde vive (caminho:linha) | Como o módulo de webhooks herda | Custo de divergir |
| --- | --- | --- | --- |
| Módulo canônico de 5 arquivos | `src/modules/customers/customer.{schemas,repository,service,controller,routes}.ts` | `webhook.schemas.ts`, `webhook.repository.ts`, `webhook.service.ts`, `webhook.controller.ts`, `webhook.routes.ts` (`F32`, `C21`) | Revisor perde o mapa mental; o módulo novo vira o único que precisa ser explicado |
| Arquivo do worker dentro do módulo | — (não existe hoje) | `webhook.worker.ts` no mesmo diretório; a entry point `src/worker.ts` só o invoca (`F33`) | Lógica de entrega fora do módulo que a possui, ou dentro da entry point, que vira arquivo gordo |
| `AppError` com `statusCode`/`errorCode`/`details` | `src/shared/errors/app-error.ts:3-15` | Todos os erros do módulo descendem dela (`F36`, `C02`) | Erro não reconhecido pelo middleware cai em 500 genérico |
| Subclasses HTTP e erro específico de domínio | `src/shared/errors/http-errors.ts:3-63`, barrel em `index.ts:1-13` | Classes `Webhook*Error` no molde de `InsufficientStockError` (`C22`), reexportadas no barrel | Código de erro solto em string no service, sem tipo e sem ponto único de mudança |
| Error middleware centralizado | `src/middlewares/error.middleware.ts:14-24` | Zero alteração: o middleware já mapeia `errorCode` → chave `code` do JSON (`C03`) | Envelope de erro divergente entre `/webhooks` e o resto da API |
| Schemas Zod + `validate()` | `src/modules/customers/customer.schemas.ts:1-33`, `src/middlewares/validate.middleware.ts:11-36` | Validação de `url` (`https` obrigatório), paginação e params na mesma forma; `ZodError` vira `VALIDATION_ERROR` sozinho | Validação manual no controller e mensagem de erro em formato próprio |
| Logger Pino com `redact` | `src/shared/logger/index.ts:4-32` | Mesmo logger, sem dependência nova (`F36`); a lista de `redact` precisa crescer — ver [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md) | Segundo formato de log estruturado no mesmo serviço |
| Envelope de resposta | `src/shared/http/response.ts:8-24`, `src/modules/customers/customer.controller.ts:18-25` | Listagens usam `paginated()`; recurso único é serializado cru (`C04`, `C24`) | Cliente precisa de dois parsers para a mesma API |
| Convenções do Prisma | `prisma/schema.prisma:25-54` | `String @id @default(uuid()) @db.Char(36)`, `@@map("snake_case")`, colunas camelCase sem `@map` (`C07`, `H19`) | Migração fora do padrão, e `[09:51] Larissa` (*"UUID, segue o padrão do resto do projeto. Tudo é uuid."*) contrariada |
| DI manual em `buildControllers` | `src/app.ts:26-53`, montagem de rotas em `src/routes/index.ts:21-31` | Mais uma tríade repository → service → controller, mais um `router.use('/webhooks', …)` | Container de DI introduzido para um módulo só |
| `authenticate` / `requireRole` | `src/middlewares/auth.middleware.ts:27-61`, uso em `src/modules/users/user.routes.ts:12-18` | Reuso direto, sem middleware novo (`C05`, `C06`) — a política de acesso é de [ADR-008](ADR-008-controle-de-acesso-dos-endpoints.md) | Segunda implementação de autorização para manter em dia |
| Sem cliente HTTP nas dependências | `package.json:25-34` | `fetch` nativo do Node 20 no worker (`C14`) | Dependência nova para fazer o que a plataforma já faz |
| Limpeza de tabelas nos testes | `tests/setup.ts:8-16` | As tabelas novas entram no `beforeEach`, respeitando ordem de FK (`C17`) | Teste vazando estado entre arquivos, com `fileParallelism: false` mascarando a causa |

### Gancho no `order.service`

A integração com a transação de pedidos é uma **função**, não uma dependência de construtor:

```ts
publishWebhookEvent(tx, order, fromStatus, toStatus)
```

Ela recebe o `tx` da transação em curso e é chamada dentro de `this.prisma.$transaction(...)` (`F37`, `C08`, `C09`). O `OrderService` continua sendo construído com `(orderRepository, prisma)` em `src/app.ts:43` — nenhum argumento novo.

### Sub-decisão: como os erros `WEBHOOK_*` são construídos

`[09:29] Larissa`: *"Prefixo WEBHOOK_ pra tudo do módulo."* O prefixo vale (`F34`), mas o código impõe uma restrição que a reunião não enxergou: **a propriedade pública de `AppError` chama-se `errorCode`, não `code`** (`src/shared/errors/app-error.ts:5`); quem mapeia para a chave `code` do JSON é o middleware (`src/middlewares/error.middleware.ts:18`). E metade das subclasses não aceita código customizado.

| Subclasse | Aceita código no construtor? | Linha | Consequência para os códigos `WEBHOOK_*` |
| --- | --- | --- | --- |
| `BadRequestError` | sim (`code = 'BAD_REQUEST'`) | `http-errors.ts:3-7` | Serve direto para 400 |
| `ConflictError` | sim (`code = 'CONFLICT'`) | `http-errors.ts:33-37` | Serve direto para 409 |
| `UnprocessableEntityError` | sim (`code = 'UNPROCESSABLE_ENTITY'`) | `http-errors.ts:39-43` | Serve direto para 422 |
| `UnauthorizedError` | **não** — fixa `UNAUTHORIZED` | `http-errors.ts:15-19` | Inutilizável para um código `WEBHOOK_*` |
| `ForbiddenError` | **não** — fixa `FORBIDDEN` | `http-errors.ts:21-25` | Inutilizável para um código `WEBHOOK_*` |
| `NotFoundError` | **não** — fixa `NOT_FOUND` | `http-errors.ts:27-31` | `new NotFoundError('Webhook')` devolveria `NOT_FOUND`, não `WEBHOOK_NOT_FOUND` |

Decisão derivada — a reunião não tratou este ponto; procedência: leitura do código. Erros de 400/409/422 estendem a subclasse correspondente, no molde de `InsufficientStockError` (`http-errors.ts:55-63`). `WEBHOOK_NOT_FOUND` estende `AppError` diretamente, porque `NotFoundError` não permite outro código:

```ts
export class WebhookNotFoundError extends AppError {
  constructor(id: string) {
    super('Webhook endpoint not found', 404, 'WEBHOOK_NOT_FOUND', { id });
  }
}
```

Toda classe nova precisa ser reexportada em `src/shared/errors/index.ts` para seguir a convenção do barrel (`C22`).

### Subtração: o código órfão `WEBHOOK_SECRET_REQUIRED`

Bruno nomeou três exemplos em `[09:28]`: *"Códigos tipo WEBHOOK_NOT_FOUND, WEBHOOK_INVALID_URL, WEBHOOK_SECRET_REQUIRED, etc."* (`F35`). Três minutos depois, `[09:31] Marcos` fecha a origem da secret — *"secret é gerada pela gente e devolvida na criação"* (`F21`) — e o terceiro código perde o caso de uso.

| Onde poderia ocorrer | Por que não ocorre | Quando voltaria a fazer sentido |
| --- | --- | --- |
| `POST /webhooks` sem `secret` no corpo | O campo não existe na entrada: nós geramos a secret (`F21`, `[09:31] Marcos`) | Se o cliente passar a poder trazer a própria secret |
| `PATCH /webhooks/:id` trocando a secret | Rotação tem endpoint próprio e também gera do nosso lado (`F20`, `[09:21] Sofia`) | Se a rotação aceitar secret fornecida pelo cliente |
| Worker tentando assinar sem secret | A coluna é obrigatória desde a criação da linha; ausência seria corrupção de dado, não erro de validação | Se a secret virar opcional para algum tipo de endpoint |

O código **não entra** na matriz de erros. O registro fica aqui para que ninguém tente reconstruí-lo lendo a transcrição isolada.

### O que este ADR não cobre

- Matriz completa dos códigos `WEBHOOK_*` com status HTTP e mensagens → `docs/FDD.md`.
- Schema das quatro tabelas novas e seus campos → `docs/FDD.md` (as convenções de PK e mapeamento estão aqui; as colunas, não).
- `PrismaClient` separado no processo do worker e ciclo de vida do worker → [ADR-002](ADR-002-worker-separado-em-polling.md).
- Redação de `secret`/`signature` no Pino e formato da assinatura → [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md).
- Quais rotas exigem `ADMIN` → [ADR-008](ADR-008-controle-de-acesso-dos-endpoints.md).
- Atomicidade do insert na outbox e rollback → [ADR-001](ADR-001-outbox-transacional-no-mysql.md).

## Alternativas Consideradas

### Injetar o repository de webhook inteiro no `OrderService`

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | Bruno — levantou as duas opções na mesma fala |
| **Quem derrubou**   | Diego                              |
| **Citação**         | > "Boa, função pura recebendo o tx. Não precisa injetar repository inteiro." — `[09:41] Diego` |
| **Trade-off aceito**| O acoplamento entre pedidos e webhooks vira um import estático, não uma dependência injetada: um teste do `OrderService` não consegue substituir a função por um dublê via construtor |

A fala de Bruno que abriu a alternativa foi *"Vai me obrigar a passar um repository do webhook pro OrderService ou uma função de 'enqueue event'. Vou propor uma função publishWebhookEvent(tx, order, fromStatus, toStatus) que aceita o tx client da transação atual. Aí o order.service chama isso."* — `[09:41] Bruno`. A recusa de Diego não é estética. Um repository instanciado em `buildControllers` carrega o `PrismaClient` raiz (`src/app.ts:34`, `:42`), e o `PrismaClient` raiz **não enxerga** a transação aberta em `order.service.ts:131`. Para participar da mesma transação, todo método desse repository teria de receber o `tx` mesmo assim — a injeção acrescentaria um quarto argumento ao construtor do `OrderService` sem eliminar a passagem do `tx`. A alternativa custa mais e entrega menos (`A13`).

O trade-off aceito é real e fica registrado: com import estático, testar o `changeStatus` isolado do módulo de webhooks exige mock de módulo em vez de troca de dependência. Como os testes do projeto são de integração real contra MySQL (`vitest.config.ts`, `C19`), o custo hoje é baixo — o teste vai exercitar a inserção de verdade.

### Nenhuma alternativa foi levantada para o restante do reuso

Não houve contraproposta ao formato de módulo, ao `AppError`, ao Pino, ao error middleware nem ao padrão de schemas Zod. Bruno perguntou explicitamente *"Faz sentido?"* em `[09:27]`, Diego respondeu *"Faz."* em `[09:28]`, e Larissa fechou em `[09:30]` sem que ninguém abrisse discussão. O critério usado foi o custo de manutenção de um segundo padrão dentro do mesmo serviço, com o time pequeno já reconhecido em `[09:07] Diego`. Registrar aqui uma alternativa sintética — container de DI, reestruturação hexagonal, framework novo — seria inventar tensão onde não houve.

## Consequências

### Positivas

- Nenhuma alteração no `error.middleware.ts`, no `validate.middleware.ts` nem no `response.ts`: o módulo novo entra sem tocar em infraestrutura compartilhada, o que reduz o raio de regressão da feature nos módulos existentes.
- Nenhuma dependência nova no `package.json` para o módulo em si: Pino, Zod e Prisma já estão lá, e o envio HTTP usa `fetch` nativo (`C14`).
- O envelope de erro `{ "error": { "code", "message", "details?" } }` e o de listagem `{ "data", "pagination" }` valem para `/api/v1/webhooks` sem trabalho adicional (`C03`, `C04`).
- Revisão de PR fica comparativa: o revisor abre `src/modules/customers/` ao lado e enxerga a diferença.

### Negativas

- **O reuso para no worker.** O `webhook.worker.ts` roda fora do pipeline do Express: não passa pelo `errorMiddleware` nem pelo `requestLogger`. Um `AppError` lançado lá não vira resposta HTTP nenhuma — precisa de tratamento próprio, com log Pino, dentro do laço de processamento. A frase *"vai pegar nossos erros sem precisar mudar nada"* (`[09:29] Bruno`) vale para o caminho HTTP, não para o worker.
- **`errorCode` é uma pegadinha herdada.** Quem escrever teste unitário assertando `err.code` recebe `undefined` sem erro de compilação, porque `AppError` não tem essa propriedade e a chave `code` só existe no JSON de saída.
- **Três subclasses não servem para códigos `WEBHOOK_*`**, o que obriga a estender `AppError` diretamente em pelo menos um caso (`WEBHOOK_NOT_FOUND`) e cria uma inconsistência interna no próprio módulo: nem todos os erros terão a mesma classe base.
- **`buildControllers` cresce mais uma vez.** É o sexto módulo montado à mão no mesmo arquivo; a função continua legível, mas o custo de cada novo módulo é linear e ninguém está pagando para reduzi-lo.
- **`tests/setup.ts` vira ponto de falha silenciosa.** As tabelas novas precisam entrar no `beforeEach` na ordem de FK; se forem esquecidas, o sintoma aparece como teste intermitente em outro módulo.
- Não existe helper genérico de sucesso (`C24`), então cada controller do módulo repete `res.status(...).json(...)` na mão, como os demais. Reuso do padrão inclui reusar sua ausência.

### Limitações conhecidas

| Limitação | Gatilho de reabertura |
| --------- | --------------------- |
| `publishWebhookEvent` acopla `order.service` ao módulo de webhooks por import estático | Quando um segundo produtor de eventos precisar do mesmo gancho, avaliar um publisher injetado por construtor |
| Erros do worker não têm caminho de saída padronizado — só log | Quando o worker precisar expor estado para fora (endpoint de health ou de status do worker), definir o formato ali |
| O prefixo `WEBHOOK_` é convenção verificada em revisão, não garantida por tipo | Quando um código sem prefixo passar por um code review, introduzir um union type de códigos do módulo |
| DI manual sem container | Quando `buildControllers` exigir ordenação não trivial entre módulos ou surgir dependência circular |
| `requireRole` só distingue `ADMIN` e `OPERATOR` (`C06`); não há noção de "usuário deste customer" | Quando a autorização do CRUD for endurecida (`Q04`, dono Sofia) |
| A escolha entre `webhook.worker.ts` e `webhook.processor.ts` ficou aberta em `[09:28]`; adotamos `webhook.worker.ts` por simetria com `src/worker.ts` — decisão derivada, procedência: escolha de quem produziu a documentação | Sem gatilho; renomear é refactor local, sem impacto de contrato |

## Referências

- Transcrição: `[09:07] Diego` · `[09:21] Sofia` · `[09:27] Bruno` · `[09:28] Diego` · `[09:28] Bruno` · `[09:29] Larissa` · `[09:29] Bruno` · `[09:30] Larissa` · `[09:31] Marcos` · `[09:41] Bruno` · `[09:41] Diego` · `[09:48] Larissa` · `[09:49]` Diego/Bruno/Sofia · `[09:51] Larissa`
- Código: `src/shared/errors/app-error.ts:3-15` · `src/shared/errors/http-errors.ts:3-63` · `src/shared/errors/index.ts:1-13` · `src/middlewares/error.middleware.ts:14-54` · `src/middlewares/validate.middleware.ts:11-36` · `src/middlewares/auth.middleware.ts:27-61` · `src/shared/logger/index.ts:4-32` · `src/shared/http/response.ts:8-24` · `src/app.ts:26-53` · `src/routes/index.ts:21-31` · `src/modules/customers/customer.schemas.ts:1-33` · `src/modules/customers/customer.controller.ts:18-25` · `src/modules/users/user.routes.ts:12-18` · `src/modules/orders/order.service.ts:131-178` · `prisma/schema.prisma:25-54` · `tests/setup.ts:8-16` · `package.json:25-34`
- Contrato de fatos: `F32` `F33` `F34` `F35` `F36` `F37` · `A13` · `C02` `C03` `C04` `C05` `C06` `C07` `C14` `C17` `C19` `C21` `C22` `C24` · `H19` · `Q04`
