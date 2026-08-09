# Da Reunião ao Documento — Design Docs Gerados por IA

Pacote de documentação técnica da feature **Sistema de Webhooks de Notificação de Pedidos**, produzido a
partir da transcrição de uma reunião técnica e do código de um Order Management System em produção.

Este README documenta **como o pacote foi produzido**. O enunciado original do desafio está no
[repositório base](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia).

---

## Sobre o desafio

Uma empresa que opera um OMS recebeu o pedido de três clientes B2B: eles querem ser notificados quando o
status dos pedidos deles muda, em vez de ficar batendo em `GET /orders` de tempos em tempos. A decisão
técnica foi tomada numa call de 55 minutos entre tech lead, PM, dois engenheiros e uma engenheira de
segurança. Nada foi registrado além da gravação — `TRANSCRICAO.md`, 155 falas, 54 timestamps.

A tarefa é transformar essa fita em um pacote de design docs acionável: PRD, RFC, FDD, um conjunto de ADRs,
um tracker de rastreabilidade e este README. A restrição que define o exercício é que **nada pode ser
inventado**: cada requisito, decisão e restrição precisa ser rastreável a uma fala da reunião ou a um
arquivo do repositório. E nem tudo que foi dito vira requisito — a reunião descarta ideias, adia outras e
deixa pontos em aberto. Identificar o que **não** entra vale tanto quanto identificar o que entra.

A IA é a ferramenta de produção. O trabalho humano é o de maestro: decidir o que precisa existir, formular
os prompts, revisar com desconfiança o que volta e corrigir até o pacote ficar consistente.

## Ferramentas de IA utilizadas

| Ferramenta | Papel |
|---|---|
| **Claude Code (Opus 5)** | Condução do trabalho: leitura do código, análise da transcrição, produção dos documentos e orquestração dos subagentes |
| **Subagentes em paralelo** | Duas frentes que não caberiam numa janela de contexto só: o mapeamento técnico do OMS e a redação simultânea dos ADRs |
| **Skill `humanize-pt-br`** | Revisão da prosa. Aplicada nas seções narrativas; tabelas, contratos e blocos de código ficam intocados por decisão da própria skill |
| **Busca e leitura web** | Verificação de padrão de mercado para os pontos que a reunião não especificou — documentação de webhooks da Stripe, do GitHub e da spec Standard Webhooks |

## Workflow adotado

A ordem importa mais do que parece. Produzir o PRD primeiro, por exemplo, garante um documento genérico:
sem as decisões técnicas na mão, não há como saber o que é escopo e o que é consequência.

**1. Ler o código antes de escrever qualquer coisa.** Um subagente varreu o repositório e devolveu um mapa
com caminhos e números de linha: onde `changeStatus` abre a transação, o que roda dentro dela e em que
ordem, quais classes de erro existem, o que o middleware de erro já trata, o formato exato dos envelopes de
resposta. Conferi pessoalmente os dois pontos mais carregados antes de tratar qualquer coisa como fato.

**2. Montar um contrato de fatos canônicos.** Antes de redigir, extraí da fita e do código uma base única
com identificadores estáveis — decisões, descartes, alternativas, fatos de código, divergências e questões
em aberto. Todo documento consome dessa base. É o que impede o PRD de dizer 5 tentativas e o FDD de
descrever 6, que é a contradição mais comum neste desafio. A base está em
[`docs/processo/FATOS.md`](docs/processo/FATOS.md).

**3. Fixar a fronteira entre os documentos.** Cada fato tem um dono. O PRD é dono da dor do cliente; o
ADR-003 é dono da curva de backoff; o FDD é dono da matriz de erros. Os outros documentos referenciam por
link em vez de reproduzir. Conteúdo duplicado é sintoma de coisa na altura errada.

**4. Definir a régua antes de escrever.** Alvo de tamanho e densidade para cada documento, e uma lista de
modos de falha para vigiar durante a produção: contradição numérica entre documentos, caminho de arquivo
inexistente, item descartado na reunião reaparecendo como requisito, RFC que incha até virar FDD. Ter a lista
antes muda o que se procura na revisão — sem ela, o revisor só encontra o que já esperava.

**5. Resolver as ambiguidades antes de redigir.** A reunião fecha o *quê* mas deixa buracos no *como*. O
formato do valor da assinatura, o tamanho do lote do worker, o nome de duas das quatro tabelas, o que
acontece quando o worker morre no meio de um envio — nada disso foi dito. Cada buraco virou uma decisão
explícita, carimbada com a procedência: padrão de mercado verificado, derivação de um padrão do código, ou
chamada de quem produziu a documentação. **Nenhuma pode se passar por fala da reunião.**

**6. Produzir em PRs empilhados, um por documento.** Cada PR sai da branch do anterior, na ordem em que os
documentos se sustentam: ADRs, RFC, FDD, PRD, Tracker e README. Cada item do plano em
[`progress.md`](progress.md) é um commit. Este README cresce a cada PR, um bloco por documento entregue —
foi assim que ele foi escrito, e é por isso que o histórico do repositório mostra a evolução em vez de um
commit único no fim.

## Os ADRs, e o que a produção deles revelou

Oito decisões, uma por arquivo, em [`docs/adrs/`](docs/adrs/). Cobrem as seis que o desafio exige, mais o
snapshot do payload — fechado em `[09:52]`, já com metade da sala fora da call — e o controle de acesso.

O [índice](docs/adrs/README.md) traz duas coisas que o formato normalmente não tem. Um grafo de dependência
entre as decisões, porque elas não são independentes: o retry só existe porque o worker pode falhar, e o
replay da dead letter só precisa de autorização porque a dead letter existe. E uma tabela do que
**deliberadamente não virou ADR**, com a fala que desqualificou cada item. A reunião fez esse trabalho
sozinha em dois casos: a Sofia, ao propor o HTTPS obrigatório, emenda *"isso na verdade nem é decisão
arquitetural, é só uma validação no schema Zod"* (`[09:23]`), e a Larissa, ao fechar o teto de payload,
diz *"não vejo como decisão arquitetural separada, é só requisito não funcional"* (`[09:24]`).

Três coisas apareceram só porque a produção passou por leitura de código e conferência mecânica, e não por
releitura da transcrição:

**"5 tentativas" não fecha com cinco intervalos de backoff.** A reunião decide os dois em `[09:15]` e
`[09:17]`, e eles são incompatíveis se lidos ao pé da letra: cinco chamadas HTTP consomem quatro esperas, não
cinco. O desempate está na própria fala do Diego, que descreve *"quase 15 horas entre primeira falha e última
tentativa"* — a contagem começa na primeira **falha**, o que só fecha se os cinco intervalos forem
retentativas posteriores à entrega inicial. `1m + 5m + 30m + 2h + 12h = 14h36min`. O número canônico virou
**5 retentativas, 5 intervalos, 6 chamadas HTTP**, com a aritmética publicada num lugar só e referenciada nos
demais.

**A classe de erro do projeto tem uma pegadinha.** A propriedade pública de `AppError` chama-se `errorCode`,
não `code`; quem mapeia para a chave `code` do JSON é o middleware. E três das subclasses — `NotFoundError`,
`UnauthorizedError` e `ForbiddenError` — não aceitam código customizado no construtor, o que impede que
`WEBHOOK_NOT_FOUND` seja construído da forma óbvia. Nada disso aparece na transcrição.

**Um código de erro nasceu órfão.** O Bruno nomeia `WEBHOOK_SECRET_REQUIRED` em `[09:28]`; três minutos
depois, em `[09:31]`, o Marcos fecha que a secret é gerada por nós. O caso de uso evapora entre uma fala e
outra. Em vez de manter o nome e inventar semântica para ele, o ADR-006 documenta a subtração — onde poderia
ocorrer, por que não ocorre, e o que faria o código voltar a fazer sentido.

## O RFC, e a regra que o manteve curto

O [RFC](docs/RFC.md) é o documento com o teto mais apertado do pacote: duas a quatro páginas. A tentação é
gastar esse espaço explicando a solução, e aí ele vira um FDD mal disfarçado.

A regra que usei para segurar: **o RFC descreve formas e omite valores.** "Polling curto", não "2 segundos".
"Intervalos crescentes até um teto", não a curva. "Segredo por endpoint com janela de convivência", não 24
horas. Cada número tem dono — um ADR ou o FDD — e o RFC referencia em vez de repetir. Isso resolve dois
problemas de uma vez: mantém o documento na altura de arquitetura e elimina a chance de um número divergir
entre documentos, porque ele só existe num lugar.

Abri **uma exceção deliberada**, na seção de riscos: a aritmética da latência. Somando a espera do polling ao
tempo de resposta de um cliente que chegue perto do limite de timeout, uma entrega **bem-sucedida** ultrapassa
os dez segundos prometidos ao cliente em `[09:02]`, sem que nada tenha falhado. Aqui o número *é* o risco, e
esconder o valor esconderia o problema. A consequência sobe para o PRD: a métrica tem que ser percentil sobre
a primeira tentativa, não teto absoluto.

Os dois diagramas C4 — contexto e contêineres — são o que mostra em vinte segundos o que a prosa leva três
parágrafos para dizer: que são **dois processos sobre o mesmo banco**, e que a API nunca faz chamada HTTP de
saída. Validei os dois renderizando localmente antes de commitar, porque diagrama quebrado num documento de
proposta custa mais caro que diagrama nenhum.

## Estado da entrega

- [x] Base do processo — `progress.md` e contrato de fatos
- [x] ADRs — 8 decisões, com índice e justificativa do que ficou de fora
- [x] RFC — proposta técnica com C4 de contexto e de contêineres
- [ ] RFC
- [ ] FDD
- [ ] PRD
- [ ] Tracker
- [ ] README final com prompts, iterações e guia de leitura
