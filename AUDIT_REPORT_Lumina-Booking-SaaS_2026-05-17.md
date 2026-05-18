# 🔍 Auditoria de Segurança — Lumina-Booking-SaaS

> **Data:** 2026-05-17
> **Método:** Aplicação do `AUDIT_CHECKLIST.md` (universal, 42 itens base + sufixos 3B, 5B, 5C, 6B, 6C, 9B, 9C, 9D, 11B, 13B, 13C, 17B, 39B) + `CONTEXT_ADDONS.md` (Seções D — Multi-tenant SaaS, F — APIs públicas, J — GraphQL).
> **Escopo:** Backend Django/Graphene, frontend React+Vite, marketing-site, Docker (dev e prod), pipeline GitHub Actions, arquivos de governança.
> **Status do projeto na data:** Versão de portfólio (prévia). O autor confirmou explicitamente decisões de escopo conscientes — ver seção "Nota sobre escopo" abaixo.
> **Resultado em uma frase:** Projeto se posiciona como SaaS B2B mas é uma demo de dashboard analítico sem multi-tenant real (decisão consciente, agora documentada no README). Independentemente disso, 13 achados técnicos de segurança merecem correção mesmo no estágio de demo — vão desde introspecção GraphQL ligada em prod até um `ProtectedRoute` que aceita qualquer string truthy como autenticado.

---

## 📖 Como ler este relatório

Auditoria pontual (sem versão anterior para comparar). O projeto é pequeno (`metrics/schema.py` com 124 linhas, `metrics/models.py` com 15, um único resolver), então grande parte dos itens dos checklists cai como **N/A por ausência de superfície** — não porque o projeto resolveu, mas porque a feature ainda não existe. Quando isso acontece, o item entra na lista N/A com a observação "ausente, não resolvido".

Organização:

1. **Nota sobre escopo** — o que é decisão consciente do autor vs. o que é bug real.
2. **Bloco 1** — Confirmado e excelente.
3. **Bloco 2** — Parcial.
4. **Bloco 3** — Achados críticos / violações (independentes do escopo de portfólio).
5. **Bloco 4** — Polimentos.
6. **Itens N/A** — com justificativa.
7. **Resumo executivo**, **Plano de ação**, **Evolução dos checklists**, **Reflexão**.

---

## 📌 Nota sobre escopo (importante antes de ler os achados)

O autor confirmou que esta é uma **versão de portfólio (prévia)** com decisões de escopo conscientes, agora documentadas no `README.md` na seção "⚠️ Escopo desta versão":

- **Multi-tenant não foi implementado** porque exigiria integração real de pagamento recorrente, gerando custo operacional incompatível com um projeto-demo.
- **Celery/Redis** mencionados no README são destino arquitetural, não implementação atual.
- **Dados são sintéticos** (gerados por Faker) — não há clientes reais nem PII real.
- **STRIPE_* em `.env.example`** é decorativo — não há rota de webhook implementada.

**Isso não é fraude — é decisão de escopo legítima para portfólio.** O problema original era que o README descrevia essas features como entregues. Com a seção "Escopo desta versão" agora no README, a distância entre promessa e entrega foi fechada honestamente.

**O que continua sendo achado real (independente do escopo):**

Os 13 achados críticos do Bloco 3 não dependem de a feature multi-tenant existir. São decisões de configuração (`csrf_exempt`, introspecção em prod, `ALLOWED_HOSTS=*` como fallback), bugs de implementação (`ProtectedRoute` que aceita qualquer string), e dependências abandonadas. Vale corrigir mesmo numa demo, porque uma demo que segura essas básicas é exatamente o que diferencia portfólio sênior.

---

## ✅ Bloco 1 — Confirmado e excelente

| # | Item do checklist | Onde |
|---|-------------------|------|
| 5 | Fail-fast da `SECRET_KEY` no boot | `backend/core/settings.py:19-21` — sem fallback, levanta `ValueError` se ausente. Item exemplar do projeto. |
| 8 (parcial) | `.env` está no `.gitignore` | `.gitignore:6` |
| 10 | Cookies de sessão com flags seguras em prod | `settings.py:137-139` — `SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`, `SESSION_COOKIE_HTTPONLY` quando `DEBUG=False`. |
| 20 (parcial) | Cabeçalhos básicos de segurança Django | `settings.py:123-136` — HSTS 1 ano, `SECURE_SSL_REDIRECT`, `X_FRAME_OPTIONS=DENY`, `nosniff`. Falta CSP. |
| 28 | CORS com allowlist (não `*`) | `settings.py:67-71` — `CORS_ALLOW_ALL_ORIGINS=False` e lista explícita. |
| J1 (parcial) | GraphiQL desabilitado em prod | `backend/core/urls.py:15-19` — `graphiql=is_debug`. Mas introspecção GraphQL em si **continua ligada** mesmo com GraphiQL off (ver Bloco 3). |
| 6B (parcial) | JWT com expiração curta | `settings.py:154-159` — access 15min, refresh 7 dias. Falta `algorithms` explícito e força do secret. |

**Sólido sem ressalvas: 1 item (o 5).** O restante está marcado como "parcial" — configurado mas com reforço faltando.

---

## 🟠 Bloco 2 — Parcial (precisa de atenção)

### Item 8 — `.env` no `.gitignore`, mas existe um `.env` no diretório com chave secreta de baixa entropia

`Lumina-Booking-SaaS/.env` existe localmente e contém `SECRET_KEY=django-dev-local-key-only-for-docker-compose-dev`. O `.gitignore:6` cobre `.env`, então provavelmente não foi commitado. Validar com `git log --all --full-history -- .env` antes de considerar resolvido.

**Risco adicional ligado ao item 5C (novo):** `docker-compose.yml:38` usa `SECRET_KEY=${SECRET_KEY:-django-dev-key-change-me-in-production}` — **fallback inseguro embutido no compose**. Anula o fail-fast do `settings.py:19` em qualquer ambiente que use esse compose sem override de env.

### Item 20 — Headers Django OK, mas sem CSP

`settings.py:122-136` configura HSTS/nosniff/XFO, mas **não há Content-Security-Policy** nem `Referrer-Policy`. Sem `django-csp` instalado em `requirements.txt`.

### Item 6B — JWT sem allowlist explícita de algoritmo e sem secret de força garantida

- `settings.py:154-159` não passa `JWT_ALGORITHM` explícito; depende do default da lib `django-graphql-jwt==0.3.4` (HS256). Versão **antiga e abandonada** (último release 2020).
- `SECRET_KEY` de dev é string digitada por humano, baixa entropia. Como o mesmo `SECRET_KEY` é usado para JWT em `django-graphql-jwt`, a recomendação do item 6B (CSPRNG ≥ 256 bits) é violada em dev e, pelo `docker-compose.yml:38`, possivelmente em qualquer ambiente que herde o fallback.
- Não há revogação de refresh token, não há rotação, não há blocklist.

---

## 🔴 Bloco 3 — Achados críticos / violações (independentes do escopo de portfólio)

### CRÍTICO 1 — `docker-compose.prod.yml` permite `ALLOWED_HOSTS=*` e senha hardcoded como fallback (itens 5C, 8)

**Arquivo:** `docker-compose.prod.yml:25,30`

```yaml
- POSTGRES_PASSWORD: ${PROD_DB_PASSWORD:-securepassword123}
- ALLOWED_HOSTS=${ALLOWED_PROD_HOSTS:-*}
```

Se o operador esquecer de definir essas envs, o container sobe em produção com:
- `ALLOWED_HOSTS=*` (habilita Host header injection / cache poisoning)
- Senha de banco `securepassword123` (literalmente conhecida via leitura do compose público)

O `settings.py:19` se esforça pra falhar sem `SECRET_KEY`, mas o compose reintroduz o padrão inseguro silenciosamente. **Esse padrão exato motivou a criação do item 5C no checklist universal** (ver "Evolução dos checklists").

**Correção:** remover os defaults, exigir env, falhar o boot.

---

### CRÍTICO 2 — Introspecção GraphQL ligada em produção (J1)

**Arquivos:** `backend/core/urls.py:15-19`, `backend/core/settings.py:146-151`

```python
path('graphql', csrf_exempt(GraphQLView.as_view(graphiql=is_debug)))
```

`graphiql=False` apenas desliga a UI; **introspecção GraphQL continua respondendo** a `__schema`/`__type` porque `graphene-django` não tem nenhuma validation rule de bloqueio configurada. Atacante consegue o schema completo via `query { __schema { types { name fields { name } } } }`.

**Severidade:** ALTA em prod. Mesmo numa demo de portfólio, dar mapa do schema pro visitante anônimo é desnecessário.

**Correção:** registrar `NoSchemaIntrospectionCustomRule` no `GraphQLView` quando `not DEBUG`.

---

### CRÍTICO 3 — `ProtectedRoute` aceita qualquer string truthy como autenticado (item 39B)

**Arquivo:** `frontend/src/components/ProtectedRoute.tsx:8-12`

```tsx
const token = localStorage.getItem('lumina_token');
if (!token) return <Navigate to="/login" />;
return children;
```

**Bypass trivial:** qualquer visitante abre o console em produção, digita `localStorage.setItem('lumina_token','x')`, dá refresh, e entra direto no `/dashboard`. Não há validação de assinatura, de expiração, de formato — só `if (token)`.

O Login.tsx:32-39 ainda piora: tem "modo portfólio" que armazena o literal `'lumina_portfolio_demo'` como token, então o autor mesmo confirmou intencionalmente que qualquer string passa.

**Correção:** validar minimamente com `jwtDecode` + verificar `exp`; idealmente fazer chamada `/me` ao backend no boot da rota protegida. **Esse achado motivou a criação do item 39B no checklist universal.**

---

### CRÍTICO 4 — Frontend armazena JWT em `localStorage` (item 39)

**Arquivos:** `frontend/src/pages/Login.tsx:35,66`, `frontend/src/pages/Dashboard.tsx:46,54`

```tsx
localStorage.setItem('lumina_token', result.data.tokenAuth.token);
```

Vulnerável a XSS — e o frontend instala 30+ dependências (`@radix-ui`, `recharts`, `embla-carousel`, etc.), aumentando superfície de supply chain. Combinado com o critico 3, qualquer XSS vira sessão eterna.

---

### CRÍTICO 5 — `csrf_exempt` no endpoint GraphQL + JWT sem revogação (itens 9, 6B, 29)

**Arquivo:** `backend/core/urls.py:19`

```python
path('graphql', csrf_exempt(GraphQLView.as_view(...)))
```

Combinado com `SESSION_COOKIE_SAMESITE` **não definido** em `settings.py` (default Django = `'Lax'`) e `CORS_ALLOW_CREDENTIALS = True` (`settings.py:72`). Funciona por sorte hoje porque o frontend não passa cookie — manda token no body. Mas o padrão é frágil.

---

### CRÍTICO 6 — Sem proteção alguma contra brute force / rate limit (itens 7, F1)

Nenhuma instalação de `django-ratelimit`, `django-axes`, nem rate limit no proxy. A mutation `tokenAuth` em `core/schema.py:9` aceita tentativas ilimitadas. Combinado com `SECRET_KEY` fraco em dev e ausência de account lockout, brute force trivial em qualquer username válido.

---

### CRÍTICO 7 — Sem query depth limit, sem complexity analysis, sem batching protection (J2, J3, J4)

`graphene-django>=3.0` instalado nu. Não há `graphql-depth-limit` nem equivalente. Query maliciosa profunda derruba o servidor. Aliasing permite N tentativas de login em 1 request.

---

### CRÍTICO 8 — Autorização field-level ausente (J5)

Único resolver (`resolve_dashboard_metrics`) faz `@login_required` no entry point e nenhuma checagem field-level. Como o backend está hoje (sem multi-tenant — decisão de escopo do autor), todo usuário autenticado vê todas as métricas. **Quando o multi-tenant for implementado**, esse achado vira a coisa mais crítica do projeto. Hoje vale como dívida arquitetural conhecida.

---

### CRÍTICO 9 — `Dockerfile` roda como root, base não-minimal, sem multi-stage (item 37)

**Arquivo:** `backend/Dockerfile:1-18`

- Single-stage (`python:3.11-alpine` carrega `gcc`, `g++`, `postgresql-dev` em runtime — superfície enorme).
- `USER` nunca trocado → roda como root.
- Sem `HEALTHCHECK`.
- Sem `.dockerignore` na raiz nem em `backend/` → `.env`, `__pycache__`, `.git` potencialmente entram na imagem.
- `COPY . /app/` (linha 18) copia tudo, incluindo possivelmente `.env`.

---

### CRÍTICO 10 — Prod expõe porta 8000 do gunicorn diretamente, sem reverse proxy / TLS

`docker-compose.prod.yml:24-25` mapeia `"8000:8000"` direto na VM, sem nginx/Caddy/Traefik na frente. `SECURE_SSL_REDIRECT=True` (`settings.py:136`) vai entrar em loop se não houver terminador TLS upstream. Não há config de proxy nem `SECURE_PROXY_SSL_HEADER` em settings.

---

### CRÍTICO 11 — Dependências críticas desatualizadas / abandonadas (item 30)

`backend/requirements.txt`:
- `django-graphql-jwt==0.3.4` — **pinned em versão de 2020, lib praticamente abandonada**.
- `Django>=4.2,<5.0` — range OK mas frouxo (README promete Django 5).
- `pandas>=2.0` e `Faker>=20.0` em produção sem necessidade aparente (Faker só é usado no seed).
- Não há `pip-audit`, `safety`, `dependabot.yml`.

---

### CRÍTICO 12 — CI/CD não roda nenhum check de qualidade ou segurança (item 12)

`.github/workflows/deploy_backend.yml` é só um SSH+`git pull`+`docker-compose up`. **Nenhum step de lint, type-check, teste, `pip-audit`, `gitleaks`, Trivy, Bandit.** O nome "Deploy" é literal — não há "CI", apenas deploy direto a partir do `main`. Qualquer commit em main sobe direto pra produção sem teste.

Padrão `git pull origin main` no servidor + `docker-compose exec ... migrate` é frágil: secrets ficam no servidor (não no registry), rollback é manual, migrações destrutivas vão direto. Não há tag/release.

---

### CRÍTICO 13 — Raiz poluída por arquivos de rascunho / IA / TODO (profissionalismo)

Olho-de-recrutador imediato. Arquivos visíveis no `ls` da raiz que merecem ir pra `docs/_internal/` ou serem deletados:

- `findings.md` (4 linhas, "Nenhuma descoberta registrada ainda.")
- `progress.md` (log narrativo de fases, jargão privado "V.L.A.E.G", "Midnight Luxe")
- `task_plan.md` (TODOs internos com checkboxes)
- `referencias997.md`
- `vercel_help.txt` (0 bytes)
- `tmp_vercel_html.txt` (0 bytes)
- `gemini.md` (8 KB — prompt/saída de IA)
- `explica.md` (pasta vazia)
- `.tmp/` e `.vercel/` na raiz versionados
- `build-error.log` em `frontend/`

Esse achado é o que mais barato corrige e mais visivelmente eleva a percepção do repo.

---

## 🟡 Bloco 4 — Polimentos

- **`frontend/build-error.log` versionado** — log de erro de build não deveria estar no repo.
- **Logging**: `print(...)` em `seed_subscriptions.py:21,30` em vez de logger estruturado (item 14).
- **`time_zone='America/Sao_Paulo'`** mas o resolver faz `datetime.fromisoformat(...)` sem tz-awareness defensiva.
- **Marketing-site `vercel.json`** e **frontend `vercel.json`** não definem nenhum header de segurança (CSP, X-Frame-Options, Referrer-Policy). Item 41 violado em ambos.
- **`SESSION_COOKIE_SAMESITE`** e **`CSRF_COOKIE_SAMESITE`** não definidos explicitamente em `settings.py`.
- **`backend/test_graphql.py` e `test_graphql_full.py`** são scripts ad-hoc com `urlopen` apontando para `localhost:8000`, não testes pytest reais (item 11). Falsa sensação de "tem teste".

---

## 📋 Itens não-aplicáveis

| Seção/item | Justificativa |
|---|---|
| Seção A (WebSocket) | Projeto não usa real-time. Nenhum `channels`, `socket.io`, SSE. |
| Seção B (Upload) | Nenhum endpoint de upload. |
| Seção C (Pagamento) | `STRIPE_*` em `.env.example` é decorativo — nenhuma integração existe. Decisão consciente do autor (ver README seção "Escopo desta versão"). |
| Seção E (IA/LLM) | Não usa LLM. |
| Seção H (PII/financeiro) | Manipula dados financeiros agregados (MRR, churn), mas como não há separação por tenant nem por usuário, **não há PII real persistida** — só dados sintéticos do Faker. Aplicar H4 (audit log de acesso a PII) seria recomendado se algum dia houver tenants reais. |
| Seção I (Filas) | README menciona Celery como destino arquitetural; não implementado — N/A por escopo declarado. |
| D1-D6 (Multi-tenant) | Não implementado por decisão consciente de escopo (ver README). Quando for implementado, deve ser primeira coisa a auditar. |
| Itens 3B, A2B (identidade async) | N/A — não há handlers async com payload de identidade. |
| Itens 9B (SSRF), 9C (deserialização/SSTI) | Não há fetch de URLs do usuário; não há `pickle`/`eval` com input do usuário. |
| Itens 13C, 17B (race conditions / cotas) | Não há lógica de cota/limite implementada. |
| Item 33 (soft delete), 34 (audit log), 36 (LGPD retenção) | Sem usuários reais nem CRUD — ainda não aplicável. |
| Item 35 (DR plan) | N/A para portfólio. Essencial num SaaS real. |

---

## 📊 Resumo executivo

| Status | Quantos |
|---|---|
| ✅ Confirmados sólidos | **1** (item 5 — SECRET_KEY fail-fast) |
| 🟠 Parciais | **4** (itens 6B, 8, 19, 20) |
| 🔴 Violados / críticos | **13** (independentes do escopo de portfólio) |
| 🟡 Polimentos | **6** |
| 📋 N/A por escopo declarado | ~15 itens / 6 seções de contexto |

**Veredicto curto:** Versão de portfólio com decisões de escopo agora documentadas honestamente no README. Os 13 achados críticos do Bloco 3 **não dependem** das features não-implementadas — são configurações de segurança e bugs de implementação que merecem correção mesmo numa demo. Corrigi-los transforma o projeto de "demo competente em features" para "demo competente em features E em postura de segurança".

---

## 🎯 Plano de ação ordenado por severidade

1. **🔴 Corrigir `ProtectedRoute.tsx`** — validar token de verdade (decode + exp), não aceitar qualquer string. Bug mais explorável do projeto hoje.
2. **🔴 Remover `${ALLOWED_PROD_HOSTS:-*}` e `${PROD_DB_PASSWORD:-securepassword123}`** do `docker-compose.prod.yml` — exigir env, falhar o boot.
3. **🔴 Desabilitar introspecção GraphQL em prod** — `NoSchemaIntrospectionCustomRule` quando `not DEBUG`.
4. **🔴 Adicionar depth limit + query complexity** + bloquear batching/aliasing abusivo.
5. **🔴 Trocar `django-graphql-jwt==0.3.4`** por lib mantida; passar `algorithms=['HS256']` explicitamente; gerar `SECRET_KEY` via CSPRNG.
6. **🔴 Rate limit + lockout** no `tokenAuth` (`django-axes` ou `django-ratelimit`).
7. **🔴 Migrar token para cookie httpOnly** com proteção CSRF, ou pelo menos documentar como ADR consciente o trade-off de localStorage.
8. **🔴 Dockerfile multi-stage** + `USER appuser`, `.dockerignore`, base distroless/slim, healthcheck.
9. **🔴 Reescrever CI** — adicionar `ruff`, `bandit`, `pip-audit`, `pytest`, `gitleaks` antes do deploy. Bloquear deploy se falhar.
10. **🟠 Limpar a raiz** — mover `findings.md`/`progress.md`/`task_plan.md`/`gemini.md`/`explica.md`/`referencias997.md` para `docs/_internal/` (e gitignorar) ou deletar; remover `tmp_*.txt`, `vercel_help.txt`, `build-error.log`, `.tmp/`, `.vercel/`.
11. **🟠 Adicionar CSP** via `django-csp` + headers no `vercel.json` de frontend e marketing-site.
12. **🟠 Substituir scripts `test_graphql.py`** por suíte `pytest-django` real — pelo menos 1 teste adversarial mesmo em demo.
13. **🟢 Adicionar `SECURITY.md`, `THREAT_MODEL.md`, `ADR-001-multi-tenant-fora-de-escopo.md`** — formalizar a decisão de escopo do README como ADR rastreável.
14. **🟢 Tag/release semântico** no deploy em vez de `git pull` na VM.

---

## 🔄 Evolução dos checklists a partir desta auditoria

Dois padrões observados nesta auditoria viraram perguntas novas nos documentos universais:

| Achado | Item novo |
|---|---|
| `docker-compose.prod.yml` reintroduzindo fallback inseguro depois que `settings.py` se esforçou para falhar-rápido | **Item 5C** no `AUDIT_CHECKLIST.md`: *"O fail-fast configurado no código é preservado em TODAS as camadas de orquestração (compose, helm, terraform), ou alguma reintroduz default inseguro?"* |
| `ProtectedRoute` no frontend aceitando qualquer string truthy como "autenticado" | **Item 39B** no `AUDIT_CHECKLIST.md` (Frontend): *"O guard de rota valida o token (decode + exp), ou só verifica `if (token)` — permitindo bypass com `localStorage.setItem('token','x')`?"* |

Ambos são bugs sutis que **passariam batido** se o auditor só rodasse os itens 5 (fail-fast) e 39 (token storage) tradicionais — porque a defesa real está quebrada por uma camada adjacente.

---

## 🧭 Reflexão final

O caso Lumina trouxe duas lições importantes para a evolução do método.

**Primeira lição — decisão de escopo é informação crítica:** uma auditoria que ignora o contexto de escopo do projeto pode reportar como "fraude" o que na verdade é decisão consciente do autor. Quando o autor confirmou que multi-tenant não foi implementado por escolha (custo de integração de pagamento), o achado original ("README mente") virou achado mais sutil ("README precisava documentar a decisão") — e a correção virou trivial (uma seção de 6 bullets no README) em vez de "reescrever metade do backend". A próxima evolução do método é perguntar ao autor sobre decisões de escopo **antes** de fechar o relatório, não depois.

**Segunda lição — defesas em camadas precisam ser auditadas em camadas:** os itens 5 (fail-fast) e 39 (token storage) do checklist clássico estavam tecnicamente OK. Mas o fail-fast era anulado pelo docker-compose (item 5C novo) e o token storage tinha um guard inútil em volta (item 39B novo). Auditoria competente não pode parar no primeiro ✅ — precisa verificar se a camada adjacente preserva ou destrói a defesa. Esses dois itens novos vão pegar exatamente esse padrão em projetos futuros.

> **Aprendizado para próximas auditorias:** confirmar o estágio do projeto (portfólio/MVP/produção) e decisões de escopo do autor **antes** de classificar a severidade dos achados. Um README "marketing acima da realidade" pode ser bug ou pode ser texto sem ADR — a diferença muda completamente o plano de ação.
