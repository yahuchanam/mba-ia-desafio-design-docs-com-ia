# ADR-002 — Worker em processo separado consumindo a outbox por polling

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:48] Larissa` (resumo) · `[09:49]` Diego, Bruno e Sofia confirmam           |
| **Relacionados** | [ADR-001](ADR-001-outbox-transacional-no-mysql.md) · [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) · [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md) · [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) |

## Contexto

O padrão outbox fechado em `[09:06] Diego` ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)) resolve como o
evento nasce, não como ele sai. Sobrou a metade consumidora, que a reunião quebrou em três perguntas
encadeadas:

1. **Onde o consumo roda** — dentro da instância da API ou num processo próprio?
2. **Como esse processo descobre que existe evento novo** — perguntando ao banco de tempos em tempos, ou sendo
   avisado por ele?
3. **Que garantia de ordem** as duas respostas anteriores produzem?

As três dependem umas das outras. A resposta de 1 define quantos processos podem ler a mesma tabela ao mesmo
tempo, o que decide 3; e a resposta de 2 só é aceitável se o intervalo escolhido couber no orçamento de
latência que o Marcos trouxe do cliente em `[09:02]`: *"Pra eles, qualquer coisa abaixo de 10 segundos já é
"tempo real"."*

Duas restrições do repositório pesaram. A primeira é o ambiente: `docker-compose.yml:2-24` sobe **só MySQL** —
não existe broker, agendador ou supervisor de processos para reaproveitar, e a reunião já tinha recusado subir
infra nova em `[09:07]`. A segunda é a forma do processo atual: `src/server.ts` é a única entrada executável, e
o encerramento em `src/server.ts:13-21` chama `server.close()` sem aguardar o callback, indo direto para
`prisma.$disconnect()` e `process.exit(0)` — ou seja, o projeto hoje não tem nenhum mecanismo que drene
trabalho em andamento antes de sair.

Contra isso, a rotina que se procurava já estava descrita por Diego em `[09:08]`: *"Worker lê só os pendentes
em batch pequeno, processa, marca como entregue."* Faltava dizer em que processo esse loop vive, com que
frequência ele acorda e o que isso custa.

## Decisão

O consumo da outbox roda num processo Node próprio, que acorda a cada 2 segundos, lê os pendentes mais antigos
e envia.

| Dimensão | Decisão | Origem |
| --- | --- | --- |
| Topologia | Processo separado, fora da instância da API | `F08` · `[09:11] Diego` |
| Entry point | `src/worker.ts` **(arquivo novo)**, espelhando o `src/server.ts` que já existe | `F09` · `[09:11] Larissa` |
| Comando | `npm run worker`, novo script ao lado do par `dev`/`start` de `package.json:10-21` | `F09` |
| Modo de consumo | Polling em loop, buscando os eventos pendentes mais antigos | `F06` · `[09:09] Diego` |
| Intervalo | **2 segundos** | `F06` |
| Latência aceita | Até 2 s de espera antes do primeiro envio, contra o requisito de < 10 s | `F07` · `[09:10] Larissa` |
| Banco | Mesmo MySQL, mesma `DATABASE_URL` | `F10` · `[09:11] Diego` |
| Conexão | **PrismaClient próprio**, instanciado no processo do worker | `F10` · `[09:30] Bruno` |
| Instâncias | Uma só — single-worker | `F11` |
| Ordering | Por `order_id`, implícita pela ordem de `created_at`, enquanto for single-worker | `F11` · `[09:12] Diego` |

Sobre a conexão, Bruno em `[09:30]`: *"Separado. PrismaClient é por processo. Mesmo banco, mesma DATABASE_URL,
mas instância nova porque é outro processo Node."* O código confirma a mecânica: `src/config/database.ts:10`
exporta uma instância criada na importação do módulo, então cada processo que o importa ganha o seu pool.

### Decisões derivadas — a reunião não tratou estes pontos

Os quatro itens abaixo fecham o *como* do loop e **não podem ser lidos como fala da reunião**. Cada um carrega
a procedência registrada na seção H do contrato de fatos.

| # | Decisão derivada | Valor | Procedência |
| --- | --- | --- | --- |
| `H08` | Tamanho do batch | 50, por env com default | `PRODUTO` — `[09:08]` diz apenas "batch pequeno" |
| `H09` | Claim das linhas | `SELECT … FOR UPDATE SKIP LOCKED` | `MERCADO` + `CODIGO` (MySQL 8.0 no `docker-compose.yml`) |
| `H10` | Lease de evento travado | `processingStartedAt` + reclaim após 60 s | `PRODUTO` — buraco que a reunião não enxergou |
| `H13` | Paralelismo do batch | Serial por `order_id`, paralelo entre `order_id`s distintos, concorrência 10 | Derivado de `[09:12]` (ordering) e `[09:04]` (cliente lento não trava os outros) |

**`H09`** existe porque marcar o lote como *processando* precisa ser atômico. Sem o claim, dois leitores
simultâneos — o worker e a instância que sobe antes da antiga morrer, num deploy — pegariam a mesma linha.
`SKIP LOCKED` faz o segundo leitor pular a linha travada em vez de esperar por ela, o que mantém o ciclo
dentro dos 2 segundos.

**`H10`** cobre o caso em que o processo morre entre marcar o evento como *processando* e concluir o envio.
Como a leitura considera só os pendentes (`F05`), essa linha nunca mais seria vista. O campo
`processingStartedAt` dá idade ao registro e permite reivindicá-lo após 60 s, seis vezes o timeout de 10 s por
tentativa (`F28`, cuja política é de [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)). O reenvio resultante é
duplicata legítima, coberta pela garantia at-least-once de
[ADR-005](ADR-005-entrega-at-least-once-com-event-id.md).

**`H13`** é a decisão derivada com o maior peso, e a aritmética explica por quê:

```text
batch            = 50 eventos                 (H08)
timeout          = 10 s por tentativa         (F28, ADR-003)

em série         : 50 × 10 s        = 500 s   → 8 min 20 s até o último evento sair
concorrência 10  : ⌈50 / 10⌉ × 10 s =  50 s
```

Processar o batch em série coloca todo endpoint atrás do mais lento do lote. É exatamente o argumento que Bruno
usou em `[09:04]` para recusar o envio síncrono: *"Se a gente acrescentar um HTTP call no meio disso, qualquer
cliente lento vai travar mudança de status pra outros pedidos."* Trocar o `order.service` pelo worker não muda
a física do problema — só muda de lugar quem fica preso. E 500 s contra o requisito de menos de 10 segundos de
`[09:02]` derruba o motivo de o polling ser de 2 s. Daí a concorrência 10, com serialização apenas dentro de
cada `order_id`, que é o escopo exato da garantia de ordem de `F11`. Os valores vão para env validada por Zod
(`H24`: `WEBHOOK_BATCH_SIZE`, `WEBHOOK_CONCURRENCY`, `WEBHOOK_LEASE_TIMEOUT_MS`, `WEBHOOK_WORKER_ENABLED`),
no mesmo schema de `src/config/env.ts:4-27`, que aborta o boot com `process.exit(1)` quando a configuração está
errada.

### O que este ADR não cobre

- Tabela da outbox, índices e atomicidade do insert → [ADR-001](ADR-001-outbox-transacional-no-mysql.md)
- O que fazer quando a entrega falha — tentativas, backoff, DLQ, replay e o timeout de 10 s →
  [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)
- Duplicidade percebida pelo cliente e dedup por `X-Event-Id` →
  [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md)
- Forma do módulo, nome do arquivo de processamento e prefixo `WEBHOOK_` →
  [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)
- Rate limiting de saída para o cliente → questão em aberto `Q01` no [RFC](../RFC.md), dono Diego

## Alternativas Consideradas

### Trigger de banco para notificar o worker

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | Bruno — `[09:09]`                  |
| **Quem derrubou**   | Diego, na mesma linha              |
| **Citação**         | > "MySQL não tem listener nativo tipo o NOTIFY/LISTEN do Postgres. Trigger no banco a gente até tem, mas ela não notifica processo externo, ela só executa SQL." — `[09:09] Diego` |
| **Trade-off aceito**| Até 2 s de espera em vez de reação imediata, e uma consulta à outbox a cada ciclo mesmo quando não há evento |

A pergunta de Bruno foi *"Não dá pra usar trigger do banco pra ser mais reativo?"*, e a recusa é de capacidade,
não de gosto: o mecanismo não existe no MySQL. Diego descreveu os improvisos possíveis — escrever num arquivo,
bater num endpoint — e os classificou como esquisitos. Cada um deles acrescentaria um canal de comunicação
próprio, com falha própria, para ganhar no máximo 2 segundos.

O critério que encerrou a discussão foi o orçamento de latência, não a elegância: 2 s cabem folgadamente na
janela de menos de 10 segundos, e Marcos ratificou em `[09:10]`: *"2 segundos serve, perfeito."*

### Worker embutido na mesma instância da API

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | — (ninguém defendeu; Diego a antecipou para recusar) |
| **Quem derrubou**   | Diego — `[09:11]`                  |
| **Citação**         | > "Uma coisa importante: o worker tem que rodar como processo separado, não dentro da mesma instância da API. Senão se a API reinicia, perde o worker." — `[09:11] Diego` |
| **Trade-off aceito**| Dois processos para subir, operar e derrubar; segundo pool de conexões contra o mesmo MySQL |

Rodar o loop dentro do processo Express acopla a entrega ao ciclo de vida da API. Todo deploy, todo restart e
todo `SIGTERM` da API passariam a interromper envio em andamento, e o encerramento atual não protegeria nada:
`src/server.ts:13-21` não aguarda o `server.close()` antes de desconectar o Prisma e chamar `process.exit(0)`.
Uma chamada HTTP de até 10 s aberta nesse instante morre no meio.

Larissa converteu a recusa em desenho concreto em `[09:11]`: *"Tipo o que a gente já tem em src/server.ts,
criar um src/worker.ts e um script "npm run worker"."* Diego fechou o contorno da separação — *"Sim, mesmo
banco, mesma stack. Só não pode ser o mesmo processo."* — o que mantém o custo baixo: nenhum serviço novo no
`docker-compose.yml` e nenhuma dependência nova, já que não há cliente HTTP nas dependências
(`package.json:25-34`) e o `fetch` nativo do Node 20 (`package.json:7-9`) resolve o envio.

### Vários workers em paralelo desde o início (análise deste documento — não levantada na reunião)

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | —                                  |
| **Quem derrubou**   | Análise deste documento, com base em `[09:12] Diego` e `[09:13] Larissa` |
| **Citação**         | > "Se a gente escala pra múltiplos workers em paralelo no futuro, perde a garantia. Por enquanto, single-worker e ordering implícita por order_id." — `[09:12] Diego` |
| **Trade-off aceito**| Vazão limitada ao que uma única instância entrega por ciclo |

Ninguém propôs subir N workers desde o primeiro dia, mas a reunião impôs a restrição que hoje inviabiliza essa
opção: a ordem em que o cliente recebe as mudanças de um mesmo pedido depende de haver um único consumidor.
Larissa registrou o limite explicitamente em `[09:13]`: *"Não é garantia de ordering global, só por order_id e
enquanto for single-worker."*

Diego já nomeou as duas saídas para quando o dia chegar — particionar por `order_id` ou lock pessimista — e as
classificou como *"problema do futuro, não agora"*. Isso vive como `Q02` no [RFC](../RFC.md), com dono. O claim
por `FOR UPDATE SKIP LOCKED` (`H09`) foi escolhido de forma a não atrapalhar essa evolução, mas ele sozinho não
recupera a ordem: dois workers com claim correto ainda podem entregar dois eventos do mesmo pedido fora de
sequência.

## Consequências

### Positivas

- Reinício, deploy ou queda da API não interrompem entrega em andamento, e o inverso também vale — foi o
  motivo declarado da separação em `[09:11]`.
- A decisão não acrescenta infraestrutura: o `docker-compose.yml` continua com um único serviço e o worker
  reaproveita a stack existente (Prisma, Pino, env Zod, `fetch` nativo).
- `src/worker.ts` espelhando `src/server.ts` mantém uma forma já conhecida no projeto — mesma validação de env
  com `process.exit(1)` em falha (`src/config/env.ts:4-27`), mesmo logger, mesmos módulos.
- Ordering por `order_id` sai de graça enquanto houver uma instância: nenhuma coordenação, nenhum lock de
  aplicação, só a ordem de `created_at`.
- `WEBHOOK_WORKER_ENABLED` (`H24`) permite subir a API sem o consumidor, o que é útil nos testes de integração
  reais contra MySQL, sem gambiarra de código.

### Negativas

- Passam a existir duas unidades de deploy e dois pools de conexão contra o mesmo banco. `src/config/database.ts:10`
  cria o client na importação do módulo, então o worker abre o seu próprio pool desde o primeiro import.
- O polling consulta a outbox a cada 2 segundos mesmo quando não há nada para enviar. É carga constante e em
  boa parte ociosa, no mesmo MySQL que atende a API.
- O worker não tem requisição de origem: o `requestId` produzido em
  `src/middlewares/request-logger.middleware.ts:6-8` só existe dentro do ciclo de request da API. Correlacionar
  a entrega com a mudança de status que a originou exige carregar esse id junto com o evento — não sai de graça.
- O encerramento de `src/server.ts:13-21` não serve como modelo para o worker. Copiado como está, ele encerra o
  processo enquanto uma tentativa de até 10 s ainda está aberta, transformando cada deploy numa fonte previsível
  de reenvio via lease (`H10`).
- Três dos números que governam o loop — batch, concorrência e lease — foram fixados fora da reunião, sem
  medição. São defensáveis por aritmética, não por observação.

### Limitações conhecidas

| Limitação | Gatilho de reabertura |
| --------- | --------------------- |
| Ordering vale só por `order_id` e só enquanto houver uma única instância do worker (`F11`) | Quando for necessário rodar mais de um worker; particionar por `order_id` ou lock pessimista já estão registrados em `Q02`, dono Diego |
| Até 2 s de espera antes do primeiro envio; somados a uma tentativa que responda perto do timeout de 10 s, a entrega ultrapassa a janela de menos de 10 segundos de `[09:02]` | Quando o tempo de resposta registrado em `GET /webhooks/:id/deliveries` ficar recorrentemente próximo do timeout para algum endpoint |
| Evento reivindicado por lease após crash (`H10`) é reenviado, e o cliente vê duplicata | Aceito por [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md); reabrir se a dedup por `X-Event-Id` do lado do cliente se mostrar insuficiente |
| Concorrência 10 (`H13`) é um valor escolhido sem medição; muito baixo atrasa o lote, muito alto pressiona o mesmo MySQL da API | Quando o número de pendentes crescer de um ciclo de polling para o seguinte de forma sustentada. Ajustável por `WEBHOOK_CONCURRENCY`, mas o default só muda por PR (`H23`) |
| Nada foi decidido sobre quem supervisiona e reinicia o processo do worker se ele morrer — a reunião só disse que ele existe | Antes do primeiro deploy, na sessão de revisão do design anunciada por Larissa em `[09:50]` |

## Referências

- Transcrição: `[09:02] Marcos` · `[09:04] Bruno` · `[09:07] Diego` · `[09:08] Diego` · `[09:09] Bruno` ·
  `[09:09] Diego` · `[09:10] Marcos` · `[09:10] Larissa` · `[09:11] Diego` · `[09:11] Larissa` ·
  `[09:12] Diego` · `[09:13] Diego` · `[09:13] Larissa` · `[09:30] Bruno` · `[09:48] Larissa` · `[09:49]` ·
  `[09:50] Larissa`
- Código: `src/server.ts:13-21` · `src/config/database.ts:10` · `src/config/env.ts:4-27` ·
  `src/middlewares/request-logger.middleware.ts:6-8` · `package.json:7-9` · `package.json:10-21` ·
  `package.json:25-34` · `docker-compose.yml:2-24`
- Contrato de fatos: `F05` `F06` `F07` `F08` `F09` `F10` `F11` · `A03` · `C13` `C14` `C16` `C20` `C23` ·
  `H08` `H09` `H10` `H13` `H23` `H24` · `Q01` `Q02` · `F28` (emprestado de [ADR-003](ADR-003-retry-com-backoff-e-dlq.md))
