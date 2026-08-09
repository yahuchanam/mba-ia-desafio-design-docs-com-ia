# ADR-001 — Padrão outbox no MySQL para publicação de eventos de pedido

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:48] Larissa` (resumo) · `[09:49]` Diego, Bruno e Sofia confirmam           |
| **Relacionados** | [ADR-002](ADR-002-worker-de-entrega-em-polling.md) · [ADR-003](ADR-003-retry-com-backoff-e-dead-letter.md) · [ADR-007](ADR-007-payload-e-headers-do-evento.md) |

## Contexto

A reunião abriu pela pergunta mais estrutural da feature, formulada por Larissa logo no começo:
> "Acho que a primeira pergunta é: a gente dispara isso sincronamente no service de orders quando o status muda, ou faz algum tipo de fila/outbox?" — `[09:03] Larissa`

A resposta condiciona todo o resto — worker, retry, assinatura e payload só fazem sentido depois que se define **onde o evento nasce** e **com qual garantia**. Este ADR fecha essa pergunta.

A restrição mais dura veio do código. A mudança de status roda inteira dentro de uma transação Prisma (`src/modules/orders/order.service.ts:131-178`): valida a transição, mexe em estoque quando a transição pede, atualiza a `order` e insere em `orderStatusHistory` (linhas 159-167). Bruno descreveu a transação como pesada em `[09:04]` e citou o débito de estoque como parte fixa dela; o código é mais restrito — estoque só é debitado em `PENDING → PAID` e reposto em `PAID|PROCESSING → CANCELLED` (`src/modules/orders/order.status.ts:29-37`, `order.service.ts:151-156`). A imprecisão não derruba o argumento: mesmo sem tocar estoque há duas escritas e uma leitura sob a mesma transação, e uma chamada HTTP a terceiros ali dentro amarraria o commit à disponibilidade do cliente.

A segunda restrição é de infraestrutura. O `docker-compose.yml:2-24` sobe um único serviço, MySQL 8.0 — não existe broker, fila ou cache no ambiente. Qualquer desenho baseado em fila externa começaria pedindo infra nova, e Diego cortou esse caminho em `[09:07]`.

Fechada a escolha pela outbox, sobrou uma pergunta derivada, levantada por Bruno no fim da reunião: o que fazer se a escrita do evento falhar depois que o status já foi atualizado na mesma transação. `[09:40] Bruno` e `[09:41] Diego` responderam junto, e a resposta virou parte da decisão.

## Decisão

Quando o status de um pedido muda, uma linha de evento é inserida na tabela `webhook_outbox` **dentro da mesma transação SQL** que atualiza a `order` e o histórico. A publicação não é uma chamada de rede: é um `INSERT` local, no MySQL que já existe.

> "quando o status do pedido muda, dentro da mesma transação SQL que atualiza orders e order_status_history, a gente também insere uma linha numa tabela tipo webhook_outbox com o evento" — `[09:06] Diego`

O ponto de inserção é logo após `tx.orderStatusHistory.create(...)` e antes do re-fetch que monta o retorno (`src/modules/orders/order.service.ts:159-169`). Bruno propôs o gancho como função pura recebendo o `tx` da transação corrente: *"Vou propor uma função publishWebhookEvent(tx, order, fromStatus, toStatus) que aceita o tx client da transação atual."* — `[09:41] Bruno`.

```ts
await tx.orderStatusHistory.create({ /* ... */ });        // order.service.ts:159-167
await publishWebhookEvent(tx, order, from, to);           // novo — mesma transação
const refreshed = await tx.order.findUnique({ /* ... */ }); // order.service.ts:169
```

Semântica garantida:

| Situação                                            | Resultado                                                       |
| --------------------------------------------------- | --------------------------------------------------------------- |
| Transação commita                                   | Evento registrado como pendente; a entrega é questão de tempo     |
| Transação dá rollback (transição inválida, estoque) | A linha da outbox desaparece junto — não existe evento fantasma   |
| `INSERT` na outbox falha                            | **Rollback da mudança de status**; a requisição devolve erro      |

> "Garante que se a transação principal commitou, o evento foi registrado, e se ela deu rollback, o evento some junto. Não tem inconsistência possível." — `[09:06] Diego`

Estados da linha na outbox, conforme `[09:08] Diego`:

| Estado        | Significado                                                        | Quem escreve   |
| ------------- | ------------------------------------------------------------------ | -------------- |
| `pendente`    | Recém-inserida pela transação de mudança de status; ainda não saiu   | `changeStatus` |
| `processando` | Reivindicada por um worker para tentativa de entrega                 | worker         |
| `falhou`      | Tentativa sem sucesso, aguardando a próxima janela                   | worker         |
| `entregue`    | Entrega concluída; a linha sai do conjunto lido                      | worker         |

A tabela leva índice no campo de status e em `created_at` — os dois juntos servem exatamente à consulta do worker, que lê **só os pendentes, em batch pequeno, e marca como entregue** (`[09:08] Diego`). A grafia final da coluna e o tipo do enum são detalhe de implementação, não desta decisão.

### O que este ADR não cobre

- **Como o worker consome a outbox** — intervalo de polling, isolamento em processo separado e claim de linhas: [ADR-002](ADR-002-worker-de-entrega-em-polling.md).
- **O que acontece quando a entrega falha** — tentativas, backoff e dead letter: [ADR-003](ADR-003-retry-com-backoff-e-dead-letter.md).
- **O conteúdo da linha** — campos do payload, headers e momento da renderização: [ADR-007](ADR-007-payload-e-headers-do-evento.md).
- **Expurgo de linhas antigas** — `[09:08] Diego` deixou explícito: *"Linhas entregues a gente arquiva depois de 30 dias ou assim, fora do escopo dessa feature."*

## Alternativas Consideradas

### Disparo HTTP síncrono dentro do `changeStatus`

|                      |                                                                                              |
| -------------------- | -------------------------------------------------------------------------------------------- |
| **Proponente**       | — (levantada como hipótese por Larissa em `[09:03]`)                                          |
| **Quem derrubou**    | Bruno `[09:04]`, Diego `[09:06]`                                                              |
| **Citação**          | > "Síncrono não rola. A transação de mudança de status hoje já é pesada — atualiza orders, insere na order_status_history, decrementa stock_quantity dos produtos do pedido. Se a gente acrescentar um HTTP call no meio disso, qualquer cliente lento vai travar mudança de status pra outros pedidos." — `[09:04] Bruno` |
| **Trade-off aceito** | A entrega deixa de ser imediata e passa a ter latência entre o commit e o envio; ganhamos um processo a mais para operar |

Bruno somou um segundo argumento que sela a alternativa: *"Sem falar que se o cliente tiver fora do ar, o que a gente faz, dá rollback na mudança de status? Não dá."* — `[09:04] Bruno`. Diego, ao entrar na call, foi direto: *"Síncrono está fora de questão."* — `[09:06] Diego`.

Vale notar a assimetria com a decisão tomada: a outbox **também** derruba a mudança de status quando a escrita do evento falha (`[09:40]`). A diferença é o que entra no escopo transacional — um `INSERT` no mesmo MySQL, cuja falha significa banco indisponível e transação perdida de qualquer forma, contra uma chamada de rede a terceiro, cuja indisponibilidade é rotina e não deveria bloquear a operação de negócio.

### Redis Streams ou fila dedicada

|                      |                                                                                              |
| -------------------- | -------------------------------------------------------------------------------------------- |
| **Proponente**       | Larissa, como contraste em `[09:07]`                                                          |
| **Quem derrubou**    | Diego `[09:07]`, com a própria Larissa concordando                                            |
| **Citação**          | > "Exato, e a gente é um time pequeno. Subir Redis Cluster pra isso é overengineering. Outbox no MySQL existente resolve." — `[09:07] Diego` |
| **Trade-off aceito** | A outbox concorre pelo mesmo MySQL da carga transacional; sem broker, escalar consumo vira problema de SQL |

Larissa levantou a opção já reconhecendo o custo: *"A alternativa seria botar Redis Streams ou alguma coisa parecida, mas a gente acabaria precisando subir mais infra."* — `[09:07] Larissa`. O `docker-compose.yml:2-24` mostra o tamanho do salto: o ambiente inteiro é um container de MySQL, e adotar fila significaria operar e monitorar um componente novo em produção com um time pequeno.

O custo operacional não é o único problema. Publicar numa fila externa não acontece dentro da transação SQL — o `enqueue` não participa do commit. Isso recria a escrita dupla que a feature existe para evitar: ou o status muda e o `enqueue` falha, ou o `enqueue` sucede e a transação aborta. Diego resumiu em `[09:41]`: *"Essencial. Se ficar fora da transação, perde a garantia toda."* Uma fila só voltaria a fazer sentido **depois** da outbox, como transporte alimentado por ela — desenho diferente do proposto aqui.

## Consequências

### Positivas

- Atomicidade real entre o fato de negócio e o evento. Não existe estado em que o pedido mudou de status e o cliente jamais saberá, nem o inverso — evento entregue de uma transição que deu rollback.
- Nenhuma infraestrutura nova. O ambiente continua com um serviço só (`docker-compose.yml:2-24`) e o worker fala com o mesmo banco.
- `changeStatus` deixa de depender da disponibilidade de terceiros: o acréscimo é um `INSERT` local, não um round-trip de rede.
- Diagnóstico trivial — a fila é uma tabela SQL. Investigar um evento perdido é um `SELECT` por status, sem ferramenta externa.

### Negativas

- A mudança de status ganha um modo de falha que não tinha. Erro de escrita na outbox derruba uma operação de negócio legítima. É a troca deliberada de `[09:40]`: consistência acima de disponibilidade no caminho de escrita.
- Quatro tabelas passam a estar sob o mesmo escopo de lock nas transições que mexem em estoque (`order.status.ts:29-37`), alargando a janela de contenção na transação mais quente do sistema.
- O módulo de pedidos passa a conhecer o de webhooks — dependência mínima, por ser função pura sobre o `tx`, mas real.
- A entrega nunca é instantânea: sempre há defasagem entre o commit e a saída do evento, custo de tirar o HTTP da transação.
- A tabela cresce indefinidamente. Arquivamento ficou fora de escopo em `[09:08]`, e os índices seguram a leitura, não o tamanho.

### Limitações conhecidas

| Limitação                                                                                              | Gatilho de reabertura                                                                                    |
| ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| Falha de `INSERT` na outbox derruba a mudança de status, mesmo quando o pedido em si estava correto      | Quando erros de mudança de status atribuíveis à outbox aparecerem em produção — reabrir para discutir degradar a garantia nas transições que ninguém assina |
| Outbox e carga transacional dividem o mesmo MySQL; picos de evento competem com escrita de pedidos       | Quando a contenção aparecer nas transições de status — reabrir `A02` (fila dedicada) com dado de contenção medido |
| Sem expurgo, a tabela só cresce; a leitura por status degrada quando o volume de entregues domina        | `Q05`, dono Diego — reabrir quando o tempo da consulta de pendentes subir de forma consistente             |
| Sem broker, distribuir o consumo entre vários workers exige coordenação feita em SQL                     | `Q02`, dono Diego — reabrir quando um worker sozinho deixar de vencer o ritmo de inserção                  |

## Referências

- Transcrição: `[09:03] Larissa` · `[09:04] Bruno` · `[09:06] Diego` · `[09:07] Larissa` · `[09:07] Diego` · `[09:08] Diego` · `[09:08] Larissa` · `[09:40] Bruno` · `[09:41] Bruno` · `[09:41] Diego` · `[09:48] Larissa`
- Código: `src/modules/orders/order.service.ts:131-178` · `src/modules/orders/order.service.ts:151-156` · `src/modules/orders/order.service.ts:159-169` · `src/modules/orders/order.status.ts:29-37` · `docker-compose.yml:2-24`
- Contrato de fatos: `F01` `F02` `F03` `F04` `F05` `F38` · `A01` `A02` · `C08` `C09` `C10` `C20` · `X04` · `Q02` `Q05`
