# ADR-003 — Retry com backoff exponencial de 5 retentativas e dead letter queue em tabela dedicada

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:48] Larissa` (resumo) · `[09:49]` Diego, Bruno e Sofia confirmam           |
| **Relacionados** | [ADR-001](ADR-001-outbox-transacional-no-mysql.md) · [ADR-002](ADR-002-worker-separado-em-polling.md) · [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md) · [ADR-008](ADR-008-controle-de-acesso-dos-endpoints.md) |

## Contexto

Larissa abriu o bloco com uma pergunta única — `[09:14] Larissa`: "Vamos pra retry. Se o cliente tá
offline, o que a gente faz?" — mas a resposta de Diego já veio partida em três perguntas encadeadas,
e é assim que elas ficam registradas aqui:

1. **Quantas vezes tentamos de novo** antes de desistir de um evento?
2. **Com que espaçamento** entre uma tentativa e a seguinte?
3. **Para onde vai o evento** quando as tentativas acabam?

A ordem em que Diego respondeu inverte a ordem natural, e por um motivo. `[09:15] Diego`: "Backoff
exponencial. Tenta de novo depois de algum tempo, vai aumentando o intervalo, e depois de um teto de
tentativas considera falha permanente e move pra DLQ." O destino de falha permanente aparece na mesma
frase que o teto porque a terceira pergunta é pré-requisito da primeira: sem um lugar para onde mandar
o evento que não entrou, não existe teto possível. A única alternativa a ter um destino terminal é
retentar para sempre — e foi exatamente essa a hipótese que Diego levantou e derrubou na frase seguinte.

O sistema atual não oferece nenhum apoio a esse mecanismo. O `docker-compose.yml:2-24` sobe só MySQL:
não há broker com retry e DLQ nativos, então a curva de reentrega precisa ser dados numa tabela, não
configuração de infraestrutura. `package.json:25-34` não tem cliente HTTP algum e `package.json:8` fixa
`node: ">=20"`, o que resolve o timeout com `fetch` nativo e `AbortSignal.timeout` sem dependência nova.
E o shutdown existente (`src/server.ts:13-21`) não aguarda `server.close()` nem drena trabalho em
andamento — o que torna obrigatório que o agendamento da próxima tentativa viva no banco, e não num
timer em memória: um restart no meio de uma espera de doze horas não pode perder o evento.

Havia um limite de paciência declarado do lado de produto. `[09:17] Marcos`: "Se um cliente meu cair por
15 horas, ele já tá com problema sério dele. Acho aceitável." Foi esse aceite que autorizou a curva a se
estender por mais de meio dia em vez de morrer em minutos.

## Decisão

Cada entrega falha é reagendada com espera crescente. São **5 retentativas agendadas** depois da entrega
inicial, com as esperas de `[09:17] Diego`. Esgotadas as retentativas, a linha sai da outbox e vai para a
tabela dedicada `webhook_dead_letter`, com payload, motivo da falha e timestamp.

### A conta, sem ambiguidade

A palavra "tentativa" aparece na fita com dois sentidos possíveis. Esta é a leitura adotada e o pacote
inteiro usa esta linha do tempo:

| Chamada HTTP | Papel | Espera anterior | Instante desde a 1ª falha |
| --- | --- | --- | --- |
| 1 | entrega inicial | — (≤ 2s após o commit) | `t = 0` |
| 2 | retentativa 1 | 1 min | `00h01` |
| 3 | retentativa 2 | 5 min | `00h06` |
| 4 | retentativa 3 | 30 min | `00h36` |
| 5 | retentativa 4 | 2 h | `02h36` |
| 6 | retentativa 5 | 12 h | `14h36` → falhou ⇒ `webhook_dead_letter` |

`1m + 5m + 30m + 2h + 12h = 14h36min`. **5 retentativas, 5 intervalos, 6 chamadas HTTP no pior caso.**

Essa é a única leitura que fecha com as duas afirmações numéricas da reunião. `[09:17] Diego`: "Total de
quase 15 horas entre primeira falha e última tentativa" e `[09:15] Diego`: "Cinco já dá pra cobrir uma
janela de até 12 ou 24 horas". Contar 5 chamadas HTTP no total consumiria só 4 intervalos, encerraria em
`02h36` e deixaria o degrau de 12 horas sem uso — contradizendo as duas falas. Onde outros documentos
deste pacote dizem "5 tentativas", incluindo o resumo de `[09:48] Larissa`, leia "5 retentativas": o
número de chamadas HTTP no pior caso é 6, e nenhum documento deve publicar outro valor.

### Valores e destino

| Item | Valor | Origem |
| --- | --- | --- |
| Retentativas agendadas | 5 | `[09:15] Diego` / `[09:17] Larissa` |
| Curva de backoff | 1m · 5m · 30m · 2h · 12h | `[09:17] Diego` |
| Janela total desde a 1ª falha | 14h36min | soma da curva acima |
| Timeout por chamada | 10 s | `[09:42] Diego` |
| Destino terminal | tabela `webhook_dead_letter` | `[09:18] Diego` |
| Conteúdo da DLQ | payload, motivo da falha, timestamp | `[09:18] Diego` |

Os valores são constantes de código, não variáveis de ambiente — mudar a curva exige PR e reabertura
deste ADR:

```ts
// src/modules/webhooks/webhook.constants.ts
export const MAX_ATTEMPTS = 5;          // retentativas · ADR-003 · [09:17] Larissa
export const BACKOFF_MS = [60_000, 300_000, 1_800_000, 7_200_000, 43_200_000]; // ADR-003 · [09:17] Diego
export const HTTP_TIMEOUT_MS = 10_000;  // ADR-003 · [09:42] Diego
```

### Decisões derivadas

Três pontos que a curva exige e a reunião não tratou. Nenhum deles é fala de participante.

| Ponto | Decisão | Procedência |
| --- | --- | --- |
| Classe de falha | Toda resposta não-2xx e toda falha de rede consomem as 5 retentativas. Sem atalho de 4xx direto para a DLQ | Decisão derivada — leitura literal de `[09:15]` e `[09:17]`, que nunca separaram erro do cliente de erro nosso |
| Jitter | Não há. A curva é exatamente a de `[09:17] Diego`. Jitter fica como melhoria futura | Decisão derivada — a reunião não mencionou aleatorização |
| Redirects 3xx | Não seguimos redirect; conta como falha e consome retentativa | Decisão derivada — padrão de mercado verificado (Stripe) |

### O que este ADR não cobre

- **Endpoint de replay da DLQ e quem pode chamá-lo** → [ADR-008](ADR-008-controle-de-acesso-dos-endpoints.md).
- **Duplicidade de entrega e dedup por `X-Event-Id`** → [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md).
- **Como o worker lê a outbox, polling e paralelismo** → [ADR-002](ADR-002-worker-separado-em-polling.md).
- **Atomicidade do registro do evento** → [ADR-001](ADR-001-outbox-transacional-no-mysql.md).
- Colunas, índices e contrato HTTP das tabelas envolvidas → `docs/FDD.md`.

## Alternativas Consideradas

### 3 tentativas

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | Bruno                              |
| **Quem derrubou**   | Diego, na mesma linha              |
| **Citação**         | > "3 é pouco. Se o cliente teve indisponibilidade de manhã, a gente retentaria três vezes em 30 minutos e mataria. Já tinha cliente nosso com indisponibilidade de duas horas em manutenção planejada." — `[09:16] Diego` |
| **Trade-off aceito**| Um endpoint definitivamente morto ocupa linha na outbox por quase 15 horas em vez de 36 minutos |

Bruno propôs em `[09:16]`: "3 não é melhor? Mais agressivo." A recusa é aritmética e verificável na
própria curva. Com 3 retentativas, os intervalos consumidos seriam 1m, 5m e 30m — a última tentativa cai
em `00h36`. O caso concreto que Diego trouxe, um cliente com duas horas de manutenção planejada, ficaria
inteiramente fora dessa janela: o evento estaria na DLQ mais de uma hora antes de o cliente voltar.

Com 5 retentativas, a quarta cai em `02h36`, depois do fim de uma manutenção de duas horas, e a quinta
estende a cobertura para além de um turno de trabalho. Larissa fechou em `[09:16]`: "Cinco fica bom."

### Retry indefinido com backoff

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | — (Diego citou como posição de terceiros para descartar) |
| **Quem derrubou**   | Diego                              |
| **Citação**         | > "Algumas pessoas defendem retry indefinido com backoff, mas isso traz o problema de evento ficar pendurado pra sempre se o cliente sumiu." — `[09:15] Diego` |
| **Trade-off aceito**| Um cliente que volta depois de 14h36min não recebe o evento automaticamente; depende de replay manual |

Retry infinito elimina a necessidade da terceira pergunta, e é justamente por isso que foi descartado.
Sem destino terminal, a outbox nunca chega a estado final para um endpoint abandonado: as linhas se
acumulam, o índice de status (`[09:08] Diego`) passa a varrer lixo permanente e nenhuma consulta consegue
distinguir "vai entregar" de "nunca vai entregar".

O preço é real e está assumido. Um cliente que fique fora por dezesseis horas perde a entrega automática
e só recupera o evento por reprocessamento manual.

### Marcar `failed` na própria outbox

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | Larissa (levantou como pergunta)   |
| **Quem derrubou**   | Diego                              |
| **Citação**         | > "Eu fazia uma tabela webhook_dead_letter separada, com a payload, motivo da falha e timestamp. Mais limpa a leitura da outbox principal, e fica como evidence pra debug e reprocessamento." — `[09:18] Diego` |
| **Trade-off aceito**| O mesmo evento passa a ter registro em dois lugares, e o replay precisa reconstruir a linha da outbox |

A pergunta veio de `[09:17] Larissa`: "Faz numa tabela separada ou marca como "failed" na própria
outbox?" O estado `falhou` já existia na lista de `[09:08] Diego`, então a opção era viável sem nenhuma
tabela nova. O argumento que decidiu foi de operação, não de modelagem: a outbox é lida em loop pelo
worker e precisa ser a tabela dos eventos vivos. Falha permanente acumulada ali é peso morto num caminho
quente.

A separação também muda o público da tabela. `webhook_dead_letter` é lida por gente, na investigação de
um problema, e não pelo worker — o que justifica guardar motivo da falha e timestamp num formato voltado
a leitura humana, sem poluir o esquema da outbox com colunas que só interessam ao caso de exceção.

## Consequências

### Positivas

- Cobertura de indisponibilidade longa sem intervenção humana: manutenção planejada de horas, o caso
  concreto de `[09:16] Diego`, é absorvida pela curva.
- Terminação garantida. Toda linha da outbox chega a um estado final, `entregue` ou saída para a DLQ.
  Nenhum evento fica pendurado.
- A leitura da outbox permanece enxuta: o worker varre eventos vivos, e o histórico de fracasso mora fora
  do caminho quente.
- O agendamento vive no banco, não em memória. Como `src/server.ts:13-21` não drena trabalho em
  andamento, um restart durante a espera de 12 horas é inofensivo.
- Nenhuma dependência nova. `fetch` nativo com `AbortSignal.timeout(10_000)` cobre o timeout de
  `[09:42] Diego` sobre `package.json:8`.

### Negativas

- Um endpoint que responde 400 a um payload que ele nunca vai aceitar consome as 6 chamadas e ocupa a
  fila por 14h36min. É o custo de não classificar erro de cliente automaticamente.
- O salto de 2h para 12h não tem degrau intermediário. Quem volta ao ar dez minutos depois da quarta
  retentativa espera quase meio dia pelo próximo envio.
- Sem jitter, eventos do mesmo endpoint que falharam juntos voltam juntos em cada degrau. Com fan-out
  por endpoint, o pico de reentrega é proporcional ao número de eventos represados.
- A DLQ separada custa um segundo lugar para olhar durante investigação, e o replay tem que recriar a
  linha da outbox a partir dela.
- Redirect 3xx tratado como falha significa que um cliente que cadastrou URL com redirecionamento
  legítimo só descobre o problema depois das 6 chamadas.
- A chegada de um evento à DLQ não dispara aviso ativo. Alerta por email (`[09:37] Larissa`) e painel
  visual (`[09:40] Larissa`) estão explicitamente fora desta fase; a detecção depende de quem consultar
  a tabela ou ler os eventos estruturados do Pino (`src/shared/logger/index.ts:4-11`).
- A tabela nova entra na limpeza de testes de `tests/setup.ts:9-15`, respeitando ordem de FK, e segue o
  padrão de PK e `@@map` de `prisma/schema.prisma:25-138`.

### Limitações conhecidas

| Limitação | Gatilho de reabertura |
| --------- | --------------------- |
| Sem jitter: reentrega sincronizada a cada degrau | Quando um endpoint acumular eventos represados suficientes para que o retorno em bloco pese na saída — reabrir junto com a questão de rate limiting de envio, dono Diego (`[09:39]`) |
| 4xx consome as 5 retentativas | Quando a DLQ mostrar que a maior parte das entradas chegou por 4xx repetido desde a primeira chamada |
| Salto de 2h → 12h sem degrau intermediário | Quando houver caso registrado de cliente que voltou dentro dessa faixa e esperou o degrau inteiro |
| Curva única para todos os endpoints | Quando um cliente pedir janela de reentrega própria por endpoint |
| Evento entra na DLQ sem aviso ativo | Na próxima fase, "depois que a gente medir o impacto" — `[09:37] Larissa` |
| 3xx não seguido | Quando um cliente cadastrar URL com redirecionamento legítimo e a entrega falhar por isso |

## Referências

- Transcrição: `[09:08] Diego` · `[09:14] Larissa` · `[09:15] Diego` · `[09:15] Bruno` · `[09:16] Bruno` · `[09:16] Diego` · `[09:16] Larissa` · `[09:17] Diego` · `[09:17] Marcos` · `[09:17] Larissa` · `[09:18] Diego` · `[09:37] Larissa` · `[09:39] Diego` · `[09:40] Larissa` · `[09:42] Diego` · `[09:48] Larissa` · `[09:49]` Diego, Bruno e Sofia
- Código: `package.json:8` · `package.json:25-34` · `src/server.ts:13-21` · `docker-compose.yml:2-24` · `prisma/schema.prisma:25-138` · `tests/setup.ts:9-15` · `src/shared/logger/index.ts:4-11`
- Contrato de fatos: `F12` `F13` `F14` `F15` `F28` · `A04` `A05` `A06` · `H11` `H12` `H14` `H23` · `X01` `X02` · `C14` `C16` `C17` `C20`
