# Copilot Instructions — PIN Vault

## Project Overview
A **single-file HTML app** for secure local storage of PINs, passcodes, usernames,
notes, and TOTP seeds. Authentication uses **Windows Hello** (WebAuthn platform
authenticator) as a presence gate, with a **master password fallback**. Data is
encrypted with **AES-GCM** keyed from the master password and stored in a
**user-selected external JSON file**, enabling use across multiple computers via a
shared folder (OneDrive, network share, etc.).

## Files
| File | Purpose |
|---|---|
| `vault.html` | Entire app — all HTML, CSS, JS in one file. Never split. |
| `*.vault.json` | Encrypted vault data. User picks location via File System Access API. The only allowed second file. |

## Hard Constraints
- **One logic file only**: all HTML, CSS, and JS live in `vault.html`. No external `.js` or `.css` files.
- **No build tooling**: no npm, no bundler, no compilation step. Opens directly from `file://`.
- **Bootstrap from CDN**: `https://cdn.jsdelivr.net/npm/bootstrap@5/`
- **No server**: purely client-side. No `fetch` to any backend.
- **No `alert()` / `confirm()`**: use Bootstrap toasts or modals exclusively.
- **No plaintext secrets in memory longer than needed**: zero out buffers after use.

## Architecture (within `vault.html`)

```
vault.html
└── <script>  Logical sections (comment-delimited, not separate files):
    ├── AUTH     — WebAuthn registration & assertion (Windows Hello presence check)
    ├── CRYPTO   — AES-GCM encrypt/decrypt; PBKDF2 key derivation from master password
    ├── STORE    — File System Access API: open/save *.vault.json; localStorage for credentialId only
    ├── ENTRIES  — Entry schema, CRUD operations
    └── UI       — DOM rendering, Bootstrap modal/toast wiring, event handlers
```

## Key Design Decisions

### Auth vs. Encryption Key — Two Separate Concerns
- **Windows Hello** (`navigator.credentials.create/get`) = presence/identity gate only.
  It does NOT derive the encryption key. Even though the same Windows account (Microsoft
  account / Entra ID) is used on all machines, Windows Hello credentials are registered
  per-device (TPM-bound). Each machine requires its own `credentials.create()` call on
  first use — `credentialId` is stored in `localStorage` per device.
- **Master password** = sole source for key derivation (PBKDF2 → AES-GCM 256-bit key).
  This is the shared secret that works identically across all machines.
- **Fallback**: if Windows Hello is unavailable or fails, prompt for master password directly.
- Store only `credentialId` in `localStorage` (not sensitive, device-specific).

### Encryption
- Algorithm: **AES-GCM**, 256-bit key.
- Key derivation: `crypto.subtle.deriveKey` with **PBKDF2**, random salt stored in vault file.
- Each entry encrypted individually with its own random 12-byte IV.
- Vault JSON shape:
  ```json
  {
    "version": 1,
    "salt": "<base64>",
    "entries": [
      { "id": "<uuid>", "label": "...", "type": "pin|password|username|note|totp", "iv": "<base64>", "cipher": "<base64>" }
    ]
  }
  ```

### Cross-Device Data Storage
Both machines open and write the **same** `*.vault.json` from a shared drive. This works because:
- The master password produces an identical AES-GCM key on every machine (PBKDF2 + the salt stored in the file).
- Windows Hello is only a local presence gate and plays no role in decryption.

- **First run** (no vault loaded): automatically call `showSaveFilePicker` to create a new `*.vault.json` — do not wait for an explicit button press.
- On open (existing vault): `window.showOpenFilePicker({ types: [{ accept: { 'application/json': ['.json'] } }] })`
- On save: write back to the same `FileSystemFileHandle`.
- The vault file lives on a shared drive managed by the user (OneDrive, Dropbox, NAS, etc.).
- **Conflict detection**: before every save, call `(await fileHandle.getFile()).lastModified` and compare against the timestamp recorded at open time. If it differs, another machine has written the file since it was loaded.
- **Conflict resolution**: on conflict, show a Bootstrap modal with three choices:
  - **Reload** — discard local unsaved changes, re-decrypt the file as updated by the other machine.
  - **Overwrite** — force-save local state, discarding the other machine's changes.
  - **Cancel** — do nothing, let the user decide manually.
- Update the recorded `lastModified` timestamp after every successful save.

### Entry Types
| Type | Encrypted fields |
|---|---|
| `pin` | numeric code |
| `password` | passphrase/passcode string |
| `username` | username + optional associated password |
| `note` | freeform text |
| `totp` | TOTP secret seed — stored and revealed on demand; no code generation. User copies the seed into their authenticator app (e.g. Microsoft Authenticator). |

### Bootstrap Usage
- Modal for add/edit entry forms and master-password prompt.
- `list-group` for vault entries list.
- `badge` for entry type tags.
- Toasts for success/error feedback.
- Prefer Bootstrap utility classes over custom CSS.

### Session & Locking
- After unlocking, the vault stays open for the session.
- **Auto-lock after 5 minutes of inactivity** — reset the timer on any user interaction (`mousemove`, `keydown`, `click`). On lock: clear the derived key from memory and return to the auth screen.
- The `FileSystemFileHandle` can be retained across a lock/unlock cycle (no need to re-pick the file).

## Developer Workflow
No build step. Edit `vault.html` directly and reload in browser.
- **Windows Hello does NOT work from `file://`** in any browser (Chromium blocks WebAuthn on file origins). The app detects this automatically and falls back to master-password-only mode.
- To test Windows Hello, serve the file over HTTPS or `localhost` (e.g. `npx serve .` then open `http://localhost:3000/vault.html`) — but production use is always `file://` with password fallback.
- **Best browser for file:// use**: Microsoft Edge or Chrome.
- **Does NOT work**: Firefox (no File System Access API).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whizzsfe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
