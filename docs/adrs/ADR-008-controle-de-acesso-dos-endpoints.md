# ADR-008 — Replay de dead letter restrito a ADMIN e CRUD de configuração apenas autenticado

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:48] Larissa` (resumo) · `[09:49]` Diego, Bruno, Marcos e Sofia confirmam    |
| **Relacionados** | [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) · [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) |

## Status

Aceito, confirmado no resumo de `[09:48] Larissa` e ratificado em `[09:49]` por Diego, Bruno e Sofia.

## Contexto

A feature abre duas superfícies HTTP com perfis de risco bem diferentes, e a reunião tratou cada uma no seu
lugar.

A primeira é o CRUD de configuração de webhook: cadastrar, listar, editar e remover endpoints de um cliente,
além de rotacionar a secret. A segunda é o replay de dead letter, `POST /admin/webhooks/dead-letter/:id/replay`
(`[09:35] Diego`), que reinjeta na outbox um evento já dado como perdido. São operações de natureza distinta —
uma configura, a outra mexe em fila de entrega já processada — e a reunião não as tratou com o mesmo peso.

Larissa levantou a pergunta assim que o endpoint de replay foi enunciado: *"Quem é admin? Tem que ser role
ADMIN do JWT?"* (`[09:35] Larissa`). Sofia respondeu com o critério: *"Tem que ser ADMIN sim. Mexer em fila de
entrega de notificação não é coisa de operador. E o endpoint de admin tem que logar quem fez o replay, pra
auditoria."* (`[09:36] Sofia`).

Antes disso, o bloco de requisitos já tinha produzido uma correção relevante sobre **de onde vem a identidade
do cliente**. Marcos abriu afirmando que o `customer_id` seria implícito do JWT (`[09:31] Marcos`). Bruno
contestou na hora: *"Espera, mas o JWT atual é do usuário operador, não do cliente."* (`[09:32] Bruno`). E
Larissa fechou: *"Então é endpoint autenticado normal, e o customer_id é passado no body ou no path. Não vem do
JWT."* (`[09:32] Larissa`). É a única divergência da reunião que foi levantada e resolvida dentro da própria
call (`D02`).

O que decide a forma da solução é o que já existe no código. Não há middleware de autorização a construir.

## Decisão

O módulo reusa `authenticate` e `requireRole` sem introduzir mecanismo novo, conforme Larissa fechou em
`[09:36]`: *"Decidido, role ADMIN obrigatório no replay e a gente reaproveita o requireRole que já existe."*

| Superfície | Exigência | Origem |
|---|---|---|
| CRUD de configuração de webhook | Autenticação apenas — qualquer role válida | `[09:36] Marcos` · `[09:37] Sofia` |
| Histórico de entregas do endpoint | Autenticação apenas | mesma superfície do CRUD |
| Replay de dead letter | Autenticação **e** role `ADMIN` | `[09:36] Sofia` · `[09:36] Larissa` |
| Listagem da dead letter | Autenticação e role `ADMIN` — decisão derivada | ver abaixo |
| Origem do `customerId` | Body na criação, parâmetro de consulta na listagem. **Nunca do JWT** | `[09:32] Larissa` |
| Auditoria do replay | Registro de quem executou | `[09:36] Sofia` |

O mecanismo é o do projeto, sem adaptação:

```ts
// rotas de configuração — mesmo padrão de customer.routes.ts:14
router.use(authenticate);

// rotas administrativas — mesmo padrão de user.routes.ts:15
router.use(authenticate, requireRole('ADMIN'));
```

`requireRole` (`src/middlewares/auth.middleware.ts:49-61`) lança `UnauthorizedError` quando não há `req.user` e
`ForbiddenError('Insufficient permissions')` quando a role está fora da lista. Os dois já viram resposta HTTP
correta pelo middleware de erro, sem nenhuma alteração (`C03`, `C05`).

**Decisão derivada** — a reunião não enunciou o endpoint de listagem da dead letter; procedência: escolha de
quem produziu a documentação. O replay é por identificador (`[09:35] Diego`), mas nada na reunião diz como um
administrador descobre esses identificadores. Sem uma listagem, o endpoint de replay é inoperável na prática.
Ela herda a mesma exigência de `ADMIN`, pelo mesmo critério da Sofia: quem não pode reprocessar a fila também
não precisa enxergá-la.

### Auditoria

O `requestLogger` já registra `userId` em toda requisição concluída
(`src/middlewares/request-logger.middleware.ts:12-25`), o que atende parcialmente o pedido de `[09:36] Sofia`.
Parcialmente porque esse log é genérico de HTTP e se perde no volume. O replay emite, além dele, um evento
próprio identificando quem executou, qual entrada da dead letter foi reprocessada e qual identificador de
evento voltou para a outbox — correlacionado pelo `requestId` que o mesmo middleware já propaga (`C13`).

### O que este ADR não cobre

- Política de retry, dead letter e o efeito do replay sobre o contador de tentativas → [ADR-003](ADR-003-retry-com-backoff-e-dlq.md).
- Reuso de `AppError`, Pino e do padrão de módulos → [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md).
- Caminhos completos, verbos, payloads e códigos de erro → `docs/FDD.md`.
- Autenticação do **cliente que recebe** o webhook, que é assinatura HMAC e não role → [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md).

## Alternativas Consideradas

### Exigir ADMIN também no CRUD de configuração

|                      |                                    |
| -------------------- | ---------------------------------- |
| **Proponente**       | Marcos — levantou como pergunta    |
| **Quem derrubou**    | Sofia, adiando                     |
| **Citação**          | > "Por enquanto sim. Mais pra frente a gente pode endurecer." — `[09:37] Sofia` |
| **Trade-off aceito** | Qualquer usuário autenticado, inclusive `OPERATOR`, pode cadastrar, alterar e remover webhooks de qualquer cliente |

A pergunta de Marcos foi direta — *"O resto do CRUD de configuração de webhook pode ser qualquer role
autenticada?"* (`[09:36] Marcos`) — e a resposta de Sofia não é um "não": é um adiamento explícito. Ela aceita
a permissividade agora e reserva o direito de endurecer depois.

Não é uma alternativa recusada, é uma alternativa **adiada**, e a diferença importa: o gatilho de reabertura já
está declarado pela própria autora da decisão. O conteúdo dela é `Q04`.

### Derivar o cliente do token em vez de recebê-lo na requisição

|                      |                                    |
| -------------------- | ---------------------------------- |
| **Proponente**       | Marcos                             |
| **Quem derrubou**    | Bruno levantou o problema, Larissa fechou |
| **Citação**          | > "Espera, mas o JWT atual é do usuário operador, não do cliente." — `[09:32] Bruno` |
| **Trade-off aceito** | O cliente vira parâmetro de entrada, então nada no protocolo impede um usuário de operar sobre um cliente que não é o dele |

O token emitido em `src/modules/auth/auth.service.ts:47-50` carrega `sub`, `email` e `role`. Não há `customerId`
nem qualquer campo equivalente, e o tipo `AuthUser` (`src/middlewares/auth.middleware.ts:6-10`) confirma:
`id`, `email` e `role`, nada mais.

Vale registrar uma imprecisão da própria transcrição. Ao defender a posição original, Marcos afirmou em
`[09:32]`: *"Pela nossa API direto, autenticado com JWT do nosso sistema. A gente tem usuários que representam
o cliente."* O schema não sustenta isso. O model `User` (`prisma/schema.prisma:25-38`) não tem coluna de
vínculo com `Customer`, e o model `Customer` (`prisma/schema.prisma:40-54`) não referencia usuários. O que
existe entre eles é indireto e vai pelo pedido: `Order.createdById` aponta para `User` e `Order.customerId`
aponta para `Customer` (`prisma/schema.prisma:87-90`). "Usuários que representam o cliente" é convenção
operacional, não modelo de dados — e é por isso que a correção de Bruno e o fechamento de Larissa estão certos.

### Criar uma role de cliente (análise deste documento — não levantada na reunião)

Ninguém propôs. O contraste é registrado porque é a resposta estrutural para o problema que `Q04` descreve.
Ela é recusada por uma restrição que a reunião de fato impôs: o enum `UserRole`
(`prisma/schema.prisma:11-14`) tem exatamente `ADMIN` e `OPERATOR`, e Larissa fechou em `[09:30]` que o
módulo reaproveita os padrões existentes, sem stack nem conceito novo. Acrescentar uma terceira role — e o
vínculo usuário↔cliente que a tornaria útil — é mudança no modelo de identidade da aplicação inteira, não
decisão de uma feature de notificação. Cabe em `Q04`, não aqui.

## Consequências

### Positivas

- Zero código de autorização novo. `requireRole` sai de um único uso em todo o projeto
  (`src/modules/users/user.routes.ts:15`) para dois, o que valida o middleware em vez de duplicá-lo.
- A operação destrutiva do módulo — reinjetar evento na fila — é a única atrás de `ADMIN`, e a superfície de
  uso diário fica desimpedida, que é o que o Marcos precisava para os clientes integrarem sem fricção.
- `UnauthorizedError` e `ForbiddenError` já produzem o envelope de erro padrão da API, então os códigos
  `WEBHOOK_*` não precisam cobrir falha de autenticação nem de autorização.
- A auditoria do replay reaproveita o `requestId` que já circula, sem instrumentação nova.

### Negativas

- **Qualquer operador autenticado enxerga e altera os webhooks de qualquer cliente.** Não existe vínculo entre
  usuário e cliente no modelo de dados, e o `customerId` chega como parâmetro. Um `OPERATOR` pode listar os
  endpoints da Atlas passando o identificador dela, e pode apagá-los. É consequência direta de `[09:37] Sofia`
  combinada com o schema atual, e não há mitigação possível dentro do escopo desta feature.
- **A secret aparece na resposta de criação para qualquer usuário autenticado.** Como a secret é gerada por nós
  e devolvida na criação (`[09:31] Marcos`), quem consegue criar um webhook para um cliente consegue uma
  credencial válida de assinatura para ele.
- **A rotação de secret também é operação sensível e não está atrás de `ADMIN`.** A reunião só classificou o
  replay; a rotação nem foi mencionada nesse bloco. Fica sob o mesmo regime permissivo do resto do CRUD.
- **O log genérico do `requestLogger` não substitui trilha de auditoria.** Ele registra `userId`, método e
  caminho, mas não sobrevive a rotação de log nem responde "quem reprocessou este evento" sem varredura.
- **`ForbiddenError` não aceita código customizado** (`src/shared/errors/http-errors.ts:21-25`), então uma
  tentativa de replay por `OPERATOR` responde `FORBIDDEN`, não um código `WEBHOOK_*`. É coerente com o resto da
  API e vale registrar para que ninguém procure esse código na matriz de erros.

### Limitações conhecidas

| Limitação | Gatilho de reabertura |
| --------- | --------------------- |
| Não há verificação de posse: qualquer autenticado opera sobre qualquer `customerId` | Quando o primeiro cliente externo receber credencial de acesso direto à API, ou ao primeiro incidente de acesso cruzado (`Q04`, dono Sofia) |
| Rotação de secret sob o mesmo regime permissivo do CRUD | Junto com `Q04`, ou na revisão de segurança de `[09:46] Sofia` |
| Trilha de auditoria do replay vive em log de aplicação, não em armazenamento durável | Quando auditoria de acesso virar requisito de conformidade |
| Só existem `ADMIN` e `OPERATOR`; não há papel de cliente | Quando o painel do cliente sair do papel — hoje fora de escopo por `[09:40] Larissa` |
| `requireRole` é aplicado por rota, sem teste que garanta a cobertura | Quando uma rota administrativa nova for adicionada ao módulo |

## Referências

- Transcrição: `[09:31] Marcos` · `[09:32] Bruno` · `[09:32] Marcos` · `[09:32] Larissa` · `[09:33] Bruno` · `[09:34] Marcos` · `[09:35] Larissa` · `[09:35] Diego` · `[09:36] Sofia` · `[09:36] Larissa` · `[09:36] Marcos` · `[09:37] Sofia` · `[09:40] Larissa` · `[09:46] Sofia` · `[09:48] Larissa`
- Código: `src/middlewares/auth.middleware.ts:6-10` · `src/middlewares/auth.middleware.ts:49-61` · `src/middlewares/request-logger.middleware.ts:12-25` · `src/modules/users/user.routes.ts:15` · `src/modules/customers/customer.routes.ts:14` · `src/modules/auth/auth.service.ts:47-50` · `src/shared/errors/http-errors.ts:21-25` · `prisma/schema.prisma:11-14` · `prisma/schema.prisma:25-38` · `prisma/schema.prisma:40-54` · `prisma/schema.prisma:87-90`
- Contrato de fatos: `F16` `F17` `F41` `F42` `F44` · `C03` `C05` `C06` `C13` · `D02` · `Q03` `Q04`
