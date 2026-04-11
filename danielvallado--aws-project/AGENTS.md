# AWS ACADEMY FOUNDATIONS API - AI REQUIREMENTS

API REST para gestionar alumnos y profesores en memoria. Este documento es la única fuente de verdad para IA al trabajar en este proyecto.

---

## STACK TÉCNICO

- **Runtime**: Java 17
- **Framework**: Spring Boot 3.x
- **Build**: Maven
- **Web**: `spring-boot-starter-web`
- **Validación**: `spring-boot-starter-validation` (Jakarta Validation)
- **Utilidades**: `lombok`
- **Tests**: `spring-boot-starter-test` (JUnit 5)
- **Logging**: Logback estándar de Spring Boot (stdout)
- **Persistencia**: Solo memoria (colecciones Java). Sin base de datos. Sin JPA.

---

## ESTRUCTURA DEL PROYECTO

```text
aws-academy-foundations-api/
├── .github
│   └── copilot-instructions.md          (este archivo)
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── example
│   │   │           └── awsacademy
│   │   │               ├── AwsAcademyApplication.java   (entry point Spring Boot)
│   │   │               ├── alumno
│   │   │               │   ├── model
│   │   │               │   │   └── Alumno.java
│   │   │               │   ├── controller
│   │   │               │   │   └── AlumnoController.java
│   │   │               │   ├── service
│   │   │               │   │   ├── AlumnoService.java
│   │   │               │   │   └── AlumnoServiceImpl.java
│   │   │               │   └── repository
│   │   │               │       ├── AlumnoRepository.java
│   │   │               │       └── InMemoryAlumnoRepository.java
│   │   │               ├── profesor
│   │   │               │   ├── model
│   │   │               │   │   └── Profesor.java
│   │   │               │   ├── controller
│   │   │               │   │   └── ProfesorController.java
│   │   │               │   ├── service
│   │   │               │   │   ├── ProfesorService.java
│   │   │               │   │   └── ProfesorServiceImpl.java
│   │   │               │   └── repository
│   │   │               │       ├── ProfesorRepository.java
│   │   │               │       └── InMemoryProfesorRepository.java
│   │   │               └── common
│   │   │                   ├── exception
│   │   │                   │   ├── NotFoundException.java
│   │   │                   │   ├── ApiError.java
│   │   │                   │   └── GlobalExceptionHandler.java
│   │   │                   └── util
│   │   │                       └── IdGenerator.java
│   │   └── resources
│   │       └── application.properties
│   └── test
│       └── java
│           └── com
│               └── example
│                   └── awsacademy
│                       ├── alumno
│                       └── profesor
```

---

## DOMINIO: MODELOS

### `Alumno`

Ubicación: `com.example.awsacademy.alumno.model.Alumno`

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Alumno {
    private Long id;
    private String nombres;
    private String apellidos;
    private String matricula;
    private Double promedio;
}
```

Reglas de campos:

- `id`: generado en servidor; ignorar valor entrante en `POST`.
- `nombres`: obligatorio, no vacío.
- `apellidos`: obligatorio, no vacío.
- `matricula`: obligatoria, no vacía; debe tratarse como única (no duplicar).
- `promedio`: obligatorio, numérico (`Double`), no nulo.

### `Profesor`

Ubicación: `com.example.awsacademy.profesor.model.Profesor`

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Profesor {
    private Long id;
    private String numeroEmpleado;
    private String nombres;
    private String apellidos;
    private Integer horasClase;
}
```

Reglas de campos:

- `id`: generado en servidor; ignorar valor entrante en `POST`.
- `numeroEmpleado`: obligatorio, no vacío; debe tratarse como único.
- `nombres`: obligatorio, no vacío.
- `apellidos`: obligatorio, no vacío.
- `horasClase`: obligatorio, numérico (`Integer`), no nulo.

---

## VALIDACIÓN DE ENTRADAS

Usar Jakarta Validation en DTOs de entrada. Los modelos de dominio (`Alumno`, `Profesor`) pueden reutilizarse como DTOs simples para esta práctica.

Reglas generales:

- `@NotBlank` para `String` obligatorios.
- `@NotNull` para numéricos obligatorios.
- `promedio`: `@NotNull` y puede añadirse validación de rango si se requiere (opcional).
- `horasClase`: `@NotNull` y opcionalmente `@Min(0)`.

En errores de validación, responder:

- HTTP 400.
- JSON con estructura `ApiError` (ver sección de errores).

---

## CAPA REPOSITORY (MEMORIA)

### Interfaz `AlumnoRepository`

Ubicación: `com.example.awsacademy.alumno.repository.AlumnoRepository`

```java
public interface AlumnoRepository {
    List<Alumno> findAll();
    Optional<Alumno> findById(Long id);
    Alumno save(Alumno alumno);
    void deleteById(Long id);
    boolean existsById(Long id);
    boolean existsByMatricula(String matricula);
}
```

### Implementación `InMemoryAlumnoRepository`

Ubicación: `com.example.awsacademy.alumno.repository.InMemoryAlumnoRepository`

Características:

- Anotación `@Repository`.
- Almacenamiento interno en `Map<Long, Alumno>` o `ConcurrentHashMap<Long, Alumno>`.
- Métodos:
    - `findAll`: retorna copia inmutable (`List.copyOf(map.values())`).
    - `findById`: usa `Optional.ofNullable(map.get(id))`.
    - `save`:
        - Si `alumno.getId() == null` → asignar nuevo id usando `IdGenerator`.
        - Insertar/actualizar en el `Map`.
    - `deleteById`: eliminar clave si existe.
    - `existsById`: verificar `map.containsKey(id)`.
    - `existsByMatricula`: buscar por campo `matricula` en values.

### Interfaz `ProfesorRepository`

Ubicación: `com.example.awsacademy.profesor.repository.ProfesorRepository`

```java
public interface ProfesorRepository {
    List<Profesor> findAll();
    Optional<Profesor> findById(Long id);
    Profesor save(Profesor profesor);
    void deleteById(Long id);
    boolean existsById(Long id);
    boolean existsByNumeroEmpleado(String numeroEmpleado);
}
```

### Implementación `InMemoryProfesorRepository`

Igual patrón que alumnos, con `Map<Long, Profesor>` y validación de unicidad por `numeroEmpleado`.

---

## CAPA SERVICE

### `AlumnoService`

Ubicación: `com.example.awsacademy.alumno.service.AlumnoService`

```java
public interface AlumnoService {
    List<Alumno> getAll();
    Alumno getById(Long id);
    Alumno create(Alumno alumno);
    Alumno update(Long id, Alumno alumno);
    void delete(Long id);
}
```

### `AlumnoServiceImpl`

Ubicación: `com.example.awsacademy.alumno.service.AlumnoServiceImpl`

Responsabilidades:

- Anotación `@Service`.
- Inyectar `AlumnoRepository`.
- Métodos:

    - `getAll`:
        - Retorna lista de todos los alumnos.

    - `getById`:
        - Usar repositorio.
        - Si no existe → lanzar `NotFoundException` con mensaje claro.

    - `create`:
        - Ignorar `id` entrante; dejar `null`.
        - Validar unicidad de `matricula` con `existsByMatricula`; si ya existe, lanzar excepción (puede ser `IllegalArgumentException` o custom).
        - Llamar `repository.save`.

    - `update`:
        - Verificar existencia por `id`.
        - Cargar entidad existente.
        - Actualizar campos mutables (`nombres`, `apellidos`, `matricula`, `promedio`).
        - Validar que la nueva `matricula` no duplique otra distinta de este `id`.
        - Guardar usando `save`.

    - `delete`:
        - Verificar existencia por `id`; si no existe → `NotFoundException`.
        - Ejecutar `deleteById`.

### `ProfesorService` y `ProfesorServiceImpl`

Misma estructura que `AlumnoService`, ajustando campos:

```java
public interface ProfesorService {
    List<Profesor> getAll();
    Profesor getById(Long id);
    Profesor create(Profesor profesor);
    Profesor update(Long id, Profesor profesor);
    void delete(Long id);
}
```

Reglas extra:

- Validar unicidad de `numeroEmpleado`.
- `horasClase` debe ser no nulo; se puede validar mínimo 0.

---

## CAPA CONTROLLER

### `AlumnoController`

Ubicación: `com.example.awsacademy.alumno.controller.AlumnoController`

Anotaciones:

- `@RestController`
- `@RequestMapping("/alumnos")`

Endpoints:

1. `GET /alumnos`

    - Método: `getAllAlumnos()`.
    - Llama `AlumnoService.getAll()`.
    - Respuesta:
        - `200 OK`.
        - Body: `List<Alumno>` (array JSON, `[]` si vacío).

2. `GET /alumnos/{id}`

    - Método: `getAlumnoById(Long id)`.
    - Llama `AlumnoService.getById(id)`.
    - Respuestas:
        - `200 OK` si existe.
        - `404 Not Found` si no existe (vía `NotFoundException`).

3. `POST /alumnos`

    - Método: `createAlumno(@Valid @RequestBody Alumno alumno)`.
    - Ignorar `alumno.id`.
    - Llama `AlumnoService.create(alumno)`.
    - Respuesta:
        - `201 Created`.
        - Body: objeto `Alumno` creado.
        - Header `Location` opcional: `/alumnos/{id}`.

4. `PUT /alumnos/{id}`

    - Método: `updateAlumno(Long id, @Valid @RequestBody Alumno alumno)`.
    - Llama `AlumnoService.update(id, alumno)`.
    - Respuesta:
        - `200 OK` con objeto actualizado.
        - `404 Not Found` si no existe.

5. `DELETE /alumnos/{id}`

    - Método: `deleteAlumno(Long id)`.
    - Llama `AlumnoService.delete(id)`.
    - Respuesta:
        - `200 OK` o `204 No Content` (usar `200` para alinearse con requerimientos).
        - `404 Not Found` si no existe.

### `ProfesorController`

Ubicación: `com.example.awsacademy.profesor.controller.ProfesorController`

Mismos patrones, cambiando ruta base a `/profesores` y usando `ProfesorService`.

Endpoints:

1. `GET /profesores`
2. `GET /profesores/{id}`
3. `POST /profesores`
4. `PUT /profesores/{id}`
5. `DELETE /profesores/{id}`

---

## MANEJO DE ERRORES

### `NotFoundException`

Ubicación: `com.example.awsacademy.common.exception.NotFoundException`

```java
public class NotFoundException extends RuntimeException {
    public NotFoundException(String message) {
        super(message);
    }
}
```

### `ApiError`

Ubicación: `com.example.awsacademy.common.exception.ApiError`

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ApiError {
    private int status;
    private String message;
    private String path;
    private Instant timestamp;
}
```

### `GlobalExceptionHandler`

Ubicación: `com.example.awsacademy.common.exception.GlobalExceptionHandler`

Usar `@RestControllerAdvice`.

Manejar:

1. `NotFoundException`:

    - HTTP 404
    - Body `ApiError`:
        - `status`: 404
        - `message`: mensaje de excepción
        - `path`: tomar de `HttpServletRequest.getRequestURI()`
        - `timestamp`: `Instant.now()`

2. `MethodArgumentNotValidException` (errores de validación):

    - HTTP 400
    - `message`: mensaje genérico o concatenación de errores de campo.

3. `IllegalArgumentException` u otras validaciones de negocio:

    - HTTP 400
    - `message`: mensaje de excepción.

4. `Exception` genérica:

    - HTTP 500
    - `message`: "Internal server error" (no exponer detalle técnico).

---

## CÓDIGOS HTTP A UTILIZAR

- **200 OK**
    - Lecturas exitosas (`GET` por id o lista).
    - Eliminaciones exitosas (`DELETE`) si se decide retornar body vacío.

- **201 Created**
    - Creación exitosa en `POST /alumnos` y `POST /profesores`.

- **400 Bad Request**
    - Error de validación de campos (`@Valid`).
    - Regla de unicidad violada (`matricula` o `numeroEmpleado`).

- **404 Not Found**
    - Entidad no encontrada para `GET /{id}`, `PUT /{id}`, `DELETE /{id}`.

- **500 Internal Server Error**
    - Errores no controlados (excepciones no mapeadas).

---

## GENERACIÓN DE IDENTIFICADORES

### `IdGenerator`

Ubicación: `com.example.awsacademy.common.util.IdGenerator`

```java
public final class IdGenerator {

    private static final AtomicLong SEQUENCE = new AtomicLong(0L);

    private IdGenerator() {
    }

    public static Long nextId() {
        return SEQUENCE.incrementAndGet();
    }
}
```

Uso:

- `InMemoryAlumnoRepository` y `InMemoryProfesorRepository` llaman a `IdGenerator.nextId()` cuando `id` es `null` en `save`.

---

## CONFIGURACIÓN DE APLICACIÓN

`application.properties` mínimo:

```properties
server.port=8080
spring.main.banner-mode=off
logging.level.root=INFO
```

No agregar configuración de base de datos (no se usa). No agregar starters de JPA.

---

## COMPORTAMIENTO ESPERADO PARA PRUEBAS AUTOMÁTICAS

### Casos base

1. `GET /alumnos` y `GET /profesores` en estado inicial:
    - Responder `200 OK`.
    - Body: `[]`.

2. `POST /alumnos` con JSON válido:
    - Crear entidad en memoria.
    - Responder `201 Created` con objeto completo incluyendo `id`.

3. `GET /alumnos/{id}` existente:
    - Responder `200 OK` con objeto correspondiente.

4. `GET /alumnos/{id}` inexistente:
    - Responder `404 Not Found` con body `ApiError`.

5. `PUT /alumnos/{id}` existente:
    - Actualizar entidad.
    - Responder `200 OK`.

6. `DELETE /alumnos/{id}` existente:
    - Eliminar entidad.
    - Responder `200 OK` con body vacío o mensaje simple.

Mismo patrón para `profesores`.

---

## RESTRICCIONES Y LINEAMIENTOS PARA IA

1. **No introducir base de datos**:
    - Prohibido agregar dependencias de JPA o drivers de BD.
    - Toda persistencia debe realizarse en estructuras en memoria (`Map`, `List`, etc.).

2. **Mantener interfaces de repositorio y servicio**:
    - No acoplar controladores a implementación concreta.
    - Siempre inyectar interfaces (`AlumnoService`, `ProfesorService`, `AlumnoRepository`, `ProfesorRepository`).

3. **Código y nombres en inglés**:
    - Clases, métodos, variables y comentarios en código deben estar en inglés.
    - Textos de mensajes de error pueden ir en inglés o español, pero ser consistentes.

4. **Sin dependencias adicionales sin permiso**:
    - No agregar nuevas dependencias Maven sin instrucción explícita del usuario.

5. **Respuestas HTTP siempre JSON**:
    - No retornar vistas HTML.
    - Usar `@RestController` y dejar que Spring gestione `application/json`.

6. **Formato de fechas**:
    - `ApiError.timestamp`: usar `Instant` y serialización por defecto de Jackson.

---

## MODO DE COLABORACIÓN

- Foco en pocos archivos por vez; no modificar todo el proyecto en una sola respuesta.
- Las respuestas en el chat deben ser concisas; el código puede ser completo.
- Si hay conflicto entre este documento y el código actual, este documento prevalece.
- La IA debe seguir esta guía para generar controladores, servicios, repositorios y modelos coherentes con los requisitos de la materia.
- Todo el código debe estar completamente en inglés, al igual qu e los mensajes de error, comentarios, mensaje de commits, a menos que se indique lo contrario.
- No asumir requisitos no especificados en este documento.
- Priorizar claridad, mantenibilidad y buenas prácticas de desarrollo en Java y Spring Boot.
- Al generar código, incluir solo el fragmento solicitado, sin explicaciones adicionales, a menos que se solicite.
- Seguir estrictamente las convenciones de nomenclatura y estructura de paquetes indicadas en este documento.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DanielVallado)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/DanielVallado)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
