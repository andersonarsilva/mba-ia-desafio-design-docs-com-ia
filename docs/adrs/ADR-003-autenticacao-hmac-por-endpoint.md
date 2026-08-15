# ADR-003 — Assinatura HMAC-SHA256 com secret por endpoint e rotação com grace period

- **Status:** Aceito
- **Data de registro:** 2026-08-15 (data de elaboração deste registro)
- **Decisores:** Sofia (Eng. de Segurança), Larissa (Tech Lead), Diego (Eng. Sênior), Bruno (Eng. Pleno)

## Contexto

Os webhooks expõem eventos com dados de pedidos para endpoints fora da infraestrutura da empresa. O cliente precisa validar que a requisição veio realmente da plataforma e que o payload não foi adulterado no caminho ([09:19] Sofia). Já houve caso de cliente que vazou secret em log da própria aplicação ([09:22] Diego).

## Decisão

Fechado em [09:22] Sofia: "Decidido: HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h."

- **Assinatura HMAC-SHA256 sobre o corpo do request**, enviada no header `X-Signature` ([09:20] Sofia). O algoritmo é o padrão de mercado com bibliotecas amplamente disponíveis ([09:20] Sofia).
- **Secret única por endpoint de webhook**, não uma secret global da plataforma: "se vaza uma, vaza tudo" ([09:21] Sofia). A configuração do webhook armazena `url + secret + customer_id + estado ativo` ([09:21] Bruno/Sofia).
- **Secret rotacionável via API**: ao rotacionar, a secret antiga permanece válida por **24 horas** em paralelo, para o cliente migrar seus sistemas; depois disso, a antiga expira ([09:21] Sofia).
- **TLS obrigatório**: a URL do webhook deve ser `https`; cadastro com `http` é recusado com erro de validação no schema Zod ([09:23] Sofia — registrado pela própria Sofia como validação, não como decisão arquitetural separada).

Observação de escopo: a reunião fechou o HMAC **sobre o corpo**. O header `X-Timestamp` é enviado para o cliente poder detectar replay se quiser ([09:44] Diego), mas **não** foi decidido que ele participa da assinatura; proteção completa contra replay fica registrada como risco/questão em aberto no RFC e no FDD.

## Alternativas consideradas

1. **Secret global da plataforma** (discutida na reunião) — rejeitada: o vazamento de uma secret comprometeria todos os clientes ([09:21] Sofia).
2. **Enviar sem assinatura, confiando apenas em TLS** (análise posterior, não discutida na reunião) — rejeitada nesta análise: TLS protege o canal, mas não permite ao cliente autenticar a origem nem detectar adulteração após o término do túnel; contraria o requisito explícito de [09:19].

## Consequências positivas

- Cliente consegue autenticar origem e integridade do payload com ferramentas padrão ([09:20]).
- Comprometimento de uma secret fica contido a um único endpoint ([09:21]).
- Rotação com grace period de 24h permite migração sem janela de indisponibilidade ([09:21]).

## Consequências negativas e trade-offs

- Duas secrets podem ser simultaneamente válidas durante o grace period, exigindo verificação dupla no lado do cliente e controle de expiração no nosso lado.
- Gestão de ciclo de vida de secrets (geração, entrega na criação, rotação, expiração) adiciona complexidade ao módulo.
- Sem inclusão do timestamp na assinatura, a mitigação de replay depende do cliente ([09:44]; risco registrado no RFC).

## Origem

- `TRANSCRICAO.md`: [09:19]–[09:23] Sofia/Bruno/Diego, [09:44] Diego/Sofia.
- Código: `src/middlewares/validate.middleware.ts` e schemas Zod por módulo (ex.: `src/modules/products/product.schemas.ts`) como padrão para a validação de URL `https`; revisão de segurança agendada com Sofia antes do deploy ([09:46]).
