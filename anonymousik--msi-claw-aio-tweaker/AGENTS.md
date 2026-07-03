# 🧬 Copilot Instructions: MSI Claw Optimizer Professional Framework

Jesteś agentem wspomagającym rozwój frameworka optymalizacyjnego dla MSI Claw (Intel Core Ultra). Twoim zadaniem jest utrzymanie standardów "Professional Edition" v4.0, z naciskiem na bezpieczeństwo sprzętowe i integralność systemu operacyjnego.

## 1. 🧭 Architektura i Kontekst Rozwiązania
- **Typ projektu:** Zaawansowany Framework PowerShell (v5.1+).
- **Logika:** Skrypt jest modularny, mimo fizycznego podziału na pliki (Part 1, 2, 3), tworzy jedną spójną sesję logiczną.
- **Główne Komponenty:**
  - `Global Configuration`: Obiekt `$Script:Config` sterujący zachowaniem (AutoBackup, LogLevel).
  - `Safety Layer`: System `New-ConfigurationBackup` i `Restore-ConfigurationBackup` (Rejestr + Pliki).
  - `UI/UX`: System nagłówków `Write-Header` i logowania `Write-InfoLog`, `Write-CriticalLog`.

## 2. 🏗️ Protokoły Rozwoju i Walidacji (STRICT)

**Dyrektywa: Nigdy nie proponuj zmian w rejestrze lub usługach bez uwzględnienia mechanizmu wycofania (Rollback).**

### 🛠 Standardy Kodowania (Clean PowerShell):
1. **Obsługa Błędów:** Każda nowa funkcja musi posiadać blok `try-catch` z logowaniem do `Write-CriticalLog -Exception $_`.
2. **Interakcja:** Używaj funkcji `Confirm-Action -Message "..."` przed każdą zmianą, która wymaga restartu lub modyfikacji plików systemowych.
3. **Typowanie:** Parametry funkcji muszą posiadać zdefiniowane typy (np. `[string]$BackupId`).

### 🚀 Komendy Walidacyjne:
- **Analiza Statyczna:** `Invoke-ScriptAnalyzer -Path .` (wymagana zgodność z regułami PSScriptAnalyzer).
- **Test Składni:** `powershell -Command "node -c MSI_Claw_Optimizer_v4_FINAL_CONSOLIDATED.ps1"`.

## 3. 🗺️ Mapa Projektu i Kluczowe Funkcje
- `Start-MSIClawOptimizer`: Główny punkt wejścia i pętla menu.
- `Optimize-WindowsForGaming`: Kontener dla tweaków OS (wyłączanie telemetry, optymalizacja usług).
- `Start-InteractiveTroubleshooting`: Moduł diagnostyki problemów z baterią i hibernacją.
- `New-ConfigurationBackup`: Tworzy punkt przywracania systemu i eksportuje klucze rejestru przed modyfikacją.

## 4. 🛡️ Wytyczne Bezpieczeństwa (Hardening)
- **Zasada Minimalnych Uprawnień:** Skrypt wymaga `#Requires -RunAsAdministrator`.
- **Intel Arc Context:** Przy optymalizacji GPU skup się na ReBar i profilach zasilania specyficznych dla Intel Meteor Lake.
- **Bateria:** MSI Claw ma specyficzne stany uśpienia. Zawsze preferuj konfigurację Hibernacji nad Modern Standby (S0) w tym urządzeniu.

## 5. 🚫 Zakazy i Ograniczenia
- **NIE** używaj aliasów (np. zamiast `gci` używaj `Get-ChildItem`).
- **NIE** modyfikuj usług MSI (np. `MSI Center M`), chyba że jest to jawna prośba o naprawę konfliktu.
- **NIE** usuwaj sekcji logowania – każdy `ChangesApplied` musi zostać odnotowany w raporcie końcowym.

---
> Source: [anonymousik/msi-claw-aio-tweaker](https://github.com/anonymousik/msi-claw-aio-tweaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-03 -->
