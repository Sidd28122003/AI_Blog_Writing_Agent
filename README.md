# 📝 Blog Writing Agent

A multi-agent blog generation system powered by **LangGraph**, **Mistral AI**, **Tavily**, and **Gemini**, with a **Streamlit** frontend. Enter any topic and the agent autonomously researches, plans, writes, and illustrates a full blog post in Markdown.

---

## 🧠 How It Works

The system is built as a **LangGraph state machine** with the following pipeline:

```
topic ──► Router ──► (Research?) ──► Orchestrator ──► Workers (parallel)
                                                              │
                                                              ▼
                                              Reducer Subgraph:
                                              merge_content → decide_images → generate_and_place_images
                                                              │
                                                              ▼
                                                       Final Blog (.md)
```

### Agent Nodes

| Node | Role |
|---|---|
| **Router** | Decides if web research is needed (`closed_book` / `hybrid` / `open_book`) and sets recency window |
| **Research** | Runs Tavily searches, synthesizes and deduplicates `EvidenceItem` objects |
| **Orchestrator** | Produces a structured `Plan` (blog title, audience, tone, kind) with a list of section `Task`s |
| **Worker** | One worker per section — writes Markdown using the plan, task details, and evidence |
| **Reducer (subgraph)** | Merges sections → decides image placement → generates images via Gemini |

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd <repo-folder>
```

### 2. Install Dependencies

```bash
pip install -r requirement.txt
```

> **Note:** The Gemini image generation feature also requires `google-genai`:
> ```bash
> pip install google-genai
> ```

### 3. Set Up Environment Variables

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_api_key
TAVILY_API_KEY=your_tavily_api_key        # Optional — skipped if not set
GOOGLE_API_KEY=your_google_api_key        # Optional — for AI image generation
```

- **`MISTRAL_API_KEY`** — Required. Powers all LLM reasoning (routing, orchestration, writing).
- **`TAVILY_API_KEY`** — Optional. Enables live web research for `hybrid` and `open_book` topics. If absent, research is silently skipped.
- **`GOOGLE_API_KEY`** — Optional. Powers AI image generation via `gemini-2.5-flash-image`. If absent, image placeholders show a fallback block instead.

### 4. Run the App

```bash
streamlit run bwa_frontend.py
```

---

## 📁 Project Structure

```
├── bwa_backend.py       # LangGraph graph definition — all agent nodes & schemas
├── bwa_frontend.py      # Streamlit UI — sidebar, tabs, streaming, download
├── requirement.txt      # Python dependencies
├── images/              # Auto-created — stores generated blog images
└── *.md                 # Auto-saved blog outputs (one file per blog)
```

---

## 🖥️ UI Overview

The Streamlit interface has a sidebar and five tabs:

**Sidebar**
- Enter a topic and an "as-of" date, then click **🚀 Generate Blog**
- A **Past blogs** panel lists previously generated `.md` files — click any to reload

**Tabs**

| Tab | Contents |
|---|---|
| 🧩 **Plan** | Structured plan: title, audience, tone, blog kind, and a task table |
| 🔎 **Evidence** | Sourced URLs and publication dates used for grounded claims |
| 📝 **Markdown Preview** | Rendered blog with inline images; download as `.md` or `.zip` bundle |
| 🖼️ **Images** | Generated image specs and previews; download as `images.zip` |
| 🧾 **Logs** | Full streaming event log for debugging |

---

## ✨ Key Features

- **Smart routing** — automatically picks `closed_book`, `hybrid`, or `open_book` mode based on topic volatility
- **Parallel section writing** — worker nodes run concurrently via LangGraph's `Send` API (fan-out)
- **Grounded citations** — `open_book` mode enforces citation-only claims with Markdown source links
- **AI image generation** — Gemini generates technical diagrams and inserts them into the blog
- **Graceful fallbacks** — missing API keys or image generation failures degrade gracefully without crashing
- **Blog history** — previously generated blogs are auto-saved as `.md` and reloadable from the sidebar
- **Bundle download** — export the full blog + images as a single `.zip`

---

## 🔧 Blog Kinds Supported

| Kind | Description |
|---|---|
| `explainer` | Evergreen concept breakdowns |
| `tutorial` | Step-by-step how-to guides with code |
| `news_roundup` | Recent events and implications (open_book) |
| `comparison` | Side-by-side technology comparisons |
| `system_design` | Architecture deep-dives |

---

## 📦 Dependencies

| Package | Purpose |
|---|---|
| `streamlit` | Web UI |
| `langgraph` | Multi-agent graph orchestration |
| `langchain` | LLM utilities and chains |
| `langchain-mistralai` | Mistral AI LLM integration |
| `langchain-core` | Core message/schema primitives |
| `langchain-google-genai` | Google GenAI integration |
| `tavily-python` | Web search for research nodes |
| `python-dotenv` | `.env` file loading |

---

## 🗒️ Notes

- Generated blog files are saved as `<slug>.md` in the working directory automatically after each run.
- Images are saved under `images/<filename>.png`.
- The research node filters results by recency: `open_book` = last 7 days, `hybrid` = last 45 days, `closed_book` = up to 10 years.
- The image planner caps output at **3 images per blog** to keep generation focused and fast.
