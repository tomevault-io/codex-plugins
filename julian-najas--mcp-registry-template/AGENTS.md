## Contrato de errores MCP

El contrato MCPError (shape, error_code) **solo aplica a endpoints bajo /mcp/**.
Errores de FastAPI (404, 422, rutas no encontradas, validaciones fuera del router MCP) siguen el formato estándar FastAPI (`detail`).
# Copilot Instructions for AI Agents

## Arquitectura y Componentes Clave

## Flujos de Desarrollo y Pruebas

  ```sh

- Infra MCP lista para producción
- Idempotencia y concurrencia probadas
- Contratos y errores MCPError
- Roles y riesgos obligatorios
- Tests e2e como gate



- Añade tools en `app/mcp/registry.py` siguiendo el estándar
- Corre tests antes de merge
- No modificar core sin actualizar tests

- **Versionado**: `get_tool("foo")` retorna la versión más alta registrada.

- Si pasa CI, es producto
- Lint: ruff
- Tests: pytest
- Build: docker
## Ejemplo de Registro de Tool
```python
from app.mcp.registry import ToolDefinition, register_tool, ToolRisk
from pydantic import BaseModel

class In(BaseModel):
    value: str
class Out(BaseModel):
    ok: bool

def handler(_: In) -> Out:
    return Out(ok=True)

register_tool(ToolDefinition(
    name="foo@1",
    description="Demo tool",
    input_model=In,
    output_model=Out,
    risk=ToolRisk.READ_ONLY,
    roles_allowed=["user"],
    handler=handler,
))
```

## Build y Ejecución
- **Docker**: El contenedor expone FastAPI en el puerto 8000. Usa:
  ```sh
  docker build -t mcp .
  docker run -p 8000:8000 mcp
  ```
- **Local**: Instala dependencias y ejecuta con Uvicorn:
  ```sh
  pip install fastapi uvicorn pydantic
  uvicorn app.main:app --reload
  ```

## Archivos Clave
- `app/mcp/registry.py`: Registro y lógica de tools
- `app/mcp/policy.py`: Políticas de roles y riesgos
- `app/mcp/audit.py`: Auditoría
- `tests/e2e/`: Pruebas de integración y contractuales

---
¿Falta algún flujo, convención o integración importante? Indícalo para mejorar estas instrucciones.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julian-najas)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/julian-najas)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
