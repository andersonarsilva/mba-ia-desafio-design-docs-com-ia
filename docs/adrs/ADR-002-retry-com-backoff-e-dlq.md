# ADR-002 — Retry com backoff exponencial e DLQ em tabela separada

- **Status:** Aceito
- **Data de registro:** 2026-08-15 (data de elaboração deste registro)
- **Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior), Bruno (Eng. Pleno), Marcos (PM)

## Contexto

O endpoint do cliente pode estar indisponível no momento da entrega do webhook. Foi citado caso real de cliente com indisponibilidade de duas horas em manutenção planejada ([09:16] Diego). É preciso decidir quantas vezes retentar, com qual progressão, e o que fazer quando as tentativas se esgotam ([09:14] Larissa).

## Decisão

- **Retry com backoff exponencial** e teto de tentativas; após o teto, o evento é considerado falha permanente ([09:15] Diego).
- **Intervalos decididos:** `1m / 5m / 30m / 2h / 12h` — "total de quase 15 horas entre primeira falha e última tentativa" ([09:17] Diego; fechado em [09:17] Larissa: "Decidido: 5 tentativas, backoff 1m/5m/30m/2h/12h").
- **DLQ em tabela separada** (`webhook_dead_letter`), com payload, motivo da falha e timestamp — mantém a leitura da outbox limpa e serve de evidência para debug e reprocessamento ([09:18] Diego).
- **Replay manual via endpoint admin** (`POST /admin/webhooks/dead-letter/:id/replay`), que recoloca o evento na outbox como pendente ([09:18] Diego), com exigência de role `ADMIN` e log de auditoria de quem executou ([09:36] Sofia) — detalhado em [ADR-003](ADR-003-autenticacao-hmac-por-endpoint.md) (autorização) e no FDD.

**Ponto pendente de confirmação (ambiguidade registrada):** a reunião fechou "5 tentativas" e, ao mesmo tempo, **5 intervalos** de retry. Se são 5 tentativas totais, apenas 4 intervalos seriam usados; se os 5 intervalos valem, são 5 *retentativas* após o envio inicial (6 envios no total). A conta de Diego ("quase 15 horas entre primeira falha e última tentativa" = 1m+5m+30m+2h+12h ≈ 14h36m) é consistente com **1 envio inicial + 5 retentativas**. Essa é a interpretação recomendada como **proposta de design**, pendente de confirmação com o time. Ver detalhamento no FDD.

## Alternativas consideradas

1. **Retry indefinido com backoff** (discutida na reunião) — rejeitada: evento fica pendurado para sempre se o cliente sumiu ([09:15] Diego).
2. **Três tentativas** (discutida na reunião) — rejeitada: cobriria só ~30 minutos e mataria eventos durante indisponibilidades reais de horas ([09:16] Bruno propôs; [09:16] Diego refutou com caso real).
3. **DLQ marcada como `failed` na própria outbox** (discutida na reunião) — rejeitada em favor de tabela separada, por legibilidade da outbox e evidência de debug ([09:17] Larissa levantou; [09:18] Diego decidiu tabela separada).

## Consequências positivas

- Cobre janelas de indisponibilidade de até ~15 horas sem intervenção manual ([09:17]).
- Falhas permanentes ficam isoladas e auditáveis na `webhook_dead_letter`, com replay controlado ([09:18]).
- Outbox principal permanece enxuta para o polling do worker.

## Consequências negativas e trade-offs

- Evento pode levar ~15 horas até ser declarado falha permanente — aceito pelo PM ("Se um cliente meu cair por 15 horas, ele já tá com problema sério dele", [09:17] Marcos).
- Reprocessamento da DLQ é manual, dependendo de um operador `ADMIN` ([09:18], [09:36]).
- A semântica exata do contador de tentativas permanece ambígua até confirmação (ver acima).

## Origem

- `TRANSCRICAO.md`: [09:14] Larissa; [09:15]–[09:18] Diego/Bruno/Larissa/Marcos ([09:15] Diego/Bruno, [09:16] Bruno/Diego/Larissa, [09:17] Diego/Marcos/Larissa, [09:18] Diego/Bruno); [09:36] Sofia.
- Código: padrão de tabelas auditáveis já existente em `prisma/schema.prisma` (`order_status_history`) como referência de modelagem.
