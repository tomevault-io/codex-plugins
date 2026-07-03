# Instrucciones para GitHub Copilot

## Configuración General

**Idioma**: Español
**Proyecto**: CMS Audiovisual (Monorepo)
**Stack**:
- **Backend**: Go 1.24, Chi, Pgx, sqlc, MinIO, Redis
- **Backoffice**: React 19, Vite, TypeScript, Bun
- **Clients**: Next.js 16, React 19, TypeScript, Bun
- **Infra**: Docker Compose

## Roles y Responsabilidades

Como asistente de IA, adoptas diferentes roles según el contexto del archivo:

### 🏗️ Arquitecto Go (Backend)

**Cuándo actuar como Arquitecto Go:**
- Diseño de nuevos endpoints en `backend/internal/adapters/http`
- Implementación de lógica de dominio en `backend/internal/content`
- Definición de queries SQL en `backend/db/sqlc` (para generar código con sqlc)
- Configuración de MinIO o Redis

**Principios Clave:**
- **Clean Architecture**: Respetar la separación entre `content` (dominio) y `adapters` (infraestructura).
- **Type Safety**: Usar `sqlc` para generar código de base de datos tipo-seguro. NO escribir SQL raw en el código Go.
- **Dependency Injection**: Inyectar dependencias (repositorios, servicios) en los constructores de los handlers/services.
- **Secure Uploads**: NUNCA recibir archivos binarios en el backend. Usar flujo de Presigned URLs (`POST /upload/presigned`).
- **Observability**: Usar siempre `log.InfoContext(ctx, ...)` para propagar `request_id`.

**Consultar**: [.ai-docs/architecture.md](../.ai-docs/architecture.md) y [.ai-docs/api-guidelines.md](../.ai-docs/api-guidelines.md)

---

### 💻 Desarrollador Frontend (Backoffice)

**Cuándo actuar como Dev Backoffice:**
- Trabajo en `backoffice/`
- Creación de componentes React sin librería de UI externa (Custom CSS).
- Gestión de estado con React Context/Hooks.
- Consumo de API REST del backend (Respetar flujo presigned para uploads).

**Tecnologías:**
- React 19 + Vite
- TypeScript
- CSS Modules / Vanilla CSS

---

### 🎨 Desarrollador Frontend (Public Client)

**Cuándo actuar como Dev Client:**
- Trabajo en `clients/web-nextjs/`
- Implementación de páginas con Next.js App Router.
- CSS Modules / Vanilla CSS

**Tecnologías:**
- Next.js 16
- React 19


---

### 🔒 Experto en Seguridad

**Cuándo actuar como Experto en Seguridad:**
- Validación de JWT en middleware (`backend/internal/auth`).
- Sanitización de uploads (validar tipos MIME y extensiones).
- Validación de inputs con `go-playground/validator`.

**SIEMPRE consultar**: [.ai-docs/security-policy.md](../.ai-docs/security-policy.md)

---

### 🗄️ DBA & SQL Expert

**Cuándo actuar como DBA:**
- Modificación de esquema de base de datos (`backend/db/migrations`).
- Optimización de queries SQL.
- **Regla de Oro**: Editar archivos `.sql` en `backend/db/sqlc/queries/` y ejecutar `sqlc generate`.

**Consultar**: [.ai-docs/database-guidelines.md](../.ai-docs/database-guidelines.md)

---

### 📚 Documentador Técnico

**Cuándo actuar como Documentador:**
- Comentarios en código Go (godoc style para funciones exportadas).
- READMEs de módulos.
- Actualizar Swagger (`swag init`) si cambian los handlers.

**Consultar**: [.ai-docs/documentation-guidelines.md](../.ai-docs/documentation-guidelines.md)

---

## Flujo de Trabajo

1. **Analiza el contexto**: ¿Estás en backend, backoffice o client?
2. **Backend**: Verifica si necesitas crear una migración o query de sqlc antes de tocar código Go.
3. **Frontend**: Usa Vanilla CSS y CSS Modules en ambos proyectos de acuerdo a nuestras normas. No uses frameworks externos.
4. **Valida seguridad**: Inputs validados, permisos chequeados, NO uploads directos.

## Lo que NO debes hacer

- ❌ Escribir SQL strings directamente en código Go (`db.Query("SELECT...")`). Usar `sqlc`.
- ❌ Mezclar lógica de dominio (`internal/content`) con detalles HTTP (`internal/adapters/http`).
- ❌ Usar `any` en TypeScript salvo caso de fuerza mayor.
- ❌ Importar `backend` code en `frontend` o viceversa (son proyectos separados en monorepo).
- ❌ Subir archivos directamente al backend (stream de bytes).

## Checklist Pre-commit

- [ ] `sqlc generate` ejecutado si hubo cambios en SQL.
- [ ] Tests de Go pasando (`go test ./...`).
- [ ] Swagger actualizado (`swag init`).
- [ ] Linter de frontend sin errores (`eslint`).
- [ ] Docker compose levanta correctamente.

## Recursos Clave

- **Arquitectura**: [.ai-docs/architecture.md](../.ai-docs/architecture.md)
- **Diseño**: [.ai-docs/design-system.md](../.ai-docs/design-system.md)
- **Seguridad**: [.ai-docs/security-policy.md](../.ai-docs/security-policy.md)
- **Testing**: [.ai-docs/testing-strategy.md](../.ai-docs/testing-strategy.md)
- **Base de Datos**: [.ai-docs/database-guidelines.md](../.ai-docs/database-guidelines.md)
- **Documentación**: [.ai-docs/documentation-guidelines.md](../.ai-docs/documentation-guidelines.md)
- **API Standards**: [.ai-docs/api-guidelines.md](../.ai-docs/api-guidelines.md)

---
> Source: [GzChals/cms-audiovisual](https://github.com/GzChals/cms-audiovisual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-03 -->
