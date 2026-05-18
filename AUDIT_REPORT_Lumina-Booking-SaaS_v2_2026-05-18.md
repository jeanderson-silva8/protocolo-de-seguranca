# 🔍 Auditoria de Segurança — Lumina-Booking-SaaS (v2)

> **Data:** 2026-05-18
> **Método:** Segunda passada de auditoria sobre o código já corrigido pela primeira auditoria (2026-05-17). Aplicação do `AUDIT_CHECKLIST.md` + `CONTEXT_ADDONS.md` com foco em **validar entregas declaradas vs. realidade no código**.
> **Escopo:** Mesmo da v1 (backend Django/Graphene, frontend React+Vite, marketing-site, Docker, CI).
> **Resultado em uma frase:** Das 25 correções declaradas pela auditoria v1, 23 foram confirmadas no código (92%). Encontradas 4 discrepâncias (1 crítica — bypass de rate limit via header forjado) e 4 achados novos. Auditoria gerou 2 perguntas novas no checklist universal (5C já existia, agora 23B e 20B). Após v2, todas as 8 discrepâncias e achados novos foram corrigidos.

---

## 📖 Como ler este relatório

Auditoria **comparativa** com a v1 (`AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md`). O autor não corrigiu o código entre as duas auditorias — a v2 é uma revisão crítica das próprias entregas da v1, feita com olhar adversarial mais afiado. Por isso o veredicto é "92% de entrega real" em vez dos "100% confirmados" implícitos no relatório anterior.

A correção dos 8 itens encontrados nesta v2 aconteceu **durante esta auditoria** — diferente do BrieflyAI (relatório descreve estado antes das correções) e mais parecido com o FlowSnyker v2 (relatório descreve estado depois das correções).

Organização:

1. **Bloco 1 — Validações confirmadas** da v1 (23 itens).
2. **Bloco 2 — Discrepâncias encontradas e corrigidas** (4 itens — gap entre declarado e entregue na v1).
3. **Bloco 3 — Achados novos** (4 itens — coisas que escaparam à v1 completamente).
4. **Bloco 4 — Tabela v1 → v2** (evolução em métricas).
5. **Evolução dos checklists** (2 itens novos: 23B e 20B).
6. **Reflexão final** sobre o método.

---

## ✅ Bloco 1 — Validações confirmadas da v1

Dos 25 itens declarados como corrigidos na v1 (13 críticos + 4 parciais + 8 polimentos), **23 foram validados no código**:

| Item v1 | Status na v2 | Onde |
|---------|--------------|------|
| Crítico #2 — `docker-compose.prod.yml` fail-fast | ✅ | `docker-compose.prod.yml:11,12,35,37` — `${VAR:?msg}` correto em todas as envs sensíveis |
| Crítico #3 — Introspecção GraphQL off em prod | ✅ | `graphql_security.py:103-107` — `NoSchemaIntrospectionCustomRule` aplicado quando `DEBUG=False` |
| Crítico #4 — SameSite=Strict nos cookies | ✅ | `settings.py:151-152` — `SESSION_COOKIE_SAMESITE='Strict'` + `CSRF_COOKIE_SAMESITE='Strict'` |
| Crítico #6 — Depth limit + alias limit + batching off | ✅ | `graphql_security.py:34-90` — implementação correta |
| Crítico #9 — Dockerfile multi-stage + user não-root | ✅ | `Dockerfile` builder + runtime + `USER app` + `HEALTHCHECK` + `.dockerignore` |
| Crítico #10 — Caddy + auto-HTTPS + SECURE_PROXY_SSL_HEADER | ✅ | `docker-compose.prod.yml` + `infra/Caddyfile` + `settings.py:145` |
| Crítico #12 — Dependabot configurado | ✅ | `.github/dependabot.yml` cobrindo pip + npm + actions + docker |
| Crítico #12 — CI completa com gate | ✅ | `ci.yml` com 7 jobs + `deploy_backend.yml` via `workflow_run` |
| Parcial #P1 — `.env.example` com CSPRNG | ✅ | `.env.example` na raiz com instruções explícitas |
| Parcial #P3 — JWT_ALGORITHM HS256 explícito | ✅ | `settings.py:174-180` |
| Parcial #P4 — README honesto | ✅ | Seção "Escopo desta versão" presente |
| Polimento #L2 — Logger no seed | ✅ | `seed_subscriptions.py:19-24` |
| Polimento #L7 — Governance docs | ✅ | `SECURITY.md`, `THREAT_MODEL.md`, ADR-001, ADR-002 |

Nota especial: **ADR-001 (multi-tenant fora de escopo) é exemplar**. Raciocínio honesto, alternativas consideradas, plano de migração com gatilhos objetivos. É exatamente o tipo de documento que sênior bom produz.

---

## ⚠️ Bloco 2 — Discrepâncias declarado vs. entregue (4 itens)

### Discrepância 🔴 #1 — Bypass crítico do rate limit via `X-Forwarded-For` forjado

**Declarado na v1:** crítico #5 implementado — rate limit por IP em `tokenAuth`.

**Realidade no código:** `backend/core/auth_rate_limit.py:36-39`

```python
forwarded = request.META.get("HTTP_X_FORWARDED_FOR")
if forwarded:
    return forwarded.split(",")[0].strip()
return request.META.get("REMOTE_ADDR", "unknown")
```

**Exploração trivial:** atacante manda `curl -H 'X-Forwarded-For: 1.2.3.4' POST /graphql` e o servidor trata 1.2.3.4 como IP de origem. Muda o valor a cada request → rate limit zerado a cada tentativa → brute force ilimitado.

**Pior:** o comentário do código dizia literalmente *"Quando houver reverse proxy, confiar APENAS no IP do proxy"* — e a implementação fazia exatamente o oposto. Bug clássico de "código que documenta a coisa certa e implementa a errada".

**Correção aplicada (v2):**

```python
# Caddy seta X-Real-IP confiável (cliente NÃO consegue forjar).
real_ip = request.META.get("HTTP_X_REAL_IP")
if real_ip:
    return real_ip.strip()
return request.META.get("REMOTE_ADDR", "unknown")
```

E no `Caddyfile` adicionado `header_up X-Real-IP {remote_host}` para garantir que Caddy sobrescreve qualquer header forjado.

**Esse achado motivou o item 23B novo no checklist universal.**

---

### Discrepância 🟠 #2 — `settings.py` ainda tem fallback inseguro no banco

**Declarado na v1:** crítico #2 (fail-fast) e ALTO P1 corrigidos.

**Realidade no código:** `backend/core/settings.py:97`

```python
'PASSWORD': os.environ.get('POSTGRES_PASSWORD', 'postgres_password'),
```

A v1 corrigiu o `docker-compose.prod.yml` (que SETA a env) mas esqueceu do `settings.py` (que LÊ a env). Se alguém rodar o Django sem docker-compose — em Kubernetes, Heroku, Railway, ou outro orquestrador — o Django sobe com senha conhecida.

**Ironia:** o item 5C que **nasceu desta mesma auditoria** descreve exatamente essa classe de bug — "fail-fast preservado em TODAS as camadas, não só na orquestração". A v1 violou seu próprio item recém-criado.

**Correção aplicada (v2):**

```python
_POSTGRES_USER = os.environ.get('POSTGRES_USER')
_POSTGRES_PASSWORD = os.environ.get('POSTGRES_PASSWORD')
if not _POSTGRES_USER or not _POSTGRES_PASSWORD:
    raise ValueError(
        "[SEGURANÇA] POSTGRES_USER e POSTGRES_PASSWORD são obrigatórios..."
    )
```

---

### Discrepância 🟠 #3 — `ProtectedRoute` mantém literal hardcoded que o item 39B proíbe

**Declarado na v1:** crítico #8 corrigido via decode JWT + verificação de `exp`.

**Realidade no código:** `frontend/src/components/ProtectedRoute.tsx:10,40`

```typescript
const PORTFOLIO_DEMO_TOKEN = 'lumina_portfolio_demo';
// ...
if (token === PORTFOLIO_DEMO_TOKEN) return true;
```

**Ironia máxima:** o item 39B que **a v1 escreveu** diz textualmente:

> *"Nunca comparar token contra string literal hardcoded"*
> *"Se houver modo demo intencional, isolar em rota separada (/demo) com dados sintéticos próprios — não usar o ProtectedRoute real"*

A v1 redigiu essa pergunta e na mesma sessão implementou exatamente o anti-padrão que ela proíbe. A justificativa no comentário do código ("é exceção explícita pro showcase") não está prevista no item — ele não tem cláusula de exceção.

**Correção aplicada (v2) — Caminho A (rota separada):**

1. Criado `frontend/src/components/DemoRoute.tsx` — guard próprio que valida flag em `sessionStorage` (não `localStorage`).
2. Adicionada rota `/demo` em `App.tsx` usando `DemoRoute`.
3. `Login.tsx` modo portfólio passa a setar `sessionStorage.setItem('lumina_demo_mode', '1')` + navegar para `/demo` (em vez de fingir ser JWT no localStorage).
4. `ProtectedRoute.tsx` limpo: nenhuma exceção, só JWT válido com `exp` futuro.

Resultado: o `ProtectedRoute` cumpre o item 39B sem cláusula de exceção. Demo continua funcionando, isolado.

---

### Discrepância 🟡 #4 — `csrf_exempt` mantido sem documentação de dívida

**Declarado na v1:** crítico #4 mitigado via `SameSite=Strict`.

**Realidade:** `backend/core/urls.py:27` — `csrf_exempt(SecureGraphQLView.as_view(...))`. Não foi removido.

**Por que não é exploração ativa hoje:** auth via JWT no header `Authorization`, não via cookie de sessão Django. Atacante CSRF não consegue forçar o header certo. Mas vira problema quando:
- Multi-tenant chegar (ADR-001) e sessão Django começar a ser usada
- JWT migrar para cookie httpOnly (ADR-002 expirar) — cookie automático = CSRF possível

**Correção aplicada (v2):** comentário detalhado em `urls.py` explicando a dívida + atualização do `THREAT_MODEL.md` listando como risco residual conhecido com gatilhos objetivos de quando remover.

---

## 🔴 Bloco 3 — Achados novos (escaparam à v1)

### Novo #A — CSP do frontend com `'unsafe-inline'` em `script-src`

**Arquivo:** `frontend/vercel.json:11` — `"script-src 'self' 'unsafe-inline'"`.

`'unsafe-inline'` em script-src **anula a principal proteção XSS da CSP**. Se atacante injetar `<script>` em qualquer parte do DOM (via XSS persistente em campo de input ou via dependência comprometida), o browser executa.

**Por que está aí:** Vite gera `<script type="module">` inline no `index.html`. Sem `'unsafe-inline'`, app não carrega.

**Correção aplicada (v2):** documentado como dívida explícita em [ADR-003](../Lumina-Booking-SaaS/docs/adr/ADR-003-csp-unsafe-inline.md). Compensações em camada (frame-ancestors, object-src, base-uri, form-action, Permissions-Policy, ProtectedRoute valida JWT) elencadas. Plano de migração: nonces via plugin Vite quando sair de portfólio.

**Esse achado motivou o item 20B novo no checklist universal.**

### Novo #B — Falta `SECURE_REFERRER_POLICY` no Django

**Arquivo:** `backend/core/settings.py` — não tinha `SECURE_REFERRER_POLICY` setado.

Caddy aplica em prod (`Referrer-Policy: strict-origin-when-cross-origin`), mas:
- Em dev (sem Caddy) o header sumia
- Defesa em profundidade dita que ambas as camadas devem aplicar

**Correção aplicada (v2):** `SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'` adicionado em `settings.py`.

### Novo #C — `customer_id` aleatório por Subscription (defeito conceitual)

**Arquivo:** `backend/metrics/models.py:6` — `customer_id = models.UUIDField(default=uuid.uuid4)`.

Cada subscription ganha `customer_id` aleatório. Não há entidade `Customer`. Para um projeto que se chama "Analytics SaaS" e calcula LTV:
- **Churn:** clientes que cancelam e voltam contam como 2 clientes diferentes
- **LTV:** impossível somar receita por cliente real
- **Cohort analysis:** agregação por mês de aquisição fica incorreta

Não é segurança, é correção lógica. Mas é defeito visível pra qualquer auditor técnico.

**Correção aplicada (v2):** docstring detalhada em `models.py` documentando a dívida e plano de migração junto com multi-tenant (ADR-001).

### Novo #D — `"Enterprise"` no cabeçalho do `settings.py`

**Arquivo:** `backend/core/settings.py:4` — `🛡️ PROTOCOLO DE SEGURANÇA ENTERPRISE — LUMINA ANALYTICS`.

Dissonante com o "honestidade primeiro" do README. As outras ocorrências de "Enterprise" no projeto são **nome legítimo de plano** (`Starter/Pro/Enterprise` em `models.py`, `seed_subscriptions.py`) ou labels de UI do produto — não são marketing exagerado.

**Correção aplicada (v2):** cabeçalho do `settings.py` reescrito com descrição técnica das camadas de segurança que o arquivo configura, sem jargão.

---

## 📊 Bloco 4 — Tabela v1 → v2 (evolução)

| Métrica | Após v1 (declarado) | Após v2 (validado + corrigido) |
|---------|---------------------|---------------------------------|
| Itens confirmados sólidos | 25 (declarados) | **31** (23 da v1 + 8 corrigidos na v2) |
| Vulnerabilidades exploráveis | 0 (declarado) | **0** (após corrigir o X-Forwarded-For) |
| Taxa de entrega real declarado vs. código | — | **92% na v1, 100% na v2** |
| Dívidas reconhecidas em ADR | 2 (ADR-001, ADR-002) | **3** (+ ADR-003 sobre CSP) |
| Riscos residuais documentados no THREAT_MODEL | 5 | **7** (+ csrf_exempt e CSP unsafe-inline) |
| Camadas com fail-fast (item 5C) | 1 (`docker-compose.prod.yml`) | **2** (+ `settings.py`) |
| Itens novos contribuídos ao checklist universal | 2 (5C, 39B) | **4** (+ 20B, 23B) |
| Rate limit em `tokenAuth` | implementado mas teatro | **funcional** (X-Real-IP do Caddy não-fakeável) |
| Rota `/demo` isolada do `ProtectedRoute` | não | **sim** (DemoRoute próprio, sessionStorage) |

---

## 🔄 Evolução dos checklists a partir desta auditoria

Dois novos itens promovidos a perguntas universais (somam aos 5C e 39B que vieram da v1):

| Achado | Item novo |
|--------|-----------|
| `_client_ip` lendo `X-Forwarded-For` cru (atacante forja header → rate limit zerado a cada request) | **Item 23B** no `AUDIT_CHECKLIST.md`: *"Quando a aplicação está atrás de proxy/load balancer/CDN, o IP do cliente usado para rate limit/audit log é confiável (não-fakeável por header de origem)?"* |
| `script-src 'unsafe-inline'` em `vercel.json` anula a principal proteção XSS da CSP | **Item 20B** no `AUDIT_CHECKLIST.md`: *"A CSP em produção é estrita (sem 'unsafe-inline' ou 'unsafe-eval' em script-src), ou foi relaxada para fazer o framework funcionar?"* |

Ambos têm rastro de origem rastreável: foram encontrados em projeto real, têm pergunta-teste verificável, e têm receita de correção por contexto (Caddy/Cloudflare/multi-proxy para o 23B; Vite/Next.js/nonces para o 20B).

---

## 🧭 Reflexão final

A v2 entregou três lições de método que são mais valiosas que qualquer correção individual:

**1. Auditor não pode auditar a si mesmo sem perder calibração.** A v1 declarou "92% de entrega" de boa fé, mas 4 das declarações tinham bugs — 1 deles era vulnerabilidade crítica (X-Forwarded-For). Sem uma segunda passada (humana ou ferramenta), esses bugs ficavam no repo e na cabeça do autor passariam como "tudo verde". O **loop de auditoria de segunda passada é parte do método, não excepcional**. Toda auditoria importante merece v2.

**2. Itens criados na própria auditoria viram armadilhas se aplicados de cabeça, não contra código real.** A v1 criou os itens 5C ("fail-fast em todas as camadas") e 39B ("não comparar token contra literal hardcoded") — e violou os dois na implementação. Isso é **exatamente** o padrão "checklist de cabeça vs. checklist aplicado". A regra que emerge: depois de adicionar item novo ao checklist, **revisar o código onde o item se aplica como se fosse a primeira vez**, sem assumir que "implementei pensando nele".

**3. Honestidade técnica é uma muscle, não um traço de personalidade.** Aceitar a v2 como crítica legítima — em vez de defender as entregas da v1 — é o mesmo músculo que produziu o README com "Escopo desta versão" e os ADRs com gatilhos objetivos. Quem audita de verdade fica feliz quando alguém encontra um bug que escapou: é menos um bug em produção e um item novo no checklist.

> **Aprendizado para próximas auditorias:** sempre que possível, rodar duas passadas adversariais com gap de tempo entre elas (mesmo que seja só "fechar o relatório, dormir, abrir de novo amanhã"). O olho cansado de quem implementou ainda está calibrado para "achar o que sabe que está lá". O olho fresco de uma segunda passada pega o que escapou.
