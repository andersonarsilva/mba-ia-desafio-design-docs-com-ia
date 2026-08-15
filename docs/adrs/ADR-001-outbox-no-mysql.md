# ADR-001 — Padrão Outbox no MySQL existente

- **Status:** Aceito
- **Data de registro:** 2026-08-15 (data de elaboração deste registro; a decisão foi tomada na reunião técnica transcrita em `TRANSCRICAO.md`)
- **Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior), Bruno (Eng. Pleno)

## Contexto

O OMS precisa notificar clientes B2B externos quando o status de um pedido muda ([09:00] Marcos). A mudança de status hoje ocorre em uma transação Prisma pesada em `src/modules/orders/order.service.ts` (método `changeStatus`): atualiza `orders`, insere em `order_status_history` e ajusta `stock_quantity` dos produtos ([09:04] Bruno; confirmado no código). Qualquer mecanismo de notificação precisa garantir que **não exista** o caso de o status mudar sem o evento ser registrado, nem o inverso ([09:40] Bruno, [09:41] Diego).

## Decisão

Adotar o **padrão Outbox no MySQL já existente**: na mesma transação SQL que atualiza `orders` e `order_status_history`, inserir uma linha em uma tabela de outbox (`webhook_outbox`) com o evento. Um worker separado lê essa tabela e dispara as chamadas HTTP ([09:06] Diego; fechado em [09:08] Larissa: "Tá decidido então: outbox em MySQL").

A tabela terá índice no campo de status do evento (pendente, processando, falhou, entregue) e em `created_at`; o worker lê apenas pendentes em batch pequeno ([09:08] Diego). A chave primária será UUID, seguindo o padrão do projeto ([09:51] Larissa; todos os modelos em `prisma/schema.prisma` usam `@default(uuid())`). O evento guarda o **payload já renderizado (snapshot)** no momento da inserção, para refletir o estado do pedido quando o status mudou ([09:52] Larissa/Diego/Bruno).

## Alternativas consideradas

1. **Chamada HTTP síncrona dentro do `changeStatus`** (discutida na reunião) — rejeitada: um cliente lento travaria mudanças de status de outros pedidos, e uma indisponibilidade do cliente exigiria rollback da transação, o que não faz sentido ([09:04] Bruno; [09:06] Diego: "Síncrono está fora de questão").
2. **Infraestrutura adicional de mensageria, ex.: Redis Streams** (discutida na reunião) — rejeitada: exigiria subir e operar mais infraestrutura; para um time pequeno é overengineering quando o MySQL existente resolve ([09:07] Larissa; [09:07] Diego).

## Consequências positivas

- Consistência garantida por atomicidade: commit da transação ⇒ evento registrado; rollback ⇒ evento descartado junto ([09:06] Diego).
- Nenhuma infraestrutura nova: reusa MySQL, Prisma e o pool de conexão existentes ([09:07] Diego; `src/config/database.ts`).
- Evento com snapshot imutável do estado do pedido ([09:52]).

## Consequências negativas e trade-offs

- A tabela de outbox cresce continuamente; linhas entregues precisarão de arquivamento futuro — sugerido "depois de 30 dias ou assim", explicitamente **fora do escopo desta feature** ([09:08] Diego).
- A entrega depende de um worker de polling (latência mínima do intervalo de polling — ver [ADR-005](ADR-005-worker-separado-com-polling.md)).
- Escrita adicional na transação de `changeStatus` (custo aceito em troca da garantia de consistência).

## Origem

- `TRANSCRICAO.md`: [09:04] Bruno, [09:06]–[09:08] Diego/Larissa, [09:40]–[09:41] Bruno/Diego, [09:51]–[09:52] Larissa/Diego/Bruno.
- Código: `src/modules/orders/order.service.ts` (transação de `changeStatus`), `prisma/schema.prisma` (padrão UUID e modelos `orders`/`order_status_history`), `src/config/database.ts`.
