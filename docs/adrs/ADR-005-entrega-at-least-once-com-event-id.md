# ADR-005 — Entrega at-least-once com idempotência delegada ao cliente via X-Event-Id

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:48] Larissa` (resumo) · `[09:49]` Diego, Bruno e Sofia confirmam           |
| **Relacionados** | [ADR-002](ADR-002-worker-dedicado-em-polling.md) · [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md) · [ADR-007](ADR-007-payload-snapshot-na-insercao.md) |

## Contexto

Quando Diego retomou o assunto da entrega em `[09:24]`, o desenho de transporte já estava fechado: outbox no MySQL gravada na mesma transação da mudança de status, worker separado em polling e retry com backoff até cinco tentativas. Sobravam três perguntas encadeadas, e a ordem entre elas é o que dá sentido a este ADR:

1. **Que garantia de entrega o sistema promete ao cliente?**
2. **Se a mesma notificação chegar duas vezes, como o cliente distingue a repetição de um evento novo?**
3. **De quem é a responsabilidade de deduplicar?**

A resposta de (1) cria o problema (2), e (2) só vale alguma coisa se (3) tiver dono explícito. Nenhuma das três dava para deixar em aberto: elas definem o contrato que o cliente escreve do lado dele.

A duplicidade não é hipótese teórica — ela nasce de um ponto concreto do desenho. O worker faz o `POST`, recebe `200`, e precisa de uma segunda escrita no banco para marcar a linha como entregue. Entre a resposta HTTP e esse `UPDATE` existe uma janela em que o processo pode morrer: `SIGTERM` durante o deploy, queda do host, erro de conexão com o MySQL. A reconciliação de lease descrita no [ADR-002](ADR-002-worker-dedicado-em-polling.md) devolve a linha travada para `pendente` — é exatamente isso que impede o evento de sumir, e é exatamente isso que o reenvia:

```
worker  → POST https://cliente/webhooks/pedidos          → 200 OK
        ✗ processo morre antes do UPDATE
        ✗ webhook_outbox.status continua 'processando'
lease   → reconciliação devolve a linha para 'pendente'
worker  → POST https://cliente/webhooks/pedidos          → 200 OK   (2ª entrega)
```

O código atual reforça o risco: o `shutdown` que existe hoje em `src/server.ts:13-21` não aguarda o `server.close()` nem drena trabalho em andamento, e o worker nasce ao lado dele como entry point irmã. A janela é real desde o primeiro deploy. Escolher "no more than once" significaria aceitar perder o evento nesse mesmo ponto — e o evento é o único aviso que o cliente recebe, porque a transação de `changeStatus` (`src/modules/orders/order.service.ts:131-178`) já commitou e não vai acontecer de novo.

Diego fechou a garantia em `[09:24]`: *"Voltando à entrega: a gente vai garantir at-least-once. Pode acontecer de o cliente receber o mesmo evento duas vezes. Ele tem que estar preparado."* Bruno puxou a pergunta seguinte — *"E como ele diferencia?"* — e a resposta introduziu o identificador estável do evento em `[09:25]`.

O ponto (3) foi o único momento de discordância explícita da reunião. Sofia reagiu direto: *"Isso joga responsabilidade pro cliente."* — `[09:25]`. A objeção não foi negada, foi aceita e respondida com argumento de mercado por Diego, e o custo foi absorvido por um compromisso de produto: Marcos assumiu em `[09:26]` documentar o comportamento no portal do desenvolvedor. Larissa fechou logo em seguida: *"At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão."*

## Decisão

O sistema entrega **at-least-once**. O cliente pode receber o mesmo evento mais de uma vez e o contrato publicado diz isso de forma explícita. Não existe supressão de duplicata do nosso lado.

Cada evento carrega um identificador estável no header **`X-Event-Id`**, um **UUID gerado no momento em que o evento entra na outbox** — não no momento do envio. Esse id é o mesmo nas cinco tentativas de retry, no registro de histórico de entregas e na linha de dead letter, e é a chave que o cliente usa para descartar repetições. O tipo é UUID, seguindo o padrão do projeto (`prisma/schema.prisma:26`: `String @id @default(uuid()) @db.Char(36)`).

Os headers da requisição de entrega, e o papel de cada um na idempotência:

| Header | Valor | Papel aqui |
| --- | --- | --- |
| `X-Event-Id` | UUID da linha da outbox | **Chave de deduplicação do cliente.** Estável entre tentativas |
| `X-Webhook-Id` | Id do endpoint cadastrado que recebeu este envio | Atribuição: cliente com vários cadastros sabe qual deles caiu |
| `X-Signature` | HMAC — ver [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md) | Autenticidade, não idempotência |
| `X-Timestamp` | Timestamp do envio | Detecção de replay pelo cliente, se ele quiser |
| `Content-Type` | `application/json` | — |

A deduplicação é **do cliente**. Não mantemos tabela de recebimentos confirmados, não consultamos o cliente antes de enviar e não tratamos um segundo `200` como erro.

**Decisão derivada — a reunião não tratou este ponto.** Uma mudança de status com N endpoints assinantes gera N linhas na outbox, com **N `X-Event-Id` distintos** — um por par (evento × endpoint), não um id compartilhado entre os N envios. A procedência é a escolha de quem produziu esta documentação, coerente com o `X-Webhook-Id` pedido em `[09:44]`: o header de atribuição só faz sentido se cada destino tiver sua própria linha e seu próprio ciclo de retry.

| Cenário | Linhas na outbox | `X-Event-Id` | Consequência para o cliente |
| --- | --- | --- | --- |
| 1 endpoint assinante | 1 | 1 | Dedup trivial por `X-Event-Id` |
| 3 endpoints, clientes diferentes | 3 | 3 distintos | Cada cliente vê um id só |
| 3 endpoints, **mesmo cliente** | 3 | 3 distintos | Recebe 3 entregas da mesma mudança de status; `X-Webhook-Id` diz qual é qual — dedup por `X-Event-Id` não colapsa as três indevidamente |

### O que este ADR não cobre

- **Assinatura, formato do `X-Signature`, secret por endpoint e rotação** → [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md).
- **Conteúdo do payload, campo `event_id` no corpo e snapshot na inserção** → [ADR-007](ADR-007-payload-snapshot-na-insercao.md). Aqui trata-se apenas do header.
- **Worker, claim de linhas, lease e reconciliação de evento travado** → [ADR-002](ADR-002-worker-dedicado-em-polling.md). Este ADR consome esse mecanismo, não o define.
- **Curva de retry, DLQ e replay administrativo** → ADR do retry e da dead letter.

## Alternativas Consideradas

### Exactly-once

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | — (levantada pelo próprio Diego como contraponto) |
| **Quem derrubou**   | Diego, sem objeção de Larissa, Bruno ou Marcos |
| **Citação**         | > "Joga, mas é o padrão de mercado. Stripe faz assim, GitHub faz assim. Garantir exactly-once exigiria coordenação dos dois lados e fica muito mais complexo. At-least-once com event_id resolve 99% dos casos." — `[09:25] Diego` |
| **Trade-off aceito**| Transferimos trabalho de integração para o cliente e não temos como verificar se ele o fez |

Exactly-once exige que emissor e receptor participem do mesmo protocolo: confirmação idempotente, janela de reconciliação combinada, e alguma forma de acordo sobre o que significa "recebido". Nada disso é negociável unilateralmente — teria de entrar no contrato de integração dos três clientes. O sistema que temos não sustenta a metade que caberia a nós: a confirmação da entrega é uma segunda escrita no MySQL que pode se perder junto com o processo, e é justamente esse buraco que a reconciliação de lease preenche reenviando.

A objeção de Sofia em `[09:25]` — *"Isso joga responsabilidade pro cliente."* — permanece registrada como tensão viva, não como detalhe resolvido. Ela não foi refutada tecnicamente; foi respondida com precedente de mercado e neutralizada por um compromisso de documentação, assumido por Marcos em `[09:26]`: *"Eu posso documentar isso bem destacado no portal de desenvolvedor pros clientes, sem problema."* Vale reter a forma dessa resolução: a decisão só se sustenta enquanto esse compromisso for cumprido. Sem a documentação, o que sobra é um comportamento surpreendente entregue sem aviso.

### Gerar o `X-Event-Id` no momento do envio (análise deste documento — não levantada na reunião)

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | —                                  |
| **Quem derrubou**   | Análise deste documento, contra a restrição de `[09:25]` e a curva de retry de `[09:17]` |
| **Citação**         | > "A gente manda um event_id no header, X-Event-Id, com um UUID gerado quando o evento entra na outbox. É único por evento. Se o cliente recebeu duas vezes, ele dedupica pelo event_id do lado dele." — `[09:25] Diego` |
| **Trade-off aceito**| Nenhum — a fala já fixa o momento da geração; a alternativa só existe como armadilha de implementação |

Gerar o id na hora do `POST` parece equivalente e é o erro fácil de cometer. Não é equivalente: a reunião fixou cinco retentativas espaçadas em `1m/5m/30m/2h/12h` (ver a aritmética em [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)), ou seja, o mesmo evento pode bater no cliente seis vezes ao longo de quase quinze horas. Se cada tentativa levar um id novo, o cliente não tem como agrupá-las e a dedup vira ruído — todas as repetições passam pelo filtro. O id precisa nascer com a linha da outbox e morrer com ela.

### Usar `(order_id, to_status)` como chave natural de idempotência (análise deste documento — não levantada na reunião)

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | —                                  |
| **Quem derrubou**   | Análise deste documento, contra `[09:44] Sofia` e o replay de DLQ de `[09:35]` |
| **Citação**         | > "Adiciona um X-Webhook-Id também, com o id do endpoint webhook, pra cliente que tem vários conseguir saber qual cadastro caiu naquele envio." — `[09:44] Sofia` |
| **Trade-off aceito**| Um campo a mais no header, em troca de uma chave que não colide |

Dispensaria o header e usaria dados que já vão no corpo. Quebra em dois casos que a própria reunião criou. O primeiro é o de Sofia: um cliente com vários cadastros recebe a mesma mudança de status uma vez por endpoint, e uma chave derivada do pedido faria ele descartar entregas legítimas de outro cadastro. O segundo é o replay manual de dead letter — recolocar o evento na outbox produziria de novo o mesmo par, sem nada que o distinga da entrega original.

## Consequências

### Positivas

- Nenhum acordo prévio com o cliente é necessário para o sistema funcionar. O worker pode morrer em qualquer ponto do envio sem perder evento, que é a propriedade que a outbox foi montada para dar.
- O `X-Event-Id` vira a chave única de rastreio ponta a ponta: mesma string na linha da outbox, no histórico de entregas, na dead letter e nos logs. "O cliente diz que não recebeu o evento X" passa a ser uma consulta, não uma investigação.
- Cliente que já integra Stripe ou GitHub reaproveita o código de dedup que ele tem. O precedente citado por Diego em `[09:25]` não é retórica: é redução real de esforço de integração.
- O replay administrativo de dead letter fica seguro por construção, porque reenvia com o mesmo id.

### Negativas

- **A responsabilidade saiu da nossa borda.** A objeção de Sofia continua verdadeira depois da decisão: o cliente que não implementar dedup vai processar o mesmo pedido duas vezes, e nós não temos como detectar nem impedir. O sintoma aparece no negócio dele.
- A decisão passa a depender de um artefato fora do código. Se o texto do portal do desenvolvedor não sair junto com a feature, entregamos um comportamento surpreendente sem aviso — e o compromisso de `[09:26]` é a única mitigação que existe.
- Cliente que dedupica de forma permanente por `X-Event-Id` **não reprocessa** um evento reenviado de propósito pelo replay de dead letter. O replay conserta a nossa ponta e pode não consertar a dele. *(Decisão derivada — a reunião não tratou este ponto: o replay preserva o `eventId` já gravado na linha de dead letter.)*
- O fan-out multiplica entregas para um mesmo cliente com vários cadastros. É correto e intencional, mas é contraintuitivo para quem lê "um evento, uma notificação" — precisa estar no mesmo texto do portal.
- Como o `X-Timestamp` fica fora da assinatura, a dedup por `X-Event-Id` acaba carregando também parte do papel de defesa contra reenvio adulterado. É acúmulo de função em um mecanismo que não foi desenhado para isso — o assunto pertence ao [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md).

### Limitações conhecidas

| Limitação | Gatilho de reabertura |
| --------- | --------------------- |
| Não existe teto para o número de repetições: cada reconciliação de lease pode gerar mais uma entrega | Quando o histórico de entregas mostrar um mesmo `eventId` com mais tentativas do que as 5 do retry, reabrir junto com o [ADR-002](ADR-002-worker-dedicado-em-polling.md) |
| Não medimos nem verificamos se o cliente dedupica de fato | Quando Atlas Comercial, MaxDistribuição ou Nova Cargo abrir chamado de processamento duplicado, reabrir a delegação e avaliar supressão do nosso lado |
| A decisão depende da documentação do portal do desenvolvedor, que é entregável de produto e não de engenharia | Se a revisão de segurança de fim de projeto começar sem o texto publicado, escalar para Marcos antes do deploy |
| Replay de dead letter reusa o `eventId` e pode ser descartado pelo cliente como duplicata | Quando um replay administrativo precisar ser efetivamente reprocessado do lado do cliente, reabrir para decidir um header que marque o reenvio |
| Um cliente com vários cadastros recebe N entregas da mesma mudança de status | Se um cliente pedir consolidação por evento em vez de por endpoint, reabrir a decisão derivada de fan-out |

## Referências

- Transcrição: `[09:24] Diego` · `[09:25] Bruno` · `[09:25] Diego` · `[09:25] Sofia` · `[09:26] Marcos` · `[09:26] Larissa` · `[09:44] Diego` · `[09:44] Sofia` · `[09:45] Diego` · `[09:48] Larissa` · `[09:49] Diego, Bruno, Sofia` · `[09:51] Larissa`
- Código: `prisma/schema.prisma:26` · `src/modules/orders/order.service.ts:131-178` · `src/modules/orders/order.service.ts:159-169` · `src/server.ts:13-21`
- Contrato de fatos: `F24` `F25` `F26` `F27` `F31` · `A07` · `H15` · apoio: `F13` `F15` `H10` `C16`
