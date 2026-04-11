# FixFlow Play - Copilo### 🗂️ GESTIÓN DE ARCHIVOS - REGLAS ESTRICTAS

### ❌ PROHIBIDO CREAR ARCHIVOS INNECESARIOS:
- **NUNCA crear archivos de backup** (backup.tsx, old.tsx, temp.tsx)
- **NUNCA crear múltiples versiones** del mismo archivo
- **NUNCA dejar archivos de prueba** o experimentales
- **JAMÁS crear carpetas vacías** o estructuras no utilizadas
- **PROHIBIDO páginas monolíticas** - SIEMPRE componentizar

### 📦 OBLIGATORIO - COMPONENTIZACIÓN TOTAL:
- **CADA PÁGINA** debe dividirse en componentes modulares
- **CREAR CARPETA** `/components/nombre-pagina/` para cada página
- **SEPARAR SECCIONES** en componentes independientes (Header, Form, Actions)
- **EXPORTS CENTRALIZADOS** en `index.ts` de cada carpeta
- **PROPS TIPADAS** con interfaces TypeScript específicasuctions

## Project Overview
FixFlow Play is a lean construction productivity tracking app for mobile and desktop, built with Next.js 14+, TypeScript (no `any`), Chakra UI + Emotions, Node.js/Express backend, and PostgreSQL on Supabase.

## 🚨 REGLAS CRÍTICAS - CHAKRA UI MANDATORIO + MINIMALISMO EXTREMO

### ❌ PROHIBICIONES ABSOLUTAS:
- **NUNCA usar HTML básico** (div, button, input) - SIEMPRE usar componentes de Chakra UI
- **NUNCA usar CSS personalizado** o estilos inline - SOLO props de Chakra UI https://chakra-ui.com/docs/components/concepts/overview
- **NUNCA usar clases de Tailwind** para componentes que existen en Chakra UI
- **CRIMEN DE LESA HUMANIDAD**: Usar elementos HTML personalizados cuando Chakra UI tiene el componente
- **PROHIBIDO ESPACIOS VERTICALES GRANDES**: gap={4}, mb={4}, mt={4} o superiores están PROHIBIDOS
- **MÁXIMO ESPACIADO PERMITIDO**: gap={2}, mb={2}, mt={2} - preferir gap={1}, mb={1}, mt={1}

### 🎯 PRINCIPIOS DE MINIMALISMO EXTREMO:
- **COMPACTACIÓN OBLIGATORIA**: Toda interfaz debe ser ultra-compacta sin espacios verticales innecesarios
- **GAPS MÍNIMOS**: Usar gap={1} como estándar, gap={2} solo en casos excepcionales  
- **MÁRGENES REDUCIDOS**: mb={1}, mt={1} como máximo - eliminar todo espacio no esencial
- **TAMAÑOS DE FUENTE**: Preferir fontSize="xs" y fontSize="sm" sobre tamaños grandes
- **ICONOS COMPACTOS**: Usar size={12} o size={10} en lugar de size={16} o superiores
- **ALTURA MÍNIMA**: Componentes deben ocupar el mínimo espacio vertical posible

## 🗂️ GESTIÓN DE ARCHIVOS - REGLAS ESTRICTAS

### ❌ PROHIBIDO CREAR ARCHIVOS INNECESARIOS:
- **NUNCA crear archivos de backup** (backup.tsx, old.tsx, temp.tsx)
- **NUNCA crear múltiples versiones** del mismo archivo
- **NUNCA dejar archivos de prueba** o experimentales
- **JAMÁS crear carpetas vacías** o estructuras no utilizadas

### 🗑️ BORRAR INMEDIATAMENTE:
- **Si un archivo ya NO será utilizado** → BORRARLO INMEDIATAMENTE
- **Si cambias la estructura** → BORRAR archivos obsoletos AL INSTANTE  
- **Si refactorizas componentes** → ELIMINAR versiones anteriores SIN DEMORA
- **Si experimentas con código** → LIMPIAR después de confirmar la implementación

### ✅ MANTENER SOLO LO ESENCIAL:
- **Un componente por propósito específico**
- **Estructura limpia y minimalista**
- **Cero redundancia de código**
- **Documentación actualizada y precisa**
- **PÁGINAS = COMPOSICIÓN de componentes modulares**

### 🎯 PATRÓN OBLIGATORIO PARA PÁGINAS:
```
src/app/nombre-pagina/page.tsx      # Solo lógica + composición
src/components/nombre-pagina/       # Componentes modulares
├── PageHeader.tsx                  # Título, subtítulo
├── MainForm.tsx                    # Formulario principal
├── ActionButtons.tsx               # Botones de acción
├── SectionName.tsx                 # Secciones específicas
└── index.ts                        # Exports centralizados
```

### ✅ USAR SIEMPRE:
- **Button** de Chakra UI (NUNCA `<button>`)
- **Box** de Chakra UI (NUNCA `<div>`)
- **Text** de Chakra UI (NUNCA `<p>`, `<span>`)
- **Heading** de Chakra UI (NUNCA `<h1>`, `<h2>`)
- **Input** de Chakra UI (NUNCA `<input>`)
- **Stack, VStack, HStack** de Chakra UI (NUNCA flexbox manual)
- **Container, Grid** de Chakra UI (NUNCA CSS Grid manual)

### 🎨 SISTEMA DE COMPONENTES:
- **Chakra UI**: Para TODOS los componentes de interfaz
- **Tailwind CSS**: SOLO para utilidades que Chakra UI no cubre
- **Props de Chakra**: bg, color, p, m, fontSize, etc.

### 📱 RESPONSIVIDAD MINIMALISTA:
- **Optimizado para móvil y desktop** - Interfaces adaptables ultra-compactas
- **Usar breakpoints de Chakra UI**: `{ base: "sm", md: "sm" }` (evitar tamaños grandes)
- **Considerar contexto de uso**: Móvil para obra, desktop para oficina
- **ESPACIADO RESPONSIVO**: Mantener minimalismo en todos los breakpoints
- **RESPONSIVIDAD AUTOMÁTICA**: Chakra UI maneja responsive por defecto - NO reinventar

### 🎨 JERARQUÍA DE ESPACIADO (DE MENOR A MAYOR PREFERENCIA):
1. **gap={0}, mb={0}, mt={0}** - ✅ PERFECTO (sin espacios)
2. **gap={1}, mb={1}, mt={1}** - ✅ EXCELENTE (mínimo necesario)  
3. **gap={2}, mb={2}, mt={2}** - ⚠️ ACEPTABLE (casos excepcionales)
4. **gap={3+}, mb={3+}, mt={3+}** - ❌ PROHIBIDO (espacios excesivos)

### 🤝 CHAKRA UI + TAILWIND CSS:

### ⚙️ ARQUITECTURA SERVER-SIDE + COMPONENTES HÍBRIDOS:
- **PRIORIDAD SERVER-SIDE**: Usar Server Components por defecto para performance
- **CLIENT-SIDE SOLO PARA**: Botones interactivos, formularios, estados reactivos
- **COMPONENTES HÍBRIDOS**: Server Component padre + Client Component para interactividad
- **'use client' MÍNIMO**: Solo en componentes que requieren useState, useEffect, onClick
- **CHAKRA UI RESPONSIVO**: Usar props responsive built-in `{ base, md }` - NO CSS custom

### 📦 COMPONENTIZACIÓN TOTAL OBLIGATORIA:
- **PÁGINAS MODULARES**: Dividir TODA página en componentes pequeños y específicos
- **UN COMPONENTE = UNA RESPONSABILIDAD**: Header, Form, Section, Button, etc.
- **CARPETA POR PÁGINA**: `/components/nombre-pagina/` con todos sus componentes
- **INDEX.TS CENTRALIZADO**: Exports organizados en `index.ts` por carpeta
- **PROPS TIPADAS**: TypeScript interfaces para todos los componentes
- **REUTILIZACIÓN**: Componentes deben ser reutilizables entre páginas
- **CHAKRA UI PURO**: Cero HTML básico, solo componentes de Chakra UI
- **EJEMPLO ESTRUCTURA**:
  ```
  /components/create-project/
  ├── ProjectHeader.tsx
  ├── BackButton.tsx  
  ├── ProjectForm.tsx
  ├── ActionButtons.tsx
  └── index.ts
  ```



## Key Architecture Patterns

### Frontend Structure (Next.js App Router)
- **App Router**: All routes in `src/app/` with TypeScript strict mode
- **UI Components**: Chakra UI for complex components, Tailwind for utilities (NO Sass/SCSS)
- **State Management**: React Context for global state, React Hook Form + Zod for forms
- **Responsive**: Mobile-first design with Chakra responsive props `{ base: "...", md: "..." }`

### Backend Architecture
- **API Structure**: Express.js with TypeScript controllers in `backend/src/controllers/`
- **Database**: PostgreSQL with Supabase, using triggers for activity classification
- **Authentication**: Supabase Auth + JWT for role-based access (worker/foreman/manager)

### Domain Models
- **Users**: Role-based (trabajador, capataz, gerente) with project associations  
- **Projects**: Contain construction phases (Cimentación, Estructura) and processes
- **Activities**: Auto-classified as VA/NVA-E/NVA-NE with lean waste categories
- **ETMo Metrics**: Construction adaptation of OEE (Availability, Performance, Quality)

### Critical Workflows
```bash
# Development
npm run dev              # Frontend (port 3000)
npm run backend:dev      # Backend (port 3001)

# Mobile/Desktop builds
npm run electron:dev     # Desktop app
npm run cap:sync         # Mobile sync
```

### Database Patterns
- **Activity Classification**: Automated via PostgreSQL triggers extracting verbs/gerunds
- **Lean Waste Types**: 10 categories mapped to construction activities
- **Privacy**: Anonymized team data for managers, individual data for workers

### UI/UX Conventions
- **Mobile**: Large buttons for construction site use, offline support, voice input
- **Desktop**: Dashboard-heavy for management reporting and analysis  
- **Forms**: Always use React Hook Form + Zod validation, no untyped forms

### Performance Considerations
- **Offline-First**: Mobile app caches data locally, syncs on reconnection
- **Real-time**: Timer functionality for live activity tracking
- **Reports**: PDF export for construction reports and lean recommendations

## 🧹 DISCIPLINA DE ARCHIVOS - ZERO TOLERANCE

### 🚨 REGLAS DE ORO PARA COPILOT:
1. **ANTES de crear cualquier archivo** → Pregúntate: "¿Es absolutamente necesario?"
2. **DESPUÉS de cualquier refactor** → Elimina INMEDIATAMENTE archivos obsoletos
3. **DURANTE el desarrollo** → NO crear "backup.tsx", "old.tsx", "temp.tsx", etc.
4. **AL experimentar** → Usa una sola versión, modifica in-place
5. **ANTE la duda** → MEJOR modificar archivo existente que crear uno nuevo

### 📋 CHECKLIST OBLIGATORIO:
- [ ] ¿Este archivo tiene un propósito único y específico?
- [ ] ¿No existe ya un archivo que haga esto mismo?
- [ ] ¿Será utilizado más allá de esta implementación?
- [ ] ¿Está en la ubicación correcta según la arquitectura?
- [ ] ¿Tiene exports/imports limpios y organizados?

### 🔄 PROCESO DE LIMPIEZA AUTOMÁTICA:
- **Al cambiar estructura** → `rm archivo-viejo.tsx` inmediatamente
- **Al refactorizar** → Eliminar versión anterior en el mismo commit
- **Al experimentar** → Usar git stash, NO crear archivos temporales
- **Al finalizar tarea** → Verificar que no quedaron archivos huérfanos
- **NUEVA PÁGINA** → Crear estructura de componentes desde el inicio
- **REFACTORIZAR** → Dividir página monolítica en componentes modulares

### ❌ EJEMPLOS DE LO QUE JAMÁS HACER:
```bash
# ❌ PROHIBIDO - Crear backups
page.tsx → page-backup.tsx
component.tsx → component-old.tsx
styles.css → styles-temp.css

# ❌ PROHIBIDO - Versiones múltiples
LoginForm.tsx
LoginFormV2.tsx
LoginForm-working.tsx
LoginForm-fixed.tsx

# ❌ PROHIBIDO - Experimentos temporales
test-component.tsx
experiment-layout.tsx
try-this-approach.tsx
```

### ✅ ENFOQUE CORRECTO:
```bash
# ✅ CORRECTO - Un archivo, una responsabilidad
LoginForm.tsx (modificar in-place)

# ✅ CORRECTO - Usar git para historial
git stash          # Guardar experimento
git commit -m "fix" # Commitear versión limpia

# ✅ CORRECTO - Borrar inmediatamente al refactorizar
rm old-component.tsx  # Tan pronto como se reemplace
```

## File Organization Specifics
- `src/types/`: All TypeScript interfaces (User, Activity, Project, ETMo)
- `src/lib/supabase.ts`: Database client configuration
- `backend/src/services/`: Business logic for lean calculations and classifications
- `src/components/lean/`: Lean construction specific UI components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BBripy207)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/BBripy207)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
