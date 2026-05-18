# Queztl-Core - Copilot Instructions (Read First)

## What this repo is
Queztl-Core is a **distributed computing system** with an **agent-based architecture** (DNA/RNA pattern).
It includes a FastAPI backend plus tooling for orchestration, training, and compute delegation.

## Physical topology (reality matters)
This system spans multiple machines. Your code and commands must respect the network constraints:

- **Beast (DHCP, RTX 4090)**: GPU-heavy workloads (image generation / accelerated tasks)
- **Sloth (DHCP)**: Orchestration node (Ray head / persistent “slow memory” role)
- **Optiplex cluster (DNS, remote site)**: CPU worker nodes (hostnames: `optiplex1`, `optiplex2`, ...)
- **Laptop (macOS)**: **Command center only** — *no compute*, no local Ray, no training, no image gen

### DHCP reality
Beast and Sloth **change IPs**. Do not hardcode them.

Use discovery when needed:
```bash
nmap -sn 192.168.1.0/24
```
If you must use explicit hosts, prefer environment variables:
- `QUEZTL_BEAST_HOST`
- `QUEZTL_SLOTH_HOST`
- `QUEZTL_DISCOVER_SUBNET`

## Non-negotiable rules
1. **No compute on the laptop.** If you need CPU/GPU, delegate to cluster (SSH / Ray / docker-on-remote).
2. **Never hardcode Beast/Sloth IPs.** Use discovery or env vars.
3. **Optiplex nodes use DNS hostnames.** Use `optiplexN` hostnames, not IPs.
4. Prefer **one execution choke point** (central command executor) rather than ad-hoc `subprocess.run(...)` everywhere.
5. Fail fast: do not silently fall back to “local mode”.

## Execution patterns
### Preferred
- Laptop submits tasks via **SSH** to Sloth/Beast.
- Sloth coordinates and/or runs Ray head.
- Beast runs GPU services.

### Helpful env vars
- `QUEZTL_EXEC_MODE`: `local|ssh|docker|ssh_docker`
- `QUEZTL_SSH_USER`, `QUEZTL_SSH_HOST`
- `QUEZTL_DOCKER_CONTAINER` (e.g. `ray-head`)
- `QUEZTL_REMOTE_CWD`

## Codebase navigation
- Backend has **many Python files**. Do not “open everything”.
- Use `grep`/search and follow call sites.
- Identify the **routing/dispatch** layer first, then agent logic.

## Agent architecture (DNA/RNA)
- **DNA**: persistent knowledge (models, code snippets, datasets, learned skills)
- **RNA**: runtime skills/behavior (functions + state)
- **Spawn/Teach pattern**: parents spawn children and transfer knowledge.

## Common pitfalls
- Starting Ray locally on the laptop (avoid `ray.init()` without an explicit remote address)
- Writing artifacts to laptop-only paths
- Assuming a stable IP for DHCP nodes
- Executing GPU work on CPU-only nodes (use scheduling / resource labels / role routing)

---
> Source: [La-Potencia-Cananbis/queztl-core](https://github.com/La-Potencia-Cananbis/queztl-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
