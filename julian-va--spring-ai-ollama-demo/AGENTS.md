# GitHub Copilot Instructions

## General Rules

1. **Commits**
    - Cuando GitHub Copilot realice un commit, **NO debe usar el usuario de Copilot**.
    - El commit debe realizarse siempre con **mi usuario configurado en Git**.

2. **Idioma**
    - **El código fuente siempre debe estar escrito en inglés**.
    - **Todas las respuestas de GitHub Copilot deben ser en español**, incluyendo explicaciones, sugerencias y
      previsualizaciones.

3. **Clases e Interfaces**
    - Cada vez que se cree una **clase, interfaz o enum**, se debe agregar **JavaDoc completo**.
    - El JavaDoc debe describir claramente el propósito de la clase o interfaz.

4. **Comentarios en el código**
    - **NO se deben agregar comentarios dentro del código** explicando métodos o lógica.
    - Las explicaciones de métodos o cambios deben mostrarse **únicamente en la previsualización** antes de que el
      usuario acepte el cambio.

5. **Previsualización**
    - Antes de aplicar cualquier cambio:
        - Explicar qué se va a modificar.
        - Explicar el propósito del código generado.
        - No incluir comentarios dentro del código final.

6. **Estructura de respuestas**
    - Cuando la respuesta incluya **pasos, opciones o posibles escenarios**, estos deben:
        - Estar **numerados**.
        - Ser claros y concisos.
        - Seguir un orden lógico.

7. **Buenas prácticas y seguridad en Java**
    - El código debe seguir **buenas prácticas de Java**, incluyendo:
        1. Principios SOLID.
        2. Encapsulamiento correcto.
        3. Uso adecuado de `final`, `Optional`, y manejo de excepciones.
        4. Validación de entradas para evitar vulnerabilidades.
        5. Prevención de problemas comunes de seguridad (inyecciones, exposición de datos sensibles, etc.).
    - Priorizar código **legible, mantenible y seguro**.

---

Estas instrucciones deben aplicarse **en todas las interacciones y sugerencias de código** realizadas por GitHub
Copilot.

---
> Source: [julian-va/spring-ai-ollama-demo](https://github.com/julian-va/spring-ai-ollama-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-26 -->
