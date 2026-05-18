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

A prova de que os checklists funcionam está em aplicá-los. Quatro relatórios completos disponíveis:

### [`AUDIT_REPORT_BrieflyAI_2026-05-16.md`](AUDIT_REPORT_BrieflyAI_2026-05-16.md)

Auditoria de um app de IA/LLM com chat, transcrição e summarização. **Resultado:** 36 de 39 itens declarados como mitigados foram confirmados no código (92%). 3 discrepâncias entre intenção e implementação. 9 achados adicionais fora da lista — 3 deles viraram perguntas novas no checklist (itens 5B, 17B, E6).

### [`AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md`](AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md)

Auditoria comparativa v1 → v2 de um SaaS colaborativo com WebSocket. **Resultado:** 26 itens universais + 7 itens de WebSocket implementados a partir da revisão anterior. 1 IDOR sutil descoberto em handler de socket (identidade vindo do payload do cliente em vez do handshake) — esse achado virou os itens 3B e A2B nos checklists.

### [`AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md`](AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md)

Segunda passada sobre o Lumina, **validando se as 25 correções declaradas pela v1 estavam de fato no código**. **Resultado:** 23 de 25 confirmadas (92%) + 4 achados novos. Encontrou 1 vulnerabilidade crítica que tinha escapado (rate limit por IP que lia `X-Forwarded-For` cru — atacante drible com `curl -H 'X-Forwarded-For: 1.2.3.4'`) e dois casos onde a v1 violou itens do checklist que ela mesma criou (5C e 39B). Auditoria gerou 2 perguntas novas: item **23B** (IP confiável atrás de proxy?) e item **20B** (CSP sem `'unsafe-inline'` em `script-src`?). **Lição meta:** auditor não pode auditar a si mesmo sem perder calibração — segunda passada com olhar fresco é parte do método.

Esses relatórios mostram o método em ação: o que foi confirmado no código, o que divergiu da intenção declarada, o que escapou ao checklist, e como cada bug encontrado retroalimentou os documentos universais.

---

## 🎯 Como usar

1. **Para auditar um projeto seu:**
   - Comece pelo `AUDIT_CHECKLIST.md`, da 🔴 Bloqueadores para baixo.
   - Para cada pergunta, verifique no código (não na memória) — anote `arquivo.js:linha` em cada ✅.
   - Rode as seções aplicáveis do `CONTEXT_ADDONS.md` para o seu contexto.
   - Documente o resultado num relatório no mesmo formato dos exemplos.

2. **Para usar como referência em code review:**
   - Os itens funcionam como categorias de revisão. "Esse PR cobre o item 9D?" é mais útil que "esse PR é seguro?".

3. **Para evoluir os checklists:**
   - Toda vez que encontrar um bug que o checklist não pegou, adicione uma pergunta nova. É assim que essa lista chegou onde está.

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
