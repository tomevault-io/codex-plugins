# Nostr Plugin for Dillo - Claude Code Context

This document provides context for Claude Code when working with this Nostr plugin for Dillo.

## Project Overview

A Nostr protocol plugin for Dillo browser written in Go. It fetches Nostr events from relays and renders them as HTML.

## Project Structure

```
nostr/
├── cmd/nostr/main.go           # Entry point, URI handling, event routing
├── internal/
│   ├── core/                   # Core Nostr functionality
│   │   ├── client.go           # WebSocket client for relay communication
│   │   ├── config.go           # Configuration management (config.ini)
│   │   ├── types.go            # Event types (Profile, Article, etc.)
│   │   ├── uri.go              # NIP-21 URI parsing (npub, note, nevent, naddr, nprofile)
│   │   ├── bech32.go           # bech32 encoding/decoding
│   │   ├── crypto.go           # Signature verification
│   │   ├── nip05.go            # NIP-05 identifier resolution
│   │   ├── thread.go           # Thread fetching (ancestors/replies)
│   │   ├── pool.go             # Connection pool
│   │   ├── cache.go            # Event caching
│   │   └── constants.go        # Constants
│   ├── dpi/
│   │   └── dpip.go             # Dillo protocol (reads stdin, writes stdout)
│   ├── logger/
│   │   └── log.go              # Debug logging
│   └── rendering/
│       ├── renderer.go         # Main renderer with content processing
│       ├── templates.go         # CSS styles (2000+ lines)
│       ├── notes.go            # Note/thread rendering
│       ├── profiles.go          # Profile rendering
│       ├── content.go           # Content parsing
│       ├── social.go            # Reactions, reposts
│       ├── zaps.go              # Zap receipt rendering
│       ├── wiki.go              # Wiki article rendering
│       ├── media.go             # Media gallery rendering
│       ├── calendar.go          # Calendar event rendering
│       ├── badges.go            # Badge rendering
│       ├── events.go            # Generic event rendering
│       ├── error.go             # Error rendering
│       └── images.go            # Image URL handling
├── config.ini                   # Plugin configuration
├── Makefile                     # Build commands
└── nostr.filter.dpi            # Compiled binary
```

## Supported URI Types

- `npub` / `npub1...` - Public key (displays profile)
- `nprofile` / `nprofile1...` - Profile with relay hints
- `note` / `note1...` - Text note by ID
- `nevent` / `nevent1...` - Event with relay hints
- `naddr` / `naddr1...` - Addressable events (repositories, articles)
- Raw hex event IDs (64 characters)
- NIP-05 identifiers (e.g., `user@domain.com`)

## Supported Event Kinds

| Kind | Description | File |
|------|-------------|------|
| 0 | Profile | `profiles.go` |
| 1 | Text Note (with thread) | `notes.go` |
| 3 | Contact List | `social.go` |
| 6/16 | Repost | `social.go` |
| 8 | Badge Definition | `badges.go` |
| 10 | Mute List | `social.go` |
| 21/22 | Video Event | `events.go` |
| 64 | Chess Game | `events.go` |
| 9735 | Zap Receipt | `zaps.go` |
| 1063 | File Metadata | `events.go` |
| 1111 | Comment | `events.go` |
| 1263 | Moderated Item | `events.go` |
| 1617/1618 | Patch (Git) | `events.go` |
| 1621 | Issue (Git) | `events.go` |
| 1985 | Label | `events.go` |
| 2003 | Torrent | `events.go` |
| 30311 | Live Event | `events.go` |
| 30315 | User Status | `events.go` |
| 30402 | Classified Listing | `events.go` |
| 30001 | Bookmark Set | `events.go` |
| 30008 | Badge Award | `badges.go` |
| 30009 | Profile Badges | `badges.go` |
| 30023 | Long-form Article (NIP-23) | `events.go` |
| 30063 | Media Gallery | `media.go` |
| 30617 | Repository (NIP-34) | `events.go` |
| 30818 | Wiki Article (NIP-54) | `wiki.go` |
| 31922/31923 | Calendar Event | `calendar.go` |
| 31989/31990 | Handler Recommendation | `events.go` |
| 34550 | Community | `events.go` |
| 9802 | Highlights | `events.go` |
| 10002 | Relay List | `events.go` |
| 10051 | Follow Set | `social.go` |

## Dillo Protocol

The plugin communicates via stdin/stdout using DPI protocol:

1. Reads `auth` command from stdin
2. Reads `open_url` command with `url='nostr:...'` 
3. Fetches event from relays
4. Outputs Dillo-specific headers:
   ```
   <cmd='start_send_page' url='<URI>'>
   HTTP/1.1 200 OK
   Content-type: text/html; charset=utf-8
   ```
5. Outputs rendered HTML

## Configuration

Located at `~/.dillo/dpi/nostr/config.ini`:

```ini
[nostr]
default_relays = wss://relay.nostr.band,wss://nos.lol,wss://nostr.land

[plugin]
max_events = 100
enable_reactions = false
enable_zaps = false

[display]
avatar_image_width = 400
content_image_width = 800

[network]
connect_timeout = 5s
request_timeout = 5s
image_fetch_timeout = 10s

[security]
verify_signatures = true
validate_events = true
```

## Key Implementation Details

- **URI Parsing**: `internal/core/uri.go` - handles bech32 decoding, TLV parsing for nprofile/nevent
- **DPI Protocol**: `internal/dpi/dpip.go` - reads commands, extracts URL, handles protocol errors
- **Rendering**: `internal/rendering/` - Go templates with custom functions
- **Thread Fetching**: Fetches ancestors and replies with 5s timeout; falls back to single note
- **Images**: Uses NIP-96 URL parameters (`?w=400`, `?w=800`) instead of embedding to avoid Dillo crashes
- **Performance**: Profile/username resolution disabled by default (5s timeout blocks rendering)

## Troubleshooting

- **Blank page**: Check `~/.dillo/dpid.log` for errors; ensure dpid is running
- **No content**: Verify `~/.dillo/filters` contains `nostr:` protocol handler
- **Slow loading**: Network timeouts may be too short for slow relays
- **Crashes**: Large images crash Dillo; ensure NIP-96 image URLs are used

## Build Commands

```bash
make build    # Build binary
make install  # Install to ~/.dillo/dpi/nostr/
make test     # Run tests
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jvalinsky)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jvalinsky)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
