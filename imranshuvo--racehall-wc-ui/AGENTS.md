# Copilot instructions for `racehall-wc-ui`

## Release packaging (mandatory)
- Just push to staging found in .local. Bump the versions. 

## Exclusions (mandatory)
Do **not** include these paths in release zips:
- `.git/`
- `.github/`
- `.builds/`
- `.local/`
- `doc/`
- `postman/`
- Any local/dev artifacts

## Versioning before zipping (mandatory)
Before creating a release zip:
1. Bump plugin header `Version` in `racehall-wc-ui.php`.
2. Bump `RACEHALL_WC_UI_VERSION` in `racehall-wc-ui.php`.
3. Commit those version changes.

## Preferred build command
From `/var/www/html`:

```bash
zip -r racehall-wc-ui/.builds/racehall-wc-ui-v<version>.zip racehall-wc-ui \
  -x 'racehall-wc-ui/.git/*' \
     'racehall-wc-ui/.github/*' \
     'racehall-wc-ui/.builds/*' \
     'racehall-wc-ui/.local/*' \
     'racehall-wc-ui/doc/*' \
     'racehall-wc-ui/postman/*'
```

---
> Source: [imranshuvo/racehall-wc-ui](https://github.com/imranshuvo/racehall-wc-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
