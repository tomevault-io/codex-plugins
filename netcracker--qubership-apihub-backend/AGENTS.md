
# Backend OpenAPI Files

Any REST endpoint or contract change **must** update the relevant OpenAPI files under `docs/api/`:

- Public API: `docs/api/APIHUB_API.yaml`
- Admin API: `docs/api/Admin API.yaml`
- Internal API: `docs/api/APIHUB_API_internal.yaml` (when internal endpoints change)

Do not introduce breaking public API changes without versioning and deprecation per `docs/development_guide.md`.

---
> Source: [Netcracker/qubership-apihub-backend](https://github.com/Netcracker/qubership-apihub-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-05 -->
