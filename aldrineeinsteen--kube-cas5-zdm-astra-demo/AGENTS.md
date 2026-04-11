# Cassandra 5 Zero Downtime Migration Demo

This project demonstrates migrating from Apache Cassandra 5 to DataStax Astra DB using Zero Downtime Migrator (ZDM) on Kubernetes.

## Project Structure

- `k8s/cassandra/` - Cassandra 5 StatefulSet (demo storage only)
- `k8s/zdm-proxy/` - ZDM Proxy deployment with ConfigMap/Secret
- `k8s/data-generator/` - Job to populate demo data
- `python-api/` - FastAPI service for data operations
- `Makefile` - Main build and deployment commands

## Migration Workflow

**Official DataStax 5-Phase Migration Process:**
- **Phase 1**: Deploy ZDM Proxy - dual writes automatically active when both clusters configured
- **Phase 2**: Migrate existing data from origin to target
- **Phase 3**: Enable async dual reads (optional testing phase)
- **Phase 4**: Route primary reads to target cluster
- **Phase 5**: Direct connection to target, decommission origin

## Essential Commands

```bash
make up       # Start kind cluster and Cassandra
make data     # Generate demo data
make zdm      # Deploy ZDM proxy
make status   # Check all components
make down     # Clean teardown
```

## ZDM Proxy Configuration

**ConfigMap Pattern:**
- Source cluster: `cassandra-svc:9042` (in-cluster service)
- Target: Astra DB via Secure Connect Bundle
- Phase switching via environment variables

**Important ZDM Behavior:**
- ZDM Phase 1 automatically activates dual-write when both origin and target are configured
- Environment variables `ZDM_READ_MODE=PRIMARY_ONLY` and `ZDM_WRITE_MODE=PRIMARY_ONLY` are correct but dual-write logic is still active per DataStax specifications

**Common Issues:**
- Proxy fails to start: Check Astra credentials in Secret
- Connection timeouts: Verify Cassandra service is ready
- Phase switching: Restart proxy deployment after config changes

## Development Notes

- Demo uses `emptyDir` storage (not persistent)
- Data schema: `UUID, name, email, gender, address`
- All secrets managed via Kubernetes Secret objects
- Use `.env.example` template for local configuration
- British English in all documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aldrineeinsteen)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/aldrineeinsteen)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
