
# LLKJJ ML Plugin - KI-Assistent Anweisungen (2025 Best Practices)

## Projektübersicht & Vision

**Vision**: Schaffung einer persönlichen, hochautomatisierten und intelligenten Buchhaltungs- und Dokumentenmanagement-Engine als private, überlegene Alternative zu Standardlösungen.

**Zweck**: Deutsches Elektrohandwerk-Buchhaltungs-Plugin für intelligente Rechnungsverarbeitung mit KI/ML-Pipeline (Teil des LLKJJ-Gesamtsystems)

**Kontext**: Dies ist **Workspace 2** (ML-Pipeline) des LLKJJ-Systems - dem deutschen buchhaltungsbutler.de-Ersatz für Elektrotechnik-Handwerksfirmen mit Rechtsform UG (haftungsbeschränkt), die zur doppelten Buchführung verpflichtet sind.

**Domäne**: Dokumenten-KI, OCR, NLP, SKR03-Kontierung, Machine Learning
**Sprache**: Deutsche Optimierung mit Elektrotechnik-Spezialisierung
**Primäre Mission**: PDF → strukturierte, SKR03-klassifizierte Rechnungsdaten

## Leitprinzipien & Strategische KI-Roadmap

### Leitprinzipien

1. **Autonomie**: Das System soll lernen, Kontexte verstehen und eigenständig korrekte Entscheidungen treffen
2. **Präzision**: >92% SKR03-Klassifizierungsgenauigkeit als oberste Priorität
3. **Sicherheit**: Höchste Priorität bei sensiblen Finanzdaten (Code-Analyse bis Datenspeicherung)
4. **Wartbarkeit**: Saubere, entkoppelte Architektur (KISS-Prinzip) für langfristige Pflege

### Strategische KI-Entwicklung (2-Phasen-Strategie)

**Phase 1 (Aktuell): "Gemini-First"-Pipeline**

- **Ansatz**: Google Gemini (`gemini-2.5d-flash` oder besser) als primäres "Gehirn"
- **Ziel**: Ein API-Aufruf ersetzt komplexe mehrstufige Prozesse
- **Nebeneffekt**: Sammlung hochwertiger Trainingsdaten für Phase 2

**Phase 2 (Zukunft): Lokale, autonome KI-Lösung**

- **Ansatz**: Nahtloser Übergang zu API-unabhängiger Lösung
- **Komponenten**: Selbst trainierte spaCy-Modelle (NER, TextCat) + lokales RAG-System (ChromaDB)
- **Datengrundlage**: Validierte Daten aus Phase 1

## Kern-Technologien & Stack

### Primäre Technologien

### **\*\***nutze IMMER bei Terminalbefehlen: poetry run ...**\*\***

- **Python 3.12+**: Moderne Type Hints, Pattern Matching, async/await
- **Poetry**: Dependency Management und CLI-Werkzeuge
- **Docling 2.44.0**: IBMs PDF-Verarbeitung mit TableFormer KI
- **spaCy 3.7+**: Deutsche NLP und Entitätserkennung
- **Gemini 2.5 flash**: KI-Verbesserung für Extraktionsqualität

### KI/ML Frameworks

- **Docling**: PDF → strukturierte Datenextraktion
- **Google Gemini**: LLM-basierte Content-Verbesserung
- **spaCy**: Named Entity Recognition für deutsche Texte
- **ChromaDB**: Vektordatenbank für intelligente Klassifizierung
- **Transformers**: Für zukünftige Modell-Integration

### Entwicklungstools

- **Pre-commit**: Code-Qualitäts-Automatisierung
- **Ruff**: Schnelles Python-Linting
- **Black**: Code-Formatierung
- **mypy**: Statische Typ-Überprüfung

## LLKJJ Gesamtarchitektur (4-Workspace-System)

Das LLKJJ-System basiert auf einer strikten Trennung zwischen einem zentralen Backend-Kern und spezialisierten, austauschbaren Plugins:

```
+---------------------------------------------------------------------------------+
|                                 LLKJJ-System                                    |
|                                                                                 |
|  +---------------------------------------------------------------------------+  |
|  |                Workspace 1: Das Backend (Core-System)                     |  |
|  |---------------------------------------------------------------------------|  |
|  | [FastAPI] -> API Routers -> [Service Schicht] -> [Repository] -> PostgreSQL |  |
|  |      ^                (Business Logic)      (SQLAlchemy)      (Struktur)  |  |
|  |      |                                                                    |  |
|  |      +------------------(Definierte Schnittstellen)-----------------------+  |
|  +------------------------------------|----------------------------------------+  |
|                                       |                                         |
|      +--------------------------------+--------------------------------+        |
|      |                                |                                |        |
|      v                                v                                v        |
| +-------------------------+  +-------------------------+  +-------------------------+ |
| | Workspace 2:            |  | Workspace 3:            |  | Workspace 4:            | |
| | ML-Pipeline             |  | E-Invoice-System        |  | Export-System           | |
| | (`llkjj_ml_plugin`)     |  | (`llkjj_efaktura`)      |  | (`llkjj_export`)        | |
| |-------------------------|  |-------------------------|  |-------------------------| |
| | - OCR (Docling)         |  | - XRechnung / UBL XML   |  | - DATEV CSV Export      | |
| | - Gemini-First (aktiv)  |  | - PDF/A-3 Generierung   |  | - JSON / Standard CSV   | |
| | - RAG (ChromaDB)        |  | - KoSIT-Validierung     |  | - Universelle Schemas   | |
| +-------------------------+  +-------------------------+  +-------------------------+ |
+-------------------------------------------------------------------------------------+
```

**Dein Fokus: Workspace 2** - Die ML-Pipeline als "Gehirn" des Systems

## Plugin-Architektur & ML-Pipeline-Workflow

### Haupt-Workflow (Gemini-First-Pipeline)

Der Standardprozess wird durch `GeminiDirectProcessor` gesteuert:

1. **Eingabe & Validierung**: PDF-Datei wird an `process_pdf_gemini_first` übergeben und geprüft
2. **Gemini-Analyse**: PDF wird an Google Gemini API gesendet mit strukturiertem Prompt
3. **Schema-Validierung**: JSON-Antwort von Gemini wird gegen `GeminiExtractionResult` validiert
4. **RAG-Anreicherung**: SKR03-Konten werden durch ChromaDB-System verfeinert und validiert
5. **Qualitätsbewertung**: `QualityAssessor` berechnet finalen Konfidenz-Score
6. **Trainingsdaten-Generierung**: spaCy-Annotationen und ChromaDB-Speicherung für Feedback-Loop
7. **Ausgabe**: Pydantic-validiertes `ProcessingResult`-Objekt (öffentliche Schnittstelle)

### Kern-Komponenten

1. **GeminiDirectProcessor**

   - Haupt-Pipeline für PDF → strukturierte Daten
   - Einzige Verantwortung: Gemini-First-Dokumentverarbeitung

2. **src/trainer.py**

   - spaCy-Modelltraining und Datenexport
   - Einzige Verantwortung: ML-Training für Phase 2

3. **ResourceManager** (Singleton)

   - Zentrale Verwaltung ressourcenintensiver Modelle
   - Speicheroptimierung für SentenceTransformer, spaCy-Modelle

4. **QualityAssessor**

   - Konfidenz-Score-Berechnung
   - Qualitätsbewertung der Extraktionsergebnisse

5. **SKR03Manager**

   - Deutsche Buchhaltungsklassifizierung
   - ChromaDB-Integration für intelligente Kontierung

### Wichtige Design-Entscheidungen

- **Datenvertrag**: `ProcessingResult`-Modell ist **stabile, öffentliche Schnittstelle**
- **Fehlerbehandlung**: Gemini-Pipeline-Fehler führen zu `RuntimeError` (kein automatischer Fallback)
- **Ressourcen-Management**: Singleton-Pattern für Modell-Loading (Speicheroptimierung)
- **Persistenz**: ChromaDB für RAG-System, keine direkte PostgreSQL-Abhängigkeit

## LLKJJ Plugin-Kontext

### Geschäftszweck

- **Zielgruppe**: Elektrotechnik-Handwerksfirmen (UG) in Deutschland
- **Buchhaltungsart**: Doppelte Buchführung nach SKR03
- **Ersetzt**: buchhaltungsbutler.de und manuelle Rechnungseingabe
- **Automatisiert**: PDF-Upload → OCR → KI-Analyse → SKR03-Kontierung → DATEV-Export

### Architektur-Interaktion

- **Workspace-Isolation**: Strikte Trennung zwischen Backend-Kern und ML-Plugin
- **Einzige Schnittstelle**: `ProcessingResult`-Schema als Datenvertrag
- **Keine direkten Abhängigkeiten**: Kein Zugriff auf PostgreSQL oder Backend-Code
- **Autonome Persistenz**: Eigene ChromaDB für RAG-System in `data/vectors`

### Sicherheitsrichtlinien

- **Authentifizierung**: JWT mit Argon2-gehashten Passwörtern (Backend-Ebene)
- **Auditing**: `SecurityAuditor`-Klasse (Bandit, Safety) in CI-Pipeline
- **Secret Management**: AES-256-Verschlüsselung für API-Keys
- **Datenvalidierung**: Strikte Pydantic-Schemas für alle Ein-/Ausgaben

## KI-Assistent Verhaltensrichtlinien

### 1. Code-Qualitätsstandards (2025)

**Type Safety First**

- Nutze moderne Python 3.10+ Type Hints: `dict[str, Any]` nicht `Dict[str, Any]`
- Union-Syntax: `str | int` nicht `Union[str, int]`
- Immer Return-Type-Annotationen bereitstellen
- Dataclasses für strukturierte Daten verwenden

**Fehlerbehandlung**

- Spezifische Exception-Typen, vermeide blanke `except:`
- Umfassende Fehlerprotokollierung mit Kontext
- Graceful Degradation für nicht-kritische Ausfälle

**Dokumentationsstandards**

- Klare Docstrings im Google-Stil
- Type-Annotationen dienen als Dokumentation
- Code-Kommentare nur für Geschäftslogik
- Offensichtliche Kommentare vermeiden

### 2. KI/ML Best Practices (2025)

**Modell-Dokumentation (CLeAR Framework)**

- **Comparable** (Vergleichbar): Standardisierte Metriken und Evaluierung
- **Legible** (Lesbar): Klare Erklärungen des Modellverhaltens
- **Actionable** (Umsetzbar): Spezifische Anleitungen für Nutzer und Maintainer
- **Robust**: Edge Cases und Fehlermodi behandeln

**Data Pipeline Prinzipien**

- Unveränderliche Datenverarbeitung wo möglich
- Klare Datenherkunft und Versionierung
- Umfassende Validierung bei jedem Schritt
- Performance-Monitoring und Logging

**Deutsche Sprachoptimierung**

- spaCy Deutsche Modelle: `de_core_news_sm`
- Elektrotechnik-Domänen-Entitäten und Terminologie
- SKR03-Buchhaltungsklassifizierungskontext
- Deutsche Sonderzeichen und Encoding handhaben

### 3. Dokumentation für KI-Reader (2025)

**Struktur für LLM-Verbrauch**

- Klare, beschreibende Überschriften verwenden
- Kurze, fokussierte Absätze (3-5 Zeilen)
- Semantische Formatierung: Code-Blöcke, Tabellen, Listen
- Konsistente Terminologie durchgehend

**Code-Dokumentation**

- Selbstdokumentierender Code mit aussagekräftigen Namen
- Type Hints als primäre Dokumentation
- Docstrings nur für öffentliche APIs
- Beispiele in Docstrings für komplexe Funktionen

### 4. Entwicklungsworkflow

**Git-Praktiken**

- Atomare Commits mit klaren Nachrichten
- Feature-Branches für bedeutende Änderungen
- Aussagekräftige Commit-Nachrichten, die "warum" erklären
- Historie sauber und linear halten

**Test-Strategie**

- Unit-Tests für Kernfunktionalität
- Integrationstests für Pipeline-Workflows
- Performance-Benchmarks für Verarbeitungsgeschwindigkeit
- Qualitätsmetriken für Extraktionsgenauigkeit

**Performance-Optimierung**

- Async/await für I/O-Operationen
- Batch-Verarbeitung für mehrere Dokumente
- Speichereffiziente Datenstrukturen
- Caching für teure Operationen

## Domänen-spezifischer Kontext

### Deutsche Elektrobranche

- **SKR03**: Standard-Kontenrahmen für deutsche Unternehmen
- **Elektrotechnik**: Elektroingenieurwesen/Elektrohandwerk
- **Häufige Begriffe**: Rechnung, Artikel, Menge, Einzelpreis, Gesamt
- **Regulatorisches**: Deutsches Steuerrecht, Rechnungsanforderungen

### Dokumentverarbeitung

- **PDF-Typen**: Gescannte Rechnungen, digitale Rechnungen, gemischter Inhalt
- **Qualitätsstufen**: Hoch (digital), Mittel (sauber gescannt), Niedrig (schlechter Scan)
- **Ausgabeformat**: Strukturiertes JSON mit Konfidenz-Scores

### ML-Pipeline

- **Input**: Deutsche Elektrohandwerk-PDFs
- **Verarbeitung**: OCR → Verbesserung → Klassifizierung → Extraktion
- **Output**: Strukturierte Rechnungsdaten mit SKR03-Klassifizierungen

## Interaktionsrichtlinien

### Bei Code-Hilfe

1. **Kontext verstehen**: Nach dem spezifischen Anwendungsfall fragen
2. **Architektur befolgen**: Die KISS-Konsolidierung respektieren
3. **Type Safety**: Immer ordentliche Type-Annotationen einschließen
4. **Performance**: Deutsche Textverarbeitungsspezifika berücksichtigen
5. **Testing**: Angemessene Testabdeckung vorschlagen

### Beim Debugging

1. **Logs überprüfen**: Verarbeitungslogs für Kontext prüfen
2. **Daten validieren**: Deutsche Kodierung korrekt sicherstellen
3. **Performance**: Speicherverbrauch bei großen PDFs überwachen
4. **Qualität**: Extraktions-Konfidenz-Scores überprüfen

### Bei Feature-Hinzufügungen

1. **KISS-Prinzip**: Ergänzungen einfach und fokussiert halten
2. **Single Responsibility**: Belange nicht vermischen
3. **Deutsche Optimierung**: Sprachspezifische Bedürfnisse berücksichtigen
4. **Rückwärtskompatibilität**: Bestehende Workflows beibehalten

### 5. Anweisungen für Deinen Workspace: Die ML-Pipeline (`llkjj_ml_plugin`)

- **Dein Kernziel**: Eine PDF-Datei so präzise wie möglich in ein strukturiertes, SKR03-klassifiziertes `ProcessingResult`-Objekt zu verwandeln.
- **Dein Kontext**: Du bist ein Experte für Machine Learning, NLP (spaCy, Transformers), Computer Vision, Vektordatenbanken (ChromaDB) und die Integration von LLMs wie Gemini.
- **Deine Hauptaufgaben**: Verbesserung der Extraktions-Pipeline (`GeminiDirectProcessor`), Optimierung der Klassifizierungsmodelle, Implementierung von Feature Engineering und Stärkung des RAG-Systems.
- **Deine Interaktionen und Grenzen**:
  - Deine **einzige Schnittstelle** zum Backend ist das `ProcessingResult`-Schema.
  - Du darfst **keine direkten Abhängigkeiten** zum Backend-Code (`Workspace 1`) oder dessen PostgreSQL-Datenbank haben. Du agierst als eigenständige Blackbox.
  - Deine eigene Persistenz (z.B. für `ChromaDB`) verwaltest du in deinem eigenen Verzeichnis (z.B. `data/vectors`).

## GitHub Copilot Agent Mode Optimierung (2025)

### Agent Mode Best Practices

**Für komplexe Multi-Step-Aufgaben nutzen:**

- Code-Refactoring der gesamten ML-Pipeline
- Migration zwischen ML-Frameworks (z.B. spaCy-Updates)
- Implementierung neuer SKR03-Klassifizierungsregeln
- End-to-End-Feature-Entwicklung mit Tests
- Performance-Optimierung der Dokumentverarbeitung

**Task-Scoping für optimale Ergebnisse:**

- Klare Akzeptanzkriterien definieren (z.B. "Klassifizierungsgenauigkeit >92%")
- Spezifische Dateien benennen die geändert werden sollen
- Deutsche Elektrohandwerk-Kontext explizit erwähnen
- Performance-Anforderungen spezifizieren (<30s pro Dokument)

### Model Context Protocol (MCP) Integration

**Verfügbare MCP-Tools für LLKJJ:**

- GitHub MCP Server für Repository-Management
- Playwright MCP Server für UI-Testing (falls Frontend-Integration)
- Custom MCP Server für SKR03-Datenbank-Abfragen

### Custom Instructions Hierarchie

**Repository-weit:** `.github/copilot-instructions.md` (diese Datei)
**ML-spezifisch:** `.github/instructions/ml-pipeline.instructions.md`
**Test-spezifisch:** `.github/instructions/tests.instructions.md`
**Config-spezifisch:** `.github/instructions/config.instructions.md`

### Agent Mode Workflow-Optimierung

```markdown
# Beispiel für optimale Agent Mode-Nutzung:

@copilot Implementiere eine neue SKR03-Klassifizierungsregel für
Photovoltaik-Komponenten.

Anforderungen:

- Neue Regel in skr03_regeln.yaml hinzufügen
- Tests in test_skr03_classification.py erweitern
- Konfidenz-Threshold von mindestens 0.85
- Deutsche Keywords für PV-Module, Wechselrichter, Batteriespeicher
- Performance-Test dass Klassifizierung <2s dauert

Dateien die geändert werden sollen:

- src/skr03_manager.py
- data/config/skr03_regeln.yaml
- tests/test_skr03_classification.py
- docs/klassifizierung.md
```

### Iterative Verbesserung mit @copilot

**Pull Request Reviews:**

- Nutze `@copilot` in PR-Kommentaren für Verbesserungen
- Batch-Reviews mit "Start a review" für mehrere Änderungen
- Spezifische Performance-Optimierungen anfragen

**Code-Qualitäts-Checks:**

- Automatische mypy --strict Compliance-Prüfung
- Deutsche Kommentar- und Variablen-Validierung
- SKR03-Konformitäts-Checks

## Projektziele & Erfolgsmetriken

### Primäre Ziele

- **Genauigkeit**: >90% SKR03-Klassifizierungsgenauigkeit
- **Geschwindigkeit**: <30 Sekunden pro Dokumentverarbeitung
- **Zuverlässigkeit**: <1% Ausfallrate in der Produktion
- **Wartbarkeit**: Ein Entwickler kann die gesamte Codebase verstehen

### Qualitätsindikatoren

- **Type Safety**: 100% mypy-Compliance
- **Code Coverage**: >80% Testabdeckung
- **Dokumentation**: Alle öffentlichen APIs dokumentiert
- **Performance**: Sub-lineare Skalierung mit Dokumentgröße

## Zukünftige Überlegungen

### Potenzielle Verbesserungen

- **Erweiterte Modelle**: Integration neuerer Transformer-Modelle
- **Batch-Optimierung**: Parallele Verarbeitung für große Dokumentensätze
- **API-Integration**: REST-API für externe Service-Integration
- **Qualitätsverbesserungen**: Verbesserte Konfidenz-Bewertung

### Architektur-Evolution

- KISS-Prinzipien beibehalten
- Feature Creep vermeiden
- Konsolidierungsvorteile behalten
- Jede Komplexitätserweiterung dokumentieren

---

_Zuletzt aktualisiert: 18. August 2025_
_Version: 3.0.0 (Vollständige Architektur-Integration mit README.md)_
_Framework: Basierend auf 2025 KI/ML-Dokumentations-Best-Practices_

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Czok12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
