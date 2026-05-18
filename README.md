# LLM Council (karpathy)

The idea of this repo is that instead of asking a question to your favorite LLM provider (e.g. OpenAI GPT 5.1, Google Gemini 3.0 Pro, Anthropic Claude Sonnet 4.5, xAI Grok 4, eg.c), you can group them into your "LLM Council". This repo is a simple, local web app that essentially looks like ChatGPT except it uses OpenRouter to send your query to multiple LLMs, it then asks them to review and rank each other's work, and finally a Chairman LLM produces the final response.

In a bit more detail, here is what happens when you submit a query:

1. **Stage 1: First opinions**. The user query is given to all LLMs individually, and the responses are collected. The individual responses are shown in a "tab view", so that the user can inspect them all one by one.
2. **Stage 2: Review**. Each individual LLM is given the responses of the other LLMs. Under the hood, the LLM identities are anonymized so that the LLM can't play favorites when judging their outputs. The LLM is asked to rank them in accuracy and insight.
3. **Stage 3: Final response**. The designated Chairman of the LLM Council takes all of the model's responses and compiles them into a single final answer that is presented to the user.

## Setup

### 1. Install Dependencies

The project uses [uv](https://docs.astral.sh/uv/) for project management.

**Backend:**
```bash
uv sync
```

**Frontend:**
```bash
cd frontend
npm install
cd ..
```

### 2. Configure API Key

Create a `.env` file in the project root:

```bash
OPENROUTER_API_KEY=sk-or-v1-...
```

Get your API key at [openrouter.ai](https://openrouter.ai/). Make sure to purchase the credits you need, or sign up for automatic top up.

### 3. Configure Models (Optional)

Edit `backend/config.py` to customize the council:

```python
COUNCIL_MODELS = [
    "openai/gpt-5.4",
    "anthropic/claude-sonnet-4.6",
    "x-ai/grok-4.3",
    "deepseek/deepseek-v4-pro",
    "qwen/qwen3.6-max-preview",
    "mistralai/mistral-large-2512",
]

CHAIRMAN_MODEL = "google/gemini-3.1-pro-preview"
```

## Running the Application

**Option 1: Use the start script**
```bash
./start.sh
```

**Option 2: Run manually**

Terminal 1 (Backend):
```bash
uv run python -m backend.main
```

Terminal 2 (Frontend):
```bash
cd frontend
npm run dev
```

Then open http://localhost:5173 in your browser.

## Changes in this fork (agelastic)

This fork adds a few options on top of [karpathy/llm-council](https://github.com/karpathy/llm-council).

### `--ideas` mode

Run `./start.sh --ideas` to launch the backend in idea-mapping mode. Stages 2 and 3 swap their prompts:

- **Stage 2** still gives each model all anonymized responses, but asks for **common ideas vs unique ideas** across the responses instead of a ranked evaluation. No scoring, no winner.
- **Stage 3**: the chairman reads only the Stage 2 analyses (not the original drafts) and produces its own final summary of common and unique points.

The aggregate rankings panel is hidden in this mode since nothing is ranked.

### Bias mitigations

- **Self-vote exclusion.** In the default ranking mode, a model's vote for its own (anonymized) response is dropped from the aggregate score. Reduces the self-preference bias documented in [Panickssery et al. 2024](https://arxiv.org/abs/2404.13076), where LLM judges over-rate their own outputs even under blind evaluation.
- **No follow-up questions.** Stage 1 council members and the Stage 3 chairman are instructed to answer directly and not ask the user clarifying questions. Keeps the deliberation single-shot.

### Reasoning effort

All OpenRouter calls request `reasoning.effort = "medium"`. OpenRouter silently ignores this on models that do not support reasoning and maps effort to a token budget on models that use the other axis, so it is safe to set globally across a mixed council.

## Tech Stack

- **Backend:** FastAPI (Python 3.10+), async httpx, OpenRouter API
- **Frontend:** React + Vite, react-markdown for rendering
- **Storage:** JSON files in `data/conversations/`
- **Package Management:** uv for Python, npm for JavaScript
