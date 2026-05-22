# GitHub Copilot Instructions - Helix6 Framework

Este proyecto está basado en el **Framework Helix6**, un framework empresarial para desarrollo de Web APIs con .NET 8 que implementa una arquitectura en capas con Clean Architecture y principios DDD.

## Contexto del Proyecto

**Framework**: Helix6 v1.0  
**Runtime**: .NET 8.0  
**Arquitectura**: N-Layer Architecture (Api → Services → Data → DataModel)  
**Patrones principales**: Repository Pattern, Service Layer, Unit of Work, DDD

## Principios Fundamentales de Helix6

### 1. Convención sobre Configuración
- Las clases siguen convenciones estrictas de nomenclatura que permiten autodescubrimiento
- Repositorios: `[Entidad]Repository` implementa `IBaseRepository<Entity>`
- Servicios: `[Entidad]Service` hereda de `BaseService<View, Entity, Metadata>`
- Views (DTOs): `[Entidad]View` implementa `IViewBase`
- Endpoints: `/api/[Entidad]/[Método]`

### 2. DRY (Don't Repeat Yourself)
- El framework proporciona clases base genéricas para CRUD automático
- No duplicar código que ya existe en `BaseService` o `BaseRepository`
- Sobrescribir solo los hooks necesarios: `ValidateView`, `PreviousActions`, `PostActions`

### 3. Separación de Responsabilidades
```
┌─────────────────────────────────────────┐
│  Api (Presentación)                     │  ← Endpoints HTTP, configuración
├─────────────────────────────────────────┤
│  Entities (DTOs/Views)                  │  ← Transferencia de datos
├─────────────────────────────────────────┤
│  Services (Lógica de Negocio)           │  ← Validaciones, reglas de dominio
├─────────────────────────────────────────┤
│  Data (Acceso a Datos)                  │  ← Repositorios, EF Core, Dapper
├─────────────────────────────────────────┤
│  DataModel (Modelo de Base de Datos)    │  ← Entidades que mapean a tablas
└─────────────────────────────────────────┘
```

**Regla crítica**: NUNCA saltar capas (ej: no llamar repositorios desde endpoints)

## Estructura de Carpetas Estándar

```
[Proyecto].Api/
├── Program.cs                    # Bootstrapping, DI, middleware
├── Endpoints/
│   ├── Endpoints.cs             # Endpoints personalizados
│   └── Base/Generator/          # Endpoints auto-generados (NO MODIFICAR)
├── Extensions/
│   ├── DependencyInjection.cs   # Registro de servicios/repositorios
│   ├── AuthConfiguration.cs     # JWT, autenticación
│   └── SwaggerConfiguration.cs  # OpenAPI
└── Security/                    # UserContext, claims mapping

[Proyecto].DataModel/
└── [Entidad].cs                 # Clases POCO que mapean a BD (implementan IEntityBase)

[Proyecto].Entities/
├── Views/                       # DTOs generados
│   ├── [Entidad]View.cs        # Auto-generado (NO MODIFICAR directamente)
│   └── Metadata/               # Atributos de validación
└── PartialViews/               # Extensiones personalizadas ([Entidad]View.Custom.cs)

[Proyecto].Data/
├── EntityModel.cs               # DbContext de EF Core
└── Repository/
    ├── [Entidad]Repository.cs   # Implementación concreta
    └── Interfaces/              # Contratos de repositorios

[Proyecto].Services/
├── [Entidad]Service.cs          # Lógica de negocio
├── ServiceConsts.cs             # Constantes de validación
└── Base/                        # Servicios del framework (seguridad, attachments)
```

## Convenciones de Código (OBLIGATORIAS)

### Nomenclatura de Clases
| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| Entidad | PascalCase singular | `Worker`, `Invoice` |
| View | `[Entidad]View` | `WorkerView` |
| Servicio | `[Entidad]Service` | `WorkerService` |
| Repositorio | `[Entidad]Repository` | `WorkerRepository` |
| Interfaz | `I[Nombre]` | `IWorkerService` |
| Constantes | `[Contexto]Consts` | `ServiceConsts` |

### Nomenclatura en Código C#
```csharp
// ✅ CORRECTO
public class WorkerService : BaseService<WorkerView, Worker, WorkerViewMetadata>
{
    private readonly IWorkerRepository _workerRepository;  // ✅ Prefijo _ para privados
    private readonly ILogger<WorkerService> _logger;       // ✅ camelCase con _
    
    public WorkerService(                                  // ✅ Parámetros en camelCase
        IApplicationContext applicationContext,
        IUserContext userContext,
        IWorkerRepository repository)
        : base(applicationContext, userContext, repository)
    {
        _workerRepository = repository;
    }
    
    public override async Task ValidateView(               // ✅ PascalCase para métodos
        HelixValidationProblem validations,
        WorkerView? view,
        EnumActionType actionType,
        string? configurationName = null)
    {
        if (view != null)
        {
            var minimumAge = 18;                           // ✅ Variables locales camelCase
            // Validaciones personalizadas
        }
        
        await base.ValidateView(validations, view, actionType, configurationName);  // ✅ SIEMPRE llamar a base
    }
}

// ❌ INCORRECTO
public class WorkerService
{
    private IWorkerRepository workerRepository;  // ❌ Falta prefijo _
    
    public WorkerService(IWorkerRepository WorkerRepository)  // ❌ Parámetro en PascalCase
    {
        workerRepository = WorkerRepository;
    }
    
    public override async Task ValidateView(...)
    {
        // lógica
        // ❌ FALTA: await base.ValidateView(...);
    }
}
```

## Reglas para Generación de Código

### Al crear una Entidad (DataModel)
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Helix6.Base.Domain.BaseInterfaces;

[Table("Workers", Schema = "dbo")]
public class Worker : IEntityBase  // ✅ Implementar SIEMPRE IEntityBase
{
    [Key]
    public int Id { get; set; }
    
    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;
    
    // Clave foránea: [Entidad]Id
    public int WorkerTypeId { get; set; }
    
    // Navegación: virtual para lazy loading
    public virtual WorkerType? WorkerType { get; set; }
    
    // Colecciones: ICollection<T>
    public virtual ICollection<Worker_Project>? Projects { get; set; }
    
    // ✅ OBLIGATORIO: Propiedades de auditoría
    public int AuditCreationUser { get; set; }
    public DateTime AuditCreationDate { get; set; }
    public int AuditModificationUser { get; set; }
    public DateTime AuditModificationDate { get; set; }
    public DateTime? AuditDeletionDate { get; set; }
}
```

### Al crear un Servicio
```csharp
public class WorkerService : BaseService<WorkerView, Worker, WorkerViewMetadata>
{
    private readonly IWorkerRepository _workerRepository;
    
    // Constructor con parámetros base SIEMPRE
    public WorkerService(
        IApplicationContext applicationContext,
        IUserContext userContext,
        IWorkerRepository repository)
        : base(applicationContext, userContext, repository)
    {
        _workerRepository = repository;
    }
    
    // Sobrescribir solo cuando sea necesario:
    
    public override async Task<WorkerView?> GetNewEntity()
    {
        // Valores por defecto para nueva entidad
        return await Task.FromResult(new WorkerView { Active = true });
    }
    
    public override async Task ValidateView(
        HelixValidationProblem validations,
        WorkerView? view,
        EnumActionType actionType,
        string? configurationName = null)
    {
        if (view != null)
        {
            // Validaciones de negocio
            if (string.IsNullOrWhiteSpace(view.Name))
                validations.AddError(ServiceConsts.Validations.Worker.NAME_REQUIRED);
        }
        
        await base.ValidateView(validations, view, actionType, configurationName);
    }
    
    public override async Task PreviousActions(
        WorkerView? view,
        EnumActionType actionType,
        string? configurationName = null)
    {
        // Lógica antes de Insert/Update/Delete
        await base.PreviousActions(view, actionType, configurationName);
    }
    
    public override async Task PostActions(
        WorkerView? view,
        EnumActionType actionType,
        string? configurationName = null)
    {
        // Lógica después de Insert/Update/Delete (ej: notificaciones)
        await base.PostActions(view, actionType, configurationName);
    }
}
```

### Al crear un Repositorio
```csharp
// Interfaz
public interface IWorkerRepository : IBaseRepository<Worker>
{
    Task<List<Worker>> GetActiveWorkers();  // Solo métodos personalizados
}

// Implementación
public class WorkerRepository : BaseRepository<Worker>, IWorkerRepository
{
    public WorkerRepository(
        IApplicationContext applicationContext,
        IUserContext userContext,
        IBaseEFRepository<Worker> baseEFRepository,
        IBaseDapperRepository<Worker> baseDapperRepository)
        : base(applicationContext, userContext, baseEFRepository, baseDapperRepository)
    {
    }
    
    public async Task<List<Worker>> GetActiveWorkers()
    {
        return await ExecuteQuery(
            "SELECT * FROM Workers WHERE AuditDeletionDate IS NULL"
        );
    }
}
```

### Al crear Endpoints Personalizados
```csharp
// En [Proyecto].Api/Endpoints/Endpoints.cs
public static void MapCustomEndpoints(this IEndpointRouteBuilder app)
{
    app.MapGet("/api/Worker/Active", async (IWorkerService service) =>
    {
        var result = await service.GetActiveWorkers();
        return Results.Ok(result);
    })
    .WithName("GetActiveWorkers")
    .WithTags("Worker")
    .RequireAuthorization();  // Si requiere autenticación
}
```

## Patrones Prohibidos

### ❌ NO HACER:
```csharp
// ❌ NO llamar repositorio desde endpoint
app.MapGet("/api/Worker/{id}", (int id, IWorkerRepository repo) => ...);

// ❌ NO usar Entity en endpoints (usar View)
app.MapPost("/api/Worker", (Worker entity, ...) => ...);

// ❌ NO implementar CRUD en repositorios (ya está en BaseRepository)
public async Task<Worker?> GetById(int id) { /* implementación manual */ }

// ❌ NO omitir llamada a base en overrides
public override async Task ValidateView(...) { /* lógica */ }  // Falta base.ValidateView

// ❌ NO saltarse capas
public class WorkerController { 
    public WorkerController(IWorkerRepository repo) { } // Debe usar IWorkerService
}

// ❌ NO modificar código en carpetas Generator/
// [Proyecto].Api/Endpoints/Base/Generator/WorkerEndpoints.cs  <- Se regenera automáticamente
```

### ✅ SÍ HACER:
```csharp
// ✅ Usar servicios en endpoints
app.MapGet("/api/Worker/{id}", (int id, IWorkerService service) => ...);

// ✅ Usar Views en endpoints
app.MapPost("/api/Worker", (WorkerView view, IWorkerService service) => ...);

// ✅ Delegar a BaseRepository
public class WorkerRepository : BaseRepository<Worker>, IWorkerRepository { }

// ✅ Llamar siempre a base
public override async Task ValidateView(...)
{
    // lógica personalizada
    await base.ValidateView(validations, view, actionType, configurationName);
}

// ✅ Respetar las capas
Api (Endpoints) → Services → Repositories → DataModel
```

## Características del Framework

### Auditoría Automática
Todas las entidades incluyen campos de auditoría que se rellenan automáticamente:
- `AuditCreationUser`, `AuditCreationDate` (al insertar)
- `AuditModificationUser`, `AuditModificationDate` (al actualizar)
- `AuditDeletionDate` (para soft delete)

### Soft Delete
El framework implementa eliminación lógica:
```csharp
// Eliminar lógicamente
await service.DeleteUndeleteLogicById(id);

// Restaurar
await service.DeleteUndeleteLogicById(id);
```

### Hooks del Ciclo de Vida (Service Layer)
```
Insert/Update/Delete Request
    ↓
ValidateView()          // Validaciones de negocio
    ↓
PreviousActions()       // Lógica previa (ej: eliminar relacionados)
    ↓
MapViewToEntity()       // Conversión View → Entity
    ↓
Repository.Insert()     // Persistencia
    ↓
MapEntityToView()       // Conversión Entity → View
    ↓
PostActions()           // Lógica posterior (ej: enviar email)
    ↓
Response
```

### Filtros Kendo
El framework soporta filtros compatibles con Kendo UI:
```csharp
// Endpoint generado automáticamente
/api/Worker/GetAllKendoFilter

// Acepta: { "Filter": {...}, "Sort": [...], "Page": 1, "PageSize": 10 }
// Retorna: { "TotalCount": 100, "Items": [...] }
```

### Seguridad y Permisos
```csharp
// IUserContext - Inyectable en servicios/repositorios
public interface IUserContext
{
    int UserId { get; }
    string UserName { get; }
    List<string> Roles { get; }
    string Language { get; }
}

// Endpoints con autorización
.RequireAuthorization()  // Requiere estar autenticado
```

## Organización de Imports

```csharp
// 1. System namespaces
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

// 2. Third-party packages
using Microsoft.EntityFrameworkCore;

// 3. Helix6.Base
using Helix6.Base.Domain.BaseInterfaces;
using Helix6.Base.Service;

// 4. Project namespaces
using [Proyecto].DataModel;
using [Proyecto].Entities.Views;
```

## Tecnologías Utilizadas

- **.NET 8.0**: Runtime principal
- **Entity Framework Core 9.0**: ORM para operaciones de escritura
- **Dapper 2.1**: Micro-ORM para consultas de lectura optimizadas
- **Mapster 7.4**: Mapeo objeto-objeto de alto rendimiento
- **Serilog**: Logging estructurado
- **JWT Bearer**: Autenticación
- **Swagger/OpenAPI**: Documentación de API

## Nullable Reference Types

El proyecto tiene habilitado `#nullable enable`. Reglas:
```csharp
public string Name { get; set; } = string.Empty;  // ✅ No nullable
public string? Email { get; set; }                // ✅ Nullable
public WorkerType? WorkerType { get; set; }       // ✅ Navegación nullable
public ICollection<Course>? Courses { get; set; } // ✅ Colección nullable
```

## Archivo HelixEntities.xml

El archivo `HelixEntities.xml` en el proyecto Api define qué entidades se exponen y qué endpoints se generan:
```xml
<HelixEntities>
  <Entities>
    <EntityName>Worker</EntityName>
    <GetById>true</GetById>
    <Insert>true</Insert>
    <Update>true</Update>
    <Delete>true</Delete>
    <GetAllKendoFilter>true</GetAllKendoFilter>
  </Entities>
</HelixEntities>
```

**Después de modificar este archivo, ejecutar el Helix Generator** para regenerar Views y Endpoints.

## Checklist para Código Generado

Antes de considerar completo cualquier código, verifica:

- [ ] Convenciones de nomenclatura seguidas correctamente
- [ ] Campos privados usan prefijo `_`
- [ ] Parámetros de constructores en camelCase
- [ ] Entidades implementan `IEntityBase` con propiedades de auditoría
- [ ] Servicios heredan de `BaseService<TView, TEntity, TMetadata>`
- [ ] Repositorios heredan de `BaseRepository<TEntity>`
- [ ] Overrides de métodos llaman a `await base.[Método]()`
- [ ] Se usan Views en endpoints, no Entities
- [ ] Todos los métodos son asíncronos (`async Task`)
- [ ] `#nullable enable` está presente
- [ ] Imports organizados (System → Third-party → Project)
- [ ] No se duplica código de clases base
- [ ] Se respeta la separación de capas

## Referencias para Documentación Completa

- Archivo de instrucciones específicas: `.github/instructions/backend.instructions.md`
- Documentación de arquitectura: `docs/[Proyecto]_Architecture.md`
- Ejemplos de código: Clases en carpetas `Base/` de cada proyecto

---

**Versión**: Helix6 v1.0  
**Última actualización**: Enero 2026

---
> Source: [malbert-infoport/AI4Devs](https://github.com/malbert-infoport/AI4Devs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
