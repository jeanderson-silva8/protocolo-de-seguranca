# 🛡️ AUDIT_CHECKLIST.md

> **Checklist Universal de Auditoria de Projetos**
> Aplicável a qualquer projeto backend/fullstack profissional.
> Para contextos específicos (WebSocket, upload, pagamento, multi-tenant, IA, GraphQL, etc.), consulte os adendos em [`context-addons/`](context-addons/).

---

## 📖 Como usar

Percorra cada pergunta **na ordem**. Para cada uma, escolha **opção 1** (problema existe — anote para resolver) ou **opção 2** (está OK — siga em frente).

Ao final, você terá uma lista priorizada de correções. Comece sempre pelos itens 🔴 BLOQUEADORES, depois 🟠 ESSENCIAIS, depois 🟡 QUALIDADE, e por fim 🟢 DIFERENCIAÇÃO. A seção 🎨 FRONTEND tem itens transversais — rodar quando o projeto tem frontend próprio.

> 💡 **Disciplina-chave:** para cada item ✅, anote `arquivo.ext:linha` que prova a mitigação. Sem âncora no código, o ✅ é só intenção. Essa regra evita a classe de bug "claimed → não entregue" identificada na auditoria do BrieflyAI.

---

## 🗂️ Índice navegável

### 🔴 BLOQUEADORES (1-18) — sem isso, é vulnerável

| # | Pergunta |
|---|----------|
| [1](#1-autenticação-está-implementada-em-todas-as-rotas-que-tocam-dados-privados) | Auth em todas as rotas privadas |
| [2](#2-existe-verificação-de-autorização-não-só-autenticação-em-toda-operação-que-toca-recurso-pertencente-a-alguém) | Autorização em toda operação |
| [3](#3-ids-e-referências-a-recursos-vêm-da-sessão-token-não-do-payload-do-cliente-quando-deveriam) | IDs sensíveis vêm da sessão (não do payload) |
| [4](#4-em-handlers-assíncronos-sockets-queues-callbacks-jobs-a-identidade-do-ator-vem-da-camada-autenticada-handshake-jwt-header-e-nunca-é-re-aceita-do-payload-do-cliente-em-handlers-subsequentes) | Identidade em handlers async (socket/queue/job) |
| [5](#5-toda-entrada-de-cliente-é-validada-com-biblioteca-zod-joi-yup-pydantic-antes-de-chegar-no-controller) | Validação Zod/Joi/Pydantic em todos os inputs |
| [6](#6-variáveis-de-ambiente-críticas-são-validadas-no-boot-fail-fast) | Envs validadas no boot (fail-fast) |
| [7](#7-dependências-críticas-de-runtime-banco-cache-broker-kms-também-causam-fail-fast-no-boot-em-produção-ou-só-envs) | Fail-fast também para dependências de runtime |
| [8](#8-o-fail-fast-configurado-no-código-da-aplicação-é-preservado-em-todas-as-camadas-de-orquestração-docker-compose-helm-terraform-kustomize-ou-alguma-reintroduz-um-default-inseguro-que-silenciosamente-neutraliza-o-fail-fast) | Fail-fast preservado em camadas de orquestração |
| [9](#9-senhas-são-armazenadas-com-algoritmo-lento-e-adequado-argon2id-bcrypt-cost--10-scrypt) | Hash de senha adequado (Argon2id/bcrypt) |
| [10](#10-jwt-está-configurado-com-algoritmo-seguro-expiração-curta-refresh-token-e-estratégia-de-revogação) | JWT com algoritmo seguro + refresh + revogação |
| [11](#11-comparações-de-valores-secretos-tokens-hashes-hmac-api-keys-usam-função-tempo-constante-cryptotimingsafeequal-ou-equivalente) | Comparação tempo-constante para tokens/HMAC |
| [12](#12-existe-proteção-contra-brute-force-em-endpoints-de-autenticação-rate-limit--account-lockout) | Brute force protection em auth |
| [13](#13-segredos-api-keys-tokens-senhas-de-banco-estão-fora-do-código-versionado) | Segredos fora do código versionado |
| [14](#14-queries-ao-banco-usam-parâmetros-não-concatenação-de-string) | Queries parametrizadas (anti-injection) |
| [15](#15-requisições-http-feitas-pelo-servidor-a-partir-de-input-do-usuário-são-protegidas-contra-ssrf) | Proteção contra SSRF |
| [16](#16-não-há-desserialização-insegura-nem-renderização-de-templates-com-input-do-usuário-ssti) | Sem desserialização insegura nem SSTI |
| [18](#18-cookies-de-sessãorefresh-têm-httponly-secure-samesite-configurados) | Cookies httpOnly/secure/sameSite |

### 🟠 ESSENCIAIS (19-33) — sem isso, não é portfólio sênior

| # | Pergunta |
|---|----------|
| 18 | Testes para caminhos críticos (auth, authz, validação) |
| 19 | Testes adversariais para cada ameaça do THREAT_MODEL |
| 20 | CI/CD com checks automáticos por PR |
| 21 | Error handler centralizado |
| 22 | Controllers usam `throw` (exercitam o middleware global) |
| 23 | Operações sensíveis sem race conditions / TOCTOU |
| 24 | Logging estruturado (JSON), padronizado |
| 25 | Logs limpos de PII/segredos |
| 26 | Documentação de API (OpenAPI ou API.md) |
| 27 | THREAT_MODEL.md honesto |
| 28 | Operações de cota usam mecanismo atômico (não count→if→write) |
| 29 | SECURITY.md presente |
| 30 | README honesto (não promete o que não entrega) |
| 31 | Headers de segurança (Helmet/equivalente) |
| 32 | CSP estrita (sem `'unsafe-inline'` em `script-src`) |

### 🟡 QUALIDADE (34-44) — visível para quem audita

| # | Pergunta |
|---|----------|
| 33 | IDs com formato estrito (regex/UUID) |
| 34 | Paginação em listagens |
| 35 | Rate limit POR USUÁRIO (não só por IP) |
| 36 | IP confiável atrás de proxy/CDN (não-fakeável) |
| 37 | Política de senha forte (mínimo, HIBP) |
| 38 | Health `live` + `ready` separados |
| 39 | Sem queries N+1 nas listagens |
| 40 | Erros sem vazar stack/internals em prod |
| 41 | CORS com allowlist (não `*`) |
| 42 | Proteção CSRF para endpoints com cookie |
| 43 | Dependências atualizadas + audit (Dependabot) |

### 🟢 DIFERENCIAÇÃO (45-52) — separa sênior bom de sênior excelente

| # | Pergunta |
|---|----------|
| 44 | Correlation ID propagado (HTTP, logs, jobs) |
| 45 | Métricas/telemetria expostas (Prometheus/OTel) |
| 46 | Soft delete em operações destrutivas |
| 47 | Audit log de ações sensíveis |
| 48 | Plano de backup + DR testado |
| 49 | Política de retenção (LGPD/GDPR) |
| 50 | Dockerfile multi-stage |
| 51 | ADRs documentando decisões |

### 🎨 FRONTEND (53-58) — específico de cliente

| # | Pergunta |
|---|----------|
| 52 | Onde o access token é armazenado |
| 53 | Guard de rota valida o token (decode + exp), não só `if (token)` |
| 54 | Conteúdo do usuário sanitizado antes de renderizar HTML |
| 55 | CSP configurada também no servidor do frontend (Vercel/Netlify) |
| 56 | Erros do backend tratados sem expor detalhes |
| 57 | Nenhum segredo vaza para o bundle público do frontend |

### 🧭 META — disciplina de cobertura (59-61)

> *Itens "meta" que verificam se as defesas listadas acima foram aplicadas com consistência. Originados de peer reviews que pegaram **defesas declaradas mas não entregues** em produção ou em parte dos endpoints.*

| # | Pergunta |
|---|----------|
| 58 | Toda função que recebe ID externo valida o formato ANTES do banco — em **todos** os pontos, não só em alguns |
| 59 | Cada endpoint **destrutivo/sensível** tem ao menos um teste adversarial **dedicado àquele endpoint** |
| 60 | Existe **smoke test automatizado em CI** que valida defesas críticas na **URL pública pós-deploy**, não só no commit local |

---

## 🔄 Nota de migração — numeração

> Antes de 2026-05-18, os itens tinham sufixos (`3B`, `5C`, `9D`, `11B`, `13C`, `17B`, `20B`, `23B`, `39B`, etc.) — esses sufixos eram adicionados ao final de cada auditoria que gerava uma pergunta nova. Na primeira renumeração (v1.2.0), a numeração foi compactada para sequencial 1-56. Após a auditoria do TrendScope (v1.5.0), shift adicional levou a **sequencial 1-57** com a inserção do item 17 novo (serverless/multi-entrypoint) entre os antigos 16 (SSTI) e 17 (cookies). Após a auditoria do Organiza (v1.8.0), o **item 58** (segredos vazando para o bundle do frontend) foi adicionado ao final da seção 🎨 FRONTEND — sem shift, pois entrou como último item. Na sequência, a peer review do mesmo projeto (v1.9.0) trouxe os **itens 59 e 60** ("meta" — consistência de validação de ID em todos os pontos, e teste adversarial dedicado por endpoint sensível) numa banda nova 🧭 META no final do documento — também sem shift. Depois, a peer review do Miniatura Forja AI (v1.10.0) trouxe o **item 61** (smoke test pós-deploy validando defesas na URL pública) à mesma banda META. A numeração atual é **sequencial 1-61**.
>
> **Relatórios de auditoria publicados antes do shift usam a nomenclatura antiga.** Mapa de equivalência consolidado (sufixo antigo → número atual 1-57):
>
> | Antigo | Atual | | Antigo | Atual | | Antigo | Atual |
> |--------|-------|---|--------|-------|---|--------|-------|
> | 3B | 4 | | 11B | 20 | | 23B | 37 |
> | 5B | 7 | | 13B | 23 | | 39B | 54 |
> | 5C | 8 | | 13C | 24 | | 20B | 33 |
> | 6B | 10 | | 17B (cotas) | 29 | | | |
> | 6C | 11 | | | | | | |
>
> ⚠️ **Atenção ao "17B" duplicado historicamente:** existem dois `17B` no histórico — o antigo (que era "operações de cota atômicas", virou item 29 após os dois shifts) e o do TrendScope 2026-05-18 (que era "serverless/multi-entrypoint", virou item 17 após shift). Quando ler relatório do Lumina v1/v2 citando "17B", refere-se ao primeiro (item 29 atual). Quando ler relatório do TrendScope 2026-05-18 citando "17B", refere-se ao segundo (item 17 atual).
>
> Os relatórios antigos (BrieflyAI 2026-05-16, FlowSnyker v2 2026-05-16, Lumina v1 2026-05-17, Lumina v2 2026-05-18, TrendScope 2026-05-18) **não foram reescritos** — preservam o histórico real do método. Quando ler item com sufixo num relatório, consulte este mapa.

---

## 🔴 BLOQUEADORES — sem isso, é vulnerável

### 1. Autenticação está implementada em TODAS as rotas que tocam dados privados?

> **Opção 1 — Se sim, existe alguma rota sem auth:**
> - Listar todas as rotas que tocam dados privados
> - Identificar quais não exigem token válido
> - Aplicar middleware de autenticação (via `router.use(auth)` ou por rota)
> - Testar manualmente: requisição sem token retorna 401?
>
> **Opção 2 — Se não, todas exigem auth:** ✅ Excelente

---

### 2. Existe verificação de AUTORIZAÇÃO (não só autenticação) em toda operação que toca recurso pertencente a alguém?

> *Pergunta-teste: "Usuário A autenticado consegue editar/deletar/ler recurso do usuário B se souber o ID?"*
>
> **Opção 1 — Se sim, autorização está incompleta:**
> - Mapear todos os endpoints que recebem `:id` ou similar
> - Para cada um, garantir que verifica propriedade ou membership do recurso
> - Criar middleware reutilizável (ex: `requireOwnership`, `requireBoardMember`)
> - Aplicar em todas as rotas afetadas
> - Adicionar teste automatizado: "user A não acessa recurso de user B → 403"
>
> **Opção 2 — Se não, toda operação valida autorização:** ✅ Excelente

---

### 3. IDs e referências a recursos vêm da SESSÃO (token), não do payload do cliente quando deveriam?

> *Exemplo problemático: criar card recebendo `userId` no body em vez de usar `req.userId` do token.*
>
> **Opção 1 — Se sim, há campos sensíveis vindos do cliente:**
> - Listar todos os endpoints que aceitam `userId`, `ownerId`, `tenantId` no body
> - Substituir por valor derivado do token autenticado (`req.userId`)
> - Remover esses campos dos schemas Zod de entrada
> - Validar que cliente não consegue mais sobrescrever esses valores
>
> **Opção 2 — Se não, IDs sensíveis sempre vêm da sessão:** ✅ Excelente

---

### 4. Em handlers assíncronos (sockets, queues, callbacks, jobs), a identidade do ator vem da camada autenticada (handshake, JWT, header) e NUNCA é re-aceita do payload do cliente em handlers subsequentes?

> *Pergunta-teste: "Se o cliente emite um evento de socket com um campo `userId` ou `user._id` no payload, o servidor usa esse valor ou usa o valor que foi guardado no handshake autenticado?"*
>
> *Esse bug é uma classe sutil de IDOR: o handler valida que "o `userId` do payload é membro do recurso" — e o usuário do payload É membro, mas é OUTRO usuário, não o atacante. Atacante autenticado se passa por terceiros.*
>
> **Opção 1 — Se algum handler aceita identidade do payload do cliente:**
> - **CRÍTICO — risco de spoofing de identidade**
> - Localizar todos os pontos onde o handler lê `payload.userId`, `payload.user._id`, `payload.actor`, etc.
> - Substituir por `socket.userId` (do handshake), `req.userId` (do JWT middleware), `job.context.userId` (de fila autenticada)
> - Para dados adicionais (nome, avatar, role), buscar do banco usando o `userId` autenticado — nunca aceitar do cliente
> - Adicionar teste: "usuário A emite evento com `payload.user._id = B` → servidor deve usar A, não B"
>
> **Opção 2 — Se identidade SEMPRE vem da camada autenticada:** ✅ Excelente

---

### 5. Toda entrada de cliente é validada com biblioteca (Zod, Joi, Yup, Pydantic) ANTES de chegar no controller?

> **Opção 1 — Se não, há entradas sem validação:**
> - Listar endpoints sem schema de validação
> - Criar schemas para cada um: tipos, tamanhos, formatos, enums
> - Aplicar como middleware ANTES do controller
> - Usar formatos estritos: regex para IDs (ObjectId, UUID), `.email()`, limites min/max
> - Rejeitar campos extras não declarados (`.strict()` no Zod)
>
> **Opção 2 — Se sim, todo input é validado:** ✅ Excelente

---

### 6. Variáveis de ambiente críticas são validadas no BOOT (fail-fast)?

> *Pergunta-teste: "Se eu deletar `JWT_SECRET` do `.env` e iniciar o servidor, ele aborta com erro claro ou inicia silenciosamente?"*
>
> **Opção 1 — Se inicia sem JWT_SECRET ou usa `as string`:**
> - Listar todas as envs obrigatórias (`JWT_SECRET`, `DATABASE_URL`, etc)
> - Criar arquivo `config/env.ts` que valida e exporta tipadas
> - No boot: se faltar env obrigatória, `logger.error` + `process.exit(1)`
> - Remover todos os `as string` espalhados pelo código
> - Considerar usar Zod para validar o objeto `process.env` inteiro
>
> **Opção 2 — Se aborta com erro claro:** ✅ Excelente
>
> 🔗 *Relacionado: item 13 (segredos fora do código versionado). Este item garante que o segredo EXISTE no boot; o item 13 garante que ele não está no repositório. Os dois trabalham juntos.*

---

### 7. Dependências críticas de runtime (banco, cache, broker, KMS) também causam fail-fast no boot em produção, ou só envs?

> *Pergunta-teste: "Se o banco estiver offline no momento do deploy, o servidor sobe e responde 200 em `/health/live`, mascarando o problema até alguém tentar usar a API?"*
>
> *Fail-fast de envs (item 6) é metade do trabalho. Sem cobrir dependências de runtime, o app sobe "verde" com banco offline e o problema só aparece quando o primeiro request quebra — frequentemente longe do horário do deploy.*
>
> **Opção 1 — Se o servidor sobe sem dependência crítica em produção:**
> - Em produção (`NODE_ENV === 'production'`): falhar conexão a dependência crítica = `process.exit(1)` no boot
> - Dependências críticas típicas: banco principal, cache se a app depende dele, broker se eventos são síncronos, KMS se segredos são descriptografados no boot
> - Em dev: warning + modo offline aceitável (não bloquear desenvolvimento)
> - `/health/ready` deve refletir status real e contínuo das dependências; `/health/live` deve estar acompanhado de boot fail-fast (uma vez que subiu, o processo está vivo)
> - Logar com clareza qual dependência falhou e por quê — exit silencioso é tão ruim quanto boot silencioso
>
> **Opção 2 — Fail-fast cobre envs E dependências críticas em prod:** ✅ Excelente

---

### 8. O fail-fast configurado no código da aplicação é preservado em TODAS as camadas de orquestração (docker-compose, helm, terraform, kustomize), ou alguma reintroduz um default inseguro que silenciosamente neutraliza o fail-fast?

> *Pergunta-teste: "Meu `settings.py` levanta erro se `SECRET_KEY` não existir, mas meu `docker-compose.yml` tem `SECRET_KEY=${SECRET_KEY:-default-inseguro}` — esse fallback é acionado em produção, e o fail-fast do código continua valendo?"*
>
> *Esse é um bug sutil: o fail-fast do app está correto e foi testado isoladamente, mas a camada de orquestração injeta um default que faz o app NUNCA receber `None`. Resultado: o fail-fast nunca dispara, e a aplicação sobe em prod com `SECRET_KEY=django-dev-key-change-me-in-production` ou `ALLOWED_HOSTS=*`.*
>
> **Opção 1 — Se há defaults inseguros em camadas de orquestração:**
> - **CRÍTICO** — anula a defesa do item 6 sem dar nenhum sinal de alerta
> - Auditar todos os arquivos de orquestração: `docker-compose.yml`, `docker-compose.prod.yml`, charts Helm, manifests Kustomize, módulos Terraform
> - Procurar por padrões `${VAR:-default}`, `default = "..."`, `value: "fallback"` em variáveis sensíveis
> - **Remover o fallback** — deixar a env var resolver pra string vazia (ou exigir explicitamente). O app falha cedo e claro, em vez de subir com config insegura.
> - Para defaults aceitáveis em DEV (ex: `DEBUG=True`), separar em arquivo dedicado (`docker-compose.dev.yml`) e nunca herdar em prod
> - Adicionar teste de integração: subir o stack com env vars **ausentes** e verificar que o container falha no boot
> - 🔗 *Relacionado: item 6 (fail-fast no app). 8 é a camada de orquestração; 6 é a camada de aplicação. Ambos precisam estar corretos para que a defesa real exista.*
>
> **Opção 2 — Se camadas de orquestração preservam o fail-fast:** ✅ Excelente

---

### 9. Senhas são armazenadas com algoritmo lento e adequado (Argon2id, bcrypt cost ≥ 10, scrypt)?

> **Opção 1 — Se usa MD5, SHA-256, SHA-1, ou senha em claro:**
> - **CRÍTICO — trocar imediatamente**
> - Migrar para Argon2id (preferencial) ou bcrypt cost 12
> - Em endpoints de login, rehashar senha no primeiro login após troca
> - Forçar reset de senha de todos os usuários se possível
>
> **Opção 2 — Se usa Argon2id ou bcrypt cost ≥ 10:** ✅ Excelente

---

### 10. JWT está configurado com algoritmo seguro, expiração curta, refresh token e estratégia de revogação?

> *JWT mal configurado é uma das fontes mais comuns de compromisso de auth em apps modernos.*
>
> **Opção 1 — Se algum dos pontos abaixo falha:**
> - **Algoritmo**: rejeitar explicitamente `alg: none`. Preferir RS256/ES256 quando há múltiplos serviços validando tokens; HS256 é aceitável em monolito **desde que o secret seja forte**.
> - **Força do secret (HS256)**: na prática, secret fraco é hoje a fonte mais comum de comprometimento de JWT — mais comum que `alg: none`. Exigências:
>   - **Gerado por CSPRNG** (`crypto.randomBytes(32)`, `openssl rand -base64 64`) — nunca string digitada por humano
>   - **Mínimo 256 bits de entropia real** (32 bytes aleatórios), idealmente 512 bits
>   - **NUNCA** `JWT_SECRET=mysecret`, `JWT_SECRET=changeme`, hash de string conhecida, ou qualquer valor adivinhável
>   - Secret diferente por ambiente (dev ≠ staging ≠ prod)
>   - Rotacionável (com período de overlap aceitando dois secrets durante a rotação)
> - **Verificação estrita**: bibliotecas como `jsonwebtoken` permitem passar `algorithms: ['HS256']` no `verify` — sempre passar a allowlist explícita; nunca deixar a lib aceitar qualquer algoritmo do header.
> - **Expiração curta do access token**: 5-15 minutos. Não usar tokens válidos por dias.
> - **Refresh token**: armazenado server-side (banco/Redis) com possibilidade de revogação. Rotação a cada uso (refresh token rotation). Detectar reuso de refresh token revogado (= sinal de comprometimento → invalidar família inteira).
> - **Revogação**: lista de denúncia (blocklist) curta para access tokens em casos de logout/comprometimento, OU access token tão curto que a janela de risco seja aceitável.
> - **Claims obrigatórias**: `iss`, `aud`, `exp`, `iat`, `sub`. Validar todas no verify.
> - **Nunca colocar dados sensíveis no payload** — JWT é base64, não é criptografado.
>
> **Opção 2 — Se JWT está configurado corretamente:** ✅ Excelente

---

### 11. Comparações de valores secretos (tokens, hashes, HMAC, API keys) usam função tempo-constante (`crypto.timingSafeEqual` ou equivalente)?

> *Comparações com `===`, `==`, ou `!==` vazam informação por timing — o operador retorna `false` no primeiro byte diferente, então um atacante medindo latência pode descobrir o token byte a byte.*
>
> **Onde aplicar (importante):**
> - ✅ Comparação de API keys (header `Authorization` customizado)
> - ✅ Comparação de password reset tokens
> - ✅ Comparação de session IDs antes de buscar no banco
> - ✅ Validação de HMAC de webhooks (mais explorável de todos — atacante controla o input)
> - ✅ Comparação de magic link tokens
> - ✅ Comparação de família de refresh token
> - ✅ Comparação de CSRF tokens (double-submit)
>
> **Onde NÃO precisa:**
> - ❌ Comparação de senha (use `bcrypt.compare` ou `argon2.verify` — já são constantes)
> - ❌ Comparação de usernames/emails (não são segredo)
> - ❌ Comparação de IDs de recursos (públicos)
>
> **Opção 1 — Se há `===` em comparações de tokens/HMAC/API keys:**
> - Implementar helper:
>   ```typescript
>   import crypto from 'crypto';
>   export function safeEqual(a: string, b: string): boolean {
>     if (a.length !== b.length) return false;
>     return crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b));
>   }
>   ```
> - Substituir comparações nos pontos listados acima
> - **Cuidado com short-circuit em length**: `Buffer.from(a).length !== Buffer.from(b).length` antes do `timingSafeEqual` é necessário (a função joga erro se tamanhos diferirem), mas a verificação de length em si JÁ vaza informação. Solução completa: comparar **hashes de tamanho fixo** (SHA-256) em vez dos valores crus — assim o tamanho é sempre o mesmo independente do segredo.
> - Equivalentes em outras linguagens: Python `hmac.compare_digest`, Go `subtle.ConstantTimeCompare`, PHP `hash_equals`, Java `MessageDigest.isEqual`.
>
> **Opção 2 — Se todas as comparações de secretos são tempo-constante:** ✅ Excelente

---

### 12. Existe proteção contra brute force em endpoints de autenticação (rate limit + account lockout)?

> **Opção 1 — Se não:**
> - Adicionar rate limit específico em `/login`, `/register`, `/forgot-password`, `/reset-password`
> - Implementar account lockout: após N tentativas falhas, bloquear conta por X minutos
> - Logar tentativas suspeitas
> - Considerar CAPTCHA após N falhas
>
> **Opção 2 — Se sim, rate limit + lockout ativos:** ✅ Excelente

---

### 13. Segredos (API keys, tokens, senhas de banco) estão FORA do código versionado?

> **Opção 1 — Se sim, há segredos no código:**
> - **CRÍTICO — qualquer segredo já commitado está comprometido para sempre**
> - Mover para `.env`
> - Adicionar `.env` ao `.gitignore`
> - Rodar `gitleaks detect --source . -v` em todo o histórico
> - **Rotacionar (gerar novos) todos os segredos que foram commitados**
> - Configurar pre-commit hook com gitleaks/trufflehog
>
> **Opção 2 — Se não, tudo via env/cofre de segredos:** ✅ Excelente
>
> 🔗 *Relacionado: item 6 (validação de envs no boot). Aqui garantimos que o segredo está fora do código; lá garantimos que ele existe quando a aplicação sobe.*

---

### 14. Queries ao banco usam parâmetros (não concatenação de string)?

> *Vale para SQL, NoSQL, comandos shell, paths.*
>
> **Opção 1 — Se há concatenação ou interpolação direta:**
> - **CRÍTICO — risco de injeção**
> - Substituir por queries parametrizadas (`$1`, `?`, ou ORM)
> - Para MongoDB: validar tipo de entrada antes (object vs string) ou usar `express-mongo-sanitize`
> - Para shell: usar libs com array de args, não string concatenada
>
> **Opção 2 — Se tudo é parametrizado:** ✅ Excelente

---

### 15. Requisições HTTP feitas pelo servidor a partir de URLs de fonte externa (usuário OU API terceira) são protegidas contra SSRF?

> *Aplica-se a: webhooks configurados pelo usuário, "fetch URL preview", import de arquivo via URL, OAuth callbacks, qualquer feature em que o backend faz `fetch(userInput)`.*
>
> ⚠️ **Também aplica-se quando a URL vem de API externa, não diretamente do usuário** (peer review TrendScope 2026-05-18). Exemplos:
> - **Scraping de resultados de busca:** API Serper/Bing/SerpAPI retorna URLs; servidor faz `fetch(item.link)` pra extrair OG image. Se a API for comprometida (ou um resultado for envenenado por SEO), pode retornar URL apontando pra metadata cloud (`169.254.169.254`) ou IP privado.
> - **Webhooks de parceiro:** payload do parceiro contém URL; servidor faz fetch dela. Parceiro comprometido = atacante interno via supply chain.
> - **Feeds RSS / sitemap import:** o feed XML lista URLs que o servidor vai puxar.
> - **OpenGraph preview de links compartilhados em chat/comentários:** mesmo padrão.
>
> A defesa é a **mesma** do SSRF clássico — allowlist de esquemas, blocklist de IPs privados, DNS rebinding guard, redirect:manual. A pergunta-teste muda: "**alguma URL que o servidor fetcha veio de fora do código?** Se sim, está validada antes do fetch?". A maioria dos guias de SSRF foca em `userInput` direto e perde esse vetor.
>
> *Ataque clássico em cloud: atacante manda URL `http://169.254.169.254/latest/meta-data/iam/security-credentials/` (AWS metadata) e seu servidor responde com credenciais IAM. Mesma ideia em GCP/Azure.*
>
> **Opção 1 — Se o servidor faz requisições a URLs fornecidas pelo cliente sem proteção:**
> - **CRÍTICO em ambiente cloud** — vazamento de credenciais IAM em segundos
> - Allowlist de hosts/domínios permitidos (preferencial sobre blocklist)
> - Resolver DNS antes da requisição e bloquear:
>   - Faixas privadas: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
>   - Loopback: `127.0.0.0/8`, `::1`
>   - Link-local: `169.254.0.0/16` (inclui metadata cloud)
>   - IPv6 equivalentes (`fc00::/7`, `fe80::/10`)
> - **DNS rebinding / TOCTOU — passo a passo correto** (a parte mais sutil do SSRF):
>   1. Resolver o hostname **UMA ÚNICA VEZ** (`dns.lookup`)
>   2. Validar o IP resolvido contra a blocklist de faixas privadas/loopback/link-local
>   3. Fazer a requisição HTTP **direto ao IP validado**, NÃO ao hostname
>   4. Passar o hostname original no header `Host` (necessário para TLS SNI e virtual hosts)
>   - ❌ **Padrão errado**: `resolve(host) → validar IP → fetch(host)` — entre a validação e o fetch, o DNS pode mudar (atacante controla o DNS, retorna IP público na primeira consulta e `169.254.169.254` na segunda). É TOCTOU clássico.
>   - ✅ **Padrão certo**: resolver, validar, conectar ao IP fixo. Use libs como `ssrf-req-filter` (Node) ou agentes HTTP customizados que aceitam IP pré-resolvido.
> - Desabilitar redirects automáticos ou validar cada redirect (a validação precisa rodar de novo a cada hop — atacante pode usar `302 → http://169.254.169.254`)
> - Em AWS/GCP: usar IMDSv2 (exige token), bloquear IMDSv1 a nível de instância
> - Timeout curto e tamanho máximo de resposta
>
> **Opção 2 — Se há proteção contra SSRF:** ✅ Excelente

---

### 16. Não há desserialização insegura nem renderização de templates com input do usuário (SSTI)?

> *Categorias diferentes, mas mesmo padrão: input do usuário sendo interpretado como código/estrutura confiável.*
>
> **Opção 1 — Se algum destes existe:**
> - **Desserialização insegura**:
>   - Python: `pickle.loads(userInput)` — RCE trivial
>   - PHP: `unserialize(userInput)` — gadget chains conhecidas
>   - Java: `ObjectInputStream` — clássico
>   - Node: `eval`, `Function`, `vm.runInNewContext` com input do usuário
>   - YAML inseguro: `yaml.load` (Python) sem `SafeLoader`; `js-yaml` `load` em vez de `safeLoad`
>   - → Substituir por formatos seguros (JSON) ou variantes safe da lib
> - **SSTI (Server-Side Template Injection)**:
>   - Jinja2/Twig/Handlebars/EJS recebendo input do usuário como TEMPLATE (não como variável)
>   - Padrão errado: `render(userInput, context)`
>   - Padrão certo: `render(templateFixo, { userInput })`
>   - Atacante consegue `{{ 7*7 }}` → 49 → RCE em vários engines
> - **Auditar** todos os pontos de `eval`, `exec`, `compile`, render dinâmico de template
>
> **Opção 2 — Se não há desserialização insegura nem SSTI:** ✅ Excelente

---

### 17. Em projetos serverless / multi-entrypoint, o caminho de produção é o MESMO que o caminho auditado, ou existe um segundo handler (Vercel function, Netlify function, edge worker, Lambda) que duplica a lógica e bypassa as defesas?

> *Pergunta-teste: "Quando audito `server/boot.ts` (ou `app.ts`, `main.ts`) e confirmo que tem CORS, headers, body limit, rate limit — esse arquivo é de fato o que processa as requisições em produção? Ou o `vercel.json` / `netlify.toml` / `serverless.yml` aponta para um handler separado em `api/`, `functions/`, `pages/api/` que reimplementa a rota sem chamar o middleware?"*
>
> *Bug clássico de apps híbridos Vite + Vercel, Next.js com Pages Router + App Router, Remix + Lambda, ou qualquer setup onde o dev local roda um servidor "completo" e a produção roda funções serverless. O desenvolvedor mantém duas implementações por conveniência. As defesas perimetrais vivem só no boot completo. Em produção, o tráfego entra pelo handler simplificado e não toca nenhuma defesa. Auditor que lê só `boot.ts` confirma tudo verde — e está errado.*
>
> **Opção 1 — Se há duplicação de handler entre dev e produção:**
> - **CRÍTICO** — anula CORS, headers de segurança, body limit, rate limit, error handler centralizado para o tráfego real
> - Localizar o entrypoint de produção: ler `vercel.json` (chave `functions`, `rewrites`), `netlify.toml`, `serverless.yml`, `Procfile`, ou a convenção da plataforma (`api/*.ts` no Vercel, `pages/api/*` no Next, `functions/*` no Netlify)
> - Seguir o handler até o ponto onde middleware é aplicado
> - **Confirmar que é o mesmo módulo do dev.** Se houver fork, escolher um dos dois caminhos:
>   - **Consolidar**: o handler serverless apenas importa e re-exporta `boot.ts` via adapter da plataforma (`hono/vercel`, `next-adapter`, etc.). Toda lógica de defesa fica em um lugar só.
>   - **Aplicar middleware no fork**: se a consolidação for impossível, aplicar exatamente o mesmo middleware no handler serverless e ter teste que valida paridade
> - Adicionar teste de produção que valida headers/CORS no domínio publicado (`curl -I https://app.exemplo.com/api/...`) — não confiar em "audita o `boot.ts`"
> - 🔗 *Relacionado: itens 32 (headers), 32 (CSP), 41 (CORS), 35 (rate limit). Este item é a meta-pergunta que valida se todos eles aplicam ao tráfego real.*
>
> **Opção 2 — Se há um único entrypoint OU se os múltiplos entrypoints aplicam o mesmo middleware:** ✅ Excelente

---

### 18. Cookies de sessão/refresh têm `httpOnly`, `secure`, `sameSite` configurados?

> **Opção 1 — Se algum está faltando:**
> - `httpOnly: true` — previne acesso via JavaScript (mitiga XSS)
> - `secure: true` em produção — só envia via HTTPS
> - `sameSite: 'strict'` (preferencial) ou `'lax'` — mitiga CSRF
> - `path` restrito ao mínimo necessário
> - `maxAge` definido
>
> **Opção 2 — Se todos configurados:** ✅ Excelente

---

## 🟠 ESSENCIAIS — sem isso, não é portfólio sênior

### 19. Existem testes automatizados para os caminhos críticos (auth, autorização, validação)?

> *Não precisa de 100% de cobertura. Precisa de testes que provem que as proteções funcionam.*
>
> **Opção 1 — Se não há testes ou só testes do caminho feliz:**
> - Adicionar Vitest + Supertest (Node) ou pytest + httpx (Python)
> - Cobrir cenários adversariais:
>   - Acesso sem token → 401
>   - Token inválido/expirado → 401
>   - Usuário A acessando recurso de B → 403
>   - Payload inválido → 400 com detalhes
>   - Payload válido → 200/201 com retorno esperado
> - Mínimo 15-20 testes bem escolhidos
>
> **Opção 2 — Se há testes cobrindo caminhos adversariais:** ✅ Excelente

---

### 20. Existem testes específicos para os cenários de exploração ADVERSARIAL identificados na modelagem de ameaças?

> *Diferença sutil em relação ao item 19: o 18 garante que existam testes negativos (token inválido, autorização negada). Este item vai além — para cada ameaça documentada no `THREAT_MODEL.md`, existe pelo menos um teste automatizado que prova que a mitigação funciona contra o cenário de ataque específico.*
>
> *Pergunta-teste: "Para cada linha do THREAT_MODEL com status ✅ (mitigado), existe um teste que dispara o ataque e verifica que ele é bloqueado?"*
>
> **Opção 1 — Se há ameaças listadas sem teste correspondente:**
> - Para cada ameaça no THREAT_MODEL, escrever ao menos um teste que:
>   1. **Configura o cenário de ataque** (atacante autenticado, payload malicioso, etc.)
>   2. **Executa a tentativa de exploração**
>   3. **Verifica que a resposta é o erro esperado** (403, 401, 400)
>   4. **Verifica que o estado do banco/sistema NÃO mudou**
> - Exemplos concretos:
>   - "Atacante autenticado tenta editar recurso de outro usuário → 403 + recurso não muda"
>   - "Refresh token revogado tenta ser usado novamente → 401 + toda família revogada"
>   - "Socket emite `board:join` com `user._id` de outra pessoa → não entra na room"
>   - "Cliente envia `amount` no body do checkout → servidor ignora e usa preço do banco"
> - Esses testes documentam a segurança melhor que qualquer comentário — ficam como prova viva de que a mitigação existe e funciona
> - Quando uma ameaça é mitigada, o teste é o "selo" da mitigação
>
> **Opção 2 — Se cada ameaça do THREAT_MODEL tem teste correspondente:** ✅ Excelente

---

### 21. Existe CI/CD com checks automáticos a cada PR?

> **Opção 1 — Se não:**
> - Criar `.github/workflows/ci.yml`
> - Steps mínimos: lint, type-check, testes, `npm audit`, `gitleaks`
> - Opcional avançado: Semgrep, Trivy (se usa Docker)
> - Bloquear merge se algum step falhar (branch protection rules)
> - Adicionar badge de status no README
>
> **Opção 2 — Se há CI rodando:** ✅ Excelente

---

### 22. Tratamento de erro é centralizado (middleware global) em vez de try/catch repetido em cada controller?

> **Opção 1 — Se cada controller tem try/catch genérico:**
> - Criar classes de erro: `NotFoundError`, `ForbiddenError`, `ValidationError`, `UnauthorizedError`
> - Criar middleware de erro global que mapeia classe → status HTTP + mensagem
> - Refatorar controllers para `throw` em vez de `res.status().json()`
> - Garantir que stack trace nunca vaza para resposta em produção
>
> **Opção 2 — Se há error handler centralizado:** ✅ Excelente

---

### 23. Os controllers usam `throw` (não `res.status().json()`) para erros operacionais, exercitando o middleware global de erro?

> *Pergunta-teste: "Se eu remover o middleware global de erro, os endpoints param de retornar 4xx corretos?" — se a resposta for "não, eles continuam funcionando", o middleware está sendo bypassed pelos controllers e o item 22 só existe no papel.*
>
> **Opção 1 — Se controllers ainda retornam erros com `res.status().json()`:**
> - Refatorar para `throw new ForbiddenError(...)`, `throw new NotFoundError(...)`, etc.
> - Adicionar wrapper `asyncHandler` que captura promise rejections, OU usar Express 5 (captura nativamente)
> - Remover try/catch genéricos dos controllers — eles ofuscam a lógica e desviam do handler global
> - Manter try/catch apenas onde há **lógica de recuperação específica** (ex: tentar fallback antes de propagar) — raro
> - Resultado esperado: ~30% menos código nos controllers, respostas padronizadas, stack trace nunca vaza, ID de correlação automático
>
> **Opção 2 — Se controllers usam `throw` e middleware faz o trabalho:** ✅ Excelente

---

### 24. Operações sensíveis estão protegidas contra race conditions / TOCTOU em lógica de negócio?

> *Padrão clássico: "validar saldo → debitar saldo" em dois passos. Dois requests simultâneos passam pela validação antes de qualquer um decrementar → ambos passam, atacante gasta R$ 100 quando só tinha R$ 60. Mesma classe de bug: usar cupom duas vezes simultaneamente, resgatar prêmio limitado em paralelo, exceder quota fazendo N requests ao mesmo tempo, votar duas vezes.*
>
> *Em sistemas financeiros e gamificação, é causa frequente de incidentes — e não aparece em testes sequenciais.*
>
> **Opção 1 — Se há padrão "leia → valide → escreva" sem proteção contra concorrência:**
> - **Transactions com isolation level adequado**:
>   - Postgres/MySQL: `SELECT ... FOR UPDATE` dentro de transaction (pessimistic locking)
>   - MongoDB: `findOneAndUpdate` com condição na própria query (atomic) em vez de `findOne` + `save`
> - **Optimistic locking**: campo `version` incrementado a cada update; query falha se versão mudou; retry
> - **Operações atômicas no banco**: `UPDATE accounts SET balance = balance - 100 WHERE id = X AND balance >= 100` — uma só query, decisão atômica, sem janela de race
> - **Unique constraint estratégico**: para "usar cupom uma vez", `UNIQUE(user_id, coupon_id)` impede duplicação independente do código de aplicação
> - **Idempotency key**: para operações externas (cobrança, envio de email), garantir que retry/duplo-click não duplica efeito
> - **Distributed locks**: Redis (Redlock) ou similar para operações que cruzam múltiplos recursos — usar com cuidado, locks distribuídos têm armadilhas
> - **Teste com concorrência real**: rodar N requests paralelos contra o endpoint e verificar invariantes — esse bug NÃO aparece em testes sequenciais
>
> **Opção 2 — Se operações sensíveis são atômicas/transacionais:** ✅ Excelente

---

### 25. Logging é estruturado (JSON) e padronizado em 100% do código?

> *Pergunta-teste: "Há `console.log` ou `console.error` espalhados pelo código?"*
>
> **Opção 1 — Se há mistura de logger + console.log:**
> - Adotar Pino, Winston, ou structlog (Python)
> - Substituir todos os `console.*` por chamadas ao logger
> - Logar em JSON, com `level`, `timestamp`, `message`, contexto adicional
> - Em produção, redirecionar para SIEM/log aggregator
>
> **Opção 2 — Se logging é 100% estruturado:** ✅ Excelente

---

### 26. Logs estão LIMPOS de dados sensíveis (senhas, tokens, PII, números de cartão)?

> **Opção 1 — Se logs contêm PII ou segredos:**
> - **CRÍTICO se em produção — logs com PII são vetor de vazamento**
> - Auditar todos os pontos de log
> - Mascarar campos sensíveis automaticamente (redaction nas configs do logger)
> - Nunca logar `req.body` cru em endpoints de login/registro
> - Logar IDs e ações, não conteúdo
>
> **Opção 2 — Se logs são limpos:** ✅ Excelente

---

### 27. Existe documentação de API (Swagger/OpenAPI ou `API.md` completo)?

> **Opção 1 — Se não há docs ou só README genérico:**
> - Gerar OpenAPI a partir do código (zod-to-openapi, tsoa) — preferencial
> - Ou criar `docs/API.md` listando todos os endpoints: método, path, payload, response, códigos de erro
> - Expor `/api/docs` com Swagger UI em dev
>
> **Opção 2 — Se há documentação completa e atualizada:** ✅ Excelente

---

### 28. Existe documento de modelagem de ameaças (`THREAT_MODEL.md`) no repositório?

> *Não precisa ser longo. Precisa existir e ser honesto.*
>
> **Opção 1 — Se não existe:**
> - Criar `docs/THREAT_MODEL.md` com seções:
>   - Ativos protegidos (dados, credenciais, disponibilidade)
>   - Atores de ameaça (script kiddie, usuário malicioso, ex-funcionário)
>   - Superfícies de ataque (endpoints públicos, uploads, terceiros)
>   - STRIDE aplicado: que vulnerabilidades de cada categoria poderiam existir aqui
>   - Mitigações implementadas e mitigações planejadas
>
> **Opção 2 — Se existe e está atualizado:** ✅ Excelente

---

### 29. Operações com cota/limite usam mecanismo atômico (`findOneAndUpdate` com filtro, transaction, ou unique constraint) em vez do padrão `count → if → write`?

> *O padrão `count → if → create` é a face mais comum de race condition em apps SaaS. Sempre que houver "verificar limite + executar ação", existe janela de race que rate limiter por IP NÃO fecha — é race no banco, não no front.*
>
> *Exemplos clássicos: "free tier limitado a 10 projetos" → atacante manda 20 requests simultâneos e cria 20 projetos; "cupom limitado a 100 usos" → 200 usos válidos em paralelo; "1 voto por usuário" → 5 votos em paralelo.*
>
> **Opção 1 — Se há padrão `count → if → write` em qualquer ponto de cota/quota:**
> - **RACE CONDITION** — N requests paralelos passam pela verificação simultaneamente antes de qualquer um incrementar
> - Substituir por **operação atômica**:
>   - MongoDB: `findOneAndUpdate({ userId, count: { $lt: LIMITE } }, { $inc: { count: 1 } })` — retorna `null` se já no limite, sem race
>   - SQL: `UPDATE quotas SET count = count + 1 WHERE user_id = X AND count < LIMITE RETURNING ...` — decisão atômica em uma query
> - **Unique constraint estratégico** quando aplicável: `UNIQUE(userId, date, slot)` para slots agendados, `UNIQUE(userId, couponId)` para uso único de cupom — o banco recusa a duplicata independente da lógica de aplicação
> - **Transaction com lock** (último recurso por reduzir throughput): `BEGIN; SELECT FOR UPDATE; ...; COMMIT;` ou Mongo transactions
> - **Testar com concorrência real**: 50 requests paralelos contra o endpoint — se passa, há race; teste sequencial NÃO pega
> - 🔗 *Relacionado: item 24 é a regra geral de race conditions; este item é a aplicação cirúrgica em cotas/limites, padrão tão comum que merece checagem dedicada.*
>
> **Opção 2 — Todas as operações de cota são atômicas:** ✅ Excelente

---

### 30. Existe `SECURITY.md` no repositório?

> *Padrão GitHub — aparece automaticamente na aba "Security".*
>
> **Opção 1 — Se não existe:**
> - Criar `SECURITY.md` com:
>   - Como reportar uma vulnerabilidade (email, formulário, GitHub Security Advisories)
>   - Versões suportadas
>   - Política de resposta (SLA de resposta inicial)
>   - Reconhecimento (hall of fame, se aplicável)
>
> **Opção 2 — Se existe:** ✅ Excelente

---

### 31. README é honesto sobre o que está e o que não está implementado?

> *Pergunta-teste: "O README promete coisas que o código não entrega?"*
>
> **Opção 1 — Se README é marketing acima da realidade:**
> - Reescrever seção de segurança/arquitetura para refletir o que existe DE FATO
> - Listar trade-offs feitos e por quê
> - Listar o que NÃO está implementado (e por quê)
> - Sênior valoriza honestidade mais que promessa
>
> **Opção 2 — Se README é factual e honesto:** ✅ Excelente

---

### 32. Headers de segurança HTTP estão configurados (Helmet ou equivalente)?

> *Teste em https://securityheaders.com*
>
> **Opção 1 — Se Helmet não está, ou só com defaults:**
> - Adicionar Helmet
> - Configurar CSP restritiva (`defaultSrc: 'none'`, allowlist explícita)
> - `frameAncestors: 'none'`, `objectSrc: 'none'`, `baseUri: 'self'`
> - HSTS com `maxAge` longo, `includeSubDomains`, `preload`
> - `Referrer-Policy: no-referrer` ou `strict-origin-when-cross-origin`
> - Testar nota A+ no securityheaders.com
>
> **Opção 2 — Se nota ≥ A em scanner externo:** ✅ Excelente

---

### 33. A CSP em produção é estrita (sem `'unsafe-inline'` ou `'unsafe-eval'` em `script-src`), ou foi relaxada para fazer o framework funcionar?

> *`'unsafe-inline'` em `script-src` anula a principal proteção XSS do CSP. Frameworks modernos (Vite, Next.js, Nuxt) suportam nonces ou hashes — relaxar pra `'unsafe-inline'` é atalho que vira teatro de segurança.*
>
> *Pergunta-teste: "Se um atacante conseguir injetar `<script>alert(1)</script>` em qualquer parte do DOM, o navegador executa? Se sim, minha CSP de `script-src` não está protegendo de nada."*
>
> **Opção 1 — Se `script-src` tem `'unsafe-inline'` ou `'unsafe-eval'`:**
> - Verificar se realmente precisa — a maioria dos casos é resolvível com nonces (Vite tem plugin, Next.js suporta nativo via middleware) ou hashes SHA-256 listados na CSP
> - Para **CSS** (`style-src 'unsafe-inline'`), trade-off é mais aceitável: atacante via XSS conseguiria estilizar (defacement), não executar código. Comum em React + Tailwind/CSS-in-JS.
> - Para **script**, nunca é aceitável em produção sem mitigação adicional (Trusted Types, isolamento de origem, sandboxed iframes)
> - Se for inevitável (mock, demo, dependência legada), **documentar em ADR** com:
>   - Por que o framework exige
>   - Quais defesas em profundidade compensam (`frame-ancestors`, `object-src 'none'`, `base-uri`, `form-action`, Permissions-Policy)
>   - Quando expira (gatilho objetivo)
> - Validar nota em `securityheaders.com` — `'unsafe-inline'` derruba pra B no mínimo
> - Ferramentas: `vite-plugin-csp-guard` (Vite), `@next/csp` (Next.js), middleware no edge do Vercel/Cloudflare que injeta nonce
>
> **Opção 2 — Se CSP em produção não tem `'unsafe-*'` em `script-src` (ou tem com mitigação documentada):** ✅ Excelente

---

## 🟡 QUALIDADE — visível para quem audita

### 34. IDs nas validações têm formato estrito (regex/UUID)?

> *`z.string().min(1)` aceita qualquer string e pode mascarar erros.*
>
> **Opção 1 — Se schemas usam `string().min(1)` para IDs:**
> - Para MongoDB ObjectId: `z.string().regex(/^[a-f0-9]{24}$/)`
> - Para UUID: `z.string().uuid()`
> - Para IDs numéricos: `z.number().int().positive()` ou `z.coerce.number()...`
> - Aplicar em todos os schemas de path params e body
>
> **Opção 2 — Se IDs têm formato validado:** ✅ Excelente

---

### 35. Existe paginação em todas as listagens que podem crescer?

> **Opção 1 — Se há endpoints `GET /resource` retornando array completo:**
> - Implementar paginação por cursor (preferencial) ou offset
> - Definir limite máximo no servidor (ex: `limit ≤ 100`)
> - Retornar metadados: `nextCursor`, `hasMore`, `total` (se viável)
> - Aplicar em todos os endpoints de listagem
>
> **Opção 2 — Se todas as listagens são paginadas:** ✅ Excelente

---

### 36. Endpoints sensíveis têm rate limiting POR USUÁRIO (não só por IP)?

> *Rate limit por IP falha atrás de NAT corporativo (um IP = mil usuários).*
>
> **Opção 1 — Se rate limit é só por IP:**
> - Para endpoints autenticados: chave de rate limit = `userId`
> - Para endpoints públicos: por IP mesmo, mas considerar IP do CDN/proxy
> - `trust proxy` configurado corretamente para extrair IP real
>
> ⚠️ **Nuance específica de serverless (peer review TrendScope 2026-05-18):**
> em Vercel/Lambda/Cloud Run com rate limit **in-memory** (`Map`, objeto local), cada cold start zera o contador. Não é só o problema de "escala horizontal" (várias instâncias com Maps próprios) — é o problema **mais imediato** de "janela de burst pós-cold-start":
> - Instância dorme após ~5min de inatividade no free tier
> - Atacante que monitora cold start (ex: ping `/health` periodicamente sem requests reais até instância dormir, depois dispara N requests imediatamente após primeira chamada) consegue burst window
> - Acontece em deploy **single-instance** também, não só horizontal
>
> **Pergunta-teste serverless:** "Se eu derrubar a instância agora, um atacante tem N segundos de burst ilimitado até o próximo cold start estabilizar os contadores?"
>
> Quando rate limit in-memory em serverless é aceito como dívida (custo, MVP, demo), **documentar explicitamente em ADR**: "rate limit é best effort, não garantido — janela de burst pós-cold-start é conhecida". Migração para Redis/KV compartilhado é a solução real.
>
> **Opção 2 — Se há rate limit por usuário com storage persistente (Redis/KV/banco) OU se a limitação em serverless está documentada como dívida consciente:** ✅ Excelente

---

### 37. Quando a aplicação está atrás de proxy/load balancer/CDN, o IP do cliente usado para rate limit, audit log e bloqueios é confiável (não-fakeável por header de origem)?

> *Pergunta-teste: "Eu mando `curl -H 'X-Forwarded-For: 1.2.3.4' https://meusite/login` várias vezes — o rate limit conta cada request como vindo de IP diferente?" Se sim, o rate limit é teatro: atacante drible em 1 linha de bash.*
>
> *Esse é o bug mais comum em apps que migram de "sem proxy" para "com proxy" sem reauditar a função que extrai IP. O padrão `request.META.get('HTTP_X_FORWARDED_FOR', '').split(',')[0]` aparece em metade dos middlewares de rate limit copiados de Stack Overflow — e é quase sempre vulnerável.*
>
> **Opção 1 — Se o código lê `X-Forwarded-For` cru sem saber quantos proxies confiáveis existem:**
> - **Atacante drible o rate limit em 1 linha de curl**
> - Correção depende do setup:
>   - **Caddy/Nginx confiável injetando `X-Real-IP`**: ler diretamente `X-Real-IP` (proxy sobrescreve, cliente não consegue forjar). Configurar no Caddyfile: `header_up X-Real-IP {remote_host}`.
>   - **Múltiplos proxies em cadeia** (Cloudflare → ALB → app): contar do final do `X-Forwarded-For` para trás N posições (N = número de proxies confiáveis). NUNCA confiar no primeiro IP da lista.
>   - **Cloudflare na frente**: usar `CF-Connecting-IP` (Cloudflare nunca o injeta se vier do cliente).
> - Em Django: configurar `SECURE_PROXY_SSL_HEADER` apenas se confiar no header; validar `USE_X_FORWARDED_HOST = False` se não usa.
> - Em Express: `app.set('trust proxy', N)` com N correto — não `true` (confia em qualquer proxy) nem `false` (ignora completamente).
> - Em FastAPI/Starlette: usar `ProxyHeadersMiddleware` configurado com `trusted_hosts`.
> - **Teste de validação obrigatório**: mandar header forjado e verificar que ele é IGNORADO (não que ele seja aceito).
> - 🔗 *Relacionado: item 36 (rate limit por usuário). Sem 36, qualquer rate limit por IP é placebo atrás de proxy.*
>
> **Opção 2 — IP confiável vem do header certo, ou de `REMOTE_ADDR` direto sem proxy:** ✅ Excelente

---

### 38. Política de senha é forte (mínimo 8, idealmente verificada contra senhas vazadas)?

> **Opção 1 — Se senha mínima é < 8 ou sem checagem de vazamento:**
> - Aumentar mínimo para 10 caracteres (NIST recomenda 8, idealmente mais)
> - Não exigir caracteres especiais (NIST recomenda contra) — comprimento importa mais
> - Integrar HIBP API (Have I Been Pwned) com k-anonymity: bloqueia senhas vazadas conhecidas sem enviar a senha real
>
> **Opção 2 — Se política é forte:** ✅ Excelente

---

### 39. Existem health checks separando "vivo" (liveness) de "pronto" (readiness)?

> **Opção 1 — Se há só um health check genérico ou nenhum:**
> - `GET /health/live` — processo está respondendo (sempre 200 se app não travou)
> - `GET /health/ready` — banco conecta, dependências respondem (200 se pronto para receber tráfego)
> - Documentar e usar nos probes do Kubernetes/Render/Railway
>
> **Opção 2 — Se há liveness + readiness:** ✅ Excelente

---

### 40. Não há queries N+1 nas listagens principais?

> *Pergunta-teste: "Para listar 100 itens, quantas queries ao banco são feitas?"*
>
> **Opção 1 — Se há `forEach`/`Promise.all` fazendo query por item:**
> - Refatorar usando `$lookup`/JOIN/aggregate
> - Carregar relações em batch (`.populate` no Mongoose, `JOIN` no SQL)
> - Medir queries por requisição (logger + APM em dev)
>
> **Opção 2 — Se queries são otimizadas:** ✅ Excelente

---

### 41. Tratamento de erro nunca vaza stack trace ou detalhes internos em produção?

> **Opção 1 — Se em produção o cliente vê stack trace ou nomes de tabela:**
> - **CRÍTICO se em produção — vazamento de informação**
> - Verificar `NODE_ENV === 'production'` no error handler
> - Em produção: mensagem genérica + ID de correlação
> - Em dev: detalhes completos
> - Logar erro completo internamente, retornar pouco ao cliente
>
> **Opção 2 — Se em produção respostas são limpas:** ✅ Excelente

---

### 42. CORS está configurado com allowlist específica (não wildcard `*`)?

> **Opção 1 — Se CORS usa `*` ou é permissivo:**
> - Configurar `origin` como função que valida contra allowlist
> - Allowlist via env var (`CLIENT_URL=https://app.exemplo.com,https://www.exemplo.com`)
> - `credentials: true` apenas se realmente necessário
> - Nunca combinar `origin: '*'` com `credentials: true`
>
> **Opção 2 — Se CORS é estrito:** ✅ Excelente

---

### 43. Há proteção contra CSRF para endpoints que aceitam cookies de autenticação?

> *Se usa apenas `Authorization: Bearer` no header, risco é menor. Se usa cookies, atenção.*
>
> **Opção 1 — Se usa cookies sem proteção CSRF:**
> - Opção mais simples: `sameSite: 'strict'` no cookie
> - Validar header `Origin` no servidor para endpoints sensíveis
> - Para apps mais críticos: double-submit token ou anti-CSRF token
>
> **Opção 2 — Se usa Bearer token OU cookies com proteção:** ✅ Excelente

---

### 44. Dependências estão atualizadas e auditadas?

> **Opção 1 — Se `npm audit` mostra vulnerabilidades altas/críticas:**
> - Rodar `npm audit fix` (ou equivalente em outras linguagens)
> - Atualizar dependências major com testes
> - Habilitar Dependabot ou Renovate no repositório
> - Configurar `npm audit` como step bloqueante no CI
>
> **Opção 2 — Se audit está limpo:** ✅ Excelente

---

## 🟢 DIFERENCIAÇÃO — separa sênior bom de sênior excelente

### 45. Há correlation ID propagado por requisição (HTTP, logs, eventos assíncronos)?

> **Opção 1 — Se não:**
> - Gerar `requestId` (UUID) em middleware no início de cada request
> - Anexar ao logger (`logger.child({ requestId })`)
> - Retornar no header `X-Request-ID` da resposta
> - Propagar para jobs assíncronos, sockets, chamadas externas
> - Permite rastrear uma ação completa nos logs
>
> **Opção 2 — Se há correlation ID end-to-end:** ✅ Excelente

---

### 46. Existem métricas/telemetria expostas (Prometheus, OpenTelemetry, APM)?

> **Opção 1 — Se não há métricas:**
> - Adicionar Prometheus (`prom-client`) ou OpenTelemetry
> - Métricas mínimas: contagem de requests, latência (histogram), erros
> - Métricas de negócio: usuários ativos, ações por hora
> - Expor em `/metrics` (protegido)
>
> **Opção 2 — Se há observabilidade:** ✅ Excelente

---

### 47. Operações destrutivas usam soft delete em vez de hard delete?

> **Opção 1 — Se `DELETE` apaga registros para sempre:**
> - Adicionar campo `deletedAt: Date | null` nos schemas
> - Filtrar `deletedAt: null` em todas as queries (ou usar middleware do ORM)
> - Job periódico para hard delete após X dias (LGPD/GDPR — direito de exclusão respeitado)
> - Permite recuperação, auditoria e undo
>
> **Opção 2 — Se há soft delete:** ✅ Excelente

---

### 48. Existe audit log para ações sensíveis (login, logout, convite, exclusão, mudança de permissão)?

> **Opção 1 — Se não há registro de ações sensíveis:**
> - Criar tabela/coleção `audit_logs`
> - Campos: `userId`, `action`, `targetType`, `targetId`, `ip`, `userAgent`, `timestamp`, `metadata`
> - Registrar: login (sucesso e falha), logout, convites, exclusões, mudanças de permissão, reset de senha
> - Retenção configurável (mínimo 90 dias, ideal 1 ano)
> - Imutável (não permitir update/delete por aplicação)
>
> **Opção 2 — Se há audit log:** ✅ Excelente

---

### 49. Existe plano de backup e recuperação documentado?

> **Opção 1 — Se não há `DISASTER_RECOVERY.md`:**
> - Criar `docs/DISASTER_RECOVERY.md`
> - Documentar: RPO (Recovery Point Objective) e RTO (Recovery Time Objective)
> - Frequência de backups (MongoDB Atlas / RDS automático)
> - Procedimento de restauração (passo a passo)
> - Última data em que o procedimento foi testado
> - **Backup não testado não é backup**
> - **Backups criptografados em repouso** (chave separada da chave de produção; comprometer a aplicação não deve dar acesso aos backups)
> - **Armazenamento isolado**: região E conta/projeto cloud diferente da produção. Se a conta de produção for comprometida (ou ransomware criptografar tudo), os backups precisam estar fora do alcance.
> - **Imutabilidade**: object lock / versioning no bucket para que um atacante com credenciais não consiga deletar/sobrescrever backups (defesa anti-ransomware)
> - **Acesso restrito**: princípio do menor privilégio — apenas o pipeline de backup escreve, e apenas o pipeline de restore lê
>
> **Opção 2 — Se há plano testado, com backups criptografados e isolados:** ✅ Excelente

---

### 50. Política de retenção de dados está definida (especialmente para LGPD/GDPR)?

> **Opção 1 — Se não há política definida:**
> - Documentar em `docs/DATA_RETENTION.md`:
>   - O que acontece quando usuário deleta conta
>   - Prazo de hard delete após soft delete
>   - O que é anonimizado vs deletado
>   - Como atender pedidos de exportação (portabilidade — LGPD/GDPR)
>   - Como atender pedidos de exclusão (direito ao esquecimento)
> - Implementar os endpoints/jobs correspondentes
>
> **Opção 2 — Se política existe e está implementada:** ✅ Excelente

---

### 51. Aplicação está containerizada com Dockerfile multi-stage?

> **Opção 1 — Se não há Docker ou só Dockerfile simples:**
> - Dockerfile multi-stage: builder (com toolchain) + runtime (mínimo)
> - Imagem base mínima (distroless, Alpine, ou Chainguard)
> - Rodar como usuário não-root
> - `.dockerignore` excluindo `node_modules`, `.env`, `.git`
> - `docker-compose.yml` para dev local com banco embutido
>
> **Opção 2 — Se há Dockerfile profissional:** ✅ Excelente

---

### 52. Há decisões arquiteturais documentadas (ADRs — Architecture Decision Records)?

> *Sênior documenta por que escolheu X em vez de Y.*
>
> **Opção 1 — Se decisões importantes não estão documentadas:**
> - Criar pasta `docs/adr/`
> - Para cada decisão significativa, criar `ADR-NNN-titulo.md`:
>   - Contexto
>   - Decisão
>   - Alternativas consideradas
>   - Trade-offs
>   - Consequências
> - Exemplos: "Por que JWT e não sessão?", "Por que MongoDB e não PostgreSQL?"
>
> **Opção 2 — Se decisões estão documentadas:** ✅ Excelente

---

## 🎨 FRONTEND — checks específicos do cliente

### 53. Onde o access token (JWT) é armazenado no frontend?

> **Opção 1 — Se está em `localStorage` ou `sessionStorage`:**
> - **Vulnerável a XSS** — qualquer script consegue ler
> - Migrar para: token em memória (Zustand/Redux sem persist) + refresh via cookie httpOnly
> - No reload da página, fazer refresh silencioso para obter novo access token
> - ⚠️ **Atenção CSRF**: o cookie de refresh httpOnly é enviado automaticamente pelo browser, então o endpoint `/refresh` fica vulnerável a CSRF. Mitigar com:
>   - `sameSite: 'strict'` (ou `'lax'` se houver navegação cross-site legítima)
>   - Validar header `Origin`/`Referer` no `/refresh`
>   - Para apps mais expostos: double-submit token ou anti-CSRF token
>   - 🔗 *Ver item 43 — CSRF é a contrapartida do trade-off "cookie httpOnly vs localStorage"*
>
> **Opção 2 — Se está em memória ou cookie httpOnly COM proteção CSRF:** ✅ Excelente

---

### 54. O guard de rota do frontend valida o token de verdade (decode + verificação de expiração + opcionalmente assinatura), ou apenas verifica `if (token)` — permitindo bypass com `localStorage.setItem('token','x')` no console?

> *Pergunta-teste: "Eu abro o console no site em produção, digito `localStorage.setItem('app_token','qualquer_string')` e dou refresh — consigo entrar na área protegida?" Se a resposta for sim, o guard é cosmético.*
>
> *Esse bug é especialmente comum em apps com "modo demo" ou "modo portfólio", onde o autor implementa um shortcut de auth que aceita um token literal hardcoded — e esquece de remover o check frouxo na rota real.*
>
> **Opção 1 — Se o `ProtectedRoute` aceita qualquer string truthy:**
> - **Bypass trivial** — qualquer visitante entra no dashboard sem credencial nenhuma
> - Validar minimamente: decode do JWT, verificar `exp` (expiração) e formato
>   ```typescript
>   import { jwtDecode } from 'jwt-decode';
>   function isTokenValid(token: string | null): boolean {
>     if (!token) return false;
>     try {
>       const payload = jwtDecode<{ exp: number }>(token);
>       return payload.exp * 1000 > Date.now();
>     } catch { return false; }
>   }
>   ```
> - Idealmente: validar contra o backend (chamada `/me` ou `/verify`) no boot da rota protegida
> - **Nunca** comparar token contra string literal hardcoded (`token === 'demo_token'`)
> - Se houver "modo demo" intencional, isolar em rota separada (`/demo`) com dados sintéticos próprios — não usar o `ProtectedRoute` real
> - Adicionar teste E2E: "console.log + localStorage.setItem + navigate(/dashboard) → redireciona pra login"
> - 🔗 *Relacionado: item 53 (onde o token é armazenado). 52 cobre o local; 53 cobre se o guard realmente valida o que está nesse local.*
>
> **Opção 2 — Se o guard valida o token de verdade:** ✅ Excelente

---

### 55. Conteúdo fornecido por usuário é sanitizado antes de renderizar como HTML?

> *Aplica-se a campos como `description`, `bio`, `comment`, qualquer texto renderizado.*
>
> **Opção 1 — Se usa `dangerouslySetInnerHTML` ou parser markdown sem sanitização:**
> - **CRÍTICO — risco de XSS persistente**
> - Adicionar DOMPurify antes de renderizar
> - Para markdown, usar parser que escapa HTML por padrão (`react-markdown` sem `rehype-raw`)
> - Allowlist explícita de tags permitidas
>
> **Opção 2 — Se conteúdo é texto puro ou sanitizado:** ✅ Excelente

---

### 56. CSP está configurada também no servidor que entrega o frontend (Vercel/Netlify/etc)?

> **Opção 1 — Se CSP só está no backend da API:**
> - Configurar headers no `vercel.json`, `_headers` (Netlify), ou Nginx
> - CSP estrita para o frontend também
> - Testar em securityheaders.com com a URL do frontend (não só da API)
>
> **Opção 2 — Se CSP cobre frontend:** ✅ Excelente

---

### 57. Erros do backend são tratados sem expor detalhes técnicos ao usuário?

> **Opção 1 — Se UI mostra stack trace ou mensagens cruas do backend:**
> - Adicionar boundary de erro (React Error Boundary)
> - Mapear códigos HTTP para mensagens amigáveis
> - Logar detalhes técnicos em monitoring (Sentry), mostrar mensagem genérica + correlation ID ao usuário
>
> **Opção 2 — Se erros são tratados com elegância:** ✅ Excelente

---

### 58. Nenhum segredo vaza para o bundle público do frontend (build entregue ao navegador)?

> *Categoria distinta do item 13. O item 13 garante que o segredo não está no **histórico do git**. Este item garante que o segredo não chega ao **navegador do usuário** — são vetores diferentes, e passar no 13 NÃO garante passar no 58.*
>
> *O erro mental clássico: "a chave está no `.env`, e o `.env` está no `.gitignore`, logo está segura". Falso. O `.env` protege contra o repositório, não contra o build. Bundlers de frontend (Vite, Next, CRA) **embutem deliberadamente** no JavaScript público qualquer variável com o prefixo de exposição da ferramenta — e esse JS é baixado por qualquer visitante.*
>
> **Pergunta-teste:** "Rodo `npm run build`, abro a pasta `dist/` (ou `.next/`, `build/`) e faço `grep` recursivo por trechos de chaves reais (`sk_`, `AKIA`, `-----BEGIN`, o valor do `JWT_SECRET`, a senha do banco). Algum aparece? Abro o site publicado, DevTools → aba Sources → algum segredo está no JS ou nos source maps?"
>
> **Vetores a verificar:**
> - **Prefixo de exposição do bundler** — `VITE_*` (Vite), `NEXT_PUBLIC_*` (Next.js), `REACT_APP_*` (CRA), `PUBLIC_*` (SvelteKit/Astro), `EXPO_PUBLIC_*` (Expo). Tudo com esse prefixo é **público por design**. Só pode conter valores não-sensíveis (URL da API, chave **publishable** do Stripe, project ID do Firebase). Uma `VITE_API_SECRET`, `NEXT_PUBLIC_STRIPE_SECRET_KEY` ou `VITE_OPENAI_KEY` é vazamento — chave de servidor exposta a qualquer visitante.
> - **Segredo hardcoded em componente/módulo de frontend** — chave colada direto no `.jsx`/`.tsx`/`.vue`. O item 13 pega isso via git; aqui confirma-se que também vai para o bundle.
> - **Import indevido de código de servidor no cliente** — um módulo `server/` ou `lib/db` importado por engano num componente React arrasta a connection string para o bundle.
> - **Source maps em produção** — `.map` publicados expõem o código-fonte original (e comentários, e às vezes segredos inline). Em build de produção, desligar: Vite `build.sourcemap: false` (default), Next `productionBrowserSourceMaps: false` (default). Confirmar que não foi religado.
> - **Chamada a API de terceiro feita direto do browser com chave secreta** — se o frontend chama OpenAI/SendGrid/etc. diretamente, a chave está no cliente. O padrão correto é o frontend chamar o **seu** backend, que guarda a chave server-side.
>
> **Opção 1 — Se algum segredo de servidor chega ao bundle público:**
> - **CRÍTICO — qualquer visitante extrai a chave abrindo o DevTools**
> - Remover a variável do escopo do bundler: tirar o prefixo de exposição e movê-la para uma env **só de servidor** (sem prefixo), lida apenas no backend
> - Qualquer chamada que precise da chave secreta passa a ir para um endpoint do próprio backend (proxy), nunca direto do browser ao terceiro
> - **Rotacionar imediatamente** toda chave que já foi publicada num bundle — ela está comprometida (alguém pode ter baixado o JS)
> - Desligar source maps de produção, ou hospedá-los em ferramenta de erro privada (Sentry) em vez de servi-los publicamente
> - Adicionar ao CI um passo que faz `grep` de padrões de segredo no diretório de build e falha se encontrar
> - 🔗 *Relacionado: item 13 (segredo fora do git) e item 6 (env validada no boot). Os três cobrem o ciclo de vida do segredo: 13 = não está no repositório · 6 = existe quando o servidor sobe · 58 = não escapa para o navegador.*
>
> **Opção 2 — Se o bundle só contém valores públicos por design (URLs, chaves publishable) e source maps de produção estão desligados:** ✅ Excelente

---

## 🧭 META — disciplina de cobertura

### 59. Toda função que recebe ID externo (path, body, query) valida o formato ANTES de tocar o banco — em **todos** os pontos, não só em alguns?

> *Esse item é o "olhar lateral" do item 34. O 34 pergunta "schemas têm regex/UUID em vez de `string().min(1)`?". Este aqui pergunta uma coisa diferente: "**eu apliquei essa validação em todos os endpoints**, ou só nos que olhei primeiro?". Padrão de bug: o desenvolvedor corrige um endpoint que viu, esquece os outros da mesma classe — o código review unitário foca no diff e não pega; só auditoria dirigida por classe de bug pega.*
>
> *Origem (peer review Organiza 2026-05-22):* a auditoria corrigiu `mongoose.Types.ObjectId.isValid` em `routes/tasks.js` (PUT e DELETE) mas esqueceu o **mesmo padrão** em `routes/auth.js` no `reset-password/:id/:token`. ID malformado virava `CastError` → 500. Não era vulnerabilidade séria — mas era a mesma classe que foi declarada como "resolvida". A correção inconsistente passou batido na 1ª passada da auditoria porque o foco estava em "o achado foi corrigido?" em vez de "o achado foi corrigido **em todos os pontos onde se aplica**?".
>
> **Pergunta-teste:** "Eu corrigi a validação de ID em X. Conferi também Y, Z, W — todos os endpoints que recebem `req.params.id`, `req.body.*Id`, `req.query.id`, ou similar?"
>
> **Onde aplicar (NoSQL — MongoDB/Mongoose):**
> - `findById(id)` sem `mongoose.Types.ObjectId.isValid(id)` antes
> - `findOne({ _id: id })` idem
> - `findOneAndUpdate({ _id: id }, ...)`, `deleteOne({ _id: id })`, etc.
>
> **Onde aplicar (SQL):**
> - Path/body param tipado como `int`/`uuid` mas usado direto numa query sem coerção
> - Postgres `WHERE id = $1` com `id` vindo do path como string — bloqueia injeção, mas malformado vira erro do driver
>
> **Onde aplicar (geral):**
> - Toda função que recebe um identificador externo e passa direto para uma camada que faria cast
>
> **Opção 1 — Se há endpoints com `findById(id)` (ou equivalente) sem validação prévia em parte dos pontos:**
> - **Inconsistência de correção** — alguns endpoints validam, outros não. Risco: fingerprinting + UX ruim no não-validado + sinal de descuido para auditor sênior.
> - Listar **todos** os endpoints que recebem ID externo (`grep -rn 'req\.params\|req\.query\|req\.body\.\w*Id'` ou equivalente)
> - Para cada um, validar o formato antes da query (`isValid`, regex, `.uuid()`, etc.)
> - Considerar middleware reutilizável: `validateObjectId('id')` aplicado por rota — reduz a chance de esquecer no próximo endpoint novo
> - Adicionar **um teste adversarial por endpoint**: ID malformado → 400 (não 500). Sem teste, a próxima correção esquecida volta a passar batido.
> - 🔗 *Relacionado: item 34 (schema de ID estrito) é a regra geral; item 59 é a meta-pergunta de cobertura. Item 60 (teste adversarial por endpoint) é o irmão deste — também sobre disciplina de cobertura.*
>
> **Opção 2 — Se a validação de formato é universal nos endpoints que recebem ID externo:** ✅ Excelente

---

### 60. Cada endpoint **destrutivo ou sensível** tem ao menos um teste adversarial **dedicado àquele endpoint** — não só testes adversariais "em geral"?

> *Esse item é o "olhar por endpoint" do item 19. O 19 pergunta "há testes adversariais cobrindo caminhos críticos?". Este aqui pergunta: "para CADA endpoint que faz operação destrutiva, eu tenho ao menos um teste de ataque específico daquele endpoint, ou minha cobertura é desigual?".*
>
> *Padrão de bug clássico: 90% dos testes adversariais ficam em `/login` (cenários genéricos: token inválido, payload errado, brute force), 0% em `/reset-password/:id/:token`, `/change-email/:token`, `/delete-account`, `/admin/promote-user`. Os endpoints com **maior poder de causar dano** são os menos testados — exatamente o oposto do que deveria ser.*
>
> *Origem (peer review Organiza 2026-05-22):* depois de adicionar 24 testes adversariais (incluindo `alg:none` rejeitado, IDOR em PUT/DELETE de tasks, anti-enumeração em login e forgot-password), a peer review notou que **zero testes** cobriam o `reset-password/:id/:token` — um endpoint que **troca a senha do usuário** com base em ID + token vindos da URL. A 1ª passada deu por bem coberto porque o endpoint *teoricamente* compartilha mitigações com o resto do `/auth/*`. A 2ª passada (peer) percebeu que "compartilha mitigação" não é o mesmo que "tem teste que prova". A correção exigiu 9 testes novos: caminho feliz, ID malformado, ID inexistente, senha fraca, `alg:none`, token expirado, token assinado sem `+user.password`, cross-id, reuso após troca de senha. Cada um valida uma propriedade de segurança específica daquele endpoint.
>
> **Pergunta-teste — listar e contar:**
> *"Quais são os endpoints destrutivos/sensíveis do meu projeto? (delete, reset password, change email, change role, transfer, payout, refund, disable 2FA, delete account, admin actions). Para cada um, **quantos testes adversariais dedicados** existem? Algum tem zero?"*
>
> **O que conta como "dedicado àquele endpoint":**
> - Teste que monta o cenário de ataque **específico daquele endpoint** (não genérico)
> - Para `reset-password`: cross-id attack, token de outro usuário, `alg:none`, expirado, reuso depois da troca, senha fraca, ID malformado
> - Para `delete-account`: usuário tentando deletar conta de outro, sem reautenticação no momento crítico
> - Para `change-email`: token de mudança de email reutilizado depois de já trocado, redirect para email já cadastrado
> - Para `admin/promote-user`: não-admin tentando, admin tentando se auto-promover indevidamente, role inválida no body
>
> **O que NÃO conta:**
> - "Teste de login com token inválido" cobrindo só `/login` — não é dedicado ao endpoint sensível.
> - "Middleware auth bloqueia request sem token" — cobre o middleware, não cada endpoint.
> - Cobertura genérica via mesma classe de teste replicada — precisa ser **específico** do endpoint.
>
> **Opção 1 — Se há endpoints destrutivos/sensíveis sem teste adversarial dedicado:**
> - Listar os endpoints destrutivos/sensíveis do projeto (não há mais que ~10 numa app típica)
> - Para cada um, escrever pelo menos:
>   1. **1 teste de caminho feliz** (regression guard — prova que não quebrei depois)
>   2. **1 teste de input adversarial** (payload malformado / valor que tenta exceder permissão)
>   3. **1+ teste de ataque específico** (cross-user, cross-tenant, reuso de token, race condition, replay, etc.)
> - Para cada propriedade matemática/criptográfica do endpoint (ex: "secret derivado da senha invalida tokens antigos"), escrever um teste que prova essa propriedade — não confiar em "está claro pelo código".
> - 🔗 *Relacionado: item 19 (testes adversariais em geral) e item 20 (teste por ameaça do THREAT_MODEL). Item 60 é a versão "por endpoint" — corta a mesma cobertura por um eixo diferente para revelar lacunas.*
>
> **Opção 2 — Se cada endpoint sensível tem ao menos um teste adversarial específico dele:** ✅ Excelente

---

### 61. Existe **smoke test automatizado em CI** que valida que as defesas críticas estão ATIVAS na URL pública pós-deploy, não apenas no commit local?

> *Esse item nasce empiricamente da auditoria do Miniatura Forja AI: a v1 (paranoica) marcou 12 correções como ✅ no commit local; a v2 (comparativa) abriu a URL ao vivo com `curl` e descobriu que **zero** das correções estava em produção — o webhook GitHub→Vercel apontava para o repositório antigo. O código estava perfeito. A defesa não estava executando.*
>
> *Auditar o repositório ≠ auditar o produto. Para o atacante, só importa o código que serve a request real.*
>
> *Pergunta-teste: "Se eu mudar a configuração de deploy (webhook, branch, environment, projeto Vercel/Netlify/etc) sem alterar o código, alguma coisa **falha automaticamente em CI**, ou eu só descubro fazendo `curl` manual depois?"*
>
> *Esse item é a versão "post-deploy" do item 17. O 17 pergunta "código de prod = código auditado?". O 61 pergunta "**alguém ainda está verificando isso depois que eu mudar a config?**".*
>
> **O que conta como smoke test:**
> - Job de CI que roda **APÓS** o deploy (deployment hook, cron, ou `needs: deploy` no GitHub Actions)
> - Que faz `curl` contra a **URL pública** (não localhost, não preview)
> - Que verifica pelo menos:
>   - Headers de segurança críticos presentes (CSP, HSTS, X-Frame-Options) — confirma `vercel.json` / equivalente está sendo aplicado
>   - Endpoints sensíveis rejeitam input adversarial — `Origin: https://evil.com` → 403, body inválido → 400, sem auth em rota privada → 401
>   - Bundle público não contém padrões de segredo (`grep` por `sk_*`, `AKIA*`, etc — já é item 58)
>   - Versão do código está correta (algum hash do bundle, ou string canária que só existe na versão atual)
>
> **O que NÃO conta:**
> - Teste local em `dev` antes do push — esse é "teste pré-deploy", não fecha o vetor
> - Smoke test que roda só uma vez na hora do deploy mas não em PR/push subsequentes — cria janela cega
> - Smoke test que valida só o frontend mas não a API (ou vice-versa)
>
> **Opção 1 — Se "corrigido" significa apenas "merged" sem validação na URL pública:**
> - Adicionar job de CI **pós-deploy**. Exemplo para GitHub Actions:
>   ```yaml
>   smoke-test-prod:
>     needs: deploy   # ou: if: github.ref == 'refs/heads/main'
>     runs-on: ubuntu-latest
>     steps:
>       - name: Aguarda propagação
>         run: sleep 60
>       - name: Headers de segurança presentes
>         run: |
>           HEADERS=$(curl -sS -I https://meusite.com/)
>           echo "$HEADERS" | grep -qi 'content-security-policy' || exit 1
>           echo "$HEADERS" | grep -qi 'x-frame-options' || exit 1
>       - name: CORS bloqueante
>         run: |
>           STATUS=$(curl -sS -o /dev/null -w '%{http_code}' \
>             -X POST -H 'Origin: https://evil.com' \
>             https://meusite.com/api/sensitive)
>           [ "$STATUS" = "403" ] || exit 1
>   ```
> - Para hosting com deploy webhook (Vercel, Netlify): configurar webhook que dispara o smoke test em vez de depender de cron
> - Falha do smoke test → alerta (Slack/email) + idealmente rollback automatizado para a versão anterior
> - 🔗 *Relacionado: itens 17 (paridade dev/prod no código), 58 (segredo no bundle), 21 (CI básico de cada PR). Este item é o último guarda — verifica que tudo que veio antes está EXECUTANDO.*
>
> **Opção 2 — Se há smoke test automatizado validando defesas críticas na URL pública pós-deploy:** ✅ Excelente

---

## 📋 RESUMO DE EXECUÇÃO

Ao final da auditoria:

1. **Conte quantos itens caíram em "Opção 1"** (problema existe)
2. **Categorize por severidade** (bloqueador / essencial / qualidade / diferenciação)
3. **Estime tempo de correção** de cada item
5. **Priorize**: bloqueadores SEMPRE primeiro, mesmo que demorem
5. **Crie um issue/task** para cada item de "Opção 1"
6. **Mantenha esse checklist atualizado** — adicione perguntas novas a cada projeto que te ensinar algo

---

## 🎯 LEMBRE-SE

- **Checklist 100% verde não é meta** — meta é entender POR QUE cada item está verde
- **Honestidade > Performance** — README honesto sobre o que falta vale mais que README mentiroso
- **Trade-offs documentados são sinal de senioridade** — explicar por que você NÃO fez algo conta a seu favor, se a justificativa for sólida
- **Esse documento é vivo** — atualize conforme aprende