# Instrucciones para GitHub Copilot - Proyecto Aria

## Arquitectura y Contexto

Aria es un sistema de gestión de tickets (tipo Jira simplificado) construido con Spring Boot 3.2.8 y Java 17. La arquitectura sigue el patrón **Hexagonal (Ports & Adapters)**, separando el dominio de la infraestructura, con H2 como base de datos en memoria.

**Principios fundamentales**:
- **Arquitectura Hexagonal**: Dominio independiente de frameworks e infraestructura
- **Clean Code**: Código legible, autodocumentado y mantenible
- **KISS (Keep It Simple, Stupid)**: Soluciones simples antes que complejas
- **SOLID**: Aplicado consistentemente en toda la arquitectura
- **JPA**: Spring Data JPA para persistencia con H2 en memoria
- **Lombok**: Usado en TODAS las clases (entidades y DTOs) para reducir boilerplate

## Estructura de Paquetes (Hexagonal)

```
com.accenture.aria/
├── controller/     # Adaptadores de entrada (REST) - Puerto HTTP
├── dto/           # Contratos de API (Request/Response con Lombok: @Data, @Builder)
├── model/         # Dominio - Entidades de negocio (@Entity con Lombok: @Getter, @Setter, @Builder)
├── repository/    # Adaptadores de salida - Puerto de Persistencia (JpaRepository)
└── service/       # Núcleo del dominio - Casos de uso y lógica de negocio + Mappers
```

**Flujo Hexagonal**:
```
HTTP Request → Controller (Adapter) → Service (Domain) → Repository (Adapter) → Database
             ↓                        ↓                   ↓
            DTOs              Lógica de Negocio      Persistencia JPA
```

**SOLID aplicado**:
- **S**ingle Responsibility: Cada clase tiene una única razón para cambiar (Controllers solo manejan HTTP, Services solo lógica de negocio)
- **O**pen/Closed: Extensible mediante herencia de repositories y composition en services
- **L**iskov Substitution: Interfaces JpaRepository permiten sustituir implementaciones
- **I**nterface Segregation: DTOs específicos por operación (Request vs Response)
- **D**ependency Inversion: Inyección por constructor, dependemos de abstracciones (Repository interfaces)

## Patrones Críticos del Proyecto

### 1. Relaciones Entity-DTO con IDs
**NO uses entidades anidadas en RequestDTOs**. Usa IDs para referencias:

```java
// ✅ CORRECTO - TicketRequestDTO
private Long reporterId;
private Long assigneeId;

// ❌ INCORRECTO - No uses esto
private Person reporter;
```

**Los ResponseDTOs SÍ contienen objetos anidados** completos:
```java
// TicketResponseDTO incluye PersonResponseDTO completo
private PersonResponseDTO reporter;
```

### 2. Mappers Estáticos con Constructor Privado
Todos los mappers son clases utilitarias con constructor privado:

```java
public class TicketMapper {
    private TicketMapper() {} // Evita instanciación
    
    public static Ticket toEntity(TicketRequestDTO dto) { ... }
    public static TicketResponseDTO toResponse(Ticket t) { ... }
}
```

### 3. Service Layer - Dos Métodos por Operación
Los servicios exponen dos métodos para crear/actualizar:

```java
// Para uso directo con entidades
public Ticket create(Ticket ticket) { ... }

// Para uso desde controllers con DTOs - resuelve relaciones por ID
public Ticket createFromDTO(TicketRequestDTO dto) {
    Ticket ticket = TicketMapper.toEntity(dto);
    if (dto.getReporterId() != null) {
        personRepo.findById(dto.getReporterId()).ifPresent(ticket::setReporter);
    }
    return create(ticket);
}
```

**Controllers DEBEN usar métodos `*FromDTO`** para manejar correctamente las relaciones.

### 4. Lazy Loading en Relaciones
Todas las relaciones `@ManyToOne` usan `FetchType.LAZY`:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "reporter_id")
private Person reporter;
```

Los mappers manejan objetos null de forma segura (ver `PersonMapper.toResponse()`).

### 5. Valores por Defecto en Service Layer
Los defaults se aplican en servicios, NO en constructores:

```java
public Ticket create(Ticket ticket) {
    if (ticket.getStatus() == null) ticket.setStatus(Status.OPEN);
    if (ticket.getPriority() == null) ticket.setPriority(Priority.MEDIUM);
    ticket.setCreatedAt(LocalDateTime.now());
    // ...
}
```

### 6. Actualización Parcial con Null-Safe
Los métodos `update` solo modifican campos no-null del DTO:

```java
if (dto.getTitle() != null) existing.setTitle(dto.getTitle());
if (dto.getStatus() != null) existing.setStatus(dto.getStatus());
```

## Convenciones de Código

### Clean Code y Buenas Prácticas
- **Nombres descriptivos**: `createFromDTO()` mejor que `create2()`
- **Métodos pequeños**: Una función = una responsabilidad
- **No Magic Numbers**: Usar constantes o enums (`Status.OPEN` no `"OPEN"`)
- **DRY**: Extraer lógica repetida a métodos privados o utilities

### Uso de Lombok
- **Entidades (@Entity)**: Usar `@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@Builder`
- **DTOs**: Usar `@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`
- **Mappers**: Constructor privado manual (no instanciables, sin Lombok)

### JPA y Persistencia
- **Enums**: Usar `@Enumerated(EnumType.STRING)` para persistencia legible
- **IDs**: Siempre `Long` con `@GeneratedValue(strategy = GenerationType.IDENTITY)`
- **Timestamps**: `LocalDateTime` para `createdAt` / `updatedAt`
- **Relaciones**: `FetchType.LAZY` por defecto, evitar N+1 queries
- **Validaciones**: Jakarta Validation solo en RequestDTOs (`@NotBlank`, `@Size`, `@Email`)

### REST y Controllers
- **Responses HTTP**: Usar `ResponseEntity<T>` con códigos apropiados (201 Created, 404 Not Found)
- **Inyección de dependencias**: Siempre por constructor (no `@Autowired` en campos)
- **KISS**: Endpoints simples, lógica compleja en services

## Comandos de Desarrollo

```bash
# Compilar
mvn clean package

# Ejecutar aplicación
mvn spring-boot:run

# Acceder a H2 Console
# URL: http://localhost:8080/h2-console
# JDBC URL: jdbc:h2:mem:ticketdb
# Usuario: sa, Password: (vacía)
```

## Estados y Enums del Dominio

```java
Status: OPEN, IN_PROGRESS, RESOLVED, CLOSED
Priority: LOW, MEDIUM, HIGH, URGENT
```

## Endpoints REST

Todos bajo `/api/{recurso}`:
- Tickets: `/api/tickets`
- Persons: `/api/persons` (incluye endpoints por email, username, department)

**Pattern**: `GET /{id}`, `POST /`, `PUT /{id}`, `DELETE /{id}`, `PATCH /{id}/deactivate` (solo Person)

## Tests

Actualmente NO hay tests automatizados. Al crear tests:
- Preferir tests unitarios con Mockito para servicios
- Tests de integración con `@SpringBootTest` y H2
- Mockear repositorios en service tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/accentureshark)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/accentureshark)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
