# ADR-006 — Reuso máximo dos padrões existentes do projeto

- **Status:** Aceito
- **Data de registro:** 2026-08-15 (data de elaboração deste registro)
- **Decisores:** Bruno (Eng. Pleno), Larissa (Tech Lead), Diego (Eng. Sênior)

## Contexto

A codebase tem um padrão claro e consolidado: cada domínio é um módulo em `src/modules` com controller, service, repository, routes e schemas ([09:27] Bruno; confirmado por inspeção em `src/modules/orders/` e `src/modules/products/`). Há hierarquia de erros (`AppError` base em `src/shared/errors/app-error.ts`, subclasses concretas como `InsufficientStockError` e `InvalidStatusTransitionError` em `src/shared/errors/http-errors.ts`), logger Pino (`src/shared/logger/index.ts`), middleware de erro centralizado que já trata `AppError`, Zod e Prisma (`src/middlewares/error.middleware.ts`) e autorização por role com `requireRole` (`src/middlewares/auth.middleware.ts`).

## Decisão

Fechado em [09:30] Larissa: "Decisão: reuso máximo do que já existe. AppError, Pino, error middleware, padrão de módulos, padrão de schemas Zod, padrão de códigos de erro. Webhook fica como módulo igual aos outros."

- Novo módulo em **`src/modules/webhooks`** com a mesma estrutura dos demais ([09:27] Bruno).
- Entry do worker em `src/worker.ts`, com a lógica de processamento dentro do módulo (ex.: `src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts` — nome exato ainda em aberto na fala de [09:28] Bruno).
- **Erros seguem o padrão existente**: códigos com prefixo `WEBHOOK_` (ex.: `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`), nos moldes de `INSUFFICIENT_STOCK`/`INVALID_STATUS_TRANSITION` ([09:28] Bruno; [09:29] Larissa). O código mostra `AppError` como classe base e subclasses concretas em `src/shared/errors/http-errors.ts`; a **proposta de design** é reutilizar essas subclasses e, quando necessário, especializar novas classes em arquivo do próprio módulo de webhooks — **sem alterar** `app-error.ts`.
- **Logger Pino** já presente no projeto inteiro; nada novo de logging ([09:29] Bruno).
- **Middleware de erro centralizado** já trata `AppError`, Zod e Prisma; capturará os erros do módulo sem mudanças ([09:29] Bruno).
- **`requireRole` reaproveitado** para o endpoint admin de replay da DLQ ([09:36] Larissa; implementação em `src/middlewares/auth.middleware.ts`, uso existente em `src/modules/users/user.routes.ts`).
- Worker usa **`PrismaClient` separado por processo**, mesma `DATABASE_URL` ([09:30] Bruno).

## Alternativas consideradas

1. **Introduzir padrões/bibliotecas novas para o módulo** (implícita na discussão; ex.: novo logger ou novo framework de erros) — rejeitada: "Não vamos botar nada novo" ([09:29] Bruno); aumentaria a superfície de manutenção sem benefício, dado que os padrões atuais já cobrem as necessidades.
2. **Módulo fora do padrão `src/modules` (ex.: serviço separado)** (análise posterior, não discutida na reunião) — rejeitada nesta análise: quebraria a convenção de composição usada em `src/app.ts` (`buildControllers`) e `src/routes/index.ts`, dificultando a integração transacional com o `OrderService`.

## Consequências positivas

- Curva de implementação e revisão baixa: o time já domina os padrões.
- Erros do módulo tratados de graça pelo middleware existente ([09:29]).
- Consistência de API (envelope de erro `{ error: { code, message, details } }`, paginação de `src/shared/http/response.ts`) para os consumidores.

## Consequências negativas e trade-offs

- O módulo herda as limitações da stack atual (ex.: sem mensageria dedicada — coerente com [ADR-001](ADR-001-outbox-no-mysql.md)).
- Acoplamento ao monólito: o worker, embora processo separado, compartilha código e schema com a API — aceitável para o tamanho do time ([09:07] Diego).

## Origem

- `TRANSCRICAO.md`: [09:27]–[09:30] Bruno/Larissa/Diego, [09:36] Larissa.
- Código: `src/modules/orders/` e `src/modules/products/` (estrutura de módulo), `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`, `src/shared/logger/index.ts`, `src/middlewares/error.middleware.ts`, `src/middlewares/auth.middleware.ts`, `src/app.ts`, `src/routes/index.ts`.
