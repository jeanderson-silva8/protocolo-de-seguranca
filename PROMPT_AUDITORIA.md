# 🤖 PROMPT_AUDITORIA.md — Template pronto para pedir auditoria

> Template para colar no chat com o Claude (ou outro LLM agente) quando for auditar projeto novo.
> Garante extração ~95% do método — força as fases que escapam em pedidos informais (preparação, evidência, classificação, promoção de achados).
>
> **Como usar:** copie o bloco abaixo, substitua os `<<placeholders>>`, cole no chat.

---

```markdown
# 🔍 PEDIDO DE AUDITORIA DE SEGURANÇA

## Projeto a auditar
- **Nome:** <<NOME-DO-PROJETO>>
- **Pasta local:** <<C:\caminho\para\o\projeto>>
- **Stack:** <<ex: Node.js + Express + MongoDB + React + Vite>>
- **Estágio:** <<portfólio | MVP | produção>>
- **URL ao vivo (se houver):** <<https://...>>

## Framework a aplicar
- **Checklist universal:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\AUDIT_CHECKLIST.md` (56 itens sequenciais, 5 seções)
- **Adendos por contexto:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\context-addons\` (índice em `00_INDICE.md`)
- **Workflow do método:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\WORKFLOW.md`
- **Changelog:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\CHANGELOG.md`
- **Relatórios de referência (templates):** `AUDIT_REPORT_BrieflyAI_2026-05-16.md`, `AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md`, `AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md`, `AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md` (na mesma pasta)

## Decisões conscientes de escopo que eu já sei (NÃO reportar como bugs)
<<Liste aqui o que NÃO foi implementado de propósito. Exemplos:
- Multi-tenant não implementado porque sem pagamento real
- Celery/Redis mencionados como destino, não atual
- "Modo demo" público intencional
Se não há nenhuma, escreva: "nenhuma — me pergunte se encontrar algo que pareça decisão consciente">>

## O que quero de você (siga TODAS as fases)

### Fase 0 — Preparação
1. Leia os 5 arquivos do framework acima COMPLETAMENTE antes de começar.
2. Leia ao menos UM dos 4 relatórios de referência para internalizar o tom e a estrutura.
3. Se houver auditoria anterior em `<<pasta-do-projeto>>/docs/AUDIT_REPORT_*.md`, leia também — esta vai ser comparativa (v2).

### Fase 1 — Aplicação do checklist
4. Vasculhe o projeto exaustivamente.
5. Aplique o `AUDIT_CHECKLIST.md` da seção 🔴 Bloqueadores até a 🎨 Frontend.
6. Use `00_INDICE.md` para identificar quais adendos (A-J) se aplicam.
7. Rode os adendos aplicáveis E leia a seção `🔗 Adendos relacionados` no fim de cada um.
8. **OBRIGATÓRIO — Tabela de cobertura completa.** Ao final do relatório, inclua uma **"Matriz de cobertura"** com 1 linha para CADA UM dos 57 itens do `AUDIT_CHECKLIST.md` + adendos rodados, mostrando status: `✅ Confirmado` / `🟠 Parcial` / `🔴 Violado` / `🚫 N/A (motivo)`. Sem essa tabela, é impossível saber se itens importantes ficaram fora da análise por esquecimento vs por escopo. Lição: auditoria do TrendScope 2026-05-18 cobriu 38 de 57 itens explicitamente — os 19 restantes ficaram em zona ambígua e exigiram 2ª passada. Para CADA adendo NÃO rodado, registre explicitamente "Adendo X — N/A: [motivo]".

### Fase 2 — Disciplina de evidência (NÃO PULE)
8. Para CADA item ✅, anote `arquivo.ext:linha` que prova a mitigação. **Sem âncora, o ✅ é só intenção — não conta.**
9. Para cada problema encontrado, classifique em:
   - 🔴 **Bug real** → vai pro plano de correção
   - 📝 **Decisão consciente do autor** → vira ADR sugerido, não bug
   - 🚫 **N/A por escopo declarado** → registra com justificativa

### Fase 3 — Achados fora do checklist
10. Liste TODOS os bugs/riscos que encontrou e que o checklist NÃO cobre, categorizados por severidade (🔴 Crítico / 🟠 Alto / 🟡 Médio / 🟢 Baixo).

### Fase 4 — Relatório
11. Escreva o relatório no MESMO formato dos 4 referenciados, com EXATAMENTE esta estrutura:
    - Cabeçalho (data, método, escopo, resultado em uma frase)
    - "Como ler este relatório"
    - "Nota sobre escopo" (se houver decisões conscientes — referencia ADRs)
    - **Bloco 1** — Confirmado e excelente (tabela # | item | arquivo:linha)
    - **Bloco 2** — Parcial (itens iniciados mas incompletos)
    - **Bloco 3** — Críticos / violações (com PoC textual quando possível)
    - **Bloco 4** — Polimentos (baixa criticidade)
    - **Itens N/A** com justificativa
    - **Resumo executivo** (tabela de contagem)
    - **Plano de ação ordenado por severidade**
    - **Evolução dos checklists** (se algum achado merece virar pergunta nova)
    - **Reflexão final** (1-2 parágrafos com aprendizado meta)

### Fase 5 — Promoção de achados ao checklist universal
12. Se algum achado é classe NOVA de bug que outros projetos terão, proponha pergunta nova para o `AUDIT_CHECKLIST.md` ou `context-addons/X.md` seguindo as regras do `WORKFLOW.md`:
    - Origem rastreável (este relatório)
    - Pergunta-teste verificável
    - Receita acionável (não só "fique atento")
    - Não duplica item existente (se duplicar, é cross-reference, não item novo)
    - ⚠️ **NUMERAÇÃO:** insira como número sequencial (não com sufixo B/C). Se o item entra entre o 16 e o 17, ele vira o NOVO 17 e todos os demais sofrem shift +1. Atualize TOC, ranges das seções (ex: "BLOQUEADORES (1-17)" → "(1-18)"), cross-refs internas, nota de migração. **NÃO use sufixos como `17B`** — isso regrediu a v1.2.0 quando aconteceu no TrendScope 2026-05-18; vide CHANGELOG `[1.5.0]`.

### Fase 6 — Entrega
13. Salve o relatório em DOIS lugares:
    - `<<pasta-do-projeto>>\docs\AUDIT_REPORT_<<DATA-HOJE>>.md` (filename sem nome do projeto)
    - `c:\Users\pedro\OneDrive\Área de Trabalho\REPOSITORIO DE SEGURANÇA-GITHUB\AUDIT_REPORT_<<NOME-DO-PROJETO>>_<<DATA-HOJE>>.md` (filename com nome)
14. Atualize o `README.md` do `REPOSITORIO DE SEGURANÇA-GITHUB` adicionando uma entrada nova na seção "🔍 Auditorias reais" (parágrafo com números concretos).
15. Se houver itens novos do checklist propostos no passo 12, atualize `AUDIT_CHECKLIST.md` (ou adendo correspondente) E o `CHANGELOG.md` com a entrada da versão nova (MINOR bump).
16. Apresente no chat um resumo executivo com:
    - Quantos itens em cada status
    - Top 3 críticos com 1 linha cada
    - Quantos itens novos foram promovidos ao checklist
    - Comandos de commit sugeridos para subir as mudanças

## Regras inegociáveis (não negocie comigo se eu pedir pra suavizar)
- **Honestidade > marketing.** Se o projeto for fraco, diga. Se promete o que não entrega, aponte. Se autor declara X mas código tem Y, registre como discrepância — nem como fraude nem como detalhe.
- **Nenhum ✅ sem `arquivo:linha`.** Memória não conta. Se você não conseguiu achar o código que prova, o item NÃO é ✅.
- **Decisões de escopo conscientes ≠ bugs.** Pergunte se tiver dúvida.
- **Não invente mitigações.** Se a mitigação não existe no código, não escreva que existe.
- **Use o tom dos relatórios de referência** — sereno, técnico, sem jargão privado, sem hype.

## Após entregar a auditoria
Se o autor (eu) corrigir os bugs depois, eu vou voltar a você pedindo uma **auditoria v2 comparativa** (padrão FlowSnyker v2 / Lumina v2). Não corrija nada AGORA — a v1 documenta o estado atual sem alterações.
```

---

## 📝 Notas sobre o uso

### Quando substituir os placeholders

| Placeholder | Exemplo |
|-------------|---------|
| `<<NOME-DO-PROJETO>>` | `MeuApp-Backend` (use o mesmo nome da pasta do repo) |
| `<<C:\caminho\para\o\projeto>>` | `c:\Users\pedro\OneDrive\Área de Trabalho\MeuApp-Backend` |
| `<<ex: Node.js + Express + ...>>` | A stack real do seu projeto |
| `<<portfólio \| MVP \| produção>>` | Escolha uma — afeta a calibração da severidade |
| `<<https://...>>` | URL do deploy ou "sem deploy ao vivo" |
| `<<DATA-HOJE>>` | Data ISO `2026-05-18` |
| Decisões de escopo | Liste o que NÃO foi implementado de propósito |

### Quando NÃO usar esse prompt

- **Audit rápida** ("dá uma olhada se a senha está sendo hasheada certo") — pedido pontual, não vale invocar o protocolo inteiro.
- **Code review de PR** — use o checklist como referência mental, mas não como protocolo formal.
- **Discussão de arquitetura** ("vale a pena trocar JWT por sessão server?") — é decisão, não auditoria.

### Quando usar mesmo se parecer demais

- **Antes de mostrar projeto pra recrutador** — você quer saber o que está exposto.
- **Antes de aceitar contrato/freela com responsabilidade de segurança** — você quer saber o que herdou.
- **Toda vez que sair de um marco grande** (MVP → 1.0, primeiro tenant pagante, primeira integração de pagamento real).
- **Comparando seu código antes/depois de um refactor de segurança** (v1 → v2).

### O que esperar de tempo

- Auditoria v1 de projeto médio (~5k linhas): 30-60 minutos de processamento + leitura do relatório.
- Auditoria v2 comparativa: 15-30 minutos (foco em validar entregas + buscar regressões).
- Aplicação das correções: depende do plano, normalmente 7-15h distribuídas em 5-7 sessões (ver `AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md` para exemplo de sessão por sessão).

---

## 🔗 Relacionados

- [`WORKFLOW.md`](WORKFLOW.md) — processo completo do método (13 fases, regras de coerência, comandos úteis)
- [`AUDIT_CHECKLIST.md`](AUDIT_CHECKLIST.md) — checklist universal
- [`context-addons/00_INDICE.md`](context-addons/00_INDICE.md) — matriz "stack → adendo aplicável"
- [`CHANGELOG.md`](CHANGELOG.md) — histórico de evolução do framework
