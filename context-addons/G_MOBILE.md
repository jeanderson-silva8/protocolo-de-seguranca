# 📱 Adendo G — Mobile (React Native, Flutter, nativo)

> Adendo do `AUDIT_CHECKLIST.md`. Aplique se o projeto tem app mobile (iOS, Android, ou cross-platform).
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### G1. Certificate pinning está ativado para conexões críticas?

> **Opção 1 — Se app aceita qualquer certificado válido:**
> - Implementar SSL pinning para endpoints sensíveis
> - Previne ataques MITM mesmo com CA comprometida
> - Cuidado com rotação de certificados (pinning de chave pública, não certificado)
>
> **Opção 2 — Se pinning está ativo:** ✅ Excelente

---

### G2. Segredos NÃO estão hard-coded no binário?

> *Apps podem ser descompilados — qualquer string no código é pública.*
>
> **Opção 1 — Se há API keys, secrets, ou URLs sensíveis no código:**
> - **CRÍTICO** — extração trivial via decompilação
> - Mover segredos para o backend (proxy via servidor)
> - Para secrets que precisam estar no app, usar Keystore (Android) / Keychain (iOS)
> - Obfuscação não substitui — só atrasa atacante
>
> **Opção 2 — Se não há secrets no binário:** ✅ Excelente

---

### G3. Dados sensíveis em armazenamento local são criptografados?

> **Opção 1 — Se tokens/dados ficam em AsyncStorage/SharedPreferences cru:**
> - Migrar para Keychain (iOS) / EncryptedSharedPreferences (Android)
> - Bibliotecas: `react-native-keychain`, `flutter_secure_storage`
> - Considerar biometria para acesso a dados críticos
>
> **Opção 2 — Se storage local é criptografado:** ✅ Excelente

---

## 🔗 Adendos relacionados

- **G2 (sem secrets no binário)** ↔ Princípio universal: **item 13** do `AUDIT_CHECKLIST.md` (segredos fora do código versionado). Reforço móvel: binário é "fora do código" só na teoria — qualquer string descompila.
- **G3 (storage local criptografado)** ↔ Mesma classe "onde o token vive" no frontend web:
  - **item 52** do checklist (onde o access token é armazenado)
  - **item 53** (guard de rota valida o token)
  - **[H1 (encryption-at-rest no banco)](H_DADOS_SENSIVEIS.md)** — mesma defesa em outra camada
- **G1 (certificate pinning)** ↔ Princípio universal: **item 31** (headers de segurança HTTP — HSTS é o equivalente web).

> ⚠️ **Adendo intencionalmente conciso.** Mobile profundo tem muito mais (jailbreak/root detection, screen recording protection, deep link hijacking, intent injection no Android, App Attestation, Play Integrity). Vão entrar quando uma auditoria em projeto mobile real ditar — não força agora pra evitar conteúdo sem rastro de origem. Ver `WORKFLOW.md` (regra: nunca adicionar item sem caso real).
