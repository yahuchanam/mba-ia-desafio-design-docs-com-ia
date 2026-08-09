# Tracker de Rastreabilidade

Referência cruzada entre cada item dos documentos e sua origem. Serve para responder, item a item, **de onde
veio isso** — e para expor o que não tem origem, em vez de escondê-lo.

| | |
|---|---|
| **Itens rastreados** | 180 |
| **Fonte `TRANSCRICAO`** | 140 (78%) |
| **Fonte `CODIGO`** | 40 linhas, 19 caminhos distintos |
| **Decisões sem âncora na reunião** | 30, listadas à parte na seção 3 |
| **Cobertura da fita** | 43 dos 54 timestamps distintos — seção 4 |

## Como ler

| Coluna | Conteúdo |
|---|---|
| **ID** | Identificador estável do item. O prefixo indica o documento dono |
| **Documento** | Arquivo onde o item aparece |
| **Tipo** | Natureza do item |
| **Conteúdo** | Resumo de uma linha |
| **Fonte** | `TRANSCRICAO` ou `CODIGO` |
| **Localização** | `[hh:mm] Nome` para a fita; `caminho:linha` para o código |
| **Seção** | Onde encontrar o item dentro do documento — torna a verificação bidirecional |

Itens **derivados** — decididos fora da reunião para o FDD ficar acionável — **não entram na tabela
principal**. Eles vivem na seção 3, com a procedência de cada um. Misturá-los aqui daria a eles uma
aparência de origem que não têm.

## 1. Itens rastreados

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização | Seção |
|---|---|---|---|---|---|---|
| `PRD-CA-01` | `docs/PRD.md` | Critério de Aceitação | Um cadastro criado passa a receber notificação na próxima mudança de status assinada | `TRANSCRICAO` | `[09:06] Diego` | 10 · Critérios de aceitação |
| `PRD-CA-02` | `docs/PRD.md` | Critério de Aceitação | Mudança de status não assinada pelo cadastro não gera notificação | `TRANSCRICAO` | `[09:34] Bruno` | 10 · Critérios de aceitação |
| `PRD-CA-03` | `docs/PRD.md` | Critério de Aceitação | O segredo é devolvido na criação e não aparece íntegro em nenhuma consulta posterior | `TRANSCRICAO` | `[09:31] Marcos` | 10 · Critérios de aceitação |
| `PRD-CA-04` | `docs/PRD.md` | Critério de Aceitação | O cliente consegue validar a assinatura com o segredo recebido | `TRANSCRICAO` | `[09:20] Sofia` | 10 · Critérios de aceitação |
| `PRD-CA-05` | `docs/PRD.md` | Critério de Aceitação | Endereço http é recusado na criação | `TRANSCRICAO` | `[09:23] Sofia` | 10 · Critérios de aceitação |
| `PRD-CA-06` | `docs/PRD.md` | Critério de Aceitação | Cliente indisponível por duas horas recebe a notificação quando volta | `TRANSCRICAO` | `[09:16] Diego` | 10 · Critérios de aceitação |
| `PRD-CA-07` | `docs/PRD.md` | Critério de Aceitação | Cliente permanentemente fora do ar tem a notificação registrada como falha definitiva, sem perda | `TRANSCRICAO` | `[09:18] Diego` | 10 · Critérios de aceitação |
| `PRD-CA-08` | `docs/PRD.md` | Critério de Aceitação | O histórico mostra cada tentativa com resultado, resposta e tempo | `TRANSCRICAO` | `[09:34] Marcos` | 10 · Critérios de aceitação |
| `PRD-CA-09` | `docs/PRD.md` | Critério de Aceitação | Rotação mantém as duas credenciais válidas por 24 horas | `TRANSCRICAO` | `[09:21] Sofia` | 10 · Critérios de aceitação |
| `PRD-CA-10` | `docs/PRD.md` | Critério de Aceitação | Reprocessamento só funciona para administrador e fica registrado com o autor | `TRANSCRICAO` | `[09:36] Sofia` | 10 · Critérios de aceitação |
| `PRD-CA-11` | `docs/PRD.md` | Critério de Aceitação | Três mudanças seguidas no mesmo pedido chegam na ordem correta | `TRANSCRICAO` | `[09:12] Diego` | 10 · Critérios de aceitação |
| `PRD-CA-12` | `docs/PRD.md` | Critério de Aceitação | Falha ao registrar o evento impede a mudança de status | `TRANSCRICAO` | `[09:40] Bruno` | 10 · Critérios de aceitação |
| `PRD-CA-13` | `docs/PRD.md` | Critério de Aceitação | Nenhum endpoint existente muda de contrato ou comportamento | `TRANSCRICAO` | `[09:30] Larissa` | 10 · Critérios de aceitação |
| `PRD-CA-14` | `docs/PRD.md` | Critério de Aceitação | Cadastro desativado para de receber sem ser apagado | `TRANSCRICAO` | `[09:21] Bruno` | 10 · Critérios de aceitação |
| `PRD-FE-01` | `docs/PRD.md` | Fora de Escopo | Alerta por e-mail quando o webhook do cliente falha — Adiado — próxima fase | `TRANSCRICAO` | `[09:37] Larissa` | 4 · Fora de escopo |
| `PRD-FE-02` | `docs/PRD.md` | Fora de Escopo | Painel visual para o cliente acompanhar entregas — Descartado desta feature | `TRANSCRICAO` | `[09:40] Larissa` | 4 · Fora de escopo |
| `PRD-FE-03` | `docs/PRD.md` | Fora de Escopo | Recebimento de webhooks enviados pelo cliente — Descartado — direção única | `TRANSCRICAO` | `[09:02] Marcos` | 4 · Fora de escopo |
| `PRD-FE-04` | `docs/PRD.md` | Fora de Escopo | Arquivamento e expurgo de eventos antigos — Adiado | `TRANSCRICAO` | `[09:08] Diego` | 4 · Fora de escopo |
| `PRD-FE-05` | `docs/PRD.md` | Fora de Escopo | Limite de taxa de envio para um mesmo cliente — Em observação | `TRANSCRICAO` | `[09:39] Diego` | 4 · Fora de escopo |
| `PRD-FE-06` | `docs/PRD.md` | Fora de Escopo | Garantia de ordem global entre pedidos diferentes — Adiado | `TRANSCRICAO` | `[09:13] Diego` | 4 · Fora de escopo |
| `PRD-FE-07` | `docs/PRD.md` | Fora de Escopo | Desativação automática de endereço com falha recorrente — Não discutido | `TRANSCRICAO` | `[09:37] Larissa` | 4 · Fora de escopo |
| `PRD-OBJ-01` | `docs/PRD.md` | Objetivo / Métrica | Notificar em tempo percebido como real — meta: Abaixo de 10 segundos, medido como percentil — ver a nota | `TRANSCRICAO` | `[09:02] Marcos` | 4 · Objetivos e métricas |
| `PRD-OBJ-02` | `docs/PRD.md` | Objetivo / Métrica | Não perder evento — meta: Zero | `TRANSCRICAO` | `[09:06] Diego` | 4 · Objetivos e métricas |
| `PRD-OBJ-03` | `docs/PRD.md` | Objetivo / Métrica | Não degradar a operação existente — meta: Zero | `TRANSCRICAO` | `[09:04] Bruno` | 4 · Objetivos e métricas |
| `PRD-OBJ-04` | `docs/PRD.md` | Objetivo / Métrica | Atender os três clientes que pediram — meta: 3 | `TRANSCRICAO` | `[09:00] Marcos` | 4 · Objetivos e métricas |
| `PRD-OBJ-05` | `docs/PRD.md` | Objetivo / Métrica | Entregar dentro do compromisso comercial — meta: 3, com a revisão de segurança inclusa | `TRANSCRICAO` | `[09:46] Larissa` | 4 · Objetivos e métricas |
| `PRD-OBJ-06` | `docs/PRD.md` | Objetivo / Métrica | Reduzir a carga de consulta que os clientes fazem hoje — meta: Sem meta numérica — ver a nota | `TRANSCRICAO` | `[09:00] Marcos` | 4 · Objetivos e métricas |
| `PRD-R-01` | `docs/PRD.md` | Risco | Um defeito no módulo novo derruba a mudança de status, a operação central do sistema — Média/Crítico | `TRANSCRICAO` | `[09:40] Bruno` | 9 · Riscos |
| `PRD-R-02` | `docs/PRD.md` | Risco | A promessa de 10 segundos não se cumpre para clientes que respondem devagar — Alta/Médio | `TRANSCRICAO` | `[09:02] Marcos` | 9 · Riscos |
| `PRD-R-03` | `docs/PRD.md` | Risco | Qualquer usuário autenticado obtém o segredo de qualquer cliente, por não haver vínculo entre usuário e cliente — Alta/Alto | `TRANSCRICAO` | `[09:37] Sofia` | 9 · Riscos |
| `PRD-R-04` | `docs/PRD.md` | Risco | Cliente não implementa deduplicação e processa a mesma notificação duas vezes — Média/Médio | `TRANSCRICAO` | `[09:26] Marcos` | 9 · Riscos |
| `PRD-R-05` | `docs/PRD.md` | Risco | Falhas se acumulam sem ninguém perceber, porque não há alerta nesta fase — Média/Médio | `TRANSCRICAO` | `[09:37] Larissa` | 9 · Riscos |
| `PRD-R-06` | `docs/PRD.md` | Risco | Cliente cadastra endereço errado e só descobre no dia seguinte — Média/Baixo | `TRANSCRICAO` | `[09:34] Marcos` | 9 · Riscos |
| `PRD-R-07` | `docs/PRD.md` | Risco | Segredo vaza em log da aplicação — já aconteceu com um cliente — Baixa/Alto | `TRANSCRICAO` | `[09:22] Diego` | 9 · Riscos |
| `PRD-R-08` | `docs/PRD.md` | Risco | Rajada de mudanças inunda o sistema de um cliente — Baixa/Médio | `TRANSCRICAO` | `[09:38] Diego` | 9 · Riscos |
| `PRD-RF-01` | `docs/PRD.md` | Requisito Funcional | O operador cadastra um endereço de webhook para um cliente | `TRANSCRICAO` | `[09:31] Marcos` | 5 · Requisitos funcionais |
| `PRD-RF-02` | `docs/PRD.md` | Requisito Funcional | O segredo de assinatura é gerado pela plataforma e devolvido na criação | `TRANSCRICAO` | `[09:31] Marcos` | 5 · Requisitos funcionais |
| `PRD-RF-03` | `docs/PRD.md` | Requisito Funcional | O cadastro define quais mudanças de status quer receber | `TRANSCRICAO` | `[09:33] Marcos` | 5 · Requisitos funcionais |
| `PRD-RF-04` | `docs/PRD.md` | Requisito Funcional | O operador edita um cadastro existente | `TRANSCRICAO` | `[09:33] Bruno` | 5 · Requisitos funcionais |
| `PRD-RF-05` | `docs/PRD.md` | Requisito Funcional | O operador remove um cadastro | `TRANSCRICAO` | `[09:33] Bruno` | 5 · Requisitos funcionais |
| `PRD-RF-06` | `docs/PRD.md` | Requisito Funcional | O operador lista os cadastros de um cliente | `TRANSCRICAO` | `[09:33] Bruno` | 5 · Requisitos funcionais |
| `PRD-RF-07` | `docs/PRD.md` | Requisito Funcional | O operador desativa um cadastro temporariamente, sem apagá-lo | `TRANSCRICAO` | `[09:21] Bruno` | 5 · Requisitos funcionais |
| `PRD-RF-08` | `docs/PRD.md` | Requisito Funcional | O operador solicita novo segredo, com o anterior válido por 24 horas | `TRANSCRICAO` | `[09:21] Sofia` | 5 · Requisitos funcionais |
| `PRD-RF-09` | `docs/PRD.md` | Requisito Funcional | Toda mudança de status gera notificação para os cadastros que a assinam | `TRANSCRICAO` | `[09:00] Marcos` | 5 · Requisitos funcionais |
| `PRD-RF-10` | `docs/PRD.md` | Requisito Funcional | Cada notificação vai assinada, para o cliente verificar origem e integridade | `TRANSCRICAO` | `[09:20] Sofia` | 5 · Requisitos funcionais |
| `PRD-RF-11` | `docs/PRD.md` | Requisito Funcional | Notificação que falha é retentada automaticamente | `TRANSCRICAO` | `[09:15] Diego` | 5 · Requisitos funcionais |
| `PRD-RF-12` | `docs/PRD.md` | Requisito Funcional | O operador consulta o histórico de entregas de um cadastro, com resultado, resposta e tempo | `TRANSCRICAO` | `[09:34] Marcos` | 5 · Requisitos funcionais |
| `PRD-RF-13` | `docs/PRD.md` | Requisito Funcional | Um administrador reprocessa manualmente uma notificação que esgotou as tentativas | `TRANSCRICAO` | `[09:35] Diego` | 5 · Requisitos funcionais |
| `PRD-RF-14` | `docs/PRD.md` | Requisito Funcional | O reprocessamento registra quem o executou | `TRANSCRICAO` | `[09:36] Sofia` | 5 · Requisitos funcionais |
| `PRD-RNF-01` | `docs/PRD.md` | Requisito Não Funcional | A notificação não pode atrasar nem afetar a mudança de status | `TRANSCRICAO` | `[09:04] Bruno` | 6 · Requisitos não funcionais |
| `PRD-RNF-02` | `docs/PRD.md` | Requisito Não Funcional | Se a mudança de status é confirmada, o evento existe; se falha, o evento não existe | `TRANSCRICAO` | `[09:06] Diego` | 6 · Requisitos não funcionais |
| `PRD-RNF-03` | `docs/PRD.md` | Requisito Não Funcional | A entrega é garantida ao menos uma vez; o cliente pode receber duplicado e deve deduplicar | `TRANSCRICAO` | `[09:24] Diego` | 6 · Requisitos não funcionais |
| `PRD-RNF-04` | `docs/PRD.md` | Requisito Não Funcional | Cinco retentativas com intervalos crescentes, cobrindo cerca de 15 horas desde a primeira falha | `TRANSCRICAO` | `[09:17] Diego` | 6 · Requisitos não funcionais |
| `PRD-RNF-05` | `docs/PRD.md` | Requisito Não Funcional | Cada notificação tem no máximo 64 KB; acima disso, erro, sem truncar | `TRANSCRICAO` | `[09:23] Sofia` | 6 · Requisitos não funcionais |
| `PRD-RNF-06` | `docs/PRD.md` | Requisito Não Funcional | O endereço cadastrado tem de usar https; http é recusado | `TRANSCRICAO` | `[09:23] Sofia` | 6 · Requisitos não funcionais |
| `PRD-RNF-07` | `docs/PRD.md` | Requisito Não Funcional | Cada cadastro tem segredo próprio; não há segredo compartilhado da plataforma | `TRANSCRICAO` | `[09:21] Sofia` | 6 · Requisitos não funcionais |
| `PRD-RNF-08` | `docs/PRD.md` | Requisito Não Funcional | O cliente tem 10 segundos para responder; além disso, a tentativa é falha | `TRANSCRICAO` | `[09:42] Diego` | 6 · Requisitos não funcionais |
| `PRD-RNF-09` | `docs/PRD.md` | Requisito Não Funcional | Notificações do mesmo pedido chegam na ordem em que ocorreram | `TRANSCRICAO` | `[09:12] Diego` | 6 · Requisitos não funcionais |
| `PRD-RNF-10` | `docs/PRD.md` | Requisito Não Funcional | A feature não introduz infraestrutura nova | `TRANSCRICAO` | `[09:07] Diego` | 6 · Requisitos não funcionais |
| `PRD-RNF-11` | `docs/PRD.md` | Requisito Não Funcional | O reprocessamento exige papel de administrador | `TRANSCRICAO` | `[09:36] Sofia` | 6 · Requisitos não funcionais |
| `PRD-RNF-12` | `docs/PRD.md` | Requisito Não Funcional | Nenhum contrato de API existente é alterado | `TRANSCRICAO` | `[09:29] Bruno` | 6 · Requisitos não funcionais |
| `RFC-ALT-01` | `docs/RFC.md` | Alternativa Descartada | Disparo HTTP síncrono na mudança de status — A transação já é pesada; um cliente lento travaria a mudança de status de outros pedidos, e um clien | `TRANSCRICAO` | `[09:04] Bruno` | Alternativas consideradas |
| `RFC-ALT-02` | `docs/RFC.md` | Alternativa Descartada | Redis Streams ou fila dedicada — Exige subir e operar infraestrutura nova. *"A gente é um time pequeno. Subir Redis Cluster pra isso é overengine | `TRANSCRICAO` | `[09:07] Larissa` | Alternativas consideradas |
| `RFC-ALT-03` | `docs/RFC.md` | Alternativa Descartada | Trigger de banco para acordar o worker — MySQL não tem NOTIFY/LISTEN: a trigger executa SQL, não notifica processo externo. As alternativas seria | `TRANSCRICAO` | `[09:09] Bruno` | Alternativas consideradas |
| `RFC-ALT-04` | `docs/RFC.md` | Alternativa Descartada | Marcar falha definitiva na própria outbox — Tabela dedicada mantém a leitura da fila limpa e serve de evidência para depuração e reprocessamento | `TRANSCRICAO` | `[09:17] Larissa` | Alternativas consideradas |
| `RFC-ALT-05` | `docs/RFC.md` | Alternativa Descartada | Retry indefinido, ou teto mais agressivo — Indefinido deixa evento pendurado para sempre quando o cliente some. Teto baixo mata o evento antes de | `TRANSCRICAO` | `[09:16] Bruno` | Alternativas consideradas |
| `RFC-ALT-06` | `docs/RFC.md` | Alternativa Descartada | Exactly-once — Exigiria coordenação dos dois lados e complexidade desproporcional. *"At-least-once com event_id resolve 99% dos casos"* | `TRANSCRICAO` | `[09:25] Diego` | Alternativas consideradas |
| `RFC-ALT-07` | `docs/RFC.md` | Alternativa Descartada | Segredo único da plataforma — *"Se vaza uma, vaza tudo."* Já houve cliente que vazou segredo no log da própria aplicação | `TRANSCRICAO` | `[09:21] Sofia` | Alternativas consideradas |
| `RFC-ALT-08` | `docs/RFC.md` | Alternativa Descartada | Montar o evento na hora do envio — Com a janela de retry longa, o evento entregue carregaria o estado atual do pedido em vez do estado que ele an | `TRANSCRICAO` | `[09:51] Bruno` | Alternativas consideradas |
| `RFC-Q01` | `docs/RFC.md` | Questão em Aberto | Limitar a taxa de envio para um mesmo cliente. Um cliente com muitos pedidos mudando em pouco tempo recebe uma rajada de chamadas — dono Diego | `TRANSCRICAO` | `[09:39] Larissa` | Questões em aberto |
| `RFC-Q02` | `docs/RFC.md` | Questão em Aberto | Escalar para múltiplos workers, e o que acontece com a ordem por pedido — dono Diego | `TRANSCRICAO` | `[09:13] Diego` | Questões em aberto |
| `RFC-Q04` | `docs/RFC.md` | Questão em Aberto | Endurecer a autorização do cadastro de webhooks. Hoje qualquer usuário autenticado opera sobre qualquer cliente, porque não existe vínculo entre  | `TRANSCRICAO` | `[09:37] Sofia` | Questões em aberto |
| `RFC-Q05` | `docs/RFC.md` | Questão em Aberto | Retenção e expurgo das linhas já entregues e do histórico de tentativas — dono Diego | `TRANSCRICAO` | `[09:08] Diego` | Questões em aberto |
| `RFC-Q06` | `docs/RFC.md` | Questão em Aberto | Guarda do segredo em repouso: texto claro ou cifrado — dono Sofia | `TRANSCRICAO` | `[09:46] Sofia` | Questões em aberto |
| `RFC-Q07` | `docs/RFC.md` | Questão em Aberto | Escopo da assinatura. O cabeçalho de horário fica fora do trecho assinado, seguindo a decisão ao pé da letra, o que o torna adulterável — a prote | `TRANSCRICAO` | `[09:22] Sofia` | Questões em aberto |
| `FDD-AC-01` | `docs/FDD.md` | Critério de Aceite Técnico | Mudança de status com endpoint assinante grava uma linha por endpoint | `TRANSCRICAO` | `[09:06] Diego` | 13 · Critérios de aceite |
| `FDD-AC-02` | `docs/FDD.md` | Critério de Aceite Técnico | Erro na publicação faz rollback da mudança de status | `TRANSCRICAO` | `[09:40] Bruno` | 13 · Critérios de aceite |
| `FDD-AC-03` | `docs/FDD.md` | Critério de Aceite Técnico | Mudança de status sem endpoint assinante não grava nada | `TRANSCRICAO` | `[09:34] Bruno` | 13 · Critérios de aceite |
| `FDD-AC-04` | `docs/FDD.md` | Critério de Aceite Técnico | Status não assinado pelo endpoint não gera linha | `TRANSCRICAO` | `[09:33] Marcos` | 13 · Critérios de aceite |
| `FDD-AC-05` | `docs/FDD.md` | Critério de Aceite Técnico | Cada linha nasce com eventId distinto | `TRANSCRICAO` | `[09:25] Diego` | 13 · Critérios de aceite |
| `FDD-AC-06` | `docs/FDD.md` | Critério de Aceite Técnico | O payload é o estado do pedido no instante da mudança | `TRANSCRICAO` | `[09:52] Larissa` | 13 · Critérios de aceite |
| `FDD-AC-07` | `docs/FDD.md` | Critério de Aceite Técnico | O worker entrega e marca DELIVERED | `TRANSCRICAO` | `[09:09] Diego` | 13 · Critérios de aceite |
| `FDD-AC-08` | `docs/FDD.md` | Critério de Aceite Técnico | Resposta não-2xx agenda retentativa com o intervalo correto | `TRANSCRICAO` | `[09:17] Diego` | 13 · Critérios de aceite |
| `FDD-AC-09` | `docs/FDD.md` | Critério de Aceite Técnico | Esgotadas as retentativas, a linha vira FAILED e entra na fila de mortos | `TRANSCRICAO` | `[09:18] Diego` | 13 · Critérios de aceite |
| `FDD-AC-10` | `docs/FDD.md` | Critério de Aceite Técnico | Timeout conta como falha | `TRANSCRICAO` | `[09:42] Diego` | 13 · Critérios de aceite |
| `FDD-AC-11` | `docs/FDD.md` | Critério de Aceite Técnico | Resposta 3xx conta como falha e não é seguida | `CODIGO` | `vitest.config.ts:4-18 · tests/setup.ts:9-15` | 13 · Critérios de aceite |
| `FDD-AC-12` | `docs/FDD.md` | Critério de Aceite Técnico | Linha travada em PROCESSING volta para PENDING após o lease | `CODIGO` | `vitest.config.ts:4-18 · tests/setup.ts:9-15` | 13 · Critérios de aceite |
| `FDD-AC-13` | `docs/FDD.md` | Critério de Aceite Técnico | A reconciliação de lease não consome retentativa | `CODIGO` | `vitest.config.ts:4-18 · tests/setup.ts:9-15` | 13 · Critérios de aceite |
| `FDD-AC-14` | `docs/FDD.md` | Critério de Aceite Técnico | Assinatura reproduzível com o segredo do cadastro | `TRANSCRICAO` | `[09:20] Sofia` | 13 · Critérios de aceite |
| `FDD-AC-15` | `docs/FDD.md` | Critério de Aceite Técnico | Durante a janela de rotação seguem duas assinaturas | `TRANSCRICAO` | `[09:21] Sofia` | 13 · Critérios de aceite |
| `FDD-AC-16` | `docs/FDD.md` | Critério de Aceite Técnico | Expirada a janela, só a assinatura nova segue | `TRANSCRICAO` | `[09:22] Sofia` | 13 · Critérios de aceite |
| `FDD-AC-17` | `docs/FDD.md` | Critério de Aceite Técnico | O segredo íntegro aparece só na criação e na rotação | `TRANSCRICAO` | `[09:31] Marcos` | 13 · Critérios de aceite |
| `FDD-AC-18` | `docs/FDD.md` | Critério de Aceite Técnico | URL http é recusada | `TRANSCRICAO` | `[09:23] Sofia` | 13 · Critérios de aceite |
| `FDD-AC-19` | `docs/FDD.md` | Critério de Aceite Técnico | Eventos do mesmo pedido são entregues em ordem | `TRANSCRICAO` | `[09:12] Diego` | 13 · Critérios de aceite |
| `FDD-AC-20` | `docs/FDD.md` | Critério de Aceite Técnico | Um destino lento não atrasa os de outros pedidos | `TRANSCRICAO` | `[09:04] Bruno` | 13 · Critérios de aceite |
| `FDD-AC-21` | `docs/FDD.md` | Critério de Aceite Técnico | Replay exige ADMIN | `TRANSCRICAO` | `[09:36] Sofia` | 13 · Critérios de aceite |
| `FDD-AC-22` | `docs/FDD.md` | Critério de Aceite Técnico | Replay recoloca na fila preservando o eventId | `TRANSCRICAO` | `[09:25] Diego` | 13 · Critérios de aceite |
| `FDD-AC-23` | `docs/FDD.md` | Critério de Aceite Técnico | Replay repetido é recusado | `CODIGO` | `vitest.config.ts:4-18 · tests/setup.ts:9-15` | 13 · Critérios de aceite |
| `FDD-AC-24` | `docs/FDD.md` | Critério de Aceite Técnico | Replay grava quem executou | `TRANSCRICAO` | `[09:36] Sofia` | 13 · Critérios de aceite |
| `FDD-AC-25` | `docs/FDD.md` | Critério de Aceite Técnico | Nenhum segredo aparece em log | `CODIGO` | `vitest.config.ts:4-18 · tests/setup.ts:9-15` | 13 · Critérios de aceite |
| `FDD-AC-26` | `docs/FDD.md` | Critério de Aceite Técnico | O requestId da requisição chega ao evento de entrega | `CODIGO` | `vitest.config.ts:4-18 · tests/setup.ts:9-15` | 13 · Critérios de aceite |
| `FDD-AC-27` | `docs/FDD.md` | Critério de Aceite Técnico | Endpoint desativado não recebe entrega | `TRANSCRICAO` | `[09:21] Bruno` | 13 · Critérios de aceite |
| `FDD-AC-28` | `docs/FDD.md` | Critério de Aceite Técnico | Payload acima do teto vai direto para a fila de mortos | `TRANSCRICAO` | `[09:24] Larissa` | 13 · Critérios de aceite |
| `FDD-AC-29` | `docs/FDD.md` | Critério de Aceite Técnico | A suíte existente passa sem alteração | `TRANSCRICAO` | `[09:29] Bruno` | 13 · Critérios de aceite |
| `FDD-AC-30` | `docs/FDD.md` | Critério de Aceite Técnico | Com WEBHOOK_WORKER_ENABLED=false nada é publicado | `CODIGO` | `vitest.config.ts:4-18 · tests/setup.ts:9-15` | 13 · Critérios de aceite |
| `FDD-CONV-01` | `docs/FDD.md` | Restrição do Código | Prefixo /api/v1 em todas as rotas | `CODIGO` | `src/app.ts:67` | 11 · Integração |
| `FDD-CONV-02` | `docs/FDD.md` | Restrição do Código | Envelope de erro { error: { code, message, details? } } | `CODIGO` | `src/middlewares/error.middleware.ts:15-24` | 11 · Integração |
| `FDD-CONV-03` | `docs/FDD.md` | Restrição do Código | Envelope de listagem { data, pagination } | `CODIGO` | `src/shared/http/response.ts:8-11` | 11 · Integração |
| `FDD-CONV-04` | `docs/FDD.md` | Restrição do Código | PK String @id @default(uuid()) @db.Char(36) e @@map snake_case | `CODIGO` | `prisma/schema.prisma:25-138` | 11 · Integração |
| `FDD-CONV-05` | `docs/FDD.md` | Restrição do Código | AppError expõe errorCode, não code | `CODIGO` | `src/shared/errors/app-error.ts:3-15` | 11 · Integração |
| `FDD-CONV-06` | `docs/FDD.md` | Restrição do Código | requireRole disponível; roles ADMIN e OPERATOR | `CODIGO` | `src/middlewares/auth.middleware.ts:49-61` | 11 · Integração |
| `FDD-CONV-07` | `docs/FDD.md` | Restrição do Código | Sem cliente HTTP nas dependências: fetch nativo do Node 20 | `CODIGO` | `package.json:25-34` | 11 · Integração |
| `FDD-CONV-08` | `docs/FDD.md` | Restrição do Código | redact do Pino não cobre secret nem signature | `CODIGO` | `src/shared/logger/index.ts:4-11` | 11 · Integração |
| `FDD-CONV-09` | `docs/FDD.md` | Restrição do Código | requestId em X-Request-Id, base da correlação | `CODIGO` | `src/middlewares/request-logger.middleware.ts:6-8` | 11 · Integração |
| `FDD-CONV-10` | `docs/FDD.md` | Restrição do Código | Shutdown atual não drena trabalho em background | `CODIGO` | `src/server.ts:13-21` | 11 · Integração |
| `FDD-CONV-11` | `docs/FDD.md` | Restrição do Código | Limpeza de tabelas no beforeEach da suíte | `CODIGO` | `tests/setup.ts:9-15` | 11 · Integração |
| `FDD-CONV-12` | `docs/FDD.md` | Restrição do Código | Estoque só muda em PENDING→PAID e em cancelamento | `CODIGO` | `src/modules/orders/order.status.ts:29-37` | 11 · Integração |
| `FDD-CT-01` | `docs/FDD.md` | Contrato Público | POST /api/v1/webhooks — cadastrar endpoint | `TRANSCRICAO` | `[09:31] Marcos` | 7.1 · Contratos públicos |
| `FDD-CT-02` | `docs/FDD.md` | Contrato Público | GET /api/v1/webhooks — listar por cliente | `TRANSCRICAO` | `[09:33] Bruno` | 7.2 · Contratos públicos |
| `FDD-CT-03` | `docs/FDD.md` | Contrato Público | GET /api/v1/webhooks/:id — detalhe | `CODIGO` | `src/routes/index.ts:21-31` | 7.3 · Contratos públicos |
| `FDD-CT-04` | `docs/FDD.md` | Contrato Público | PATCH /api/v1/webhooks/:id — editar | `TRANSCRICAO` | `[09:33] Bruno` | 7.4 · Contratos públicos |
| `FDD-CT-05` | `docs/FDD.md` | Contrato Público | DELETE /api/v1/webhooks/:id — remover | `TRANSCRICAO` | `[09:33] Bruno` | 7.5 · Contratos públicos |
| `FDD-CT-06` | `docs/FDD.md` | Contrato Público | POST /api/v1/webhooks/:id/rotate-secret — rotacionar segredo | `TRANSCRICAO` | `[09:21] Sofia` | 7.6 · Contratos públicos |
| `FDD-CT-07` | `docs/FDD.md` | Contrato Público | GET /api/v1/webhooks/:id/deliveries — histórico de entregas | `TRANSCRICAO` | `[09:34] Marcos` | 7.7 · Contratos públicos |
| `FDD-CT-08` | `docs/FDD.md` | Contrato Público | GET /api/v1/admin/webhooks/dead-letter — listar fila de mortos | `CODIGO` | `src/routes/index.ts:21-31` | 7.8 · Contratos públicos |
| `FDD-CT-09` | `docs/FDD.md` | Contrato Público | POST /api/v1/admin/webhooks/dead-letter/:id/replay — reprocessar | `TRANSCRICAO` | `[09:35] Diego` | 7.9 · Contratos públicos |
| `FDD-ERR-01` | `docs/FDD.md` | Erro previsto | WEBHOOK_NOT_FOUND — Endpoint inexistente em GET, PATCH, DELETE, rotação ou entregas | `TRANSCRICAO` | `[09:28] Bruno` | 8 · Matriz de erros |
| `FDD-ERR-02` | `docs/FDD.md` | Erro previsto | WEBHOOK_CUSTOMER_NOT_FOUND — customerId do corpo não existe em customers | `CODIGO` | `src/shared/errors/http-errors.ts:3-63` | 8 · Matriz de erros |
| `FDD-ERR-03` | `docs/FDD.md` | Erro previsto | WEBHOOK_INVALID_STATUS — subscribedStatuses contém valor fora de OrderStatus | `TRANSCRICAO` | `[09:33] Marcos` | 8 · Matriz de erros |
| `FDD-ERR-04` | `docs/FDD.md` | Erro previsto | WEBHOOK_NO_STATUS_SUBSCRIBED — subscribedStatuses vazio — o cadastro não receberia nada | `TRANSCRICAO` | `[09:33] Marcos` | 8 · Matriz de erros |
| `FDD-ERR-05` | `docs/FDD.md` | Erro previsto | WEBHOOK_ENDPOINT_INACTIVE — Replay para endpoint com active = false | `TRANSCRICAO` | `[09:21] Bruno` | 8 · Matriz de erros |
| `FDD-ERR-06` | `docs/FDD.md` | Erro previsto | WEBHOOK_DEAD_LETTER_NOT_FOUND — Identificador inexistente na fila de mortos | `TRANSCRICAO` | `[09:35] Diego` | 8 · Matriz de erros |
| `FDD-ERR-07` | `docs/FDD.md` | Erro previsto | WEBHOOK_ALREADY_REPLAYED — Entrada com replayedAt preenchido | `TRANSCRICAO` | `[09:35] Diego` | 8 · Matriz de erros |
| `FDD-ERR-08` | `docs/FDD.md` | Erro previsto | WEBHOOK_DELIVERY_HTTP_ERROR — Resposta fora da faixa 2xx | `TRANSCRICAO` | `[09:14] Larissa` | 8 · Matriz de erros |
| `FDD-ERR-09` | `docs/FDD.md` | Erro previsto | WEBHOOK_DELIVERY_TIMEOUT — Destino não respondeu dentro do limite | `TRANSCRICAO` | `[09:42] Diego` | 8 · Matriz de erros |
| `FDD-ERR-10` | `docs/FDD.md` | Erro previsto | WEBHOOK_DELIVERY_NETWORK_ERROR — Falha de DNS, TLS ou conexão recusada | `CODIGO` | `src/shared/errors/http-errors.ts:3-63` | 8 · Matriz de erros |
| `FDD-ERR-11` | `docs/FDD.md` | Erro previsto | WEBHOOK_DELIVERY_REDIRECT_REFUSED — Resposta 3xx; não seguimos redirecionamento | `CODIGO` | `src/shared/errors/http-errors.ts:3-63` | 8 · Matriz de erros |
| `FDD-ERR-12` | `docs/FDD.md` | Erro previsto | WEBHOOK_RETRIES_EXHAUSTED — Motivo final registrado ao mover para a fila de mortos | `TRANSCRICAO` | `[09:15] Diego` | 8 · Matriz de erros |
| `FDD-ERR-13` | `docs/FDD.md` | Erro previsto | WEBHOOK_ENDPOINT_INACTIVE — Endpoint desativado entre a inserção e a entrega | `TRANSCRICAO` | `[09:21] Bruno` | 8 · Matriz de erros |
| `FDD-ERR-14` | `docs/FDD.md` | Erro previsto | WEBHOOK_PAYLOAD_TOO_LARGE — Evento acima do teto de tamanho | `TRANSCRICAO` | `[09:24] Larissa` | 8 · Matriz de erros |
| `FDD-ERR-15` | `docs/FDD.md` | Erro previsto | WEBHOOK_SECRET_REQUIRED — O campo não existe na entrada: nós geramos o segredo — [09:31] Marcos | `TRANSCRICAO` | `[09:28] Bruno` | 8 · Matriz de erros |
| `FDD-ERR-16` | `docs/FDD.md` | Erro previsto | WEBHOOK_INVALID_URL — A verificação vive no schema Zod e produz VALIDATION_ERROR — [09:23] Sofia | `TRANSCRICAO` | `[09:28] Bruno` | 8 · Matriz de erros |
| `FDD-ERR-17` | `docs/FDD.md` | Erro previsto | WEBHOOK_BATCH_SIZE — WEBHOOK_BATCH_SIZE | `CODIGO` | `src/shared/errors/http-errors.ts:3-63` | 8 · Matriz de erros |
| `FDD-ERR-18` | `docs/FDD.md` | Erro previsto | WEBHOOK_CONCURRENCY — WEBHOOK_CONCURRENCY | `CODIGO` | `src/shared/errors/http-errors.ts:3-63` | 8 · Matriz de erros |
| `FDD-ERR-19` | `docs/FDD.md` | Erro previsto | WEBHOOK_LEASE_TIMEOUT_MS — WEBHOOK_LEASE_TIMEOUT_MS | `CODIGO` | `src/shared/errors/http-errors.ts:3-63` | 8 · Matriz de erros |
| `FDD-ERR-20` | `docs/FDD.md` | Erro previsto | WEBHOOK_WORKER_ENABLED — WEBHOOK_WORKER_ENABLED | `CODIGO` | `src/shared/errors/http-errors.ts:3-63` | 8 · Matriz de erros |
| `FDD-INT-1` | `docs/FDD.md` | Integração com o Código | Integração em src/modules/orders/order.service.ts | `CODIGO` | `src/modules/orders/order.service.ts` | 11.1 · Integração |
| `FDD-INT-2` | `docs/FDD.md` | Integração com o Código | Integração em src/modules/orders/order.repository.ts | `CODIGO` | `src/modules/orders/order.repository.ts` | 11.2 · Integração |
| `FDD-INT-3` | `docs/FDD.md` | Integração com o Código | Integração em src/shared/errors/ | `CODIGO` | `src/shared/errors/` | 11.3 · Integração |
| `FDD-INT-4` | `docs/FDD.md` | Integração com o Código | Integração em src/middlewares/error.middleware.ts | `CODIGO` | `src/middlewares/error.middleware.ts` | 11.4 · Integração |
| `FDD-INT-5` | `docs/FDD.md` | Integração com o Código | Integração em src/middlewares/auth.middleware.ts | `CODIGO` | `src/middlewares/auth.middleware.ts` | 11.5 · Integração |
| `FDD-MOD-WebhookDeadLetter` | `docs/FDD.md` | Modelo de Dados | Model WebhookDeadLetter | `CODIGO` | `prisma/schema.prisma:25-138` | 6.3 · Models propostos |
| `FDD-MOD-WebhookDelivery` | `docs/FDD.md` | Modelo de Dados | Model WebhookDelivery | `CODIGO` | `prisma/schema.prisma:25-138` | 6.3 · Models propostos |
| `FDD-MOD-WebhookEndpoint` | `docs/FDD.md` | Modelo de Dados | Model WebhookEndpoint | `CODIGO` | `prisma/schema.prisma:25-138` | 6.3 · Models propostos |
| `FDD-MOD-WebhookOutbox` | `docs/FDD.md` | Modelo de Dados | Model WebhookOutbox | `CODIGO` | `prisma/schema.prisma:25-138` | 6.3 · Models propostos |
| `FDD-OBJ-01` | `docs/FDD.md` | Objetivo Técnico | Registrar o evento atomicamente com a mudança de status | `TRANSCRICAO` | `[09:40] Bruno` | 2 · Objetivos técnicos |
| `FDD-OBJ-02` | `docs/FDD.md` | Objetivo Técnico | Nenhuma chamada HTTP de saída dentro da transação de pedidos | `TRANSCRICAO` | `[09:04] Bruno` | 2 · Objetivos técnicos |
| `FDD-OBJ-03` | `docs/FDD.md` | Objetivo Técnico | Entrega em polling curto, sem dependência nova no projeto | `TRANSCRICAO` | `[09:09] Diego` | 2 · Objetivos técnicos |
| `FDD-OBJ-04` | `docs/FDD.md` | Objetivo Técnico | Tolerar indisponibilidade do destino sem perder evento | `TRANSCRICAO` | `[09:15] Diego` | 2 · Objetivos técnicos |
| `FDD-OBJ-05` | `docs/FDD.md` | Objetivo Técnico | Permitir ao cliente verificar origem e integridade do payload | `TRANSCRICAO` | `[09:22] Sofia` | 2 · Objetivos técnicos |
| `FDD-OBJ-06` | `docs/FDD.md` | Objetivo Técnico | Não alterar contrato nem latência de endpoint existente — suíte atual passa sem modificação | `CODIGO` | `tests/orders.test.ts` | 2 · Objetivos técnicos |
| `FDD-OBJ-07` | `docs/FDD.md` | Objetivo Técnico | Absorver o módulo na infraestrutura compartilhada — error e validate middlewares e `response.ts` intocados | `CODIGO` | `src/middlewares/error.middleware.ts` | 2 · Objetivos técnicos |
| `FDD-R-01` | `docs/FDD.md` | Risco Técnico | Defeito no publisher derruba a mudança de status, a operação central do OMS — Média/Crítico | `TRANSCRICAO` | `[09:41] Diego` | 14 · Riscos |
| `FDD-R-02` | `docs/FDD.md` | Risco Técnico | A meta de latência não se cumpre para clientes lentos — Alta/Médio | `TRANSCRICAO` | `[09:02] Marcos` | 14 · Riscos |
| `FDD-R-03` | `docs/FDD.md` | Risco Técnico | Qualquer operador autenticado obtém segredo de qualquer cliente — Alta/Alto | `TRANSCRICAO` | `[09:37] Sofia` | 14 · Riscos |
| `FDD-R-04` | `docs/FDD.md` | Risco Técnico | Fila de mortos cresce sem ninguém perceber, por não haver alerta — Média/Médio | `TRANSCRICAO` | `[09:37] Larissa` | 14 · Riscos |
| `FDD-R-05` | `docs/FDD.md` | Risco Técnico | Worker morto passa despercebido; eventos acumulam — Média/Alto | `TRANSCRICAO` | `[09:11] Diego` | 14 · Riscos |
| `FDD-R-06` | `docs/FDD.md` | Risco Técnico | Consulta de endpoints dentro da transação degrada changeStatus — Baixa/Médio | `TRANSCRICAO` | `[09:04] Bruno` | 14 · Riscos |
| `FDD-R-07` | `docs/FDD.md` | Risco Técnico | Cada implantação gera entregas duplicadas — Média/Baixo | `CODIGO` | `src/server.ts:13-21` | 14 · Riscos |
| `FDD-R-08` | `docs/FDD.md` | Risco Técnico | Segredo vaza em log de erro — Baixa/Alto | `TRANSCRICAO` | `[09:22] Diego` | 14 · Riscos |
| `FDD-R-09` | `docs/FDD.md` | Risco Técnico | Cliente não implementa deduplicação e processa evento duas vezes — Média/Médio | `TRANSCRICAO` | `[09:25] Sofia` | 14 · Riscos |
| `FDD-R-10` | `docs/FDD.md` | Risco Técnico | Rajada de mudanças inunda um cliente — Baixa/Médio | `TRANSCRICAO` | `[09:38] Diego` | 14 · Riscos |
| `ADR-001` | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Decisão | Padrão outbox no MySQL para publicação de eventos de pedido | `TRANSCRICAO` | `[09:48] Larissa · [09:03] Larissa` | Decisão |
| `ADR-002` | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão | Worker em processo separado consumindo a outbox por polling | `TRANSCRICAO` | `[09:48] Larissa · [09:06] Diego` | Decisão |
| `ADR-003` | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Decisão | Retry com backoff exponencial de 5 retentativas e dead letter queue em tabela dedicada | `TRANSCRICAO` | `[09:48] Larissa · [09:14] Larissa` | Decisão |
| `ADR-004` | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Decisão | Autenticação de origem com HMAC-SHA256 e secret única por endpoint | `TRANSCRICAO` | `[09:48] Larissa · [09:19] Sofia` | Decisão |
| `ADR-005` | `docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md` | Decisão | Entrega at-least-once com idempotência delegada ao cliente via X-Event-Id | `TRANSCRICAO` | `[09:48] Larissa · [09:25] Diego` | Decisão |
| `ADR-006` | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Reuso dos padrões existentes do projeto no módulo de webhooks | `TRANSCRICAO` | `[09:48] Larissa · [09:27] Bruno` | Decisão |
| `ADR-007` | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Decisão | Payload materializado como snapshot na inserção e filtragem de destinatários na origem | `TRANSCRICAO` | `[09:52] Bruno · [09:51] Bruno` | Decisão |
| `ADR-008` | `docs/adrs/ADR-008-controle-de-acesso-dos-endpoints.md` | Decisão | Replay de dead letter restrito a ADMIN e CRUD de configuração apenas autenticado | `TRANSCRICAO` | `[09:48] Larissa · [09:35] Diego` | Decisão |

## 2. Cobertura por documento

| Documento | Itens | `TRANSCRICAO` | `CODIGO` |
|---|---|---|---|
| `docs/PRD.md` | 61 | 61 | 0 |
| `docs/RFC.md` | 14 | 14 | 0 |
| `docs/FDD.md` | 97 | 57 | 40 |
| `docs/adrs/` (8 arquivos) | 8 | 8 | 0 |

> **Fórmula da cobertura.** Itens rastreados dividido por itens identificáveis nos documentos, onde
> *identificável* é todo item que carrega identificador próprio: `RF`, `RNF`, `OBJ`, `FE`, `R`, `CA`, `ALT`,
> `Q`, `CT`, `ERR`, `AC`, `MOD`, `INT`, `CONV` e os oito `ADR`. Prosa explicativa não conta. Publicar o
> percentual sem a fórmula ao lado torna o número inauditável.

## 3. Decisões sem âncora na reunião

Trinta decisões de implementação que a reunião não tomou. Nenhuma pode ser apresentada como fala de
participante. Cada uma carrega procedência: `MERCADO` para padrão de indústria verificado, `CODIGO` para
derivação de padrão já existente no repositório, `PRODUTO` para escolha de quem produziu a documentação.

| ID | Decisão | Procedência | Base |
|---|---|---|---|
| `H01` | Valor de X-Signature — v1=<hex>; múltiplas assinaturas separadas por vírgula durante a rotação | `MERCADO` | MERCADO Stripe / GitHub |
| `H02` | String canônica assinada — Somente o corpo cru. Literal a [09:22] Sofia ("HMAC-SHA256 sobre o corpo do request"). O X | `PRODUTO` | [09:22] — assinar {timestamp}.{corpo}, ao padrão Stripe, foi cogitado e descartado por ext |
| `H03` | Formato de X-Timestamp — Unix epoch em segundos (payload segue ISO 8601 conforme [09:43]) | `MERCADO` | MERCADO Stripe / Slack |
| `H04` | Tolerância de recência — 5 minutos, recomendada ao cliente na doc — não validamos, somos o emissor. Limitação a dec | `MERCADO` | MERCADO Stripe + limitação derivada de H02 |
| `H05` | Geração do secret — 32 bytes de crypto.randomBytes em hex, prefixo whsec_ | `MERCADO` | MERCADO Stripe + CODIGO (sem dependência nova, node:crypto) |
| `H06` | Secret at-rest — Texto claro + redact no Pino. Criptografia não decidida → Q06, dono Sofia, prazo revisão p | `PRODUTO` | PRODUTO |
| `H07` | Exposição do secret — Retornado íntegro só em POST /webhooks e POST /webhooks/:id/rotate-secret; mascarado (whse | `MERCADO` | MERCADO + [09:31] |
| `H08` | Tamanho do batch — 50, via env com default | `PRODUTO` | PRODUTO — [09:08] diz só "batch pequeno" |
| `H09` | Claim de linhas — SELECT … FOR UPDATE SKIP LOCKED | `MERCADO` | MERCADO + CODIGO (MySQL 8.0 no docker-compose.yml) |
| `H10` | Lease de evento travado — processingStartedAt + reclaim após 60s (6× o timeout de 10s) | `PRODUTO` | PRODUTO — buraco que a reunião não enxergou |
| `H11` | Classe de falha — Toda resposta não-2xx e toda falha de rede consomem as 5 tentativas. Sem atalho para 4xx | `PRODUTO` | [09:15] [09:17] — literal, sem extrapolação |
| `H12` | Jitter no backoff — Não. Curva exata de [09:17]. Jitter fica como melhoria futura | `PRODUTO` | [09:17] |
| `H13` | Paralelismo do batch — Serial dentro de cada order_id, paralelo entre order_ids distintos, concorrência 10 | `PRODUTO` | [09:12] (ordering) + [09:04] (cliente lento não trava os outros) |
| `H14` | Redirects 3xx — Não seguir; conta como falha | `MERCADO` | MERCADO Stripe |
| `H15` | Fan-out — Uma mudança de status com N endpoints assinantes gera N linhas na outbox, com N eventId di | `PRODUTO` | PRODUTO — coerente com X-Webhook-Id de [09:44] |
| `H16` | subscribedStatuses — Coluna Json | `CODIGO` | CODIGO — customers.address já é Json (prisma/schema.prisma:44) |
| `H17` | Retenção de deliveries — Guarda todas as tentativas; o endpoint devolve as 100 mais recentes. Expurgo fora de escop | `PRODUTO` | [09:34] + PRODUTO |
| `H18` | Secret antigo na rotação — Colunas na própria linha — a janela de 24h garante no máximo 2 secrets vivos | `PRODUTO` | [09:21] + PRODUTO |
| `H19` | Padrão de PK e mapeamento — String @id @default(uuid()) @db.Char(36), @@map("snake_case"), colunas camelCase sem @map | `CODIGO` | CODIGO prisma/schema.prisma + [09:51] |
| `H20` | Desativar sem apagar — PATCH { active: false }; DELETE é hard delete | `CODIGO` | [09:21] ("estado ativo") + CODIGO |
| `H21` | Envelope de listagem — { data, pagination }, pageSize default 20, teto 100 | `CODIGO` | CODIGO src/shared/http/response.ts:8 |
| `H22` | Envelope de erro — { error: { code, message, details? } } — nada a mudar no middleware | `CODIGO` | CODIGO src/middlewares/error.middleware.ts:15-24 |
| `H23` | Constantes congeladas — POLL_INTERVAL_MS=2000, HTTP_TIMEOUT_MS=10000, MAX_ATTEMPTS=5, BACKOFF_MS=[60s,300s,1800s,7 | `PRODUTO` | PRODUTO — mudar exige PR e reabrir o ADR |
| `H24` | Env de tuning — WEBHOOK_BATCH_SIZE=50, WEBHOOK_CONCURRENCY=10, WEBHOOK_LEASE_TIMEOUT_MS=60000, WEBHOOK_WOR | `CODIGO` | CODIGO src/config/env.ts:4-27 |
| `H25` | Observabilidade sem vendor — Eventos Pino estruturados: webhook_event_enqueued, webhook_delivery_attempted, webhook_del | `CODIGO` | CODIGO src/shared/logger/index.ts |
| `H26` | Tracing — Propagar o requestId da requisição de origem até a entrega, via coluna na outbox | `CODIGO` | CODIGO src/middlewares/request-logger.middleware.ts:6-8 |
| `H27` | Redação de segredos no log — Adicionar *.secret, *.previousSecret, *.signature ao redact | `CODIGO` | CODIGO src/shared/logger/index.ts:4-11 (hoje não cobre) |
| `H28` | Cliente HTTP — fetch nativo do Node 20 com AbortSignal.timeout(10_000). Nenhuma dependência nova | `CODIGO` | CODIGO package.json (sem axios/undici) + [09:42] |
| `H29` | Drenagem no shutdown — Worker precisa de hook de drenagem — o shutdown atual não aguarda server.close() | `CODIGO` | CODIGO src/server.ts:13-21 |
| `H30` | Limpeza nos testes — As 4 tabelas novas entram no beforeEach, respeitando ordem de FK | `CODIGO` | CODIGO tests/setup.ts:9-15 |

## 4. Índice reverso — o que da reunião entrou no pacote

A tabela acima responde *de onde veio cada item*. Esta responde a pergunta inversa, que é a que expõe
omissão: **o que da fita ficou de fora**. São 54 timestamps distintos e 151 falas.

| Timestamp | Coberto por | Se não, por quê |
|---|---|---|
| `[09:00]` | `PRD-OBJ-04`, `PRD-OBJ-06`, `PRD-RF-09` | — |
| `[09:01]` | — | Bruno pergunta se é tempo real mesmo — a resposta em [09:02] é que vira requisito |
| `[09:02]` | `PRD-FE-03`, `PRD-OBJ-01`, `PRD-R-02`, `FDD-R-02` | — |
| `[09:03]` | `ADR-001` | — |
| `[09:04]` | `PRD-OBJ-03`, `PRD-RNF-01`, `RFC-ALT-01`, `FDD-AC-20`, `FDD-R-06`, `FDD-OBJ-02` | — |
| `[09:05]` | — | Diego entra na call; coordenação |
| `[09:06]` | `PRD-CA-01`, `PRD-OBJ-02`, `PRD-RNF-02`, `FDD-AC-01`, `ADR-002` | — |
| `[09:07]` | `PRD-RNF-10`, `RFC-ALT-02` | — |
| `[09:08]` | `PRD-FE-04`, `RFC-Q05` | — |
| `[09:09]` | `RFC-ALT-03`, `FDD-AC-07`, `FDD-OBJ-03` | — |
| `[09:10]` | — | fala de coordenação, sem conteúdo decisório |
| `[09:11]` | `FDD-R-05` | — |
| `[09:12]` | `PRD-CA-11`, `PRD-RNF-09`, `FDD-AC-19` | — |
| `[09:13]` | `PRD-FE-06`, `RFC-Q02` | — |
| `[09:14]` | `FDD-ERR-08`, `ADR-003` | — |
| `[09:15]` | `PRD-RF-11`, `FDD-ERR-12`, `FDD-OBJ-04` | — |
| `[09:16]` | `PRD-CA-06`, `RFC-ALT-05` | — |
| `[09:17]` | `PRD-RNF-04`, `RFC-ALT-04`, `FDD-AC-08` | — |
| `[09:18]` | `PRD-CA-07`, `FDD-AC-09` | — |
| `[09:19]` | `ADR-004` | — |
| `[09:20]` | `PRD-CA-04`, `PRD-RF-10`, `FDD-AC-14` | — |
| `[09:21]` | `PRD-CA-09`, `PRD-CA-14`, `PRD-RF-07`, `PRD-RF-08`, `PRD-RNF-07`, `RFC-ALT-07` … +5 | — |
| `[09:22]` | `PRD-R-07`, `RFC-Q07`, `FDD-AC-16`, `FDD-R-08`, `FDD-OBJ-05` | — |
| `[09:23]` | `PRD-CA-05`, `PRD-RNF-05`, `PRD-RNF-06`, `FDD-AC-18` | — |
| `[09:24]` | `PRD-RNF-03`, `FDD-AC-28`, `FDD-ERR-14` | — |
| `[09:25]` | `RFC-ALT-06`, `FDD-AC-05`, `FDD-AC-22`, `FDD-R-09`, `ADR-005` | — |
| `[09:26]` | `PRD-R-04` | — |
| `[09:27]` | `ADR-006` | — |
| `[09:28]` | `FDD-ERR-01`, `FDD-ERR-15`, `FDD-ERR-16` | — |
| `[09:29]` | `PRD-RNF-12`, `FDD-AC-29` | — |
| `[09:30]` | `PRD-CA-13` | — |
| `[09:31]` | `PRD-CA-03`, `PRD-RF-01`, `PRD-RF-02`, `FDD-AC-17`, `FDD-CT-01` | — |
| `[09:32]` | — | fala de coordenação, sem conteúdo decisório |
| `[09:33]` | `PRD-RF-03`, `PRD-RF-04`, `PRD-RF-05`, `PRD-RF-06`, `FDD-AC-04`, `FDD-CT-02` … +4 | — |
| `[09:34]` | `PRD-CA-02`, `PRD-CA-08`, `PRD-R-06`, `PRD-RF-12`, `FDD-AC-03`, `FDD-CT-07` | — |
| `[09:35]` | `PRD-RF-13`, `FDD-CT-09`, `FDD-ERR-06`, `FDD-ERR-07`, `ADR-008` | — |
| `[09:36]` | `PRD-CA-10`, `PRD-RF-14`, `PRD-RNF-11`, `FDD-AC-21`, `FDD-AC-24` | — |
| `[09:37]` | `PRD-FE-01`, `PRD-FE-07`, `PRD-R-03`, `PRD-R-05`, `RFC-Q04`, `FDD-R-03` … +1 | — |
| `[09:38]` | `PRD-R-08`, `FDD-R-10` | — |
| `[09:39]` | `PRD-FE-05`, `RFC-Q01` | — |
| `[09:40]` | `PRD-CA-12`, `PRD-FE-02`, `PRD-R-01`, `FDD-AC-02`, `FDD-OBJ-01` | — |
| `[09:41]` | `FDD-R-01` | — |
| `[09:42]` | `PRD-RNF-08`, `FDD-AC-10`, `FDD-ERR-09` | — |
| `[09:43]` | — | fala de coordenação, sem conteúdo decisório |
| `[09:44]` | — | fala de coordenação, sem conteúdo decisório |
| `[09:45]` | — | Larissa pergunta sobre prazo; a resposta está em [09:45] Marcos e [09:46] Larissa |
| `[09:46]` | `PRD-OBJ-05`, `RFC-Q06` | — |
| `[09:47]` | — | Larissa e Marcos confirmam o prazo já registrado em [09:46] |
| `[09:48]` | `ADR-001`, `ADR-002`, `ADR-003`, `ADR-004`, `ADR-005`, `ADR-006` … +1 | — |
| `[09:49]` | — | Confirmação final dos participantes; registrada no cabeçalho dos ADRs |
| `[09:50]` | — | Encerramento e agendamento da sessão de revisão; registrado no índice dos ADRs |
| `[09:51]` | `RFC-ALT-08`, `ADR-007` | — |
| `[09:52]` | `FDD-AC-06`, `ADR-007` | — |
| `[09:53]` | — | Encerramento da call |

**43 de 54 timestamps** (79%) têm pelo menos um item do pacote apontando
para eles. Os 11 restantes são falas de coordenação — abertura de bloco, entrada na call,
passagem de palavra, encerramento — listadas nominalmente acima, para que a ausência seja verificável em vez
de assumida.

---

**Manutenção.** Este tracker é gerado a partir das tabelas dos próprios documentos, que carregam a coluna de
origem. Quando um documento muda, o tracker precisa ser regenerado — e a divergência entre os dois é
detectável comparando os identificadores.
