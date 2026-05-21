# GitHub Copilot Instructions

This is a n8n workflow as code project.
We store workflow.json file in each workflows own folder.
Workflow files are updated, often using Ai-Editors and agents.

Use npm scripts to run scripts I mention. Try to stay away from running scripts directly with node or sh. Only do so when the script we're trying to run is not registered in package.json.

We have scripts folder that has automating n8n workflow updates using its API.
Use `n8n_api_openapi.yml` file as reference to n8n api endpoints, request parameters.

NEVER edit .env file. Suggest edits but never change it directly.

N8N api credentials saved in .env file with following variables:

- N8N_API_URL
- N8N_API_KEY

Other .env settings:

- BACKUP_WORKFLOWS_ON_PULL
- BACKUP_WORKFLOWS_ON_PUSH

---
> Source: [mfyz/n8n-workflows-as-code](https://github.com/mfyz/n8n-workflows-as-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-21 -->
