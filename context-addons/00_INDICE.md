# 🎯 Adendos de Auditoria por Contexto — Índice

> Adendos do `AUDIT_CHECKLIST.md`. Use o checklist universal primeiro, depois rode apenas os adendos que se aplicam ao seu projeto.
>
> Se o projeto tem WebSocket + upload + multi-tenant, rode os três adendos correspondentes. Se for SaaS B2B com GraphQL e IA, rode D + J + E.

---

## 🗺️ Mapa rápido: o que rodar conforme o stack

| Se o projeto tem... | Rode o adendo... |
|---|---|
| Socket.io, ws, SSE, real-time | [A — WebSocket](A_WEBSOCKET.md) |
| Upload de arquivos pelo usuário | [B — Upload](B_UPLOAD.md) |
| Stripe, MercadoPago, PagSeguro, cobrança recorrente | [C — Pagamento](C_PAGAMENTO.md) |
| Múltiplos tenants/organizações com isolamento | [D — Multi-tenant](D_MULTI_TENANT.md) |
| OpenAI, Anthropic, Groq, LLM próprio, RAG | [E — LLM](E_LLM.md) |
| API pública (com ou sem API key), webhooks pra fora | [F — APIs públicas](F_APIS_PUBLICAS.md) |
| App React Native, Flutter, iOS/Android nativo | [G — Mobile](G_MOBILE.md) |
| PII real, dados de saúde, financeiros, jurídicos | [H — Dados sensíveis](H_DADOS_SENSIVEIS.md) |
| BullMQ, RabbitMQ, Kafka, SQS, Redis Streams, Celery | [I — Filas](I_FILAS.md) |
| Apollo, Graphene, Yoga, Mercurius, Strawberry | [J — GraphQL](J_GRAPHQL.md) |

---

## 📋 Os 10 adendos disponíveis

| Adendo | Itens | Arquivo |
|--------|-------|---------|
| 🔌 **A — WebSocket / Real-time** | 8 (A1, A2, A2B, A3-A7) | [A_WEBSOCKET.md](A_WEBSOCKET.md) |
| 📁 **B — Upload de arquivos** | 7 (B1-B7) | [B_UPLOAD.md](B_UPLOAD.md) |
| 💳 **C — Pagamento** | 5 (C1-C5) | [C_PAGAMENTO.md](C_PAGAMENTO.md) |
| 🏢 **D — Multi-tenant / SaaS B2B** | 6 (D1-D6) | [D_MULTI_TENANT.md](D_MULTI_TENANT.md) |
| 🤖 **E — Aplicações com IA / LLM** | 6 (E1-E6) | [E_LLM.md](E_LLM.md) |
| 📡 **F — APIs públicas / Webhooks** | 4 (F1-F4) | [F_APIS_PUBLICAS.md](F_APIS_PUBLICAS.md) |
| 📱 **G — Mobile** | 3 (G1-G3) | [G_MOBILE.md](G_MOBILE.md) |
| 🧪 **H — Dados sensíveis (saúde, financeiro, jurídico)** | 4 (H1-H4) | [H_DADOS_SENSIVEIS.md](H_DADOS_SENSIVEIS.md) |
| 📨 **I — Filas / Workers / Mensageria** | 6 (I1-I6) | [I_FILAS.md](I_FILAS.md) |
| 🔍 **J — GraphQL** | 6 (J1-J6) | [J_GRAPHQL.md](J_GRAPHQL.md) |

**Total:** 55 perguntas distribuídas em 10 adendos. Nenhum projeto usa todos — a média é 2-4 adendos aplicáveis por projeto.

---

## 🎯 Como usar

1. **Identifique os contextos** do seu projeto pelo mapa acima.
2. **Termine antes o `AUDIT_CHECKLIST.md`** (universal — Bloqueadores → Essenciais → Qualidade → Diferenciação).
3. **Rode apenas os adendos aplicáveis.** Não force adendos que não fazem sentido (ex: não rodar `B — Upload` num projeto que só serve métricas read-only).
4. **Documente o que escolheu NÃO fazer e por quê** — em ADR. Isso é sinal de senioridade tão forte quanto implementar a defesa.
5. **Atualize estes adendos** sempre que uma auditoria encontrar bug que não estava coberto. O ciclo "checklist → audita projeto → bug novo vira pergunta nova" é o que mantém o framework vivo.

---

## 📚 Contextos que podem ser adicionados no futuro

À medida que novos tipos de projeto forem auditados, considere criar adendos para:

- gRPC
- IoT / dispositivos embarcados
- Blockchain / contratos inteligentes
- Streaming (vídeo, áudio)
- Geoespacial
- Compliance específico (HIPAA, PCI-DSS, SOC 2)

Adicione um novo `K_NOMEDOCONTEXTO.md` na pasta + atualize este índice quando o caso real aparecer.
