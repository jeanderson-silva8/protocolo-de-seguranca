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

---

## 🔗 Adendos relacionados

- **C2 (validação de assinatura de webhook recebido)** ↔ **[F3 (assinar webhooks que você envia)](F_APIS_PUBLICAS.md)** — duas pontas da mesma defesa HMAC.
- **C3 (idempotency keys)** ↔ Mesma classe em outras superfícies:
  - **[I1 (consumer idempotente)](I_FILAS.md)** — fila pode entregar mensagem 2×
  - **[F4 (retry de webhook)](F_APIS_PUBLICAS.md)** — provider remoto pode reenviar
  - Princípio universal: **item 23** do `AUDIT_CHECKLIST.md` (race conditions / TOCTOU) e **item 28** (operações de cota atômicas).
- **C5 (histórico imutável)** ↔ Princípio universal: **item 47** (audit log de ações sensíveis).
- **C1 (valor vem do servidor, nunca do cliente)** ↔ Princípio universal: **item 3** (IDs/valores sensíveis vêm da sessão, não do payload).

> ⚠️ **Adendo intencionalmente conciso.** Pagamento real tem muito mais armadilhas (3D Secure, SCA europeia, conciliação, chargebacks, fraude/velocity checks, dunning management). Essas vão entrar quando uma auditoria em projeto de pagamento real ditar — não força agora pra evitar conteúdo sem rastro de origem. Ver `WORKFLOW.md` (regra: nunca adicionar item sem caso real).
