
# nanocontacts CLI

Use `nanocontacts` directly. Always pass `--json` for structured output.

## Read

```bash
nanocontacts me --json                           # your own contact card
nanocontacts groups --json
nanocontacts search "Jenny" --limit 10 --json   # searches name and organization
nanocontacts show --id "UUID:ABPerson" --json
```

## Write

```bash
nanocontacts create --first "Jane" --last "Doe" --email "jane@example.com" --phone "+15551234567" --json
nanocontacts update --id "UUID:ABPerson" --first "Jane" --last "Smith" --json
nanocontacts update --id "UUID:ABPerson" --add-email "new@example.com" --json
nanocontacts update --id "UUID:ABPerson" --add-phone "+15559876543" --json
```

## Workflow

1. Search before creating to avoid duplicates.
2. Use IDs from `search` results for `show` and `update`.
3. Ask for missing details before creating a new contact.
4. Ask for confirmation before creating or updating contacts.

---
> Source: [shareup/nanocontacts](https://github.com/shareup/nanocontacts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
