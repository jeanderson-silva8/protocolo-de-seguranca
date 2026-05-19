# 📜 CHANGELOG — Protocolo de Segurança

> Histórico de evolução do framework. Cada versão registra **o que mudou e por quê**, mantendo rastro de cada item novo até a auditoria de projeto real que o originou.
>
> Convenção: [SemVer](https://semver.org/lang/pt-BR/) adaptado para documentos vivos —
> **MAJOR** = reorganização que muda a forma de usar (split de arquivos, renumeração).
> **MINOR** = item novo derivado de auditoria real.
> **PATCH** = correção, clarificação, polimento sem mudar conteúdo.

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
