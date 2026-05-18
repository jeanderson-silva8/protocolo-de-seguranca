# 📨 Adendo I — Filas / Workers / Mensageria (BullMQ, RabbitMQ, Kafka, SQS, Redis Streams)

> Adendo do `AUDIT_CHECKLIST.md`. Aplique se o projeto usa fila de mensagens ou workers assíncronos.
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### I1. Consumers são idempotentes?

> *Toda fila pode entregar a mesma mensagem duas vezes (at-least-once delivery é o padrão). Se seu consumer não for idempotente, retries duplicam efeitos colaterais — cobranças duplicadas, emails repetidos, contadores errados.*
>
> **Opção 1 — Se processar a mesma mensagem duas vezes causa dano:**
> - Identificar cada mensagem com `jobId` / `messageId` único e estável
> - Tabela `processed_jobs` (ou cache Redis com TTL) registrando IDs já processados
> - No início do consumer, verificar se já foi processado → ignorar
> - Para operações no banco: usar `INSERT ... ON CONFLICT` / upserts em vez de inserts cegos
> - Operações externas (Stripe, email): usar `Idempotency-Key` do próprio job
>
> **Opção 2 — Se consumers são idempotentes:** ✅ Excelente

---

### I2. Existe Dead Letter Queue (DLQ) para mensagens que falham repetidamente?

> *Sem DLQ, mensagem com bug envenena a fila: tenta, falha, volta, tenta de novo, infinitamente — consumindo workers e bloqueando mensagens legítimas atrás dela (poison-pill).*
>
> **Opção 1 — Se mensagens com erro ficam em retry infinito ou são silenciosamente descartadas:**
> - Configurar limite de tentativas (ex: 5 tentativas com backoff exponencial)
> - Após esgotar, mover para DLQ (fila separada)
> - **Alerta** quando algo cai na DLQ — não pode ser silencioso
> - Dashboard para inspecionar conteúdo da DLQ
> - Procedimento documentado de reprocessamento manual após fix
>
> **Opção 2 — Se há DLQ + alerta + procedimento:** ✅ Excelente

---

### I3. Payloads de mensagens são validados pelo consumer (não confiando no producer)?

> *Mesmo que o producer seja "interno", esquemas evoluem, versões antigas convivem, e tipos podem mudar. Consumer que confia cego quebra em produção.*
>
> **Opção 1 — Se consumer usa `JSON.parse` direto e acessa campos sem validar:**
> - Schema Zod/Pydantic para cada tipo de mensagem
> - Validar no início do handler; mensagem inválida vai para DLQ com motivo
> - Versionar o schema da mensagem (`version: 1`, `version: 2`) para evolução compatível
>
> **Opção 2 — Se há validação:** ✅ Excelente

---

### I4. Acesso ao broker (Redis, RabbitMQ, Kafka) está restrito (auth + rede privada)?

> **Opção 1 — Se o broker está exposto publicamente ou sem auth:**
> - **CRÍTICO** — Redis sem senha em IP público é incidente clássico
> - Auth obrigatória (senha forte ou TLS mutual auth)
> - VPC / rede privada — só workers/API acessam
> - TLS em trânsito se cruza rede pública
> - Princípio do menor privilégio: producer só publica, consumer só consome
>
> **Opção 2 — Se o broker está protegido:** ✅ Excelente

---

### I5. Jobs sensíveis NÃO carregam dados sensíveis no payload?

> *Payloads de fila são frequentemente persistidos (Redis dump, Kafka log) e replicados — local errado para PII, tokens ou segredos.*
>
> **Opção 1 — Se jobs contêm senhas, tokens, números de cartão, PII completa:**
> - Passar apenas IDs no payload; consumer busca dados frescos do banco
> - Se segredo é inevitável no payload, criptografar com chave do KMS
> - Reduz blast radius se o broker for comprometido ou dumps vazarem
>
> **Opção 2 — Se payloads são "magros":** ✅ Excelente

---

### I6. Tempo de execução do job tem timeout?

> **Opção 1 — Se um job travado pode rodar para sempre:**
> - Timeout por job (ex: 30s default, configurável por tipo)
> - Após timeout: matar, marcar como failed, retry com backoff
> - Métrica de duração por tipo de job para detectar regressões
>
> **Opção 2 — Se há timeout:** ✅ Excelente
