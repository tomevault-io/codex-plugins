# Copilot Instructions for Work_Playground

## Project Overview
This project is a notebook-centric workspace for generating and managing metadata using LLMs (OpenAI, Google Gemini) and MongoDB. All main logic is implemented in Jupyter notebooks, with functions for data loading, sampling, LLM prompt construction, LLM invocation, and MongoDB upload.

## Key Conventions for AI Coding Agents

### 1. Data Loading and Sampling
- Use `pandas.read_csv` to load CSV files.
- Always sample up to 20 rows for LLM input (use all rows if fewer than 20).
- Use `df.to_json(orient='records', force_ascii=False)` to convert samples to JSON for LLMs.
- If a file is empty, return an empty string and print a warning.

### 2. LLM Integration
- Support both OpenAI (via `langchain_openai.ChatOpenAI`) and Google Gemini (via `google-genai`).
- Use a system prompt and a human prompt for metadata generation. See `metadata_generator_system_prompt` and `metadata_info_prompt` in `MetadataAgent.ipynb`.
- For OpenAI, set `model="gpt-4o"`, `temperature=0.2`, `max_tokens=2000` by default.
- For Gemini, use the latest available model (e.g., `gemini-2.0-flash`).
- Always extract JSON from LLM output using regex (e.g., `re.search(r'\{.*\}', ...)`).
- If LLM output is not valid JSON, print the raw response and error.

### 3. Error Handling
- Handle API errors (quota, rate limit, token limit, credential errors) gracefully.
- Print clear error messages for LLM and MongoDB operations.
- If LLM response cannot be parsed as JSON, print the extracted string and the error.
- If API keys are missing, print a warning and do not proceed.

### 4. Secrets and API Keys
- Always load API keys from `.env` using `dotenv.load_dotenv()`.
- Use `os.getenv('OPENAI_API_KEY')`, `os.getenv('GEMINI_API_KEY')`, and `os.getenv('MONGODB_URI')` as needed.
- Never hardcode secrets in code or notebooks.

### 5. MongoDB Integration
- Use `pymongo` to connect to MongoDB with `MONGODB_URI` from `.env`.
- Store all metadata in the `airspace` database, `metadata_full` collection.
- Use `insert_one()` to upload metadata JSON.
- Print the inserted document ID on success.

### 6. Notebook Workflow
- All main logic should be implemented in notebook cells.
- Functions for data loading, metadata generation, and upload should be reusable and well-documented.
- Use markdown cells to explain workflow and document assumptions.

### 7. Additional Info Handling
- If additional info (e.g., from `.txt` files) is provided, append it to the LLM prompt.
- Use the `add_info_prompt` template for incorporating extra context.

### 8. General Best Practices
- Keep LLM input size small by sampling data.
- Use snake_case for all identifiers in generated metadata.
- Always output only the final JSON from LLMs (no extra text or explanations).
- Document any new functions or workflows in markdown cells.

---

For any new features, follow the above conventions and update this file if project patterns change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JingYuan-Peh)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/JingYuan-Peh)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
