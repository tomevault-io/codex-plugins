# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FTP Sync CI is a single-file Python tool that watches local directories for file changes and automatically syncs them to remote servers via FTP, FTPS, or SFTP. It is designed for editors (like Zed) that lack built-in remote sync.

## Running the Tool

```bash
# Install dependencies
pip install -r requirements.txt    # watchdog, paramiko

# Run with default config.json
python file_sync.py

# Run with custom config
python file_sync.py -c myconfig.json

# Generate example config file
python file_sync.py --create-config

# Test connection without syncing
python file_sync.py --test-connection

# Download all remote files to local directory
python file_sync.py --download

# Via wrapper scripts (auto-install deps)
./sync.sh          # Unix/Linux
sync.bat           # Windows
```

There are no tests, linting, or build steps configured.

## Architecture

Everything lives in `file_sync.py` (~620 lines). The classes form a pipeline:

1. **Config** — Loads/creates JSON config files. Static utility methods only.
2. **SFTPUploader / FTPUploader** — Protocol-specific connection handlers with a shared interface (`connect`, `upload`, `download`, `delete`, `walk_remote`, `close`). SFTPUploader uses paramiko; FTPUploader uses stdlib ftplib.
3. **FileSyncHandler** (extends watchdog `FileSystemEventHandler`) — Receives filesystem events, applies ignore patterns, and queues uploads with a 0.5s debounce to batch rapid changes.
4. **FileSyncTool** — Orchestrator: loads config, creates the correct uploader based on `protocol` field, wires up the watchdog Observer, and runs the main loop (`sleep 0.1s` → `process_pending_uploads`).

### Key flows

- **File change detected** → `on_created`/`on_modified` → `schedule_upload` (checks `auto_upload` config, adds to `pending_uploads` dict with timestamp) → main loop calls `process_pending_uploads` after 0.5s delay → `uploader.upload()`
- **File deleted** → `on_deleted` → checks `auto_delete` config → immediate `uploader.delete()` (no debounce)
- **File download** → `--download` CLI flag → `download_all_files()` → `uploader.walk_remote()` recursively lists remote files → `uploader.download()` for each file
- **Loop prevention** — Downloaded files are tracked in `_downloaded_paths` with timestamps. `schedule_upload` skips any file downloaded within the last 10 seconds, preventing download→upload loops.
- **Path conversion** — `get_remote_path()` computes relative path from `local_path`, joins with `remote_path`, normalizes to forward slashes
- **Ignore matching** — Wildcard patterns (`*.pyc`) match via suffix; plain names (`.git`, `node_modules`) match against individual path components

### Configuration

JSON config file (default `config.json`). See `config.example.json` for template. Key fields:

| Field | Description | Default |
|-------|-------------|---------|
| `protocol` | `sftp`, `ftp`, or `ftps` | `sftp` |
| `host` | Remote server hostname | — |
| `port` | Remote server port | 22 (sftp) / 21 (ftp) |
| `username` | Login username | — |
| `password` / `key_file` | Auth credentials (key_file for SFTP only) | — |
| `local_path` | Local directory to watch | — |
| `remote_path` | Remote directory to sync to | — |
| `ignore_patterns` | List of patterns to exclude | `[]` |
| `auto_create_dirs` | Create remote dirs as needed | `true` |
| `passive_mode` | Use passive mode for FTP | `true` |
| `auto_upload` | Auto-upload files on local changes | `true` |
| `auto_delete` | Auto-delete remote file on local delete | `true` |
| `sync_on_start` | Upload all files on startup | `false` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codesandideas)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/codesandideas)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
