# 🔄 WORKFLOW — Como aplicar o método

> Guia operacional para auditar projeto novo e manter o framework vivo.
> Atualizado em 2026-05-18 (v1.4.0) com fluxo expandido de 13 fases, derivado das lições acumuladas em 4 auditorias reais (BrieflyAI, FlowSnyker v1/v2, Lumina v1/v2).
>
> Se você é visitante e quer apenas usar os checklists, leia o [`README.md`](README.md) — este documento é meta.

---

## 🧭 Princípio geral

```
Auditoria de projeto real  →  cada bug não-pego pelo checklist vira pergunta nova  →  próxima auditoria pega automaticamente
```

Cada commit aqui deve refletir esse ciclo. Adicionar item ao checklist "porque parece boa ideia" enfraquece o método — só adicione o que tem rastro de origem em uma auditoria real.

---

## 🚀 Atalho: prompt pronto para LLM

Se você usa Claude (ou outro LLM agente) para executar a auditoria, **use o [`PROMPT_AUDITORIA.md`](PROMPT_AUDITORIA.md)** — template já com as 16 instruções ordenadas, regras inegociáveis e placeholders. Cola e pronto.

O documento abaixo descreve o método humano (independente de LLM) — útil para entender as 13 fases por trás do prompt e para auditar com discernimento próprio.

---

## 📋 As 13 fases (humano ou LLM)

### Fase 0 — Preparação (antes de abrir o código)

**1. Releitura do framework.** Se faz tempo da última auditoria, releia `AUDIT_CHECKLIST.md`, `CHANGELOG.md` e este `WORKFLOW.md`. Itens novos da última revisão (ex: 23B sobre IP confiável atrás de proxy) só pegam bug se você lembrar deles.

**2. Calibração pelo CHANGELOG.** Itens da v1.0 são battle-tested em 3+ projetos. Itens da v1.3 saíram de 1 só. Bom auditor sabe quais classes de bug são certas e quais são hipóteses ainda em validação.

**3. Pergunte ao autor 3 coisas ANTES de começar** (lição do Lumina):
- **Em que estágio está o projeto?** (portfólio / MVP / produção) — afeta calibração de severidade
- **Há decisões de escopo conscientes que parecem "marketing" mas são intencionais?** (ex: "não tem multi-tenant porque...")
- **Há documentação do que NÃO foi implementado?** (READMEs honestos evitam que você reporte feature ausente como bug crítico)

> **Por que essa fase importa:** sem ela, você reporta como "fraude" o que é decisão consciente do autor. Custa pouco perguntar; custa muito reescrever relatório quando descobre depois.

### Fase 1 — Aplicação do checklist universal

**4. Vasculhe o projeto.** Mapa de pastas, package.json/requirements.txt, configs principais (`*.config.*`, `settings.*`, `docker-compose*.yml`, `.env.example`), CI/CD (`.github/`, `Dockerfile`), README, docs/.

**5. Aplique o `AUDIT_CHECKLIST.md` em ordem.** 🔴 Bloqueadores (1-17) → 🟠 Essenciais (18-32) → 🟡 Qualidade (33-43) → 🟢 Diferenciação (44-51) → 🎨 Frontend (52-56). Não pule — itens depois dependem de decisões dos itens antes.

**6. Identifique contextos no [`context-addons/00_INDICE.md`](context-addons/00_INDICE.md).** Matriz "stack → adendo aplicável" diz quais adendos rodar.

**7. Rode os adendos aplicáveis E leia `🔗 Adendos relacionados` no fim de cada um.** O cross-reference horizontal te lembra que A3 (room control), B7 (download authz), D3 (no cross-tenant) e E4 (RAG tenant-aware) são manifestações do MESMO princípio em superfícies diferentes — auditor que cruza contextos não audita isolado.

### Fase 2 — Disciplina de evidência (a regra mais importante)

**8. Para CADA item ✅, anote `arquivo.ext:linha` que prova a mitigação.**

> Esta é a regra que evita o erro #17 do BrieflyAI — o autor marcou um item como mitigado de boa fé, mas o código não entregava. A âncora no código separa "lembro que implementei" de "está implementado e provo onde".

Padrão recomendado na tabela do relatório:

```markdown
| 5 | Fail-fast da SECRET_KEY no boot | `backend/core/settings.py:19-21` |
```

**9. Classifique cada problema encontrado em uma de 3 categorias:**

| Classe | Quando | Vira... |
|--------|--------|---------|
| 🔴 **Bug real** | Mitigação não existe e devia | Item no plano de correção |
| 📝 **Decisão consciente do autor** | Autor confirmou que NÃO implementou de propósito | ADR sugerido (`docs/adr/ADR-XXX-tema.md`) |
| 🚫 **N/A por escopo declarado** | Funcionalidade não existe no projeto (ex: WebSocket num app REST puro) | Linha na seção "Itens N/A" do relatório, com justificativa |

> Confundir 🔴 com 📝 é o erro mais comum em auditoria de portfólio. Lição Lumina: "multi-tenant não implementado" não é bug — é ADR-001.

### Fase 3 — Achados fora do checklist

**10. Liste bugs/riscos que escaparam ao checklist**, categorizados por severidade (🔴 Crítico / 🟠 Alto / 🟡 Médio / 🟢 Baixo).

Esses são os mais valiosos — significam que o checklist tem um buraco que vale ser preenchido (ver Fase 5).

### Fase 4 — Escrita do relatório

**11. Use o template dos 4 relatórios de referência.** Estrutura obrigatória:

1. **Cabeçalho** — data, método, escopo, resultado em uma frase.
2. **"Como ler este relatório"** — orienta o visitante.
3. **"Nota sobre escopo"** (se houver decisões conscientes — referencia ADRs).
4. **Bloco 1 — Confirmado** (tabela `# | item | arquivo:linha`).
5. **Bloco 2 — Parcial** (iniciados mas incompletos, com correção sugerida).
6. **Bloco 3 — Críticos** (com PoC textual quando possível).
7. **Bloco 4 — Polimentos** (baixa criticidade).
8. **Itens N/A** com justificativa por escopo.
9. **Resumo executivo** (tabela de contagem).
10. **Plano de ação ordenado por severidade**.
11. **Evolução dos checklists** (achados que merecem virar pergunta nova).
12. **Reflexão final** (1-2 parágrafos com aprendizado meta — o que esse projeto te ensinou que você não sabia antes).

> **Reflexão final é obrigatória.** Sem ela, o relatório é só checklist preenchido.

Qual relatório usar como inspiração:

| Situação | Use como template |
|----------|-------------------|
| Projeto se considera "pronto" e quero confrontar declarado vs entregue | [BrieflyAI](AUDIT_REPORT_BrieflyAI_2026-05-16.md) |
| Já houve auditoria anterior, agora é comparativa v1→v2 | [FlowSnyker v2](AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md) |
| Projeto se descreve com "marketing" e preciso separar escopo de bug | [Lumina v1](AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md) |
| Segunda passada no mesmo projeto pós-correções | [Lumina v2](AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md) |

### Fase 5 — Promoção de achados a itens novos do checklist

**12. Se algum achado é classe NOVA**, proponha pergunta nova. Critérios:

| Pergunta | Se a resposta for SIM... |
|----------|--------------------------|
| Essa classe de bug é genérica (vai aparecer em outros projetos)? | Vira item |
| Outro auditor pensaria nisso sem ser provocado? | Se NÃO, vira item |
| A mitigação tem receita acionável? | Se SIM, vira item |
| Algum item existente já cobre? | Se SIM, NÃO vira item — vira cross-reference |

Quando virar item, atualize:
- `AUDIT_CHECKLIST.md` (item novo no nível de prioridade certo) OU adendo correspondente em `context-addons/`
- `CHANGELOG.md` com entrada nova (MINOR bump) citando a auditoria que originou
- README do repo se o item afeta um adendo principal

### Fase 6 — Entrega e persistência

**13. Salve a auditoria em DOIS lugares:**

```
seu-projeto/docs/AUDIT_REPORT_YYYY-MM-DD.md         ← filename SEM nome do projeto
                                                       (já está dentro do repo do projeto)

REPOSITORIO DE SEGURANÇA-GITHUB/
  AUDIT_REPORT_NomeDoProjeto_YYYY-MM-DD.md          ← filename COM nome do projeto
                                                       (vai conviver com várias auditorias aqui)
```

Atualize o **README** do `REPOSITORIO DE SEGURANÇA-GITHUB` com uma entrada nova na seção "🔍 Auditorias reais" — parágrafo com números concretos (não promessas).

---

## 🔁 A segunda passada (v2 comparativa)

**Lição Lumina v2:** auditor não pode auditor a si mesmo sem perder calibração. A v1 do Lumina declarou 92% de entrega e 4 declarações tinham bugs — incluindo 1 crítico explorável (rate limit via `X-Forwarded-For` forjado).

Regra: depois que o autor corrigir os bugs da v1, **faça auditoria v2 comparativa**. Padrão:

- Use [Lumina v2](AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md) ou [FlowSnyker v2](AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md) como template
- Foco: validar que as correções declaradas existem no código (anchor)
- Foco: buscar regressões e achados novos com olhos frescos
- Tabela v1 → v2 destacada (métrica de evolução em números concretos)

Se você é o autor das correções, peça pra outra pessoa (humano ou outra instância de LLM). Se for você mesmo, espere alguns dias — olhos cansados não pegam o que olhos frescos pegam.

---

## 📏 Regras inegociáveis (não negocie nem com você mesmo)

1. **Honestidade > marketing.** Relatório honesto sobre o que falta vale mais que relatório elogioso. Recrutador valoriza.
2. **Nenhum ✅ sem `arquivo:linha`.** Memória não conta.
3. **Decisões de escopo conscientes ≠ bugs.** Pergunte sempre.
4. **Não invente mitigações.** Se o código não tem, o relatório não diz que tem.
5. **Tom sereno.** Sem jargão privado ("Protocolo V.L.A.E.G", "Midnight Luxe"), sem hype ("Enterprise-grade", "blindado"). Tom dos relatórios de referência.
6. **Nunca edite auditoria publicada.** Adendo no fim com data, ou nova auditoria. Histórico é evidência do método.
7. **Publique auditorias modestas também.** Consistência de publicação vale mais que só os relatórios espetaculares.
8. **Nunca adicione item ao checklist sem caso real.** Origem rastreável obrigatória.
9. **Reflexão final obrigatória.** Sem ela, o relatório é só formulário preenchido.
10. **Português puro.** Termos técnicos em inglês ficam em código/inline; prosa é PT-BR.

---

## 🛠️ Comandos úteis

**Configurar identidade git (uma vez por máquina):**
```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
```

**Subir mudanças do repo do framework:**
```bash
git status
git add .
git commit -m "tipo: descrição curta"
git push
```

**Convenção de tipos de commit:**
- `feat:` adicionar item novo ao checklist ou adendo
- `docs:` adicionar auditoria, atualizar README, melhorar prosa
- `fix:` corrigir erro ou inconsistência em item existente
- `refactor:` renumerar, reorganizar sem mudar conteúdo (ex: split, renumber)
- `chore:` ajuste menor (CHANGELOG, gitignore)

---

## 📊 Tabela: o que cada fase entrega

| Fase | Sem ela, você... | Com ela, você ganha... |
|------|------------------|------------------------|
| 0 — Preparação | Reporta decisões de escopo como bugs | Calibração + contexto |
| 1 — Checklist | Tem cobertura mínima | Cobertura sistemática |
| 2 — Evidência | Repete erro #17 do BrieflyAI | Relatório auditável |
| 3 — Achados fora | Cobre só o conhecido | Pega o sutil, alimenta evolução |
| 4 — Relatório | Escreve em formato inconsistente | Histórico comparável |
| 5 — Promoção | Bug do projeto X repete no projeto Y | Framework evolui |
| 6 — Entrega | Auditoria fica perdida | Histórico público acumula |
| Bonus — v2 | Vive em 100% verde ilusório | Calibração real (lição Lumina) |

---

## 🧭 O que diferencia "usar checklist" de "aplicar o método"

| Quem só usa o checklist | Quem aplica o método completo |
|-------------------------|-------------------------------|
| Marca tudo verde de boa fé | Anota `arquivo:linha` em cada ✅ |
| Vê tudo como bug | Distingue bug / ADR / N/A |
| Faz auditoria uma vez | Faz v2 depois das correções |
| Encontra bug e corrige | Encontra bug, corrige, **promove a pergunta no checklist** |
| Cobre só o que o checklist tem | Lista achados fora do checklist |
| Audita cada contexto isolado | Lê cross-references entre adendos |
| Esquece o que descobriu | Reflexão final destila aprendizado |

A diferença não está nas perguntas — está nas **disciplinas em volta delas**.
