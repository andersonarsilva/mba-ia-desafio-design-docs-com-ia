# RFC — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
|---|---|
| **Autor** | Anderson Silva |
| **Status** | Em revisão |
| **Data de elaboração** | 2026-08-15 |
| **Revisores** | Larissa (Tech Lead), Marcos (PM), Bruno (Eng. Pleno), Diego (Eng. Sênior), Sofia (Eng. de Segurança) — participantes da reunião técnica |
| **Documentos relacionados** | [PRD](PRD.md) · [FDD](FDD.md) · [Tracker](TRACKER.md) |

## Resumo executivo (TL;DR)

Propomos notificar clientes B2B sobre mudanças de status de pedidos por meio de **webhooks outbound** baseados no **padrão Outbox no MySQL existente**: a mudança de status registra o evento na mesma transação que já atualiza `orders` e `order_status_history`; um **worker em processo separado**, em **polling de 2 segundos**, entrega os eventos via HTTP POST assinado com **HMAC-SHA256 e secret por endpoint**, com **retry com backoff (1m/5m/30m/2h/12h)** e **DLQ em tabela separada** com replay administrativo. Garantia de entrega **at-least-once**, com deduplicação pelo cliente via `X-Event-Id`. Nenhuma infraestrutura nova; reuso integral dos padrões do projeto.

## Contexto e problema

Três clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo) pediram formalmente notificação em tempo real quando o status dos seus pedidos muda. Hoje eles fazem polling no `GET /orders`, o que torna a integração deles lenta e cara; a Atlas sinalizou risco de migração para concorrente se não houver entrega até o fim do trimestre ([09:00] Marcos). Para os clientes, "tempo real" é qualquer entrega **abaixo de 10 segundos** ([09:02] Marcos). O escopo é exclusivamente **outbound**: só enviamos, não recebemos ([09:02] Marcos/Sofia).

O OMS atual (Node.js + TypeScript, Express, Prisma/MySQL) não possui nenhum mecanismo de eventos, filas ou notificações externas. A mudança de status ocorre numa transação pesada em `src/modules/orders/order.service.ts` (`changeStatus`), que atualiza o pedido, grava histórico e ajusta estoque — qualquer solução precisa preservar essa atomicidade.

## Proposta técnica (visão geral)

1. **Registro do evento — Outbox transacional** ([ADR-001](adrs/ADR-001-outbox-no-mysql.md)): ao mudar o status, inserir o evento numa tabela `webhook_outbox` **dentro da mesma transação Prisma** do `changeStatus`. Commit ⇒ evento garantido; rollback ⇒ evento descartado. O evento guarda o payload como **snapshot** do estado do pedido no momento da mudança. A inserção respeita o **filtro de eventos** de cada webhook: se nenhum webhook do customer assina aquele status, nada é inserido ([09:34] Bruno).
2. **Entrega — worker separado com polling** ([ADR-005](adrs/ADR-005-worker-separado-com-polling.md)): novo entry point `src/worker.ts` (processo próprio, `PrismaClient` próprio, mesmo banco), lendo pendentes em batch a cada 2 segundos e fazendo o POST no endpoint do cliente com timeout de 10 segundos.
3. **Resiliência — retry + DLQ** ([ADR-002](adrs/ADR-002-retry-com-backoff-e-dlq.md)): falhas agendam retentativas com backoff 1m/5m/30m/2h/12h; esgotado o teto, o evento vai para a tabela `webhook_dead_letter`, com replay manual por endpoint admin (role `ADMIN`, com auditoria).
4. **Segurança — HMAC por endpoint** ([ADR-003](adrs/ADR-003-autenticacao-hmac-por-endpoint.md)): assinatura HMAC-SHA256 sobre o corpo, header `X-Signature`, secret única por endpoint, rotação via API com grace period de 24h, URLs exclusivamente `https`.
5. **Semântica de entrega — at-least-once** ([ADR-004](adrs/ADR-004-entrega-at-least-once.md)): duplicidades são possíveis; o cliente deduplica pelo `X-Event-Id` (UUID por evento).
6. **Gestão pelos clientes — CRUD de configuração**: endpoints autenticados para cadastrar, listar, editar e remover webhooks, escolher os status assinados, consultar o histórico de entregas e rotacionar a secret ([09:31]–[09:34]).
7. **Aderência ao projeto** ([ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)): módulo `src/modules/webhooks` no padrão existente; `AppError` + códigos `WEBHOOK_*`; Pino; error middleware e `requireRole` reaproveitados.

O detalhamento de contratos, fluxos, matriz de erros e integração com o código fica no [FDD](FDD.md).

## Alternativas consideradas

| ID | Alternativa | Trade-off que levou ao descarte |
|---|---|---|
| RFC-ALT-01 | **Disparo síncrono dentro do `changeStatus`** | Cliente lento travaria mudanças de status de outros pedidos; indisponibilidade do cliente exigiria rollback da transação de negócio ([09:04] Bruno; [09:06] Diego). |
| RFC-ALT-02 | **Mensageria dedicada (ex.: Redis Streams)** | Exigiria subir e operar infraestrutura nova; overengineering para um time pequeno quando o MySQL existente resolve ([09:07] Larissa/Diego). |
| RFC-ALT-03 | **Trigger do MySQL para acordar o worker** | MySQL não tem NOTIFY/LISTEN; trigger só executa SQL e notificar processo externo exigiria improvisos frágeis. Polling de 2s já atende o requisito de <10s ([09:09] Bruno/Diego). |
| RFC-ALT-04 | **Retry indefinido / apenas 3 tentativas** | Indefinido deixa evento pendurado para sempre; 3 tentativas cobrem só ~30 min e falham em manutenções reais de horas ([09:15]–[09:16] Diego/Bruno). |
| RFC-ALT-05 | **Exactly-once** | Exigiria coordenação dos dois lados e complexidade muito maior; at-least-once com `event_id` cobre 99% dos casos ([09:25] Diego). |

## Questões em aberto

| ID | Questão | Situação |
|---|---|---|
| RFC-QA-01 | **Rate limiting de envio por cliente** (ex.: 50 mudanças de status em 1 minuto ⇒ 50 POSTs) | Adiado: "observar e decidir depois" ([09:38]–[09:39] Diego/Larissa). |
| RFC-QA-02 | **Semântica de "5 tentativas"** vs. os 5 intervalos de backoff (5 envios totais com 4 intervalos, ou envio inicial + 5 retentativas?) | Ambiguidade na reunião ([09:15]–[09:17]); interpretação recomendada no FDD como proposta pendente de confirmação. |
| RFC-QA-03 | **Medição da meta de <10s** | O polling de 2s cobre a latência de detecção, mas a reunião não definiu percentil, janela de medição nem volume esperado; a latência ponta a ponta sob carga (fila acumulada, retries) precisa ser medida e a métrica operacionalizada. |
| RFC-QA-04 | **Proteção contra replay na entrega** | `X-Timestamp` é enviado, mas a assinatura fechada cobre apenas o corpo ([09:22], [09:44]); incluir o timestamp na assinatura ou exigir validação do cliente não foi decidido. |
| RFC-QA-05 | **Endurecimento de autorização do CRUD** | Por enquanto qualquer role autenticada; "mais pra frente a gente pode endurecer" ([09:36]–[09:37] Sofia). |
| RFC-QA-06 | **Escala do worker e ordering sob retries** | Single-worker por ora; particionamento por `order_id` ou lock pessimista são "problema do futuro" ([09:13] Diego). Análise posterior: retries em backoff quebram a ordering por pedido mesmo em single-worker — reter eventos posteriores do mesmo pedido durante o backoff é mitigação possível, ainda sem decisão. |

## Impacto e riscos

- **Impacto no código existente:** concentrado no `changeStatus` de `src/modules/orders/order.service.ts` (inserção na outbox dentro da transação) e na composição de rotas/dependências (`src/app.ts`, `src/routes/index.ts`). O restante é módulo novo.
- **Impacto operacional:** um processo novo para operar (worker), tabelas novas no MySQL e crescimento contínuo da outbox (arquivamento ficou fora do escopo, [09:08]).
- **Riscos principais** (avaliação do documento, detalhes no [PRD](PRD.md)): cliente que não deduplica recebe duplicatas (mitigado por documentação no portal, [09:26]); acúmulo de eventos degradando o polling (mitigado por índices e batch pequeno, [09:08]); vazamento de secret (mitigado por secret por endpoint e rotação, [09:21]–[09:22]).
- **Prazo:** estimativa de 3 sprints, incluindo revisão de segurança da Sofia (mínimo 2 dias úteis antes do deploy) ([09:45]–[09:47]).

## Decisões relacionadas

- [ADR-001 — Padrão Outbox no MySQL](adrs/ADR-001-outbox-no-mysql.md)
- [ADR-002 — Retry com backoff e DLQ](adrs/ADR-002-retry-com-backoff-e-dlq.md)
- [ADR-003 — HMAC-SHA256 com secret por endpoint](adrs/ADR-003-autenticacao-hmac-por-endpoint.md)
- [ADR-004 — Entrega at-least-once](adrs/ADR-004-entrega-at-least-once.md)
- [ADR-005 — Worker separado com polling](adrs/ADR-005-worker-separado-com-polling.md)
- [ADR-006 — Reuso dos padrões do projeto](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)
