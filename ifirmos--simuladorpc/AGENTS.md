# Copilot Instructions for SimuladorPC

## Project Overview

**SimuladorPC** is a PC component simulator and builder application. Users can browse hardware components (CPU, GPU, RAM, motherboard, etc.) and assemble a virtual PC build. The project is written in Brazilian Portuguese for domain terminology.

## Architecture

The solution follows **Clean Architecture** with four layers:

- **`SimuladorPC.Domain`** – Core business logic, entities, interfaces, and domain services. No external dependencies.
- **`SimuladorPC.Application`** – DTOs and AutoMapper configuration (`Mapper/AutoMappingConfig.cs`).
- **`SimuladorPC.Infrastructure`** – Entity Framework Core (SQL Server) data access, `DbContext` (`SimuladorPcContext`), and repository implementations.
- **`SimuladorPC.Api`** – ASP.NET Core Web API controllers. Entry point is `Program.cs`.
- **`SimuladorPCSpa`** – Angular 17 single-page application frontend.

## Backend (.NET / C#)

- **Framework**: ASP.NET Core Web API (.NET 8)
- **ORM**: Entity Framework Core with SQL Server
- **Mapping**: AutoMapper (`SimuladorPC.Application/Mapper/AutoMappingConfig.cs`)
- **Dependency injection** is wired in `SimuladorPC.Api/Program.cs`
- **Repository pattern**: `IBaseRepository<T>` / `BaseRepository<T>` for generic CRUD; specialized repositories (e.g. `ICpuRepository`) extend it.
- **Service pattern**: Domain services in `SimuladorPC.Domain/Services`; interfaces in `SimuladorPC.Domain/Interfaces/Services`.
- **Controllers** live in `SimuladorPC.Api/Controllers/HardwareControllers/` and follow the pattern `[Entity]Controller.cs`.
- **Enums** are serialized as strings (`JsonStringEnumConverter`).
- CORS is configured to allow all origins (development setting).

### Key Domain Entities (Hardware)

All hardware entities inherit from `Componente` (base component):

| Entity | Description |
|---|---|
| `Cpu` | Processor |
| `GPU` | Graphics card |
| `Ram` | Memory |
| `Ssd` | Solid state drive |
| `Hd` | Hard disk drive |
| `PlacaMae` | Motherboard |
| `Gabinete` | PC case |
| `Fonte` | Power supply unit |
| `WaterCooler` | Liquid cooler |
| `Chipset` | Chipset |
| `PciExpressSlot` | PCIe slot |
| `PortaUSB` | USB port |

Software entities live in `SimuladorPC.Domain/Entities/Software/`.

`Computador` and `SetupPc` represent an assembled PC build.

### Naming Conventions (Backend)

- **Brazilian Portuguese** for domain terms, entity names, property names, and variable names (e.g. `Fabricante` = manufacturer, `PlacaMae` = motherboard, `Gabinete` = case).
- Interface names are prefixed with `I` (e.g. `ICpuRepository`, `ICpuService`).
- Repository interfaces: `I[Entity]Repository` in `SimuladorPC.Domain/Interfaces/Repositories/`.
- Service interfaces: `I[Entity]Service` in `SimuladorPC.Domain/Interfaces/Services/`.
- Concrete repositories: `[Entity]Repository` in `SimuladorPC.Infrastructure/Repositories/`.
- Concrete services: `[Entity]Service` in `SimuladorPC.Domain/Services/`.
- Controllers: `[Entity]Controller` in `SimuladorPC.Api/Controllers/HardwareControllers/`.

## Frontend (Angular / TypeScript)

- **Framework**: Angular 17
- **Language**: TypeScript
- **Testing**: Karma + Jasmine (`ng test`)
- **Build**: Angular CLI (`ng build`)
- **Dev server**: `ng serve --host=127.0.0.1`
- Source lives in `SimuladorPCSpa/src/app/`.
- The main service is `pc-builder.service.ts`.

### Naming Conventions (Frontend)

- Angular conventions: kebab-case file names, PascalCase class names.
- Service files: `[feature].service.ts`; component files: `[feature].component.ts`.

## How to Run

### Backend
```bash
cd SimuladorPC.Api
dotnet run
```
Requires a SQL Server connection string named `SimuladorPcDatabase2` in `appsettings.json`.

### Frontend
```bash
cd SimuladorPCSpa
npm install
npm start
```

### Run Backend Tests
```bash
dotnet test
```

### Run Frontend Tests
```bash
cd SimuladorPCSpa
npm test
```

## Code Style Guidelines

- Follow existing patterns when adding new hardware components: create entity → repository interface → service interface → repository implementation → service implementation → controller → DTO → mapper config → register in `Program.cs`.
- Keep domain logic in `SimuladorPC.Domain`; do not add infrastructure or framework dependencies there.
- Use `async`/`await` for all database operations.
- Add new Angular features using Angular CLI (`ng generate component`, `ng generate service`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ifirmos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
