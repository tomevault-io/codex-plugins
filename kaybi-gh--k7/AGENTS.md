
# Domain Layer

Zero dependencies on other projects. Entities inherit `BaseEntity`. Events inherit `BaseEvent`, raised via `AddDomainEvent()`.

Structure: `Common/`, `Constants/`, `Entities/`, `Enums/`, `Events/`, `Exceptions/`, `Interfaces/`, `Models/`, `Settings/`, ...

Service contracts live in Domain; Infrastructure implements them. `IApplicationDbContext` lives in Application (`Common/Interfaces/`).

---
> Source: [kaybi-gh/K7](https://github.com/kaybi-gh/K7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-30 -->
