# Distributed Library Management System - AI Coding Instructions

## Architecture Overview

This is an **academic distributed database project** implementing a multi-branch library management system. The project demonstrates core distributed database concepts through practical implementation.

### Technology Stack
- **Backend**: Go microservices with MSSQL via Parallels on macOS
- **Frontend**: Flutter (desktop/web) with Riverpod state management  
- **Database**: MSSQL distributed across 2 sites (Site_TV_Q1, Site_TV_Q3)
- **Development**: VS Code with specialized chat modes and MCP tools

## Distributed Database Implementation

### Data Distribution Strategy
- **Fully Replicated Tables**: `CHINHANH`, `SACH` (shared across all sites)
- **Horizontally Fragmented Tables**: `QUYENSACH`, `DOCGIA`, `PHIEUMUON` (partitioned by `MaCN` branch code)
- **Fragmentation Key**: `MaCN` (branch code: 'Q1', 'Q3') determines data location

### Role-Based Access Control
- **THUTHU (Librarian)**: Branch-specific operations only, local data access
- **QUANLY (Manager)**: System-wide access, cross-site queries, distributed statistics

## Project Structure Patterns

### Backend (Go) - Expected Structure
```
backend/
├── cmd/
│   ├── site-q1/         # Site Q1 microservice
│   ├── site-q3/         # Site Q3 microservice  
│   └── coordinator/     # Distributed transaction coordinator
├── internal/
│   ├── distributed/     # 2PC protocol, query routing, site coordination
│   ├── repository/      # Data access with fragmentation logic
│   └── auth/           # JWT with role-based permissions
```

### Frontend (Flutter) - Riverpod Patterns
- Use `StateNotifierProvider` for complex state (auth, borrowing operations)
- Use `FutureProvider.family` for parameterized API calls by site/branch
- Implement `AsyncValue` patterns for distributed query loading states
- Create separate UI flows for THUTHU vs QUANLY roles

## Critical Development Context

### Database Connection on macOS
- MSSQL runs in Parallels Windows VM (typical host: `10.211.55.3:1433`)
- Implement connection pooling with retry logic for VM environment
- Handle network latency and connection failures gracefully

### Distributed Query Requirements
- **FR6**: Cross-site statistics must aggregate results from all sites
- **FR7**: Book search must query all sites and merge availability results
- Implement transparent query routing based on user role and data location

### Two-Phase Commit (2PC) Implementation
- Required for academic demonstration (book transfer between branches)
- Implement coordinator pattern in `cmd/coordinator/`
- Document transaction phases for academic report

## Chat Modes Integration

Use the specialized chat modes in `.github/chatmodes/` for context-specific assistance:
- **distributed-architecture.chatmode.md**: For 2PC, fragmentation, replication logic
- **backend-api.chatmode.md**: For Go microservices and MSSQL integration
- **flutter.chatmode.md**: For Riverpod state management and role-based UI
- **database.chatmode.md**: For schema design and query optimization

## Development Workflow

### MCP Tools Available
Leverage the configured MCP servers (see `.vscode/mcp.json`):
- **filesystem**: Project structure management
- **memory**: Track distributed system relationships and patterns
- **sequential-thinking**: Complex distributed problem solving
- **git**: Version control for multi-component system

### Key Implementation Requirements
1. **Transparency**: QUANLY users must see unified system view regardless of data distribution
2. **Site Isolation**: THUTHU users can only access their branch data
3. **Academic Focus**: Prioritize demonstrating distributed concepts over production optimization
4. **Documentation**: Explain distributed system design decisions for academic evaluation

## Testing Strategy

Focus on distributed system scenarios:
- Test fragmentation logic with different `MaCN` values
- Verify role-based access controls across sites
- Test cross-site query aggregation and result merging
- Simulate network failures between sites
- Validate 2PC protocol with concurrent transactions

When implementing features, always consider the distributed nature and which site(s) the data resides on.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zoroONE01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
