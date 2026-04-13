<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Sistema RAG para Atención al Cliente

Este proyecto es un sistema RAG (Retrieval-Augmented Generation) completo para atención al cliente construido con Node.js.

## Arquitectura del Proyecto

- **Backend**: Node.js con Express.js
- **Base de datos vectorial**: Pinecone para almacenar embeddings
- **IA**: OpenAI GPT para generación de respuestas y embeddings
- **Frontend**: HTML, CSS (Tailwind), JavaScript vanilla
- **WebSockets**: Socket.io para chat en tiempo real
- **Autenticación**: JWT tokens

## Estructura de Directorios

```
src/
├── config/         # Configuraciones (database, etc.)
├── controllers/    # Controladores (sin usar actualmente)
├── middleware/     # Middleware (auth, rate limiting, error handling)
├── models/         # Modelos de datos (sin usar actualmente)
├── routes/         # Rutas de la API (auth, chat, documents, admin)
├── services/       # Servicios principales (chat, documents, openai, socket)
└── utils/          # Utilidades (logger)
```

## Funcionalidades Principales

1. **Procesamiento de Documentos**: Subida y procesamiento de PDF, TXT, MD, DOCX
2. **Chat Inteligente**: Sistema de chat con IA que busca en documentos
3. **Administración**: Panel de admin para gestión de usuarios y sistema
4. **Autenticación**: Sistema de login con roles (user/admin)
5. **WebSockets**: Chat en tiempo real con Socket.io
6. **Rate Limiting**: Protección contra abuso de API

## Variables de Entorno Requeridas

- `OPENAI_API_KEY`: Clave de API de OpenAI
- `PINECONE_API_KEY`: Clave de API de Pinecone
- `JWT_SECRET`: Secreto para tokens JWT

## Instrucciones para Copilot

- Este es un proyecto de Node.js con arquitectura RESTful
- Usa async/await para operaciones asíncronas
- Implementa logging detallado con Winston
- Sigue patrones de error handling centralizados
- Usa validación de entrada con express-validator
- Implementa rate limiting para protección
- Usa Socket.io para comunicación en tiempo real

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabrbl)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gabrbl)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
