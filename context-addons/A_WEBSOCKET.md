# 🔌 Adendo A — WebSocket / Real-time (Socket.io, ws, SSE)

> Adendo do `AUDIT_CHECKLIST.md`. Aplique apenas se o projeto usa comunicação real-time (sockets, SSE).
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### A1. O handshake do socket exige autenticação (token válido)?

> **Opção 1 — Se não:**
> - Adicionar `io.use((socket, next) => { ... validar token ... })`
> - Token via `socket.handshake.auth.token` ou query string com HTTPS
> - Rejeitar conexão sem token válido
> - Anexar `userId` ao objeto socket para uso posterior
>
> **Opção 2 — Se sim:** ✅ Excelente

---

### A2. Cada event handler valida AUTORIZAÇÃO do `userId` sobre o recurso do payload?

> *Autenticar o socket UMA VEZ não é suficiente — cada evento deve revalidar acesso ao recurso específico que está sendo manipulado.*
>
> **Opção 1 — Se handlers confiam no payload do cliente:**
> - **CRÍTICO** — qualquer usuário autenticado pode manipular recursos de outros
> - Criar função `assertCanAccessResource(userId, resourceId)` no início de cada handler
> - Validar membership/ownership ANTES de qualquer operação no banco
> - Retornar erro via `socket.emit('error', ...)` se sem permissão
>
> **Opção 2 — Se cada handler valida autorização:** ✅ Excelente

---

### A2B. O `socket.userId` (ou equivalente) é GUARDADO no handshake e NUNCA reescrito por handlers subsequentes?

> *Erro comum: o handshake valida o JWT e seta `socket.userId = decoded.userId` ✅. Mas depois um handler como `board:join` aceita `user._id` do payload e faz `socket.userId = payload.user._id`. Resultado: a partir daí, todos os eventos do socket usam a identidade FALSIFICADA.*
>
> **Opção 1 — Se algum handler sobrescreve `socket.userId`:**
> - **CRÍTICO** — atacante autenticado consegue se passar por outros usuários
> - Auditar todos os pontos onde aparece `socket.userId =`, `(socket as any).userId =`, ou similar
> - Identidade do socket é **IMUTÁVEL após handshake** — só pode ser limpa no `disconnect`
> - Para dados adicionais do usuário (nome, avatar, role), buscar do banco usando o `userId` do handshake
> - Adicionar teste: "atacante emite `board:join` com `user._id` da vítima → seu socket continua identificado como atacante"
> - 🔗 *Relacionado: item de identidade em handlers async no `AUDIT_CHECKLIST.md` (mesma classe de bug).*
>
> **Opção 2 — Se identidade do socket é estabelecida apenas no handshake:** ✅ Excelente

---

### A3. Existe controle de quem entra em cada socket room?

> *Sem isso, `socket.to(roomId).emit(...)` é falso — qualquer um pode entrar em qualquer room.*
>
> **Opção 1 — Se cliente faz `socket.join(roomId)` livremente:**
> - Criar handler `socket.on('room:join', async (roomId) => { ... })`
> - Validar membership/ownership do `roomId` antes de `socket.join(roomId)`
> - Mesmo para `socket.leave()` — registrar/auditar
> - Em desconexão, limpar referências
>
> **Opção 2 — Se entrada em rooms é controlada:** ✅ Excelente

---

### A4. Payloads de socket events são validados com schema (Zod, Joi)?

> **Opção 1 — Se handlers recebem `payload: any` e usam direto:**
> - Criar schema Zod para cada event
> - Validar no início do handler com `safeParse`
> - Rejeitar e logar payloads inválidos
> - Particularmente importante para evitar NoSQL injection
>
> **Opção 2 — Se há validação de schema:** ✅ Excelente

---

### A5. Há rate limiting de eventos de socket usando o middleware oficial `socket.use()`?

> *Sockets podem emitir centenas de eventos por segundo se o cliente quiser. Implementações via wrapping de `socket.on` quase sempre ficam quebradas em casos sutis (não pegam `socket.onAny`, não pegam eventos com ACK, esquecem de limpar contadores no disconnect). O middleware oficial é a única forma confiável.*
>
> **Opção 1 — Se rate limit é por wrapping de `socket.on` ou inexistente:**
> - **Testar antes**: emitir 100 eventos/segundo do cliente e verificar se o servidor bloqueia. Se não bloqueia, a "proteção" atual é placebo.
> - Substituir por middleware oficial:
>   ```typescript
>   socket.use((packet, next) => {
>     const [event, ...args] = packet;
>     // contar eventos no janela; bloquear se exceder
>     if (excedeuLimite) return next(new Error('Rate limit exceeded'));
>     next();
>   });
>   ```
> - Contagem por `socket.id` E por `userId` — usuário pode abrir N sockets para multiplicar o limite
> - Limite razoável: 30-50 eventos/segundo (medir o legítimo)
> - Considerar limites diferentes por tipo de evento (read vs write; eventos de typing/cursor vs eventos de mutação)
> - Limpar contadores no `disconnect` para não vazar memória
>
> **Opção 2 — Se há `socket.use()` rate limit funcional e testado:** ✅ Excelente

---

### A6. CORS do Socket.io está configurado corretamente (allowlist, não wildcard)?

> **Opção 1 — Se `cors: { origin: '*' }` ou ausente:**
> - Configurar `origin` com allowlist (string array ou função)
> - `credentials: true` apenas se necessário
> - Métodos restritos a `['GET', 'POST']`
>
> **Opção 2 — Se CORS é estrito:** ✅ Excelente

---

### A7. Erros em handlers de socket são tratados sem vazar detalhes?

> **Opção 1 — Se handler propaga `error.stack` ou detalhes internos:**
> - Wrappear handlers em try/catch
> - Logar erro completo internamente
> - Emitir apenas mensagem genérica: `socket.emit('error', { message: 'Operação falhou', code: 'OP_FAILED' })`
>
> **Opção 2 — Se erros são tratados:** ✅ Excelente

---

## 🔗 Adendos relacionados

A maioria das classes de bug deste adendo aparece em outros adendos com forma diferente. Quando o projeto cruzar contextos, audite o conjunto, não cada um isolado.

- **A2, A2B (identidade do socket)** ↔ Se o app tem queues ou jobs, mesma classe vale lá → **item 4 do `AUDIT_CHECKLIST.md`** (identidade em handlers async).
- **A3 (controle de room)** ↔ É a versão "socket" da autorização granular a recurso. Mesma classe em outras superfícies:
  - **[B7 (download authz)](B_UPLOAD.md)** — versão "arquivo"
  - **[D3 (no cross-tenant)](D_MULTI_TENANT.md)** — versão "tabela/banco"
  - **[E4 (RAG tenant-aware)](E_LLM.md)** — versão "vector search"
  - Princípio universal: **item 2** do checklist (autorização em toda operação)
- **A5 (rate limit por socket)** ↔ Mesma classe de rate limit aparece em:
  - **[E6 (rate limit de LLM distribuído)](E_LLM.md)** — atenção a in-memory vs Redis
  - **[F1 (rate limit em APIs públicas)](F_APIS_PUBLICAS.md)**
  - Princípio universal: **item 35** (rate limit por usuário, não só IP) + **item 36** (IP confiável atrás de proxy)
- **A4 (validação de payload Zod)** ↔ **[I3](I_FILAS.md)** (consumer valida payload de fila) — mesma defesa em superfície diferente.
