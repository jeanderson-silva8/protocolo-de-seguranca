# 📜 CHANGELOG — Protocolo de Segurança

> Histórico de evolução do framework. Cada versão registra **o que mudou e por quê**, mantendo rastro de cada item novo até a auditoria de projeto real que o originou.
>
> Convenção: [SemVer](https://semver.org/lang/pt-BR/) adaptado para documentos vivos —
> **MAJOR** = reorganização que muda a forma de usar (split de arquivos, renumeração).
> **MINOR** = item novo derivado de auditoria real.
> **PATCH** = correção, clarificação, polimento sem mudar conteúdo.

---

## [1.9.0] — 2026-05-22 (peer review do Organiza)

### Adicionado
- **Item 59 — "Toda função que recebe ID externo valida o formato ANTES do banco — em TODOS os pontos, não só em alguns"** na banda nova 🧭 META.
  - Origem: a auditoria do Organiza corrigiu `mongoose.Types.ObjectId.isValid` em `routes/tasks.js` (PUT e DELETE) mas esqueceu o **mesmo padrão** em `routes/auth.js` → `reset-password/:id/:token`. ID malformado virava `CastError` → 500. A peer review pegou; a 1ª passada interna não.
  - Categoria: **inconsistência de correção** — não é "achado novo", é "achado antigo aplicado em parte dos pontos". Padrão difícil de pegar em code review unitário porque o reviewer foca no diff; precisa de auditoria por classe de bug.
- **Item 60 — "Cada endpoint destrutivo/sensível tem ao menos um teste adversarial DEDICADO àquele endpoint"** na banda 🧭 META.
  - Origem: a 1ª passada da auditoria do Organiza adicionou 24 testes adversariais (incluindo `alg:none` rejeitado, IDOR, anti-enumeração), mas **zero** cobriam `reset-password/:id/:token` — endpoint que troca a senha do usuário com ID + token da URL. A peer review notou: "compartilhar mitigação com `/auth/*` ≠ ter teste que prova". Resultado: 9 testes novos só para `reset-password` (caminho feliz, ID malformado, ID inexistente, senha fraca, `alg:none`, expirado, token sem `+user.password`, cross-id, reuso após troca).
  - Categoria: **cobertura desigual** — 13 testes em login, 0 em reset-password. Endpoints com maior poder de causar dano costumam ser os menos testados.
- Banda nova 🧭 META no final do checklist, abrigando itens que verificam **disciplina de cobertura** das defesas listadas nas bandas anteriores. Sem shift — itens 59 e 60 entraram no final, numeração agora **sequencial 1-60**.

### Aplicado em projeto real (Organiza, mesma data)
- `backend/routes/auth.js` — `mongoose.Types.ObjectId.isValid` adicionado no `reset-password` (mesmo padrão do `tasks.js`, agora consistente).
- `backend/server.js` — `credentials: false` no CORS (auth via Bearer, sem cookie; `credentials: true` era permissão sem uso).
- `docs/adr/ADR-002` — nota explicando por que o `reset-password` compartilha o `authLimiter` em vez de ter limiter dedicado (256 bits de entropia do token + `authLimiter` apertado de 5/min cobrem o risco).
- `backend/tests/auth.test.js` — 9 testes adversariais novos para `reset-password` (suíte total: 24 → 33 testes, todos verdes).
- Estrutura dos testes refatorada (file-level hooks em vez de per-describe) para suportar múltiplos `describe` no mesmo arquivo sem reconexão Mongo.

### Lições meta documentadas
1. **"Resolvido em um lugar" não é "resolvido".** O item 59 nasce dessa lição: a correção do `ObjectId.isValid` em `tasks.js` foi aplicada com cuidado mas não replicada em `auth.js`. Auditoria por classe de bug ("onde mais se aplica isso?") pega; auditoria por arquivo não.
2. **Cobertura de testes precisa de eixo "por endpoint", não só "por categoria".** Suíte adversarial é assimétrica por natureza — gravita pros endpoints mais óbvios (login). Forçar contagem "testes por endpoint sensível" revela a lacuna.
3. **Peer review continua agregando valor mesmo no modo paranoico.** Terceira vez consecutiva (Lumina v2 → originou paranoico, TrendScope → originou item 17 + matriz obrigatória, Organiza → originou itens 59 e 60). A lição é estrutural: a 2ª passada do mesmo auditor pega ~95%; olho fresco humano pega o ~5% que vinha da fadiga ou foco do auditor.

---

## [1.8.0] — 2026-05-22

### Adicionado
- **Item 58 — "Nenhum segredo vaza para o bundle público do frontend"** na seção 🎨 FRONTEND do `AUDIT_CHECKLIST.md`. Numeração agora **sequencial 1-58**, sem shift (entrou como último item).
  - Cobre o vetor distinto do item 13: um segredo pode estar corretamente fora do git (`.env` no `.gitignore`) e **ainda assim** chegar ao navegador do usuário, embutido no bundle JavaScript.
  - Vetores cobertos: prefixos de exposição do bundler (`VITE_*`, `NEXT_PUBLIC_*`, `REACT_APP_*`, `PUBLIC_*`, `EXPO_PUBLIC_*`); segredo hardcoded em componente; import indevido de código de servidor no cliente; source maps de produção; chamada direta do browser a API de terceiro com chave secreta.
  - Pergunta-teste: `grep` de padrões de segredo no diretório de build (`dist/`, `.next/`, `build/`) + inspeção do JS no DevTools do site publicado.
  - Trio do ciclo de vida do segredo agora completo: **13** (não está no repositório) · **6** (existe quando o servidor sobe) · **58** (não escapa para o navegador).

### Origem
- Auditoria do **Organiza** (`organiza-dashboard-full`, MERN, modo paranoico, 2026-05-22). Durante a auditoria, a verificação "alguma `VITE_*` carrega valor sensível?" foi feita de forma **ad hoc** (no Organiza, `api.js` só usa `VITE_API_URL` — uma URL, sem segredo) — e o relatório do Nexus Portal já havia checado source maps também de forma pontual. O autor notou que nenhum item do checklist **obrigava** essa verificação: o item 13 olha o histórico do git, não o que chega ao browser. Lacuna real → item novo.

### Lições meta documentadas
1. **"Está no `.env`" não significa "está seguro".** O `.env` protege contra o repositório, não contra o build. Bundlers de frontend embutem por design qualquer variável com o prefixo de exposição — e esse JS é público.
2. **Verificação feita ad hoc em duas auditorias seguidas = sinal de item faltando.** Quando o auditor checa algo "por iniciativa" repetidamente, o checklist deveria estar obrigando — caso contrário a próxima auditoria pode esquecer.

---

## [1.7.0] — 2026-05-18

### Adicionado
- **Peer review externa do TrendScope** (após as 10 sessões de correção) identificou 1 crítico novo + 3 médios que escaparam às passadas internas (padrão + cobertura completa):
  - **C20** — SSRF em `fetchOgImage`: função fazia `fetch(targetUrl)` onde `targetUrl` vem de resultado da API Serper. Não é input direto do usuário, mas vem de fonte externa — atacante via supply chain (Serper comprometida ou SEO poisoning) pode forçar fetch para AWS metadata, loopback, RFC1918. Auditor interno minimizou ("já em C10 v1") em vez de classificar como crítico próprio.
  - Refinamento ao **ADR-003**: documentação de "cold start = burst window" em rate limit serverless (não só escala horizontal).
  - **ADR-004 novo**: formalização de `'unsafe-inline'` em `script-src` como dívida com gatilhos objetivos (mesmo padrão do Lumina ADR-003).
  - Fechamento do **item 39** (split `/api/health/live` + `/api/health/ready`).

### Mudado
- **Item 15 (SSRF) expandido** no `AUDIT_CHECKLIST.md`: cobertura explícita de "URL vem de API externa, não direto do usuário" — supply chain SSRF via search results, webhooks de parceiro, feeds RSS, OpenGraph preview. A palavra "userInput" do texto antigo sugeria restritivamente input direto; nova versão deixa explícito que API externa conta também.
- **Item 36 (rate limit por usuário) expandido**: nuance "cold start = burst window pós-cold-start" em serverless. Não é só problema de escala horizontal — é problema imediato single-instance também. Pergunta-teste serverless adicionada.
- **Convenção reforçada**: refinamento de classe existente vira **expansão do item**, NÃO item novo. Inflar a contagem só pra refinar gera ruído. Item 15 e 36 ganharam parágrafos novos; numeração 1-57 permanece.

### Origem
- Peer review externa (humano com olho fresco) do TrendScope após as 10 sessões de correção. **Segunda vez** em projetos auditados aqui que peer review pega crítico onde a auditoria interna minimizou (primeira foi Lumina v2 → originou o modo paranoico). Reforça empiricamente: peer review externa **não é "extra"**, é parte do método para projetos que importam.

### Lições meta documentadas
1. **"Aplica-se a:" do checklist precisa ser generoso, não restritivo.** Item 15 dizia "qualquer feature em que o backend faz `fetch(userInput)`" — palavra "userInput" sugeriu ao auditor que API externa não conta. Agora explícito.
2. **Mesmo paranoico precisa de olho fresco.** Auto-revisão pega ~95%; peer review pega o ~5% restante onde o auditor minimizou de boa fé. Em projetos críticos: paranoico + peer review humano = 99%.

---

## [1.6.1] — 2026-05-18

### Adicionado
- **Padrão de README pós-auditoria** documentado em `WORKFLOW.md` (seção nova entre v2 comparativa e Regras inegociáveis). Dois sub-padrões:
  - **Padrão A** (projetos sem auth complexa) — 1 seção `🔒 Segurança — camadas e status` com `<a id="seg-camadas">` (tabela + "O que NÃO está implementado"). Exemplos: TrendScope, Lumina.
  - **Padrão B** (projetos com auth complexa: login + refresh + sockets/multi-tenant/convites) — **2 seções** linkadas via anchors explícitos: `seg-camadas` (tabela) + `arq-auth` (6-8 etapas narrativas explicando o encadeamento contra XSS/CSRF/IDOR/spoofing/roubo de token). Exemplos: BrieflyAI, FlowSnyker.
- Regra de **anchors explícitos** (`<a id="seg-camadas">`, `<a id="arq-auth">`, `<a id="arq-auth-ws">`, etc.) em vez de slugs automáticos do GitHub. Emojis + em-dashes + acentos + parênteses no header quebram o slug automático em alguns parsers. Anchor explícito é HTML invisível e funciona em 100%.
- Tabela "quando aplicar qual padrão" mapeando stack → padrão A ou B.
- Referência cruzada no `PROMPT_AUDITORIA.md` e `PROMPT_AUDITORIA_PARANOICO.md` apontando para a seção do WORKFLOW.

### Mudado
- BrieflyAI e FlowSnyker READMEs migrados pro padrão novo: header da Segurança virou "Segurança — camadas e status" + anchor `<a id="seg-camadas">`; header da Auth virou "Arquitetura de Auth — como o fluxo resiste a [vetores] (o porquê e o encadeamento)" + anchor `<a id="arq-auth">` (BrieflyAI) ou `<a id="arq-auth-ws">` (FlowSnyker).
- FlowSnyker README expandido: seção "Arquitetura de Auth + WebSocket" agora tem 6 etapas explicando o encadeamento contra IDOR via socket (versão anterior tinha 3 etapas só sobre convites).

### Origem
- Pergunta direta do mantenedor: "é coerente deixar essas duas áreas de segurança no BrieflyAI?". Resposta: sim, mas é necessário diferenciar tabela (o que existe) de narrativa (por que assim), e padronizar pra outros projetos. Esse loop "pergunta → padronização" é o ciclo que vem evoluindo o framework desde a v1.0.

---

## [1.6.0] — 2026-05-18

### Adicionado
- `PROMPT_AUDITORIA_PARANOICO.md` — variante do PROMPT_AUDITORIA padrão que exige **passada dupla automática** no mesmo turno (1ª passada normal + auto-revisão adversarial obrigatória da matriz). Custo: ~2× tempo, ~30% mais tokens. Benefício: cobertura sobe de ~85% real (padrão) para ~95% real. Inclui:
  - Tabela "quando usar paranoico vs padrão" (portfólio → padrão; produção crítica/PII real/pós-incidente → paranoico)
  - 12 fases (vs 6 do padrão): adiciona auto-revisão da matriz (Fase 5), caça-aos-fantasmas com lentes específicas (Fase 6), verificação de amostragem de 3 ✅ aleatórios (Fase 7), aplicação cruzada de padrões (Fase 8), reflexão meta sobre as duas passadas (Fase 10)
  - Regra reforçada de ✅: âncora `arquivo:linha` **+ confirmação de execução em produção** (não dev/test/staging — lição direta do TrendScope onde `boot.ts` tinha defesas mas não era o entrypoint de prod)

### Origem
- 2ª passada de cobertura do TrendScope (entrada [1.5.0] acima) achou 6 críticos novos que escaparam à Passada 1 — incluindo C14 (`strict-ssl false` + mirror npm não-oficial). Isso mostrou que pra projetos críticos, a 2ª passada **não pode ser opcional** — vira parte do mesmo workflow. Modo paranoico automatiza o que fizemos manualmente.
- Decisão de manter os DOIS templates (padrão + paranoico) em vez de só o paranoico: nem todo projeto justifica 2× o custo. Portfólios e MVPs ficam bem com 85%.

---

## [1.5.0] — 2026-05-18

### Adicionado
- **Item 17 novo** no `AUDIT_CHECKLIST.md` (🔴 Bloqueadores): "Em projetos serverless / multi-entrypoint, o caminho de produção é o MESMO que o caminho auditado?" — derivado da auditoria do TrendScope, onde o projeto mantém `server/boot.ts` (com CORS+headers+bodyLimit) e `api/trpc/[...trpc].ts` (handler Vercel que duplica 262 linhas SEM nenhuma dessas defesas), e é o segundo que serve o tráfego de produção. Bug invisível para auditor que lê só o entrypoint de dev.
- `AUDIT_REPORT_TrendScope_2026-05-18.md` documenta a primeira auditoria do TrendScope. **Após 2 passadas**: 4 sólidos + 4 parciais + **11 críticos** (5 originais + 6 da passada de cobertura) + 8 achados fora do checklist + matriz completa de 57 itens + 10 adendos.
- **Regra nova no `PROMPT_AUDITORIA.md` (Fase 1, instrução 8 — OBRIGATÓRIO):** relatório deve conter **matriz de cobertura completa** com 1 linha por item (57 do checklist + adendos rodados/N/A). Sem isso, lacunas viram zona cinzenta.

### Mudado
- **Renumeração:** o agente adicionou inicialmente como "17B" (regredindo a v1.2.0). Imediatamente shift aplicado: 17B → 17, e todos os itens 17-56 → 18-57. Total agora **1-57 sequencial**. TOC, ranges das seções, cross-refs internas e nota de migração atualizados na mesma passada. Os relatórios anteriores (incluindo a versão do TrendScope publicada minutos antes desta renumeração) **não foram reescritos** — preservam o "17B" original, com tradução via nota de migração.

### Origem
- TrendScope 2026-05-18: README declara "Protocolo de Segurança Enterprise" detalhando CORS, headers, body limit, rate limit em `server/boot.ts`. Auditoria confirma que o `boot.ts` está correto — mas `vercel.json` é praticamente vazio (`{"framework": "vite"}`) e o tráfego real entra por `api/trpc/[...trpc].ts`, um handler Vercel Node.js que reimplementa toda a lógica sem chamar nenhum middleware. Lição: em apps serverless híbridos, "auditar `boot.ts`" não basta — é preciso seguir o entrypoint da plataforma.

### Lições meta documentadas
1. **Sufixo B/C reaparece por inércia** — o agente que executou a v1 via PROMPT_AUDITORIA adicionou "17B" por reflexo, regredindo v1.2.0. Reforça que renumeração imediata é parte do fluxo. PROMPT_AUDITORIA Fase 5 ganhou ⚠️ explícito contra sufixos.
2. **Cobertura implícita não é cobertura** — a v1 cobriu 38 de 57 itens explicitamente; os 19 restantes ficaram em zona ambígua (nem confirmados, nem N/A, nem violados — simplesmente não mencionados). 2ª passada de cobertura achou 6 críticos novos, incluindo **C14 — Dockerfile com `npm config set strict-ssl false` + registry mirror não-oficial** (supply chain attack vector latente). Forçou regra nova de matriz obrigatória.

---

## [1.4.0] — 2026-05-18

### Adicionado
- `PROMPT_AUDITORIA.md` — template pronto pra colar em LLM agente quando for auditar projeto novo. 16 instruções ordenadas + 5 regras inegociáveis + placeholders. Garante extração ~95% do método sem precisar lembrar de cada disciplina.
- `WORKFLOW.md` reescrito com **13 fases explícitas** derivadas das lições acumuladas (vs. versão anterior de 6 passos). Adiciona Fase 0 (preparação + perguntas ao autor), Fase 2 (classificação 3-categorias bug/ADR/N/A), Fase 5 (promoção de achados), seção dedicada à v2 comparativa, tabela "checklist sozinho vs método completo".

### Origem
- Pergunta direta: "se eu seguir o workflow original de 6 passos, extraio 100%?". Resposta honesta: ~70-80%. Faltavam as disciplinas que vieram das lições reais (Fase 0 do Lumina, Fase 2 do BrieflyAI #17, Fase v2 do Lumina v2).

---

## [1.3.0] — 2026-05-18

### Adicionado
- Seção `🔗 Adendos relacionados` no fim de **cada um dos 10 adendos**. Conecta horizontalmente itens que são manifestações do mesmo princípio em superfícies diferentes (ex: A3 ↔ B7 ↔ D3 ↔ E4 ↔ J5 — "autorização granular a recurso"; A5 ↔ E6 ↔ F1 — "rate limit distribuído"; C2 ↔ F3 — "HMAC nas duas pontas"). Auditor que cruza dois contextos não audita cada um isolado.
- Nota explícita no `00_INDICE.md` marcando **adendos C (Pagamento) e G (Mobile) como intencionalmente concisos** — não por descuido, por disciplina: nada entra sem rastro de auditoria real. Lista de tópicos planejados para quando o primeiro caso aparecer.

### Mudado
- Removido o número fixo "55 perguntas" do índice de adendos. Vira mentira na primeira atualização que esquece de propagar — agora só os adendos individuais sabem quantos têm.

### Origem
- Crítica externa de auditor de método (Anthropic Claude) ao estado pós-1.2.0: identificou fragmentação horizontal sem cross-references entre adendos que tratam da mesma classe de bug em superfícies diferentes.

---

## [1.2.0] — 2026-05-18

### Adicionado
- **Item 23B novo** no `AUDIT_CHECKLIST.md` (Qualidade): "IP confiável atrás de proxy/CDN" — derivado da auditoria v2 do Lumina (crítico #1: `_client_ip` lia `X-Forwarded-For` cru, atacante zerava rate limit com 1 linha de curl).
- **Item 20B novo** no `AUDIT_CHECKLIST.md` (Essenciais): "CSP estrita sem `'unsafe-inline'` em `script-src`" — derivado da auditoria v2 do Lumina (achado novo #A: CSP do vercel.json era teatro de segurança).
- `AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md` documenta a segunda passada que originou ambos.

### Mudado
- `AUDIT_CHECKLIST.md` **renumerado para sequencial 1-56** (eliminando sufixos `3B`, `5C`, `6B`, `6C`, `9B`, `9C`, `11B`, `13B`, `13C`, `17B`, `20B`, `23B`, `39B`). Adicionado **TOC navegável por seção** + **nota de migração** com tabela de equivalência abreviada.
- `CONTEXT_ADDONS.md` (arquivo monolítico de 750 linhas) **splittado em pasta `context-addons/`** com 10 arquivos modulares + `00_INDICE.md` (matriz "stack → adendo aplicável").
- README do repo atualizado pra apontar pra nova estrutura.

### Origem
- Lição meta: "auditor não pode auditar a si mesmo sem perder calibração". A v1 do Lumina declarou 92% de entrega e 4 das declarações tinham bugs — 1 deles vulnerabilidade crítica que escapou ao próprio auditor (eu) que tinha acabado de criar os itens 5C e 39B e os violou na implementação.

---

## [1.1.0] — 2026-05-17

### Adicionado
- **Item 5C novo** no `AUDIT_CHECKLIST.md`: "Fail-fast preservado em camadas de orquestração" — derivado da auditoria do Lumina v1 (`docker-compose.prod.yml` reintroduzia fallback inseguro `${VAR:-default}` que anulava o fail-fast do `settings.py`).
- **Item 39B novo** no `AUDIT_CHECKLIST.md` (Frontend): "Guard de rota valida token (decode + exp), não só `if (token)`" — derivado da auditoria do Lumina v1 (`ProtectedRoute` aceitava qualquer string truthy; bypass via `localStorage.setItem('lumina_token','x')` no console).
- **Item E6 novo** no Seção E (LLM): "Rate limit de LLM funciona em múltiplas instâncias (Redis/banco), não in-memory" — derivado do `chatRateLimiter` in-memory do BrieflyAI que quebraria silenciosamente em escala horizontal.
- `AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md` (primeira auditoria do Lumina, 1 sólido + 4 parciais + 13 críticos).
- `WORKFLOW.md` descrevendo o processo de manutenção do repo (criar auditoria nova, evoluir checklist, regras de coerência).

### Origem
- Primeira auditoria de demo de portfólio com descobertas de classes novas. A maior lição: a diferença entre "decisão consciente de escopo" (multi-tenant não implementado → ADR-001) e "bug real" (introspecção GraphQL ligada em prod) é o que separa marketing de honestidade.

---

## [1.0.1] — 2026-05-16

### Adicionado
- **Item 3B novo**: "Identidade do ator em handlers async vem da camada autenticada, não do payload" — derivado de IDOR sutil no `presenceEvents.ts` do FlowSnyker v2 (atacante autenticado emite `board:join` com `user._id` da vítima e socket passa a se identificar como vítima).
- **Item A2B novo** (Seção A — WebSocket): "`socket.userId` é guardado no handshake e nunca reescrito" — mesma origem do 3B, versão socket-específica.
- **Item A5 reescrito** (Seção A): rate limit por `socket.use()` oficial em vez de wrapping de `socket.on` (que não pega `socket.onAny` nem eventos com ACK).
- `AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md` — auditoria comparativa v1→v2 mostrando 26 itens universais + 7 itens de socket implementados a partir da revisão anterior.
- `AUDIT_REPORT_BrieflyAI_2026-05-16.md` — auditoria de app de IA/LLM (36/39 declarados confirmados; 3 discrepâncias; 9 achados novos).

---

## [1.0.0] — 2026-05-15

### Lançamento inicial
- `AUDIT_CHECKLIST.md` monolítico com 42 itens base (mais alguns sufixos `5B`, `9B`, `9C`, `11B`, `13B`, `13C`, `17B` adicionados nas primeiras revisões) organizado em 4 níveis de prioridade (🔴 Bloqueadores, 🟠 Essenciais, 🟡 Qualidade, 🟢 Diferenciação) + seção 🎨 Frontend.
- `CONTEXT_ADDONS.md` monolítico com 10 seções (A-J) para contextos específicos.
- `README.md` explicando o método ("checklist → audita projeto real → cada bug novo vira pergunta nova → próximo projeto pega").
- Licença MIT.
- Primeira publicação no GitHub.

### Filosofia fundadora
- **Checklist 100% verde não é meta** — meta é entender POR QUE cada item está verde.
- **Honestidade > Performance** — relatório honesto sobre o que falta vale mais que relatório mentiroso.
- **Trade-offs documentados são sinal de senioridade.**
- **Esse documento é vivo** — atualize conforme aprende.

---

## 🔄 Regras de evolução

Pra um item novo entrar (release MINOR):

1. **Origem rastreável obrigatória** — apontar pra auditoria de projeto real que justifica.
2. **Não duplicar item existente** — se outro adendo tem versão do mesmo princípio, adicionar como cross-reference em vez de novo item.
3. **Pergunta-teste verificável** — auditor deve conseguir responder com "sim" ou "não" olhando o código.
4. **Receita acionável** — não basta apontar problema; tem que dizer como corrigir.

Mudanças MAJOR (split, renumeração) exigem:

- Nota de migração com tabela de equivalência antigo→novo.
- Relatórios antigos **não são reescritos** — preservam o histórico real do método.
- Comunicação clara em commit message + entrada no changelog.
