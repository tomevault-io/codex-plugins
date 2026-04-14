# Instructions pour GitHub Copilot (Projet CyberGhost-GUI)

## 1. Contexte du Projet
Tu assistes un développeur dans la création de **CyberGhost-GUI**, une interface graphique Linux (Ubuntu) pour le client VPN CyberGhost. 
Ce projet est un **Wrapper**. L'application ne gère aucun protocole réseau directement. Elle se contente d'exécuter des commandes shell du CLI officiel `cyberghostvpn` en arrière-plan et d'en parser les résultats pour mettre à jour l'interface graphique.

## 2. Stack Technique
- **Langage :** Python 3.10+
- **Interface Graphique :** `customtkinter` (N'utilise **jamais** `tkinter` standard ou `PyQt`).
- **Communication Système :** Module standard `subprocess`.
- **Concurrence :** Module standard `threading`.

## 3. Règles d'Architecture Strictes
- **Séparation des responsabilités :** Le code UI (`src/ui/`) ne doit jamais faire d'appels directs à `subprocess`. Il doit passer par le contrôleur métier (`src/backend/vpn_controller.py`).
- **Arborescence à respecter :**
  - `src/ui/` : Fenêtres, frames, composants CustomTkinter.
  - `src/backend/cli_wrapper.py` : Fonctions pures exécutant les commandes `subprocess`.
  - `src/backend/vpn_controller.py` : Logique de parsing, gestion d'état et multithreading.

## 4. Règles de Code - Backend & CLI
- **Appels Sous-processus :** Utilise toujours `subprocess.run()` avec `capture_output=True` et `text=True`. Gère toujours les exceptions `subprocess.CalledProcessError`.
- **Parsing et Nettoyage :** Le CLI de CyberGhost renvoie souvent des codes de couleur ANSI. Prévois toujours de nettoyer le `stdout` via Regex (`re.sub(r'\x1b\[[0-9;]*m', '', text)`) avant de l'analyser.
- **Sécurité :** Utilise `shlex` si tu dois construire des commandes complexes.
- **Tests :** Écris des tests unitaires pour toutes les fonctions critiques, en particulier celles qui interagissent avec le CLI.
- **Élévation de privilèges :** Si une commande nécessite les droits root, propose l'utilisation de `pkexec` plutôt que `sudo` pour déclencher une popup GUI native sous Linux.

## 5. Règles de Code - Interface (CustomTkinter) & Concurrence
- **Ne jamais bloquer le MainThread :** Les commandes CLI réseau (comme `cyberghostvpn --connect`) peuvent prendre plusieurs secondes. Elles doivent **obligatoirement** être exécutées dans un thread séparé (`threading.Thread`) pour ne pas faire geler l'interface (`mainloop`).
- **Mise à jour de l'UI :** Depuis un thread secondaire, utilise toujours les méthodes thread-safe de CustomTkinter (comme `.after()`) pour mettre à jour les éléments graphiques (boutons, labels).
- **Design :** Préfère les couleurs et les styles dynamiques (Light/Dark mode) natifs à CustomTkinter.

## 6. Conventions de Codage
- **Langue :** Rédige les noms de variables, fonctions, classes en **Anglais** et les commentaires/docstrings en **Français**. Les textes affichés à l'utilisateur dans l'UI doivent être en **Français**.
- **Typage :** Utilise systématiquement le Type Hinting de Python (ex: `def get_status() -> dict:`).
- **Gestion des erreurs :** Préfère retourner des dictionnaires ou des objets de statut (ex: `{"status": "error", "message": "..."}`) plutôt que de crasher l'application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MithrandirEa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
