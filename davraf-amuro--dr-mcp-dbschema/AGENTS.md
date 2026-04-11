# Copilot Instructions (AI Agent)

Progetto: Minimal API Template su .NET 10 e C# 14.

## Stack e architettura
- Minimal APIs + Asp.Versioning (UrlSegmentApiVersionReader)
- Scalar per documentazione, ProblemDetails per errori
- Struttura base: src/<project>/Program.cs, Endpoints/, Infrastructure/

## Convenzioni essenziali
- Endpoint in extension methods: app.Map{Domain}Endpoints(versionSet)
- Versioning URL: /api/v{version}/...
- OpenAPI in Development, Scalar UI
- Primary constructors, async/await per I/O
- Logging strutturato con placeholder
- Naming: namespace snake_case, classi PascalCase, variabili camelCase

## ✅ Checklist Post-Generazione
- [ ] Ho seguito le istruzioni modulari pertinenti
- [ ] Endpoint creati come extension methods e mappati in Program.cs
- [ ] Versioning URL segment configurato correttamente
- [ ] OpenAPI + Scalar coerenti con le regole del progetto
- [ ] Logging strutturato e async/await usati dove serve

## ✅ Verifica post-modifica (qualsiasi file)
Dopo ogni modifica a un file:
1. Rileggi il file modificato
2. Confronta il contenuto con quanto richiesto
3. Solo se corrispondono, dichiara la modifica completata

## Ciclo di sviluppo obbligatorio
Ogni task segue il ciclo definito in `dev-cycle.instructions.md`:
- **Dichiara** scope e file prima di agire
- **Esegui** un'operazione alla volta
- **Verifica** (rileggi) dopo ogni modifica
- **Segnala** incertezza — non assumere silenziosamente

*Template v1.3 - .NET 10 - Token-optimized for AI agents* - Last Update 2026-03-25 — claude-sonnet-4-6

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davraf-amuro)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/davraf-amuro)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
