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
