# Auditoria de Segurança — Nexus Portal

- **Data:** 2026-05-21
- **Método:** AUDIT_CHECKLIST.md v1.7.0 (itens 1-57) — **MODO PADRÃO** (1 passada) + verificação dos 10 adendos de contexto
- **Escopo declarado pelo autor:** frontend estático puro de showcase de portfólio — experiência 3D interativa, **sem backend, sem autenticação, sem banco de dados, sem API própria, sem serverless functions**. Deploy estático na CDN da Vercel. A ausência de backend é **decisão consciente de escopo**, não bug.
- **URL ao vivo:** https://nexus-portal-one.vercel.app/
- **Sem auditoria anterior.**
- **Resultado em uma frase:** projeto pequeno, limpo e honesto — o código-fonte React/Three.js não contém nenhuma vulnerabilidade explorável, as dependências de produção estão sem CVEs conhecidos, e nenhum segredo foi versionado; os três achados são todos de baixa severidade (ausência de headers de segurança / CSP no deploy estático, ausência de Subresource Integrity nas fontes do Google Fonts, e ruído de organização no repositório).

---

## Como ler este relatório

Cada item do checklist universal foi verificado contra o código real do repositório (`nexus.md/`), não contra o README. Para cada ✅, há `arquivo:linha` que prova a mitigação. Como o projeto é um **frontend estático sem backend**, a maioria dos 57 itens é **🚫 N/A por escopo** — e isso é um resultado legítimo, não uma lacuna. Um site estático bem-feito tem, por construção, uma superfície de ataque mínima.

Os achados estão agrupados em quatro blocos:

1. **Confirmado** — mitigação existe e está provada no código, ou a propriedade de segurança é satisfeita por construção.
2. **Parcial** — algo existe mas não está completo.
3. **Críticos / violações** — bug real ou risco em produção. *(Nenhum nesta auditoria.)*
4. **Polimentos** — baixa criticidade, ruído ou inconsistência cosmética.

---

## Nota sobre escopo

O Nexus Portal é uma **experiência 3D de showcase** — um artefato interativo (o "Anel Nexus") renderizado em tempo real com Three.js + React Three Fiber, com customização visual ao vivo (cor, brilho, partículas), scroll cinematográfico (Lenis), animações (GSAP) e efeitos de áudio sintetizados via Web Audio API.

**Não há backend, autenticação, banco de dados, API própria nem serverless functions.** Toda a aplicação é HTML + CSS + JavaScript estático, servida pela CDN da Vercel. Esta é uma decisão de arquitetura consciente e adequada ao propósito: um portfólio que demonstra domínio de shaders GLSL, renderização 3D e animação de alto desempenho não precisa de dados de usuário nem de servidor.

Em consequência, **toda a seção 🔴 Bloqueadores que trata de auth/authz/banco/secrets de runtime, toda a seção 🟢 Diferenciação que trata de observabilidade/backup/audit log, e quase toda a seção 🟠 Essenciais que pressupõe um servidor de API são 🚫 N/A por escopo.** Isso não é deficiência — é a forma correta de classificar itens que descrevem superfícies que o projeto não possui. Os itens que **de fato importam** num site estático são poucos e estão tratados nos Blocos 1, 2 e 4.

Não há `THREAT_MODEL.md`, `SECURITY.md` ou `docs/adr/` no projeto, e isso também é razoável para um showcase visual sem dados — ver classificação na matriz.

---

## Bloco 1 — Confirmado

| # | Item do checklist | Evidência |
|---|---|---|
| 13 | Segredos fora do código versionado | `git log --all --full-history -- .env*` retorna vazio — nenhum `.env` jamais existiu ou foi commitado. `git ls-files` lista 35 arquivos, nenhum com credencial. Não há nenhuma chave de API, token ou senha no código — a aplicação não consome serviço externo algum. |
| 14 | Sem injeção (queries / shell / path) | Não há banco de dados, query, comando shell ou manipulação de path com input do usuário. Toda a "entrada" do usuário é interação com o canvas 3D (drag, scroll, sliders) que altera estado numérico no Zustand store (`src/store/useStore.ts:38-58`). |
| 16 | Sem desserialização insegura nem SSTI | Busca por `eval`, `new Function`, `document.write` em todo `src/` — zero ocorrências. Sem render dinâmico de template com input do usuário. |
| 32 | HTTPS forçado | A Vercel termina TLS e redireciona HTTP→HTTPS automaticamente para todo deploy. URL ao vivo serve sob `https://`. |
| 43 | Dependências auditadas | `npm audit --omit=dev` → **0 vulnerabilidades** nas 9 dependências de produção (React 19.2, Three 0.184, @react-three/fiber 9.6, @react-three/drei 10.7, gsap 3.15, lenis 1.3, zustand 5.0). Three.js, GSAP e o ecossistema R3F estão em versões recentes sem CVE conhecido. (Ver Bloco 4 para a única pendência — dev-only.) |
| 55 | Sem `dangerouslySetInnerHTML` / sanitização de HTML | Busca por `dangerouslySetInnerHTML` e `innerHTML` em todo `src/` — zero ocorrências. Todo conteúdo textual é literal hardcoded no JSX (React escapa por padrão). Não há conteúdo fornecido por usuário renderizado como HTML. |
| 57 | Erros do frontend tratados sem expor detalhes | A aplicação não faz nenhuma chamada de rede em runtime — não há resposta de backend a tratar. O carregamento da cena 3D usa `<Suspense fallback={null}>` (`src/App.tsx:34`) e há um `LoadingScreen` controlado. Os blocos `try/catch` em `src/hooks/useAudio.ts:38,63` engolem falhas de áudio silenciosamente (degradação graciosa, correto). |

> **Observação sobre o item 16/55 (frontend XSS):** a aplicação é imune a XSS por construção — não há nenhuma fonte de dado externa ou input textual de usuário que chegue ao DOM. Os únicos inputs (sliders de glow/partículas, swatches de cor, drag de órbita) produzem valores numéricos/enum consumidos como uniforms de shader e estado de câmera.

---

## Bloco 2 — Parcial

### Item 31 / 56 — Headers de segurança e CSP no servidor estático

**Existe:** o deploy na Vercel já fornece, por padrão, terminação TLS e HSTS no domínio `*.vercel.app`.

**Falta:** não há arquivo `vercel.json` no projeto, portanto **nenhum header de segurança explícito é configurado**. O HTML estático servido em `/` não emite `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options` / `frame-ancestors`, `Referrer-Policy` nem `Permissions-Policy`.

Para um site estático sem auth, sem cookies e sem dados de usuário, o impacto real é **baixo**: não há sessão a roubar via XSS, não há clickjacking de ação sensível a explorar. Ainda assim, é a melhoria de maior valor disponível — `X-Content-Type-Options: nosniff` e `frame-ancestors 'none'` são defesa em profundidade barata, e uma CSP fecha a porta caso uma dependência futura introduza um sink.

**Correção sugerida:** criar `nexus.md/vercel.json`:
```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" },
        {
          "key": "Content-Security-Policy",
          "value": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; img-src 'self' data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; object-src 'none'"
        }
      ]
    }
  ]
}
```

> **Nota sobre o item 33 (CSP estrita sem `'unsafe-inline'` em `script-src`):** o Vite com `base: './'` gera os scripts da aplicação como `<script type="module" src="...">` externos (não inline) — então `script-src 'self'` é viável **sem** `'unsafe-inline'`. O `'unsafe-inline'` acima aparece apenas em `style-src`, exigido pelo CSS-in-JS / estilos inline do React + Tailwind; esse trade-off é aceitável (XSS via estilo permite defacement, não execução de código). Se for desejável fechar também `style-src`, recomenda-se validar com `vite-plugin-csp-guard`. Recomenda-se testar a nota em https://securityheaders.com após aplicar.

### Item 30 — README honesto

**Existe:** o `README.md` descreve corretamente a stack, as funcionalidades e a estrutura de pastas — é factual e não promete segurança que não existe (não há bloco de "segurança enterprise" inflado, ao contrário do TrendScope).

**Pequena imprecisão:** o `tech-spec.md` (na pasta-pai) lista React `^18.3`, Vite `^6.0` e inclui `@react-three/post-processing` — mas o `package.json` real usa **React 19.2, Vite 7.2 e não inclui post-processing**. O documento de spec ficou desatualizado em relação à implementação. Não é um problema de segurança, mas é uma divergência declarado-vs-entregue que vale corrigir para coerência. O `README.md` em si está correto (cita React 19 + Vite 7).

---

## Bloco 3 — Críticos / violações

**Nenhum.**

Não foi encontrada nenhuma violação crítica nem de alta severidade. O código-fonte React/TypeScript está limpo: sem `eval`, sem `dangerouslySetInnerHTML`, sem `localStorage`/`sessionStorage`, sem `fetch` ou chamada de rede em runtime, sem `console.log` de debug, sem segredos, sem TODO/FIXME revelador. As dependências de produção não têm CVE. Este é o resultado esperado e legítimo para um site estático bem construído.

---

## Bloco 4 — Polimentos

- **`info.md` versionado é ruído de organização** (`nexus.md/info.md`). É a saída de scaffold de uma ferramenta de UI (shadcn) — lista 40+ componentes shadcn, menciona `src/components/ui/`, `src/types/`, `src/App.css` que **não existem no projeto** (não há pasta `ui/`, só `ui-custom/`). É um arquivo de processo que escapou ao housekeeping; sugere-se removê-lo do repositório ou adicioná-lo ao `.gitignore`. Não vaza nada sensível (só o caminho de build `/mnt/agents/output/app`), mas polui a impressão para quem audita o repo.

- **`tech-spec.md` desatualizado** (na pasta-pai `NEXUS PORTAL/`). Lista versões antigas (React 18, Vite 6) e uma dependência (`@react-three/post-processing`) que não está no `package.json`. Como está fora da pasta `nexus.md/`, não é versionado pelo repo — mas convém atualizá-lo ou marcá-lo como spec histórica de MVP. *(O bloom mencionado no tech-spec parece ter sido implementado direto no shader do `NexusRing`, sem a lib de post-processing — coerente com o `package.json`.)*

- **A pasta do repositório se chama `nexus.md`** — nome incomum para a raiz de um repositório de código (sugere arquivo Markdown). É puramente cosmético e não afeta o deploy, mas renomear para `nexus-portal/` ou similar evita confusão.

- **Sem source maps explicitamente desligados** — `vite.config.ts` não define `build.sourcemap`, então o default do Vite (`false` em produção) prevalece. Não há vazamento de código-fonte via sourcemap em produção. Item já correto; registrado apenas para constar que foi verificado.

- **`brace-expansion` 5.0.2 — vulnerabilidade moderada, escopo dev-only.** `npm audit` completo aponta 1 vulnerabilidade moderada (GHSA-jxxr-4gwj-5jf2, DoS por range numérico) em `brace-expansion`, dependência transitiva de `@typescript-eslint` (ferramenta de lint). **Não entra no bundle de produção** — `npm audit --omit=dev` retorna 0. Corrigível com `npm audit fix` sem risco. Severidade efetiva para o site publicado: nenhuma.

- **Sem CI/CD** (`.github/workflows/` inexistente). Para um portfólio estático, é aceitável; um workflow mínimo (`npm run lint && npm run build`) seria um diferencial barato, mas não é uma falha de segurança.

---

## Achados fora do checklist (raciocínio adversarial para site estático)

| Vetor investigado | Resultado |
|---|---|
| **Subresource Integrity (SRI)** nas fontes do Google Fonts | 🟡 **Ausente.** O `index.html:14` carrega `<link href="https://fonts.googleapis.com/css2?...">` sem atributo `integrity`. Fontes do Google Fonts (CSS dinâmico por user-agent) tecnicamente **não suportam SRI** de forma estável — o CSS retornado varia. O risco residual (CDN do Google comprometido injetando CSS malicioso) é baixo e fora do controle do projeto; a mitigação prática é a CSP do Bloco 2 restringindo `style-src`/`font-src` aos domínios do Google. Não há nenhum `<script src>` de CDN externo no HTML — então não há superfície de SRI para script (a parte realmente perigosa). Registrado como observação, não como bug. |
| **`dangerouslySetInnerHTML` / `innerHTML`** | ✅ Zero ocorrências em `src/`. |
| **`eval` / `new Function` / `document.write`** | ✅ Zero ocorrências. |
| **`localStorage` / `sessionStorage` / cookies** | ✅ Não utilizados — não há estado persistido no browser, nada a roubar via XSS. |
| **`fetch` / `XMLHttpRequest` em runtime** | ✅ Zero — a aplicação não faz nenhuma requisição de rede após o carregamento estático. Sem superfície de SSRF (não há servidor) nem de injeção de URL. |
| **`console.log` com dado sensível / comentários reveladores / TODO** | ✅ Zero `console.*` no código de aplicação. Comentários são todos descritivos (em PT-BR, explicando shaders e animação). |
| **Source maps em produção** | ✅ Default do Vite (`sourcemap: false`) — código-fonte não exposto. |
| **Assets em `public/`** | ✅ Apenas `public/favicon.svg`. Nada indevido. |
| **Web Audio API** (`src/hooks/useAudio.ts`) | ✅ Sintetiza tons via osciladores — não carrega nem reproduz arquivo de áudio externo. Sem superfície. |

---

## Itens N/A com justificativa por escopo

Site estático sem backend/auth/banco — a maioria dos 57 itens descreve superfícies inexistentes.

| # | Item | Justificativa de N/A |
|---|---|---|
| 1, 2, 3, 4 | Auth, autorização, IDs da sessão, identidade em handlers async | Não há autenticação nem backend. Nenhuma rota, nenhum recurso "pertencente a alguém", nenhum handler assíncrono de servidor. |
| 5 | Validação de input (Zod/Joi/Pydantic) | Não há input de cliente chegando a um controller — não há controller. Os "inputs" são interações de UI que produzem estado numérico/enum local, tipado em TypeScript. |
| 6, 7, 8 | Fail-fast de envs / dependências runtime / orquestração | Não há variáveis de ambiente, dependências de runtime nem camada de orquestração. Build é estático. |
| 9, 10, 11, 12 | Hash de senha, JWT, comparação tempo-constante, brute force | Sem autenticação, sem senha, sem token, sem comparação de segredo. |
| 15 | SSRF | O servidor (CDN estática) não faz requisição alguma a partir de input. A aplicação cliente também não faz `fetch`. |
| 17 | Serverless / multi-entrypoint | Não há serverless function nem segundo handler. Deploy é 100% estático (CDN). Único "entrypoint" é o `index.html`. |
| 18 | Cookies httpOnly/secure/sameSite | Não há cookies. |
| 19, 20 | Testes de caminhos críticos / adversariais | Não há caminho crítico de segurança a testar (sem auth/authz/validação de servidor). Testes de unidade da lógica 3D seriam qualidade, não segurança. |
| 21, 22, 23 | CI/CD, error handler centralizado, controllers usando `throw` | Sem servidor / controllers. CI seria diferencial (ver Bloco 4). |
| 24, 25, 26 | Race conditions, logging estruturado, logs limpos de PII | Sem servidor, sem operações concorrentes, sem logs de servidor, sem PII. |
| 27, 28, 29 | Documentação de API, THREAT_MODEL.md, cota atômica | Não há API a documentar nem cota/limite. THREAT_MODEL é dispensável para showcase visual sem dados (registrado na Nota de escopo). |
| 33 | IDs com formato estrito | Não há ID de recurso. |
| 34 | Paginação | Não há listagem de dados. |
| 35, 36, 37 | Rate limit por usuário / IP confiável atrás de proxy / política de senha | Sem servidor para limitar, sem proxy de API, sem senha. |
| 38 | Health live/ready | Não há processo de servidor com health check. |
| 39 | Queries N+1 | Não há banco de dados. |
| 40 | Erros sem vazar stack em prod | Não há backend gerando stack trace. (Frontend coberto no item 57 — Bloco 1.) |
| 41 | CORS com allowlist | Não há API com CORS. |
| 42 | CSRF | Não há cookie de sessão nem endpoint que muta estado de servidor. |
| 44 | Correlation ID | Sem requisições de servidor a correlacionar. |
| 45 | Métricas / telemetria | Sem backend; observabilidade de servidor não se aplica. |
| 46 | Soft delete | Não há operação destrutiva sobre dados. |
| 47 | Audit log | Não há ação sensível de usuário a auditar. |
| 48 | Backup / DR | Não há dado persistido. O código-fonte tem backup via Git. |
| 49 | Política de retenção (LGPD/GDPR) | Não há dado pessoal coletado nem persistido. |
| 50 | Dockerfile multi-stage | Deploy é estático na Vercel — não usa container. Dockerfile seria desnecessário. |
| 51 | ADRs | Razoável para um showcase pequeno não ter `docs/adr/`. O `tech-spec.md` cumpre parcialmente o papel de registrar decisões de arquitetura (canvas fixo, isolamento de contexto R3F, shader memoizado). |
| 52 | OpenAPI / contrato de API | Não há API. |
| 53 | Onde o access token é armazenado | Sem token. |
| 54 | Guard de rota valida token | Aplicação é single-page sem rotas protegidas — uma única página pública. |

### Adendos de contexto (00_INDICE.md) — todos N/A

| Adendo | Status | Motivo |
|---|:---:|---|
| A — WebSocket / Real-time | 🚫 N/A | Sem Socket.io/ws/SSE. Toda interatividade é local no browser. |
| B — Upload de arquivos | 🚫 N/A | Não há upload — nenhum `<input type="file">` nem endpoint. |
| C — Pagamento | 🚫 N/A | Sem Stripe/MercadoPago nem fluxo de cobrança. |
| D — Multi-tenant | 🚫 N/A | Sem conceito de tenant/organização. |
| E — LLM / IA | 🚫 N/A | Sem OpenAI/Anthropic/RAG. |
| F — APIs públicas / Webhooks | 🚫 N/A | Não expõe API nem recebe webhook. |
| G — Mobile | 🚫 N/A | Web SPA pura — sem React Native/Flutter/app nativo, sem WebView, sem deep link. |
| H — Dados sensíveis | 🚫 N/A | Não coleta, processa nem persiste PII, dado de saúde, financeiro ou jurídico. |
| I — Filas / Workers | 🚫 N/A | Sem BullMQ/Celery/Kafka/SQS. Toda execução é no event loop do browser. |
| J — GraphQL | 🚫 N/A | Sem Apollo/Yoga/schema GraphQL. |

---

## Resumo executivo

| Status | Quantidade |
|---|---:|
| ✅ Confirmado | 7 (itens 13, 14, 16, 32, 43, 55, 57) |
| 🟠 Parcial | 3 (itens 30, 31, 56) |
| 🔴 Crítico / violação | 0 |
| 🚫 N/A por escopo | 47 |
| Achados fora do checklist | 1 🟡 (SRI ausente — risco residual baixo) + 5 polimentos |
| Adendos rodados | 0 de 10 (todos N/A — declarados explicitamente) |

**Régua aplicada:** frontend estático puro de showcase. Com essa régua, o projeto está **sólido**. A distância entre o que o README declara e o que o código entrega é mínima (única imprecisão é o `tech-spec.md` desatualizado, fora do repo). Não há nenhum bug de segurança explorável. As três pendências classificadas como 🟠 Parcial são de baixa severidade e fáceis de fechar — a de maior valor é adicionar `vercel.json` com headers de segurança.

---

## Plano de ação ordenado por severidade

### 🟡 Recomendado (baixa severidade, alto custo-benefício)
1. **Criar `nexus.md/vercel.json` com headers de segurança + CSP** (itens 31/56) — bloco JSON pronto na seção "Bloco 2". Fecha clickjacking, MIME sniffing e blinda contra futura introdução de sink XSS. ~15 min. Validar em securityheaders.com.

### 🟢 Polimento (sem impacto de segurança, melhora a impressão de auditoria)
2. **Remover `info.md` do repositório** (ou adicioná-lo ao `.gitignore`) — é ruído de scaffold que descreve uma estrutura inexistente.
3. **Atualizar `tech-spec.md`** para refletir React 19 / Vite 7 e a ausência de `@react-three/post-processing`, ou marcá-lo como spec histórica de MVP.
4. **`npm audit fix`** — resolve a vulnerabilidade moderada dev-only de `brace-expansion`.
5. *(Opcional)* Renomear a pasta-raiz `nexus.md` para `nexus-portal`.
6. *(Opcional)* Adicionar workflow de CI mínimo (`npm run lint && npm run build`).

### 🚫 Conscientemente fora de escopo (não fazer — registrado para clareza)
- Autenticação, backend, banco de dados, THREAT_MODEL.md, SECURITY.md, Dockerfile, testes de auth/authz — **não se aplicam** a um showcase 3D estático. Implementá-los seria over-engineering, não melhoria de segurança.

---

## Evolução dos checklists

Esta auditoria **não gera nenhuma pergunta nova** para o `AUDIT_CHECKLIST.md`.

Todos os achados (headers/CSP em deploy estático, SRI em fontes de CDN, ruído de repositório) já são cobertos por itens existentes (31, 33, 56, 30) ou pelo raciocínio adversarial padrão de Fase 4. O caso "frontend estático sem backend" é exatamente o cenário que o checklist já trata bem via classificação massiva de N/A — não há lacuna a preencher.

Registra-se uma observação meta, sem ação: este é o **primeiro projeto auditado no histórico do framework que é frontend estático puro** (BrieflyAI, FlowSnyker, Lumina e TrendScope todos têm backend). Confirmou que o checklist universal, projetado para backend/fullstack, **se aplica sem fricção a um estático** desde que o auditor classifique N/A com disciplina e justificativa — o que valida a robustez do método. Não justifica um adendo novo: "site estático" não é um *contexto* com perguntas próprias, é a *ausência* de quase todos os contextos.

---

## Reflexão final

O Nexus Portal é o oposto do TrendScope. Onde o TrendScope tinha um README anunciando "Protocolo de Segurança Enterprise" sobre um backend que vazava credenciais e duplicava handlers, o Nexus Portal tem um README modesto que descreve exatamente o que o código é: uma experiência 3D bem-feita, sem pretensão de ser uma aplicação com dados. Auditá-lo foi rápido porque não há o que esconder — e essa é a lição.

A tentação, ao auditar um projeto pequeno e limpo, é "encontrar algo" para justificar o esforço. Inflar a ausência de `vercel.json` para crítico, ou tratar a ausência de backend como falha, seria desonesto. A regra inegociável do método — "se o projeto é seguro, diga que é" — existe precisamente para este caso. Um site estático bem construído tem, por construção, uma superfície de ataque que cabe em uma página: sem servidor não há SSRF, sem auth não há IDOR, sem banco não há injeção, sem cookies não há CSRF, sem input de usuário renderizado não há XSS. O trabalho do auditor aqui não foi caçar bugs — foi **confirmar com evidência que cada uma dessas portas está fechada porque nunca foi construída**, e classificar os 47 N/A com justificativa em vez de deixá-los em zona cinzenta.

A meta-lição: a disciplina de evidência (`arquivo:linha`, `git log` vazio, `npm audit` com saída concreta) vale tanto para provar que algo está seguro quanto para provar que algo está quebrado. Um relatório de "0 críticos, 3 polimentos" só tem valor se cada ✅ e cada N/A puder ser verificado por outra pessoa. Foi o que se buscou aqui.

---

## Matriz de cobertura completa — 57 itens + 10 adendos

Legenda: ✅ Confirmado | 🟠 Parcial | 🔴 Crítico/violado | 🚫 N/A por escopo

| # | Item | Status | Referência / Justificativa |
|---|------|:---:|---|
| 1 | Auth em rotas privadas | 🚫 | Sem backend/auth |
| 2 | Autorização em recursos | 🚫 | Sem backend/recursos |
| 3 | IDs vindos da sessão | 🚫 | Sem sessão |
| 4 | Identidade em handlers async | 🚫 | Sem handlers de servidor |
| 5 | Validação Zod/Joi/Pydantic no input | 🚫 | Sem controller; inputs são estado local tipado em TS |
| 6 | Envs validadas no boot | 🚫 | Sem variáveis de ambiente |
| 7 | Fail-fast para dependências runtime | 🚫 | Sem dependências de runtime |
| 8 | Fail-fast preservado em orquestração | 🚫 | Sem camada de orquestração |
| 9 | Hash de senha adequado | 🚫 | Sem senha |
| 10 | JWT seguro | 🚫 | Sem JWT |
| 11 | Comparação tempo-constante | 🚫 | Sem comparação de segredo |
| 12 | Brute force protection | 🚫 | Sem auth |
| 13 | Segredos fora do código | ✅ | `git log --all -- .env*` vazio; sem credencial em `git ls-files` |
| 14 | Queries parametrizadas | 🚫→✅ | Sem banco/query/shell — propriedade satisfeita por ausência de superfície |
| 15 | SSRF | 🚫 | Sem servidor que faça fetch; cliente não faz fetch |
| 16 | Sem desserialização/SSTI | ✅ | Zero `eval`/`new Function`/`document.write` em `src/` |
| 17 | Serverless / multi-entrypoint | 🚫 | Deploy 100% estático; único entrypoint é `index.html` |
| 18 | Cookies httpOnly/secure | 🚫 | Sem cookies |
| 19 | Testes para caminhos críticos | 🚫 | Sem caminho crítico de segurança |
| 20 | Testes adversariais p/ THREAT_MODEL | 🚫 | Sem THREAT_MODEL (N/A por escopo) |
| 21 | CI/CD com checks por PR | 🚫 | Sem servidor; CI seria diferencial (Bloco 4) |
| 22 | Error handler centralizado | 🚫 | Sem servidor/controllers |
| 23 | Race conditions / TOCTOU | 🚫 | Sem operação concorrente de servidor |
| 24 | Logging estruturado | 🚫 | Sem logs de servidor; zero `console.*` no app |
| 25 | Logs limpos de PII | 🚫 | Sem logs e sem PII |
| 26 | THREAT_MODEL.md | 🚫 | Showcase visual sem dados — dispensável (Nota de escopo) |
| 27 | SECURITY.md | 🚫 | Sem superfície de vulnerabilidade reportável — dispensável |
| 28 | Cota atômica | 🚫 | Sem cota/limite |
| 29 | Documentação de API (OpenAPI/API.md) | 🚫 | Sem API |
| 30 | README honesto | 🟠 | README correto; `tech-spec.md` desatualizado (versões/lib) — Bloco 2/4 |
| 31 | Headers de segurança | 🟠 | Sem `vercel.json`; só defaults da Vercel — Bloco 2 |
| 32 | HTTPS forçado | ✅ | Vercel termina TLS e redireciona HTTP→HTTPS |
| 33 | CSP estrita sem `unsafe-inline` em script-src | 🟠 | CSP ausente; quando criada, `script-src 'self'` é viável (scripts externos do Vite) — Bloco 2 |
| 34 | Paginação | 🚫 | Sem listagem de dados |
| 35 | Rate limit por usuário | 🚫 | Sem servidor |
| 36 | IP confiável atrás de proxy | 🚫 | Sem API/proxy |
| 37 | Política de senha forte | 🚫 | Sem auth |
| 38 | Health live + ready | 🚫 | Sem processo de servidor |
| 39 | Sem queries N+1 | 🚫 | Sem banco |
| 40 | Erros sem vazar stack em prod | 🚫 | Sem backend (frontend coberto no 57) |
| 41 | CORS com allowlist | 🚫 | Sem API |
| 42 | CSRF protection | 🚫 | Sem cookie/endpoint mutável |
| 43 | Dependências atualizadas + audit | ✅ | `npm audit --omit=dev` → 0 vulnerabilidades (9 deps prod) |
| 44 | Correlation ID | 🚫 | Sem requisições de servidor |
| 45 | Métricas / telemetria | 🚫 | Sem backend |
| 46 | Soft delete | 🚫 | Sem dados/operação destrutiva |
| 47 | Audit log | 🚫 | Sem ação sensível |
| 48 | Backup + DR | 🚫 | Sem dado persistido (código tem Git) |
| 49 | Política de retenção (LGPD/GDPR) | 🚫 | Sem dado pessoal |
| 50 | Dockerfile multi-stage | 🚫 | Deploy estático na Vercel — sem container |
| 51 | ADRs documentando decisões | 🚫 | `tech-spec.md` cobre parcialmente; dispensável p/ showcase |
| 52 | OpenAPI / contrato versionado | 🚫 | Sem API |
| 53 | Onde o access token é armazenado | 🚫 | Sem token; sem uso de localStorage/sessionStorage |
| 54 | Guard de rota valida token | 🚫 | SPA de página única pública, sem rota protegida |
| 55 | Sanitização de HTML do usuário | ✅ | Zero `dangerouslySetInnerHTML`/`innerHTML`; sem conteúdo de usuário |
| 56 | CSP no servidor do frontend (Vercel) | 🟠 | Sem `vercel.json` — Bloco 2 |
| 57 | Erros do backend tratados na UI | ✅ | Sem chamada de rede em runtime; `Suspense` + `try/catch` de áudio degradam graciosamente |
| **Adendo A** | WebSocket / Real-time | 🚫 | Sem WebSocket/SSE |
| **Adendo B** | Upload de arquivos | 🚫 | Sem upload |
| **Adendo C** | Pagamento | 🚫 | Sem cobrança |
| **Adendo D** | Multi-tenant | 🚫 | Sem tenant |
| **Adendo E** | LLM / IA | 🚫 | Sem LLM |
| **Adendo F** | APIs públicas / Webhooks | 🚫 | Sem API/webhook |
| **Adendo G** | Mobile | 🚫 | Web SPA pura |
| **Adendo H** | Dados sensíveis | 🚫 | Sem PII |
| **Adendo I** | Filas / Workers | 🚫 | Sem fila/worker |
| **Adendo J** | GraphQL | 🚫 | Sem GraphQL |

**Contagem final:** 57 itens — 7 ✅ (contando item 14 como propriedade satisfeita por ausência de superfície) · 3 🟠 · 0 🔴 · 47 🚫. 10 adendos — 0 aplicáveis, 10 🚫 N/A declarados.

---

*Relatório gerado em 2026-05-21 — MODO PADRÃO, 1 passada. Auditoria conduzida com o framework Protocolo de Segurança v1.7.0.*
