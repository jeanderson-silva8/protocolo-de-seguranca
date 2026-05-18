# 📁 Adendo B — Upload de arquivos

> Adendo do `AUDIT_CHECKLIST.md`. Aplique apenas se o projeto aceita upload de arquivos pelo usuário.
> Ver [`00_INDICE.md`](00_INDICE.md) para a lista completa de adendos por contexto.

---

### B1. Tipos MIME aceitos são validados por allowlist (não blocklist)?

> **Opção 1 — Se aceita tudo ou bloqueia apenas alguns tipos:**
> - Definir allowlist explícita: `['image/png', 'image/jpeg', 'application/pdf']`
> - **NÃO confiar no header `Content-Type`** — pode ser forjado
> - Validar pela assinatura mágica do arquivo (magic bytes) — use `file-type` ou similar
> - Rejeitar tudo que não bater
>
> **Opção 2 — Se há allowlist + validação por magic bytes:** ✅ Excelente

---

### B2. Há limite estrito de tamanho de arquivo?

> **Opção 1 — Se não há limite ou é muito alto:**
> - Limite por endpoint (ex: avatares ≤ 2MB, documentos ≤ 10MB)
> - Configurar no nível do framework (Multer, Busboy) + no proxy/load balancer
> - Limite de tamanho agregado por usuário (cota)
>
> **Opção 2 — Se há limites em camadas:** ✅ Excelente

---

### B3. Arquivos são armazenados FORA do diretório servido publicamente?

> **Opção 1 — Se uploads vão para `public/`, `static/`:**
> - **CRÍTICO** — qualquer upload é servido como conteúdo do site
> - Armazenar em S3/R2/Cloud Storage, ou pasta privada
> - Servir via signed URLs com expiração
> - Aplicação intermedeia download (com autorização)
>
> **Opção 2 — Se storage é isolado:** ✅ Excelente

---

### B4. Nome do arquivo salvo é gerado pelo servidor (não usa o nome enviado)?

> **Opção 1 — Se usa `req.file.originalname`:**
> - **Risco de path traversal** (`../../etc/passwd`) e XSS via nome
> - Gerar nome próprio: UUID ou hash + extensão validada
> - Sanitizar extensão estritamente (só letras/números, allowlist)
>
> **Opção 2 — Se servidor gera nome:** ✅ Excelente

---

### B5. Para imagens, há reprocessamento (resize/recompress) para neutralizar payloads escondidos?

> **Opção 1 — Se imagens são salvas como recebidas:**
> - **Riscos reais**:
>   - **Polyglot files**: arquivo que é simultaneamente uma imagem válida e um HTML/JS/PHP válido. Servido com o Content-Type errado (ou aberto pelo navegador adivinhando tipo), executa como script.
>   - **Metadados (EXIF/IPTC/XMP) com payloads** projetados para explorar parsers vulneráveis downstream (libs de leitura de EXIF, indexadores, geradores de thumbnail) — não "scripts" no sentido executável, mas vetores de RCE em libs com CVEs.
>   - **Exploits de parser de imagem** (ImageTragick e similares): a própria decodificação do arquivo dispara o exploit.
> - **Mitigação**: processar com Sharp / libvips / ImageMagick (com policy.xml restritivo): redimensionar, recompactar, **strip de todos os metadados**
> - Salvar a versão processada, não a original
> - Servir sempre com `Content-Type` correto e `X-Content-Type-Options: nosniff`
>
> **Opção 2 — Se imagens são reprocessadas:** ✅ Excelente

---

### B6. Há scan antivírus em arquivos não-imagem (PDFs, docs, executáveis)?

> **Opção 1 — Se arquivos arbitrários são aceitos sem scan:**
> - Integrar ClamAV (`clamscan`, `clamdjs`)
> - Ou usar serviço (VirusTotal API, AWS GuardDuty Malware Protection)
> - Quarentena antes de disponibilizar
>
> **Opção 2 — Se há scan:** ✅ Excelente

---

### B7. Endpoints de download verificam autorização do solicitante?

> **Opção 1 — Se URL direta basta para baixar:**
> - **Risco de IDOR** em uploads (URLs previsíveis vazam arquivos)
> - Servir via endpoint autenticado que valida ownership
> - Ou signed URLs de curta duração (S3 pre-signed URLs)
>
> **Opção 2 — Se downloads são autorizados:** ✅ Excelente

---

## 🔗 Adendos relacionados

- **B7 (download authz)** ↔ É a versão "arquivo" da autorização granular a recurso. Mesma classe em outras superfícies:
  - **[A3 (room control em socket)](A_WEBSOCKET.md)**
  - **[D3 (no cross-tenant)](D_MULTI_TENANT.md)**
  - **[E4 (RAG tenant-aware)](E_LLM.md)**
  - Princípio universal: **item 2** do `AUDIT_CHECKLIST.md` (autorização em toda operação) + **item 33** (IDs com formato estrito — chave de URL não deve ser previsível).
- **B5 (reprocessar imagens) + B6 (scan AV)** ↔ Se o upload alimenta um LLM, ver **[E2 (validar output do LLM)](E_LLM.md)** — payload malicioso pode atravessar via prompt injection.
- **B1 (allowlist por magic bytes)** ↔ Princípio universal: **item 5** do checklist (input validation por biblioteca antes do controller).
- **B3 (storage isolado)** ↔ Se o link de download é JWT-signed, confirmar **item 10** (JWT com algoritmo seguro).
