# Fleet Management Application - Copilot Instructions

## Proyecto de Gestión de Flotas

Este es un proyecto moderno de gestión de flotas implementado con las mejores prácticas de ingeniería de software.

### Arquitectura
- **Backend**: Node.js + TypeScript + Express con arquitectura limpia
- **Frontend**: React + TypeScript + Vite
- **Base de datos**: PostgreSQL
- **Contenedorización**: Docker + Docker Compose
- **Testing**: Jest + React Testing Library
- **Patrones**: Repository, Factory, Dependency Injection, SOLID

### Estado del Proyecto
- [x] Crear archivo copilot-instructions.md
- [x] Scaffolding del proyecto
- [x] Personalizar el proyecto
- [x] Instalar dependencias (requiere npm en el sistema)
- [x] Documentación completa

### Para instalar las dependencias

Ejecuta los siguientes comandos:

```bash
# Instalar Node.js si no está instalado
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Backend
cd backend && npm install

# Frontend  
cd ../frontend && npm install
```

### Para iniciar el proyecto

**Opción 1 - Con Docker (recomendado):**
```bash
docker-compose up -d
```

**Opción 2 - Sin Docker:**
```bash
# Terminal 1 - Backend
cd backend && npm run dev

# Terminal 2 - Frontend
cd frontend && npm run dev
```

Consulta INSTALL.md para instrucciones detalladas.

---
> Source: [efrenbohorquez/flota-veh-culos-con-notificaci-n](https://github.com/efrenbohorquez/flota-veh-culos-con-notificaci-n) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-25 -->
