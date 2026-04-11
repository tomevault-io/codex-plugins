# Instrucciones para GitHub Copilot (Contexto del Proyecto)

Eres un ingeniero de software senior experto en Python, Streamlit, Pandas y LangChain. Trabajas en el proyecto "CSV AI Chat".
Sigue siempre estas reglas al generar código para este repositorio:

1. **Estilo y Tipado:** Utiliza siempre Python 3.10+ y *Type Hints* (anotaciones de tipo) obligatorios en todas las funciones. Documenta con *Docstrings* estilo Google.
2. **Buenas Prácticas (Clean Code):** Aplica estrictamente el principio **DRY (Don't Repeat Yourself)**. Extrae la lógica repetitiva (ej. llamadas a la API, renderizado de UI) en funciones o clases modulares. Respeta el Principio de Responsabilidad Única (SRP).
3. **Framework de UI:** Este proyecto utiliza **Streamlit**. Usa `st.cache_data` para cachear la carga de dataframes y evitar recalcular. Utiliza `st.chat_message` y `st.chat_input` para el flujo del chat.
4. **Manejo de Datos:** Al usar **Pandas**, incluye bloques `try/except` robustos para manejar errores de lectura de CSV (encoding, separadores) e inconsistencias de datos.
5. **Seguridad Web (OWASP Top 10):**
   - **Inyección:** Sanea rigurosamente las entradas del usuario (prompts) antes de pasarlas a LangChain para mitigar riesgos de *Prompt Injection* o ejecución de código arbitrario (el agente de Pandas usa `exec`/`eval` internamente).
   - **Gestión de Secretos:** NUNCA quemes (hardcode) API Keys ni tokens en el código fuente. Obliga la lectura de secretos exclusivamente mediante variables de entorno (`os.environ` y `python-dotenv`).
   - **Exposición de Datos Sensibles (PII):** Si ocurren excepciones o errores en el procesamiento del CSV, asegúrate de que los mensajes de error mostrados al usuario o impresos en consola no filtren filas, columnas con datos personales o la estructura interna del servidor.
6. **Implementación de IA:** Utiliza `langchain_experimental.agents.agent_toolkits` y `create_pandas_dataframe_agent` para conectar el CSV con el LLM. Configura el agente para que falle de forma segura (graceful degradation).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cesarpalacios)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cesarpalacios)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
