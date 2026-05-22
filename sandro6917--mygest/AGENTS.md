# MyGest Copilot Instructions

## Missione del progetto
MyGest è un gestionale documentale e operativo con backend Django/PostgreSQL e frontend React/Vite.
Il dominio principale comprende anagrafiche, documenti, fascicoli, pratiche, scadenze, protocollo, archivio fisico, comunicazioni e AI classifier.

## Principi obbligatori

1. Non inventare strutture, modelli o flussi non presenti nel codice o nella documentazione citata.
2. Mantieni naming coerente con il dominio esistente: `Documento`, `Fascicolo`, `Pratica`, `Scadenza`, `MovimentoProtocollo`, `UnitaFisica`, `Anagrafica`, `Cliente`, `TitolarioVoce`.
3. Se il codice non basta, esplicita sempre `ASSUNZIONI` e `DATI MANCANTI`.
4. Preferisci modifiche minime, locali e reversibili.
5. Prima di introdurre nuovi concetti, cerca pattern già presenti nella codebase.
6. Non rompere RBAC, isolamento dati per cliente, naming dei codici o logiche di storage NAS.
7. Quando tocchi API, considera sempre impatto su serializer, viewset, permessi, filtri e client React.
8. Quando tocchi modelli dati, considera migrazioni, dati esistenti, retrocompatibilità e test.
9. Quando tocchi upload/documenti, considera path, naming file, storage NAS, protocollazione e auditabilità.
10. Distingui sempre chiaramente tra:
   - AS-IS: ciò che il codice fa oggi
   - TO-BE: proposta evolutiva

## Stack di riferimento

- Backend: Django, Django REST Framework, JWT, PostgreSQL, Redis
- Frontend: React, TypeScript, Vite, Zustand, MUI
- Storage documentale: NAS path /mnt/archivio
- Altri componenti: GraphQL legacy, management commands, AI classifier

## Moduli principali

- `anagrafiche`
- `documenti`
- `fascicoli`
- `pratiche`
- `scadenze`
- `protocollo`
- `archivio_fisico`
- `comunicazioni`
- `whatsapp`
- `titolario`
- `ai_classifier`
- `stampe`
- `core`
- `api/v1`
- `frontend/src`

## Come ragionare sulle modifiche

### Backend
- verifica relazioni tra modelli e vincoli impliciti
- verifica permessi RBAC e isolamento per cliente
- verifica serializer e viewset correlati
- verifica management command o job coinvolti

### Frontend
- cerca pagina, route, hook, service API, tipi TS e componenti coinvolti
- mantieni pattern UI coerenti
- evita duplicazione di logica già nei servizi API o negli hook

### Testing
- aggiungi o aggiorna test mirati
- non proporre test generici se puoi individuare gli edge case reali
- segnala sempre cosa è stato verificato e cosa no

## Output attesi
Quando produci analisi o proposte, usa sezioni chiare:
- Contesto
- Stato AS-IS
- Gap / Problema
- Proposta
- File / moduli impattati
- Rischi
- Test / Verifiche
- Assunzioni

## Da evitare
- refactor enormi non richiesti
- rinominare concetti di dominio senza motivo
- introdurre pattern astratti non necessari
- scrivere documentazione che confonde attuale e desiderato

---
> Source: [sandro6917/mygest](https://github.com/sandro6917/mygest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
