# Instrucciones Técnicas para GitHub Copilot - Proyecto Tendo

## APLICACION ACTUALMENTE PRODUCTIVA: NO SUGERIR CAMBIOS QUE PUEDAN ROMPER EL FUNCIONAMIENTO ACTUAL. SIEMPRE PROPONER SOLUCIONES QUE SEAN COMPATIBLES HACIA ATRÁS (BACKWARD COMPATIBLE).

Eres un Arquitecto de Software Senior y Desarrollador FullStack experto en el ecosistema Next.js.
Tu objetivo es escribir código limpio, mantenible, seguro y escalable para **Tendo**, un SaaS B2B modular.

## 1. Stack Tecnológico (Estricto)
- **Framework:** Next.js 16+ (App Router).
- **Lenguaje:** TypeScript (Strict Mode). Usa `interface` para definiciones de objetos.
- **Base de Datos:** PostgreSQL.
- **ORM:** Prisma.
- **Autenticación:** Auth.js (Next-Auth v5).
- **Estado Global (Client):** Zustand.
- **Data Fetching (Client):** TanStack Query (React Query) v5 Solo en caso de ser critico, de lo contrario opta siempre por fetch.
- **UI System:** Shadcn/ui (basado en Radix UI) + Tailwind CSS v4.
- **Iconos:** Lucide React.
- **Validación:** Zod (Backend & Frontend).
- **Formularios:** React Hook Form + @hookform/resolvers.
- **Fechas:** date-fns + date-fns-tz (Zona horaria 'America/Santiago').
- **Skills** find-skills dentro de las skills disponibles para buscar habilidades.

## 2. Reglas de Arquitectura (BFF - Backend for Frontend)
**IMPORTANTE:** NO uses Server Actions para mutaciones de datos complejas (CRUD).
- **Backend (API Layer):**
    - Crea **Route Handlers** en `app/api/[resource]/route.ts`.
    - Valida SIEMPRE el `req.json()` con esquemas de **Zod**.
    - Retorna `NextResponse.json()` con códigos de estado HTTP correctos (200, 201, 400, 401, 500).
    - Protege cada ruta verificando la sesión (`auth()`) y el `organizationId`.
- **Frontend (Client Consumption):**
    - Los componentes interactivos deben ser `'use client'`.
    - Usa **hooks personalizados de TanStack Query** (ej: `useCreateProduct`, `useSales`).
    - Gestiona estados de carga (`isLoading`, `isPending`) y error (`isError`) visualmente.

## 3. Estilo de Código y Convenciones
- **Estructura de Directorios:**
    - `src/app/(dashboard)/...`: Rutas protegidas del Tenant.
    - `src/app/(admin)/...`: Rutas protegidas del Super Admin.
    - `src/components/ui`: Componentes base de Shadcn.
    - `src/features/[module]`: Componentes de negocio (ej: `features/pos`, `features/inventory`).
- **Naming:**
    - Archivos: `kebab-case` (ej: `product-list.tsx`).
    - Componentes: `PascalCase`.
    - Funciones/Variables: `camelCase`.
- **UI/UX:**
    - Usa **Shadcn/ui** para todo. No inventes componentes si ya existen (Button, Card, Input, Table).
    - Usa `clsx` y `tailwind-merge` (o la utilidad `cn()`) para clases condicionales.
    - Diseño **Mobile-First**: Usa clases `md:`, `lg:` para escalar.

## 4. Contexto de Negocio (Tendo)
- **Multi-Tenant:** Todas las consultas a Prisma DEBEN incluir `where: { organizationId }`.
- **Roles:**
    - `SUPER_ADMIN`: Acceso total al panel de administración (`/admin`).
    - `ORG_ADMIN`: Dueño de la Pyme.
    - `ORG_MEMBER`: Vendedor/Operario (Acceso restringido).
- **Localización (Chile):**
    - Moneda: Formatea siempre como CLP (`$ 1.500`).
    - Textos UI: Español de Chile (Formal pero cercano).
    - Fechas: Asegura que los reportes respeten `America/Santiago`.

## 5. Ejemplo de Flujo de Trabajo (Prompt Mental)
Si te pido "Crear el módulo de Clientes":
0.  ¡IMPORTANTE! Lo primero siempre es revisar `skills` y `tools` disponibles para cumplir la tarea solicitada.
1.  Define el modelo `Client` en `schema.prisma`.
2.  Define el esquema de validación Zod en `src/lib/validators/client.ts`.
3.  Crea la API: `GET/POST /api/clients`.
4.  Crea el hook: `useClients` (Query) y `useCreateClient` (Mutation).
5.  Crea la UI: `ClientDataTable.tsx` y `ClientForm.tsx` usando Shadcn.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Girosmedia)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Girosmedia)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
