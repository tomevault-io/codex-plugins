# Copilot Instructions

## Build, Test, and Lint

```bash
# Run all tests (with coverage)
pytest

# Run a single test file
pytest tests/test_connection.py

# Run a single test by name
pytest tests/test_connection.py::TestCassandraConnection::test_connect_with_auth

# Format code
black src/ tests/

# Lint
flake8 src/ tests/
pylint src/ tests/
```

Coverage reports are written to `htmlcov/` (HTML) and printed to the terminal. Coverage is configured automatically via `pyproject.toml`.

## Architecture

The project has three modules under `src/cassandra_python/`:

- **`connection.py`** — `CassandraConnection` wraps the `cassandra-driver` `Cluster` object. Call `.connect(keyspace)` to get a session, `.disconnect()` to shut down.
- **`table.py`** — `CassandraTable` takes a live session + keyspace + table name and exposes `create_table`, `drop_table`, `insert_data`, and `query_data`. All queries are built as f-strings and executed via `session.execute()`.
- **`main.py`** — Entry point. Loads `config/config.yaml` via PyYAML, instantiates `CassandraConnection` and `CassandraTable`, and demonstrates basic operations.

Configuration lives in `config/config.yaml` (hosts, port, keyspace, optional auth). `main.py` resolves this path relative to `__file__` at runtime.

## Key Conventions

- **Tests use mocks, not a live Cassandra instance.** `unittest.mock.patch` is applied at the `cassandra_python.connection.Cluster` import path. No Docker/Cassandra is needed to run the test suite.
- **`query_data` always appends `ALLOW FILTERING`** when a condition is provided. This is intentional for the demo scope; be aware of the Cassandra performance implications when extending it.
- **`insert_data` uses `%s` positional placeholders** (cassandra-driver style), not named parameters. The column order matches `dict.keys()` insertion order (Python 3.7+).
- **Line length is 88** (Black default), enforced via `pyproject.toml`.
- **`src` layout**: the package root is `src/`, so `sys.path.insert(0, "../src")` appears in both `main.py` and the test files for local imports.

## Running Cassandra (Docker)

```bash
docker run --name cassandra -p 9042:9042 \
  -e CASSANDRA_RACK=RAC1 -e CASSANDRA_DC=DC1 \
  -e CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch \
  -d cassandra:latest

# Wait until ready (~30s)
docker logs cassandra | grep -i "state jump to NORMAL"
```

The app reads connection settings from `config/config.yaml`. Defaults: `localhost:9042`, keyspace `demo`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ermac517) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
