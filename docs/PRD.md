# PRD — Sistema de Webhooks de Notificação de Pedidos

|                |                                                                                 |
| -------------- | ------------------------------------------------------------------------------- |
| **Status**     | Em revisão                                                                       |
| **Data**       | 2026-08-09                                                                       |
| **Produto**    | Marcos — Product Manager                                                         |
| **Engenharia** | Larissa (Tech Lead) · Bruno · Diego · Sofia                                      |
| **Prazo**      | 3 sprints · compromisso comercial para o fim de novembro                          |
| **Relacionados** | [`RFC.md`](RFC.md) · [`FDD.md`](FDD.md) · [`adrs/`](adrs/) · [`TRACKER.md`](TRACKER.md) |

---

## Resumo e contexto da feature

Três clientes B2B — Atlas Comercial, MaxDistribuição e Nova Cargo — pediram formalmente para ser notificados
quando o status dos pedidos deles muda. Hoje eles descobrem por consulta repetida ao nosso `GET /orders`, o que
torna a integração lenta e cara do lado deles. A Atlas sinalizou que pode migrar para o concorrente se isso não
sair neste trimestre.

Vamos entregar **webhooks de saída**: o cliente cadastra um endereço, escolhe quais mudanças de status quer
receber, e passa a ser avisado automaticamente. Cada notificação vai assinada, para que ele possa verificar que
veio de nós e que ninguém alterou o conteúdo no caminho.

**Escopo desta fase:** cadastro por API, notificação de mudança de status, histórico de entregas e
reprocessamento manual pelo time. **Fora:** painel visual, alerta por e-mail e recebimento de webhooks.

**Custo:** três sprints, incluindo dois dias reservados para revisão de segurança antes do deploy.

## Problema e motivação

Um cliente que precisa saber quando o pedido dele mudou de status só tem hoje uma opção: perguntar. De tempos
em tempos, indefinidamente, para todos os pedidos.

Isso produz três efeitos, todos relatados por Marcos em `[09:00]`:

- **Integração lenta.** O cliente descobre a mudança no próximo ciclo de consulta, não quando ela acontece.
- **Integração cara.** O custo é dele — infraestrutura e chamadas — e nosso, porque a carga de consulta chega
  ao nosso banco.
- **Risco comercial.** A Atlas colocou prazo. O pedido dos três chegou formalmente, não como sugestão.

O sistema não tem hoje nenhum mecanismo de notificação externa. Não é um recurso incompleto: é um recurso
inexistente.

## Restrições e premissas

Parte do desenho já estava decidida antes de alguém falar em arquitetura.

**O time é pequeno.** Diego disse com todas as letras em `[09:07]`: *"a gente é um time pequeno. Subir Redis
Cluster pra isso é overengineering"*. Larissa tinha levantado a alternativa um minuto antes e chegado ao mesmo
lugar — *"a gente acabaria precisando subir mais infra"*. A restrição é de operação: quem sobe também mantém.

**A data veio do cliente.** Marcos trouxe em `[09:45]`: *"A Atlas quer pra fim de novembro"*. O prazo entrou
na conversa como fato, e a estimativa teve de caber nele.

**Existe um usuário nosso que representa o cliente.** Premissa que sustenta todo o cadastro desta fase, e que
Marcos estabeleceu em `[09:32]`: *"A gente tem usuários que representam o cliente"*. Se o cliente B2B um dia
passar a integrar direto com a nossa API, o modelo de acesso muda inteiro.

## Público-alvo e cenários de uso

| Público | Quem é | O que faz com a feature |
|---|---|---|
| **Cliente B2B** | Atlas Comercial, MaxDistribuição, Nova Cargo | Recebe as notificações e reage a elas no próprio sistema |
| **Usuário operador** | Quem opera o OMS e representa o cliente internamente | Cadastra e mantém os endereços de webhook pela API |
| **Administrador** | Papel `ADMIN` do sistema | Investiga falhas e reprocessa eventos que não foram entregues |

> O cliente B2B **não** acessa a API diretamente nesta fase. O cadastro é feito por um usuário autenticado do
> nosso sistema, conforme Marcos esclareceu em `[09:32]`.

**Cenário 1 — integração inicial.** A Atlas quer ser avisada quando um pedido é despachado e quando é entregue.
Um operador cadastra o endereço deles assinando `SHIPPED` e `DELIVERED`, e entrega o segredo gerado. A partir
daí, toda mudança para esses dois status vira uma chamada ao sistema da Atlas.

**Cenário 2 — indisponibilidade do cliente.** O sistema da MaxDistribuição fica fora do ar durante uma
manutenção de duas horas. As notificações do período não se perdem: são retentadas com intervalos crescentes e
chegam quando o sistema volta.

**Cenário 3 — investigação.** A Nova Cargo diz que não recebeu a notificação de um pedido. O operador consulta
o histórico de entregas daquele cadastro e vê as tentativas, o código de resposta devolvido pelo sistema deles
e o tempo de cada uma.

**Cenário 4 — credencial comprometida.** O cliente informa que o segredo vazou. O operador rotaciona; durante
24 horas as duas credenciais funcionam, para que o cliente migre sem perder notificação.

**Cenário 5 — cliente com mais de um cadastro.** O mesmo cliente mantém dois endereços registrados, cada um
assinando um conjunto diferente de status. Toda notificação carrega a identificação do cadastro que a originou,
pedido de Sofia em `[09:44]`: *"pra cliente que tem vários conseguir saber qual cadastro caiu naquele envio"*.
Cada cadastro tem segredo próprio e histórico próprio. E o filtro olha todos os cadastros do cliente antes de
gerar o evento, como Bruno registrou em `[09:34]`: se nenhum assina aquele status, nada é gerado.

## Objetivos e métricas de sucesso

| # | Objetivo | Métrica | Meta | Origem |
|---|---|---|---|---|
| OBJ-01 | Notificar em tempo percebido como real | Tempo entre a mudança de status e a chegada ao cliente, na primeira tentativa | **Abaixo de 10 segundos**, medido como percentil — ver a nota | `[09:02] Marcos` |
| OBJ-02 | Não perder evento | Eventos com mudança de status confirmada e sem registro correspondente | **Zero** | `[09:06] Diego` · `[09:40] Bruno` |
| OBJ-03 | Não degradar a operação existente | Endpoints existentes com contrato ou comportamento alterado | **Zero** | `[09:04] Bruno` |
| OBJ-04 | Atender os três clientes que pediram | Clientes com webhook ativo recebendo notificação | **3** | `[09:00] Marcos` |
| OBJ-05 | Entregar dentro do compromisso comercial | Sprints até o deploy | **3**, com a revisão de segurança inclusa | `[09:46] Larissa` · `[09:46] Sofia` |
| OBJ-06 | Reduzir a carga de consulta que os clientes fazem hoje | Volume de chamadas ao `GET /orders` vindas dos três clientes | **Sem meta numérica** — ver a nota | `[09:00] Marcos` |

> **Sobre a meta do OBJ-01.** O número acordado é o de `[09:02] Marcos`: *"Pra eles, qualquer coisa abaixo de
> 10 segundos já é 'tempo real'."* A **formulação como percentil é derivação nossa**, não decisão da reunião.
> Somando a espera do ciclo de leitura ao tempo de resposta de um cliente que chegue perto do limite de
> timeout, uma entrega bem-sucedida ultrapassa os 10 segundos sem que nada tenha falhado. Um teto absoluto
> seria um compromisso que o desenho não sustenta. A aritmética está no [RFC](RFC.md) e no
> [FDD](FDD.md); o percentil exato precisa ser acordado com os clientes antes do deploy.

> **Sobre o OBJ-06 não ter meta.** A redução de consultas é consequência esperada e foi o motivo declarado do
> pedido. Mas não medimos hoje o volume de chamadas por cliente, e a reunião não estabeleceu linha de base nem
> alvo. Colocar um número aqui seria inventar um compromisso que ninguém assumiu. Fica como objetivo
> qualitativo até existir a medição.

### O que produto aceitou pelos clientes

Duas coisas foram aceitas na reunião por Marcos, em nome dos três clientes, sem que nenhum deles estivesse na
chamada. O compromisso é nosso; a conta chega neles.

O primeiro aceite é o piso de latência. Diego explicou o ciclo de leitura de 2 segundos e Marcos respondeu em
`[09:10]`: *"2 segundos serve, perfeito"*. Larissa transformou aquilo em decisão na mesma linha: *"A latência
mínima vai ser 2 segundos no pior caso. Aceitamos."* Cabe com folga dentro dos 10 segundos do OBJ-01, mas
existe: nenhuma notificação chega mais rápido que isso.

O segundo é o teto da janela de retentativa. Apresentada a progressão dos intervalos em `[09:17]`, Marcos
aceitou pelo cliente: *"Se um cliente meu cair por 15 horas, ele já tá com problema sério dele. Acho
aceitável."* A frase carrega um julgamento sobre a operação alheia. Se para algum dos três uma fila de quinze
horas for pior que uma falha rápida e visível, o número volta à mesa.

Nenhum dos dois foi validado com quem vai conviver com ele. Marcos assumiu levar o assunto aos clientes — ver
**Entregáveis além do software** — e a conversa fica pendente, ao lado da definição do percentil do OBJ-01.

## Escopo

### Incluído

Cadastro de endereços de webhook por cliente, com escolha de quais mudanças de status receber. Notificação
automática a cada mudança. Assinatura de cada notificação. Retentativa automática quando o cliente está
indisponível. Histórico de entregas consultável. Reprocessamento manual, restrito a administradores.

### Catálogo de eventos assináveis

O que o cliente escolhe ouvir é a chegada do pedido a um status, e o vocabulário dessa escolha é a máquina de
estados que o pedido já tem. São seis situações no ciclo de vida — `PENDING`, `PAID`, `PROCESSING`, `SHIPPED`,
`DELIVERED` e `CANCELLED` (`prisma/schema.prisma:16-23`).

Nem todas servem de assinatura. O mapa de transições em `src/modules/orders/order.status.ts:3-10` mostra que
`PENDING` é onde o pedido nasce e que nenhuma transição leva de volta até lá; assinar `PENDING` nunca
produziria notificação. Sobram cinco destinos: de `PENDING` o pedido vai para `PAID` ou `CANCELLED`; de `PAID`,
para `PROCESSING` ou `CANCELLED`; de `PROCESSING`, para `SHIPPED` ou `CANCELLED`; de `SHIPPED`, para
`DELIVERED`. `DELIVERED` e `CANCELLED` encerram o ciclo, e depois deles não há o que notificar.

Marcos deu o caso típico em `[09:33]`: *"só quero saber quando vira SHIPPED e DELIVERED"*. É a assinatura do
Cenário 1 — quem quer despacho e entrega ignora o resto do caminho. Nada impede a assinatura completa, e é o
caso que Larissa levantou em `[09:12]`: um pedido passando por `PAID`, `PROCESSING` e `SHIPPED` em sequência
rápida. As três notificações chegam, na ordem em que aconteceram (RNF-09).

### Fora de escopo

| Item | Classificação | Justificativa | Origem |
|---|---|---|---|
| Alerta por e-mail quando o webhook do cliente falha | **Adiado** — próxima fase | *"Email tá fora de escopo dessa fase. Talvez próxima fase, depois que a gente medir o impacto"* | `[09:37] Larissa` |
| Painel visual para o cliente acompanhar entregas | **Descartado** desta feature | *"Painel é projeto separado do time de frontend"* | `[09:40] Larissa` |
| Recebimento de webhooks enviados pelo cliente | **Descartado** — direção única | *"Só saindo da gente pra eles. Eles querem receber, não mandar"* | `[09:02] Marcos` |
| Arquivamento e expurgo de eventos antigos | **Adiado** | *"Linhas entregues a gente arquiva depois de 30 dias ou assim, fora do escopo dessa feature"* | `[09:08] Diego` |
| Limite de taxa de envio para um mesmo cliente | **Em observação** | *"A gente observa e implementa se virar problema"* | `[09:39] Diego` |
| Garantia de ordem global entre pedidos diferentes | **Adiado** | Ordem é garantida por pedido; global fica para quando houver mais de um consumidor | `[09:13] Diego` |
| Desativação automática de endereço com falha recorrente | **Não discutido** | Consequência de o alerta estar fora desta fase; sem decisão da reunião | — |

Cada item aparece **uma única vez** e com **uma única classificação** em todo o pacote. "Adiado" tem fase
prevista; "descartado" não; "em observação" tem gatilho declarado.

## Plano de entrega

Marcos perguntou em `[09:45]` em quantas sprints aquilo cabia. Larissa respondeu em `[09:46]` com a conta
aberta, e é a decomposição dela que organiza a entrega.

**Marco 1 — o evento tem onde morar.** *"Modelagem de outbox e DLQ é uma sprint."* Nada chega ao cliente
ainda. Fica pronto o registro dos eventos e o lugar onde as falhas definitivas param, que é a rede de que
R-05 depende.

**Marco 2 — a entrega funciona.** *"Worker e retry é uma sprint."* A partir daqui existe notificação saindo,
com a retentativa que sustenta o Cenário 2, e ainda sem nada acoplado à operação de pedidos.

**Marco 3 — o operador ganha superfície.** *"CRUD de configuração e deliveries é meio sprint."* Cadastro,
edição, remoção, listagem e o histórico de entregas de que o Cenário 3 depende.

**Marco 4 — o gatilho encosta na operação.** *"Integração no order.service e testes ponta a ponta é mais
meio."* Único momento em que a feature toca a transação central do sistema.

Assinatura, validação de esquema e regras de recusa correm por dentro desses blocos, sem sprint própria:
*"HMAC, schemas, validações, mais um pouco"*. No fim, os dois dias úteis de revisão de segurança. Larissa
confirmou o pacote em `[09:47]`: *"Três sprints com a revisão da Sofia incluída no fim."*

A ordem listada deixa a integração com pedidos por último, com o resto já em pé. Ler isso como redução
deliberada da exposição de R-01 é **interpretação nossa**: a fita registra a sequência e para por aí.

## Requisitos funcionais

| # | Requisito | Origem |
|---|---|---|
| RF-01 | O operador cadastra um endereço de webhook para um cliente | `[09:31] Marcos` |
| RF-02 | O segredo de assinatura é gerado pela plataforma e devolvido na criação | `[09:31] Marcos` |
| RF-03 | O cadastro define quais mudanças de status quer receber | `[09:33] Marcos` |
| RF-04 | O operador edita um cadastro existente | `[09:33] Bruno` |
| RF-05 | O operador remove um cadastro | `[09:33] Bruno` |
| RF-06 | O operador lista os cadastros de um cliente | `[09:33] Bruno` |
| RF-07 | O cadastro guarda um estado ativo, ao lado do endereço, do segredo e do cliente a que pertence | `[09:21] Bruno` · `[09:21] Sofia` |
| RF-08 | O operador solicita novo segredo, com o anterior válido por 24 horas | `[09:21] Sofia` |
| RF-09 | Toda mudança de status gera notificação para os cadastros que a assinam | `[09:00] Marcos` · `[09:06] Diego` |
| RF-10 | Cada notificação vai assinada, para o cliente verificar origem e integridade | `[09:20] Sofia` |
| RF-11 | Notificação que falha é retentada automaticamente | `[09:15] Diego` |
| RF-12 | O operador consulta o histórico de entregas de um cadastro, com resultado, resposta e tempo | `[09:34] Marcos` |
| RF-13 | Um administrador reprocessa manualmente uma notificação que esgotou as tentativas | `[09:35] Diego` |
| RF-14 | O reprocessamento registra quem o executou | `[09:36] Sofia` |

## Requisitos não funcionais

| # | Requisito | Origem |
|---|---|---|
| RNF-01 | A notificação não pode atrasar nem afetar a mudança de status | `[09:04] Bruno` |
| RNF-02 | Se a mudança de status é confirmada, o evento existe; se falha, o evento não existe | `[09:06] Diego` · `[09:40] Bruno` |
| RNF-03 | A entrega é garantida ao menos uma vez; o cliente pode receber duplicado e deve deduplicar | `[09:24] Diego` |
| RNF-04 | Cinco retentativas com intervalos crescentes, cobrindo cerca de 15 horas desde a primeira falha | `[09:15]` · `[09:17] Diego` |
| RNF-05 | Cada notificação tem no máximo 64 KB; acima disso, erro, sem truncar | `[09:23] Sofia` · `[09:24] Larissa` |
| RNF-06 | O endereço cadastrado tem de usar `https`; `http` é recusado | `[09:23] Sofia` |
| RNF-07 | Cada cadastro tem segredo próprio; não há segredo compartilhado da plataforma | `[09:21] Sofia` |
| RNF-08 | O cliente tem 10 segundos para responder; além disso, a tentativa é falha | `[09:42] Diego` |
| RNF-09 | Notificações do mesmo pedido chegam na ordem em que ocorreram | `[09:12] Diego` |
| RNF-10 | A feature não introduz infraestrutura nova | `[09:07] Diego` |
| RNF-11 | O reprocessamento exige papel de administrador | `[09:36] Sofia` |
| RNF-12 | Nenhum contrato de API existente é alterado | `[09:04] Bruno` |

## Decisões e trade-offs principais

Decisões técnicas foram fechadas na reunião e cada uma tem registro próprio em [`adrs/`](adrs/). O que importa
para produto são os trade-offs que elas produzem:

| Trade-off aceito | O que ganhamos | O que custa |
|---|---|---|
| Notificação assíncrona em vez de imediata | Cliente lento nunca trava a operação de pedidos | Existe atraso entre a mudança e a chegada |
| Ao menos uma entrega, em vez de exatamente uma | Desenho muito mais simples e previsível | O cliente precisa tratar duplicata — Marcos documenta no portal |
| Retentativa longa, cobrindo cerca de 15 horas | Cliente com manutenção planejada não perde nada | Endereço cadastrado errado só aparece como falha definitiva no dia seguinte |
| Notificação enxuta, sem os itens do pedido | Payload pequeno e barato de trafegar | Quem precisa de detalhe faz uma consulta a mais |
| Sem infraestrutura nova | Cabe em três sprints, com o time atual | Ordem garantida só enquanto houver um único consumidor |
| Cadastro por qualquer usuário autenticado | Integração sem fricção nesta fase | Não há verificação de posse entre usuário e cliente — ver riscos |

## Dependências

| Dependência | Natureza | Situação |
|---|---|---|
| Revisão de segurança da Sofia, dois dias úteis antes do deploy | **Bloqueante** | Combinada em `[09:46]`; três pendências endereçadas a ela |
| MySQL já existente | Técnica | Disponível; nenhuma infraestrutura nova |
| Endereço `https` acessível no lado do cliente | Externa | Cada cliente precisa expor e informar |
| Documentação de integração no portal do desenvolvedor | Produto | Marcos assumiu em `[09:26]` e `[09:40]` |
| Definição do percentil de latência com os clientes | Produto | Pendente — ver OBJ-01 |

## Entregáveis além do software

Marcos assumiu obrigações que não são código, e sem as quais a feature não funciona do lado de lá.

**A documentação da deduplicação.** Fechada a entrega ao menos uma vez, Sofia observou que aquilo empurrava
trabalho para o cliente. Marcos se ofereceu em `[09:26]`: *"Eu posso documentar isso bem destacado no portal de
desenvolvedor pros clientes"*. O destaque faz parte do entregável — quem ler por alto e não deduplicar processa
o mesmo pedido duas vezes, que é o R-04.

**O guia de integração.** Em `[09:40]`, descartado o painel visual: *"Eu documento no portal pros clientes
saberem como integrar via API"*. Sem interface, o portal vira a única superfície pela qual o cliente descobre
como cadastrar um endereço, escolher os status e verificar a assinatura.

Some-se a comunicação com os três: confirmar o prazo com a Atlas, assumido em `[09:47]`, e a atualização geral
prometida em `[09:49]`. Nenhum dos itens ganhou data; a referência é a mesma da revisão de segurança.

## Riscos e mitigação

| # | Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|---|
| R-01 | Um defeito no módulo novo derruba a mudança de status, a operação central do sistema | Média | **Crítico** | Gate de ambiente que desliga a publicação do evento sem redeploy, no padrão de variável validada por Zod já em uso em `src/config/env.ts:4-27`; CA-12 cobrindo explicitamente o caminho de erro; os dois dias úteis de revisão de segurança que Sofia pediu em `[09:46]` |
| R-02 | A promessa de 10 segundos não se cumpre para clientes que respondem devagar | Alta | Médio | Meta formulada como percentil, não teto; tempo de resposta medido por cadastro; conversa com o cliente antes do deploy |
| R-03 | Qualquer usuário autenticado obtém o segredo de qualquer cliente, por não haver vínculo entre usuário e cliente | Alta | Alto | Aceito conscientemente nesta fase (`[09:37] Sofia`); entra na pauta da revisão de segurança |
| R-04 | Cliente não implementa deduplicação e processa a mesma notificação duas vezes | Média | Médio | Identificador estável em toda notificação; documentação destacada no portal, conforme `[09:26] Marcos` |
| R-05 | Falhas se acumulam sem ninguém perceber, porque não há alerta nesta fase | Média | Médio | Toda falha definitiva fica registrada na tabela de dead letter com motivo e timestamp, consultável a qualquer momento (`[09:18] Diego`); o endpoint de replay recoloca o evento em fila quando alguém olha (`[09:35] Diego`) |
| R-06 | Cliente cadastra endereço errado e só descobre no dia seguinte | Média | Baixo | Histórico de entregas mostra a falha desde a primeira tentativa; validação de formato na criação |
| R-07 | Segredo vaza em log da aplicação — já aconteceu com um cliente | Baixa | Alto | Segredo por cadastro, o que confina o vazamento a um cliente (`[09:21] Sofia`); segredo fora dos logs, pela lista de campos ocultados em `src/shared/logger/index.ts:4-11`; rotação pela API, com a credencial anterior válida por 24 horas para o cliente migrar |
| R-08 | Rajada de mudanças inunda o sistema de um cliente | Baixa | Médio | Aceito nesta fase; o histórico de entregas fornece o dado para decidir se vale limitar |

## Critérios de aceitação

| # | Critério | Requisito coberto |
|---|---|---|
| CA-01 | Um cadastro criado passa a receber notificação na próxima mudança de status assinada | RF-01, RF-03, RF-09 |
| CA-02 | Mudança de status não assinada pelo cadastro não gera notificação | RF-03 |
| CA-03 | O segredo é devolvido na criação e não aparece íntegro em nenhuma consulta posterior | RF-02, RNF-07 |
| CA-04 | O cliente consegue validar a assinatura com o segredo recebido | RF-10 |
| CA-05 | Endereço `http` é recusado na criação | RNF-06 |
| CA-06 | Cliente indisponível por duas horas recebe a notificação quando volta | RF-11, RNF-04 |
| CA-07 | Cliente permanentemente fora do ar tem a notificação registrada como falha definitiva, sem perda | RF-11, RF-13 |
| CA-08 | O histórico mostra cada tentativa com resultado, resposta e tempo | RF-12 |
| CA-09 | Rotação mantém as duas credenciais válidas por 24 horas | RF-08 |
| CA-10 | Reprocessamento só funciona para administrador e fica registrado com o autor | RF-13, RF-14, RNF-11 |
| CA-11 | Três mudanças seguidas no mesmo pedido chegam na ordem correta | RNF-09 |
| CA-12 | Falha ao registrar o evento impede a mudança de status | RNF-02 |
| CA-13 | Nenhum endpoint existente muda de contrato ou comportamento | RNF-12, OBJ-03 |
| CA-14 | Cadastro desativado para de receber sem ser apagado | RF-07 |
| CA-15 | Cliente que passa de 10 segundos para responder tem a tentativa contada como falha e entra na retentativa | RNF-08, RF-11 |
| CA-16 | Notificação acima de 64 KB não é enviada; nada chega cortado ao cliente | RNF-05 |
| CA-17 | A mesma notificação entregue duas vezes chega com o mesmo identificador, e o cliente descarta a repetida | RNF-03 |
| CA-18 | Cliente com mais de um cadastro identifica, em cada notificação, qual deles a originou | RF-01, RF-12 |

> **Origem dos quatro últimos.** CA-15 vem do prazo que Diego fixou em `[09:42]`: *"Cliente lento que não
> responde em 10s a gente trata como falha e marca pra retry"*. CA-16 junta a escolha de Sofia em `[09:23]` —
> *"Eu sou a favor de erra"* — ao teto que Diego propôs em `[09:24]`. CA-17 vem de `[09:25] Diego`; CA-18, de
> `[09:44] Sofia`.

## Estratégia de testes e validação

**Automatizado.** Os trinta critérios técnicos do [FDD](FDD.md) cobrem os critérios acima. A suíte do projeto é
de integração real contra o banco, então os testes exercitam a transação de verdade — inclusive o caso em que
a falha ao registrar o evento derruba a mudança de status, que é o comportamento mais importante e o mais
fácil de quebrar sem perceber.

**Revisão de segurança.** Dois dias úteis reservados antes do deploy, conforme Sofia pediu em `[09:46]`. Três
pontos vão à pauta: guarda do segredo em repouso, escopo da assinatura, e a ausência de vínculo entre usuário e
cliente. Os dois primeiros não foram tratados na reunião; o terceiro foi aceito conscientemente e merece
reavaliação.

**Validação com cliente.** Três coisas precisam ser confirmadas com quem vai receber, antes do deploy: que a
verificação de assinatura do lado dele funciona, que o tempo de resposta real dele cabe no percentil que ainda
falta acordar no OBJ-01, e que ele descarta a notificação repetida em vez de processá-la de novo. Marcos é o
dono dessa conversa, pelo que assumiu em `[09:47]` e `[09:49]`.

**O que não será validado nesta fase.** Comportamento sob rajada, porque o limite de taxa está fora de escopo;
e comportamento com mais de um consumidor, porque a ordem só é garantida com um.

**Teste de carga fica de fora, e a razão importa.** A reunião não estabeleceu volume esperado de eventos nem
meta de vazão. Não há uma linha sobre isso na fita, em quase uma hora de conversa. Sem alvo, o teste mede
contra um número que nós mesmos escolhemos, e esse número vira compromisso na primeira vez que alguém lê o
relatório fora de contexto. Os parâmetros de dimensionamento que existem — o intervalo do ciclo de leitura e o
tamanho do lote — vieram de folga contra a promessa de latência, e não de capacidade medida. Fica para quando
houver volume observado e alguém disposto a assumir um alvo.

---

**Rastreabilidade.** A origem de cada item deste documento está em [`TRACKER.md`](TRACKER.md). As decisões
técnicas que sustentam os requisitos estão em [`adrs/`](adrs/); a proposta, em [`RFC.md`](RFC.md); e o
detalhamento de implementação, em [`FDD.md`](FDD.md).
