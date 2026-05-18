# 🏢 Adendo D — Multi-tenant / SaaS B2B

> Adendo do `AUDIT_CHECKLIST.md`. Aplique apenas se o projeto serve múltiplos tenants (empresas/organizações) com isolamento de dados.
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### D1. Todo dado tem `tenantId`/`organizationId` no schema?

> **Opção 1 — Se há tabelas sem identificador de tenant:**
> - Adicionar campo `tenantId` em todas as entidades
> - Index composto em `(tenantId, id)` para queries
> - Backfill dos dados existentes
>
> **Opção 2 — Se todo schema tem tenantId:** ✅ Excelente

---

### D2. Toda query filtra por `tenantId` automaticamente (não confia no programador lembrar)?

> **Opção 1 — Se filtro de tenant depende do desenvolvedor lembrar em cada query:**
> - **RISCO ALTO** — uma query esquecida vaza dados entre tenants
> - Implementar middleware/scope no ORM: queries automaticamente filtram por tenant da sessão
> - Mongoose: plugin global; Prisma: extension; SQLAlchemy: query mixin
> - Forçar explicitamente "cross-tenant" quando legítimo (jobs administrativos)
>
> **Opção 2 — Se filtro é automático:** ✅ Excelente

---

### D3. Não há jeito de o usuário acessar/listar dados de outro tenant via API?

> *Testar manualmente: usuário do tenant A consegue, alterando `tenantId` no body ou URL, acessar tenant B?*
>
> **Opção 1 — Se há vazamento entre tenants:**
> - **CRÍTICO** — quebra fundamental de isolamento
> - `tenantId` vem SEMPRE do token, NUNCA do cliente
> - Ignorar `tenantId` em payloads de entrada
>
> **Opção 2 — Se isolamento é total:** ✅ Excelente

---

### D4. Roles/permissões são verificadas no backend (não só no frontend)?

> *Frontend escondendo botão de admin não é segurança — é UX.*
>
> **Opção 1 — Se autorização baseada em role é só no frontend:**
> - Middleware no backend que valida role do usuário para cada operação privilegiada
> - Decoradores/helpers: `requireRole('admin')`, `requirePermission('billing:write')`
> - Considerar RBAC ou ABAC se permissões forem complexas
>
> **Opção 2 — Se autorização é no backend:** ✅ Excelente

---

### D5. Convites para tenant validam que convidador tem permissão de convidar?

> **Opção 1 — Se qualquer membro pode convidar qualquer um:**
> - Definir quem pode convidar (admin/owner/member)
> - Validar role do convidador antes de criar convite
> - Audit log do convite (quem convidou quem, quando, com qual role)
>
> **Opção 2 — Se convites são autorizados:** ✅ Excelente

---

### D6. Quando usuário sai do tenant, acesso é revogado imediatamente?

> **Opção 1 — Se remoção do membro não invalida sessão ativa:**
> - Token JWT pode continuar válido por minutos/horas após remoção
> - Adicionar verificação de membership em cada request (cache curto)
> - Ou invalidar refresh tokens do usuário ao removê-lo
>
> **Opção 2 — Se acesso é revogado em tempo real:** ✅ Excelente

---

## 🔗 Adendos relacionados

- **D3 (no cross-tenant)** ↔ É a versão "banco" da autorização granular a recurso. Mesma classe em outras superfícies:
  - **[A3 (room control em socket)](A_WEBSOCKET.md)**
  - **[B7 (download authz)](B_UPLOAD.md)**
  - **[E4 (RAG tenant-aware)](E_LLM.md)** — especialmente importante: vector search vaza chunks de docs de outros tenants se não filtrar
  - **[J5 (authz por resolver GraphQL)](J_GRAPHQL.md)** — campo aninhado pode vazar mesmo com auth na query raiz
  - Princípio universal: **item 2** do `AUDIT_CHECKLIST.md` (autorização em toda operação) + **item 3** (IDs sensíveis vêm da sessão, não do payload).
- **D2 (filtro automático por tenant no ORM)** ↔ **[E4](E_LLM.md)** — vector search também precisa de filtro automático.
- **D6 (revogação ao sair)** ↔ Princípio universal: **item 10** (JWT com estratégia de revogação) e **item 47** (audit log da remoção).
