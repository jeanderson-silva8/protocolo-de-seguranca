# 🧪 Adendo H — Projetos com dados sensíveis (saúde, financeiro, jurídico)

> Adendo do `AUDIT_CHECKLIST.md`. Aplique se o projeto manipula PII real, dados de saúde, financeiros ou jurídicos.
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### H1. Dados em repouso são criptografados (database encryption)?

> ⚠️ **Entenda o escopo antes de marcar como resolvido:**
> - TDE / encryption-at-rest protege contra um cenário específico: **roubo físico do disco, snapshot vazado, descarte indevido de hardware, ou acesso direto aos arquivos do banco**.
> - **NÃO protege contra**: SQL injection, credencial de banco vazada, aplicação comprometida, dump via query legítima, backup desprotegido, ou qualquer ataque que use a conexão "normal" ao banco — porque nesses casos o banco descriptografa os dados antes de servi-los, como faria para qualquer cliente legítimo.
> - É uma camada de defesa em profundidade, não uma bala de prata.
>
> **Opção 1 — Se banco está sem encryption-at-rest:**
> - Habilitar TDE/encryption no provedor (MongoDB Atlas, RDS, etc.) — geralmente um toggle
> - **Para defesa real contra os ataques acima**: criptografia em nível de aplicação para campos especialmente sensíveis (PII, dados de saúde, financeiros) — assim o banco armazena ciphertext mesmo do ponto de vista da aplicação
> - Chaves gerenciadas em KMS (AWS KMS, GCP KMS, HashiCorp Vault) — nunca no código nem em env var
> - Rotação periódica de chaves documentada
>
> **Opção 2 — Se há encryption-at-rest + criptografia em nível de aplicação para campos críticos:** ✅ Excelente

---

### H2. PII é minimizada (coletar apenas o necessário)?

> **Opção 1 — Se coleta dados "para o caso de precisar":**
> - Revisar campos coletados
> - Eliminar o que não tem uso ativo
> - Documentar finalidade de cada campo de PII (LGPD/GDPR exige)
> - Anonimizar/pseudonimizar onde possível
>
> **Opção 2 — Se coleta é mínima e justificada:** ✅ Excelente

---

### H3. Há separação de PII em tabela/coleção dedicada com acesso restrito?

> **Opção 1 — Se PII está espalhada por todas as tabelas:**
> - Centralizar em tabela `personal_info` com acesso restrito
> - Outras tabelas referenciam por ID
> - Permite anonimização (zerar a tabela de PII mantendo análises)
>
> **Opção 2 — Se PII é segregada:** ✅ Excelente

---

### H4. Existe relatório de acesso a dados sensíveis (quem acessou, quando, o quê)?

> **Opção 1 — Se acessos a PII não são logados:**
> - Audit log específico para acesso a dados sensíveis
> - Granularidade: registro por leitura, não só por modificação
> - Acessível para o titular (transparência — LGPD/GDPR)
>
> **Opção 2 — Se há audit log de acesso a PII:** ✅ Excelente
