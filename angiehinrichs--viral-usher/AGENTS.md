<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

This project is a Python module with a driver script that:
- Interactively queries the user for a viral species name
- Queries NCBI to find the Taxonomy ID for the species
- Queries NCBI to find the RefSeq IDs associated with the Taxonomy ID
- Queries the user to select from among the available RefSeq IDs
- Uses the NCBI Datasets API to download the selected RefSeqs and all GenBank genomes associated with the Taxonomy ID

---
> Source: [AngieHinrichs/viral_usher](https://github.com/AngieHinrichs/viral_usher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
