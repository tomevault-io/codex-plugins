# .github/copilot-instructions.md

## Instrucciones del Proyecto Google Search AI

### Descripción General
Sistema integral de búsqueda para empresas que integra:
- Backend Flask con Python
- Base de datos PostgreSQL
- Google Search API
- Interfaz web tipo chat

### Estructura del Proyecto

```
backend/          - Aplicación Flask
  - app.py        - Servidor principal
  - models.py     - Modelos de BD
  - config.py     - Configuración
  - database_service.py  - Operaciones BD
  - google_search.py     - Integración Google

frontend/         - Interfaz web
  - index.html    - HTML del chat
  - styles.css    - Estilos
  - script.js     - Lógica del cliente

database/
  - schema.sql    - Esquema PostgreSQL

config/
  - .env.example  - Variables de entorno
```

### Configuración Requerida

1. **PostgreSQL**: Crear base de datos `google_search_db`
2. **Google API**: Obtener API Key y Search Engine ID
3. **.env**: Copiar `.env.example` a `.env` y completar credenciales

### Comandos Principales

```bash
# Backend
cd backend
python app.py

# Cargar documentos de ejemplo
python load_documents.py

# Frontend
# Abrir frontend/index.html en navegador
```

### Endpoints API

- `POST /api/search` - Búsqueda unificada
- `GET/POST /api/documents` - Gestionar documentos
- `GET /api/search-history/<user_id>` - Historial
- `GET /api/stats` - Estadísticas

### Notas de Desarrollo

- CORS habilitado para desarrollo
- Búsqueda local + Google Search API
- Historial de búsquedas por usuario
- Full-text search con PostgreSQL

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivofran)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ivofran)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
