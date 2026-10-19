# UdaPlay – AI Research Agent for Video Games

UdaPlay is an AI research agent for the video game industry, built for the Udacity *Building Agents* final project.

## What it does

- **RAG over a local game dataset** – 15 games (name, platform, genre, publisher, year, description) are embedded with OpenAI embeddings and stored in a persistent ChromaDB collection (`udaplay`).
- **LLM evaluation** – an LLM-as-judge (`evaluate_retrieval`) decides, with a structured Pydantic result (useful / explanation / confidence), whether the retrieved documents are enough to answer the question.
- **Web fallback with citations** – only when the judge says the internal data is not enough, the agent searches the web with Tavily (`game_web_search`) and cites the source URLs.
- **Stateful agent with a state machine** – the agent (`lib/agents.py`) runs on a state machine (`message_prep → llm_processor ⇄ tool_executor → termination`) and keeps per-session conversation memory, so follow-up questions such as "Which platform was it released on?" work.

Workflow for every question: `retrieve_game` → `evaluate_retrieval` → `game_web_search` (only if needed) → structured answer (answer, details, source, path, confidence).

## Setup

Requires Python 3.11+.

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install "chromadb>=1.0.4" "openai>=1.73.0" "pydantic>=2.11.3" "python-dotenv>=1.1.0" "tavily-python>=0.5.4" "pdfplumber>=0.11.6" jupyter ipykernel nbconvert
```

Copy `.env.example` to `.env` and add your keys:

```
OPENAI_API_KEY="YOUR_KEY"
OPENAI_BASE_URL="https://openai.vocareum.com/v1"
TAVILY_API_KEY="YOUR_KEY"
```

`.env` is ignored by git and must never be committed.

## How to run

Run the notebooks in order from the project folder:

1. `Udaplay_01_starter_project.ipynb` – Part 1: builds the `chromadb/` vector database and demonstrates semantic search.
2. `Udaplay_02_starter_project.ipynb` – Part 2: loads the same database and runs the agent.

```bash
jupyter nbconvert --to notebook --execute --inplace Udaplay_01_starter_project.ipynb
jupyter nbconvert --to notebook --execute --inplace Udaplay_02_starter_project.ipynb
```

## Project structure

```
.
├── games/                              # 15 JSON files, one per game
├── lib/                                # Helper modules (agent, state machine, LLM, tools, memory, vector DB, ...)
├── Udaplay_01_starter_project.ipynb    # Part 1 – RAG pipeline
├── Udaplay_02_starter_project.ipynb    # Part 2 – Agent, example queries and report
├── .env.example                        # Template for API keys
└── README.md
```

## Example outputs

The example queries are in **`Udaplay_02_starter_project.ipynb`** (saved with outputs):

- When was Pokémon Gold and Silver released? – answered from the internal database
- Which one was the first 3D platformer Mario game? – internal
- Was Mortal Kombat X released for PlayStation 5? – not in the dataset, so it uses the web fallback with a cited URL
- Who published Halo Infinite and in which year was it released? – internal
- A follow-up question in the same session ("Which platform was it released on, and who published it?") to show conversation memory

Each query prints a trace (tool calls in order, the evaluation verdict, whether the web was used, and the final answer). A **Report** section at the end of the notebook summarizes the path, answer, source and confidence for every query.
