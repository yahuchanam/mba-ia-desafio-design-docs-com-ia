# ADR-007 — Payload materializado como snapshot na inserção e filtragem de destinatários na origem

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:52] Bruno`: *"Beleza, snapshot. Decidido."*                                |
| **Relacionados** | [ADR-001](ADR-001-outbox-transacional-no-mysql.md) · [ADR-002](ADR-002-worker-separado-em-polling.md) · [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md) |

## Status

Aceito. O snapshot não entrou no resumo de `[09:48]` — a pergunta de Bruno só veio depois que Marcos e
Sofia já tinham saído da call. A decisão fechou em `[09:52] Bruno`: *"Beleza, snapshot. Decidido."*

## Contexto

Duas perguntas ficaram para o fim da call, e ambas decidem o mesmo ponto: **o que exatamente é gravado na
linha da outbox no instante em que o status do pedido muda.**

1. A linha guarda o payload já renderizado, ou guarda apenas `order_id` e o worker monta o JSON na hora do envio?
2. O filtro de status assinados é aplicado quando a linha é inserida, ou quando a entrega é feita?

A primeira só foi levantada depois que Marcos e Sofia já tinham saído. Bruno emendou: *"Eu também tenho uma
dúvida última: o evento da outbox guarda o payload renderizado já, ou guarda só order_id e renderiza na hora do
envio?"* — `[09:51] Bruno`. É uma pergunta de modelagem que parece detalhe e não é: entre a inserção e a
entrega passam-se, no melhor caso, 2 segundos; no pior, quase 15 horas, se a entrega percorrer toda a
progressão de retry de [ADR-003](ADR-003-retry-com-backoff-e-dlq.md). Nesse intervalo o pedido continua vivo e
pode mudar de novo.

A segunda foi decidida mais cedo, no bloco de requisitos. Marcos descreveu o filtro como recurso de produto —
*"Filtro de eventos é uma lista dos status que o webhook quer ouvir. Tipo 'só quero saber quando vira SHIPPED e
DELIVERED', e a gente filtra na hora de inserir na outbox."* (`[09:33] Marcos`) — e Diego questionou o momento
da aplicação: *"Filtra na inserção do outbox ou na hora de mandar?"* (`[09:34] Diego`).

As duas se encontram no mesmo lugar do código: a função que roda dentro da transação de `changeStatus`
(`src/modules/orders/order.service.ts:131-178`). Decidir "renderizado" e "filtrado na inserção" significa que
essa função consulta os endpoints do cliente, monta o JSON e grava tudo antes do commit.

## Decisão

**O payload é renderizado e congelado no momento da inserção na outbox.** A linha carrega o JSON completo do
evento, não uma referência ao pedido.

**O filtro de status assinados é aplicado na inserção.** Se nenhum endpoint ativo do cliente escuta aquela
transição, nenhuma linha é criada.

| Aspecto | Decisão | Origem |
|---|---|---|
| Momento da renderização | Inserção, dentro da transação | `[09:52] Larissa` · `[09:52] Diego` |
| Conteúdo persistido | JSON completo do evento (`F29`) | `[09:43] Diego` |
| Campos deliberadamente ausentes | `items` do pedido | `[09:43] Diego` |
| Momento da filtragem | Inserção (`F40`) | `[09:34] Bruno` · `[09:34] Diego` |
| Critério do filtro | Lista de status assinados pelo endpoint, mais o vínculo com o `customerId` do pedido | `[09:33] Marcos` |
| Cardinalidade | Uma linha por endpoint assinante, com identificador de evento próprio (`H15`) | derivada — ver abaixo |

A composição do payload foi enumerada por Diego em `[09:43]`: identificador do evento, tipo do evento
(`order.status_changed`), timestamp ISO 8601, identificador e número do pedido, status de origem e de destino,
identificador do cliente e os campos básicos do pedido. E a exclusão foi explícita: *"Não manda items pra não
inflar. Se o cliente quiser detalhes, ele bate no GET /orders/:id depois."* Bruno fechou em `[09:44]`:
*"Bom, mantém payload enxuto."*

O tipo `OrderWithRelations` (`src/modules/orders/order.repository.ts:12-16`), que o `changeStatus` já devolve,
é o material de origem natural do snapshot — inclusive porque o `customer` ali já vem reduzido a `id`, `name`
e `email` pelo `include` de `order.service.ts:169-176`. O evento carrega **menos** que isso, não mais.

**Decisão derivada** — a reunião não tratou a cardinalidade quando um cliente tem mais de um endpoint
cadastrado; procedência: escolha de quem produziu a documentação. Cada endpoint assinante recebe sua própria
linha na outbox, com seu próprio identificador de evento (`H15`). É o que torna coerente o `X-Webhook-Id` que
Sofia pediu em `[09:44]`, *"pra cliente que tem vários conseguir saber qual cadastro caiu naquele envio"*, e o
que permite que estado de entrega — contador de tentativas, próxima tentativa, destino final — seja por
destinatário, e não compartilhado.

### O que este ADR não cobre

- Atomicidade do insert e rollback da mudança de status → [ADR-001](ADR-001-outbox-transacional-no-mysql.md).
- Como o worker seleciona e processa as linhas → [ADR-002](ADR-002-worker-separado-em-polling.md).
- Identificador de evento como chave de deduplicação do cliente → [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md).
- JSON de exemplo, nomes exatos de campo e schema das tabelas → `docs/FDD.md`.

## Alternativas Consideradas

### Renderizar o payload na hora do envio

|                      |                                    |
| -------------------- | ---------------------------------- |
| **Proponente**       | Bruno — apresentou as duas opções na mesma pergunta |
| **Quem derrubou**    | Larissa, com concordância de Diego |
| **Citação**          | > "Eu prefiro renderizado já, na hora da inserção. Se o pedido mudar depois, o evento ainda reflete o estado de quando o status mudou. Senão tem caso esquisito." — `[09:52] Larissa` |
| **Trade-off aceito** | O payload passa a ser dado duplicado: o mesmo conteúdo existe na outbox e nas tabelas de pedido, e divergirá delas por construção |

O "caso esquisito" que a Larissa evita é concreto. Um pedido que vai para `PAID` e, dez minutos depois, para
`PROCESSING`, geraria dois eventos. Se o payload fosse montado no envio e a primeira entrega estivesse em
retry, o cliente receberia um evento anunciando a transição para `PAID` **contendo o estado atual do pedido**,
que já é `PROCESSING`. O evento passaria a mentir sobre o instante que ele mesmo descreve — e quanto mais longa
a janela de retry, mais grave, até o limite de quase 15 horas de [ADR-003](ADR-003-retry-com-backoff-e-dlq.md).

Havia um argumento a favor de renderizar no envio: economia de espaço e um único lugar de verdade. Ele perde
porque um evento é um **fato histórico**, não uma projeção do presente. O que o cliente precisa saber é o que
aconteceu, com os dados de quando aconteceu.

### Filtrar os destinatários na hora de mandar

|                      |                                    |
| -------------------- | ---------------------------------- |
| **Proponente**       | Diego — levantou como pergunta aberta |
| **Quem derrubou**    | Bruno, com concordância de Diego   |
| **Citação**          | > "Na inserção. Se nenhum webhook do customer quer aquele status, nem insere. Economiza linha na tabela." — `[09:34] Bruno` |
| **Trade-off aceito** | O conjunto de destinatários fica congelado no instante da mudança de status: quem cadastrar um endpoint depois não recebe eventos anteriores, e quem alterar os status assinados não afeta linhas já gravadas |

Filtrar no envio manteria a outbox como registro de tudo que aconteceu, independente de quem estava ouvindo, e
permitiria que uma mudança de assinatura alcançasse eventos ainda não entregues. O custo é escrever linhas que
podem nunca ter destinatário, e resolver a consulta de endpoints a cada ciclo do worker em vez de uma vez por
transação.

O argumento do Bruno é de volume de escrita, e vale mais do que parece: a inserção acontece **dentro** da
transação de `changeStatus`, que já atualiza o pedido, insere no histórico e, na transição `PENDING → PAID`,
mexe no estoque (`src/modules/orders/order.status.ts:29-31`). Cada linha desnecessária ali é peso na
transação mais quente do sistema.

### Guardar o payload comprimido (análise deste documento — não levantada na reunião)

Nenhum participante propôs compressão. O contraste é registrado porque o snapshot cria duplicação de dado, e
compressão seria a resposta óbvia. Ela é recusada por uma restrição que a reunião de fato impôs: o teto de
64 KB por evento (`[09:24] Larissa`) e a exclusão dos `items` (`[09:43] Diego`) já mantêm o payload pequeno.
Comprimir acrescentaria custo de CPU no caminho da transação para economizar bytes que ninguém demonstrou estar
faltando.

## Consequências

### Positivas

- O evento entregue descreve fielmente o instante em que o status mudou, mesmo que a entrega só aconteça horas
  depois. É o que dá sentido à progressão longa de retry.
- O worker não precisa consultar `orders`, `order_items`, `customers` nem `products` para montar o envio: ele lê
  a linha da outbox e envia. O processo de entrega fica com **uma** dependência de leitura, o que simplifica o
  laço e reduz a superfície de falha fora da transação.
- Reprocessar uma linha da dead letter reenvia exatamente o mesmo conteúdo que foi tentado da primeira vez, sem
  risco de o replay entregar um evento diferente do original.
- Linhas que ninguém iria receber não são criadas, o que mantém a tabela mais enxuta e a transação de
  `changeStatus` mais leve.

### Negativas

- **O payload é dado duplicado e vai divergir do pedido.** É intencional, mas significa que a outbox não pode
  ser usada como fonte de verdade sobre o estado atual de nada.
- **Corrigir um evento mal formado exige tocar em linhas persistidas.** Se o formato do payload mudar ou sair
  com defeito, as linhas já gravadas carregam o defeito até serem entregues ou expiradas. Não há como consertar
  alterando apenas o código do worker.
- **A transação de `changeStatus` ganha uma consulta a mais.** Para filtrar na inserção é preciso ler os
  endpoints ativos do cliente dentro da transação, no caminho mais quente do sistema.
- **Assinar um endpoint não alcança o passado.** Um cliente que cadastre um webhook logo depois de uma mudança
  de status não recebe aquele evento, e nem tem como saber que o perdeu.
- **Alterar os status assinados não afeta linhas pendentes.** Um cliente que remova `SHIPPED` da lista pode
  continuar recebendo eventos de `SHIPPED` que já estavam na outbox, inclusive horas depois.
- **O evento é enxuto por decisão, o que empurra trabalho para o cliente.** Quem precisar dos itens do pedido
  terá de fazer uma chamada de volta, como Diego previu em `[09:43]`.

### Limitações conhecidas

| Limitação | Gatilho de reabertura |
| --------- | --------------------- |
| Payload persistido com formato defeituoso não é corrigível sem alterar linhas gravadas | Quando a primeira mudança incompatível de formato de payload for necessária, definir versionamento do evento antes de aplicá-la |
| Assinatura de endpoint não retroage a eventos já ocorridos | Quando um cliente pedir reprocessamento de histórico ao integrar |
| Alteração de status assinados não afeta linhas pendentes na outbox | Quando um cliente reclamar de receber um status que removeu da lista |
| A consulta de endpoints ativos roda dentro da transação de `changeStatus` | Quando a latência de `PATCH /orders/:id/status` regredir de forma mensurável |
| Duplicação do payload entre outbox e dead letter cresce com o volume | Junto com a política de arquivamento adiada em `[09:08] Diego` (`Q05`) |

## Referências

- Transcrição: `[09:08] Diego` · `[09:24] Larissa` · `[09:33] Marcos` · `[09:34] Diego` · `[09:34] Bruno` · `[09:43] Diego` · `[09:44] Bruno` · `[09:44] Sofia` · `[09:51] Bruno` · `[09:52] Larissa` · `[09:52] Diego` · `[09:52] Bruno`
- Código: `src/modules/orders/order.service.ts:131-178` · `src/modules/orders/order.service.ts:169-176` · `src/modules/orders/order.repository.ts:12-16` · `src/modules/orders/order.status.ts:29-31`
- Contrato de fatos: `F29` `F30` `F40` · `A10` `A12` · `H15` `H16` · `C18` · `Q05`
