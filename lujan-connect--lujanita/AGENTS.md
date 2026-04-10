# Instrucciones de GitHub Copilot - Lujanita

**Proyecto de Desarrollo Guiado por Especificaciones (SDD)**

Este proyecto sigue el [toolkit de Spec-Driven Development de GitHub](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/) para generar y mantener la experiencia del chatbot **Lujanita**.

---

## Contexto del Proyecto

Plataforma de automatización conversacional para Lujan de Cuyo Express:
- **Widget conversacional**: React + Vite (`apps/widget/`)
- **BFF (Backend for Frontend)**: Java 21 + Spring Boot (`apps/middleware/`) que orquesta interacción con LLM (Ollama) y MCP (Odoo)
- **Backend fuente**: Odoo (`platform/odoo/`) con servidor MCP oficial
- **Contratos MCP**: `packages/contracts/`
- **Infraestructura**: Google Cloud Run + Cloud Build

> Nota: MCP se refiere a "Model Context Protocol" (no "Model Content Protocol"). Usar siempre esta denominación para evitar confusiones.

---

## 🎯 Flujo de Desarrollo Guiado por Especificaciones

### Fase 0: Especificación (BLOQUEANTE)

**DETENTE** - Antes de cualquier código, verifica si existe especificación en `/specs/NNN-feature/`:

```
/specs/NNN-feature-name/
├── spec.md         # Historias de usuario, requisitos FR-XXX, criterios de éxito
├── plan.md         # Decisiones de arquitectura
├── research.md     # Evaluación de tecnologías
├── data-model.md   # Esquemas de entidades y relaciones MCP/Odoo
└── tasks.md        # Desglose de tareas con enfoque test-first
```

**Si la especificación no existe** → Créala primero usando `.github/prompts/speckit.*.prompt.md`

### Fase 0.5: Generación de Feature Files Gherkin (OBLIGATORIO)

**DESPUÉS de crear `spec.md` y ANTES de implementar código**:

1. Usa `.github/prompts/speckit.gherkin.prompt.md` para generar feature files.
2. Cada User Story debe producir un feature file Gherkin con ubicación según capa:
   - Widget React → `apps/widget/features/`
   - BFF (Java) → `apps/middleware/features/`
   - Orquestación u otros servicios → `apps/services/features/`
3. Cada feature file DEBE incluir:
   - Tags Jira (`@JIRA:LUJ-XXX`)
   - Tags de prioridad (`@P1`, `@P2`, `@P3`)
   - Tags de requisitos (`@FR-XXX`)
   - Escenarios ejecutables Given-When-Then
4. Crear issue Jira "BDD: <Nombre Corto Feature>" con:
   - Ruta del `spec.md`
   - Lista de feature files creados + paths
   - Conteo de escenarios y tags
   - Comando inicial (debe fallar): p.ej. `cd apps/widget && npm run test:bdd -- features/chatbot.feature`
5. Comentar en el issue de negocio original: "Feature Gherkin generada en: <paths>; ver issue BDD: LUJ-XYZ".

```gherkin
@JIRA:LUJ-2
@P1
Feature: Recepción de órdenes Odoo
  Como asistente Lujanita necesito consultar órdenes confirmadas en Odoo vía MCP

  @smoke @FR-001
  Scenario: Obtener orden confirmada
    Given existe una orden confirmada en Odoo
    When Lujanita consulta la orden por su ID
    Then responde con estado, fecha y datos de contacto
```

### Fase 1: Recopilación de Contexto por IA

Lee en este orden:
1. `/specs/NNN-feature/spec.md` - Requisitos de negocio
2. Feature files Gherkin - Criterios de aceptación
3. Este archivo - Patrones generales del proyecto
4. `.github/copilot-knowledge/` - Patrones técnicos específicos
5. `/docs/` - Documentación de referencia (Odoo, MCP, flujos internos)

### Fase 2: Implementación Test-First con BDD (NO NEGOCIABLE)

**A. Implementar Step Definitions (Tests BDD - DEBEN FALLAR)**

```bash
# Widget (React + Cucumber)
cd apps/widget && npm run test:bdd  # DEBE FALLAR - steps undefined
# Implementar steps en apps/widget/steps/*_steps.ts con Testing Library

# BFF (Java + Cucumber + JUnit)
cd apps/middleware && mvn test -Dcucumber.filter.tags="@bdd"  # DEBE FALLAR - steps undefined
# Implementar steps en apps/middleware/src/test/java/.../*Steps.java usando RestAssured/Ollama/MCP client
```

**B. Implementar Código Mínimo**

```bash
# Widget
cd apps/widget && npm run test:unit  # Vitest + Testing Library

# BFF
cd apps/middleware && mvn test  # JUnit + WireMock para Odoo/MCP + Ollama
```

**C. Verificar que Tests Pasen**

```bash
cd apps/widget && npm run test:bdd && npm run test
cd apps/middleware && mvn test -Dcucumber.filter.tags="@bdd" && mvn test
```

**D. Refactorizar y Commitear**

```bash
git add .
git commit -m "feat(LUJ-XXX): Implementar [feature] con tests BDD"
```

### Fase 3: Actualización de Documentación

- Actualiza `.github/copilot-knowledge/` si surge un nuevo patrón
- Actualiza `/docs/` con cualquier convención nueva
- Anota lecciones aprendidas en `spec.md`

---

## 📖 Base de Conocimiento (Leer Antes de Codificar)

**Ubicación**: `.github/copilot-knowledge/`

| Artículo | Cuándo Leer |
|---------|-------------|
| **widget-patterns.md** | Componentes React, i18n, hooks de chat |
| **middleware-patterns.md** | Integración BFF↔MCP↔Ollama, Spring Boot, clientes HTTP |
| **testing-guide.md** | Estrategia de pruebas BDD/Unitarias | 
| **contracts-mcp.md** | Contratos MCP (Model Context Protocol) entre BFF y Odoo |

**Uso**: referencia artículos específicos en prompts, por ejemplo: "Implementar step siguiendo middleware-patterns.md".

---

## 🏗️ Estructura del Monorepo

```
apps/
  widget/      # Widget React + Vite (puerto 4173)
  middleware/  # BFF Java Spring Boot + Cucumber (puerto 9000)
  services/    # Orquestaciones adicionales

packages/
  contracts/   # Definiciones MCP, DTOs compartidos
  ui/          # Componentes compartidos (si aplica)

platform/
  odoo/        # Configuración del servidor MCP en Odoo
```

### Convenciones de Rutas

```
¿Es compartido entre widget y BFF?
├─ SÍ → packages/
│  ├─ Contratos MCP → packages/contracts/
│  └─ Componentes UI → packages/ui/
└─ NO → apps/
   ├─ Conversación UI → apps/widget/
   ├─ BFF/API REST → apps/middleware/
```

---

## 💻 Estándares de Código

### React Widget (TypeScript)
- `strict` + `strictNullChecks`
- Componentes funcionales con hooks
- i18n obligatorio (`apps/widget/src/i18n`)
- Pruebas con Vitest + Testing Library
- Datos mockeados deben ir en `apps/widget/mocks/`

### BFF Java (Spring Boot)
- Java 21, Maven
- Orquestación BFF ↔ Ollama (ollama4j) ↔ MCP Odoo
- Pruebas con JUnit 5 + Mockito + WireMock
- Métricas y logs estructurados (Micrometer + JSON)

---

## 🏷️ Uso de Lombok en Java

- Todas las clases Java de modelo, DTOs y configuraciones en `apps/middleware/` deben usar anotaciones de Lombok (
  `@Data`, `@Getter`, `@Setter`, `@Builder`, etc.) para evitar boilerplate de getters/setters, equals, hashCode y
  toString.
- No escribir manualmente métodos de acceso salvo casos especiales (por ejemplo, lógica de validación o serialización
  custom).
- Importar Lombok en el `pom.xml` y asegurarse de que esté habilitado en el IDE.
- Ejemplo:

```java
import lombok.Data;

@Data
public class MiDTO {
    private String nombre;
    private int edad;
}
```

- Revisar y refactorizar clases existentes para cumplir este estándar.

---

## 🧪 Testing (Test-First OBLIGATORIO)

### Pirámide

```
    ┌──────────┐
    │   E2E    │  Escenarios críticos de conversación
    ├──────────┤
    │ Integr.  │  Contratos MCP y llamadas Odoo/LLM
    ├──────────┤
    │   Unit   │  Hooks, servicios Java, reducers
    └──────────┘
```

### Comandos

```bash
cd apps/widget && npm run test        # Unit
cd apps/widget && npm run test:bdd    # BDD
cd apps/middleware && mvn test        # Unit/Integration
cd apps/middleware && mvn testBdd     # BDD
```

### Reglas
1. Tests se escriben antes que la implementación
2. Cada paso BDD debe usarse en CI
3. Contratos MCP deben tener mocks en `packages/contracts/mocks/`

---

## 🔍 Observabilidad

- Logs estructurados con `correlationId`, `llmOperation`, `mcpOperation`
- Errores deben incluir códigos (`OD00X`, `LLM00X`, `MW00X`, `UI00X`)
- Métricas clave: latencia MCP/LLM, tasa de respuestas Odoo, satisfacción del usuario

---

## 🌍 i18n

- Claves en inglés camelCase
- `translations.ts` para widget
- No hardcodear textos visibles

---

## 🚩 Feature Flags

- Utiliza `FEATURE_<NOMBRE>`
- Por defecto deshabilitadas
- Documenta estrategia de despliegue en `plan.md`

---

## 📝 Tareas Comunes

### Nueva Feature
1. Crear `/specs/NNN-feature/` (spec, plan, data-model, tasks)
2. Generar feature files Gherkin (widget/BFF)
3. Tests que fallen
4. Implementar lo mínimo para pasar
5. Actualizar documentación

### Nuevo Contrato MCP
1. Definir DTO en `packages/contracts/src`
2. Documentar en `data-model.md`
3. Agregar tests de compatibilidad (Java + TypeScript)
4. Publicar mocks en `packages/contracts/mocks/`

---

## 🤖 Protocolo para Asistentes IA

1. Verifica especificación (`/specs/`)
2. Lee base de conocimiento relevante
3. Genera tests BDD/Unit que fallen
4. Implementa siguiendo patrones
5. Sugerir comandos de validación (`npm run test`, `mvn test`, etc.)

---

## ⚡ Referencia Rápida

| Tarea | Comando |
|------|---------|
| Configurar widget | `cd apps/widget && npm install` |
| Configurar BFF | `cd apps/middleware && mvn clean build` |
| Ejecutar tests widget | `npm run test` |
| Ejecutar tests BFF | `mvn test` |
| Ejecutar BDD widget | `npm run test:bdd` |
| Ejecutar BDD BFF | `mvn testBdd` |

---

**Versión**: 1.0.0 (Lujanita SDD)  
**Última Actualización**: 2025-11-29  
**Cumplimiento SDD**: 100% ✅  
**GitHub SDD Toolkit**: [Spec-Driven Development](https://github.com/github/spec-kit/blob/main/spec-driven.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lujan-connect)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/lujan-connect)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
