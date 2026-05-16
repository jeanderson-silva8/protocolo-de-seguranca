# 🔄 WORKFLOW — Como manter este repositório

> Guia operacional para o autor. Descreve como adicionar uma auditoria nova, como evoluir os checklists a partir de achados reais, e as regras que mantêm o repo coerente ao longo do tempo.
>
> Se você é visitante e quer apenas usar os checklists, leia o [README.md](README.md) — este documento é meta.

---

## 🧭 Princípio geral

```
Auditoria de projeto real  →  cada bug não-pego pelo checklist vira pergunta nova  →  próxima auditoria pega automaticamente
```

Cada commit aqui deve refletir esse ciclo. Adicionar item ao checklist "porque parece boa ideia" enfraquece o método — só adicione o que tem rastro de origem em uma auditoria real.

---

## 📋 Fluxo para cada nova auditoria

### Passo 1 — Auditar o projeto

1. Aplique o `AUDIT_CHECKLIST.md` no projeto, da seção 🔴 Bloqueadores para baixo.
2. Para cada item ✅, anote a referência `arquivo.js:linha` que prova a mitigação. **Sem âncora no código, o ✅ é só intenção** (essa é a lição-chave do relatório do BrieflyAI).
3. Rode as seções aplicáveis do `CONTEXT_ADDONS.md` para o contexto do projeto.
4. Anote tudo que escapou ao checklist em uma seção "Achados fora da lista".

### Passo 2 — Escrever o relatório

Use a estrutura dos relatórios existentes como template. Estrutura mínima:

1. **Cabeçalho com metadata**: data, método, escopo, resultado em uma frase.
2. **"Como ler este relatório"** — orienta o visitante.
3. **Bloco 1 — Confirmado**: tabela com `#`, item do checklist, arquivo:linha.
4. **Bloco 2 — Discrepâncias**: itens declarados mas não entregues (ou parciais).
5. **Bloco 3 — Achados fora da lista**: por severidade (🔴 Crítico / 🟠 Alto / 🟡 Médio / 🟢 Baixo).
6. **Resumo executivo**: tabela de contagem por categoria.
7. **Plano de ação ordenado por severidade**.
8. **Evolução dos checklists**: quais achados deste relatório viraram perguntas novas (referência cruzada para item criado).
9. **Reflexão final**: 1-2 parágrafos com aprendizado meta.

### Passo 3 — Salvar em dois lugares

**No repo do projeto auditado**:
```
meuprojeto/docs/AUDIT_REPORT_YYYY-MM-DD.md
```
Filename **sem** nome do projeto (o repo já identifica).

**Neste repo (showcase público)**:
```
REPOSITORIO DE SEGURANÇA-GITHUB/AUDIT_REPORT_NomeDoProjeto_YYYY-MM-DD.md
```
Filename **com** nome do projeto (vai ter vários relatórios coexistindo aqui — precisa desambiguar).

### Passo 4 — Atualizar o README

Adicione um parágrafo novo na seção `## 🔍 Auditorias reais`, no formato:

```markdown
### [`AUDIT_REPORT_NomeDoProjeto_YYYY-MM-DD.md`](AUDIT_REPORT_NomeDoProjeto_YYYY-MM-DD.md)

Uma frase de contexto (que tipo de projeto, qual stack). **Resultado:** números concretos
(X de Y itens confirmados, N% de match, K achados fora da lista — M deles viraram perguntas novas).
```

Mantenha o estilo dos parágrafos existentes (BrieflyAI, FlowSnyker) — números primeiro, jargão de marketing nunca.

### Passo 5 — Commit

```bash
git add AUDIT_REPORT_NomeDoProjeto_*.md README.md
git commit -m "docs: adicionar auditoria de NomeDoProjeto (YYYY-MM-DD)"
git push
```

---

## 🌱 Fluxo para evoluir os checklists

Quando uma auditoria revelar um bug que o checklist não pegou:

### Passo 1 — Decidir se merece virar item

Pergunta-teste honesta:
- **Sim**: a classe de bug é genérica o suficiente pra aparecer em outros projetos?
- **Sim**: o auditor próximo dificilmente pensaria nisso sem ser provocado?
- **Sim**: a mitigação tem receita acionável (não só "fique atento")?

Se as três respostas forem sim, vira item. Se for caso muito específico de um projeto, fica só no relatório como achado.

### Passo 2 — Adicionar o item

Decidir entre `AUDIT_CHECKLIST.md` (universal) ou `CONTEXT_ADDONS.md` (contexto específico).

Numeração: enquanto não houver renumeração geral, use sufixo de letra (`3B`, `9D`, `13B`) inserindo perto do tema relacionado. Não invente novo número alto.

Formato obrigatório do item:
- **Título** com o número e a pergunta.
- **Pergunta-teste** em itálico (cenário concreto que prova ou refuta a mitigação).
- **Opção 1 — Se [problema existe]**: passos de correção acionáveis.
- **Opção 2 — Se [está OK]**: ✅ Excelente.

Opcional: referência cruzada com itens relacionados (`🔗 Relacionado: item X — ...`).

### Passo 3 — Documentar a origem no relatório

Na seção "Evolução dos checklists" do relatório que originou o item, deixe explícito:

```markdown
| Achado | Item novo |
|--------|-----------|
| Descrição do bug | **XX** no `AUDIT_CHECKLIST.md` — "Pergunta..." |
```

Isso fecha o loop: visitante vê o item no checklist, pode rastrear o caso real que o gerou.

### Passo 4 — Commit

```bash
git add AUDIT_CHECKLIST.md  # ou CONTEXT_ADDONS.md
git add AUDIT_REPORT_NomeDoProjeto_*.md
git commit -m "feat: adicionar item XX ao checklist (derivado da auditoria de NomeDoProjeto)"
git push
```

---

## 📏 Regras pra manter o repo coerente

1. **Nunca edite auditoria antiga depois de publicada.** Se descobrir erro factual depois, adicione um adendo no fim do arquivo com data; se for grande, faça nova auditoria. Histórico de evolução vale mais que perfeição retroativa.

2. **Publica auditorias modestas também.** Se uma auditoria nova tem poucos achados (projeto pequeno, escopo curto), publica mesmo assim com honestidade — "auditoria curta, escopo X, achados modestos". Consistência de publicação vale mais que só os relatórios espetaculares.

3. **Numeração dos checklists vai inchar.** Com sufixos `3B, 6B, 6C, 9B, 9C, 11B, 13B, 13C, 17B...`, em algum momento (uns 5+ relatórios publicados) vale renumerar tudo sequencialmente numa versão 2.0. Antes disso, não compensa o trabalho.

4. **Não adicione itens de checklist sem caso real.** A regra de ouro do repo: se você não consegue apontar pra um achado de auditoria que justifica o item, ele não entra. "Adicionei porque achei boa ideia" enfraquece o método.

5. **Reflexão final do relatório é obrigatória.** É a parte que destila aprendizado meta — o que esse projeto te ensinou que você não sabia antes. Sem ela, o relatório é só checklist preenchido.

6. **Use português puro.** Mantenha consistência de idioma — todo o repo é em PT-BR. Termos técnicos em inglês ficam em código/inline (`throw`, `findOneAndUpdate`, `httpOnly`), prosa é em português.

7. **Honestidade sobre o que NÃO foi feito.** Se o projeto auditado tem um item legitimamente fora de escopo (não usa LLM, não tem upload), mencione isso explicitamente no cabeçalho do relatório. Recrutador valoriza saber o que ficou de fora e por quê.

---

## 🛠️ Comandos úteis

**Adicionar identidade git (uma vez por máquina):**
```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
```

**Atualizar repo após mudanças:**
```bash
git status
git add .
git commit -m "tipo: descrição curta"
git push
```

**Convenção de tipos de commit:**
- `feat:` adicionar item novo ao checklist
- `docs:` adicionar auditoria, atualizar README, melhorar prosa
- `fix:` corrigir erro ou inconsistência em item existente
- `refactor:` renumerar, reorganizar sem mudar conteúdo
