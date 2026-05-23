# Building an AI GitHub Repo Summarizer — Learning Log
> A beginner's complete guide to MCP, FastAPI, Streamlit, and AI APIs

---

## What I Built

An **AI-powered GitHub Repo Summarizer** that:
- Takes any public GitHub URL as input
- Fetches the README, file structure, and key source files
- Sends that context to an AI (Groq/LLaMA)
- Returns a beginner-friendly summary, folder explanation, and code breakdown

**Tech Stack:** Streamlit · FastAPI · Groq API · GitHub API · Python

---

## Project Structure

```
repo-assistant/
│
├── frontend/
│   └── app.py              # Streamlit UI — what the user sees
│
├── backend/
│   ├── main.py             # FastAPI server — central hub
│   ├── github_service.py   # Talks to GitHub API
│   ├── prompt_builder.py   # Builds the AI prompt
│   ├── grok_client.py      # Talks to Groq AI API
│   └── repo_reader.py      # Picks the best file to show AI
│
├── repos/
├── .env                    # API keys (never commit this!)
└── requirements.txt
```

---

## Core Concepts Learned

### 1. What is MCP (Model Context Protocol)?

MCP is a pattern for giving AI models a structured **toolbox** to interact with the real world.

Instead of manually copy-pasting code into an AI chat, MCP lets the AI **call tools itself**:

```
User asks → AI decides which tool to use → Tool fetches data → AI answers
```

**In this project, our "MCP tools" were:**

| Tool | What it does |
|------|-------------|
| `get_readme()` | Fetches the README from GitHub |
| `get_file_tree()` | Gets the full folder/file structure |
| `get_file_content()` | Reads a specific file's contents |
| `get_best_sample_file()` | Picks the most useful file for the AI |

**Key insight:** Your backend *is* the MCP server. It exposes tools. The AI uses them through prompts. That's the whole concept at its core.

---

### 2. FastAPI — The Backend Brain

FastAPI is a Python web framework for building APIs quickly.

**How it works:**
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class RepoRequest(BaseModel):   # Defines what data we accept
    url: str

@app.post("/analyze")           # Listens for POST requests at /analyze
def analyze_repo(request: RepoRequest):
    # do stuff
    return {"result": "..."}    # Returns JSON
```

**Key concepts:**
- `@app.post("/analyze")` — defines a route/endpoint
- `BaseModel` — validates incoming data automatically (Pydantic)
- Returns a Python dict → FastAPI converts it to JSON automatically
- Run with: `uvicorn main:app --reload`

**Why FastAPI over Flask?**
FastAPI gives automatic data validation, auto-generated docs at `/docs`, and is faster out of the box.

---

### 3. Streamlit — The Frontend

Streamlit turns Python scripts into web apps with zero HTML/CSS needed.

```python
import streamlit as st

st.title("My App")                        # Big heading
st.text_input("Enter URL")               # Input box
st.button("Click me")                    # Button
st.markdown("**Bold text**")             # Markdown rendering
st.spinner("Loading...")                 # Loading animation
st.error("Something broke")             # Red error box
st.success("It worked!")                # Green success box
```

**How frontend talks to backend:**
```python
import httpx

response = httpx.post(
    "http://localhost:8000/analyze",     # FastAPI endpoint
    json={"url": url},                   # Send data as JSON
    timeout=60
)
data = response.json()                   # Parse response
```

**Key insight:** Streamlit reruns the entire script top-to-bottom every time the user interacts. Keep state in `st.session_state` if needed.

---

### 4. GitHub API — Fetching Repo Data

GitHub exposes a free REST API to read public repo data without cloning.

**Useful endpoints:**

```python
# Get README content
GET https://api.github.com/repos/{owner}/{repo}/readme

# Get full file tree (all files recursively)
GET https://api.github.com/repos/{owner}/{repo}/git/trees/HEAD?recursive=1

# Get a specific file's content
GET https://api.github.com/repos/{owner}/{repo}/contents/{path}
```

**No API key needed for public repos** (but rate-limited to 60 req/hour unauthenticated).

**Parsing messy GitHub URLs:**
```python
# Always strip query params and extra path segments
url = url.split("?")[0].rstrip("/")
parts = url.split("/")
gh_index = parts.index("github.com")
owner = parts[gh_index + 1]
repo  = parts[gh_index + 2]
```

This handles URLs like:
- `https://github.com/owner/repo` ✅
- `https://github.com/owner/repo/tree/main?search=1` ✅

---

### 5. Groq API — Talking to AI

Groq is a free, fast AI inference service that runs open-source models like LLaMA 3.

**It uses the same format as OpenAI** — so skills transfer directly.

```python
import httpx

headers = {
    "Authorization": f"Bearer {GROQ_API_KEY}",
    "Content-Type": "application/json"
}

body = {
    "model": "llama3-8b-8192",
    "messages": [
        {"role": "user", "content": "Your prompt here"}
    ],
    "temperature": 0.7   # 0 = precise, 1 = creative
}

response = httpx.post(url, json=body, headers=headers, timeout=30)
answer = response.json()["choices"][0]["message"]["content"]
```

**Get your free key at:** [console.groq.com](https://console.groq.com)

**Why Groq over Grok (xAI)?**
- Groq has a generous free tier
- Faster response times
- Simpler setup — just sign up and go

---

### 6. Prompt Engineering — Talking to AI Effectively

The quality of your AI output depends entirely on the quality of your prompt.

**Structure used in this project:**
```python
prompt = f"""
You are a helpful assistant that explains GitHub repositories to beginners.

--- README ---
{readme[:3000]}

--- FILE STRUCTURE ---
{file_list}

--- SAMPLE CODE ---
{sample_code[:2000]}

Please provide:
1. A simple summary (2-3 sentences)
2. The folder structure explained simply
3. What the main code does, in plain English

Keep your explanation beginner-friendly.
"""
```

**Prompt engineering tips:**
- Give the AI a **role** ("You are a...")
- Clearly **separate sections** with headers (---)
- Give **specific instructions** on what to output
- Set the **tone** ("beginner-friendly")
- **Limit input size** — don't dump 10,000 lines of code

---

### 7. Environment Variables & `.env` Files

Never hardcode API keys in your code. Use `.env` files instead.

```bash
# .env file
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

```python
# In Python
from dotenv import load_dotenv
from pathlib import Path
import os

# Explicitly point to .env file location
dotenv_path = Path(__file__).parent / ".env"
load_dotenv(dotenv_path=dotenv_path)

API_KEY = os.getenv("GROQ_API_KEY")
```

**Rules:**
- Add `.env` to `.gitignore` — never commit it
- Use `python-dotenv` to load it
- Always validate the key exists before using it

---

### 8. Error Handling — Writing Defensive Code

Always expect things to fail, especially with external APIs.

**Pattern used:**
```python
# In backend — raise meaningful HTTP errors
from fastapi import HTTPException

if not file_tree:
    raise HTTPException(status_code=404, detail="Repo not found or is private")

# In AI client — check response before accessing keys
if "error" in result:
    raise ValueError(f"API error: {result['error']}")

if "choices" not in result:
    raise ValueError(f"Unexpected response: {result}")
```

```python
# In frontend — show user-friendly messages
if response.status_code != 200:
    error_detail = response.json().get("detail", "Unknown error")
    st.error(f"Backend error: {error_detail}")
```

**Always print raw API responses during development:**
```python
print("=== RAW RESPONSE ===")
print(result)
```

---

### 9. Building a File Tree from Flat Paths

GitHub returns files as a flat list:
```
["folder/subfolder/file.py", "README.md", "folder/other.txt"]
```

Converting to a nested tree using `setdefault`:
```python
def build_tree(paths: list) -> dict:
    tree = {}
    for path in paths:
        parts = path.split("/")
        node = tree
        for part in parts:
            node = node.setdefault(part, {})  # Creates key if missing
    return tree
```

Then render it recursively with icons:
```python
def render_tree(tree: dict, indent: int = 0):
    for name, children in sorted(tree.items()):
        is_folder = bool(children)
        prefix = "　" * indent
        if is_folder:
            st.markdown(f"{prefix}📂 **{name}**")
            render_tree(children, indent + 1)
        else:
            st.markdown(f"{prefix}📄 {name}")
```

---

## Common Bugs & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `NameError: response is not defined` | Used variable before assigning it | Always assign `response = httpx.get(...)` before using `response.status_code` |
| `KeyError: 'choices'` | AI API returned an error, not a valid response | Print raw response, check for `"error"` key first |
| `GROQ_API_KEY is missing` | `.env` not found or wrong path | Use `Path(__file__).parent / ".env"` to explicitly point to it |
| `Expecting value: line 1 column 1` | Backend returned empty/non-JSON response | Add proper error handling in FastAPI, check status codes |
| URL parsing breaks | GitHub URL has `/tree/main?search=1` appended | Strip query params with `.split("?")[0]` first |

---

## How to Run the Project

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Start backend (Terminal 1)
cd backend
uvicorn main:app --reload
# Backend runs at http://localhost:8000
# Auto-generated API docs at http://localhost:8000/docs

# 3. Start frontend (Terminal 2)
cd frontend
streamlit run app.py
# Frontend runs at http://localhost:8501
```

---

## The Full Request Flow (What Happens When You Click "Analyze")

```
1. User pastes GitHub URL in Streamlit UI
        ↓
2. Streamlit sends POST request to FastAPI /analyze
        ↓
3. FastAPI parses URL → extracts owner & repo name
        ↓
4. GitHub API calls (MCP Tools):
   ├── get_readme()        → fetches README.md
   ├── get_file_tree()     → fetches all file paths
   └── get_best_sample_file() → picks main .py/.js file
        ↓
5. prompt_builder assembles everything into one prompt
        ↓
6. Groq API receives prompt → LLaMA 3 generates response
        ↓
7. FastAPI returns JSON: { summary, file_tree, repo }
        ↓
8. Streamlit renders: AI summary + visual file tree
```

---

## Interview Talking Points

**"What is MCP?"**
> MCP (Model Context Protocol) is a standard way to give AI models access to external tools and data sources. Instead of the user manually providing context, the AI can call defined tools — like fetching a file, querying a database, or calling an API — to gather what it needs before responding.

**"How does your frontend talk to your backend?"**
> The Streamlit frontend sends an HTTP POST request using `httpx` to a FastAPI endpoint. FastAPI validates the incoming data using Pydantic models, processes it, and returns a JSON response that Streamlit renders.

**"What is an API endpoint?"**
> An endpoint is a URL your server listens on. When someone sends a request to that URL (like POST `/analyze`), the server runs a specific function and returns a response.

**"Why use environment variables?"**
> To keep secrets like API keys out of your code. If you hardcode keys and push to GitHub, anyone can steal them. `.env` files store secrets locally and are never committed to version control.

**"What is prompt engineering?"**
> Crafting the input text to an AI model carefully so it produces useful, accurate, structured output. It involves giving context, a role, clear instructions, and formatting the data cleanly.

---

## What to Build Next

- **Chat history** — let users ask follow-up questions about the repo
- **File picker** — let users choose which file to analyze
- **True MCP server** — use the official `mcp` Python library to register tools formally
- **GitHub auth** — add a GitHub token to handle private repos and higher rate limits
- **Multiple repo comparison** — analyze two repos side by side

---

*Built with: Python · FastAPI · Streamlit · Groq API · GitHub API*
*Concept: Model Context Protocol (MCP) for structured AI tool use*
