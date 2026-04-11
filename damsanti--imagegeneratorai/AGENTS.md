# Copilot Instructions - Image Generator AI

## Proyecto
**Nombre**: Image Generator AI - Generador de Imágenes Local con IA
**Stack**: Next.js 14 + React 18 + TypeScript + Python FastAPI + Stable Diffusion
**Objetivo**: Interfaz web profesional para generar imágenes usando GPU local (RTX 1070+)

## Arquitectura General

```
Frontend (Next.js) ←→ Backend API (FastAPI) ←→ AI Models (Ollama/ComfyUI)
```

## Convenciones de Código

### Frontend (TypeScript/React)
- Usar componentes funcionales con hooks
- Typed props con interfaces
- useClient en componentes interactivos
- Paths relativo con alias @/

### Backend (Python)
- FastAPI con async
- Pydantic models para validación
- Type hints en todas las funciones
- CORS habilitado

## Stack Tecnológico

**Frontend**:
- Next.js 14 (App Router)
- React 18
- TypeScript 5.3
- Tailwind CSS 3.4
- Zustand (estado global)
- React Hook Form (formularios)

**Backend**:
- FastAPI (async API)
- Python 3.10+
- Diffusers (generación IA)
- PyTorch con CUDA support
- Ollama (LLM local)

**IA**:
- Stable Diffusion v1.5 / XL
- FLUX.1 (opcional)
- LoRA personalizado (opcional)

## Características Principales

✅ Generación local (sin APIs cloud)
✅ Control total de parámetros (steps, guidance, sampler, seed)
✅ Vista previa en tiempo real
✅ Historial de generaciones
✅ Galería con descargas
✅ Batch processing (múltiples imágenes)
✅ Escalable para H200 droplet

## Carpetas Clave

```
app/              - Rutas Next.js y layout
components/       - Componentes React reutilizables
backend/          - API FastAPI
types/            - Tipos TypeScript compartidos
hooks/            - Custom hooks React
lib/              - Utilidades compartidas
```

## Flujo de Desarrollo

1. **Agregar Parámetro**: Frontend (GeneratorForm) → Backend (GenerateRequest) → Procesamiento
2. **Nuevo Modelo**: Agregar endpoint backend → Selector frontend → Lógica generación
3. **Optimización**: Ajustar config según GPU (RTX 1070 vs H200)

## Dependencias Críticas

- `pytorch` - Motor de IA
- `diffusers` - Pipelines Stable Diffusion
- `fastapi` - API backend
- `next` - Framework frontend
- `tailwindcss` - Estilos

## Variables de Entorno

```
NEXT_PUBLIC_API_URL=http://localhost:8000
DEVICE=cuda
MODEL_NAME=stable-diffusion-v1-5
OUTPUT_DIR=./generated_images
```

## Comandos Importantes

```
npm install              - Instalar dependencias
npm run dev             - Iniciar frontend
npm run backend         - Iniciar backend
npm run dev:all         - Ambos simultáneamente
npm run build           - Build producción
npm run docker:build    - Crear imagen Docker
```

## Performance

**RTX 1070 (8GB)**:
- Resolución: 512x512 óptima
- Steps: 12-20 recomendado
- Dtype: float16 (half precision)
- Tiempo: ~45 seg por imagen

**H200 (141GB)**:
- Resolución: 1024x1024
- Steps: 50 máximo
- Dtype: float32
- Tiempo: ~10 seg por imagen

## Deploy

Local: `npm run dev:all`
Docker: `npm run docker:build && docker run --gpus all image-generator-ai`
H200: `docker-compose up --build` (con NVIDIA Docker)

## Debugging

- Frontend logs: Developer Tools (F12)
- Backend logs: Terminal donde corre uvicorn
- GPU: `nvidia-smi -l 1` (tiempo real)

## Recursos

- Documentación: README.md, SETUP.md, DEVELOPMENT.md
- API Docs: http://localhost:8000/docs
- Frontend: http://localhost:3000

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DAMSanti)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/DAMSanti)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
