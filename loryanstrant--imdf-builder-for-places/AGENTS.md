# Copilot instructions — IMDF-Builder-for-Places

> Canonical standards live in the `dev-standards` repo on SOUNDWAVE/Gitea.
> Read by Copilot chat **and** inline suggestions.

## What this repo is

A standalone **Dockerised Node web app** — an editor for building IMDF (Indoor
Mapping Data Format) for Apple Maps Places. Express server + a browser canvas UI
built on Fabric.js. Not a Home Assistant component.

## Repo shape

- `server.js` — Express entrypoint.
- `public/` — frontend: `index.html`, `css/styles.css`, `js/app.js` (the editor),
  and **`lib/fabric.min.js`** (vendored third-party — don't hand-edit or "tidy").
- `Dockerfile`, `docker-compose.yml`, `package.json` (+ lock).
- `.github/workflows/docker-publish.yml` — builds + publishes the image.

## Conventions

- Node web app: no `manifest.json`/`hassfest`/HACS.
- Your code is `server.js` + `public/css` + `public/js/app.js`; leave the vendored
  `public/lib/` alone.
- JS deps via npm; commit `package-lock.json`. CI publishes the image.
- Any Apple/MapKit credentials or API keys are config via env — never commit them.

## Never

- Don't commit API keys or credentials. Don't hand-edit vendored libs.

---
> Source: [loryanstrant/IMDF-Builder-for-Places](https://github.com/loryanstrant/IMDF-Builder-for-Places) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-04 -->
