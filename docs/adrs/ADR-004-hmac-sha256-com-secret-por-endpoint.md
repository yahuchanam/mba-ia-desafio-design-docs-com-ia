# ADR-004 — Autenticação de origem com HMAC-SHA256 e secret única por endpoint

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status**       | Aceito                                                                          |
| **Data**         | 2026-08-08                                                                      |
| **Decisores**    | Larissa (Tech Lead) · Diego (Eng. Plataforma) · Bruno (Eng. Pleno) · Sofia (Eng. Segurança) · Marcos (PM) |
| **Confirmado**   | `[09:48] Larissa` (resumo) · `[09:49]` Diego, Bruno e Sofia confirmam           |
| **Relacionados** | [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md) · [ADR-008](ADR-008-controle-de-acesso-dos-endpoints.md) |

## Status

Aceito, confirmado no resumo de `[09:48] Larissa` e ratificado em `[09:49]` por Diego, Bruno e Sofia.

## Contexto

O sistema passa a enviar dados de pedidos para uma URL que não é nossa e que não controlamos.
Sofia abriu o bloco de segurança colocando o problema no lugar certo: *"O cliente tem que conseguir
validar que a requisição veio realmente da gente, e que ninguém adulterou o payload no meio."*
(`[09:19] Sofia`). Sem isso, qualquer um que descubra a URL cadastrada consegue injetar um evento
falso de mudança de status no ERP do cliente — e o cliente não tem como distinguir.

O bloco fechou cinco perguntas encadeadas:

1. **Que prova de origem e integridade vamos mandar?** — resolvida em `[09:20]` e `[09:22]`.
2. **Uma secret da plataforma ou uma por endpoint?** — resolvida em `[09:21]`, e é a pergunta que
   define o raio de dano de um vazamento.
3. **Como trocar a secret sem derrubar a integração do cliente?** — resolvida em `[09:21]`.
4. **Quem gera a secret e quando ela aparece?** — resolvida em `[09:31] Marcos`.
5. **O transporte precisa ser TLS?** — resolvida em `[09:23]`.

A pergunta 2 condiciona as demais. Com secret por endpoint a rotação fica barata: afeta um cliente
só, o segredo antigo convive com o novo em colunas da própria linha de configuração e não existe
evento de "trocar a chave da plataforma inteira". Diego trouxe o precedente que sustenta a pergunta
3: *"A gente já teve cliente que vazou secret em log de aplicação dele uma vez."* (`[09:22] Diego`).

Duas restrições do repositório pesaram. Não há dependência de criptografia nem cliente HTTP em
`package.json:25-34`, então HMAC e geração de secret saem de `node:crypto`. E o `redact` do Pino em
`src/shared/logger/index.ts:4-11` cobre apenas `authorization`, `cookie`, `*.password`,
`*.passwordHash`, `*.token` e `*.accessToken` — nada de `secret` ou `signature`. Somado a
`src/middlewares/error.middleware.ts:57`, que loga o objeto de erro inteiro
(`logger.error({ err, requestId, ... })`), hoje um erro carregando a configuração do webhook
derrama a secret no nosso próprio log. O mesmo acidente que Diego descreveu do lado do cliente.

## Decisão

Toda entrega sai assinada com HMAC-SHA256 sobre o corpo do request, usando uma secret exclusiva
daquele endpoint cadastrado.

| Item | Valor | Origem |
| --- | --- | --- |
| Algoritmo | HMAC-SHA256 | `F18` — `[09:20]` `[09:22]` |
| String assinada | somente o corpo cru enviado, byte a byte | `F18` + `H02` (derivada) |
| Header da assinatura | `X-Signature: v1=<hex>` | `F26` + `H01` (derivada) |
| Escopo da secret | uma por endpoint cadastrado, nunca global | `F19` — `[09:21]` |
| Geração | 32 bytes de `crypto.randomBytes`, hex, prefixo `whsec_` | `H05` (derivada) |
| Origem e entrega ao cliente | gerada por nós, devolvida íntegra na criação; mascarada nos `GET` | `F21` — `[09:31]` + `H07` |
| Rotação | `POST /api/v1/webhooks/:id/rotate-secret`; secret antiga válida em paralelo por **24h**, depois morre | `F20` — `[09:21]` |
| Guarda da secret antiga | colunas na própria linha de configuração — no máximo 2 secrets vivas | `H18` (derivada) |
| Armazenamento | texto claro no MySQL, com `redact` no logger | `H06` (derivada) → `Q06` |
| Transporte | `https` obrigatório; `http` recusado na validação do schema Zod | `F22` — `[09:23]` |
| Corpo acima do limite | **64KB**; ultrapassou, erra — não trunca | `F23` — `[09:23]` `[09:24]` |
| `X-Timestamp` | Unix epoch em segundos | `H03` (derivada) |
| Tolerância de recência | 5 minutos, **recomendada ao cliente na documentação** — não validamos, somos o emissor | `H04` (derivada) |

Formato do valor assinado, incluindo a janela de rotação:

```text
assinatura = HMAC_SHA256(chave = secret_do_endpoint, mensagem = corpo_cru)
X-Signature: v1=<hex(assinatura)>

# durante as 24h de grace period, as duas convivem no mesmo header:
X-Signature: v1=<hex(com_secret_nova)>,v1=<hex(com_secret_antiga)>
```

Headers de cada envio (`F26`, `F27`), definidos por Diego em `[09:44]` com a adição de Sofia:

| Header | Conteúdo |
| --- | --- |
| `X-Event-Id` | UUID do evento — dedup do lado do cliente, ver [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md) |
| `X-Signature` | `v1=<hex>`, conforme acima |
| `X-Timestamp` | momento do envio |
| `X-Webhook-Id` | id do endpoint cadastrado |
| `Content-Type` | `application/json` |

Quatro itens da tabela são **decisões derivadas — a reunião não tratou estes pontos**: o formato
`v1=<hex>` com assinaturas separadas por vírgula (`H01`, procedência: padrão Stripe e GitHub); a
escolha de assinar **somente** o corpo cru (`H02`, procedência: leitura literal de `[09:22] Sofia`);
o tamanho e o prefixo da secret (`H05`, procedência: padrão de mercado sem dependência nova); e o
armazenamento em texto claro (`H06`, procedência: escolha de quem produziu a documentação, com a
questão aberta declarada abaixo).

### O que este ADR não cobre

- **Idempotência, `X-Event-Id` e a garantia at-least-once** → [ADR-005](ADR-005-entrega-at-least-once-com-event-id.md). Aqui o header aparece só como
  dependência da defesa contra replay.
- **Autorização dos endpoints de configuração, de rotação e do replay de DLQ** → [ADR-008](ADR-008-controle-de-acesso-dos-endpoints.md).
- **Colunas, contrato HTTP e códigos de erro do módulo** → FDD. ADR registra a decisão, não o contrato.

## Alternativas Consideradas

### Secret global da plataforma

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | — (levantada por Sofia como contraste) |
| **Quem derrubou**   | Sofia                              |
| **Citação**         | > "cada endpoint de webhook do cliente tem que ter uma secret única. Não é uma secret global da nossa plataforma. Senão se vaza uma, vaza tudo." — `[09:21] Sofia` |
| **Trade-off aceito**| Passamos a gerenciar N segredos em vez de um: N rotações, N linhas com material sensível, nenhuma chave única para revogar tudo de uma vez |

Uma secret única simplificaria a operação — um valor para gerar, guardar e rotacionar. O custo é o
raio de dano: o vazamento de uma integração qualquer permitiria forjar eventos para todos os
clientes, e a resposta ao incidente seria rotacionar a plataforma inteira, quebrando quem não tem
nada a ver com o incidente. O precedente de `[09:22] Diego` mostra que o vazamento tende a
acontecer do lado do cliente, fora do nosso controle, o que torna o isolamento por endpoint a única
contenção viável.

### Truncar o payload acima do limite

|                     |                                    |
| ------------------- | ---------------------------------- |
| **Proponente**      | — (Sofia levantou as duas opções)  |
| **Quem derrubou**   | Sofia, com Diego e Larissa fechando o valor |
| **Citação**         | > "Trunca? Erra? Eu sou a favor de erra. Se chegou nesse tamanho, tem algo errado." — `[09:23] Sofia` |
| **Trade-off aceito**| O evento gigante não chega ao cliente de forma alguma: vira falha, consome retry e termina na DLQ |

Truncar mantém alguma entrega, mas produz um corpo que não corresponde ao evento e cuja assinatura
é válida — o cliente verificaria o HMAC com sucesso sobre um dado mutilado, o pior dos dois mundos.
Diego calibrou o teto (*"Acho que 64KB já é um teto generoso. Nenhum evento nosso vai chegar perto
disso."*, `[09:24] Diego`) e Larissa fechou (*"64KB de limite, erro caso ultrapasse."*,
`[09:24] Larissa`), classificando o item como requisito não funcional e não como decisão separada.

### Assinar `timestamp.corpo`, ao padrão Stripe (análise deste documento — não levantada na reunião)

Incluir o `X-Timestamp` na string assinada tornaria o header inviolável e daria ao cliente uma
verificação de recência com valor real. A reunião não discutiu isso, e a restrição que impede
adotá-lo aqui veio dela: `[09:22] Sofia` fecha, literalmente, *"HMAC-SHA256 sobre o corpo do
request"*. Mudar o escopo da string assinada é matéria da revisão que Sofia reservou em `[09:46]`,
não de quem escreve o documento. Fica registrado como `Q07`.

## Consequências

### Positivas

- O cliente valida origem e integridade com uma primitiva que ele já tem em qualquer linguagem —
  *"HMAC-SHA256 é o padrão de mercado, todo cliente sério tem biblioteca pra isso."* (`[09:20] Sofia`).
- Vazamento fica contido em um endpoint. A resposta ao incidente é rotacionar aquele cadastro.
- Rotação sem janela de indisponibilidade: durante 24h o request carrega as duas assinaturas e o
  cliente migra quando conseguir, exatamente como Sofia desenhou em `[09:21]`.
- Nenhuma dependência nova. `node:crypto` cobre HMAC e geração de secret, coerente com
  `package.json:25-34`.
- O `http` é recusado na criação do endpoint, não na hora do envio: a validação Zod segue o padrão
  de `src/modules/customers/customer.schemas.ts:15` e `src/config/env.ts:4-27`, então nenhuma URL
  insegura chega a virar linha na outbox.

### Negativas

- A secret fica em texto claro no banco. Quem lê o MySQL lê todas as secrets, e o backup carrega o
  mesmo material. Consciente, não resolvido (`Q06`).
- O `redact` do Pino em `src/shared/logger/index.ts:4-11` não cobre `secret` nem `signature`, e
  `src/middlewares/error.middleware.ts:57` loga o objeto de erro inteiro. Enquanto os dois pontos
  não forem corrigidos, o acidente descrito em `[09:22] Diego` pode acontecer do nosso lado.
- O `X-Timestamp` foi pedido *"pra cliente conseguir detectar replay attack se quiser"*
  (`[09:44] Diego`), mas fica fora da assinatura: quem capturar um request pode reenviá-lo com o
  timestamp reescrito sem invalidar o HMAC. A detecção de replay recai inteiramente sobre a dedup
  por `X-Event-Id`, que é responsabilidade do cliente ([ADR-005](ADR-005-entrega-at-least-once-com-event-id.md)). É `Q07`.
- Rotação é ação do cliente. Se ele nunca chamar o endpoint, uma secret comprometida continua válida
  indefinidamente — não há expiração automática, e desativar endpoint sozinho está fora de escopo.
- Duas assinaturas no mesmo header quebram cliente que compara string exata em vez de percorrer a
  lista separada por vírgula. Custo que cai na documentação do portal.
- Durante as 24h de grace period existem duas secrets válidas: a superfície dobra na janela.
- Evento acima de 64KB simplesmente não é entregue, por decisão consciente de `[09:23]`.

### Limitações conhecidas

| Limitação | Gatilho de reabertura |
| --------- | --------------------- |
| Secret em texto claro no banco (`Q06`, dono Sofia) | Revisão de segurança de `[09:46]`, antes do deploy; ou qualquer suspeita de acesso indevido ao MySQL |
| `X-Timestamp` fora da string assinada, logo adulterável (`Q07`, dono Sofia) | Mesma revisão de `[09:46]`; ou o primeiro replay relatado que a dedup por `X-Event-Id` não tenha contido |
| `redact` sem `secret`/`signature` e erro logado inteiro | Bloqueante: entra como item obrigatório da revisão de `[09:46]`, junto de `src/middlewares/error.middleware.ts:57` |
| Secret sem expiração automática, rotação só sob demanda do cliente | Vazamento confirmado de um endpoint, ou pedido explícito do cliente |
| Versão de assinatura congelada em `v1` | Quando SHA-256 deixar de ser aceitável: o prefixo permite publicar `v2` em paralelo e migrar sem quebrar quem já integrou |
| Não validamos recência no emissor; os 5 minutos são recomendação na doc | Cliente relatar que a recomendação não é suficiente, ou `Q07` ser resolvida com o timestamp assinado |

A revisão de `[09:46]` é o portão desta decisão, e Sofia reforçou na despedida: *"Ok, só não
esqueçam de me agendar pra revisão de segurança antes de subir."* (`[09:49] Sofia`).

## Referências

- Transcrição: `[09:19] Sofia`, `[09:20] Sofia`, `[09:21] Sofia`, `[09:22] Diego`, `[09:22] Sofia`,
  `[09:23] Sofia`, `[09:24] Diego`, `[09:24] Larissa`, `[09:31] Marcos`, `[09:44] Diego`,
  `[09:44] Sofia`, `[09:46] Sofia`, `[09:48] Larissa`, `[09:49] Sofia`
- Código: `src/shared/logger/index.ts:4-11`, `src/middlewares/error.middleware.ts:57`,
  `src/config/env.ts:4-27`, `src/modules/customers/customer.schemas.ts:15`, `package.json:25-34`,
  `src/app.ts:67`
- Contrato de fatos: `F18` `F19` `F20` `F21` `F22` `F23` `F26` `F27` · `A08` `A09` ·
  `H01` `H02` `H03` `H04` `H05` `H06` `H07` `H18` · `Q06` `Q07` · `C12`
