# 🛡️🛡️ PROMPT_AUDITORIA_PARANOICO.md — Template para projetos críticos

> Versão estendida do [`PROMPT_AUDITORIA.md`](PROMPT_AUDITORIA.md) que exige **passada dupla automática** no mesmo prompt: 1ª passada normal + auto-revisão adversarial da matriz.
>
> **Quando usar:** projetos em produção real com clientes pagantes, sistemas financeiros, apps de saúde, qualquer coisa onde o custo de um bug escapar é alto demais pra confiar em ~85%.
>
> **Quando NÃO usar:** projetos de portfólio, MVPs, demos — o template padrão dá conta com ~85% real, e o custo extra (2× tempo de execução, ~30% mais tokens) não compensa.
>
> **Cobertura esperada:** ~95% real (vs ~85% do template padrão). Custo: ~2× tempo + ~30% tokens.

---

## 🎯 Quando vale o paranoico vs. o padrão

| Cenário | Template recomendado |
|---------|---------------------|
| Portfólio / MVP / demo | [PROMPT_AUDITORIA.md](PROMPT_AUDITORIA.md) (padrão) |
| Antes de mostrar pra recrutador | Padrão |
| Antes de aceitar contrato/freela | Padrão |
| Antes de marco grande (1.0, primeiro tenant) | **Paranoico** |
| Sistema com dinheiro real fluindo | **Paranoico** |
| App com PII real (saúde, financeiro, jurídico) | **Paranoico** |
| Pós-incidente de segurança | **Paranoico** (auditoria forense também) |
| Pré-due-diligence (venda da empresa, auditoria externa contratada) | **Paranoico** |

---

## 📋 Template paronoico (copia + cola)

```markdown
# 🛡️🛡️ PEDIDO DE AUDITORIA DE SEGURANÇA — MODO PARANOICO

## Projeto a auditar
- **Nome:** <<NOME-DO-PROJETO>>
- **Pasta local:** <<C:\caminho\para\o\projeto>>
- **Stack:** <<ex: Node.js + Express + MongoDB + React + Vite>>
- **Estágio:** <<produção crítica | pré-due-diligence | pós-incidente | outro>>
- **URL ao vivo (se houver):** <<https://...>>
- **Por que paranoico?** <<ex: clientes pagantes reais; PII de saúde; sistema financeiro; pós-incidente de SQL injection em 2026-04>>

## Framework a aplicar
- **Checklist universal:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\AUDIT_CHECKLIST.md` (57 itens sequenciais, 5 seções)
- **Adendos por contexto:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\context-addons\` (índice em `00_INDICE.md`)
- **Workflow do método:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\WORKFLOW.md`
- **Changelog:** `c:\Users\pedro\OneDrive\Área de Trabalho\Protocolo de Segurança\CHANGELOG.md`
- **Relatórios de referência:** `AUDIT_REPORT_BrieflyAI_2026-05-16.md`, `AUDIT_REPORT_FlowSnyker_v2_2026-05-16.md`, `AUDIT_REPORT_Lumina-Booking-SaaS_2026-05-17.md`, `AUDIT_REPORT_Lumina-Booking-SaaS_v2_2026-05-18.md`, `AUDIT_REPORT_TrendScope_2026-05-18.md` (este último é EXCELENTE template de modo paranoico — tem matriz completa de 57 itens + lições da 2ª passada incorporadas)

## Decisões conscientes de escopo que eu já sei (NÃO reportar como bugs)
<<Liste decisões intencionais. Se não houver, escreva "nenhuma — me pergunte se encontrar algo que pareça decisão consciente">>

---

## ⚠️ MODO PARANOICO — você vai executar DUAS PASSADAS no mesmo turno

A diferença pro modo padrão é que a 2ª passada (auto-revisão adversarial) **não é opcional** — é parte do mesmo trabalho. Você não entrega até as duas estarem completas.

---

## 🔵 PASSADA 1 — Auditoria normal (idêntica ao PROMPT_AUDITORIA padrão)

### Fase 0 — Preparação
1. Leia os 5 arquivos do framework COMPLETAMENTE antes de começar.
2. Leia o `AUDIT_REPORT_TrendScope_2026-05-18.md` para internalizar o tom + a matriz de cobertura obrigatória.
3. Se houver auditoria anterior em `<<pasta-do-projeto>>/docs/AUDIT_REPORT_*.md`, leia também.

### Fase 1 — Aplicação do checklist
4. Vasculhe o projeto exaustivamente.
5. Aplique o `AUDIT_CHECKLIST.md` da seção 🔴 Bloqueadores até a 🎨 Frontend (todos os 57 itens).
6. Use `00_INDICE.md` para identificar adendos (A-J) aplicáveis.
7. Rode os adendos aplicáveis E leia a seção `🔗 Adendos relacionados` no fim de cada um.
8. **OBRIGATÓRIO — Matriz de cobertura completa.** Tabela com 1 linha para CADA UM dos 57 itens + adendos rodados/N/A, status (`✅`/`🟠`/`🔴`/`🚫`) + arquivo:linha onde aplicável.

### Fase 2 — Disciplina de evidência
9. Para CADA ✅, anote `arquivo:linha` que prova. **Sem âncora, NÃO é ✅.**
10. Classifique problemas em 🔴 bug real / 📝 decisão consciente / 🚫 N/A por escopo.

### Fase 3 — Achados fora do checklist (raciocínio adversarial)
11. Foque especialmente em:
    - **Supply chain:** `package.json` deps obscuras, `Dockerfile` com flags inseguras (`strict-ssl false`, registry não-oficial), CI puxando de fontes não-confiáveis
    - **Side channels:** timing leaks, error messages diferentes para "usuário existe" vs "não existe", cache que vaza
    - **Multi-entrypoint:** se app tem 2+ formas de entrar (Vercel function + dev server, Lambda + EC2, browser + mobile), TODAS têm as mesmas defesas?
    - **State injetado:** env vars, cookies, headers — qual deles o usuário controla? algum dispara comportamento privilegiado?

### Fase 4 — Relatório (estrutura padrão)
12. Cabeçalho, "Como ler", "Nota sobre escopo", Bloco 1-4, N/A, Resumo executivo, Plano de ação, Evolução dos checklists, **Reflexão final** (obrigatória), **Matriz de cobertura completa**.

---

## 🔴 PASSADA 2 — Auto-revisão adversarial (obrigatória no modo paranoico)

Depois de terminar a Passada 1, você vai **se auditar**. Aja como se fosse outro auditor que recebeu o relatório e tem 1h pra encontrar o que escapou.

### Fase 5 — Auto-revisão da matriz (sistemática)
13. **Re-leia a Matriz de Cobertura inteira** e, para CADA linha, responda:
    - Se marcou **✅**: a âncora `arquivo:linha` **realmente** prova a mitigação em PRODUÇÃO, ou prova só em dev/test/staging? (Lição TrendScope: `server/boot.ts` tinha CORS+headers mas era ignorado em prod — `api/trpc/[...trpc].ts` que servia o tráfego não tinha nenhum.)
    - Se marcou **🚫 N/A**: existe alguma feature do projeto que **eu desconheço** que talvez ative esse item? Ex: marquei "sem upload" mas tem alguma rota `/avatar` que aceita arquivo?
    - Se marcou **🟠 parcial**: o gap é genuinamente parcial ou é "começou e abandonou" (que é pior — gera falsa sensação de segurança)?

### Fase 6 — Caça-aos-fantasmas (procurar o que não procurei)
14. Re-leia o relatório com lentes específicas que a Passada 1 pode ter ignorado:
    - **Defaults inseguros**: Procurar `process.env.X || 'default'`, `os.environ.get('X', 'default')`, `${VAR:-default}` em qualquer arquivo de config/compose. Cada um é uma violação potencial do item 8 (fail-fast em orquestração).
    - **Comentários `TODO`/`FIXME`/`XXX`/`HACK` em código de segurança**: cada um é uma dívida documentada — vale flag.
    - **Strings hardcoded suspeitas**: `localhost`, `admin`, `test`, `password123`, `change-me`, URLs `.local`, IPs privados (`10.x`, `172.16-31.x`, `192.168.x`) em código que vai pra produção.
    - **Arquivos com nomes suspeitos**: `_old`, `_backup`, `_temp`, `.bak`, `.orig`, `_test_real`, `gemini.md`/`task_plan.md` (lição Lumina/TrendScope: docs internos vazam).
    - **`.env`, `.env.local`, `.env.production`**: estão no `.gitignore`? `git log --all --full-history -- .env*` mostra que NUNCA foram commitados?
    - **Permissões de arquivo no Dockerfile**: `COPY --chown=root:root` (perigoso) vs `USER nobody` (bom).
    - **Binários no `node_modules` na imagem final**: imagem multi-stage que copia `node_modules` inteiro vs só `node_modules/production`.

### Fase 7 — Verificação de amostragem dos ✅
15. **Escolha 3 itens ✅ aleatoriamente** da matriz. Para cada um:
    - Abra o `arquivo:linha` âncora de novo
    - Confirme que o código FAZ o que o item exige (não algo similar)
    - Confirme que esse código é executado em produção (não está atrás de `if (process.env.NODE_ENV !== 'production')` ou similar)
    - Se algum falhar, downgrade pra 🟠 ou 🔴 e documente na seção "Correções da auto-revisão"

### Fase 8 — Aplicação cruzada dos achados
16. Para CADA achado da Passada 1, pergunte: "esse mesmo padrão pode aparecer em OUTRO lugar do código?" Ex: se achou `console.log(req.body)` num endpoint, busque por TODOS os `console.log(req.body)` ou `console.log(req)` em qualquer lugar.

### Fase 9 — Relatório de auto-revisão (adicionar ao mesmo arquivo)
17. Adicione ao relatório uma seção nova chamada **"🔴 Passada 2 — Auto-revisão adversarial"** com:
    - **Subseção "Reclassificações"**: itens que mudaram de status na auto-revisão (✅ → 🟠, etc.) com justificativa
    - **Subseção "Achados novos (caça-aos-fantasmas)"**: bugs descobertos na Fase 6
    - **Subseção "Verificação de amostragem"**: os 3 ✅ checados, qual cada um virou
    - **Subseção "Aplicação cruzada"**: padrões da Passada 1 que apareceram em outros lugares
    - **Resumo executivo atualizado**: tabela de contagem antes/depois da auto-revisão
    - **Plano de ação atualizado**: incorporando os novos achados

### Fase 10 — Reflexão meta sobre as duas passadas
18. Última parte do relatório: **"🧭 Reflexão meta sobre a auto-revisão"** com 1-2 parágrafos sobre:
    - Quantos achados da Passada 1 sobreviveram à Passada 2 vs. quantos foram reclassificados/expandidos?
    - Qual padrão a Passada 1 mais errou (false ✅, missed achados, etc.)?
    - O que isso sugere de evolução pro PROMPT_AUDITORIA padrão?

---

## Fase 11 — Promoção de achados ao framework universal
19. Se algum achado das Passadas 1 ou 2 é classe NOVA, proponha pergunta nova. Mesmas regras do PROMPT padrão:
    - Origem rastreável
    - Pergunta-teste verificável
    - Receita acionável
    - Não duplica item existente
    - ⚠️ **NUMERAÇÃO:** insira como número sequencial (NÃO use sufixos B/C). Se entra entre 16 e 17, vira NOVO 17 e todos shift +1. Atualize TOC, ranges, cross-refs, nota de migração.

## Fase 12 — Entrega
20. Salve o relatório em DOIS lugares:
    - `<<pasta-do-projeto>>\docs\AUDIT_REPORT_<<DATA-HOJE>>.md`
    - `c:\Users\pedro\OneDrive\Área de Trabalho\REPOSITORIO DE SEGURANÇA-GITHUB\AUDIT_REPORT_<<NOME-DO-PROJETO>>_<<DATA-HOJE>>.md`
21. Atualize o `README.md` do `REPOSITORIO DE SEGURANÇA-GITHUB` (seção "🔍 Auditorias reais") — mencione **explicitamente que foi modo paranoico** e que o relatório tem as 2 passadas integradas.
22. Se houver itens novos: atualize `AUDIT_CHECKLIST.md` e `CHANGELOG.md` (MINOR bump). Não sincronize com `Protocolo de Segurança/` — eu faço depois via rsync.
23. Apresente no chat:
    - Resumo executivo final (após auto-revisão)
    - Quantos achados a Passada 1 deixou passar (que a Passada 2 pegou)
    - Top 3 críticos com 1 linha cada
    - Quantos itens novos foram promovidos
    - Comandos de commit sugeridos

---

## Regras inegociáveis (mais rígidas que o padrão)

- **Honestidade > marketing.** Aceitável em modo padrão; OBRIGATÓRIO em paranoico.
- **Nenhum ✅ sem `arquivo:linha`** — em modo paranoico, ✅ adicional: confirma que o código é executado em PRODUÇÃO (não dev/test).
- **Decisões conscientes ≠ bugs** — mas em modo paranoico, pergunte AO USUÁRIO sobre cada decisão suspeita antes de classificar como N/A.
- **Não invente mitigações** — em paranoico, se na Passada 1 você marcou ✅ baseado em "deve estar ok", a Passada 2 vai pegar isso.
- **Tom sereno e técnico.** Sem hype, sem jargão privado.
- **Auto-revisão é obrigatória** — não entregue sem a seção "Passada 2".
- **Se a Passada 2 não achou nada novo, anote isso explicitamente** ("Passada 2: 0 achados novos, 0 reclassificações — Passada 1 estava sólida"). Isso é sinal positivo, não falta de esforço.

## Após entregar a auditoria
Se o autor (eu) corrigir os bugs, vou pedir auditoria v2 comparativa (padrão FlowSnyker v2 / Lumina v2). **Não corrija nada agora** — a v1 paranoica documenta o estado atual sem alterações.
```

---

## 📝 Notas sobre o uso

### Como o paranoico difere do padrão (resumo executivo)

| Aspecto | Padrão | Paranoico |
|---------|--------|-----------|
| Número de passadas | 1 | 2 (no mesmo turno) |
| Matriz de cobertura | Obrigatória | Obrigatória + auto-revisada |
| Verificação de ✅ | Âncora arquivo:linha | Âncora + confirmação de execução em prod |
| Achados fora do checklist | Fase 3 padrão | Fase 3 + Fase 6 (caça-aos-fantasmas com lentes específicas) |
| Amostragem de verificação | Não exigida | 3 ✅ aleatórios re-verificados |
| Aplicação cruzada de padrões | Implícita | Explícita (Fase 8) |
| Reflexão meta | Sobre o projeto | Sobre o projeto + sobre a própria auditoria |
| Cobertura esperada | ~85% real | ~95% real |
| Tempo de execução | 1× | ~2× |
| Tokens consumidos | 1× | ~1.3× |

### Sinais de que a Passada 2 fez seu trabalho

Esperado: Passada 2 acha entre 0 e 5 achados novos por projeto. Se achar 0, anota "Passada 1 estava sólida". Se achar muito (>10), é sinal que a Passada 1 foi superficial — vale rodar uma 3ª passada ou revisar a metodologia.

Histórico de referência (TrendScope foi auditado com modo "padrão + 2ª passada manual", que aproxima do paranoico):
- Passada 1: 5 críticos + 3 altos + 5 médios
- Passada 2: +6 críticos novos (incluindo C14 supply chain) + matriz completa
- **Conclusão:** auto-revisão dobrou os críticos encontrados. Justifica o modo paranoico pra projetos críticos.

### Quando NÃO vale o paranoico

- Auditoria rápida pra responder pergunta pontual ("está sanitizando input?"): use o checklist como referência mental, não como protocolo.
- Code review de PR pequeno: items isolados, não vale.
- Projeto de aprendizado / hobby: gap de 15% é aceitável.
- Quando o autor já fez auditoria do padrão recentemente: paranoico pode ser usado como "v2 comparativa", mas aí a estrutura é a do FlowSnyker v2.

---

## 🔗 Relacionados

- [`PROMPT_AUDITORIA.md`](PROMPT_AUDITORIA.md) — versão padrão (~85% real, 1× tempo)
- [`WORKFLOW.md`](WORKFLOW.md) — método humano (independente de LLM)
- [`AUDIT_REPORT_TrendScope_2026-05-18.md`](AUDIT_REPORT_TrendScope_2026-05-18.md) — exemplo real onde "padrão + 2ª passada manual" achou 6 críticos novos. O modo paranoico transforma isso em padrão automatizado.
- [`CHANGELOG.md`](CHANGELOG.md) — origem (lição da 2ª passada do TrendScope que motivou a criação deste arquivo)
