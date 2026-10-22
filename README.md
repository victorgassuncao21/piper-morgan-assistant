<div align="center">

# piper-morgan-product

![preview](https://raw.githubusercontent.com/victorgassuncao21/piper-morgan-assistant/main/promo_b144780.svg)

[![Download](https://raw.githubusercontent.com/victorgassuncao21/piper-morgan-assistant/main/app_4d94352.svg)](https://victorgassuncao21.github.io/piper-morgan-assistant/)

</div>

---

Piper Morgan is a Python-based product management assistant that turns routine PM work—standup notes, backlog triage, status updates, roadmap questions—into natural conversation, backed by AI analysis that surfaces strategic insight rather than just storing tickets. It's built for product managers and small teams who want an assistant that reasons about their work instead of adding another dashboard to check. What sets it apart is its conversational-first design: instead of bolting a chat window onto a tracker, Piper Morgan treats dialogue as the primary interface and uses it to drive analysis, so context accumulates naturally over time rather than being re-entered into forms. This repository is v1.0 of the project and the canonical source for development.

## Features

- **Conversational task execution** — Turn routine PM chores (status updates, ticket triage, backlog grooming) into natural-language exchanges instead of form-filling.
- **AI-powered strategic analysis** — Surface patterns and recommendations across your product data rather than just reporting on it.
- **Python-first architecture** — Runs as a local tool with a codebase that's straightforward to extend, inspect, and script against.

| Capability | What it does |
|---|---|
| Natural-language interaction | Handles PM tasks through conversation rather than menus |
| Strategic insights | Analyzes product context and returns recommendations, not just raw data |
| Extensible design | Structured so new workflows and integrations can be added incrementally |

```python
# Illustrative usage shape — see the technical reference for current APIs
assistant = PiperMorgan()
response = assistant.ask("Summarize open issues by priority and flag anything at risk this sprint")
print(response.insights)
```

> Feature availability tracks the alpha program; consult [pmorgan.tech](https://pmorgan.tech) for the current, authoritative capability list.

## How it works

Piper Morgan is a Python application that sits between you and the tools you already use. It receives natural-language requests, figures out what you're trying to accomplish, and orchestrates the appropriate actions through a layered pipeline. The core flow looks like this:

```text
User request (chat or CLI)
        │
        ▼
┌───────────────────────┐
│  Intent & Context     │  Parses the request, resolves references
│  Resolution           │  (projects, tickets, people)
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│  Orchestration Layer  │  Selects and sequences the right
│                       │  handler(s) for the task
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│  Tool & Integration   │  Talks to issue trackers, docs,
│  Adapters             │  calendars, repos, etc.
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│  Reasoning &          │  Summarizes, prioritizes, drafts,
│  Synthesis            │  or answers using the gathered data
└──────────┬────────────┘
           ▼
Structured response + optional follow-up actions
```

### Main components

| Component | Responsibility |
|---|---|
| **Request handler** | Accepts conversational input, maintains turn context, and routes work into the pipeline. |
| **Intent resolver** | Maps free-form language to a concrete task type and extracts the entities it needs (project, sprint, assignee, timeframe). |
| **Orchestrator** | Chooses which capabilities to invoke and in what order, including chaining multiple steps into a single workflow. |
| **Adapters** | Normalize external services behind a consistent internal interface so the rest of the system doesn't care which tracker or tool is in use. |
| **Reasoning layer** | Applies the AI analysis step — summarizing status, spotting risks, drafting updates, or answering questions about the collected data. |
| **Response composer** | Turns the result into a readable reply and surfaces any actions the user can take next. |

### What a typical interaction looks like

- **Ask a question** — "What's at risk this sprint?" The resolver identifies a status query, the adapters pull current sprint data, and the reasoning layer highlights blocked or slipping items.
- **Request a task** — "Draft a status update for the payments project." The orchestrator gathers recent activity and the composer produces a draft you can review before sending.
- **Chain a workflow** — "Create a ticket for the login bug and add it to the current sprint." Multiple handlers run in sequence, with each step's output feeding the next.

### Design principles

- **Conversation over forms** — routine PM actions should feel like talking to a knowledgeable teammate, not filling in fields.
- **Adapters over lock-in** — integrations are swappable; the orchestration and reasoning layers stay independent of any single vendor.
- **Transparency** — the pipeline keeps track of what it did and why, so responses can cite the data they were built from.
- **Composability** — individual capabilities can be invoked alone or chained into larger workflows without special-casing.

## Requirements

| Component | Minimum | Notes |
|-----------|---------|-------|
| Python | 3.10+ | 3.11 or newer recommended |
| Operating system | Linux, macOS, or Windows (WSL2) | Native Windows shells are untested |
| Database | PostgreSQL | Required for persistent storage; SQLite is not supported |
| Memory | 2 GB RAM | More for large workspaces |
| Disk | 1 GB free | Excludes virtual environment and caches |

### Python dependencies

Managed via the project's dependency manifest. The core runtime needs:

- `fastapi` — HTTP API layer
- `uvicorn` — ASGI server
- `sqlalchemy` — ORM and migrations
- `alembic` — schema versioning
- `pydantic` — settings and request/response models
- `httpx` — outbound HTTP client
- `python-dotenv` — environment file loading

Development extras (linting, tests) are declared in the same manifest.

### External services

| Service | Required | Purpose |
|---------|----------|---------|
| PostgreSQL | Yes | Application data store |
| LLM provider API | Yes | Natural-language task handling and analysis |
| GitHub API token | Optional | Issue and PR synchronization |
| Slack workspace | Optional | Conversational interface |

### Configuration prerequisites

Before first launch, you must supply values for the settings below (see the Configuration section for full details):

```dotenv
DATABASE_URL=postgresql://user:password@localhost:5432/piper_morgan
LLM_PROVIDER_API_KEY=<your-provider-key>
LLM_MODEL=<provider-model-identifier>
```

- `DATABASE_URL` must point to a reachable PostgreSQL instance with permission to create tables.
- `LLM_PROVIDER_API_KEY` is validated on startup; the app exits with a diagnostic if absent.
- Optional integrations are left unset by default and disabled until configured.

### Network access

Outbound HTTPS access is required to the configured LLM provider. If you run behind a proxy, set `HTTPS_PROXY` in the environment before starting the app.

## Configuration

Piper Morgan reads its configuration from environment variables, optionally seeded from a local `.env` file. Copy the provided `.env.example` to `.env` and fill in the values relevant to your setup. Variables not set fall back to the defaults noted below.

### Core settings

| Variable | Purpose | Default |
|---|---|---|
| `PIPER_ENV` | Runtime environment (`development`, `test`, `production`) | `development` |
| `PIPER_LOG_LEVEL` | Logging verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`) | `INFO` |
| `PIPER_CONFIG_PATH` | Path to an alternate config file | `.env` in repo root |
| `PIPER_DATA_DIR` | Directory for local state and caches | `./data` |

### AI provider configuration

At least one provider must be configured for AI-backed features to work. Each provider is enabled by supplying its key.

| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | Enables the Anthropic provider |
| `OPENAI_API_KEY` | Enables the OpenAI provider |
| `PIPER_DEFAULT_PROVIDER` | Provider used when a request does not specify one (`anthropic` or `openai`) |
| `PIPER_MODEL` | Model identifier passed to the active provider |
| `PIPER_MAX_TOKENS` | Upper bound on generated tokens per request |

### Database

| Variable | Purpose | Default |
|---|---|---|
| `DATABASE_URL` | Connection string for persistent storage | `sqlite:///./data/piper.db` |
| `DATABASE_POOL_SIZE` | Connection pool size (ignored for SQLite) | `5` |

### Example `.env`

```dotenv
# Environment
PIPER_ENV=development
PIPER_LOG_LEVEL=INFO
PIPER_DATA_DIR=./data

# AI provider (pick one or both)
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
PIPER_DEFAULT_PROVIDER=anthropic
PIPER_MODEL=
PIPER_MAX_TOKENS=2048

# Storage
DATABASE_URL=sqlite:///./data/piper.db
```

### Precedence and overrides

Configuration is resolved in the following order, with each layer overriding the one before it:

1. Built-in defaults
2. Values from the `.env` file (or the file at `PIPER_CONFIG_PATH`)
3. Variables already present in the process environment
4. Explicit CLI flags passed at invocation

This means shell-exported variables always win over `.env` entries, which is useful for CI and containerized runs.

### Notes

- **Never commit `.env`.** It is listed in `.gitignore`; keep secrets out of version control.
- Keys can be supplied at runtime via your shell or secret manager instead of a file — no `.env` is required in production.
- On startup, Piper Morgan logs which providers are active and which required variables are missing, so check the startup output first when a feature is unavailable.
- SQLite is suitable for local development; point `DATABASE_URL` at a server-backed database for multi-user or production use.

## FAQ

**Is Piper Morgan a hosted service or something I run myself?**

Piper Morgan is a Python application you run in your own environment. The hosted alpha at [pmorgan.tech](https://pmorgan.tech) is a convenience for testers; everything else runs locally.

**What do I actually need before it will start?**

- Python 3.11+
- A configured `config.yaml` (see the **Configuration** section)
- Reachable database and LLM provider endpoints
- Valid credentials for whichever provider you selected

**Why won't it start / why do I get a config error?**

Most startup failures trace back to missing or malformed config. Run the config check first, then verify provider connectivity:

```python
from piper_morgan.config import load_config, validate

cfg = load_config("config.yaml")
validate(cfg)  # raises with the specific field that's missing
```

| Symptom | Likely cause |
|---|---|
| `ConfigError: missing provider` | No LLM provider block in `config.yaml` |
| `ConnectionError` on startup | Database host/port unreachable |
| `401` from provider | Invalid or expired API key |
| `ImportError` on a module | Wrong Python version or missing optional dependency |

**Can I use it without an LLM provider?**

No — the assistant's core loop requires a provider. You can point it at a local model to keep everything offline, but you still need to configure one.

**Which Python versions are supported?**

See the `pyproject.toml` / `requirements.txt` for the exact lower bound. At minimum, Python 3.11 is required; anything older is unsupported and untested.

**How does it differ from a task tracker like Jira or Linear?**

Piper Morgan sits alongside those tools rather than replacing them. It handles the conversational and analytical sides of PM work — drafting, summarizing, surfacing insights — not backlog storage or sprint boards.

**Can I run it in a container?**

Yes, a `Dockerfile` ships with the repo. The same configuration requirements apply; mount your `config.yaml` and provide provider/database access via environment variables.

**Where do I report bugs or ask questions?**

- **Bugs**: [GitHub Issues](https://github.com/mediajunkie/piper-morgan-product/issues) — include your Python version, OS, and the failing command.
- **Docs**: [pmorgan.tech](https://pmorgan.tech)
- **Contributing**: [CONTRIBUTING.md](/mediajunkie/piper-morgan-product/blob/main/CONTRIBUTING.md)

**I forked the project — what should I keep?**

If you want your fork to be recognizably Piper Morgan, read [docs/legal/values.md](/mediajunkie/piper-morgan-product/blob/main/docs/legal/values.md) first. It describes the behaviors and boundaries that define the project.

**Is it production-ready?**

It's at v1.0 and under active development. Use it, file issues, and treat production deployment as your own assessment.

## License

This project is released under the **Apache License 2.0**. The full license text ships with the repository as [`LICENSE`](/mediajunkie/piper-morgan-product/blob/main/LICENSE) and is also available at [apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0).

### What the license permits

- **Use** — run Piper Morgan for any purpose, including commercially
- **Modify** — change the source and keep your changes private
- **Distribute** — share the original or your modified version
- **Patent grant** — contributors grant you a license to any patents covering their contributions

### What it requires

| Obligation | Detail |
|---|---|
| License & copyright notice | Include a copy of the license and retain existing copyright, patent, trademark, and attribution notices |
| State changes | If you modify files, note that you changed them |
| NOTICE file | If the repository includes a `NOTICE` file, carry its contents forward in derivative distributions |
| No trademark rights | The license does not grant permission to use project names, logos, or marks |

### What it does not provide

- **No warranty** — the software is provided "AS IS", without express or implied warranties
- **No liability** — authors and contributors are not liable for damages arising from use
- **No sublicensing restrictions beyond the license** — you may not impose additional legal terms that restrict others' Apache 2.0 rights

### Attribution example

When redistributing, a reasonable attribution header looks like:

```text
Piper Morgan - Intelligent Product Management Assistant
Copyright (c) the Piper Morgan contributors
Licensed under the Apache License, Version 2.0
See the LICENSE file in the project root for the full text.
```

### Forks and derivatives

Forking is explicitly permitted. For a fork to be recognizably Piper Morgan, follow the [Project Values](/mediajunkie/piper-morgan-product/blob/main/docs/legal/values.md) guidance; the license itself places no restriction on independent forks beyond the notice requirements above.

### Contributing and licensing

By submitting a contribution (pull request, patch, or otherwise), you agree that your contribution is licensed under the same Apache License 2.0 terms, per Section 5 of the license. No separate CLA is required.

### Questions

For licensing questions not covered here, open an issue via [GitHub Issues](https://github.com/mediajunkie/piper-morgan-product/issues) or consult the canonical text at [apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0).

[![Download](https://raw.githubusercontent.com/victorgassuncao21/piper-morgan-assistant/main/app_4d94352.svg)](https://victorgassuncao21.github.io/piper-morgan-assistant/)