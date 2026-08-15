# ADR-004 — Garantia de entrega at-least-once com deduplicação por `X-Event-Id`

- **Status:** Aceito
- **Data de registro:** 2026-08-15 (data de elaboração deste registro)
- **Decisores:** Diego (Eng. Sênior), Larissa (Tech Lead), Sofia (Eng. de Segurança), Marcos (PM)

## Contexto

Com outbox + worker + retry, existe a possibilidade de o mesmo evento ser entregue mais de uma vez (ex.: entrega bem-sucedida cuja confirmação falha antes de o worker marcar o evento como entregue). É preciso definir a semântica de entrega oferecida ao cliente ([09:24] Diego).

## Decisão

Fechado em [09:26] Larissa: "At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão."

- A plataforma garante **at-least-once**: o cliente pode receber o mesmo evento duas vezes e deve estar preparado ([09:24] Diego).
- Cada evento carrega um **`event_id` UUID**, gerado quando o evento entra na outbox, enviado no header **`X-Event-Id`**. É único por evento; o cliente deduplica por ele ([09:25] Diego).
- A responsabilidade de deduplicação é do consumidor — reconhecido explicitamente como transferência de responsabilidade ([09:25] Sofia), aceito por ser o padrão de mercado (Stripe e GitHub fazem assim, [09:25] Diego). O PM documentará isso em destaque no portal de desenvolvedores ([09:26] Marcos).

## Alternativas consideradas

1. **Exactly-once** (discutida na reunião) — rejeitada: exigiria coordenação dos dois lados e ficaria muito mais complexo; at-least-once com `event_id` "resolve 99% dos casos" ([09:25] Diego).
2. **At-most-once (enviar uma única vez, sem retry)** (análise posterior, não discutida na reunião) — incompatível com a decisão de retry/backoff do [ADR-002](ADR-002-retry-com-backoff-e-dlq.md): perderia eventos em qualquer indisponibilidade do cliente, contrariando o requisito central da feature.

## Consequências positivas

- Semântica simples e implementável sem coordenação com o cliente ([09:25]).
- Alinhada ao padrão de mercado, com documentação farta para os clientes ([09:25]–[09:26]).
- O `event_id` na outbox também serve de chave natural para histórico e replay.

## Consequências negativas e trade-offs

- Duplicidades são possíveis por design; cliente que não deduplicar processará eventos repetidos ([09:24]–[09:25]).
- A garantia depende de comunicação clara: mitigada pela documentação destacada no portal ([09:26] Marcos).

## Origem

- `TRANSCRICAO.md`: [09:24] Diego; [09:25] Bruno/Diego/Sofia; [09:26] Marcos/Larissa.
- Código: geração de UUID já é padrão do projeto (`uuid` em `package.json`; `@default(uuid())` em `prisma/schema.prisma`; `uuidv4()` em `src/middlewares/request-logger.middleware.ts`).
