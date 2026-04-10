# Copilot Instructions: mvintage

## ⚠️ COMANDOS OBLIGATORIOS PARA EJECUTAR SERVICIOS ⚠️

### � ANTES DE INICIAR - Verificar puertos:
```powershell
# Backend (puerto 8000):
netstat -an | findstr :8000

# Frontend (puerto 5173):
netstat -an | findstr :5173
```
**Si hay salida, el servidor YA ESTÁ CORRIENDO - NO iniciar uno nuevo**

### �🚀 BACKEND - Secuencia OBLIGATORIA:
```powershell
cd c:\Repositorio\mvintage\backend
; .\.venv\Scripts\Activate.ps1 ;
python -m uvicorn app.main:app --reload
```
**NUNCA ejecutar uvicorn sin estos pasos previos**

### 🎨 FRONTEND - Secuencia OBLIGATORIA:
```powershell
cd c:\Repositorio\mvintage\frontend ;
npm run dev
```

## Reglas para agentes Copilot

1. **PROTOCOLO OBLIGATORIO DE ARRANQUE:**
   - **ANTES DE INICIAR CUALQUIER SERVIDOR:** Verificar si ya está corriendo con `netstat -an | findstr :8000` (backend) o `netstat -an | findstr :5173` (frontend)
   - **Backend:** SIEMPRE ejecutar estos 3 comandos en orden:
     1. `cd c:\Repositorio\mvintage\backend`
     2. `& .\.venv\Scripts\Activate.ps1`
     3. `python -m uvicorn app.main:app --reload`
   - **Frontend:** SIEMPRE ejecutar estos 2 comandos en orden:
     1. `cd c:\Repositorio\mvintage\frontend`
     2. `npm run dev`
   - **Tests Backend:** SIEMPRE ejecutar estos 3 comandos en orden:
     1. `cd c:\Repositorio\mvintage\backend`
     2. `& .\.venv\Scripts\Activate.ps1`
     3. `python -m pytest`
   - NUNCA ejecutar servidores sin seguir estos pasos exactos.
   - SI YA HAY UN SERVIDOR CORRIENDO, informar al usuario y NO iniciar uno nuevo.

2. Antes de actuar, leer en este orden: `README.md` → `docs/planning.md` (o issues) → `docs/tech-stack.md` → `docs/structure.md`.
3. Si falta alguno, crearlo con un esqueleto mínimo y confirmar.
4. Solo implementar/editar una feature o endpoint si está en `planning.md` (o issues); si no, agregarlo primero.
5. Tras cambios relevantes (feature/endpoint/config), **actualizar README** (sección API: ruta, método, ejemplos de request/response, errores).
6. **Frontend:** TypeScript estricto; usando tailwinds css para maquetado, lint con ESLint/Prettier; tests (Vitest) cuando corresponda.
7. **Backend:** Validación con SQLModel (antes Pydantic); tests con pytest (y/o requests a la API); seguir convenciones.
8. **Limpieza de código:** identificar y eliminar imports, funciones, tipos, módulos y archivos no usados.
   - Comprobar referencias en todo el workspace antes de borrar.
   - Si algo es parte de una API pública, marcar deprecado y planificar su remoción en `docs/planning.md`.
   - Si hay dudas, abrir PR explicando qué se elimina y por qué; actualizar `README`/`planning` si corresponde.
9. Cuando completes una tarea de `docs/planning.md`, muévela de la sección 'Tareas' a 'Hecho' y agrega la fecha de finalización. Ejemplo:
   - Tarea completada [29/07/2025]

---

## 1. Contexto y Arquitectura

- Monorepo con dos carpetas principales: `frontend/` (Vue 3 + TypeScript + Vite) y `backend/` (FastAPI + Python).
- Base de datos principal: PostgreSQL (Railway), pero puede usarse SQLite en desarrollo.
- Storage de imágenes: Firebase Storage.
- Autenticación: Firebase Auth + JWT (backend).
- Despliegue: Frontend en Firebase Hosting, backend en Railway.

## 2. Flujos de desarrollo

- **Backend:**  
  - Modelos de datos con SQLModel (antes SQLAlchemy + Pydantic).
  - Migraciones con Alembic (`alembic revision --autogenerate`, `alembic upgrade head`).
  - API REST estructurada en `/backend/app/routes/` (ejemplo: `products.py`, `customers.py`).
  - Configuración en `.env` y `app/core/config.py`.
  - Tests con pytest y requests.
  - **ARRANQUE OBLIGATORIO DEL BACKEND:**  
    ```powershell
    # SIEMPRE ejecutar en este orden exacto:
    cd c:\Repositorio\mvintage\backend
    & .\.venv\Scripts\Activate.ps1
    python -m uvicorn app.main:app --reload
    ```
    **NUNCA ejecutar uvicorn sin activar el .venv primero y estar en el directorio backend**

- **Frontend:**  
  - SPA con Vue 3, Vite y TypeScript estricto.
  - Maquetado con Tailwind CSS.
  - Estado con Pinia (opcional).
  - Linting: ESLint + Prettier.
  - Tests: Vitest + Testing Library.
  - **ARRANQUE OBLIGATORIO DEL FRONTEND:**  
    ```powershell
    # SIEMPRE ejecutar en este orden exacto:
    cd c:\Repositorio\mvintage\frontend
    npm run dev
    ```

## 3. Convenciones y patrones

- Solo implementar features/endpoints si están en `docs/planning.md` o issues.
- Actualizar `README.md` tras cambios relevantes (API, endpoints, ejemplos).
- Limpiar código muerto siguiendo `docs/structure.md` y `.github/instructions/04-dead-code-policy.instructions.md`.
- Variables sensibles y configuración en `.env.example` (ambos servicios).
- Endpoints RESTful bajo `/api/v1/` (ejemplo: `/api/v1/products`).
- Validación de datos y serialización con SQLModel (antes Pydantic).
- Migraciones y cambios de modelos deben reflejarse en Alembic y en la base de datos.
- Tests y CI/CD: pytest (backend), Vitest (frontend), GitHub Actions para lint/test/build/deploy.

## 4. Integraciones y dependencias

- **Firebase:**  
  - Hosting, Storage y Auth para frontend y backend.
- **Railway:**  
  - Despliegue backend y base de datos PostgreSQL.
- **Docker:**  
  - Archivos Dockerfile por servicio, docker-compose para entorno local.

## 5. Ejemplos clave

- Modelos en `backend/app/models/*.py` usando SQLModel.
- Rutas en `backend/app/routes/*.py` siguiendo el patrón CRUD.
- Configuración de sesión DB en `backend/app/db/session.py`.
- Frontend principal en `frontend/src/main.ts` y componentes en `frontend/src/components/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonatha1992)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jonatha1992)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
