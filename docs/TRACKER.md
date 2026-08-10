# Tracker de Rastreabilidade

Referência cruzada entre cada item dos documentos e sua origem. Serve para responder, item a item, **de onde
veio isso** — e para expor o que não tem origem, em vez de escondê-lo.

| | |
|---|---|
| **Itens rastreados** | 276 |
| **Fonte `TRANSCRICAO`** | 216 (78%) |
| **Fonte `CODIGO`** | 60 linhas, 24 caminhos distintos |
| **Decisões sem âncora na reunião** | 51, listadas à parte na seção 3 |
| **Cobertura da fita** | 51 dos 54 timestamps distintos — seção 4 |

## Como ler

| Coluna | Conteúdo |
|---|---|
| **ID** | Identificador estável do item. O prefixo indica o documento dono |
| **Documento** | Arquivo onde o item aparece |
| **Tipo** | Natureza do item |
| **Conteúdo** | Resumo de uma linha |
| **Fonte** | `TRANSCRICAO` ou `CODIGO` |
| **Localização** | `[hh:mm] Nome` para a fita, `caminho:linha` para o código; depois de ` · `, o ponteiro para a seção do documento |

O ponteiro de seção vive dentro da célula de localização. Linhas de `docs/PRD.md` não o carregam: os
cabeçalhos daquele documento não são numerados, e um ponteiro que não resolve engana mais do que a ausência
dele. FDD, RFC e ADRs têm numeração ou títulos estáveis, e ali o ponteiro vale.

Itens **derivados** — decididos fora da reunião para o FDD ficar acionável — **não entram na tabela
principal**. Eles vivem na seção 3, com a procedência de cada um. Misturá-los aqui daria a eles uma
aparência de origem que não têm.

## 1. Itens rastreados

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| `PRD-CA-01` | `docs/PRD.md` | Critério de Aceitação | Um cadastro criado passa a receber notificação na próxima mudança de status assinada | `TRANSCRICAO` | `[09:06] Diego` |
| `PRD-CA-02` | `docs/PRD.md` | Critério de Aceitação | Mudança de status não assinada pelo cadastro não gera notificação | `TRANSCRICAO` | `[09:34] Bruno` |
| `PRD-CA-03` | `docs/PRD.md` | Critério de Aceitação | O segredo é devolvido na criação e não aparece íntegro em nenhuma consulta posterior | `TRANSCRICAO` | `[09:31] Marcos` |
| `PRD-CA-04` | `docs/PRD.md` | Critério de Aceitação | O cliente consegue validar a assinatura com o segredo recebido | `TRANSCRICAO` | `[09:20] Sofia` |
| `PRD-CA-05` | `docs/PRD.md` | Critério de Aceitação | Endereço http é recusado na criação | `TRANSCRICAO` | `[09:23] Sofia` |
| `PRD-CA-06` | `docs/PRD.md` | Critério de Aceitação | Cliente indisponível por duas horas recebe a notificação quando volta | `TRANSCRICAO` | `[09:16] Diego` |
| `PRD-CA-07` | `docs/PRD.md` | Critério de Aceitação | Cliente permanentemente fora do ar tem a notificação registrada como falha definitiva, sem perda | `TRANSCRICAO` | `[09:18] Diego` |
| `PRD-CA-08` | `docs/PRD.md` | Critério de Aceitação | O histórico mostra cada tentativa com resultado, resposta e tempo | `TRANSCRICAO` | `[09:34] Marcos` |
| `PRD-CA-09` | `docs/PRD.md` | Critério de Aceitação | Rotação mantém as duas credenciais válidas por 24 horas | `TRANSCRICAO` | `[09:21] Sofia` |
| `PRD-CA-10` | `docs/PRD.md` | Critério de Aceitação | Reprocessamento só funciona para administrador e fica registrado com o autor | `TRANSCRICAO` | `[09:36] Sofia` |
| `PRD-CA-11` | `docs/PRD.md` | Critério de Aceitação | Três mudanças seguidas no mesmo pedido chegam na ordem correta | `TRANSCRICAO` | `[09:12] Diego` |
| `PRD-CA-12` | `docs/PRD.md` | Critério de Aceitação | Falha ao registrar o evento impede a mudança de status | `TRANSCRICAO` | `[09:40] Bruno` |
| `PRD-CA-13` | `docs/PRD.md` | Critério de Aceitação | Nenhum endpoint existente muda de contrato ou comportamento | `TRANSCRICAO` | `[09:30] Larissa` |
| `PRD-CA-14` | `docs/PRD.md` | Critério de Aceitação | Cadastro desativado para de receber sem ser apagado | `TRANSCRICAO` | `[09:21] Bruno` |
| `PRD-CA-15` | `docs/PRD.md` | Critério de Aceitação | Cliente que passa de 10 segundos tem a tentativa contada como falha e entra na retentativa | `TRANSCRICAO` | `[09:42] Diego` |
| `PRD-CA-16` | `docs/PRD.md` | Critério de Aceitação | Notificação acima de 64 KB não é enviada e nada chega cortado | `TRANSCRICAO` | `[09:23] Sofia · [09:24] Diego` |
| `PRD-CA-17` | `docs/PRD.md` | Critério de Aceitação | Notificação repetida chega com o mesmo identificador e o cliente a descarta | `TRANSCRICAO` | `[09:25] Diego` |
| `PRD-CA-18` | `docs/PRD.md` | Critério de Aceitação | Cliente com mais de um cadastro identifica qual deles originou cada notificação | `TRANSCRICAO` | `[09:44] Sofia` |
| `PRD-CEN-01` | `docs/PRD.md` | Cenário de uso | Integração inicial: cadastro assinando `SHIPPED` e `DELIVERED`, com entrega do segredo gerado | `TRANSCRICAO` | `[09:33] Marcos` |
| `PRD-CEN-02` | `docs/PRD.md` | Cenário de uso | Indisponibilidade do cliente por duas horas; as notificações do período chegam quando ele volta | `TRANSCRICAO` | `[09:16] Diego` |
| `PRD-CEN-03` | `docs/PRD.md` | Cenário de uso | Investigação: o operador lê no histórico as tentativas, o código de resposta e o tempo de cada uma | `TRANSCRICAO` | `[09:34] Marcos` |
| `PRD-CEN-04` | `docs/PRD.md` | Cenário de uso | Credencial comprometida: rotação com as duas credenciais válidas durante 24 horas | `TRANSCRICAO` | `[09:21] Sofia` |
| `PRD-CEN-05` | `docs/PRD.md` | Cenário de uso | Cliente com mais de um cadastro identifica a origem de cada envio; o filtro olha todos os cadastros antes de gerar o evento | `TRANSCRICAO` | `[09:44] Sofia · [09:34] Bruno` |
| `PRD-DEP-01` | `docs/PRD.md` | Dependência | Revisão de segurança da Sofia, dois dias úteis antes do deploy — bloqueante | `TRANSCRICAO` | `[09:46] Sofia` |
| `PRD-DEP-02` | `docs/PRD.md` | Dependência | MySQL já existente; nenhuma infraestrutura nova a provisionar | `TRANSCRICAO` | `[09:07] Diego` |
| `PRD-DEP-03` | `docs/PRD.md` | Dependência | Endereço `https` acessível no lado do cliente, que cada um precisa expor e informar | `TRANSCRICAO` | `[09:23] Sofia` |
| `PRD-DEP-04` | `docs/PRD.md` | Dependência | Documentação de integração no portal do desenvolvedor, assumida por produto | `TRANSCRICAO` | `[09:26] Marcos · [09:40] Marcos` |
| `PRD-DEP-05` | `docs/PRD.md` | Dependência | Definição do percentil de latência com os clientes — pendente, ver OBJ-01 | `TRANSCRICAO` | `[09:02] Marcos` |
| `PRD-ACE-01` | `docs/PRD.md` | Aceite de nível de serviço | Produto aceitou o piso de 2 segundos em nome dos clientes | `TRANSCRICAO` | `[09:10] Marcos · [09:10] Larissa` |
| `PRD-ACE-02` | `docs/PRD.md` | Aceite de nível de serviço | Produto aceitou a janela de retentativa de quase 15 horas em nome dos clientes | `TRANSCRICAO` | `[09:17] Marcos` |
| `PRD-ACE-03` | `docs/PRD.md` | Pendência de produto | Nenhum dos dois aceites foi validado com os três clientes | `TRANSCRICAO` | `[09:49] Marcos` |
| `PRD-ENT-01` | `docs/PRD.md` | Entregável de produto | Documentação destacada da deduplicação no portal do desenvolvedor | `TRANSCRICAO` | `[09:26] Marcos` |
| `PRD-ENT-02` | `docs/PRD.md` | Entregável de produto | Guia de integração via API, única superfície do cliente na ausência de painel | `TRANSCRICAO` | `[09:40] Marcos` |
| `PRD-ENT-03` | `docs/PRD.md` | Entregável de produto | Confirmação de prazo com a Atlas e atualização dos três clientes | `TRANSCRICAO` | `[09:47] Marcos · [09:49] Marcos` |
| `PRD-EV-01` | `docs/PRD.md` | Escopo | O vocabulário assinável são os seis status do ciclo de vida do pedido | `CODIGO` | `prisma/schema.prisma:16-23` |
| `PRD-EV-02` | `docs/PRD.md` | Escopo | Sobram cinco destinos assináveis; `PENDING` nunca é destino de transição | `CODIGO` | `src/modules/orders/order.status.ts:3-10` |
| `PRD-EV-03` | `docs/PRD.md` | Escopo | A assinatura típica é `SHIPPED` + `DELIVERED`; a completa é válida e a ordem por pedido se mantém | `TRANSCRICAO` | `[09:33] Marcos · [09:12] Larissa` |
| `PRD-FE-01` | `docs/PRD.md` | Fora de Escopo | Alerta por e-mail quando o webhook do cliente falha — Adiado — próxima fase | `TRANSCRICAO` | `[09:37] Larissa` |
| `PRD-FE-02` | `docs/PRD.md` | Fora de Escopo | Painel visual para o cliente acompanhar entregas — Descartado desta feature | `TRANSCRICAO` | `[09:40] Larissa` |
| `PRD-FE-03` | `docs/PRD.md` | Fora de Escopo | Recebimento de webhooks enviados pelo cliente — Descartado — direção única | `TRANSCRICAO` | `[09:02] Marcos` |
| `PRD-FE-04` | `docs/PRD.md` | Fora de Escopo | Arquivamento e expurgo de eventos antigos — Adiado | `TRANSCRICAO` | `[09:08] Diego` |
| `PRD-FE-05` | `docs/PRD.md` | Fora de Escopo | Limite de taxa de envio para um mesmo cliente — Em observação | `TRANSCRICAO` | `[09:39] Diego` |
| `PRD-FE-06` | `docs/PRD.md` | Fora de Escopo | Garantia de ordem global entre pedidos diferentes — Adiado | `TRANSCRICAO` | `[09:13] Diego` |
| `PRD-OBJ-01` | `docs/PRD.md` | Objetivo / Métrica | Notificar em tempo percebido como real — meta: abaixo de 10 segundos, medido como percentil | `TRANSCRICAO` | `[09:02] Marcos` |
| `PRD-OBJ-02` | `docs/PRD.md` | Objetivo / Métrica | Não perder evento — meta: Zero | `TRANSCRICAO` | `[09:06] Diego · [09:40] Bruno` |
| `PRD-OBJ-03` | `docs/PRD.md` | Objetivo / Métrica | Não degradar a operação existente — meta: Zero | `TRANSCRICAO` | `[09:04] Bruno` |
| `PRD-OBJ-04` | `docs/PRD.md` | Objetivo / Métrica | Atender os três clientes que pediram — meta: 3 | `TRANSCRICAO` | `[09:00] Marcos` |
| `PRD-OBJ-05` | `docs/PRD.md` | Objetivo / Métrica | Entregar dentro do compromisso comercial — meta: 3 sprints, com a revisão de segurança inclusa | `TRANSCRICAO` | `[09:46] Larissa · [09:46] Sofia` |
| `PRD-OBJ-06` | `docs/PRD.md` | Objetivo / Métrica | Reduzir a carga de consulta que os clientes fazem hoje — sem meta numérica | `TRANSCRICAO` | `[09:00] Marcos` |
| `PRD-PE-01` | `docs/PRD.md` | Marco de entrega | Modelagem do registro de eventos e da fila de falhas definitivas, uma sprint | `TRANSCRICAO` | `[09:46] Larissa` |
| `PRD-PE-02` | `docs/PRD.md` | Marco de entrega | Worker e retentativa, uma sprint | `TRANSCRICAO` | `[09:46] Larissa` |
| `PRD-PE-03` | `docs/PRD.md` | Marco de entrega | CRUD de configuração e histórico de entregas, meia sprint | `TRANSCRICAO` | `[09:46] Larissa` |
| `PRD-PE-04` | `docs/PRD.md` | Marco de entrega | Integração no serviço de pedidos e testes ponta a ponta, meia sprint | `TRANSCRICAO` | `[09:46] Larissa` |
| `PRD-PE-05` | `docs/PRD.md` | Marco de entrega | Assinatura, esquemas e validações correm por dentro dos blocos; o pacote fecha em três sprints | `TRANSCRICAO` | `[09:46] Larissa · [09:47] Larissa` |
| `PRD-PR-01` | `docs/PRD.md` | Restrição de contexto | O time é pequeno e quem sobe infraestrutura também a mantém | `TRANSCRICAO` | `[09:07] Diego · [09:07] Larissa` |
| `PRD-PR-02` | `docs/PRD.md` | Restrição de contexto | A data de fim de novembro veio do cliente e a estimativa teve de caber nela | `TRANSCRICAO` | `[09:45] Marcos` |
| `PRD-PR-03` | `docs/PRD.md` | Premissa | Existe um usuário do nosso sistema que representa o cliente | `TRANSCRICAO` | `[09:32] Marcos` |
| `PRD-R-01` | `docs/PRD.md` | Risco | Um defeito no módulo novo derruba a mudança de status, a operação central do sistema — Média/Crítico | `TRANSCRICAO` | `[09:40] Bruno` |
| `PRD-R-02` | `docs/PRD.md` | Risco | A promessa de 10 segundos não se cumpre para clientes que respondem devagar — Alta/Médio | `TRANSCRICAO` | `[09:02] Marcos` |
| `PRD-R-03` | `docs/PRD.md` | Risco | Qualquer usuário autenticado obtém o segredo de qualquer cliente — Alta/Alto | `TRANSCRICAO` | `[09:37] Sofia` |
| `PRD-R-04` | `docs/PRD.md` | Risco | Cliente não implementa deduplicação e processa a mesma notificação duas vezes — Média/Médio | `TRANSCRICAO` | `[09:26] Marcos` |
| `PRD-R-05` | `docs/PRD.md` | Risco | Falhas se acumulam sem ninguém perceber, porque não há alerta nesta fase — Média/Médio | `TRANSCRICAO` | `[09:37] Larissa` |
| `PRD-R-06` | `docs/PRD.md` | Risco | Cliente cadastra endereço errado e só descobre no dia seguinte — Média/Baixo | `TRANSCRICAO` | `[09:34] Marcos` |
| `PRD-R-07` | `docs/PRD.md` | Risco | Segredo vaza em log da aplicação — já aconteceu com um cliente — Baixa/Alto | `TRANSCRICAO` | `[09:22] Diego` |
| `PRD-R-08` | `docs/PRD.md` | Risco | Rajada de mudanças inunda o sistema de um cliente — Baixa/Médio | `TRANSCRICAO` | `[09:38] Diego` |
| `PRD-RF-01` | `docs/PRD.md` | Requisito Funcional | O operador cadastra um endereço de webhook para um cliente | `TRANSCRICAO` | `[09:31] Marcos` |
| `PRD-RF-02` | `docs/PRD.md` | Requisito Funcional | O segredo de assinatura é gerado pela plataforma e devolvido na criação | `TRANSCRICAO` | `[09:31] Marcos` |
| `PRD-RF-03` | `docs/PRD.md` | Requisito Funcional | O cadastro define quais mudanças de status quer receber | `TRANSCRICAO` | `[09:33] Marcos` |
| `PRD-RF-04` | `docs/PRD.md` | Requisito Funcional | O operador edita um cadastro existente | `TRANSCRICAO` | `[09:33] Bruno` |
| `PRD-RF-05` | `docs/PRD.md` | Requisito Funcional | O operador remove um cadastro | `TRANSCRICAO` | `[09:33] Bruno` |
| `PRD-RF-06` | `docs/PRD.md` | Requisito Funcional | O operador lista os cadastros de um cliente | `TRANSCRICAO` | `[09:33] Bruno` |
| `PRD-RF-07` | `docs/PRD.md` | Requisito Funcional | O cadastro guarda um estado ativo, ao lado do endereço, do segredo e do cliente a que pertence | `TRANSCRICAO` | `[09:21] Bruno · [09:21] Sofia` |
| `PRD-RF-08` | `docs/PRD.md` | Requisito Funcional | O operador solicita novo segredo, com o anterior válido por 24 horas | `TRANSCRICAO` | `[09:21] Sofia` |
| `PRD-RF-09` | `docs/PRD.md` | Requisito Funcional | Toda mudança de status gera notificação para os cadastros que a assinam | `TRANSCRICAO` | `[09:00] Marcos · [09:06] Diego` |
| `PRD-RF-10` | `docs/PRD.md` | Requisito Funcional | Cada notificação vai assinada, para o cliente verificar origem e integridade | `TRANSCRICAO` | `[09:20] Sofia` |
| `PRD-RF-11` | `docs/PRD.md` | Requisito Funcional | Notificação que falha é retentada automaticamente | `TRANSCRICAO` | `[09:15] Diego` |
| `PRD-RF-12` | `docs/PRD.md` | Requisito Funcional | O operador consulta o histórico de entregas de um cadastro, com resultado, resposta e tempo | `TRANSCRICAO` | `[09:34] Marcos` |
| `PRD-RF-13` | `docs/PRD.md` | Requisito Funcional | Um administrador reprocessa manualmente uma notificação que esgotou as tentativas | `TRANSCRICAO` | `[09:35] Diego` |
| `PRD-RF-14` | `docs/PRD.md` | Requisito Funcional | O reprocessamento registra quem o executou | `TRANSCRICAO` | `[09:36] Sofia` |
| `PRD-RNF-01` | `docs/PRD.md` | Requisito Não Funcional | A notificação não pode atrasar nem afetar a mudança de status | `TRANSCRICAO` | `[09:04] Bruno` |
| `PRD-RNF-02` | `docs/PRD.md` | Requisito Não Funcional | Se a mudança de status é confirmada, o evento existe; se falha, o evento não existe | `TRANSCRICAO` | `[09:06] Diego · [09:40] Bruno` |
| `PRD-RNF-03` | `docs/PRD.md` | Requisito Não Funcional | A entrega é garantida ao menos uma vez; o cliente deve deduplicar | `TRANSCRICAO` | `[09:24] Diego` |
| `PRD-RNF-04` | `docs/PRD.md` | Requisito Não Funcional | Cinco retentativas com intervalos crescentes, cobrindo cerca de 15 horas desde a primeira falha | `TRANSCRICAO` | `[09:15] Diego · [09:17] Diego` |
| `PRD-RNF-05` | `docs/PRD.md` | Requisito Não Funcional | Cada notificação tem no máximo 64 KB; acima disso, erro, sem truncar | `TRANSCRICAO` | `[09:23] Sofia · [09:24] Larissa` |
| `PRD-RNF-06` | `docs/PRD.md` | Requisito Não Funcional | O endereço cadastrado tem de usar `https`; `http` é recusado | `TRANSCRICAO` | `[09:23] Sofia` |
| `PRD-RNF-07` | `docs/PRD.md` | Requisito Não Funcional | Cada cadastro tem segredo próprio; não há segredo compartilhado da plataforma | `TRANSCRICAO` | `[09:21] Sofia` |
| `PRD-RNF-08` | `docs/PRD.md` | Requisito Não Funcional | O cliente tem 10 segundos para responder; além disso, a tentativa é falha | `TRANSCRICAO` | `[09:42] Diego` |
| `PRD-RNF-09` | `docs/PRD.md` | Requisito Não Funcional | Notificações do mesmo pedido chegam na ordem em que ocorreram | `TRANSCRICAO` | `[09:12] Diego` |
| `PRD-RNF-10` | `docs/PRD.md` | Requisito Não Funcional | A feature não introduz infraestrutura nova | `TRANSCRICAO` | `[09:07] Diego` |
| `PRD-RNF-11` | `docs/PRD.md` | Requisito Não Funcional | O reprocessamento exige papel de administrador | `TRANSCRICAO` | `[09:36] Sofia` |
| `PRD-RNF-12` | `docs/PRD.md` | Requisito Não Funcional | Nenhum contrato de API existente é alterado | `TRANSCRICAO` | `[09:04] Bruno` |
| `PRD-TO-01` | `docs/PRD.md` | Trade-off | Notificação assíncrona em vez de imediata: cliente lento nunca trava pedidos, ao preço do atraso | `TRANSCRICAO` | `[09:04] Bruno · [09:06] Diego` |
| `PRD-TO-02` | `docs/PRD.md` | Trade-off | Ao menos uma entrega em vez de exatamente uma: desenho simples, cliente trata duplicata | `TRANSCRICAO` | `[09:25] Diego · [09:26] Marcos` |
| `PRD-TO-03` | `docs/PRD.md` | Trade-off | Retentativa longa: manutenção planejada não perde nada, endereço errado só aparece no dia seguinte | `TRANSCRICAO` | `[09:17] Diego · [09:17] Marcos` |
| `PRD-TO-04` | `docs/PRD.md` | Trade-off | Notificação enxuta, sem os itens do pedido: payload barato, uma consulta a mais para quem precisa de detalhe | `TRANSCRICAO` | `[09:43] Diego` |
| `PRD-TO-05` | `docs/PRD.md` | Trade-off | Sem infraestrutura nova: cabe em três sprints, e a ordem só é garantida com um consumidor | `TRANSCRICAO` | `[09:07] Diego · [09:12] Diego` |
| `PRD-TO-06` | `docs/PRD.md` | Trade-off | Cadastro por qualquer usuário autenticado: integração sem fricção, sem verificação de posse | `TRANSCRICAO` | `[09:37] Sofia` |
| `NA-01` | `docs/PRD.md` | Decisão não arquitetural | Limite de 64 KB no payload — Larissa classifica na hora como requisito não funcional | `TRANSCRICAO` | `[09:24] Larissa` |
| `RFC-ALT-01` | `docs/RFC.md` | Alternativa Descartada | Disparo HTTP síncrono na mudança de status | `TRANSCRICAO` | `[09:04] Bruno · RFC §Alternativas consideradas` |
| `RFC-ALT-02` | `docs/RFC.md` | Alternativa Descartada | Redis Streams ou fila dedicada | `TRANSCRICAO` | `[09:07] Larissa · RFC §Alternativas consideradas` |
| `RFC-ALT-03` | `docs/RFC.md` | Alternativa Descartada | Trigger de banco para acordar o worker | `TRANSCRICAO` | `[09:09] Bruno · RFC §Alternativas consideradas` |
| `RFC-ALT-04` | `docs/RFC.md` | Alternativa Descartada | Marcar falha definitiva na própria outbox | `TRANSCRICAO` | `[09:17] Larissa · RFC §Alternativas consideradas` |
| `RFC-ALT-05` | `docs/RFC.md` | Alternativa Descartada | Retry indefinido, ou teto mais agressivo | `TRANSCRICAO` | `[09:16] Bruno · RFC §Alternativas consideradas` |
| `RFC-ALT-06` | `docs/RFC.md` | Alternativa Descartada | Exactly-once | `TRANSCRICAO` | `[09:25] Diego · RFC §Alternativas consideradas` |
| `RFC-ALT-07` | `docs/RFC.md` | Alternativa Descartada | Segredo único da plataforma | `TRANSCRICAO` | `[09:21] Sofia · RFC §Alternativas consideradas` |
| `RFC-ALT-08` | `docs/RFC.md` | Alternativa Descartada | Montar o evento na hora do envio | `TRANSCRICAO` | `[09:51] Bruno · RFC §Alternativas consideradas` |
| `RFC-Q01` | `docs/RFC.md` | Questão em Aberto | Limitar a taxa de envio para um mesmo cliente — dono Diego | `TRANSCRICAO` | `[09:38] Diego · [09:39] Diego · RFC §Questões em aberto` |
| `RFC-Q02` | `docs/RFC.md` | Questão em Aberto | Escalar para múltiplos workers, e o que acontece com a ordem por pedido — dono Diego | `TRANSCRICAO` | `[09:13] Diego · RFC §Questões em aberto` |
| `RFC-Q04` | `docs/RFC.md` | Questão em Aberto | Endurecer a autorização do cadastro; hoje não existe vínculo entre usuário e cliente — dona Sofia | `TRANSCRICAO` | `[09:37] Sofia · RFC §Questões em aberto` |
| `RFC-Q05` | `docs/RFC.md` | Questão em Aberto | Retenção e expurgo das linhas entregues e do histórico de tentativas — dono Diego | `TRANSCRICAO` | `[09:08] Diego · RFC §Questões em aberto` |
| `RFC-Q06` | `docs/RFC.md` | Questão em Aberto | Guarda do segredo em repouso: texto claro ou cifrado — dona Sofia | `TRANSCRICAO` | `[09:46] Sofia · RFC §Questões em aberto` |
| `RFC-IMP-01` | `docs/RFC.md` | Risco | Entra um processo novo para operar e a transação de `changeStatus` ganha mais um gate; infraestrutura nova, nenhuma | `TRANSCRICAO` | `[09:11] Diego · [09:07] Diego · RFC §Impacto e riscos` |
| `RFC-IMP-02` | `docs/RFC.md` | Risco | O ponto de integração fica dentro da transação mais quente; é o único risco capaz de causar dano fora da feature | `TRANSCRICAO` | `[09:40] Bruno · [09:41] Diego · RFC §Impacto e riscos` |
| `RFC-IMP-03` | `docs/RFC.md` | Risco | A promessa de dez segundos não fecha como garantia absoluta; o compromisso precisa ser percentil | `TRANSCRICAO` | `[09:02] Marcos · [09:42] Diego · RFC §Impacto e riscos` |
| `RFC-IMP-04` | `docs/RFC.md` | Risco | A janela de retry é longa, e a chegada à fila de mortos não dispara aviso ativo | `TRANSCRICAO` | `[09:17] Marcos · [09:37] Larissa · RFC §Impacto e riscos` |
| `RFC-IMP-05` | `docs/RFC.md` | Risco | Nenhum usuário está vinculado a um cliente; é o maior buraco do desenho, aceito como `Q04` | `TRANSCRICAO` | `[09:37] Sofia · RFC §Impacto e riscos` |
| `FDD-AC-01` | `docs/FDD.md` | Critério de Aceite Técnico | Mudança de status com endpoint assinante grava uma linha por endpoint | `TRANSCRICAO` | `[09:06] Diego · FDD §13` |
| `FDD-AC-02` | `docs/FDD.md` | Critério de Aceite Técnico | Erro na publicação faz rollback da mudança de status | `TRANSCRICAO` | `[09:40] Bruno · FDD §13` |
| `FDD-AC-03` | `docs/FDD.md` | Critério de Aceite Técnico | Mudança de status sem endpoint assinante não grava nada | `TRANSCRICAO` | `[09:34] Bruno · FDD §13` |
| `FDD-AC-04` | `docs/FDD.md` | Critério de Aceite Técnico | Status não assinado pelo endpoint não gera linha | `TRANSCRICAO` | `[09:33] Marcos · FDD §13` |
| `FDD-AC-05` | `docs/FDD.md` | Critério de Aceite Técnico | Cada linha nasce com `eventId` distinto | `TRANSCRICAO` | `[09:25] Diego · FDD §13` |
| `FDD-AC-06` | `docs/FDD.md` | Critério de Aceite Técnico | O payload é o estado do pedido no instante da mudança | `TRANSCRICAO` | `[09:52] Larissa · FDD §13` |
| `FDD-AC-07` | `docs/FDD.md` | Critério de Aceite Técnico | O worker entrega e marca `DELIVERED` | `TRANSCRICAO` | `[09:09] Diego · FDD §13` |
| `FDD-AC-08` | `docs/FDD.md` | Critério de Aceite Técnico | Resposta não-2xx agenda retentativa com o intervalo correto | `TRANSCRICAO` | `[09:17] Diego · FDD §13` |
| `FDD-AC-09` | `docs/FDD.md` | Critério de Aceite Técnico | Esgotadas as retentativas, a linha vira `FAILED` e entra na fila de mortos | `TRANSCRICAO` | `[09:18] Diego · FDD §13` |
| `FDD-AC-10` | `docs/FDD.md` | Critério de Aceite Técnico | Timeout conta como falha | `TRANSCRICAO` | `[09:42] Diego · FDD §13` |
| `FDD-AC-11` | `docs/FDD.md` | Critério de Aceite Técnico | Resposta 3xx conta como falha e não é seguida | `CODIGO` | `package.json:25-34 · FDD §13` |
| `FDD-AC-12` | `docs/FDD.md` | Critério de Aceite Técnico | Linha travada em `PROCESSING` volta para `PENDING` após o lease | `CODIGO` | `src/server.ts:13-21 · FDD §13` |
| `FDD-AC-13` | `docs/FDD.md` | Critério de Aceite Técnico | A reconciliação de lease não consome retentativa | `CODIGO` | `src/server.ts:13-21 · FDD §13` |
| `FDD-AC-14` | `docs/FDD.md` | Critério de Aceite Técnico | Assinatura reproduzível com o segredo do cadastro | `TRANSCRICAO` | `[09:20] Sofia · FDD §13` |
| `FDD-AC-15` | `docs/FDD.md` | Critério de Aceite Técnico | Durante a janela de rotação seguem duas assinaturas | `TRANSCRICAO` | `[09:21] Sofia · FDD §13` |
| `FDD-AC-16` | `docs/FDD.md` | Critério de Aceite Técnico | Expirada a janela, só a assinatura nova segue | `TRANSCRICAO` | `[09:22] Sofia · FDD §13` |
| `FDD-AC-17` | `docs/FDD.md` | Critério de Aceite Técnico | O segredo íntegro aparece só na criação e na rotação | `TRANSCRICAO` | `[09:31] Marcos · FDD §13` |
| `FDD-AC-18` | `docs/FDD.md` | Critério de Aceite Técnico | URL `http` é recusada | `TRANSCRICAO` | `[09:23] Sofia · FDD §13` |
| `FDD-AC-19` | `docs/FDD.md` | Critério de Aceite Técnico | Eventos do mesmo pedido são entregues em ordem | `TRANSCRICAO` | `[09:12] Diego · FDD §13` |
| `FDD-AC-20` | `docs/FDD.md` | Critério de Aceite Técnico | Um destino lento não atrasa os de outros pedidos | `TRANSCRICAO` | `[09:04] Bruno · FDD §13` |
| `FDD-AC-21` | `docs/FDD.md` | Critério de Aceite Técnico | Replay exige `ADMIN` | `TRANSCRICAO` | `[09:36] Sofia · FDD §13` |
| `FDD-AC-22` | `docs/FDD.md` | Critério de Aceite Técnico | Replay recoloca na fila preservando o `eventId` | `TRANSCRICAO` | `[09:25] Diego · FDD §13` |
| `FDD-AC-23` | `docs/FDD.md` | Critério de Aceite Técnico | Replay repetido é recusado com 409 | `CODIGO` | `src/shared/errors/http-errors.ts:33-38 · FDD §13` |
| `FDD-AC-24` | `docs/FDD.md` | Critério de Aceite Técnico | Replay grava quem executou | `TRANSCRICAO` | `[09:36] Sofia · FDD §13` |
| `FDD-AC-25` | `docs/FDD.md` | Critério de Aceite Técnico | Nenhum segredo aparece em log | `CODIGO` | `src/shared/logger/index.ts:4-11 · FDD §13` |
| `FDD-AC-26` | `docs/FDD.md` | Critério de Aceite Técnico | O `requestId` da requisição chega ao evento de entrega | `CODIGO` | `src/middlewares/request-logger.middleware.ts:6-8 · FDD §13` |
| `FDD-AC-27` | `docs/FDD.md` | Critério de Aceite Técnico | Endpoint desativado não recebe entrega | `TRANSCRICAO` | `[09:21] Bruno · FDD §13` |
| `FDD-AC-28` | `docs/FDD.md` | Critério de Aceite Técnico | Payload acima do teto vai direto para a fila de mortos | `TRANSCRICAO` | `[09:24] Larissa · FDD §13` |
| `FDD-AC-29` | `docs/FDD.md` | Critério de Aceite Técnico | A suíte existente passa sem alteração | `TRANSCRICAO` | `[09:29] Bruno · FDD §13` |
| `FDD-AC-30` | `docs/FDD.md` | Critério de Aceite Técnico | Com `WEBHOOK_WORKER_ENABLED=false` nada é publicado | `CODIGO` | `src/config/env.ts:4-27 · FDD §13` |
| `FDD-CONV-01` | `docs/FDD.md` | Restrição do Código | Prefixo `/api/v1` em todas as rotas | `CODIGO` | `src/app.ts:67 · FDD §11.6` |
| `FDD-CONV-02` | `docs/FDD.md` | Restrição do Código | Envelope de erro `{ error: { code, message, details? } }` | `CODIGO` | `src/middlewares/error.middleware.ts:15-24 · FDD §11.4` |
| `FDD-CONV-03` | `docs/FDD.md` | Restrição do Código | Envelope de listagem `{ data, pagination }` | `CODIGO` | `src/shared/http/response.ts:8-11 · FDD §7.2` |
| `FDD-CONV-04` | `docs/FDD.md` | Restrição do Código | PK `String @id @default(uuid()) @db.Char(36)` e `@@map` em snake_case | `CODIGO` | `prisma/schema.prisma:25-138 · FDD §6.1` |
| `FDD-CONV-05` | `docs/FDD.md` | Restrição do Código | `AppError` expõe `errorCode`, não `code` | `CODIGO` | `src/shared/errors/app-error.ts:3-15 · FDD §11.3` |
| `FDD-CONV-06` | `docs/FDD.md` | Restrição do Código | `requireRole` disponível; papéis `ADMIN` e `OPERATOR` | `CODIGO` | `src/middlewares/auth.middleware.ts:49-61 · FDD §11.5` |
| `FDD-CONV-07` | `docs/FDD.md` | Restrição do Código | Sem cliente HTTP nas dependências: `fetch` nativo do Node 20 | `CODIGO` | `package.json:25-34 · FDD §1` |
| `FDD-CONV-08` | `docs/FDD.md` | Restrição do Código | O `redact` do Pino não cobre `secret` nem `signature` | `CODIGO` | `src/shared/logger/index.ts:4-11 · FDD §10.4` |
| `FDD-CONV-09` | `docs/FDD.md` | Restrição do Código | `requestId` em `X-Request-Id`, base da correlação | `CODIGO` | `src/middlewares/request-logger.middleware.ts:6-8 · FDD §10.3` |
| `FDD-CONV-10` | `docs/FDD.md` | Restrição do Código | O encerramento atual não drena trabalho em background | `CODIGO` | `src/server.ts:13-21 · FDD §9.5` |
| `FDD-CONV-11` | `docs/FDD.md` | Restrição do Código | Limpeza de tabelas no `beforeEach` da suíte | `CODIGO` | `tests/setup.ts:9-15 · FDD §6.5` |
| `FDD-CONV-12` | `docs/FDD.md` | Restrição do Código | Estoque só muda em `PENDING→PAID` e no cancelamento | `CODIGO` | `src/modules/orders/order.status.ts:29-37 · FDD §11.1` |
| `FDD-CONV-13` | `docs/FDD.md` | Restrição do Código | Nenhum broker, cache ou fila no ambiente: o compose sobe só o MySQL | `CODIGO` | `docker-compose.yml:2-24 · FDD §1` |
| `FDD-CONV-14` | `docs/FDD.md` | Restrição do Código | A transação chega ao gancho como `Prisma.TransactionClient`, não como cliente raiz | `CODIGO` | `src/modules/orders/order.service.ts:131 · FDD §1` |
| `FDD-CONV-15` | `docs/FDD.md` | Restrição do Código | `createdAt @default(now())` e `updatedAt @updatedAt` em todo model de domínio | `CODIGO` | `prisma/schema.prisma:25-38 · FDD §6.1` |
| `FDD-CONV-16` | `docs/FDD.md` | Restrição do Código | Coluna `Json` para estrutura que não ganha tabela própria | `CODIGO` | `prisma/schema.prisma:46 · FDD §6.1` |
| `FDD-CONV-17` | `docs/FDD.md` | Restrição do Código | Enum em `SCREAMING_SNAKE` | `CODIGO` | `prisma/schema.prisma:11-14 · FDD §6.1` |
| `FDD-CONV-18` | `docs/FDD.md` | Restrição do Código | Índice explícito em toda coluna usada como filtro | `CODIGO` | `prisma/schema.prisma:92-95 · FDD §6.1` |
| `FDD-CONV-19` | `docs/FDD.md` | Restrição do Código | Montagem manual em `buildControllers`, sem container de injeção | `CODIGO` | `src/app.ts:26-53 · FDD §11.6` |
| `FDD-CONV-20` | `docs/FDD.md` | Restrição do Código | Registro de router por prefixo em `buildApiRouter` | `CODIGO` | `src/routes/index.ts:21-31 · FDD §11.6` |
| `FDD-CONV-21` | `docs/FDD.md` | Restrição do Código | Variável de ambiente validada por Zod, com `process.exit(1)` quando falta obrigatória | `CODIGO` | `src/config/env.ts:4-27 · FDD §11.7` |
| `FDD-CONV-22` | `docs/FDD.md` | Restrição do Código | `changeStatus` ganha um quarto parâmetro `requestId`, opcional e posicional | `CODIGO` | `src/modules/orders/order.service.ts:126-130 · FDD §11.1` |
| `FDD-CT-01` | `docs/FDD.md` | Contrato Público | `POST /api/v1/webhooks` — cadastrar endpoint | `TRANSCRICAO` | `[09:31] Marcos · FDD §7.1` |
| `FDD-CT-02` | `docs/FDD.md` | Contrato Público | `GET /api/v1/webhooks` — listar por cliente | `TRANSCRICAO` | `[09:33] Bruno · FDD §7.2` |
| `FDD-CT-03` | `docs/FDD.md` | Contrato Público | `GET /api/v1/webhooks/:id` — detalhe | `TRANSCRICAO` | `[09:33] Bruno · FDD §7.3` |
| `FDD-CT-04` | `docs/FDD.md` | Contrato Público | `PATCH /api/v1/webhooks/:id` — editar | `TRANSCRICAO` | `[09:33] Bruno · FDD §7.4` |
| `FDD-CT-05` | `docs/FDD.md` | Contrato Público | `DELETE /api/v1/webhooks/:id` — remover | `TRANSCRICAO` | `[09:33] Bruno · FDD §7.5` |
| `FDD-CT-06` | `docs/FDD.md` | Contrato Público | `POST /api/v1/webhooks/:id/rotate-secret` — rotacionar segredo | `TRANSCRICAO` | `[09:21] Sofia · FDD §7.6` |
| `FDD-CT-07` | `docs/FDD.md` | Contrato Público | `GET /api/v1/webhooks/:id/deliveries` — histórico de entregas | `TRANSCRICAO` | `[09:34] Marcos · FDD §7.7` |
| `FDD-CT-09` | `docs/FDD.md` | Contrato Público | `POST /api/v1/admin/webhooks/dead-letter/:id/replay` — reprocessar | `TRANSCRICAO` | `[09:35] Diego · FDD §7.9` |
| `FDD-CT-10` | `docs/FDD.md` | Contrato Público | Contrato de saída — a entrega ao cliente, corpo e cabeçalhos | `TRANSCRICAO` | `[09:43] Diego · [09:44] Diego · FDD §7.10` |
| `FDD-CT-11` | `docs/FDD.md` | Contrato Público | `webhook.schemas.ts` — os schemas Zod que sustentam os contratos | `CODIGO` | `src/modules/customers/customer.schemas.ts:23 · src/middlewares/validate.middleware.ts:18-20 · FDD §7.11` |
| `FDD-CT-12` | `docs/FDD.md` | Contrato Público | Corpos de erro, um por família, no envelope real | `CODIGO` | `src/middlewares/error.middleware.ts:15-24 · FDD §7.12` |
| `FDD-HDR-01` | `docs/FDD.md` | Cabeçalho de saída | `X-Event-Id` — UUID gerado na entrada da outbox, estável entre retentativas e no replay | `TRANSCRICAO` | `[09:25] Diego · FDD §7.10` |
| `FDD-HDR-02` | `docs/FDD.md` | Cabeçalho de saída | `X-Webhook-Id` — identificador do cadastro que originou a entrega | `TRANSCRICAO` | `[09:44] Sofia · FDD §7.10` |
| `FDD-HDR-03` | `docs/FDD.md` | Cabeçalho de saída | `X-Timestamp` — instante do envio, em segundos desde a época Unix | `TRANSCRICAO` | `[09:44] Diego · FDD §7.10` |
| `FDD-HDR-04` | `docs/FDD.md` | Cabeçalho de saída | `X-Signature` — `v1=<hex>`; na rotação, dois valores separados por vírgula | `TRANSCRICAO` | `[09:20] Sofia · FDD §7.10` |
| `FDD-HDR-05` | `docs/FDD.md` | Cabeçalho de saída | `Content-Type: application/json` | `TRANSCRICAO` | `[09:44] Diego · FDD §7.10` |
| `FDD-ERR-01` | `docs/FDD.md` | Erro previsto | `WEBHOOK_NOT_FOUND` — endpoint inexistente em consulta, edição, remoção, rotação ou entregas | `TRANSCRICAO` | `[09:28] Bruno · FDD §8` |
| `FDD-ERR-02` | `docs/FDD.md` | Erro previsto | `WEBHOOK_CUSTOMER_NOT_FOUND` — `customerId` do corpo não existe em `customers` | `CODIGO` | `src/shared/errors/http-errors.ts:3-63 · FDD §8` |
| `FDD-ERR-03` | `docs/FDD.md` | Erro previsto | `WEBHOOK_INVALID_STATUS` — `subscribedStatuses` com valor fora de `OrderStatus` | `TRANSCRICAO` | `[09:33] Marcos · FDD §8` |
| `FDD-ERR-04` | `docs/FDD.md` | Erro previsto | `WEBHOOK_NO_STATUS_SUBSCRIBED` — lista vazia; o cadastro não receberia nada | `TRANSCRICAO` | `[09:33] Marcos · FDD §8` |
| `FDD-ERR-05` | `docs/FDD.md` | Erro previsto | `WEBHOOK_ENDPOINT_INACTIVE` — replay para endpoint com `active = false` | `TRANSCRICAO` | `[09:21] Bruno · FDD §8` |
| `FDD-ERR-06` | `docs/FDD.md` | Erro previsto | `WEBHOOK_DEAD_LETTER_NOT_FOUND` — identificador inexistente na fila de mortos | `TRANSCRICAO` | `[09:35] Diego · FDD §8` |
| `FDD-ERR-07` | `docs/FDD.md` | Erro previsto | `WEBHOOK_ALREADY_REPLAYED` — entrada com `replayedAt` preenchido | `TRANSCRICAO` | `[09:35] Diego · FDD §8` |
| `FDD-ERR-08` | `docs/FDD.md` | Erro previsto | `WEBHOOK_DELIVERY_HTTP_ERROR` — resposta fora da faixa 2xx | `TRANSCRICAO` | `[09:15] Diego · FDD §8` |
| `FDD-ERR-09` | `docs/FDD.md` | Erro previsto | `WEBHOOK_DELIVERY_TIMEOUT` — destino não respondeu dentro do limite | `TRANSCRICAO` | `[09:42] Diego · FDD §8` |
| `FDD-ERR-10` | `docs/FDD.md` | Erro previsto | `WEBHOOK_DELIVERY_NETWORK_ERROR` — falha de DNS, TLS ou conexão recusada | `CODIGO` | `src/shared/errors/http-errors.ts:3-63 · FDD §8` |
| `FDD-ERR-11` | `docs/FDD.md` | Erro previsto | `WEBHOOK_DELIVERY_REDIRECT_REFUSED` — resposta 3xx; não seguimos redirecionamento | `CODIGO` | `src/shared/errors/http-errors.ts:3-63 · FDD §8` |
| `FDD-ERR-12` | `docs/FDD.md` | Erro previsto | `WEBHOOK_RETRIES_EXHAUSTED` — motivo final registrado ao mover para a fila de mortos | `TRANSCRICAO` | `[09:15] Diego · FDD §8` |
| `FDD-ERR-13` | `docs/FDD.md` | Erro previsto | `WEBHOOK_ENDPOINT_INACTIVE` — endpoint desativado entre a inserção e a entrega | `TRANSCRICAO` | `[09:21] Bruno · FDD §8` |
| `FDD-ERR-14` | `docs/FDD.md` | Erro previsto | `WEBHOOK_PAYLOAD_TOO_LARGE` — evento acima do teto de tamanho | `TRANSCRICAO` | `[09:24] Larissa · FDD §8` |
| `FDD-ERR-15` | `docs/FDD.md` | Erro previsto | `WEBHOOK_SECRET_REQUIRED` — o campo não existe na entrada; o segredo é gerado por nós | `TRANSCRICAO` | `[09:28] Bruno · FDD §8.4` |
| `FDD-ERR-16` | `docs/FDD.md` | Erro previsto | `WEBHOOK_INVALID_URL` — a verificação vive no schema Zod e produz `VALIDATION_ERROR` | `TRANSCRICAO` | `[09:28] Bruno · FDD §8.4` |
| `FDD-ERR-21` | `docs/FDD.md` | Erro previsto | `WEBHOOK_HAS_PENDING_EVENTS` — 409 na remoção de endpoint com evento pendente | `CODIGO` | `src/modules/orders/order.service.ts:181-192 · FDD §6.7` |
| `FDD-ENV-01` | `docs/FDD.md` | Parâmetro de Configuração | `WEBHOOK_BATCH_SIZE`, default 50 | `CODIGO` | `src/config/env.ts:15-23 · FDD §9.1` |
| `FDD-ENV-02` | `docs/FDD.md` | Parâmetro de Configuração | `WEBHOOK_CONCURRENCY`, default 10 | `CODIGO` | `src/config/env.ts:15-23 · FDD §9.1` |
| `FDD-ENV-03` | `docs/FDD.md` | Parâmetro de Configuração | `WEBHOOK_LEASE_TIMEOUT_MS`, default 60000 | `CODIGO` | `src/config/env.ts:15-23 · FDD §9.1` |
| `FDD-ENV-04` | `docs/FDD.md` | Parâmetro de Configuração | `WEBHOOK_WORKER_ENABLED`, default `true` — é o gate de publicação e de consumo | `CODIGO` | `src/config/env.ts:15-23 · FDD §9.1` |
| `FDD-FE-01` | `docs/FDD.md` | Fora de Escopo | Alerta por e-mail quando o webhook falha | `TRANSCRICAO` | `[09:37] Larissa · FDD §3` |
| `FDD-FE-02` | `docs/FDD.md` | Fora de Escopo | Painel visual para o cliente | `TRANSCRICAO` | `[09:40] Larissa · FDD §3` |
| `FDD-FE-03` | `docs/FDD.md` | Fora de Escopo | Recebimento de webhooks; a direção é só de saída | `TRANSCRICAO` | `[09:02] Marcos · FDD §3` |
| `FDD-FE-04` | `docs/FDD.md` | Fora de Escopo | Arquivamento e expurgo de linhas antigas — `Q05` | `TRANSCRICAO` | `[09:08] Diego · FDD §3` |
| `FDD-FE-05` | `docs/FDD.md` | Fora de Escopo | Limite de taxa de envio por cliente — `Q01` | `TRANSCRICAO` | `[09:39] Larissa · FDD §3` |
| `FDD-FE-06` | `docs/FDD.md` | Fora de Escopo | Múltiplos workers e ordem global — `Q02` | `TRANSCRICAO` | `[09:13] Diego · FDD §3` |
| `FDD-FE-10` | `docs/FDD.md` | Fora de Escopo | Evento de criação de pedido; há um único tipo, `order.status_changed` | `TRANSCRICAO` | `[09:43] Diego · FDD §3` |
| `FDD-INT-1` | `docs/FDD.md` | Integração com o Código | Gancho na transação de `changeStatus` | `CODIGO` | `src/modules/orders/order.service.ts · FDD §11.1` |
| `FDD-INT-2` | `docs/FDD.md` | Integração com o Código | Origem do snapshot no repositório de pedidos | `CODIGO` | `src/modules/orders/order.repository.ts · FDD §11.2` |
| `FDD-INT-3` | `docs/FDD.md` | Integração com o Código | Classes de erro novas no molde existente | `CODIGO` | `src/shared/errors/ · FDD §11.3` |
| `FDD-INT-4` | `docs/FDD.md` | Integração com o Código | Middleware de erro não muda | `CODIGO` | `src/middlewares/error.middleware.ts · FDD §11.4` |
| `FDD-INT-5` | `docs/FDD.md` | Integração com o Código | Reuso direto de `authenticate` e `requireRole` | `CODIGO` | `src/middlewares/auth.middleware.ts · FDD §11.5` |
| `FDD-INT-6` | `docs/FDD.md` | Integração com o Código | Montagem do módulo e registro do router | `CODIGO` | `src/app.ts:26-53 · src/routes/index.ts:21-31 · FDD §11.6` |
| `FDD-INT-7` | `docs/FDD.md` | Integração com o Código | O worker como irmão do servidor, com o mesmo `envSchema` | `CODIGO` | `src/config/env.ts:4-27 · src/server.ts:13-21 · FDD §11.7` |
| `FDD-INT-8` | `docs/FDD.md` | Integração com o Código | Limpeza das tabelas novas e fábricas de teste | `CODIGO` | `tests/setup.ts:9-15 · tests/helpers/factories.ts · FDD §11.8` |
| `FDD-INT-9` | `docs/FDD.md` | Integração com o Código | Esqueleto de `src/worker.ts`: laço, drenagem em `SIGINT`/`SIGTERM`, limitador de concorrência | `CODIGO` | `src/server.ts:13-21 · FDD §11.9` |
| `FDD-MOD-WebhookDeadLetter` | `docs/FDD.md` | Modelo de Dados | Model `WebhookDeadLetter` | `CODIGO` | `prisma/schema.prisma:25-138 · FDD §6.3` |
| `FDD-MOD-WebhookDelivery` | `docs/FDD.md` | Modelo de Dados | Model `WebhookDelivery` | `CODIGO` | `prisma/schema.prisma:25-138 · FDD §6.3` |
| `FDD-MOD-WebhookEndpoint` | `docs/FDD.md` | Modelo de Dados | Model `WebhookEndpoint` | `CODIGO` | `prisma/schema.prisma:25-138 · FDD §6.3` |
| `FDD-MOD-WebhookOutbox` | `docs/FDD.md` | Modelo de Dados | Model `WebhookOutbox` | `CODIGO` | `prisma/schema.prisma:25-138 · FDD §6.3` |
| `FDD-MOD-Migracao` | `docs/FDD.md` | Modelo de Dados | DDL das quatro tabelas no formato que `prisma migrate` gera | `CODIGO` | `prisma/migrations/20260519182739_init/migration.sql · FDD §6.6` |
| `FDD-MOD-Exclusao` | `docs/FDD.md` | Modelo de Dados | Política de exclusão: `RESTRICT` nas chaves para `webhook_endpoints`, `CASCADE` em `webhook_outbox.orderId` | `CODIGO` | `prisma/schema.prisma:25-138 · FDD §6.7` |
| `FDD-OBJ-01` | `docs/FDD.md` | Objetivo Técnico | Registrar o evento atomicamente com a mudança de status | `TRANSCRICAO` | `[09:40] Bruno · FDD §2` |
| `FDD-OBJ-02` | `docs/FDD.md` | Objetivo Técnico | Nenhuma chamada HTTP de saída dentro da transação de pedidos | `TRANSCRICAO` | `[09:04] Bruno · FDD §2` |
| `FDD-OBJ-03` | `docs/FDD.md` | Objetivo Técnico | Entrega em polling curto, sem dependência nova no projeto | `TRANSCRICAO` | `[09:09] Diego · FDD §2` |
| `FDD-OBJ-04` | `docs/FDD.md` | Objetivo Técnico | Tolerar indisponibilidade do destino sem perder evento | `TRANSCRICAO` | `[09:15] Diego · FDD §2` |
| `FDD-OBJ-05` | `docs/FDD.md` | Objetivo Técnico | Permitir ao cliente verificar origem e integridade do payload | `TRANSCRICAO` | `[09:22] Sofia · FDD §2` |
| `FDD-OBJ-06` | `docs/FDD.md` | Objetivo Técnico | Não alterar contrato nem latência de endpoint existente | `CODIGO` | `tests/orders.test.ts · FDD §2` |
| `FDD-OBJ-07` | `docs/FDD.md` | Objetivo Técnico | Absorver o módulo na infraestrutura compartilhada sem tocá-la | `CODIGO` | `src/middlewares/error.middleware.ts · FDD §2` |
| `FDD-PLAN-01` | `docs/FDD.md` | Plano | Ordem de merge em cinco entregas, derivada da decomposição da reunião | `TRANSCRICAO` | `[09:46] Larissa · [09:50] Larissa · FDD §15.1` |
| `FDD-PLAN-02` | `docs/FDD.md` | Plano | As cinco entregas vão para `main` atrás do gate; o release é ligar a variável | `CODIGO` | `src/config/env.ts:15-23 · FDD §15.2` |
| `FDD-PLAN-03` | `docs/FDD.md` | Plano | Revisão de segurança de dois dias úteis como porteiro antes de ligar a variável | `TRANSCRICAO` | `[09:46] Sofia · FDD §15.3` |
| `FDD-R-01` | `docs/FDD.md` | Risco Técnico | Defeito no publisher derruba a mudança de status, a operação central do OMS — Média/Crítico | `TRANSCRICAO` | `[09:41] Diego · FDD §14` |
| `FDD-R-02` | `docs/FDD.md` | Risco Técnico | A meta de latência não se cumpre para clientes lentos — Alta/Médio | `TRANSCRICAO` | `[09:02] Marcos · FDD §14` |
| `FDD-R-03` | `docs/FDD.md` | Risco Técnico | Qualquer operador autenticado obtém segredo de qualquer cliente — Alta/Alto | `TRANSCRICAO` | `[09:37] Sofia · FDD §14` |
| `FDD-R-04` | `docs/FDD.md` | Risco Técnico | Fila de mortos cresce sem ninguém perceber, por não haver alerta — Média/Médio | `TRANSCRICAO` | `[09:37] Larissa · FDD §14` |
| `FDD-R-05` | `docs/FDD.md` | Risco Técnico | Worker morto passa despercebido; eventos acumulam — Média/Alto | `TRANSCRICAO` | `[09:11] Diego · FDD §14` |
| `FDD-R-06` | `docs/FDD.md` | Risco Técnico | Consulta de endpoints dentro da transação degrada `changeStatus` — Baixa/Médio | `TRANSCRICAO` | `[09:04] Bruno · FDD §14` |
| `FDD-R-07` | `docs/FDD.md` | Risco Técnico | Cada implantação gera entregas duplicadas — Média/Baixo | `CODIGO` | `src/server.ts:13-21 · FDD §14` |
| `FDD-R-08` | `docs/FDD.md` | Risco Técnico | Segredo vaza em log de erro — Baixa/Alto | `TRANSCRICAO` | `[09:22] Diego · FDD §14` |
| `FDD-R-09` | `docs/FDD.md` | Risco Técnico | Cliente não implementa deduplicação e processa evento duas vezes — Média/Médio | `TRANSCRICAO` | `[09:25] Sofia · FDD §14` |
| `FDD-R-10` | `docs/FDD.md` | Risco Técnico | Rajada de mudanças inunda um cliente — Baixa/Médio | `TRANSCRICAO` | `[09:38] Diego · FDD §14` |
| `FDD-RES-01` | `docs/FDD.md` | Resiliência | Aritmética do lease: teto de 25 eventos por segundo e no máximo 6 eventos do mesmo pedido por lote | `TRANSCRICAO` | `[09:38] Diego · FDD §9.6` |
| `NA-02` | `docs/FDD.md` | Decisão não arquitetural | HTTPS obrigatório na URL — a própria Sofia desqualifica ao propor; vive na validação Zod | `TRANSCRICAO` | `[09:23] Sofia · FDD §7.11` |
| `NA-05` | `docs/FDD.md` | Decisão não arquitetural | Conjunto de cabeçalhos do envio — enumeração, não escolha entre opções | `TRANSCRICAO` | `[09:44] Diego · [09:44] Sofia · FDD §7.10` |
| `ADR-001` | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Decisão | Padrão outbox no MySQL para publicação de eventos de pedido | `TRANSCRICAO` | `[09:48] Larissa · [09:03] Larissa · ADR-001 §Decisão` |
| `ADR-001-ALT-01` | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Alternativa Descartada | Disparo HTTP síncrono dentro do `changeStatus` | `TRANSCRICAO` | `[09:04] Bruno · ADR-001 §Alternativas Consideradas` |
| `ADR-001-ALT-02` | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Alternativa Descartada | Redis Streams ou fila dedicada | `TRANSCRICAO` | `[09:07] Diego · ADR-001 §Alternativas Consideradas` |
| `ADR-002` | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão | Worker em processo separado consumindo a outbox por polling | `TRANSCRICAO` | `[09:48] Larissa · [09:06] Diego · ADR-002 §Decisão` |
| `ADR-002-ALT-01` | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Alternativa Descartada | Trigger de banco para notificar o worker | `TRANSCRICAO` | `[09:09] Diego · ADR-002 §Alternativas Consideradas` |
| `ADR-002-ALT-02` | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Alternativa Descartada | Worker embutido na mesma instância da API | `TRANSCRICAO` | `[09:11] Diego · ADR-002 §Alternativas Consideradas` |
| `ADR-003` | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Decisão | Retry com backoff de cinco retentativas e fila de mortos em tabela dedicada | `TRANSCRICAO` | `[09:48] Larissa · [09:14] Larissa · ADR-003 §Decisão` |
| `ADR-003-ALT-01` | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | Três tentativas | `TRANSCRICAO` | `[09:16] Diego · ADR-003 §Alternativas Consideradas` |
| `ADR-003-ALT-02` | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | Retry indefinido com backoff | `TRANSCRICAO` | `[09:15] Diego · ADR-003 §Alternativas Consideradas` |
| `ADR-003-ALT-03` | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | Marcar `failed` na própria outbox | `TRANSCRICAO` | `[09:18] Diego · ADR-003 §Alternativas Consideradas` |
| `ADR-004` | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Decisão | Autenticação de origem com HMAC-SHA256 e segredo único por endpoint | `TRANSCRICAO` | `[09:48] Larissa · [09:19] Sofia · ADR-004 §Decisão` |
| `ADR-004-ALT-01` | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Alternativa Descartada | Segredo global da plataforma | `TRANSCRICAO` | `[09:21] Sofia · ADR-004 §Alternativas Consideradas` |
| `ADR-004-ALT-02` | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Alternativa Descartada | Truncar o payload acima do limite | `TRANSCRICAO` | `[09:23] Sofia · ADR-004 §Alternativas Consideradas` |
| `ADR-005` | `docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md` | Decisão | Entrega ao menos uma vez, com deduplicação delegada ao cliente via `X-Event-Id` | `TRANSCRICAO` | `[09:48] Larissa · [09:25] Diego · ADR-005 §Decisão` |
| `ADR-005-ALT-01` | `docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md` | Alternativa Descartada | Exactly-once | `TRANSCRICAO` | `[09:25] Diego · ADR-005 §Alternativas Consideradas` |
| `ADR-006` | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Reuso dos padrões existentes do projeto no módulo de webhooks | `TRANSCRICAO` | `[09:48] Larissa · [09:27] Bruno · ADR-006 §Decisão` |
| `ADR-006-ALT-01` | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | Injetar o repository de webhook inteiro no `OrderService` | `TRANSCRICAO` | `[09:41] Diego · ADR-006 §Alternativas Consideradas` |
| `ADR-007` | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Decisão | Payload materializado como snapshot na inserção e filtragem de destinatários na origem | `TRANSCRICAO` | `[09:52] Bruno · [09:51] Bruno · ADR-007 §Decisão` |
| `ADR-007-ALT-01` | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Alternativa Descartada | Renderizar o payload na hora do envio | `TRANSCRICAO` | `[09:52] Larissa · ADR-007 §Alternativas Consideradas` |
| `ADR-007-ALT-02` | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Alternativa Descartada | Filtrar os destinatários na hora de mandar | `TRANSCRICAO` | `[09:34] Bruno · ADR-007 §Alternativas Consideradas` |
| `ADR-008` | `docs/adrs/ADR-008-controle-de-acesso-dos-endpoints.md` | Decisão | Replay de fila de mortos restrito a `ADMIN` e CRUD de configuração apenas autenticado | `TRANSCRICAO` | `[09:48] Larissa · [09:35] Diego · ADR-008 §Decisão` |
| `ADR-008-ALT-01` | `docs/adrs/ADR-008-controle-de-acesso-dos-endpoints.md` | Alternativa Descartada | Exigir `ADMIN` também no CRUD de configuração | `TRANSCRICAO` | `[09:37] Sofia · ADR-008 §Alternativas Consideradas` |
| `ADR-008-ALT-02` | `docs/adrs/ADR-008-controle-de-acesso-dos-endpoints.md` | Alternativa Descartada | Derivar o cliente do token em vez de recebê-lo na requisição | `TRANSCRICAO` | `[09:32] Bruno · ADR-008 §Alternativas Consideradas` |
| `NA-03` | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Decisão não arquitetural | Timeout de 10 s por tentativa — parâmetro da política de entrega, decidido em uma linha | `TRANSCRICAO` | `[09:42] Diego · ADR-003 §Decisão` |
| `NA-04` | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão não arquitetural | Intervalo de polling de 2 s — parâmetro do modo de consumo, dimensionado contra o requisito de `[09:02]` | `TRANSCRICAO` | `[09:09] Diego · ADR-002 §Decisão` |
| `NA-06` | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão não arquitetural | UUID como identificador — segue o padrão já estabelecido no projeto | `TRANSCRICAO` | `[09:51] Larissa · ADR-006 §Decisão` |

### As seis que a reunião desqualificou

As linhas `NA-01` a `NA-06` vinham de um índice de ADRs que não existe mais. São itens técnicos que alguém
na call classificou, na hora, como não sendo decisão arquitetural — e a subtração vale registro tanto quanto
a soma. Larissa em `[09:24]`: *"Anotado, mas não vejo como decisão arquitetural separada, é só requisito não
funcional"*. Sofia em `[09:23]`, desqualificando a própria proposta: *"Isso na verdade nem é decisão
arquitetural, é só uma validação no schema Zod"*. Larissa de novo em `[09:51]`: *"UUID, segue o padrão do
resto do projeto. Tudo é uuid."* Os outros três — timeout, intervalo de polling e conjunto de cabeçalhos —
foram fechados sem alternativa em disputa, e por isso viraram parâmetro dentro de
[ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md),
[ADR-002](adrs/ADR-002-worker-separado-em-polling.md) e do contrato de saída do FDD.

## 2. Cobertura por documento

| Documento | Itens (seção 1) | `TRANSCRICAO` | `CODIGO` | Derivados (seção 3) |
|---|---|---|---|---|
| `docs/PRD.md` | 98 | 96 | 2 | 1 |
| `docs/RFC.md` | 18 | 18 | 0 | 1 |
| `docs/FDD.md` | 134 | 76 | 58 | 42 |
| `docs/adrs/` (8 arquivos) | 26 | 26 | 0 | 7 |
| **Total** | **276** | **216** | **60** | **51** |

> **Fórmula da cobertura.** Itens rastreados dividido por itens identificáveis nos documentos, onde
> *identificável* é todo item que carrega identificador próprio ou ocupa uma linha de tabela numerada: `RF`,
> `RNF`, `OBJ`, `FE`, `R`, `CA`, `TO`, `DEP`, `PE`, `PR`, `EV`, `ACE`, `ENT`, `CEN`, `ALT`, `Q`, `IMP`, `CT`,
> `HDR`, `ERR`, `ENV`, `AC`, `MOD`, `INT`, `CONV`, `RES`, `PLAN`, `NA` e os oito `ADR`. Prosa explicativa não
> conta. Publicar o percentual sem a fórmula ao lado torna o número inauditável.

Com os blocos desta rodada, todo item identificável dos quatro documentos tem linha: 276 na tabela acima e 51
na seção 3, 327 no total. Os 51 não são omissão — são itens que a reunião não decidiu, e que ficariam
falsificados se aparecessem ao lado dos outros com uma âncora de fita.

## 3. Decisões sem âncora na reunião

Cinquenta e uma decisões de implementação que a reunião não tomou. Nenhuma pode ser apresentada como fala de
participante. Cada uma carrega procedência: `MERCADO` para padrão de indústria verificado, `CODIGO` para
derivação de padrão já existente no repositório, `PRODUTO` para escolha de quem produziu a documentação.

| ID | Decisão | Procedência | Base |
|---|---|---|---|
| `H01` | Valor de X-Signature — v1=<hex>; múltiplas assinaturas separadas por vírgula durante a rotação | `MERCADO` | MERCADO Stripe / GitHub |
| `H02` | String canônica assinada — Somente o corpo cru. Literal a [09:22] Sofia ("HMAC-SHA256 sobre o corpo do request") | `PRODUTO` | [09:22] — assinar {timestamp}.{corpo}, ao padrão Stripe, foi cogitado e descartado por extrapolar a fala |
| `H03` | Formato de X-Timestamp — Unix epoch em segundos (payload segue ISO 8601 conforme [09:43]) | `MERCADO` | MERCADO Stripe / Slack |
| `H04` | Tolerância de recência — 5 minutos, recomendada ao cliente na doc; não validamos, somos o emissor | `MERCADO` | MERCADO Stripe + limitação derivada de H02 |
| `H05` | Geração do secret — 32 bytes de crypto.randomBytes em hex, prefixo whsec_ | `MERCADO` | MERCADO Stripe + CODIGO (sem dependência nova, node:crypto) |
| `H06` | Secret at-rest — Texto claro + redact no Pino. Criptografia não decidida → Q06, dona Sofia | `PRODUTO` | PRODUTO |
| `H07` | Exposição do secret — Íntegro só em POST /webhooks e POST /webhooks/:id/rotate-secret; mascarado no resto | `MERCADO` | MERCADO + [09:31] |
| `H08` | Tamanho do batch — 50, via env com default | `PRODUTO` | PRODUTO — [09:08] diz só "batch pequeno" |
| `H09` | Claim de linhas — SELECT … FOR UPDATE SKIP LOCKED | `MERCADO` | MERCADO + CODIGO (MySQL 8.0 no docker-compose.yml) |
| `H10` | Lease de evento travado — processingStartedAt + reclaim após 60s (6× o timeout de 10s) | `PRODUTO` | PRODUTO — buraco que a reunião não enxergou |
| `H11` | Classe de falha — Toda resposta não-2xx e toda falha de rede consomem as 5 tentativas. Sem atalho para 4xx | `PRODUTO` | [09:15] [09:17] — literal, sem extrapolação |
| `H12` | Jitter no backoff — Não. Curva exata de [09:17]. Jitter fica como melhoria futura | `PRODUTO` | [09:17] |
| `H13` | Paralelismo do batch — Serial dentro de cada order_id, paralelo entre order_ids distintos, concorrência 10 | `PRODUTO` | [09:12] (ordering) + [09:04] (cliente lento não trava os outros) |
| `H14` | Redirects 3xx — Não seguir; conta como falha | `MERCADO` | MERCADO Stripe |
| `H15` | Fan-out — Uma mudança de status com N endpoints assinantes gera N linhas na outbox, com N eventId distintos | `PRODUTO` | PRODUTO — coerente com X-Webhook-Id de [09:44] |
| `H16` | subscribedStatuses — Coluna Json | `CODIGO` | CODIGO — customers.address já é Json (prisma/schema.prisma:46) |
| `H17` | Retenção de deliveries — Guarda todas as tentativas; o endpoint devolve as 100 mais recentes | `PRODUTO` | [09:34] + PRODUTO |
| `H18` | Secret antigo na rotação — Colunas na própria linha; a janela de 24h garante no máximo 2 secrets vivos | `PRODUTO` | [09:21] + PRODUTO |
| `H19` | Padrão de PK e mapeamento — String @id @default(uuid()) @db.Char(36), @@map("snake_case"), colunas camelCase | `CODIGO` | CODIGO prisma/schema.prisma + [09:51] |
| `H20` | Desativar sem apagar — PATCH { active: false }; DELETE é hard delete | `CODIGO` | [09:21] ("estado ativo") + CODIGO |
| `H21` | Envelope de listagem — { data, pagination }, pageSize default 20, teto 100 | `CODIGO` | CODIGO src/shared/http/response.ts:8-11 |
| `H22` | Envelope de erro — { error: { code, message, details? } } — nada a mudar no middleware | `CODIGO` | CODIGO src/middlewares/error.middleware.ts:15-24 |
| `H23` | Constantes congeladas — POLL_INTERVAL_MS, HTTP_TIMEOUT_MS, MAX_RETRIES e BACKOFF_MS em código, não em env | `PRODUTO` | PRODUTO — mudar exige PR e reabrir o ADR |
| `H24` | Env de tuning — WEBHOOK_BATCH_SIZE, WEBHOOK_CONCURRENCY, WEBHOOK_LEASE_TIMEOUT_MS, WEBHOOK_WORKER_ENABLED | `CODIGO` | CODIGO src/config/env.ts:4-27 |
| `H25` | Observabilidade sem vendor — Eventos Pino estruturados, um por etapa do ciclo de entrega | `CODIGO` | CODIGO src/shared/logger/index.ts |
| `H26` | Tracing — Propagar o requestId da requisição de origem até a entrega, via coluna na outbox | `CODIGO` | CODIGO src/middlewares/request-logger.middleware.ts:6-8 |
| `H27` | Redação de segredos no log — Adicionar *.secret, *.previousSecret, *.signature ao redact | `CODIGO` | CODIGO src/shared/logger/index.ts:4-11 (hoje não cobre) |
| `H28` | Cliente HTTP — fetch nativo do Node 20 com AbortSignal.timeout. Nenhuma dependência nova | `CODIGO` | CODIGO package.json:25-34 (sem axios/undici) + [09:42] |
| `H29` | Drenagem no shutdown — Worker precisa de hook de drenagem; o encerramento atual não aguarda server.close() | `CODIGO` | CODIGO src/server.ts:13-21 |
| `H30` | Limpeza nos testes — As 4 tabelas novas entram no beforeEach, respeitando ordem de FK | `CODIGO` | CODIGO tests/setup.ts:9-15 |
| `H31` | Transação de claim declara READ COMMITTED, para o lock de faixa não alcançar o índice em que o publisher insere | `CODIGO` | FDD 5.6 — publisher e claim escrevem e varrem a mesma tabela |
| `H32` | DELETE de endpoint recusado com 409 enquanto houver evento pendente; P2003 capturado no service | `CODIGO` | FDD 6.7 — src/middlewares/error.middleware.ts:37-54 só trata P2002 e P2025 |
| `H33` | webhook_outbox.orderId com onDelete: Cascade, única divergência do padrão RESTRICT | `CODIGO` | FDD 6.3 e 6.7 — para não regredir DELETE /orders/:id |
| `H34` | subscribedStatuses validado como array de string no Zod e convertido no service | `PRODUTO` | FDD 7.11 — nativeEnum responderia 400 e mataria dois códigos da seção 8 |
| `H35` | replayed como z.enum(['true','false']).transform, não z.coerce.boolean() | `CODIGO` | FDD 7.11 — Boolean('false') é true |
| `H36` | requestId como quarto parâmetro opcional e posicional de changeStatus | `CODIGO` | FDD 11.1 — chamador único; req.id é string ou undefined |
| `H37` | Teto de 6 eventos do mesmo pedido por lote (lease dividido pelo timeout) | `PRODUTO` | FDD 9.6 — derivação de 9.1, não escrita em lugar nenhum do código |
| `H38` | HMAC na quarta entrega, contra a ordem literal de [09:46] Larissa, que o colocava por último | `PRODUTO` | FDD 15.1 — AC-14, AC-15 e AC-16 exigem entrega real para se verificar |
| `PRD-FE-07` | Desativação automática de endereço com falha recorrente, fora desta fase | `PRODUTO` | O próprio PRD marca o item como sem decisão da reunião; a fala de [09:37] Larissa trata só de e-mail |
| `RFC-Q07` | Escopo da assinatura: o cabeçalho de horário fica fora do trecho assinado e é adulterável | `PRODUTO` | O RFC rotula a questão como derivada da análise dele; [09:22] e [09:44] são contexto, não origem |
| `FDD-CT-08` | GET /api/v1/admin/webhooks/dead-letter — listar fila de mortos | `PRODUTO` | A reunião decidiu o replay em [09:35], nunca a listagem. O endpoint é consequência do runbook de 16.1 |
| `FDD-FE-07` | Desativação automática de endpoint com falha recorrente | `PRODUTO` | FDD 3 — nunca discutido; consequência de o alerta estar fora |
| `FDD-FE-08` | Circuit breaker por endpoint de destino | `PRODUTO` | FDD 3 — nunca discutido; a resiliência desenhada é timeout, retry e fila de mortos |
| `FDD-FE-09` | Teste de carga e meta de vazão | `PRODUTO` | FDD 3 — sem volume esperado na reunião, não há alvo contra o qual medir |
| `ADR-002-ALT-03` | Vários workers em paralelo desde o início | `PRODUTO` | Análise do ADR-002, contra [09:12] Diego e [09:13] Larissa |
| `ADR-004-ALT-03` | Assinar timestamp.corpo, ao padrão Stripe | `MERCADO` | MERCADO Stripe; barrado pela literalidade de [09:22] Sofia e remetido a Q07 |
| `ADR-005-ALT-02` | Gerar o X-Event-Id no momento do envio | `PRODUTO` | Análise do ADR-005, contra a restrição de [09:25] e a curva de retry de [09:17] |
| `ADR-005-ALT-03` | (order_id, to_status) como chave natural de idempotência | `PRODUTO` | Análise do ADR-005, contra [09:44] Sofia e o replay de fila de mortos de [09:35] |
| `ADR-006-ALT-02` | Desenho próprio, fora do formato de cinco arquivos | `CODIGO` | src/app.ts:26-53 e src/routes/index.ts:21-31; refutada com [09:27] Bruno, [09:30] Larissa e [09:07] Diego |
| `ADR-007-ALT-03` | Guardar o payload comprimido | `PRODUTO` | Análise do ADR-007, contra o teto de [09:24] Larissa e a exclusão dos itens de [09:43] Diego |
| `ADR-008-ALT-03` | Criar uma role de cliente | `CODIGO` | prisma/schema.prisma:11-14 — o enum tem só ADMIN e OPERATOR; [09:30] Larissa fecha em reuso |

## 4. Índice reverso — o que da reunião entrou no pacote

A tabela acima responde *de onde veio cada item*. Esta responde a pergunta inversa, que é a que expõe
omissão: **o que da fita ficou de fora**. São 54 timestamps distintos e 151 falas.

| Timestamp | Coberto por | Se não, por quê |
|---|---|---|
| `[09:00]` | `PRD-OBJ-04`, `PRD-OBJ-06`, `PRD-RF-09` | — |
| `[09:01]` | — | Bruno pergunta se é tempo real mesmo — a resposta em [09:02] é que vira requisito |
| `[09:02]` | `PRD-FE-03`, `PRD-OBJ-01`, `PRD-R-02`, `PRD-DEP-05`, `FDD-FE-03`, `FDD-R-02` … +1 | — |
| `[09:03]` | `ADR-001` | — |
| `[09:04]` | `PRD-OBJ-03`, `PRD-RNF-01`, `PRD-RNF-12`, `PRD-TO-01`, `RFC-ALT-01`, `FDD-AC-20` … +4 | — |
| `[09:05]` | — | Diego entra na call; coordenação |
| `[09:06]` | `PRD-CA-01`, `PRD-OBJ-02`, `PRD-RNF-02`, `PRD-TO-01`, `FDD-AC-01`, `ADR-002` | — |
| `[09:07]` | `PRD-RNF-10`, `PRD-PR-01`, `PRD-DEP-02`, `PRD-TO-05`, `RFC-ALT-02`, `RFC-IMP-01` … +2 | — |
| `[09:08]` | `PRD-FE-04`, `RFC-Q05`, `FDD-FE-04` | — |
| `[09:09]` | `RFC-ALT-03`, `FDD-AC-07`, `FDD-OBJ-03`, `ADR-002-ALT-01`, `NA-04` | — |
| `[09:10]` | `PRD-ACE-01` | — |
| `[09:11]` | `FDD-R-05`, `RFC-IMP-01`, `ADR-002-ALT-02` | — |
| `[09:12]` | `PRD-CA-11`, `PRD-RNF-09`, `PRD-EV-03`, `PRD-TO-05`, `FDD-AC-19` | — |
| `[09:13]` | `PRD-FE-06`, `RFC-Q02`, `FDD-FE-06` | — |
| `[09:14]` | `ADR-003` | — |
| `[09:15]` | `PRD-RF-11`, `PRD-RNF-04`, `FDD-ERR-08`, `FDD-ERR-12`, `FDD-OBJ-04`, `ADR-003-ALT-02` | — |
| `[09:16]` | `PRD-CA-06`, `PRD-CEN-02`, `RFC-ALT-05`, `ADR-003-ALT-01` | — |
| `[09:17]` | `PRD-RNF-04`, `PRD-ACE-02`, `PRD-TO-03`, `RFC-ALT-04`, `RFC-IMP-04`, `FDD-AC-08` | — |
| `[09:18]` | `PRD-CA-07`, `FDD-AC-09`, `ADR-003-ALT-03` | — |
| `[09:19]` | `ADR-004` | — |
| `[09:20]` | `PRD-CA-04`, `PRD-RF-10`, `FDD-AC-14`, `FDD-HDR-04` | — |
| `[09:21]` | `PRD-CA-09`, `PRD-CA-14`, `PRD-CEN-04`, `PRD-RF-07`, `PRD-RF-08`, `PRD-RNF-07` … +6 | — |
| `[09:22]` | `PRD-R-07`, `FDD-AC-16`, `FDD-R-08`, `FDD-OBJ-05` | — |
| `[09:23]` | `PRD-CA-05`, `PRD-CA-16`, `PRD-DEP-03`, `PRD-RNF-05`, `PRD-RNF-06`, `FDD-AC-18` … +3 | — |
| `[09:24]` | `PRD-RNF-03`, `PRD-CA-16`, `FDD-AC-28`, `FDD-ERR-14`, `NA-01` | — |
| `[09:25]` | `PRD-CA-17`, `PRD-TO-02`, `RFC-ALT-06`, `FDD-AC-05`, `FDD-AC-22`, `FDD-HDR-01` … +3 | — |
| `[09:26]` | `PRD-R-04`, `PRD-DEP-04`, `PRD-ENT-01`, `PRD-TO-02` | — |
| `[09:27]` | `ADR-006` | — |
| `[09:28]` | `FDD-ERR-01`, `FDD-ERR-15`, `FDD-ERR-16` | — |
| `[09:29]` | `FDD-AC-29` | — |
| `[09:30]` | `PRD-CA-13` | — |
| `[09:31]` | `PRD-CA-03`, `PRD-RF-01`, `PRD-RF-02`, `FDD-AC-17`, `FDD-CT-01` | — |
| `[09:32]` | `PRD-PR-03`, `ADR-008-ALT-02` | — |
| `[09:33]` | `PRD-RF-03`, `PRD-RF-04`, `PRD-RF-05`, `PRD-RF-06`, `PRD-CEN-01`, `PRD-EV-03` … +7 | — |
| `[09:34]` | `PRD-CA-02`, `PRD-CA-08`, `PRD-CEN-03`, `PRD-CEN-05`, `PRD-R-06`, `PRD-RF-12` … +4 | — |
| `[09:35]` | `PRD-RF-13`, `FDD-CT-09`, `FDD-ERR-06`, `FDD-ERR-07`, `ADR-008` | — |
| `[09:36]` | `PRD-CA-10`, `PRD-RF-14`, `PRD-RNF-11`, `FDD-AC-21`, `FDD-AC-24` | — |
| `[09:37]` | `PRD-FE-01`, `PRD-R-03`, `PRD-R-05`, `PRD-TO-06`, `RFC-Q04`, `RFC-IMP-04` … +4 | — |
| `[09:38]` | `PRD-R-08`, `RFC-Q01`, `FDD-R-10`, `FDD-RES-01` | — |
| `[09:39]` | `PRD-FE-05`, `RFC-Q01`, `FDD-FE-05` | — |
| `[09:40]` | `PRD-CA-12`, `PRD-FE-02`, `PRD-R-01`, `PRD-DEP-04`, `PRD-ENT-02`, `RFC-IMP-02` … +3 | — |
| `[09:41]` | `FDD-R-01`, `RFC-IMP-02`, `ADR-006-ALT-01` | — |
| `[09:42]` | `PRD-CA-15`, `PRD-RNF-08`, `RFC-IMP-03`, `FDD-AC-10`, `FDD-ERR-09`, `NA-03` | — |
| `[09:43]` | `PRD-TO-04`, `FDD-CT-10`, `FDD-FE-10` | — |
| `[09:44]` | `PRD-CA-18`, `PRD-CEN-05`, `FDD-CT-10`, `FDD-HDR-02`, `FDD-HDR-03`, `FDD-HDR-05` … +1 | — |
| `[09:45]` | `PRD-PR-02` | — |
| `[09:46]` | `PRD-OBJ-05`, `PRD-DEP-01`, `PRD-PE-01`, `PRD-PE-02`, `PRD-PE-03`, `PRD-PE-04` … +4 | — |
| `[09:47]` | `PRD-PE-05`, `PRD-ENT-03` | — |
| `[09:48]` | `ADR-001`, `ADR-002`, `ADR-003`, `ADR-004`, `ADR-005`, `ADR-006` … +2 | — |
| `[09:49]` | `PRD-ACE-03`, `PRD-ENT-03` | — |
| `[09:50]` | `FDD-PLAN-01` | — |
| `[09:51]` | `RFC-ALT-08`, `ADR-007`, `NA-06` | — |
| `[09:52]` | `FDD-AC-06`, `ADR-007`, `ADR-007-ALT-01` | — |
| `[09:53]` | — | Encerramento da call |

**51 de 54 timestamps** (94%) têm pelo menos um item do pacote apontando para eles. Os três restantes são a
pergunta de Bruno que a resposta seguinte já absorve, a entrada do Diego na call e a despedida — listados
nominalmente acima, para que a ausência seja verificável em vez de assumida.

---

**Manutenção.** Este tracker é gerado a partir das tabelas dos próprios documentos, que carregam a coluna de
origem. Quando um documento muda, o tracker precisa ser regenerado — e a divergência entre os dois é
detectável comparando os identificadores.
