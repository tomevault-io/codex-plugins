# GitHub Copilot Instructions für MBlock

## Projektstatus

Dieses Addon ist ab Version 4.6 im Wartungsmodus.

- Keine neuen großen Features einführen, wenn sie bestehende Module, Datenformate oder öffentliche APIs gefährden.
- Priorität haben Bugfixes, Stabilität, Editor-Kompatibilität und Rückwärtskompatibilität.
- Für neue Module ist MForm 9 mit Flex-Repeater die modernere Wahl, aber MBlock muss für bestehende Projekte zuverlässig weiterlaufen.

## Ziele bei Änderungen

- Bestehende MBlock-Module müssen ohne Content-Migration weiter funktionieren.
- Gespeicherte Datenstrukturen dürfen nicht stillschweigend geändert werden.
- Öffentliche APIs und Legacy-Einstiegspunkte müssen möglichst stabil bleiben.
- Änderungen an JavaScript-Reindexing, Widget-IDs, Popup-Callbacks und Sortierung sind besonders vorsichtig umzusetzen.

## Wichtige Pfade

### Dokumentation und Metadaten

- `README.md` – öffentliche Hauptdokumentation und Kompatibilitätshinweise
- `CHANGELOG.md` – Release-Historie, muss bei relevanten Änderungen gepflegt werden
- `API.md` – API-Dokumentation und Nutzungsbeispiele
- `docs/tutorial.md` – Backend-Dokumentation/Tutorial
- `package.yml` – Version, Anforderungen, Backend-Seiten, Default-Konfiguration

### PHP-Code

- `lib/MBlock/` – Kernklassen, Parser, Processor, Utils
- `lib/yform/` – YForm-Integration
- `boot.php` – Asset-Einbindung, Extension-Registrierung, Runtime-Verhalten im Backend
- `install.php`, `update.php` – Installations-/Update-Logik

### Assets

- `assets/mblock.js` – JS-Quellstand für Entwicklung
- `assets/mblock.min.js` – ausgelieferte Minified-Datei
- `assets/mblock.css` – Styles
- `assets/sortable.min.js` – lokaler Fallback für SortableJS
- `build.js` – Minify-/Build-Skript
- `package.json` – Build-Entry (`npm run build`)

### Tests und Hilfen

- `tests/` – projektinterne Test- und Notizdateien

## Rückwärtskompatibilität

### Zwingend erhalten

- Legacy-Nutzung von MBlock neben Namespace-Nutzung weiter mitdenken
- Ausgabeformate existierender Inhalte unverändert lassen
- Öffentliche Methoden wie `getDataArray()`, `getOnlineDataArray()`, `getOfflineDataArray()`, `filterByField()`, `sortByField()`, `groupByField()`, `limitItems()` nicht ohne sehr guten Grund ändern
- Bestehende Konfigurationswerte in `package.yml` und Addon-Config nicht brechen
- Bestehende JS-Events, DOM-Strukturen und Widget-Anbindungen nur minimal-invasiv ändern

### Besondere Vorsicht bei

- Reindexing von `REX_MEDIA_*`, `REX_LINK_*`, `REX_MEDIALIST_*`, `REX_LINKLIST_*`
- MForm-9-Kompatibilität in MBlock
- Copy/Paste-Logik
- Online/Offline-Toggle
- TinyMCE-, CKEditor- und markitup-Verhalten
- SortableJS-Laden über `bloecks` oder lokalen Fallback in `boot.php`

## Code-Qualität

### PHP

- REDAXO-Core-APIs bevorzugen: `rex_sql`, `rex_path`, `rex_file`, `rex_addon`, `rex_i18n`, `rex_view`
- Keine unnötigen Refactorings außerhalb des betroffenen Bereichs
- Keine Public-API-Änderungen ohne klare Notwendigkeit
- Bestehenden Stil im Addon respektieren, auch wenn ältere Bereiche noch historisch gewachsen sind

### JavaScript

- Änderungen möglichst im Quellfile `assets/mblock.js` machen
- Danach die Minified-Datei aktualisieren, wenn ausgelieferter Code verändert wurde
- Keine zufälligen Umbauten der DOM-Struktur ohne Prüfung der bestehenden Selektoren und Reindexing-Logik

## Pflichtprüfungen nach Änderungen

### Immer prüfen

1. Code-Qualität mit statischer Analyse:

```bash
docker exec -it coreweb bash -c "cd /var/www/html/public && php redaxo/bin/console rexstan:analyze redaxo/src/addons/mblock/"
```
(Pfade können je nach Setup variieren)

Sollten die Pfade oder Autoloading betroffen sein, vorher `composer dump-autoload --optimize` ausführen.

2. Falls JavaScript geändert wurde, Build aktualisieren:

```bash
cd public/redaxo/src/addons/mblock
npm run build
```

3. Bei relevanten Änderungen manuell mitdenken oder prüfen:

- Block hinzufügen, sortieren, löschen
- Copy/Paste
- Online/Offline
- MForm-Felder in MBlock
- Link-/Media-Widgets nach Reindexing

## Doku-Pflege ist Pflicht

Wenn sich Verhalten, API, Kompatibilität, Installation, Build oder bekannte Einschränkungen ändern, müssen auch die begleitenden Dateien geprüft und bei Bedarf angepasst werden:

- `CHANGELOG.md`
- `README.md`
- `API.md`
- `docs/tutorial.md`

Nicht nur Code ändern und Dokumentation vergessen.

## Changelog-Regeln

- Neue Einträge in `CHANGELOG.md` in einer release-tauglichen Formulierung schreiben
- Keine Chat-Notizen oder Entwicklungszwischenstände eintragen
- Rückwärtskompatibilität oder mögliche Risiken klar benennen, wenn relevant

## README-Regeln

- README als öffentliche Einstiegsdoku behandeln
- Features, Kompatibilität, Maintenance-Status und zentrale Nutzungspfade aktuell halten
- Wenn sich der empfohlene Einsatz mit MForm ändert, README mitziehen

## Arbeitsweise

- Zuerst den direkt betroffenen Codepfad identifizieren
- Klein und lokal ändern
- Danach sofort den engsten sinnvollen Check ausführen
- Keine breiten Umbauten starten, wenn ein lokaler Fix reicht

## Wenn du unsicher bist

- Lieber bestehende Patterns im Addon wiederverwenden als neue Abstraktionen einführen
- Bei API- oder Datenformat-Fragen immer zugunsten der Rückwärtskompatibilität entscheiden
- Bei Änderungen mit Nutzerwirkung immer auch Doku, README und Changelog mitprüfen

---
> Source: [FriendsOfREDAXO/mblock](https://github.com/FriendsOfREDAXO/mblock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-16 -->
