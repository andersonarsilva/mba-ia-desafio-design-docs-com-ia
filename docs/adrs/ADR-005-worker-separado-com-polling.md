# ADR-005 — Worker em processo separado com polling de 2 segundos

- **Status:** Aceito
- **Data de registro:** 2026-08-15 (data de elaboração deste registro)
- **Decisores:** Diego (Eng. Sênior), Larissa (Tech Lead), Bruno (Eng. Pleno), Marcos (PM)

## Contexto

Com a outbox decidida ([ADR-001](ADR-001-outbox-no-mysql.md)), é preciso definir como os eventos pendentes são consumidos e entregues. Os clientes consideram "tempo real" qualquer entrega abaixo de 10 segundos ([09:02] Marcos).

## Decisão

- **Polling em loop: a cada 2 segundos**, o worker busca os eventos pendentes mais antigos, processa e marca ([09:09] Diego; fechado em [09:10] Larissa: "Worker em polling, 2s. A latência mínima vai ser 2 segundos no pior caso. Aceitamos."; [09:10] Marcos: "2 segundos serve").
- **Processo separado da API**: o worker não roda dentro da instância da API — se a API reinicia, perderia o worker ([09:11] Diego). Entry point novo no projeto, nos moldes de `src/server.ts`: criar `src/worker.ts` e um script `npm run worker` ([09:11] Larissa).
- **Mesmo banco, mesma stack, processo distinto**: o worker conecta na mesma `DATABASE_URL`, mas com **instância própria de `PrismaClient`**, pois o client é por processo ([09:11] Bruno/Diego; [09:30] Bruno; padrão visível em `src/config/database.ts`).
- **Single-worker com ordering implícita por `order_id`**: um único worker processando em ordem de `created_at` da outbox entrega em ordem; escalar para múltiplos workers perderia a garantia. Registrado como **limitação conhecida**: não há garantia de ordering global, só por pedido e enquanto for single-worker ([09:12]–[09:13] Diego/Larissa). Particionamento por `order_id` ou lock pessimista ficam como "problema do futuro" ([09:13] Diego).

## Alternativas consideradas

1. **Worker dentro do mesmo processo da API** (discutida na reunião) — rejeitada: reinício da API derrubaria o worker ([09:11] Diego).
2. **Trigger do banco para notificar o worker** (discutida na reunião) — rejeitada: MySQL não tem listener nativo tipo NOTIFY/LISTEN do Postgres; trigger só executa SQL e avisar um processo externo exigiria improvisos (escrever em arquivo, bater em endpoint), "fica esquisito" ([09:09] Bruno propôs; [09:09] Diego refutou).
3. **Polling** (escolhida) — atende o requisito de menos de 10 segundos com folga ([09:09] Diego).

## Consequências positivas

- Isolamento de falhas: reinícios/deploys da API não interrompem a entrega de webhooks ([09:11]).
- Latência de detecção de no máximo ~2s, compatível com o requisito de <10s ([09:09]–[09:10]).
- Reuso integral da stack existente (Node/TS, Prisma, Pino), sem infraestrutura nova.

## Consequências negativas e trade-offs

- Polling gera consultas constantes ao MySQL mesmo sem eventos (custo aceito; batch pequeno e índices previstos, [09:08] Diego).
- Single-worker é um limite de throughput e um ponto único de processamento; escala horizontal exigirá particionamento futuro ([09:13]).
- **Análise posterior (não discutida na reunião):** o retry com backoff ([ADR-002](ADR-002-retry-com-backoff-e-dlq.md)) quebra a ordering por `order_id` mesmo em single-worker — um evento posterior do mesmo pedido pode ser entregue enquanto o anterior aguarda backoff. A ordering por pedido é melhor esforço, não garantia; a mitigação por bloqueio de eventos posteriores do mesmo pedido durante o backoff fica como proposta em aberto (ver FDD e RFC).
- O intervalo de 2s é a latência **mínima** adicionada no pior caso; a latência ponta a ponta sob carga precisa ser medida (questão registrada no RFC).

## Origem

- `TRANSCRICAO.md`: [09:02] Marcos, [09:08]–[09:13] Diego/Larissa/Bruno, [09:30] Bruno.
- Código: `src/server.ts` (modelo de entry point), `src/config/database.ts` (criação de `PrismaClient` por processo), `package.json` (seção `scripts`, onde entraria `npm run worker`).
