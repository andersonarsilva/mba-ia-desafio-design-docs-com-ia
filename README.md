# Da Reunião ao Documento — Design Docs Gerados por IA

Entrega do desafio: pacote completo de design docs (PRD, RFC, FDD, ADRs e Tracker) para a feature **Sistema de Webhooks de Notificação de Pedidos** de um OMS, produzido a partir da transcrição de uma reunião técnica (`TRANSCRICAO.md`) e do código existente do repositório. Este README documenta o processo real de produção.

## Sobre o desafio

O desafio parte de um cenário realista: uma reunião técnica de ~55 minutos decidiu a arquitetura de um sistema de webhooks outbound para notificar clientes B2B sobre mudanças de status de pedidos, mas nada foi registrado além da transcrição da call. A tarefa é transformar essa transcrição — somada ao código real do OMS (Node.js/TypeScript, Express, Prisma/MySQL) — em um pacote de documentação acionável: PRD (produto), RFC (proposta arquitetural), FDD (especificação de implementação), ADRs (decisões isoladas) e um Tracker de rastreabilidade que liga cada item à sua origem.

A regra central é a não invenção: nenhum requisito, decisão ou restrição pode aparecer nos documentos sem origem identificável na transcrição ou no código. Igualmente importante é o que **não** entra: ideias explicitamente descartadas ou adiadas na reunião (email de alerta, rate limiting, dashboard, arquivamento da outbox) precisam ficar registradas como fora de escopo, não promovidas a requisito.

## Ferramentas de IA utilizadas

- **Claude Code (Anthropic)** — ferramenta principal de produção, operando como agente com acesso direto ao repositório. Fez a leitura integral da transcrição e do código-fonte, a extração e classificação de evidências, a redação de todos os documentos, as duas auditorias (semântica e mecânica, esta última com scripts Python descartáveis executados fora do repositório) e o fluxo Git (feature branch, commit e merge local). O trabalho foi dirigido por um prompt customizado versionado (v1 → v2, ver abaixo).
- **Codex (OpenAI)** — apoio na formatação do prompt customizado (evolução v1 → v2) e dupla análise: revisão cruzada do prompt e do resultado por um segundo modelo, para reduzir vieses de uma única ferramenta.

## Workflow adotado

1. **Preflight Git**: registro do baseline (`git status`, branch, commit) e criação da feature branch `feature/design-docs-webhooks` a partir de `dev`, seguindo GitFlow local (sem push).
2. **Contextualização**: leitura integral do enunciado e da `TRANSCRICAO.md`; leitura dirigida do código real (`order.service.ts` e a transação de `changeStatus`, módulo `products` como referência de CRUD, hierarquia de erros, middlewares, `env.ts`, `database.ts`, `schema.prisma`, testes). Montagem de um ledger de evidências com categoria (DECIDIDO, REQUISITO, CÓDIGO EXISTENTE, DESCARTADO/ADIADO, ABERTO, AMBÍGUO, DERIVADO/PROPOSTO), timestamp/falante ou caminho, e documento de destino. Todos os números foram registrados com unidade (2s, 10s, 64KB, 1m/5m/30m/2h/12h, 24h, 3 sprints).
3. **Produção na ordem**: ADRs → RFC → FDD → PRD → Tracker. Os IDs estáveis (`PRD-RF-01`, `RFC-ALT-01`, `FDD-CONTRATO-01`, `ADR-001`...) foram definidos antes da redação e usados identicamente nos documentos e no Tracker.
4. **Auditoria 1 (conteúdo/semântica)**: revisão item a item de decisão vs. proposta vs. questão aberta, itens descartados, números e aderência ao código.
5. **Auditoria 2 (mecânica)**: script Python validando todos os timestamps + falantes contra a transcrição, existência de todos os caminhos de código citados, resolução de todos os links Markdown relativos, contagem e seções dos 6 ADRs, unicidade de IDs e correspondência integral docs ↔ Tracker.
6. **README** (este arquivo), escrito somente após as auditorias, para relatar o processo real.
7. **Fechamento Git**: verificação final da checklist do enunciado, commit documental único na feature branch e merge local `--no-ff` em `dev`.

## Prompts customizados

A execução foi dirigida por um prompt autossuficiente versionado. A **v2** corrige e amplia a v1 — a mudança mais relevante foi incluir o README no fluxo, gerado apenas **depois** das auditorias reais, para que o relato do processo seja verdadeiro (a v1 não previa essa ordem e não foi preservada no working tree desta execução; os trechos abaixo são da v2).

Trecho 1 — política de evidência e não invenção:

```text
Não invente requisitos, decisões, restrições, metas quantitativas, fatos sobre
o sistema, falas, timestamps, participantes ou caminhos de arquivo.

Classifique as informações de trabalho nestas categorias:
- DECIDIDO: decisão explicitamente fechada na reunião;
- REQUISITO: necessidade funcional ou não funcional expressa na reunião;
- CÓDIGO EXISTENTE: comportamento ou padrão confirmado no repositório;
- DESCARTADO/ADIADO: ideia rejeitada ou postergada, com motivo;
- ABERTO: questão levantada e não resolvida;
- AMBÍGUO/CONFLITANTE: falas que admitem interpretações incompatíveis;
- DERIVADO/PROPOSTO: detalhe de design necessário para tornar o FDD
  acionável, mas não fechado literalmente na reunião.
```

Trecho 2 — tratamento explícito de ambiguidade (evitou que a IA "resolvesse" sozinha um conflito da reunião):

```text
Detecte contradições antes de redigir. Em especial, não resolva
silenciosamente a ambiguidade entre "5 tentativas" e os cinco intervalos de
retry 1m/5m/30m/2h/12h. Registre a interpretação proposta e mantenha a
semântica exata — tentativas totais versus retentativas após o primeiro
envio — como ponto pendente de confirmação.
```

Trecho 3 — auditorias obrigatórias antes do README:

```text
Execute no mínimo duas passagens distintas. Não apenas declare que auditou:
inspecione os arquivos, registre internamente os achados e corrija-os.
[...]
Auditoria 2 — validação mecânica e rastreabilidade: valide integralmente,
não por amostragem: todos os timestamps citados existem com o falante correto;
todos os caminhos citados existem; todos os links Markdown relativos resolvem;
existem exatamente 6 arquivos ADR-NNN-*.md com as seções MADR; IDs não estão
duplicados; cobertura do Tracker, percentual de TRANSCRICAO e mínimo de fontes
de código.
```

## Iterações e ajustes

Contagem real desta execução: **1 passagem de geração + 2 passagens de auditoria com correções + 1 rodada de dupla análise (Codex) com correções**, além da iteração prévia de prompt (v1 → v2). Ajustes concretos:

1. **Evolução do prompt v1 → v2**: a v2 passou a exigir que o README fosse gerado somente após as auditorias (relato verdadeiro do processo), adicionou a política de classificação de evidências, o fluxo Git obrigatório e a instrução de não resolver silenciosamente a ambiguidade das "5 tentativas".
2. **Correção semântica no PRD (Auditoria 1)**: o critério `PRD-CA-03` dizia que cada mudança de status gera "exatamente uma notificação" — contradizendo a garantia at-least-once decidida na reunião (duplicidades são possíveis por design). Reescrito.
3. **Rotulagem de proposta no FDD (Auditoria 1)**: o comportamento de assinatura durante o grace period de rotação de secret (assinar com a nova imediatamente) estava redigido como se decidido; a reunião só fechou que a secret antiga "fica válida por 24 horas". Rotulado explicitamente como proposta pendente de confirmação nos fluxos 4 e 9.
4. **Correções de citação (Auditoria 2)**: o script de validação detectou 5 citações com intervalo de timestamps em que o falante ficava pareado ao timestamp errado (ex.: `[09:24]–[09:26] Diego/...` quando Diego fala em [09:24]/[09:25], não em [09:26]). Todas reescritas com pareamento correto; a revalidação terminou sem erros.
5. **Normalização de IDs**: os critérios de aceitação e objetivos do PRD foram renomeados de `CA-*`/`OBJ-*` para `PRD-CA-*`/`PRD-OBJ-*` para garantir unicidade global entre documentos (o FDD tem `FDD-CA-*`); um ID órfão (`RFC-PROP-01`) citado sem definição foi removido do RFC.
6. **Correções da dupla análise (Codex)**: sete achados em duas rodadas de revisão cruzada por segundo modelo, todos confirmados e corrigidos — (a) o histórico de entregas não retornava o payload exigido em [09:34], resolvido via snapshot da outbox; (b) o FDD atribuía à lista `redact` existente do Pino uma proteção de secrets que ela não tem — a extensão com `*.secret`/`*.previousSecret` foi rotulada como proposta; (c) o critério FDD-CA-09 apresentava como fechada a política de assinatura durante o grace period, que é proposta pendente; (d) registrada a limitação (análise posterior) de que retries em backoff quebram a ordering por pedido mesmo em single-worker; (e) observabilidade e dependências ganharam IDs (`FDD-OBS-*`, `FDD-DEP-*`, `PRD-DEP-*`) e entraram no denominador do Tracker, que foi recalculado; (f) na segunda rodada, a ordering foi alinhada em todos os documentos como melhor esforço — o PRD ainda a declarava "garantida" — com o bloqueio de eventos posteriores do mesmo pedido registrado como mitigação proposta em aberto; (g) o resumo de FDD-CA-09 no Tracker foi atualizado para distinguir o decidido (antiga válida por 24h) da proposta (assinatura imediata com a nova).

## Como navegar a entrega

| Ordem | Arquivo | O que contém |
|---|---|---|
| 1 | [docs/PRD.md](docs/PRD.md) | O quê e por quê: problema, escopo, 12 requisitos funcionais, 10 não funcionais, fora de escopo, riscos, critérios de aceitação |
| 2 | [docs/RFC.md](docs/RFC.md) | Proposta arquitetural para revisão: TL;DR, alternativas descartadas, 6 questões em aberto, impacto e riscos |
| 3 | [docs/adrs/](docs/adrs/) | 6 ADRs: [outbox no MySQL](docs/adrs/ADR-001-outbox-no-mysql.md), [retry + DLQ](docs/adrs/ADR-002-retry-com-backoff-e-dlq.md), [HMAC por endpoint](docs/adrs/ADR-003-autenticacao-hmac-por-endpoint.md), [at-least-once](docs/adrs/ADR-004-entrega-at-least-once.md), [worker com polling](docs/adrs/ADR-005-worker-separado-com-polling.md), [reuso dos padrões](docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md) |
| 4 | [docs/FDD.md](docs/FDD.md) | Como construir: modelagem proposta, 9 fluxos, 8 contratos (com payloads), matriz de erros `WEBHOOK_*`, resiliência, observabilidade, integração com 10 caminhos reais do código |
| 5 | [docs/TRACKER.md](docs/TRACKER.md) | Rastreabilidade: 126 itens, 100% de cobertura, 107 linhas com origem na transcrição e 19 no código |
| — | [TRANSCRICAO.md](TRANSCRICAO.md) | Fonte primária (não alterada) |

Sugestão de leitura: PRD → RFC → ADRs → FDD, com o Tracker aberto ao lado para conferir a origem de qualquer item.
