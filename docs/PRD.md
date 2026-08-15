# PRD — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
|---|---|
| **Autor** | Anderson Silva |
| **Data de elaboração** | 2026-08-15 |
| **Status** | Em revisão |
| **Documentos relacionados** | [RFC](RFC.md) · [FDD](FDD.md) · [ADRs](adrs/) · [Tracker](TRACKER.md) |

## 1. Resumo e contexto

O OMS passará a notificar ativamente os clientes B2B, via webhooks HTTP, sempre que o status de um pedido deles mudar. Hoje a plataforma não tem nenhum mecanismo de notificação externa; os clientes descobrem mudanças fazendo polling no `GET /orders`. A feature inclui o cadastro e gestão de endpoints de webhook pelo cliente (com filtro dos status de interesse), entrega assinada e confiável dos eventos, histórico de entregas consultável e ferramentas administrativas de reprocessamento.

## 2. Problema e motivação

Três clientes B2B — Atlas Comercial, MaxDistribuição e Nova Cargo — pediram formalmente notificação em tempo real de mudança de status de pedidos. O polling atual torna a integração deles "lenta e cara"; a Atlas sinalizou que pode migrar para o concorrente se não houver entrega até o fim do trimestre ([09:00] Marcos). Para os clientes, "tempo real" significa qualquer coisa **abaixo de 10 segundos** — o essencial é não depender de atualização manual ([09:02] Marcos).

## 3. Público-alvo e cenários de uso

**Público-alvo:** clientes B2B integradores da plataforma (inicialmente Atlas Comercial, MaxDistribuição e Nova Cargo), seus times de engenharia de integração, e o time interno de operação (role `ADMIN`) para reprocessamento ([09:00], [09:36]).

**Cenários de uso:**

1. Cliente cadastra um endpoint de webhook e escolhe receber apenas os status que lhe interessam (ex.: "só quero saber quando vira SHIPPED e DELIVERED") ([09:31]–[09:33]).
2. Um pedido do cliente muda de status; em segundos, o sistema entrega um POST assinado no endpoint cadastrado; o cliente valida a assinatura e atualiza o sistema dele ([09:19]–[09:20], [09:43]).
3. O endpoint do cliente fica indisponível por algumas horas (ex.: manutenção planejada); o sistema retenta com intervalos crescentes e entrega quando o endpoint volta ([09:16]–[09:17]).
4. Cliente audita a integração consultando o histórico das últimas entregas (sucesso/falha, resposta, tempo) ([09:34]).
5. Cliente rotaciona a secret após suspeita de vazamento, com 24h de convivência entre secrets para migrar seus sistemas ([09:21]–[09:22]).
6. Operador `ADMIN` reprocessa manualmente um evento que esgotou as tentativas e caiu na fila de falhas permanentes ([09:18], [09:36]).

## 4. Objetivos e métricas de sucesso

- **PRD-OBJ-01 (quantitativo):** entregar a notificação ao endpoint do cliente em **menos de 10 segundos** após a mudança de status — necessidade explicitada pelos clientes ([09:02] Marcos). O desenho suporta a meta com polling de 2s ([09:09]–[09:10]), mas intervalo de polling não é latência ponta a ponta: **percentil, janela de medição e volume não foram definidos na reunião — a operacionalização da métrica está pendente** (ver [RFC-QA-03](RFC.md#questões-em-aberto)).
- **PRD-OBJ-02:** eliminar a necessidade de polling dos clientes no `GET /orders` como mecanismo primário de acompanhamento ([09:00]).
- **PRD-OBJ-03 (negócio):** atender o pedido formal dos três clientes dentro do prazo — Atlas quer para fim de novembro; estimativa interna de 3 sprints ([09:45]–[09:47]).

## 5. Escopo incluso

- Cadastro, listagem, edição e remoção de webhooks por cliente, com filtro de status por endpoint ([09:31]–[09:33]).
- Geração de secret pela plataforma na criação, com devolução na resposta; rotação via API com grace period de 24h ([09:31], [09:21]).
- Entrega de eventos `order.status_changed` via HTTP POST assinado (HMAC-SHA256, headers de identificação), com payload enxuto sem itens do pedido ([09:43]–[09:44]).
- Confiabilidade: outbox transacional, retry com backoff, DLQ e replay administrativo ([09:06]–[09:18]).
- Histórico de entregas consultável por webhook ([09:34]).
- Somente **outbound**: nós enviamos, o cliente recebe ([09:02]).

## 6. Fora de escopo

| ID | Item | Motivo e origem |
|---|---|---|
| PRD-FE-01 | Aviso por email ao cliente quando o webhook dele falha repetidamente | Explicitamente adiado: "Email tá fora de escopo dessa fase. Talvez próxima fase, depois que a gente medir o impacto" ([09:37] Larissa; pedido de [09:37] Marcos) |
| PRD-FE-02 | Rate limiting de envio para o cliente | Adiado para observação: "A gente observa e implementa se virar problema (...) Fica como observar e decidir depois" ([09:38]–[09:39] Diego/Larissa) |
| PRD-FE-03 | Dashboard/painel visual para o cliente | Descartado nesta fase: "Não, agora não. Só endpoints. Painel é projeto separado do time de frontend" ([09:39]–[09:40] Marcos/Larissa) |
| PRD-FE-04 | Arquivamento das linhas entregues da outbox | Sugerido "depois de 30 dias ou assim", colocado explicitamente fora do escopo da feature ([09:08] Diego) |
| PRD-FE-05 | Webhooks inbound (cliente enviando para a plataforma) | Escopo confirmado como só outbound ([09:02] Marcos/Sofia) |

## 7. Requisitos funcionais

| ID | Requisito | Origem |
|---|---|---|
| PRD-RF-01 | Cliente pode cadastrar um webhook informando URL (https) e a lista de status que deseja receber; a secret é gerada pela plataforma e devolvida na criação; o `customerId` é informado na requisição (não vem do JWT, que é do usuário operador) | [09:31] Marcos; [09:32] Larissa; [09:23] Sofia |
| PRD-RF-02 | Cliente pode listar os webhooks de um customer | [09:33] Bruno |
| PRD-RF-03 | Cliente pode editar a configuração do webhook, incluindo o filtro de eventos | [09:33] Bruno/Marcos |
| PRD-RF-04 | Cliente pode remover um webhook | [09:33] Bruno |
| PRD-RF-05 | O filtro de eventos é aplicado na inserção na outbox: se nenhum webhook do customer assina o status, o evento não é registrado | [09:33]–[09:34] Marcos/Bruno |
| PRD-RF-06 | O sistema entrega, para cada mudança de status assinada, um HTTP POST com payload JSON do evento e headers de identificação e assinatura | [09:43]–[09:44] Diego/Sofia |
| PRD-RF-07 | Cliente pode consultar o histórico de entregas de um webhook (sucesso/falha, payload, response, tempo de resposta) | [09:34] Marcos |
| PRD-RF-08 | Cliente pode rotacionar a secret via API; a secret antiga permanece válida por 24 horas | [09:21] Sofia |
| PRD-RF-09 | Operador com role `ADMIN` pode reprocessar um item da fila de falhas permanentes (DLQ); a operação registra em auditoria quem a executou | [09:18] Diego; [09:36] Sofia/Larissa |
| PRD-RF-10 | Entregas com falha são retentadas automaticamente com intervalos crescentes; após o teto de tentativas o evento vai para a DLQ | [09:15]–[09:17] Diego/Larissa |
| PRD-RF-11 | Cada evento carrega um identificador único (`X-Event-Id`) que permite ao cliente deduplicar recebimentos repetidos | [09:25] Diego |
| PRD-RF-12 | O CRUD de configuração exige apenas usuário autenticado (qualquer role); apenas o replay de DLQ exige `ADMIN` | [09:36]–[09:37] Sofia/Marcos |

## 8. Requisitos não funcionais

| ID | Requisito | Origem |
|---|---|---|
| PRD-RNF-01 | Latência de notificação abaixo de 10 segundos (operacionalização da medição pendente — PRD-OBJ-01) | [09:02] Marcos |
| PRD-RNF-02 | Garantia de entrega at-least-once; duplicidades possíveis por design, deduplicação a cargo do consumidor | [09:24] Diego; [09:26] Larissa |
| PRD-RNF-03 | Consistência: evento registrado atomicamente com a mudança de status (mesma transação) | [09:40]–[09:41] Bruno/Diego |
| PRD-RNF-04 | Autenticidade e integridade: assinatura HMAC-SHA256 sobre o corpo, secret única por endpoint | [09:20]–[09:22] Sofia |
| PRD-RNF-05 | Transporte seguro: apenas URLs `https`; cadastro com `http` é recusado | [09:23] Sofia |
| PRD-RNF-06 | Payload máximo de 64 KB; acima disso o envio é tratado como erro, não truncado | [09:23] Sofia; [09:24] Diego/Larissa |
| PRD-RNF-07 | Timeout de 10 segundos por tentativa de entrega | [09:42] Diego |
| PRD-RNF-08 | Ordering por pedido (`order_id`) em melhor esforço: mantida com worker único e sem retentativas; não é garantia — retentativas em backoff podem entregar eventos do mesmo pedido fora de ordem (limitação registrada; mitigação por bloqueio de eventos posteriores do mesmo pedido é proposta em aberto no FDD/RFC) | [09:12]–[09:13] Diego/Larissa |
| PRD-RNF-09 | Auditoria: replay administrativo registra o usuário executor | [09:36] Sofia |
| PRD-RNF-10 | Disponibilidade da entrega independente da API: worker roda em processo separado | [09:11] Diego |

## 9. Decisões e trade-offs principais

Registradas em ADRs individuais; resumo de produto:

- **Outbox no MySQL** em vez de infraestrutura nova — consistência sem custo operacional extra ([ADR-001](adrs/ADR-001-outbox-no-mysql.md)).
- **Retry 1m/5m/30m/2h/12h + DLQ** — cobre ~15h de indisponibilidade; eventos além disso exigem replay manual ([ADR-002](adrs/ADR-002-retry-com-backoff-e-dlq.md)).
- **HMAC-SHA256 com secret por endpoint e rotação (24h de grace)** — contenção de vazamentos ao custo de gestão de ciclo de vida de secrets ([ADR-003](adrs/ADR-003-autenticacao-hmac-por-endpoint.md)).
- **At-least-once com `X-Event-Id`** — simplicidade de plataforma ao custo de exigir deduplicação do cliente, comunicada no portal de desenvolvedores ([ADR-004](adrs/ADR-004-entrega-at-least-once.md); [09:26] Marcos).
- **Worker separado em polling de 2s** — resiliência a deploys da API; latência mínima de 2s aceita ([ADR-005](adrs/ADR-005-worker-separado-com-polling.md)).
- **Reuso dos padrões do projeto** — velocidade e consistência ([ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)).

## 10. Dependências

- PRD-DEP-01 — Aplicação OMS existente: transação de `changeStatus` (`src/modules/orders/order.service.ts`), MySQL/Prisma, autenticação JWT e `requireRole` — a feature é construída inteiramente sobre eles.
- PRD-DEP-02 — Documentação para o cliente no portal de desenvolvedores (deduplicação por `X-Event-Id`, validação de assinatura, prazos de retry) — compromisso do PM ([09:26], [09:40] Marcos).
- PRD-DEP-03 — Revisão de segurança da Sofia (mínimo 2 dias úteis) antes do deploy, com foco em HMAC e geração de secret ([09:46] Sofia).
- PRD-DEP-04 — Confirmação de prazo com os clientes pelo PM ([09:47] Marcos).

## 11. Riscos e mitigação

Probabilidade e impacto abaixo são **avaliação deste documento** (não foram estimados pelos participantes da reunião).

| ID | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| PRD-RISCO-01 | Cliente não implementa deduplicação e processa eventos duplicados (duplicidade é possível por design no at-least-once, [09:24]) | Média | Médio | Documentação destacada no portal de desenvolvedores ([09:26] Marcos); `X-Event-Id` presente em toda entrega ([09:25]) |
| PRD-RISCO-02 | Acúmulo de eventos na outbox degrada a latência de entrega sob pico (preocupação levantada em [09:07] Bruno) | Baixa | Alto | Índices em status/created_at e leitura em batch pequeno ([09:08] Diego); monitoramento de backlog proposto no FDD; escala futura por particionamento ([09:13]) |
| PRD-RISCO-03 | Vazamento de secret pelo cliente (caso real já ocorrido, [09:22] Diego) | Média | Alto | Secret por endpoint em vez de global ([09:21] Sofia); rotação self-service com grace de 24h ([09:21]); HTTPS obrigatório ([09:23]) |
| PRD-RISCO-04 | Perda do prazo do trimestre e churn da Atlas ([09:00] Marcos) | Baixa | Alto | Estimativa de 3 sprints com revisão de segurança incluída ([09:46]–[09:47]); escopo enxuto com adiamentos explícitos (seção 6) |

## 12. Critérios de aceitação

- PRD-CA-01 — Cliente consegue cadastrar, listar, editar e remover webhooks pela API autenticada, e recebe a secret na criação.
- PRD-CA-02 — Cadastro com URL `http` é recusado com erro de validação.
- PRD-CA-03 — Mudança de status assinada por um webhook gera notificação para cada webhook interessado, entregue no endpoint cadastrado com assinatura verificável (duplicidades são possíveis, conforme a garantia at-least-once).
- PRD-CA-04 — Mudança de status **não** assinada por nenhum webhook do customer não gera notificação.
- PRD-CA-05 — Com o endpoint indisponível, o sistema retenta nos intervalos definidos e, esgotadas as tentativas, o evento fica visível para reprocessamento administrativo.
- PRD-CA-06 — Replay é negado a usuários sem role `ADMIN` e, quando executado, fica registrado quem o fez.
- PRD-CA-07 — Cliente consulta o histórico de entregas com resultado, resposta e tempo de cada tentativa.
- PRD-CA-08 — Após rotação de secret, entregas continuam validáveis pelo cliente durante as 24h de convivência.
- PRD-CA-09 — Em condições normais de operação, a notificação chega em menos de 10 segundos após a mudança de status (forma de medição a definir — PRD-OBJ-01).

## 13. Estratégia de testes e validação

- **Testes de integração** no padrão existente do repositório (Vitest + Supertest, `tests/orders.test.ts`, `tests/helpers/factories.ts`, `tests/setup.ts`): CRUD completo, validação de URL, geração/rotação de secret, autorização do replay (403 sem `ADMIN`) — **proposta de abordagem** alinhada ao código.
- **Testes da transação**: rollback do `changeStatus` não deixa evento na outbox; commit deixa (critério FDD-CA-01).
- **Testes do worker**: com endpoint fake (sucesso, falha 5xx, timeout), verificando marcação de status do evento, agenda de retry, DLQ e histórico de entregas.
- **Validação de assinatura**: recomputação do HMAC-SHA256 sobre o corpo recebido em teste ponta a ponta.
- **Validação com clientes-piloto**: os três clientes solicitantes como primeiros integradores, apoiados pela documentação do portal ([09:26], [09:40] Marcos).
- **Revisão de segurança** dedicada da Sofia antes do deploy ([09:46]).
