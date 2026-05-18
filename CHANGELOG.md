# 📜 CHANGELOG — Protocolo de Segurança

> Histórico de evolução do framework. Cada versão registra **o que mudou e por quê**, mantendo rastro de cada item novo até a auditoria de projeto real que o originou.
>
> Convenção: [SemVer](https://semver.org/lang/pt-BR/) adaptado para documentos vivos —
> **MAJOR** = reorganização que muda a forma de usar (split de arquivos, renumeração).
> **MINOR** = item novo derivado de auditoria real.
> **PATCH** = correção, clarificação, polimento sem mudar conteúdo.

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
