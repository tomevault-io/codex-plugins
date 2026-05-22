# Reglas Clean Code y Buenas Prácticas Python

- Sigue PEP8: nombres descriptivos, indentación de 4 espacios, líneas <80 caracteres.
- Cada función/módulo debe tener una única responsabilidad.
- Evita duplicidad y lógica innecesaria; reutiliza utilidades.
- Prefiere funciones puras y evita efectos secundarios inesperados.
- Usa docstrings en todas las funciones y módulos.
- Maneja errores con try/except y mensajes claros para el usuario.
- No uses valores mágicos; define constantes o usa `.env` para configuración.
- Separa lógica de negocio, utilidades y presentación en archivos distintos.
- El flujo principal (`main.py`) solo orquesta, nunca implementa lógica detallada.
- Prefiere imports explícitos y ordenados.
- Documenta cualquier convención o patrón no estándar en el README o instrucciones.

# Copilot Repository Instructions

Este repositorio es un proyecto Python modular para análisis automatizado de dependencias, escaneo de vulnerabilidades y generación de reportes. Integra Snyk para seguridad/licencias y PyPI para enriquecimiento de metadatos.

## Arquitectura y Componentes Clave
- **main.py**: Orquesta el flujo principal. Crea venv temporal, instala dependencias, ejecuta Snyk, mapea dependencias, enriquece con PyPI, aplica reglas de negocio y genera el reporte.
- **snyk_analyzer.py**: Ejecuta Snyk CLI y retorna objetos JSON de dependencias y vulnerabilidades.
- **dependency_utils.py**: Extrae recursivamente todo el árbol de dependencias desde la salida de Snyk.
- **package_utils.py**: Enriquece cada paquete (incluyendo subdependencias) con metadatos de PyPI.
- **report_utils.py**: Genera el reporte JSON jerárquico, anidando dependencias como objetos.
- **business_rules.py**: Aplica reglas de aprobación usando configuración en `.env`.
- **pypi_info.py**: Consulta la API de PyPI para cada paquete@versión.

## Flujo de Desarrollo
- Activa el entorno virtual: `.venv\Scripts\activate`
- Instala dependencias: `pip install -r requirements.txt`
- Ejecuta el análisis: `python main.py`
- El reporte se genera en `consolidated_report.json` (estructura de árbol, incluye todas las dependencias/subdependencias)

## Patrones y Convenciones
- **Separación estricta de responsabilidades**: Cada módulo tiene una función clara.
- **Recorrido recursivo de dependencias**: Se enriquecen todas las dependencias y subdependencias, no solo las principales.
- **Formato de reporte**: El reporte final anida dependencias como objetos, reflejando el árbol real.
- **Manejo de errores**: Todos los llamados a subprocess y APIs están protegidos con try/except y mensajes claros.
- **Configuración sensible**: Usa `.env` para reglas de negocio; nunca subas secretos.

## Integraciones
- **Snyk CLI**: Para análisis de vulnerabilidades/licencias. La salida se procesa directamente.
- **PyPI API**: Para enriquecer cada paquete del árbol.

## Ejemplos de Extensión
- Para agregar una nueva regla de negocio, extiende `business_rules.py` y actualiza el filtrado en `main.py`.
- Para modificar el formato del reporte, ajusta la lógica de árbol en `report_utils.py`.

---

## Chat Style (Overrides Defaults)

Estas reglas se aplican específicamente al comportamiento conversacional de GitHub Copilot Chat y los modelos avanzados (Claude, GPT-4, etc.):

- **Modo principal:** Devuelve solo código o texto plano funcional.  
  No generes explicaciones ni archivos Markdown a menos que el usuario lo solicite explícitamente.
- **No documentación automática:**  
  No crees `README.md`, `.md` de explicación, ni resúmenes automáticos.
- **Confirmaciones concisas:**  
  Al completar una tarea, responde con una confirmación breve (por ejemplo, “✅ Listo”) en lugar de explicar lo que hiciste.
- **Formato de salida:**  
  Cuando se requiera mostrar ejemplos o resultados, usa bloques de código simples (```` ```python ```` o ```` ```json ````) sin encabezados ni texto explicativo.
- **Explicación bajo demanda:**  
  Solo agrega comentarios o documentación si el usuario dice “explica”, “documenta”, o “resume”.
- **Prioridad:**  
  Estas reglas prevalecen sobre las instrucciones por defecto del modelo en cuanto a formato y nivel de detalle.

---

## Referencias
- Instrucciones avanzadas Copilot: https://docs.github.com/es/copilot/how-tos/configure-custom-instructions/add-repository-instructions

---
> Source: [estibenlicona/scan-pypi](https://github.com/estibenlicona/scan-pypi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
