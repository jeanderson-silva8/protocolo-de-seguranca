# 📡 Adendo F — APIs públicas / Webhooks

> Adendo do `AUDIT_CHECKLIST.md`. Aplique se o projeto expõe API pública (com ou sem API key) ou envia webhooks pra terceiros.
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### F1. Endpoints públicos têm rate limiting agressivo (por IP e/ou por API key)?

> **Opção 1 — Se sem rate limit:**
> - Rate limit estrito por IP
> - Se há API key, rate limit por key também
> - Considerar diferenciação por tier (free vs paid)
>
> **Opção 2 — Se há rate limit em camadas:** ✅ Excelente

---

### F2. Há versionamento da API (v1, v2)?

> **Opção 1 — Se endpoints não são versionados:**
> - Adicionar prefixo `/api/v1/...`
> - Documentar política de deprecação
> - Permite evoluir sem quebrar clientes
>
> **Opção 2 — Se há versionamento:** ✅ Excelente

---

### F3. Webhooks que você envia para terceiros são assinados?

> **Opção 1 — Se webhooks saem sem assinatura:**
> - Calcular HMAC-SHA256 do payload com secret do receptor
> - Enviar no header (ex: `X-Signature`)
> - Documentar para que receptores validem
>
> **Opção 2 — Se webhooks são assinados:** ✅ Excelente

---

### F4. Há retry com backoff exponencial para webhooks que falham?

> **Opção 1 — Se webhook que falha é perdido:**
> - Fila de retry (BullMQ, SQS, etc.)
> - Backoff exponencial com jitter
> - Limite de tentativas (ex: 5 retries em 24h)
> - Dead-letter queue para investigação
>
> **Opção 2 — Se há retry resiliente:** ✅ Excelente

---

## 🔗 Adendos relacionados

- **F1 (rate limit em APIs públicas)** ↔ Mesma armadilha de rate limit em outras superfícies:
  - **[A5 (rate limit de socket)](A_WEBSOCKET.md)**, **[E6 (rate limit de LLM distribuído)](E_LLM.md)** — atenção a in-memory vs Redis quando escalar.
  - Princípio universal: **item 35** do `AUDIT_CHECKLIST.md` (rate limit por usuário, não só IP) + **item 36** (IP confiável atrás de proxy — sem isso o rate limit por IP é teatro).
- **F3 (assinar webhooks que você envia)** ↔ **[C2 (validar assinatura de webhook recebido)](C_PAGAMENTO.md)** — duas pontas da mesma defesa HMAC.
- **F4 (retry com backoff)** ↔ Mesma classe de "operação que pode falhar e precisa retry" aparece em:
  - **[I1 (consumer idempotente)](I_FILAS.md)** + **[I2 (DLQ)](I_FILAS.md)**
  - **[C3 (idempotency keys de pagamento)](C_PAGAMENTO.md)**
- **F2 (versionamento de API)** ↔ Princípio universal: **item 30** (README honesto sobre o que está e o que não está estável).
