## Project Overview
This project is a Python-based application for deploying an AI chatbot with useful capabilities like web search and memory retention inside a user-friendly UI.  

### Golden Rules  
* Keep it simple.   
* When running code, always use `uv` as launcher and not `python` or `python3`.  e.g., `uv run main.py`.  
* Only use try-except blocks when necessary and to catch specific exceptions.  
* Always se the `feature-dev` skill when developing new features.  
* When debugging follow these steps: Reproduce the bug -> Find the root cause -> Apply possible fix -> Validate the error is fixed  

### Coding Style Standards  
* Never, under any circumstance, use emojis.    
* Functions/methods: Always include a numpy-style docstring briefly describing its purpose and parameters, along with type hints for parameters and return values. No need to over-explain obvious functionality.  
* Use `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants and `snake_case` for constants.  
* Always use **logger** from `loguru` for logging with its appropriate level of urgency instead of generic `print` statements.  
* Keep imports at the file's top level and sort the following way from top to bottom with one line in between: standard libraries, third-party, local imports.  

---
> Source: [luisfg10/AI-Chatbot-Assistant](https://github.com/luisfg10/AI-Chatbot-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
