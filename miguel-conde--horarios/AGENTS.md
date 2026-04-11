# Instrucciones para GitHub Copilot

## Contexto del Proyecto

Este es un proyecto de análisis científico sobre los diferentes escenarios de horarios en España. Combina Python y R para realizar cálculos astronómicos, análisis de datos y visualizaciones comparativas.

## Objetivo Principal

Estudiar el impacto de 6 escenarios de horario:
1. **Horario Actual** (CET/CEST): UTC+1 invierno, UTC+2 verano
2. **Horario Verano Permanente**: UTC+2 todo el año
3. **Horario Invierno Permanente**: UTC+1 todo el año
4. **Horario Portugal** (WET/WEST): UTC+0 invierno, UTC+1 verano
5. **Horario Portugal Verano Permanente**: UTC+1 todo el año
6. **Horario Portugal Invierno Permanente**: UTC+0 todo el año

## Reglas de Código

### Python

#### Estilo y Convenciones
- **OBLIGATORIO**: Seguir PEP 8 estrictamente en todo el código
- Usar type hints en todas las funciones y métodos
- Preferir pathlib sobre os.path
- Usar f-strings para formateo
- Manejar errores con try/except específicos
- Usar logging en lugar de print para debug

#### Documentación
- **OBLIGATORIO**: Documentar TODOS los módulos con docstring al inicio del archivo
- **OBLIGATORIO**: Documentar TODAS las clases con docstring describiendo su propósito
- **OBLIGATORIO**: Documentar TODAS las funciones con docstring (parámetros, retorno, excepciones)
- **OBLIGATORIO**: Documentar TODOS los métodos (públicos y privados) con docstring
- Usar formato Google Style para docstrings:
  ```python
  def funcion(parametro: tipo) -> tipo_retorno:
      """Descripción breve de la función.
      
      Descripción más detallada si es necesario.
      
      Args:
          parametro: Descripción del parámetro.
          
      Returns:
          Descripción de lo que retorna.
          
      Raises:
          TipoExcepcion: Cuándo se lanza.
          
      Examples:
          >>> funcion(valor)
          resultado_esperado
      """
  ```

#### Programación Orientada a Objetos
- **OBLIGATORIO**: Usar OOP (Programación Orientada a Objetos) siempre que sea posible
- Crear clases para encapsular datos y comportamiento relacionado
- Usar herencia cuando haya relaciones "es-un"
- Usar composición cuando haya relaciones "tiene-un"
- Aplicar principios SOLID:
  - **S**ingle Responsibility: Una clase, una responsabilidad
  - **O**pen/Closed: Abierto a extensión, cerrado a modificación
  - **L**iskov Substitution: Las subclases deben ser sustituibles
  - **I**nterface Segregation: Interfaces pequeñas y específicas
  - **D**ependency Inversion: Depender de abstracciones

#### Patrones de Diseño
- **OBLIGATORIO**: Usar patrones de diseño cuando sea necesario
- Patrones recomendados para este proyecto:
  - **Strategy**: Para diferentes algoritmos de cálculo horario
  - **Factory**: Para crear objetos de análisis según escenario
  - **Observer**: Para notificar cambios en datos
  - **Singleton**: Para configuración global (si aplica)
  - **Builder**: Para construir objetos complejos de análisis
  - **Template Method**: Para flujos de análisis con pasos comunes
  - **Decorator**: Para añadir funcionalidad a cálculos existentes
  - **Adapter**: Para integrar diferentes fuentes de datos

### R
- Usar tidyverse cuando sea posible
- Nombres de variables en snake_case
- Comentarios descriptivos en español
- Usar %>% (pipe) para operaciones encadenadas
- Preferir ggplot2 para visualizaciones
- Funciones documentadas con roxygen2 style
- Aplicar programación funcional cuando sea apropiado

## Estructura de Datos

### Ciudades (config/ciudades.yaml)
```yaml
ciudad:
  lat: float
  lon: float
  region: str
  poblacion: int
  zona_climatica: str
```

### Escenarios (config/escenarios.yaml)
```yaml
escenario:
  nombre: str
  descripcion: str
  invierno_utc: int
  verano_utc: int
  cambio_horario: bool
```

## Convenciones de Nombres

- **Archivos**: snake_case.py, snake_case.R
- **Clases Python**: PascalCase (ej: `HorarioAnalyzer`, `EscenarioFactory`)
- **Funciones**: snake_case (ej: `calcular_amanecer`, `generar_reporte`)
- **Variables**: snake_case (ej: `ciudad_nombre`, `utc_offset`)
- **Constantes**: UPPER_SNAKE_CASE (ej: `MAX_CIUDADES`, `DEFAULT_YEAR`)
- **Métodos privados**: _snake_case (ej: `_validar_fecha`)

## Librerías Principales

### Python
- astral: Cálculos de amanecer/atardecer
- pandas: DataFrames y análisis
- matplotlib/seaborn: Visualizaciones
- pytz: Zonas horarias
- pyyaml: Configuración
- typing: Type hints

### R
- lubridate: Fechas y tiempos
- ggplot2: Gráficos
- dplyr: Manipulación de datos
- tidyr: Limpieza de datos
- yaml: Configuración

## Preferencias de Visualización

- Gráficos con títulos descriptivos en español
- Ejes etiquetados claramente
- Leyendas cuando hay múltiples series
- Colores consistentes por escenario:
  - Actual: azul (hex: 1f77b4)
  - Verano permanente: rojo (hex: d62728)
  - Invierno permanente: verde (hex: 2ca02c)
  - Portugal: naranja (hex: ff7f0e)
  - Portugal verano permanente: púrpura (hex: 9467bd)
  - Portugal invierno permanente: marrón (hex: 8c564b)
- Guardar en results/plots/ con nombres descriptivos
- Formato: PNG (300 DPI mínimo) para presentaciones, PDF para publicaciones

## Análisis Típicos

1. **Comparación de amaneceres/atardeceres** por ciudad y escenario
2. **Horas de luz solar** a lo largo del año
3. **Impacto en horarios laborales** (7:00-15:00, 9:00-18:00)
4. **Análisis regional** por zonas climáticas
5. **Variación estacional** entre regiones

## Consideraciones Importantes

- España está geográficamente en UTC-0/UTC+1, pero usa UTC+1/UTC+2
- Las Islas Canarias históricamente usaron UTC+0/UTC+1
- El cambio horario ocurre el último domingo de marzo y octubre
- Latitudes: 28°N (Canarias) a 43°N (Norte peninsular)
- Considerar impacto en salud, economía y sociedad

## Formato de Salida

### DataFrames
- Fechas en formato ISO 8601 (YYYY-MM-DD)
- Horas en formato 24h (HH:MM:SS)
- Nombres de columnas en español y snake_case

### Reportes
- Markdown para documentación
- CSV para datos tabulares
- PNG/PDF para gráficos
- Guardar en results/reports/ y results/plots/

### Documentos Quarto (.qmd)
- **OBLIGATORIO**: Incluir cabecera YAML completa al inicio del documento
- Estructura de cabecera estándar:
  ```yaml
  ---
  title: "Título Descriptivo del Análisis"
  author: "Proyecto Horarios"
  date: "Mes Año"
  format:
    html:
      toc: true
      toc-depth: 3
      number-sections: true
      code-fold: false
      embed-resources: true
      theme: cosmo
  jupyter:
    kernelspec:
      name: horarios-venv
      language: python
      display_name: Python (horarios)
  lang: es
  ---
  ```
- **Campos obligatorios**:
  - `title`: Título descriptivo en español
  - `author`: "Proyecto Horarios" (consistencia)
  - `format.html.embed-resources: true`: Para generar HTML autocontenido
  - `jupyter.kernelspec.name: horarios-venv`: Usar el kernel del venv
- **Campos recomendados**:
  - `toc: true`: Tabla de contenidos
  - `number-sections: true`: Numeración automática
  - `code-fold: false`: Mostrar código por defecto (análisis científico)
  - `theme: cosmo`: Tema limpio y profesional
  - `lang: es`: Idioma español para metadatos
- **Bloques de código Python**: Usar ` ```{python} ` con opciones:
  - `#| echo: true` - Mostrar código (por defecto)
  - `#| warning: false` - Ocultar warnings si son irrelevantes
  - `#| fig-cap: "Descripción"` - Caption para figuras
- **Renderizado**: Ejecutar `quarto render archivo.qmd` desde el directorio docs/
- **Conversión de tiempo**: Siempre convertir HH:MM:SS a decimal con `map(float, ...)` no `map(int, ...)`
- **Manejo de NaN**: Verificar con `.isna()` antes de usar `.idxmin()` o `.idxmax()`

## Testing

- Escribir tests unitarios para funciones de cálculo
- Validar que las fechas estén en rangos correctos
- Verificar que las coordenadas sean válidas (lat: -90 a 90, lon: -180 a 180)
- Comprobar que los offsets UTC sean coherentes
- Usar pytest para Python
- Tests en archivos separados: tests/test_*.py

## Idioma

- **Código**: Nombres de variables y funciones en inglés
- **Comentarios**: En español
- **Documentación**: En español
- **Logs/Mensajes**: En español
- **Commits**: En español
- **Docstrings**: En español

## Cuando añadas código nuevo:

1. Mantener consistencia con el código existente
2. Seguir PEP 8 sin excepciones
3. Documentar con docstrings completos
4. Usar OOP si el código lo amerita
5. Aplicar patrones de diseño cuando mejoren la arquitectura
6. Agregar comentarios explicativos para lógica compleja
7. Actualizar requirements.txt o packages.R si añades dependencias
8. Documentar en docs/ si es una feature importante
9. Sugerir tests si es lógica crítica
10. Validar con linters (pylint, flake8, black)

## Ejemplo de Clase Bien Documentada

```python
"""
Módulo para el análisis de escenarios horarios en España.

Este módulo proporciona clases y funciones para calcular y comparar
diferentes configuraciones horarias en ciudades españolas.
"""

from typing import Dict, List, Optional
from datetime import datetime
from abc import ABC, abstractmethod


class EscenarioHorario(ABC):
    """Clase base abstracta para representar un escenario horario.
    
    Esta clase define la interfaz común para todos los escenarios
    horarios que se pueden analizar en España.
    
    Attributes:
        nombre: Nombre descriptivo del escenario.
        descripcion: Descripción detallada del escenario.
    """
    
    def __init__(self, nombre: str, descripcion: str) -> None:
        """Inicializa un escenario horario.
        
        Args:
            nombre: Nombre del escenario.
            descripcion: Descripción del escenario.
        """
        self.nombre = nombre
        self.descripcion = descripcion
    
    @abstractmethod
    def obtener_utc_offset(self, fecha: datetime) -> int:
        """Obtiene el offset UTC para una fecha específica.
        
        Args:
            fecha: Fecha para la cual calcular el offset.
            
        Returns:
            Offset UTC en horas (puede ser negativo).
            
        Raises:
            ValueError: Si la fecha no es válida.
        """
        pass
    
    def _validar_fecha(self, fecha: datetime) -> bool:
        """Valida que una fecha sea correcta.
        
        Método privado para verificar que la fecha esté en rango válido.
        
        Args:
            fecha: Fecha a validar.
            
        Returns:
            True si la fecha es válida, False en caso contrario.
        """
        return fecha.year >= 1900 and fecha.year <= 2100
```

## Prioridades

1. **Código limpio y mantenible**: PEP 8, OOP, patrones de diseño
2. **Documentación exhaustiva**: Docstrings en todos los elementos
3. **Precisión científica**: Cálculos astronómicos correctos
4. **Claridad**: Código legible y bien estructurado
5. **Reproducibilidad**: Análisis fáciles de replicar
6. **Visualización**: Gráficos informativos y atractivos

## No hacer:

- No violar PEP 8 bajo ninguna circunstancia
- No dejar código sin documentar (ni siquiera métodos privados)
- No usar funciones procedurales cuando una clase sería mejor
- No ignorar patrones de diseño cuando simplifican el código
- No usar horarios en formato 12h (AM/PM)
- No asumir zona horaria local sin especificar
- No hardcodear rutas absolutas
- No mezclar idiomas en strings user-facing
- No ignorar manejo de errores en I/O
- No usar variables globales (preferir clases/singletons)
- No crear funciones con más de 50 líneas (refactorizar)
- No usar iconos/emoticonos en exceso en respuestas y comentarios

## Recursos

- PEP 8: https://peps.python.org/pep-0008/
- Google Python Style Guide: https://google.github.io/styleguide/pyguide.html
- Design Patterns: Gang of Four (GoF)
- SOLID Principles: Robert C. Martin
- Datos de IGN para coordenadas precisas
- Fórmulas astronómicas estándar
- Directiva UE 2000/84/CE para cambios horarios
- AEMET para datos climáticos de referencia

## Herramientas de Calidad de Código

### Python
- **flake8**: Verificación de estilo (OBLIGATORIO - se usa en el proyecto)
- **SonarQube**: Análisis de calidad y seguridad (OBLIGATORIO - se usa en el proyecto)
- **black**: Formateo automático
- **pylint**: Análisis estático
- **mypy**: Verificación de tipos
- **isort**: Ordenamiento de imports

### Configuración flake8
El proyecto usa flake8 con las siguientes configuraciones:
- Max line length: 88 (compatible con black)
- Ignorar: E203, W503 (compatibilidad con black)
- Excluir: .venv, __pycache__, *.egg-info

### Configuración SonarQube
- Análisis automático de código
- Detección de code smells
- Verificación de seguridad
- Cobertura de código

### Comandos útiles
```bash
# Verificar estilo con flake8
flake8 src/python/

# Formatear código
black src/python/

# Verificar tipos
mypy src/python/

# Análisis completo
pylint src/python/
```

### Reglas Específicas flake8/SonarQube
- No usar variables no utilizadas
- No dejar código comentado sin explicación
- Evitar funciones con más de 15 parámetros
- Mantener complejidad ciclomática baja (< 10)
- No duplicar código (DRY principle)
- Siempre cerrar recursos (archivos, conexiones)

## Control de Versiones y .gitignore

### Archivos Versionados
El proyecto usa Git con un `.gitignore` completo que cubre:
- **Python**: Entornos virtuales, `__pycache__`, `.pytest_cache`, archivos compilados
- **R**: `.Rhistory`, `.RData`, `.Rproj.user`
- **Quarto**: `.quarto/`, `_site/`, `*.quarto_ipynb`, `_freeze/`
- **IDEs**: `.vscode/` (excepto configuración específica), `.idea/`, archivos temporales
- **Sistema**: `.DS_Store`, `Thumbs.db`, archivos de sistema
- **Calidad**: `.sonar/`, `.scannerwork/`, reportes de coverage

### Archivos que SÍ se Versionan
- **Código fuente**: Todo en `src/`
- **Configuración**: `config/*.yaml`, `.flake8`, `sonar-project.properties`
- **Documentación**: `README.md`, `.github/copilot-instructions.md`, `docs/*.qmd`
- **Dependencias**: `requirements.txt`, `packages.R`, `setup.py`
- **Resultados finales**: CSV y PNG en `results/` (configurables según necesidad)

### Archivos que NO se Versionan
- **Entornos virtuales**: `.venv/`, `venv/`
- **Archivos temporales**: `*.tmp`, `*.log`, caches
- **Archivos IDE**: Configuraciones locales
- **Archivos intermedios Quarto**: `*.quarto_ipynb`, `_files/`
- **Datos sensibles**: `.env.local`, `secrets.yaml`
- **Archivos grandes**: Descomentar líneas en `.gitignore` para excluir CSVs grandes

### Buenas Prácticas Git
- **Commits descriptivos**: Usar mensajes en español describiendo el cambio
- **Commits atómicos**: Un cambio lógico por commit
- **Branches**: Crear ramas para features o experimentos
- **No commitear**: Archivos generados automáticamente (`.pyc`, `__pycache__`)
- **Revisar antes**: Usar `git status` y `git diff` antes de commitear
- **Ignorar sensibles**: Nunca versionar credenciales, tokens, o datos personales

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miguel-conde)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/miguel-conde)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
