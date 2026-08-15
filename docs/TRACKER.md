# Tracker de Rastreabilidade — Sistema de Webhooks de Notificação de Pedidos

Cada linha mapeia um item identificável dos documentos à sua origem primária: `TRANSCRICAO` (formato `[hh:mm] Nome`, conforme `TRANSCRICAO.md`) ou `CODIGO` (caminho real no repositório). Itens marcados como proposta nos documentos apontam para a decisão/requisito que os motivou ou para o padrão de código do qual derivam.

**Cobertura:** 126 itens identificáveis nos documentos · 126 rastreados · **100%**.
**Fontes:** 107 linhas `TRANSCRICAO` (84,9%) · 19 linhas `CODIGO`.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|----|-----------|------|-------------------|-------|-------------|
| PRD-OBJ-01 | docs/PRD.md | Objetivo | Entrega da notificação em <10s (operacionalização da métrica pendente) | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Objetivo | Eliminar polling dos clientes no GET /orders | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-03 | docs/PRD.md | Objetivo | Atender pedido dos 3 clientes no prazo (fim de novembro / 3 sprints) | TRANSCRICAO | [09:45] Marcos |
| PRD-RF-01 | docs/PRD.md | Requisito Funcional | Cadastro de webhook: URL https, filtro de status, secret gerada e devolvida; customerId no request | TRANSCRICAO | [09:31] Marcos |
| PRD-RF-02 | docs/PRD.md | Requisito Funcional | Listar webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-03 | docs/PRD.md | Requisito Funcional | Editar configuração/filtro de eventos (PATCH) | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-04 | docs/PRD.md | Requisito Funcional | Remover webhook (DELETE) | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-05 | docs/PRD.md | Requisito Funcional | Filtro de eventos aplicado na inserção na outbox | TRANSCRICAO | [09:34] Bruno |
| PRD-RF-06 | docs/PRD.md | Requisito Funcional | Entrega de POST JSON assinado por mudança de status assinada | TRANSCRICAO | [09:43] Diego |
| PRD-RF-07 | docs/PRD.md | Requisito Funcional | Histórico de entregas (sucesso/falha, payload, response, tempo) | TRANSCRICAO | [09:34] Marcos |
| PRD-RF-08 | docs/PRD.md | Requisito Funcional | Rotação de secret via API com grace de 24h | TRANSCRICAO | [09:21] Sofia |
| PRD-RF-09 | docs/PRD.md | Requisito Funcional | Replay de DLQ por ADMIN com auditoria do executor | TRANSCRICAO | [09:36] Sofia |
| PRD-RF-10 | docs/PRD.md | Requisito Funcional | Retry automático com backoff; esgotado o teto, DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-RF-11 | docs/PRD.md | Requisito Funcional | X-Event-Id único por evento para deduplicação do cliente | TRANSCRICAO | [09:25] Diego |
| PRD-RF-12 | docs/PRD.md | Requisito Funcional | CRUD com qualquer role autenticada; só replay exige ADMIN | TRANSCRICAO | [09:37] Sofia |
| PRD-RNF-01 | docs/PRD.md | Requisito Não Funcional | Latência de notificação <10s | TRANSCRICAO | [09:02] Marcos |
| PRD-RNF-02 | docs/PRD.md | Requisito Não Funcional | Garantia at-least-once; dedup pelo consumidor | TRANSCRICAO | [09:26] Larissa |
| PRD-RNF-03 | docs/PRD.md | Requisito Não Funcional | Evento atômico com a mudança de status (mesma transação) | TRANSCRICAO | [09:40] Bruno |
| PRD-RNF-04 | docs/PRD.md | Requisito Não Funcional | HMAC-SHA256 sobre o corpo, secret por endpoint | TRANSCRICAO | [09:22] Sofia |
| PRD-RNF-05 | docs/PRD.md | Requisito Não Funcional | Apenas URLs https; http recusado na validação | TRANSCRICAO | [09:23] Sofia |
| PRD-RNF-06 | docs/PRD.md | Requisito Não Funcional | Payload máximo 64 KB; excedente vira erro | TRANSCRICAO | [09:24] Larissa |
| PRD-RNF-07 | docs/PRD.md | Requisito Não Funcional | Timeout de 10s por tentativa de entrega | TRANSCRICAO | [09:42] Diego |
| PRD-RNF-08 | docs/PRD.md | Restrição | Ordering por order_id em melhor esforço (single-worker, sem retries); retries podem inverter a ordem — limitação registrada | TRANSCRICAO | [09:13] Larissa |
| PRD-RNF-09 | docs/PRD.md | Requisito Não Funcional | Auditoria de quem executa o replay | TRANSCRICAO | [09:36] Sofia |
| PRD-RNF-10 | docs/PRD.md | Requisito Não Funcional | Worker em processo separado da API | TRANSCRICAO | [09:11] Diego |
| PRD-FE-01 | docs/PRD.md | Fora de escopo | Email de aviso de falha — adiado para próxima fase | TRANSCRICAO | [09:37] Larissa |
| PRD-FE-02 | docs/PRD.md | Fora de escopo | Rate limiting de envio — observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| PRD-FE-03 | docs/PRD.md | Fora de escopo | Dashboard visual — projeto separado do frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-FE-04 | docs/PRD.md | Fora de escopo | Arquivamento de linhas entregues ("30 dias ou assim") fora do escopo | TRANSCRICAO | [09:08] Diego |
| PRD-FE-05 | docs/PRD.md | Fora de escopo | Webhooks inbound — escopo é só outbound | TRANSCRICAO | [09:02] Marcos |
| PRD-RISCO-01 | docs/PRD.md | Risco | Cliente sem dedup processa duplicatas (responsabilidade transferida) | TRANSCRICAO | [09:25] Sofia |
| PRD-RISCO-02 | docs/PRD.md | Risco | Acúmulo na outbox degradando latência | TRANSCRICAO | [09:07] Bruno |
| PRD-RISCO-03 | docs/PRD.md | Risco | Vazamento de secret pelo cliente (caso real citado) | TRANSCRICAO | [09:22] Diego |
| PRD-RISCO-04 | docs/PRD.md | Risco | Perda de prazo e churn da Atlas | TRANSCRICAO | [09:00] Marcos |
| PRD-CA-01 | docs/PRD.md | Critério de aceitação | CRUD completo funcional com secret devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-CA-02 | docs/PRD.md | Critério de aceitação | URL http recusada com erro de validação | TRANSCRICAO | [09:23] Sofia |
| PRD-CA-03 | docs/PRD.md | Critério de aceitação | Notificação assinada e verificável por webhook interessado | TRANSCRICAO | [09:20] Sofia |
| PRD-CA-04 | docs/PRD.md | Critério de aceitação | Status não assinado não gera notificação | TRANSCRICAO | [09:34] Bruno |
| PRD-CA-05 | docs/PRD.md | Critério de aceitação | Retry nos intervalos definidos e evento visível para reprocessamento | TRANSCRICAO | [09:17] Larissa |
| PRD-CA-06 | docs/PRD.md | Critério de aceitação | Replay negado sem ADMIN e auditado quando executado | TRANSCRICAO | [09:36] Sofia |
| PRD-CA-07 | docs/PRD.md | Critério de aceitação | Histórico de entregas consultável | TRANSCRICAO | [09:34] Marcos |
| PRD-CA-08 | docs/PRD.md | Critério de aceitação | Convivência de secrets por 24h após rotação | TRANSCRICAO | [09:21] Sofia |
| PRD-CA-09 | docs/PRD.md | Critério de aceitação | Notificação em <10s em operação normal | TRANSCRICAO | [09:02] Marcos |
| PRD-DEP-01 | docs/PRD.md | Dependência | Aplicação OMS existente (transação changeStatus, MySQL/Prisma, JWT, requireRole) | CODIGO | src/modules/orders/order.service.ts |
| PRD-DEP-02 | docs/PRD.md | Dependência | Documentação da integração no portal de desenvolvedores (compromisso do PM) | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Revisão de segurança da Sofia (mínimo 2 dias úteis) antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-04 | docs/PRD.md | Dependência | Confirmação de prazo com os clientes pelo PM | TRANSCRICAO | [09:47] Marcos |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo síncrono no changeStatus | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Mensageria dedicada (Redis Streams) | TRANSCRICAO | [09:07] Larissa |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger do MySQL para acordar o worker | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Retry indefinido / apenas 3 tentativas | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-QA-01 | docs/RFC.md | Questão em aberto | Rate limiting de envio por cliente | TRANSCRICAO | [09:39] Larissa |
| RFC-QA-02 | docs/RFC.md | Questão em aberto | Semântica de "5 tentativas" vs 5 intervalos de backoff | TRANSCRICAO | [09:17] Larissa |
| RFC-QA-03 | docs/RFC.md | Questão em aberto | Medição da meta de <10s (percentil/janela/volume indefinidos) | TRANSCRICAO | [09:02] Marcos |
| RFC-QA-04 | docs/RFC.md | Questão em aberto | Proteção contra replay (X-Timestamp fora da assinatura) | TRANSCRICAO | [09:44] Diego |
| RFC-QA-05 | docs/RFC.md | Questão em aberto | Endurecimento futuro da autorização do CRUD | TRANSCRICAO | [09:37] Sofia |
| RFC-QA-06 | docs/RFC.md | Questão em aberto | Escala do worker e ordering sob retries (bloqueio de eventos posteriores como mitigação proposta, sem decisão) | TRANSCRICAO | [09:13] Diego |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Padrão Outbox no MySQL existente, evento na mesma transação | TRANSCRICAO | [09:08] Larissa |
| ADR-002 | docs/adrs/ADR-002-retry-com-backoff-e-dlq.md | Decisão | Retry com backoff 1m/5m/30m/2h/12h e DLQ em tabela separada | TRANSCRICAO | [09:17] Larissa |
| ADR-003 | docs/adrs/ADR-003-autenticacao-hmac-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo, secret por endpoint, rotação com grace 24h | TRANSCRICAO | [09:22] Sofia |
| ADR-004 | docs/adrs/ADR-004-entrega-at-least-once.md | Decisão | At-least-once com X-Event-Id para dedup do cliente | TRANSCRICAO | [09:26] Larissa |
| ADR-005 | docs/adrs/ADR-005-worker-separado-com-polling.md | Decisão | Worker em processo separado com polling de 2s | TRANSCRICAO | [09:10] Larissa |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reuso máximo dos padrões existentes do projeto | TRANSCRICAO | [09:30] Larissa |
| FDD-FLUXO-01 | docs/FDD.md | Fluxo | Mudança de status e criação atômica do evento via publishWebhookEvent(tx,...) | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-02 | docs/FDD.md | Fluxo | Filtragem por status na inserção da outbox | TRANSCRICAO | [09:34] Bruno |
| FDD-FLUXO-03 | docs/FDD.md | Fluxo | Polling do worker a cada 2s, batch pequeno, ordem por created_at | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-04 | docs/FDD.md | Fluxo | Assinatura HMAC e entrega HTTP com timeout | TRANSCRICAO | [09:22] Sofia |
| FDD-FLUXO-05 | docs/FDD.md | Fluxo | Sucesso e persistência do histórico de entrega | TRANSCRICAO | [09:34] Marcos |
| FDD-FLUXO-06 | docs/FDD.md | Fluxo | Falha, agendamento de retry e backoff (ambiguidade documentada) | TRANSCRICAO | [09:17] Diego |
| FDD-FLUXO-07 | docs/FDD.md | Fluxo | Esgotamento das tentativas e movimentação para DLQ | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-08 | docs/FDD.md | Fluxo | Replay administrativo da DLQ | TRANSCRICAO | [09:35] Diego |
| FDD-FLUXO-09 | docs/FDD.md | Fluxo | Rotação de secret com grace period de 24h | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST /api/v1/webhooks — criar configuração (path proposto) | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET /api/v1/webhooks?customerId= — listar (path proposto) | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | PATCH /api/v1/webhooks/:id — editar (path proposto) | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | DELETE /api/v1/webhooks/:id — remover (path proposto) | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | GET /webhooks/:id/deliveries — histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | POST /api/v1/webhooks/:id/rotate-secret — rotação (path proposto) | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | POST /admin/webhooks/dead-letter/:id/replay — replay ADMIN | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | POST de entrega ao endpoint do cliente (headers, payload, 64KB, 10s) | TRANSCRICAO | [09:44] Diego |
| FDD-ERR-01 | docs/FDD.md | Erro | WEBHOOK_NOT_FOUND (404) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-02 | docs/FDD.md | Erro | WEBHOOK_INVALID_URL (400, URL não-https) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-03 | docs/FDD.md | Erro | WEBHOOK_SECRET_REQUIRED (400; condição proposta) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-04 | docs/FDD.md | Erro (proposto) | WEBHOOK_INVALID_STATUS_FILTER — derivado do enum OrderStatus | CODIGO | prisma/schema.prisma |
| FDD-ERR-05 | docs/FDD.md | Erro (proposto) | WEBHOOK_PAYLOAD_TOO_LARGE — limite de 64KB decidido | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-06 | docs/FDD.md | Erro (proposto) | WEBHOOK_DELIVERY_TIMEOUT — timeout de 10s decidido | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-07 | docs/FDD.md | Erro (proposto) | WEBHOOK_DEAD_LETTER_NOT_FOUND — replay de item inexistente | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-08 | docs/FDD.md | Erro (proposto) | WEBHOOK_ALREADY_INACTIVE — nos moldes de ConflictError existente | CODIGO | src/shared/errors/http-errors.ts |
| FDD-RES-01 | docs/FDD.md | Resiliência | Timeout de entrega de 10s | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | docs/FDD.md | Resiliência | Backoff 1m/5m/30m/2h/12h sem arredondamento | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-03 | docs/FDD.md | Resiliência | DLQ separada + replay manual | TRANSCRICAO | [09:18] Diego |
| FDD-RES-04 | docs/FDD.md | Resiliência | Atomicidade transacional do evento | TRANSCRICAO | [09:41] Diego |
| FDD-RES-05 | docs/FDD.md | Resiliência | Isolamento de processo do worker | TRANSCRICAO | [09:11] Diego |
| FDD-RES-06 | docs/FDD.md | Resiliência | Limite de payload 64KB com erro | TRANSCRICAO | [09:24] Larissa |
| FDD-OBS-01 | docs/FDD.md | Observabilidade (proposta) | Logs estruturados do ciclo de vida do evento no padrão Pino; extensão do redact para secrets | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-02 | docs/FDD.md | Observabilidade (proposta) | Métricas de backlog, latência de entrega e ponta a ponta, falhas e DLQ — fundamentadas na meta de <10s | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-03 | docs/FDD.md | Observabilidade (proposta) | Correlação por eventId nos logs, nos moldes do requestId existente | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-DEP-01 | docs/FDD.md | Dependência | Stack existente suficiente; HMAC via node:crypto; sem dependência nova obrigatória | CODIGO | package.json |
| FDD-DEP-02 | docs/FDD.md | Dependência | Novas tabelas via migration Prisma; nenhuma rota/tabela existente muda | CODIGO | prisma/migrations/20260519182739_init/migration.sql |
| FDD-DEP-03 | docs/FDD.md | Dependência | Novo processo worker (npm run worker) com PrismaClient próprio e mesma DATABASE_URL | TRANSCRICAO | [09:11] Larissa |
| FDD-DEP-04 | docs/FDD.md | Dependência (proposta) | Configuração por ambiente via schema Zod de env.ts, se valores virarem configuráveis | CODIGO | src/config/env.ts |
| FDD-RISCO-01 | docs/FDD.md | Risco técnico | Backlog na outbox sob pico | TRANSCRICAO | [09:08] Diego |
| FDD-RISCO-02 | docs/FDD.md | Risco técnico | Crash do worker com eventos presos em PROCESSING (mitigação proposta coberta pelo at-least-once) | TRANSCRICAO | [09:24] Diego |
| FDD-RISCO-03 | docs/FDD.md | Risco técnico | Replay attack na entrega (timestamp fora da assinatura) | TRANSCRICAO | [09:44] Diego |
| FDD-INT-01 | docs/FDD.md | Integração | changeStatus passa a inserir na outbox dentro da transação existente | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Novos modelos Prisma seguindo padrões do schema (proposta de modelagem) | CODIGO | prisma/schema.prisma |
| FDD-INT-03 | docs/FDD.md | Integração | Composição das dependências do módulo em buildControllers | CODIGO | src/app.ts |
| FDD-INT-04 | docs/FDD.md | Integração | Montagem das rotas do módulo no router da API | CODIGO | src/routes/index.ts |
| FDD-INT-05 | docs/FDD.md | Integração | authenticate nas rotas e requireRole('ADMIN') no replay | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-06 | docs/FDD.md | Integração | Reuso da hierarquia AppError/subclasses sem alterar a base | CODIGO | src/shared/errors/http-errors.ts |
| FDD-INT-07 | docs/FDD.md | Integração | Error middleware existente captura os erros do módulo sem mudanças | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-08 | docs/FDD.md | Integração | Logger Pino compartilhado (com redact) na API e no worker | CODIGO | src/shared/logger/index.ts |
| FDD-INT-09 | docs/FDD.md | Integração | createPrismaClient reutilizado; instância própria no processo do worker | CODIGO | src/config/database.ts |
| FDD-INT-10 | docs/FDD.md | Integração | Padrão de testes de integração (Vitest/Supertest/factories/setup) | CODIGO | tests/orders.test.ts |
| FDD-CA-01 | docs/FDD.md | Critério de aceite | Rollback não deixa linha na outbox; commit deixa | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-02 | docs/FDD.md | Critério de aceite | Status não assinado não insere na outbox | TRANSCRICAO | [09:34] Bruno |
| FDD-CA-03 | docs/FDD.md | Critério de aceite | Sucesso gera registro em deliveries sem reenvio indevido | TRANSCRICAO | [09:34] Marcos |
| FDD-CA-04 | docs/FDD.md | Critério de aceite | X-Signature validável com HMAC-SHA256 sobre o corpo | TRANSCRICAO | [09:20] Sofia |
| FDD-CA-05 | docs/FDD.md | Critério de aceite | Timeout de 10s gera falha e retry na progressão decidida | TRANSCRICAO | [09:42] Diego |
| FDD-CA-06 | docs/FDD.md | Critério de aceite | Esgotamento leva o evento à DLQ com payload/motivo/timestamp | TRANSCRICAO | [09:18] Diego |
| FDD-CA-07 | docs/FDD.md | Critério de aceite | Replay exige ADMIN e loga o executor | TRANSCRICAO | [09:36] Sofia |
| FDD-CA-08 | docs/FDD.md | Critério de aceite | URL http recusada com WEBHOOK_INVALID_URL | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-09 | docs/FDD.md | Critério de aceite | Rotação: secret antiga válida só por 24h (decidido); assinatura imediata com a nova é proposta pendente de confirmação | TRANSCRICAO | [09:21] Sofia |
| FDD-CA-10 | docs/FDD.md | Critério de aceite | Payload >64KB não é enviado e registra falha | TRANSCRICAO | [09:24] Larissa |
| FDD-CA-11 | docs/FDD.md | Critério de aceite | Testes de integração no padrão existente do repositório | CODIGO | tests/orders.test.ts |

## Cálculo de cobertura

```text
cobertura = itens identificáveis presentes no Tracker / total de itens identificáveis nos documentos × 100
          = 126 / 126 × 100 = 100%
```

- Total de itens identificáveis nos documentos (denominador): **126** — PRD: 47 (3 objetivos, 12 RF, 10 RNF, 5 fora de escopo, 4 dependências, 4 riscos, 9 critérios de aceitação) · RFC: 11 (5 alternativas, 6 questões em aberto) · ADRs: 6 · FDD: 62 (9 fluxos, 8 contratos, 8 erros, 6 resiliência, 3 observabilidade, 4 dependências, 3 riscos, 10 integrações, 11 critérios de aceite).
- Itens rastreados: **126** (100%).
- Linhas com fonte `TRANSCRICAO`: **107** (84,9% ≥ 70%).
- Linhas com fonte `CODIGO`: **19** (≥ 5).
- Parágrafos de ligação, metadados administrativos e texto de contexto não entram no denominador.
