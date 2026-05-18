# 🔍 Adendo J — GraphQL

> Adendo do `AUDIT_CHECKLIST.md`. Aplique se o projeto expõe API GraphQL (Apollo, Graphene, Yoga, Mercurius, Strawberry).
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### J1. Introspecção está desabilitada em produção?

> *Introspecção expõe o schema completo — todos os tipos, campos, queries, mutations. Em prod, dá ao atacante o mapa do tesouro.*
>
> **Opção 1 — Se introspecção está ativa em prod:**
> - Desabilitar via config do servidor (Apollo: `introspection: false` quando `NODE_ENV === 'production'`)
> - Manter ativa em dev/staging para tooling
> - GraphQL Playground / Apollo Sandbox: desabilitar em prod também
>
> **Opção 2 — Se introspecção é só em dev:** ✅ Excelente

---

### J2. Há limite de profundidade (depth limit) nas queries?

> *Query maliciosa `user { friends { friends { friends { ... } } } }` aninhada N níveis explode o servidor.*
>
> **Opção 1 — Se não há depth limit:**
> - Usar `graphql-depth-limit` ou validation rule equivalente
> - Limite razoável: 5-7 níveis (mensurar o legítimo)
> - Resposta clara quando excede: `Query depth N exceeds maximum of M`
>
> **Opção 2 — Se há depth limit:** ✅ Excelente

---

### J3. Há limite de complexidade de query (query cost analysis)?

> *Depth limit não basta — query pode ser rasa mas pedir milhões de registros: `users(first: 1000000) { posts(first: 1000) }`.*
>
> **Opção 1 — Se não há análise de custo:**
> - Atribuir custo a cada campo/lista (`graphql-query-complexity`, `graphql-cost-analysis`)
> - Definir budget máximo por query (ex: 1000 pontos)
> - Considerar custo diferenciado por usuário (free vs paid tier)
> - Rejeitar antes de executar
>
> **Opção 2 — Se há análise de complexidade:** ✅ Excelente

---

### J4. Há proteção contra batching/aliasing attacks?

> *GraphQL permite múltiplas operações na mesma request (batching) e renomeação de campos (aliasing). Atacante usa para bypassar rate limit: 1000 tentativas de login em 1 request HTTP.*
>
> **Opção 1 — Se rate limit é por request HTTP:**
> - Limitar número de operações por request (`graphql-no-batch` ou config do server)
> - Limitar número de aliases por campo sensível (mesma mutation `login` repetida com aliases conta como N tentativas)
> - Rate limit no nível do resolver para campos sensíveis (login, password reset)
> - **Sobre batching no Apollo Server (esclarecimento)**: Apollo Server 4 **não** faz HTTP batching por padrão. Ele só aceita batches se `allowBatchedHttpRequests: true` estiver explicitamente ligado. Auditoria correta = "verificar que essa flag não foi habilitada sem necessidade", não "desabilitar". Outros servers GraphQL (Yoga, Mercurius) têm defaults próprios — checar o seu.
>
> **Opção 2 — Se há proteção contra batching/aliasing:** ✅ Excelente

---

### J5. Autorização é verificada em CADA resolver (não só no entry point)?

> *REST: uma rota = uma checagem. GraphQL: cada campo pode resolver dados independentes. Checar só o login não basta — campos aninhados precisam validar autorização individual.*
>
> **Opção 1 — Se autorização é só no nível da query raiz:**
> - **RISCO ALTO** — `user(id: X) { secretField }` pode vazar se autorização não estiver no resolver de `secretField`
> - Implementar autorização por campo (field-level): graphql-shield, directives `@auth`, ou checagem manual nos resolvers
> - Considerar DataLoader + checagem em batch para não matar performance
> - Auditar especialmente: relações entre objetos, campos administrativos, campos derivados
>
> **Opção 2 — Se autorização é por resolver/campo:** ✅ Excelente

---

### J6. Erros do GraphQL não vazam informação sensível?

> *Por padrão, muitos servers GraphQL retornam stack traces completos no campo `errors` da resposta.*
>
> **Opção 1 — Se erros expõem stack/internals em prod:**
> - Customizar `formatError` (Apollo) para sanitizar
> - Em prod: mensagem genérica + código + correlation ID
> - Em dev: erro completo
> - Cuidado especial com erros do banco que podem vazar nome de tabela/coluna
>
> **Opção 2 — Se erros são sanitizados em prod:** ✅ Excelente

---

## 🔗 Adendos relacionados

- **J5 (autorização por resolver)** ↔ É a versão "GraphQL" da autorização granular a recurso. Mesma classe em outras superfícies:
  - **[A3 (room control)](A_WEBSOCKET.md)**, **[B7 (download authz)](B_UPLOAD.md)**, **[D3 (no cross-tenant)](D_MULTI_TENANT.md)**, **[E4 (RAG tenant-aware)](E_LLM.md)**
  - Princípio universal: **item 2** do `AUDIT_CHECKLIST.md` (autorização em toda operação) e **item 4** (identidade em handlers async).
- **J3 (query complexity)** ↔ Versão GraphQL do rate limit por custo. Princípio universal: **item 35** (rate limit por usuário, não só IP) e **item 45** (métricas pra detectar custo anômalo).
- **J4 (batching/aliasing attacks)** ↔ Mesma classe "atacante faz N operações em 1 request" aparece em:
  - **[A5 (rate limit de socket event)](A_WEBSOCKET.md)** — alguns clientes emitem rajada num só pacote.
  - Princípio universal: **item 12** (brute force protection) — aliasing é o vetor moderno de bypass de rate limit.
- **J1 (introspecção off em prod)** ↔ Princípio universal: **item 40** (erros sem vazar internals em prod) — schema é um tipo de "internal" que vaza.
- **Se o GraphQL é multi-tenant**, J5 + **[D2 (filtro automático no ORM)](D_MULTI_TENANT.md)** precisam trabalhar juntos: cada resolver field-level + cada query escopada por tenant.
