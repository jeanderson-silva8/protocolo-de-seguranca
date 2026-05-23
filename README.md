# 🛡️ Protocolo de Segurança

> **Checklists de auditoria de segurança para projetos backend/fullstack profissionais — em português, vivos, e validados em projetos reais.**

Esse repositório reúne dois checklists universais de auditoria e os relatórios reais produzidos ao aplicá-los em projetos próprios. Não é teoria copiada do OWASP — é um método de revisão que evolui a cada bug encontrado.

---

## 🧠 O método

```
Checklist  →  Audita projeto real  →  Cada bug novo vira pergunta nova  →  Próximo projeto pega automaticamente
```

A premissa é simples: checklist genérico estagna; checklist usado de verdade evolui. Cada item desses documentos ou já existia em referências como OWASP, ou nasceu de um achado concreto numa auditoria feita aqui. Nenhuma pergunta foi adicionada por "achismo" — todas têm justificativa rastreável.

**Por que isso importa:** os checklists não pegam tudo. O raciocínio adversarial ainda é do auditor. Mas quando um bug sutil é encontrado, ele vira pergunta — e a partir daí o próximo projeto não comete o mesmo erro. É como um teste de regressão para conhecimento de segurança.

---

## 📋 Os documentos

### [`AUDIT_CHECKLIST.md`](AUDIT_CHECKLIST.md)

Checklist universal aplicável a qualquer projeto backend/fullstack. Organizado em quatro níveis de prioridade:

- 🔴 **Bloqueadores** — sem isso, é vulnerável
- 🟠 **Essenciais** — sem isso, não é portfólio sênior
- 🟡 **Qualidade** — visível para quem audita
- 🟢 **Diferenciação** — separa sênior bom de sênior excelente

Cobre autenticação, autorização, validação, secrets, criptografia, injeção, SSRF, desserialização insegura, race conditions, timing attacks, JWT, CSRF, headers de segurança, testes adversariais, observabilidade, backups, threat modeling, ADRs, e mais.

Formato em árvore de decisão (`Opção 1 — se sim, faça X` / `Opção 2 — se não, ✅`) torna cada item acionável, não só uma boa intenção.

### [`context-addons/`](context-addons/)

Adendos por contexto, **um arquivo por seção** — abra só os que se aplicam ao seu projeto. Veja o [`00_INDICE.md`](context-addons/00_INDICE.md) para a matriz "stack → adendo aplicável".

- 🔌 [A — WebSocket / Real-time](context-addons/A_WEBSOCKET.md)
- 📁 [B — Upload de arquivos](context-addons/B_UPLOAD.md)
- 💳 [C — Pagamento](context-addons/C_PAGAMENTO.md)
- 🏢 [D — Multi-tenant / SaaS B2B](context-addons/D_MULTI_TENANT.md)
- 🤖 [E — Aplicações com IA / LLM](context-addons/E_LLM.md)
- 📡 [F — APIs públicas / Webhooks](context-addons/F_APIS_PUBLICAS.md)
- 📱 [G — Mobile](context-addons/G_MOBILE.md)
- 🧪 [H — Dados sensíveis (saúde, financeiro, jurídico)](context-addons/H_DADOS_SENSIVEIS.md)
- 📨 [I — Filas / Workers / Mensageria](context-addons/I_FILAS.md)
- 🔍 [J — GraphQL](context-addons/J_GRAPHQL.md)

---

## 🔍 Auditorias reais

A prova de que os checklists funcionam está em aplicá-los. Sete relatórios completos disponíveis:

### [`AUDIT_REPORT_BrieflyAI_2026-05-16.md`](AUDIT_REPORT_BrieflyAI_2026-05-16.md)

Auditoria de um app de IA/LLM com chat, transcrição e summarização. **Resultado:** 36 de 39 itens declarados como mitigados foram confirmados no código (92%). 3 discrepâncias entre intenção e implementação. 9 achados adicionais fora da lista — 3 deles viraram perguntas novas no checklist (itens 5B, 17B, E6).

### [`AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md`](AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md)

Auditoria comparativa v1 → v2 de um SaaS colaborativo com WebSocket. **Resultado:** 26 itens universais + 7 itens de WebSocket implementados a partir da revisão anterior. 1 IDOR sutil descoberto em handler de socket (identidade vindo do payload do cliente em vez do handshake) — esse achado virou os itens 3B e A2B nos checklists.

### [`AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md`](AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md)

Auditoria de uma demo de portfólio (Django + GraphQL + React) que se posicionava como SaaS B2B. **Resultado:** 1 item sólido, 4 parciais, 13 críticos — a maior parte dos críticos é **independente do escopo de portfólio** (introspecção GraphQL em prod, `ProtectedRoute` aceitando qualquer string como autenticado, fallback inseguro no `docker-compose`, lib JWT abandonada). Auditoria gerou 2 perguntas novas: item **5C** (orquestração preserva o fail-fast do app?) e item **39B** (guard de rota valida token de verdade ou só `if (token)`?). Exemplo de como o método trata diferença entre "decisão consciente de escopo" e "bug real".

### [`AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md`](AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md)

Segunda passada sobre o Lumina, **validando se as 25 correções declaradas pela v1 estavam de fato no código**. **Resultado:** 23 de 25 confirmadas (92%) + 4 achados novos. Encontrou 1 vulnerabilidade crítica que tinha escapado (rate limit por IP que lia `X-Forwarded-For` cru — atacante drible com `curl -H 'X-Forwarded-For: 1.2.3.4'`) e dois casos onde a v1 violou itens do checklist que ela mesma criou (5C e 39B). Auditoria gerou 2 perguntas novas: item **23B** (IP confiável atrás de proxy?) e item **20B** (CSP sem `'unsafe-inline'` em `script-src`?). **Lição meta:** auditor não pode auditar a si mesmo sem perder calibração — segunda passada com olhar fresco é parte do método.

### [`AUDIT_REPORT_TrendScope_2026-05-18.md`](AUDIT_REPORT_TrendScope_2026-05-18.md)

Auditoria de um motor de curadoria de tendências (React + Hono + tRPC + Drizzle + TiDB Serverless, deploy Vercel) que se descreve como "Protocolo de Segurança Enterprise". **Resultado (após 2 passadas):** 4 sólidos, 4 parciais, **11 críticos** + 8 achados fora do checklist + matriz completa dos 57 itens. O bug central é arquitetural: o projeto mantém **duas implementações divergentes do backend** (`server/boot.ts` com CORS+headers+bodyLimit, e `api/trpc/[...trpc].ts` que duplica 262 linhas SEM nenhuma dessas defesas) — e é o segundo que efetivamente serve o tráfego em produção. Combinado com `.env` real copiado para dentro da imagem Docker, rate limit in-memory + IP forjável, e — descoberto na 2ª passada de cobertura — **Dockerfile com `npm config set strict-ssl false` + registry mirror não-oficial** (supply chain attack vector latente) + ErrorBoundary vazando `error.message` cru ao usuário em prod. Auditoria gerou **1 pergunta nova:** item **17** — "Em projetos serverless / multi-entrypoint, o caminho de produção é o MESMO que o caminho auditado?" (bug clássico em apps serverless onde middleware vive no entrypoint dev mas não no handler Vercel/Lambda). A inclusão deste item disparou shift dos demais (17→18, ..., 56→57), totalizando agora 1-57 sequencial. **Lição meta:** a 1ª passada cobriu 38 de 57 itens explicitamente; os 19 restantes ficaram ambíguos. A 2ª passada (com matriz exaustiva de cobertura) achou C14 (supply chain), C15 (Docker root), C16 (sem Dependabot), C17 (sem ADRs), C18 (ErrorBoundary leak), C19 (`dangerouslySetInnerHTML` morto). Forçou regra nova no `PROMPT_AUDITORIA.md`: relatório DEVE conter matriz de 1 linha por item — sem isso, lacunas viram zona cinzenta. **Lição meta:** em projetos serverless, auditar `boot.ts` não basta — é preciso seguir o `vercel.json` até o handler que de fato processa o request.

### [`AUDIT_REPORT_Nexus-Portal_2026-05-21.md`](AUDIT_REPORT_Nexus-Portal_2026-05-21.md)

Auditoria de um **frontend estático puro** — o Nexus Portal, uma experiência 3D de showcase de portfólio (React 19 + Vite 7 + Three.js + React Three Fiber + GSAP + Lenis + Zustand, deploy estático na CDN da Vercel). Sem backend, sem autenticação, sem banco, sem API — ausência **consciente e adequada ao propósito**. **Resultado (MODO PADRÃO, 1 passada):** 7 itens confirmados, 3 parciais, **0 críticos**, 47 N/A por escopo. O código-fonte React/Three.js não contém nenhuma vulnerabilidade explorável (zero `eval`/`dangerouslySetInnerHTML`/`fetch`/`localStorage`, zero `console.log`), as 9 dependências de produção têm 0 CVEs (`npm audit --omit=dev`), e nenhum `.env` jamais foi versionado (`git log --all` vazio). Os três achados são todos de baixa severidade: ausência de `vercel.json` com headers de segurança/CSP no deploy estático, ausência de SRI nas fontes do Google Fonts (risco residual baixo, fora do controle do projeto), e ruído de organização no repositório (`info.md`/`tech-spec.md` desatualizados). **Nenhuma pergunta nova** gerada — o caso "estático sem backend" já é coberto pela classificação massiva de N/A. **Lição meta:** primeiro projeto estático puro do histórico — confirmou que o checklist universal (pensado para backend/fullstack) se aplica sem fricção a um estático quando o auditor classifica N/A com disciplina e evidência. E reforçou a regra inegociável: projeto seguro merece um relatório que **diz que é seguro** — inflar severidade para "parecer que trabalhou" seria desonesto.

### [`AUDIT_REPORT_Organiza_2026-05-22.md`](AUDIT_REPORT_Organiza_2026-05-22.md)

Auditoria de um **task manager MERN** — o Organiza (MongoDB Atlas + Mongoose + Express 5 + React 19, deploy Vercel Serverless) com autenticação JWT, bcrypt e reset de senha por e-mail. **Resultado (MODO PARANOICO, 2 passadas):** 19 itens confirmados sólidos, 17 de qualidade, **0 vulnerabilidades críticas exploráveis** — base de auth acima da média para portfólio (anti-IDOR real nas tarefas, bcrypt cost 12, Helmet, CORS allowlist, segredos fora do versionado). O achado central **não é de segurança clássica e sim funcional**: o enum do model `Task` (`['Pendente','Em Progresso','Concluída']`) não bate com os valores que a rota grava (`['todo','in-progress','done']`) — `POST /api/tasks` retorna HTTP 500 em produção e `PUT` grava valores inválidos silenciosamente (`findByIdAndUpdate` não roda validadores). A 1ª passada tratou isso como "inconsistência de nomenclatura"; a **2ª passada adversarial**, seguindo o caminho real de execução até `Mongoose.save()`, reclassificou de 🟡 para 🔴 — a feature central do produto está quebrada. Achados 🟠: rate limit in-memory em serverless (janela de burst pós-cold-start), JWT de 15min sem refresh token nem revogação, `jwt.verify` sem `algorithms` fixado. **Nenhuma pergunta nova** — todos os achados já são cobertos pela grade dos 57 itens. **Lição meta:** o bug 🔴 parecia cosmético e era funcional — exatamente o tipo de achado que justifica a 2ª passada paranoica; a calibração "decisão consciente vs. bug real" depende de seguir o código até o ponto de execução, não de ler a intenção declarada.

Esses relatórios mostram o método em ação: o que foi confirmado no código, o que divergiu da intenção declarada, o que escapou ao checklist, e como cada bug encontrado retroalimentou os documentos universais.

---

## 🎯 Como usar

### 🤖 Caminho mais rápido — usando LLM agente (Claude, etc.)

Dois templates disponíveis, dependendo da criticidade do projeto:

- **[`PROMPT_AUDITORIA.md`](PROMPT_AUDITORIA.md)** — padrão. 16 instruções, 5 regras inegociáveis. Cobertura ~85% real. 1× tempo. Use pra portfólio, MVP, demos.
- **[`PROMPT_AUDITORIA_PARANOICO.md`](PROMPT_AUDITORIA_PARANOICO.md)** — modo paranoico. Passada dupla automática (1ª passada + auto-revisão adversarial da matriz). Cobertura ~95% real. ~2× tempo. Use pra produção crítica, sistemas financeiros, apps com PII real, pós-incidente, pré-due-diligence.

Em ambos: copia, troca os `<<placeholders>>` (nome do projeto, pasta, stack, etc.), cola no chat. O agente segue as fases sozinho — leitura dos arquivos do framework, auditoria, classificação dos achados em bug/ADR/N/A, escrita do relatório no formato dos exemplos, promoção de achados ao checklist se for classe nova.

### 👤 Auditando manualmente (sem LLM)

Leia o [`WORKFLOW.md`](WORKFLOW.md) — descreve as **13 fases do método** com regras de coerência. Resumo:

1. **Para auditar um projeto seu:**
   - Comece pelo [`AUDIT_CHECKLIST.md`](AUDIT_CHECKLIST.md), da 🔴 Bloqueadores para baixo.
   - Para cada pergunta, verifique no código (não na memória) — anote `arquivo.js:linha` em cada ✅. **Disciplina-chave.**
   - Identifique contextos em [`context-addons/00_INDICE.md`](context-addons/00_INDICE.md) (matriz "stack → adendo aplicável") e rode só os aplicáveis.
   - Classifique cada problema em: 🔴 bug real / 📝 decisão consciente (vira ADR) / 🚫 N/A por escopo.
   - Documente o resultado num relatório no mesmo formato dos exemplos.
   - Quando o autor corrigir os bugs, faça **auditoria v2 comparativa** (padrão FlowSnyker v2 / Lumina v2).

2. **Para usar como referência em code review:**
   - Os itens funcionam como categorias de revisão. "Esse PR cobre o item 35?" é mais útil que "esse PR é seguro?".

3. **Para evoluir os checklists:**
   - Toda vez que encontrar um bug que o checklist não pegou, adicione uma pergunta nova (com origem rastreável). É assim que essa lista chegou onde está — ver [`CHANGELOG.md`](CHANGELOG.md).

---

## ⚠️ Escopo e limitações (honestidade primeiro)

**Esses documentos NÃO substituem:**

- Pentest profissional com tooling adversarial real.
- Análise de código com ferramentas estáticas (Semgrep, CodeQL).
- Revisão de infraestrutura (cloud, Kubernetes, IAM, rede).
- Compliance específico (PCI-DSS, HIPAA, SOC 2, ISO 27001).
- Modelagem de ameaças formal (STRIDE detalhado, attack trees).

**O que esses documentos SÃO:**

- Uma camada de revisão estruturada que pega 80% dos bugs comuns antes de chegarem em prod.
- Um ponto de partida honesto para discussão de segurança em times pequenos e médios.
- Uma ferramenta de autoavaliação para quem está construindo portfólio ou se preparando para entrevista técnica sênior.
- Um artefato vivo, em português, mantido com disciplina.

---

## 🧭 Filosofia

> **Checklist 100% verde não é meta** — meta é entender POR QUE cada item está verde.
>
> **Honestidade > Performance** — relatório honesto sobre o que falta vale mais que relatório mentiroso.
>
> **Trade-offs documentados são sinal de senioridade** — explicar por que você NÃO fez algo conta a seu favor, se a justificativa for sólida.
>
> **Esse documento é vivo** — atualize conforme aprende.

---

## 📜 Licença

MIT. Use, modifique, distribua. Atribuição é apreciada mas não obrigatória.

Se aplicar em projeto próprio e descobrir bug que o checklist não pegou, **abra uma issue ou PR** — é exatamente assim que esses documentos crescem.
