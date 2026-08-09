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
commit único no fim. A pilha cobra um preço na hora de mergear, e ele está na iteração 8.

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

## O FDD, e as três decisões que a reunião não sabia que precisava tomar

O [FDD](docs/FDD.md) é o oposto do RFC: aqui os valores importam, e omiti-los seria inútil para quem vai codar.
Dez contratos HTTP com exemplo de requisição e resposta, os quatro models Prisma, a matriz de erros, cinco
diagramas, trinta critérios de aceite e um runbook.

Escrever o detalhe expôs três lugares onde a reunião fechou o quê sem perceber uma consequência.

**O teto de payload podia derrubar a mudança de status.** A Sofia foi categórica em `[09:23]`: evento grande
demais, *"eu sou a favor de erra"*. Mas com o payload materializado na inserção, o tamanho já é conhecido
**dentro da transação** — e errar ali significa fazer rollback de uma mudança de status legítima por causa de
um webhook. As três saídas possíveis estão tabeladas no documento; a escolhida grava o evento e o manda direto
para a fila de mortos na hora da entrega. Honra o "erra" da Sofia, preserva a atomicidade que o Bruno exigiu
em `[09:40]`, e não deixa a feature nova derrubar a operação central do sistema.

**Um mesmo código de erro ia acumular dois significados.** `WEBHOOK_ENDPOINT_INACTIVE` serve tanto como
resposta `409` de um replay recusado quanto como motivo de falha gravado na fila de mortos. São eventos
diferentes em superfícies diferentes. A matriz de erros ficou dividida em dois vocabulários — contrato HTTP e
motivo de falha registrado — com a regra de que nenhum código vive nos dois papéis sem que isso esteja escrito.

**Bloquear rotação de segredo durante a janela de convivência pareceria seguro e seria um erro.** O cenário que
justifica rotacionar é credencial vazada. Bloquear por 24 horas desativaria exatamente o caminho de emergência
que a funcionalidade existe para oferecer. Rotacionar dentro da janela é permitido, e o runbook trata o caso.

O segundo exemplo que o Bruno deu em `[09:28]` também não sobreviveu: `WEBHOOK_INVALID_URL` some, porque a
Sofia já tinha classificado a exigência de `https` como validação de schema em `[09:23]` — e validação de
schema produz `VALIDATION_ERROR`. Junto com `WEBHOOK_SECRET_REQUIRED`, são dois dos três exemplos dele que a
própria reunião invalidou depois. Ambos ficam documentados como ausência, com o que os faria voltar.

## O PRD, e o objetivo que ficou sem número

O [PRD](docs/PRD.md) vem por último entre os grandes documentos, e por isso é o mais curto: com as decisões,
a proposta e a especificação prontas, ele vira consolidação. **2.912 palavras** — abaixo das 3.500 que a régua
em [`progress.md`](progress.md) tinha fixado como piso, e com 14 requisitos funcionais onde o alvo pedia 16.
O alvo foi escrito antes de existir texto, e errou: o que sobrou depois do corte por densidade não pedia mais
espaço. A régua registra o desvio em vez de ser reescrita para caber no resultado.

Duas coisas dele valem o registro.

**A meta de latência não pôde ser um teto.** O número acordado é o do Marcos em `[09:02]`: abaixo de dez
segundos. Mas a aritmética que apareceu no RFC mostra que uma entrega bem-sucedida pode passar disso sem que
nada tenha falhado. Escrever "abaixo de 10 segundos" como compromisso absoluto seria prometer o que o desenho
não sustenta. A meta virou percentil sobre a primeira tentativa, com o número acordado preservado e a
**formulação marcada como derivação nossa** — não como decisão da reunião. O percentil exato fica pendente de
acordo com os clientes.

**Um dos seis objetivos não tem número, de propósito.** A redução da carga de consulta que os clientes fazem
hoje foi o motivo declarado do pedido, mas ninguém mede esse volume e a reunião não estabeleceu linha de base
nem alvo. Qualquer número ali seria um compromisso que ninguém assumiu. Ficou como objetivo qualitativo, com a
ausência justificada por escrito — é mais útil que uma meta inventada, e mais honesto.

A tabela de fora de escopo tem sete itens, cada um com **uma única classificação**: adiado tem fase prevista,
descartado não tem, em observação tem gatilho. O mesmo item não aparece classificado de formas diferentes em
documentos diferentes — o que é fácil de acontecer quando o PRD, o RFC e o FDD tratam o mesmo assunto.

## O Tracker, e a pergunta que ninguém faz

O [tracker](docs/TRACKER.md) responde *de onde veio cada item*: 180 linhas, 78% ancoradas numa fala com
timestamp, o resto em caminho de código real. Foi **gerado a partir das tabelas dos próprios documentos**, que
já carregam a coluna de origem — transcrever à mão 180 linhas é como se introduz divergência.

Ele tem três coisas além do formato pedido.

**Uma coluna de seção.** A tabela padrão diz que `PRD-RF-09` veio de `[09:06] Diego`. Ela não diz onde
encontrar `PRD-RF-09`. Com a seção de destino, a verificação funciona nos dois sentidos: da fala para o
documento e do documento para a fala.

**Uma tabela separada para o que não tem âncora.** As trinta decisões de implementação que a reunião não tomou
não entram na tabela principal — elas ficam numa seção própria, cada uma com procedência: padrão de mercado
verificado, derivação do código, ou escolha de quem produziu a documentação. Misturá-las daria a elas uma
aparência de origem que não têm.

**Um índice reverso.** A tabela principal responde de onde veio cada item. O índice reverso responde a
pergunta inversa, que é a que expõe omissão: **o que da reunião ficou de fora**. São 54 timestamps distintos
na fita; 43 têm pelo menos um item do pacote apontando para eles. Os 11 restantes estão listados um a um, com
o motivo — abertura de bloco, entrada na call, passagem de palavra, encerramento. A ausência fica verificável
em vez de assumida.

E a fórmula da cobertura vem escrita ao lado do número. Percentual sem fórmula é número que ninguém consegue
auditar.

## Prompts customizados

Prompt vago produz documento vago. Os quatro abaixo são os que mais mudaram o resultado. Cada um vem precedido
do modo de falha que ele existe para evitar — porque prompt sem problema declarado é prompt que ninguém sabe
avaliar.

### 1 · Mapeamento do código

**Problema que resolve:** documento que cita `order.service.ts` sem saber o que tem dentro. A IA parafraseia a
transcrição e inventa o resto; o revisor não tem como perceber sem abrir o arquivo.

```text
Preciso de um MAPA TÉCNICO PRECISO E VERIFICÁVEL para escrever um Feature Design
Document. Exatidão é crítica: todo caminho de arquivo, nome de classe, nome de
método e nome de campo que você reportar precisa existir de fato. Cite caminho:linha.

Investigue e reporte, com trechos curtos de código quando ajudar:

1. O método changeStatus: assinatura completa, o que roda dentro da transação,
   em que ordem, quais tabelas toca. Onde exatamente um publishWebhookEvent(tx, ...)
   se encaixaria.
2. A classe AppError: construtor, propriedades. Liste TODAS as subclasses existentes
   e seus códigos. Qual o padrão exato a seguir para criar erros novos.
3. O middleware de erro: formato exato do JSON de resposta.
[...]

Formato do retorno: markdown por seção, com caminhos e nomes exatos. Inclua no fim
uma lista "GANCHOS DE INTEGRAÇÃO" com os pontos de acoplamento mais relevantes, cada
um com arquivo:linha e uma frase sobre como estender.

Não invente nada; se algo não existir, diga explicitamente "não existe".
```

A última linha é a que mais rende. Foi ela que produziu os dois achados que viraram restrição de projeto: não
existe nenhum código de webhooks no repositório, e não há cliente HTTP nas dependências. Um prompt otimista
teria enterrado os dois.

### 2 · Redação de ADR

**Problema que resolve:** ADR que é prosa elogiosa da decisão tomada, sem alternativa real, sem consequência
negativa e sem nada que possa ser conferido.

```text
Escreva UM Architecture Decision Record em português do Brasil.

## Leia ANTES de escrever, nesta ordem
1. docs/processo/FATOS.md — contrato de fatos canônicos. Fonte única de verdade.
   Nenhum número, nome de tabela, caminho ou header pode divergir dele.
2. TRANSCRICAO.md — para extrair citações LITERAIS. Toda citação entre aspas tem
   que ser cópia exata, com o [hh:mm] Nome correto. Uma citação inventada invalida
   o documento.
3. Os arquivos de código mencionados no escopo. LEIA de verdade, confira a linha.

## Regras de qualidade
1. Tamanho: 8 a 12 KB. Acima disso o retorno é negativo. Densidade, não volume.
2. Nunca 100% prosa. Pelo menos 2 tabelas.
3. Altura de ADR: registra UMA decisão. NÃO é FDD (nada de payload, colunas,
   status code). NÃO é PRD (nada de valor de negócio).
4. Não duplique outro ADR. Use a seção "O que este ADR não cobre" com link.
5. Consequências negativas de verdade. Se só houver positivas, você não pensou
   o suficiente. Cada limitação precisa de gatilho de reabertura nomeado.
6. Rotule o que não veio da reunião como decisão derivada, com a procedência.
7. Não invente: SLA numérico, ferramenta de observabilidade, custo, headcount,
   data absoluta da reunião, nome de pessoa fora dos 5 participantes.
8. Não reintroduza o que foi descartado. Podem aparecer só como exclusão.
```

A regra 5 é a que separa ADR de release note. A 7 é uma lista negra concreta, não um pedido genérico de "não
alucine" — e por isso funciona.

### 3 · Auditoria mecânica

**Problema que resolve:** revisão por releitura não pega citação levemente parafraseada nem número de linha
errado. O olho humano lê o que espera ler.

```text
Audite os documentos. Seja implacável. Cada verificação é mecânica:

1. CONTRADIÇÕES. Extraia de cada documento todo número e nome próprio (tentativas,
   intervalos, timeout, tamanho de payload, nomes de tabela, nomes de header).
   Compare entre documentos e contra o contrato de fatos. Qualquer divergência é bug.
2. CITAÇÕES. Para CADA trecho entre aspas atribuído a um participante, confirme que
   o texto é literal e que o [hh:mm] Nome bate.
3. CAMINHOS. Para cada arquivo:linha citado, confirme que o arquivo existe e que a
   linha contém o que o documento afirma.
4. ESCOPO. Confirme que nenhum item descartado aparece como requisito.
5. ALTURA. Nenhum ADR pode conter payload de exemplo, matriz de erros completa ou
   argumento de valor de negócio.
6. INVENÇÃO. Procure por percentual de SLA, ferramenta de observabilidade, volume
   por período, custo, nome de pessoa fora dos 5 participantes.
```

Rodei isso como script, não como conversa. Os resultados aparecem na seção seguinte.

### 4 · Extração do contrato de fatos

**Problema que resolve:** o PRD dizer 5 tentativas e o FDD descrever 6. Sem uma base única, cada documento
reinterpreta a fita — e as reinterpretações divergem.

```text
Da transcrição e do código, extraia uma base de fatos com identificadores estáveis,
separada em seis grupos:

F — decisões FECHADAS na reunião, cada uma com o timestamp e o nome de quem fechou
X — itens DESCARTADOS ou adiados, com a fala que os descartou
A — ALTERNATIVAS consideradas e recusadas, com o trade-off que motivou o descarte
C — fatos VERIFICADOS no código, com caminho:linha
D — DIVERGÊNCIAS entre o que a fala afirma e o que o código faz
Q — questões deixadas EM ABERTO, com dono

Regra: se um item não couber em nenhum grupo, ele não tem origem e não entra em
documento nenhum. Liste esses separadamente, como "não inventar".
```

O grupo `D` foi o mais produtivo. Ele obriga a confrontar cada afirmação técnica da reunião com o arquivo — e
foi assim que apareceu que a transação não decrementa estoque em toda mudança de status, e que o token não
carrega identificador de cliente.

## Iterações e ajustes

Oito correções materiais. Nenhuma veio de reler o texto. Todas vieram de conferir o que estava escrito contra
alguma outra coisa: o código, a fita, ou o próprio repositório.

**1 · A assinatura extrapolava a fala, e voltou atrás.** Decidi assinar `{timestamp}.{corpo}`, ao padrão
Stripe, porque isso torna a proteção contra reenvio realmente eficaz — sem o timestamp assinado, ele é
adulterável e a defesa não vale nada. Revisei contra a regra de rastreabilidade estrita e a decisão não
sobreviveu: `[09:22] Sofia` diz *"HMAC-SHA256 sobre o corpo do request"*, e nada mais. A assinatura voltou a
cobrir só o corpo, e a fragilidade virou `Q07`, com dona e prazo na revisão de segurança. **Documentar a
limitação é honesto; resolvê-la inventando uma decisão não é.**

**2 · "5 tentativas" não fechava com cinco intervalos.** Cinco chamadas HTTP consomem quatro esperas. A
transcrição decide os dois números em minutos diferentes e eles são incompatíveis se lidos ao pé da letra. O
desempate estava na própria fala do Diego — *"quase 15 horas entre primeira falha e última tentativa"* — que
só fecha se os intervalos forem retentativas posteriores à entrega inicial. Virou **5 retentativas, 5
intervalos, 6 chamadas HTTP**, com a conta publicada num lugar só e o contrato de fatos atualizado. Depois
propaguei o termo pelo título do ADR-003, pelo ADR-005 e pelo índice, porque metade dos documentos ainda dizia
"tentativas".

**3 · Dois ADRs viraram um, e sobrou espaço para uma decisão que faltava.** Eu tinha separado retry e dead
letter em ADRs distintos. Separar fatia uma decisão em vez de acrescentar outra — e o enunciado lista as duas
como uma linha só. Fundi, e usei o slot liberado para o controle de acesso, que é a decisão de `[09:36] Sofia`
e que ainda cita `requireRole` no código.

**4 · O teto de payload ia derrubar a mudança de status.** Escrevendo o FDD, percebi que com o payload
materializado na inserção o tamanho é conhecido **dentro da transação** — e errar ali faz rollback de uma
mudança de status legítima. A reunião decidiu "erra" sem enxergar essa consequência. Tabelei as três saídas e
adotei a que grava o evento e o descarta na entrega.

**5 · Um agente morreu no meio e seis arquivos sobreviveram.** Disparei oito subagentes para escrever um ADR
cada. Todos falharam por limite de sessão e o orquestrador reportou zero produzidos. Ao conferir o disco,
**seis ADRs estavam lá, completos** — eles falharam ao devolver o resumo, não ao escrever. Escrevi os dois
restantes eu mesmo. Lição barata: não confie no relatório do orquestrador, confira o artefato.

**6 · O tracker nasceu com 52% de âncora na transcrição.** O gerador inferiu a fonte pelo texto da linha, e
marcou como código muita coisa que nasce na fita. Reescrevi com um mapa explícito de âncora por item e subiu
para 78%. O gerador automático estava errado de um jeito que uma leitura casual aprovaria — o número só
apareceu porque eu o medi.

**7 · Uma regra derivada quase desativou o caso de uso da funcionalidade.** Ia bloquear rotação de segredo
durante a janela de convivência de 24 horas. Parece prudente. Mas o cenário que justifica rotacionar é
credencial vazada — e bloquear por 24 horas desativa exatamente o caminho de emergência que a rotação existe
para oferecer. Toda regra derivada passou a levar o teste: *ela quebra algum cenário declarado no PRD?*

**8 · A pilha de PRs entregou os documentos um degrau antes da `main`.** Cada PR tinha como base a branch do
anterior. O primeiro deles foi mergeado na `main` sete horas antes dos outros quatro — e quando o FDD chegou
na branch do RFC, a `main` já tinha se servido dela. O defeito passou despercebido porque os arquivos
existiam: `docs/FDD.md` estava no lugar certo com **70 bytes**, o stub que veio do repositório do desafio.
Descobri pelo aviso de push pendente do GitHub, não por revisão. Um oitavo PR levou os quatro documentos
que faltavam. **Pilha se mergeia do topo para a base**, ou cada PR precisa ser reapontado conforme sua base
entra.

## Como navegar a entrega

```text
.
├── README.md                    ← você está aqui: o processo
├── progress.md                  ← plano, régua e checklist de verificação
├── TRANSCRICAO.md               ← a fonte (não alterado)
└── docs/
    ├── PRD.md                   ← por que e o quê        · produto
    ├── RFC.md                   ← como propomos resolver · arquitetura
    ├── FDD.md                   ← como construir         · implementação
    ├── TRACKER.md               ← de onde veio cada coisa
    ├── adrs/
    │   ├── README.md            ← índice, grafo e o que não virou ADR
    │   └── ADR-001 … ADR-008    ← uma decisão por arquivo
    └── processo/
        └── FATOS.md             ← base de fatos (artefato de processo, não entregável)
```

**Ordem de leitura sugerida**, dependendo do que você quer:

| Se você quer | Leia nesta ordem |
|---|---|
| Entender a feature em dez minutos | `RFC.md` → os dois diagramas C4 → `PRD.md` |
| Começar a implementar | `FDD.md` inteiro → os ADRs conforme o FDD os referencia |
| Auditar se algo foi inventado | `TRACKER.md` → escolha uma linha → abra a fala em `TRANSCRICAO.md` |
| Entender por que foi decidido assim | `docs/adrs/README.md` → o grafo → o ADR que interessa |
| Ver o que ficou de fora e por quê | `PRD.md` seção 4 → `RFC.md` questões em aberto → índice dos ADRs |

Os documentos não se repetem. Se você encontrar a mesma informação em dois deles com nível de detalhe
parecido, é defeito — a fronteira está descrita em cada documento, na seção que diz o que ele **não** cobre.

## Cobertura dos critérios de aceite

| Critério do enunciado | Onde é atendido |
|---|---|
| PRD com as 12 seções obrigatórias | `docs/PRD.md`, seções 1 a 12 |
| PRD com ao menos 8 requisitos funcionais | 14 requisitos, seção 5 |
| PRD com objetivo e meta quantitativa | 6 objetivos, 5 com meta numérica, seção 4 |
| PRD com 2+ itens fora de escopo descartados na reunião | 7 itens, seção 4, cada um com a fala que o descartou |
| PRD com 2+ riscos com probabilidade, impacto e mitigação | 8 riscos, seção 9 |
| RFC com as 8 seções obrigatórias | `docs/RFC.md` |
| RFC com 2+ alternativas descartadas e trade-off | 8 alternativas, cada uma com origem e ADR |
| RFC com 2+ questões em aberto | 6 questões, cada uma com dono e gatilho |
| RFC referenciando 2+ ADRs com link | os 8 ADRs |
| FDD com as 11 seções obrigatórias | `docs/FDD.md`, seções 1 a 14 |
| FDD com 4+ endpoints com payload e status code | 10 contratos, seção 7 |
| FDD com matriz de erros `WEBHOOK_*` | 14 códigos ativos, seção 8, mais 2 documentados como ausentes |
| FDD com "Integração com o sistema existente" e 4+ caminhos reais | 21 caminhos, seção 11 |
| FDD com observabilidade citando métricas, logs e tracing | seção 10, sem nomear ferramenta |
| 5 a 8 ADRs no formato `ADR-NNN-titulo.md` | 8 arquivos em `docs/adrs/` |
| Cada ADR com Status, Contexto, Decisão, Alternativas, Consequências | todos os 8, mais duas seções extras |
| Cobrir 5 das 6 decisões principais | as 6, mais snapshot e controle de acesso |
| 1+ ADR referenciando código real | ADR-006 e ADR-008 |
| Tracker no formato de tabela definido | `docs/TRACKER.md`, seção 1 |
| Tracker com 80%+ dos itens identificáveis cobertos | 100%, 107 de 107 — a fórmula está na seção 2 do tracker |
| Tracker com 70%+ de fonte `TRANSCRICAO` com timestamp | 78%, 140 de 180 |
| Tracker com 5+ linhas de fonte `CODIGO` | 40 linhas sobre 19 caminhos |
| README com as 6 seções obrigatórias | este arquivo |
| README com 1+ ferramenta de IA listada | 4, na segunda seção |
| README com 2+ prompts customizados | 4 prompts |
| README com 2+ iterações concretas | 8 iterações |
| Nenhum requisito, decisão ou restrição contradiz a transcrição ou o código | conferido por script: toda citação é literal, todo `[hh:mm] Nome` existe na fita, nenhum número diverge entre documentos, nenhum item descartado reaparece como requisito |
| Nenhum arquivo de código inexistente citado | verificado por script; os únicos ausentes são os 10 que a feature cria, marcados como `criar` |
| `src/`, `prisma/`, `tests/` e configuração intocados | nenhum arquivo de aplicação alterado |

## Estado da entrega

- [x] Base do processo — `progress.md` e contrato de fatos
- [x] ADRs — 8 decisões, com índice e justificativa do que ficou de fora
- [x] RFC — proposta técnica com C4 de contexto e de contêineres
- [x] FDD — implementação, com contratos, modelos, matriz de erros e runbook
- [x] PRD — problema, escopo, requisitos, métricas e riscos
- [x] Tracker — 180 itens, com índice reverso da transcrição
- [x] README final — prompts, iterações, guia de leitura e matriz de cobertura
- [x] Integração na `main` — os quatro documentos que a pilha de PRs deixou um degrau atrás
