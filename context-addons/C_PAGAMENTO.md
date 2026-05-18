# 💳 Adendo C — Pagamento (Stripe, MercadoPago, PagSeguro)

> Adendo do `AUDIT_CHECKLIST.md`. Aplique apenas se o projeto processa pagamentos.
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### C1. Valores monetários NUNCA vêm do cliente?

> **Opção 1 — Se o cliente envia `amount` no body:**
> - **CRÍTICO** — atacante paga R$ 0,01 pelo produto de R$ 1.000
> - Cliente envia apenas `productId` / `cartId`
> - Servidor calcula o valor a partir do banco
> - Verificar moeda também (não confiar no cliente)
>
> **Opção 2 — Se valores vêm do servidor:** ✅ Excelente

---

### C2. Webhooks de gateway de pagamento têm assinatura validada?

> **Opção 1 — Se aceita webhooks sem verificar:**
> - **CRÍTICO** — atacante envia webhooks falsos confirmando pagamentos
> - Validar `Stripe-Signature` / equivalente do gateway
> - Usar secret de webhook (diferente da API key)
> - Rejeitar webhooks sem assinatura válida
>
> **Opção 2 — Se webhooks são verificados:** ✅ Excelente

---

### C3. Endpoints sensíveis usam idempotency keys?

> **Opção 1 — Se cliente pode disparar a mesma cobrança duas vezes:**
> - Aceitar header `Idempotency-Key` do cliente
> - Cachear resultado por ID por X horas
> - Se mesma key vier de novo, retornar mesma resposta sem reprocessar
> - Crítico para evitar cobranças duplicadas em retry
>
> **Opção 2 — Se há idempotência:** ✅ Excelente

---

### C4. Dados de cartão NUNCA passam pelo seu servidor?

> **Opção 1 — Se você recebe número de cartão no backend:**
> - **CRÍTICO — PCI-DSS** entra em cena (escopo enorme de compliance)
> - Usar tokenização do gateway (Stripe Elements, Stripe.js, etc.)
> - Cliente envia diretamente ao gateway, recebe token, envia só o token para você
>
> **Opção 2 — Se usa tokenização:** ✅ Excelente

---

### C5. Histórico de transações é imutável (não pode ser editado pela aplicação)?

> **Opção 1 — Se transações são editáveis no banco:**
> - Tabela append-only (não permitir UPDATE)
> - Estornos/cancelamentos como NOVAS entradas, não edição
> - Audit log obrigatório
>
> **Opção 2 — Se histórico é imutável:** ✅ Excelente
