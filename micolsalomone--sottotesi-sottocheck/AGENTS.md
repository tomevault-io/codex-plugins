# Copilot Instructions — Sottotesi

## Prima di qualsiasi modifica
- Leggi `ARCHITECTURE.md` per la struttura generale e le route
- Leggi `docs/views/[vista].md` per la vista su cui stai lavorando

## Viste del progetto
| Vista | Path | File di riferimento |
|-------|------|---------------------|
| Admin | `/` | `docs/views/admin.md` |
| Coach | `/coach-view` | `docs/views/coach.md` |
| Student | `/student-view` | `docs/views/student.md` |

## Regole sempre valide
- Usare sempre alias `@` per gli import, mai path relativi lunghi
- Variabili CSS solo da `src/styles/theme.css`, nessun valore hardcoded
- Non condividere Layout, Header, Sidebar tra viste diverse
- Non aggiungere route coach/student alla sidebar admin
- Non unire i Container.tsx delle viste diverse
- Drawers solo per azioni, mai come fonte di verità
- Nessuna feature non esplicitamente richiesta

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/micolsalomone)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/micolsalomone)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
