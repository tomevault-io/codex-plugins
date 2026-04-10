## 📘 Project Overview:

### **AI Virtual Assistant for Student Q\&A Using LangChain**

This project is a **lightweight AI-powered assistant** designed to enhance the **learning experience of students in higher education**. Users can ask questions in natural language based on uploaded study materials (PDFs), and receive intelligent, context-aware answers directly sourced from the selected topic.

Built with **LangChain**, **Chroma**, and **OpenAI**, the system leverages large language models and a retrieval-augmented generation (RAG) architecture to provide accurate, conversational answers through a minimal Streamlit interface.

The assistant supports:

- Ingesting and indexing lecture PDFs using **Chroma** for efficient semantic search.
- Real-time Q\&A powered by OpenAI’s GPT model.
- Streamlit UI for easy interaction.

> Ideal for test prep, revision, or flipped classroom models.

## ✅ Objectives:

- Boost **student engagement** and **comprehension**.
- Provide an always-available **academic Q\&A tool**.
- Demonstrate RAG with **LangChain + Chroma + OpenAI**.

## ✅ Benefits:

- **Quick development** using modern LLM tooling.
- **Runs locally**—just Python, internet, and a laptop.
- Ideal for **2-week hackathons, demos, or prototypes**.

## ✅ **2. Key Features (Updated Scope)**

- **Admin topic creation and PDF upload**
- **Student topic selection and Q\&A**
- **SQLite database** for managing users and chat sessions
- Continued use of **LangChain**, **Chroma**, and **OpenAI**

### ✅ Responsibilities Breakdown

| Feature          | Responsibility                           | Implemented By                    |
| ---------------- | ---------------------------------------- | --------------------------------- |
| **PDF Parsing**  | Break PDF into clean text chunks         | `load_and_split_pdf()`            |
| **Vector Store** | Embed & store by topic ID                | `create_chroma_index_for_topic()` |
| **LLM Chain**    | Ask topic-specific questions             | `build_qa_chain()`                |
| **Admin Portal** | Create topic, upload study material      | `admin_topic_upload_ui()`         |
| **Student UI**   | Select topic, ask questions              | `student_qa_ui()`                 |
| **SQLite DB**    | Manage users, topics, sessions, messages | `setup_sqlite_database()`         |

## ✅ **4. LangChain + Chroma + OpenAI Integration**

**`document_loader.py`**

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import CharacterTextSplitter

def load_and_split_pdf(file_path, chunk_size=500, chunk_overlap=50):
    loader = PyPDFLoader(file_path)
    pages = loader.load()
    splitter = CharacterTextSplitter(chunk_size=chunk_size, chunk_overlap=chunk_overlap)
    return splitter.split_documents(pages)
```

**`vector_store.py`**

```python
from langchain.vectorstores import Chroma
from langchain.embeddings.openai import OpenAIEmbeddings

PERSIST_DIR = "chroma_db"

def create_chroma_index(chunks, persist_directory=PERSIST_DIR):
    embeddings = OpenAIEmbeddings()
    vectorstore = Chroma.from_documents(documents=chunks, embedding=embeddings, persist_directory=persist_directory)
    vectorstore.persist()

def load_chroma_retriever(persist_directory=PERSIST_DIR):
    embeddings = OpenAIEmbeddings()
    vectorstore = Chroma(persist_directory=persist_directory, embedding_function=embeddings)
    return vectorstore.as_retriever()
```

**`qa_chain.py`**

```python
from langchain.chains import RetrievalQA
from langchain.chat_models import ChatOpenAI

def build_qa_chain(retriever):
    llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
    return RetrievalQA.from_chain_type(llm=llm, retriever=retriever)
```

## Copilot assumptions

- Always assume the dev server is running and avoid starting the server

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/myckhel)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/myckhel)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
