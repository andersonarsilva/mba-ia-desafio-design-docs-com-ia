# Architectural Decision Records

Este diretório armazena os ADRs (Architectural Decision Records) do projeto.
Cada decisão arquitetural relevante é registrada em arquivo individual, no
formato `ADR-NNN-titulo-em-kebab-case.md`, seguindo uma variante do MADR.

## Índice — Sistema de Webhooks de Notificação de Pedidos

| ADR | Decisão | Status |
|-----|---------|--------|
| [ADR-001](ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL existente | Aceito |
| [ADR-002](ADR-002-retry-com-backoff-e-dlq.md) | Retry com backoff exponencial (1m/5m/30m/2h/12h) e DLQ em tabela separada | Aceito |
| [ADR-003](ADR-003-autenticacao-hmac-por-endpoint.md) | Assinatura HMAC-SHA256 com secret por endpoint e rotação com grace period de 24h | Aceito |
| [ADR-004](ADR-004-entrega-at-least-once.md) | Garantia de entrega at-least-once com deduplicação por `X-Event-Id` | Aceito |
| [ADR-005](ADR-005-worker-separado-com-polling.md) | Worker em processo separado com polling de 2 segundos | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso máximo dos padrões existentes do projeto | Aceito |

A origem de cada decisão (timestamps da reunião e caminhos de código) está na
seção "Origem" de cada ADR e no [Tracker de rastreabilidade](../TRACKER.md).
