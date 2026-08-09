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
| **Subagentes em paralelo** | Duas frentes que não caberiam numa janela de contexto só: o mapeamento técnico do OMS e a análise competitiva dos forks |
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

**4. Comparar com quem já resolveu.** O repositório base tem 151 forks. Uma varredura da árvore git
identificou os **131 que de fato concluíram o desafio** — tamanho de repositório era um proxy ruim, e vários
dos melhores não apareciam por ele. Doze foram lidos a fundo por subagentes, decisão a decisão. Isso deu
duas coisas: uma régua de qualidade com números por documento e uma lista de armadilhas observadas, não
imaginadas.

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

## Estado da entrega

- [x] Base do processo — `progress.md` e contrato de fatos
- [ ] ADRs
- [ ] RFC
- [ ] FDD
- [ ] PRD
- [ ] Tracker
- [ ] README final com prompts, iterações e guia de leitura
