# Agent Catalog Quickstart

This repository provides a quickstart guide for using the Agent Catalog with Capella Model Services and Couchbase.

Each example is built around a real industry scenario — not a toy demo — so you can see exactly which business problem the pattern solves, how it works, and what impact it delivers before adapting it to your own domain.

## Examples at a Glance

| Example | Industry | Use Case | Framework | Business Impact |
|---|---|---|---|---|
| ✈️ **Flight Search & Booking Agent** (`notebooks/flight_search_agent_langraph/`) | Airlines & Online Travel | Self-service booking assistant that searches routes, creates and retrieves bookings, and answers questions from airline reviews | LangGraph | Deflects routine booking calls from contact centers, shortens time-to-book, and keeps revenue in the direct channel |
| 🏨 **Hotel Support Agent** (`notebooks/hotel_search_agent_langchain/`) | Hospitality | Concierge-style hotel discovery over unstructured property data using semantic (vector) search | LangChain | Lifts look-to-book conversion by matching guests to properties by intent ("quiet, near the beach, free breakfast"), not just keywords |
| 🗺️ **Landmark Discovery Agent** (`notebooks/landmark_search_agent_llamaindex/`) | Tourism & Destination Marketing | Attraction and points-of-interest recommendations grounded in a curated destination catalog | LlamaIndex | Drives engagement and in-destination spend with grounded recommendations instead of hallucinated ones |

All three run on the same foundation — **Agent Catalog** for versioned prompts/tools, **Couchbase** as the data platform and vector store, and **Arize Phoenix** for evaluation — so the pattern you learn in one industry transfers directly to yours. Full details for each example are under [Per-Agent Details](#per-agent-details), and the mapping to other industries is under [Adapting These Patterns to Your Industry](#adapting-these-patterns-to-your-industry).

## Prerequisites

- Python 3.12+
- Poetry ([Installation Guide](https://python-poetry.org/docs/#installation))
- Git (for repository management)
- An OpenAI API Key (or other LLM provider)
- Couchbase Capella account (or local Couchbase installation)

## Quick Start

Two ways to get running fast. Choose one.

### 1) Full repo setup (script)

```bash
git clone --recursive https://github.com/couchbaselabs/agent-catalog-quickstart.git
cd agent-catalog-quickstart
bash scripts/setup.sh --yes               # add --skip-testing to speed up

# pick an agent and configure env
cd notebooks/hotel_search_agent_langchain
cp .env.sample .env && $EDITOR .env

# run
poetry run python main.py
```

### 2) Per-agent setup (fastest)

```bash
# from repo root
poetry -C notebooks/flight_search_agent_langraph install --no-root
cp notebooks/flight_search_agent_langraph/.env.sample notebooks/flight_search_agent_langraph/.env
$EDITOR notebooks/flight_search_agent_langraph/.env
poetry -C notebooks/flight_search_agent_langraph run python main.py
```

### 3) Installation Methods & Package Management

#### **Recommended: pipx for CLI (Isolated)**

For the cleanest installation that avoids system conflicts:

```bash
# Install pipx if not available
brew install pipx  # macOS
# or: python3 -m pip install --user pipx

# Use the proper pipx setup script
bash scripts/setup_pipx.sh
```

**Benefits:**

- Isolated environments prevent conflicts
- Clean separation between CLI tools and project dependencies
- Works with externally managed Python environments (Homebrew, system Python)

#### **Alternative: Global pip installs (PyPI)**

⚠️ **Note**: This method may conflict with externally managed environments. Use only if pipx fails.

```bash
# Install Agent Catalog packages (add --break-system-packages if needed)
pip3 install agentc agentc-core agentc-cli agentc-langchain agentc-langgraph agentc-llamaindex

# Install Arize Phoenix and evaluation dependencies
pip3 install "arize-phoenix[evals]" arize arize-otel openinference-instrumentation-langchain openinference-instrumentation-openai openinference-instrumentation-llama-index

# Fix OpenTelemetry version conflicts (if needed)
pip3 install --upgrade opentelemetry-instrumentation-asgi opentelemetry-instrumentation-fastapi opentelemetry-util-http
```

Then run the per-agent commands under "2) Per-agent setup (fastest)" above.

#### **Development Note**

These packages are currently installed from source during development. Once agentc packages are available on PyPI, the installation will be simplified to standard pip/pipx commands.

### Working with Git Submodules

This repository uses git submodules to manage the Agent Catalog dependency. If you encounter issues:

```bash
# If you cloned without --recursive, initialize submodules manually:
git submodule update --init --recursive

# Update submodules to latest versions:
git submodule update --remote

# Verify submodules are properly initialized:
git submodule status
```

**Note**: The `agent-catalog` directory is managed as a submodule and should not be manually edited.

## Per-Agent Details

Each example is independent and includes code, prompts, tools, and evals. Each one targets a specific industry scenario so you can map it directly onto an equivalent problem in your own business.

### ✈️ Flight Search & Booking Agent (`notebooks/flight_search_agent_langraph/`)

**Industry:** Airlines & Online Travel Agencies (OTAs)

**Use case:** A customer-facing booking assistant. Travelers ask in plain language and the agent looks up routes between airports (SQL++ over Couchbase), creates and retrieves bookings, and answers "what's this airline like?" questions using semantic search over airline reviews.

**Try queries like:**

- `"Find flights from JFK to LAX"`
- `"Book a flight from SFO to ATL tomorrow for 2 passengers"`
- `"Show me my current bookings"`
- `"What do passengers say about SpiceJet's service?"`

**What it demonstrates:** A multi-tool LangGraph agent where every tool (`lookup_flight_info`, `save_flight_booking`, `retrieve_flight_bookings`, `search_airline_reviews`) and prompt is versioned and discovered through Agent Catalog — combining structured SQL++ queries, transactional writes, and vector search in one agent.

**Business impact:**

- **Lower cost-to-serve** — route lookups, booking creation, and "where's my booking?" requests are among the highest-volume contact-center drivers for airlines; each conversation the agent completes is an agent-handled call or chat that never reaches a human.
- **More direct-channel revenue** — a 24/7 conversational booking path shortens time-to-book and keeps customers on your property instead of a metasearch site.
- **Higher trust in answers** — review questions are answered from your own review corpus via vector search, not from the LLM's memory, which is what makes the assistant safe to put in front of customers.

### 🏨 Hotel Support Agent (`notebooks/hotel_search_agent_langchain/`)

**Industry:** Hospitality & Accommodation Booking

**Use case:** A concierge-style discovery assistant. Guests describe what they want the way they'd tell a person — location, vibe, amenities — and the agent runs semantic vector search over real hotel data (`travel-sample.inventory.hotel`) to surface properties that match the *intent*, not just the keywords.

**Try queries like:**

- `"Find me a hotel in San Francisco"`
- `"Find hotels in Paris with free breakfast"`
- `"Somewhere quiet near the beach with parking"`

**What it demonstrates:** A LangChain agent with an Agent Catalog-managed vector search tool (`search_vector_database`) over Couchbase, using Capella Model Services or OpenAI embeddings — the core retrieval pattern behind every "help me find the right product" experience.

**Business impact:**

- **Higher look-to-book conversion** — keyword search returns "no results" or noise when guests search the way they speak; semantic search turns those failed searches into qualified matches.
- **Fewer pre-booking support contacts** — amenity and location questions ("does it have parking?", "is it walkable to downtown?") get answered in the discovery flow instead of via email or phone.
- **Better inventory utilization** — properties that don't match popular keywords still surface when they genuinely fit a guest's described need.

### 🗺️ Landmark Discovery Agent (`notebooks/landmark_search_agent_llamaindex/`)

**Industry:** Tourism Boards, Destination Marketing & Travel Media

**Use case:** An attraction-recommendation assistant. Visitors ask for things to see and do, and the agent answers with semantic search over a curated landmark catalog (`travel-sample.inventory.landmark`) — museums, monuments, parks, and points of interest — so every recommendation is grounded in your destination data.

**Try queries like:**

- `"Find me landmarks in Tokyo"`
- `"Show me museums in London"`
- `"Historic sites within walking distance of the old town"`

**What it demonstrates:** A LlamaIndex ReAct agent with an Agent Catalog-managed semantic search tool (`search_landmarks`) — the retrieval-augmented recommendation pattern for any curated content catalog.

**Business impact:**

- **Grounded, brand-safe recommendations** — the agent recommends only what's in your catalog, eliminating the hallucinated or outdated suggestions that erode traveler trust in generic chatbots.
- **More engagement and in-destination spend** — personalized itinerary-style discovery keeps visitors exploring (and booking) instead of bouncing to a search engine.
- **Content ROI** — destination content you already maintain becomes an interactive product instead of static pages.

## Adapting These Patterns to Your Industry

The examples use travel data because it ships with Couchbase (`travel-sample`), but each one is an industry-agnostic pattern. Swap the data, prompts, and tools — the Agent Catalog workflow (`agentc init` → `index` → `publish`) stays identical.

| Pattern (example) | Financial Services | Healthcare | Retail & E-commerce | Manufacturing & Logistics |
|---|---|---|---|---|
| **Transactional multi-tool agent** (flight booking) | Order status, payments, and account-servicing assistant — deflects tier-1 banking calls | Appointment scheduling and prescription-refill assistant — cuts no-shows and front-desk load | Order tracking, returns, and exchange agent — lowers cost per support ticket | Shipment booking and track-and-trace agent — reduces "where is my order?" escalations |
| **Semantic product discovery** (hotel search) | Fund/product finder matched to stated goals and risk appetite — improves qualified lead rate | Provider/specialist finder by symptoms, insurance, and location — speeds patient access | Natural-language product search ("waterproof jacket for spring hiking") — recovers failed keyword searches as sales | Parts and equipment finder by described function — shortens procurement cycles |
| **Curated-catalog recommendations** (landmark discovery) | Grounded research/insights assistant over your published analyses — scales advisor reach safely | Patient-education assistant over approved clinical content — safe answers, fewer nurse-line calls | Grounded gift/style recommendations from your catalog — raises average order value | Grounded maintenance and troubleshooting guidance from service manuals — less machine downtime |

To build your own: start from the example closest to your use case, replace the `data/` loaders with your collections, edit the `prompts/` and `tools/` (starter files live in the `templates/` directory — see the Templates section below), then re-run `agentc index` and `agentc publish`. The [Adding New Agents](#adding-new-agents) section covers the full workflow.

## Environment Configuration

Each agent needs its own `.env` file with your credentials:

```bash
# Copy the sample file and edit it
cp .env.sample .env
# Edit .env with your actual credentials
```

**Required files:**

- `notebooks/flight_search_agent_langraph/.env`
- `notebooks/hotel_search_agent_langchain/.env`
- `notebooks/landmark_search_agent_llamaindex/.env`

For complete environment configuration examples (Capella vs Local), see **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md#environment-configuration-examples)**.

## Usage

Each agent starts an interactive demo — type your queries at the prompt (`quit` to exit):

```bash
# Airlines / OTA: search routes, book flights, check airline reviews
poetry -C notebooks/flight_search_agent_langraph run python main.py
# then try: "Find flights from JFK to LAX" or "Show me my current bookings"

# Hospitality: intent-based hotel discovery
poetry -C notebooks/hotel_search_agent_langchain run python main.py
# then try: "Find hotels in Paris with free breakfast"

# Tourism / destination marketing: grounded attraction recommendations
poetry -C notebooks/landmark_search_agent_llamaindex run python main.py
# then try: "Show me museums in London"

# run the built-in test suite for an agent
poetry -C notebooks/hotel_search_agent_langchain run python main.py test

# run evaluations (Arize)
poetry -C notebooks/hotel_search_agent_langchain run python evals/eval_arize.py
```

### Using Global CLI (After Full Setup)

If you installed the global CLI:

```bash
cd notebooks/hotel_search_agent_langchain

# Initialize Agent Catalog
agentc init

# Index your agent
agentc index .

# Publish your agent (requires clean git)
git add . && git commit -m "Your changes"
agentc publish

# Run the agent (interactive demo)
python main.py
# then try: "Find hotels in Paris with free breakfast"
```

## Agent Catalog CLI Commands

| Command          | Description                                          |
| ---------------- | ---------------------------------------------------- |
| `agentc init`    | Initialize agent catalog in current directory        |
| `agentc index .` | Index the current agent directory                    |
| `agentc publish` | Publish agent to catalog (requires clean git status) |
| `agentc --help`  | Show all available commands                          |
| `agentc env`     | Show environment configuration                       |

## Troubleshooting

Having issues? Check our comprehensive troubleshooting guide: **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**

### Quick Fixes

- "No module named 'agentc'": ensure you ran `poetry -C notebooks/<agent> install --no-root` and are executing with Poetry (`poetry run ...`).
- Evaluation deps missing: run the agent-specific Poetry install again; eval deps are included per agent.
- Poetry issues: delete the agent’s `poetry.lock` and run `poetry -C notebooks/<agent> install --no-root`.
- Environment errors: copy `.env.sample` to `.env` in the agent folder and fill in credentials.
- CLI not found after script: restart your shell or run `export PATH="$PATH:$HOME/.local/bin"`; rerun `bash scripts/setup.sh --yes` if needed.
- Submodule issues: run `git submodule update --init --recursive` to initialize the agent-catalog dependency.

For detailed solutions, environment configuration examples, and debugging commands, see **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**.

## Development

### Adding New Agents

1. Create a new directory under `notebooks/`
2. Add your agent code, prompts, and tools
3. Create appropriate configuration files (`pyproject.toml`, `.env`)
4. Install Agent Catalog: `pip3 install agentc agentc-core agentc-cli agentc-langchain agentc-langgraph agentc-llamaindex`
5. Install root dependencies: `poetry install`
6. Run `poetry install --no-root` in the new directory
7. Run `agentc init` and `agentc index .`

### Evaluation

Run evaluations with Arize:

```bash
poetry run python evals/eval_arize.py
```

## Repository Resources

### 📚 Documentation (`docs/`)

Comprehensive guides and references for working with the Agent Catalog ecosystem:

| Document | Purpose |
|---|---|
| **[PYPROJECT_GUIDE.md](docs/PYPROJECT_GUIDE.md)** | Complete guide to `pyproject.toml` configuration |
| **[AGENTC_GUIDE.md](docs/AGENTC_GUIDE.md)** | Agent Catalog CLI and usage documentation |
| **[PYTHON_LINUX.md](docs/PYTHON_LINUX.md)** | Linux Python setup and pip troubleshooting |
| **[PYTHON_MAC.md](docs/PYTHON_MAC.md)** | macOS Python environment setup guide |
| **[CAPELLA_MODELS.md](docs/CAPELLA_MODELS.md)** | Couchbase Capella model services integration |
| **[EVALUATION_FRAMEWORKS_COMPARISON.md](docs/EVALUATION_FRAMEWORKS_COMPARISON.md)** | Comparison of AI evaluation frameworks |

**Quick access:**
```bash
# View documentation
ls docs/                    # List all documentation files
cat docs/PYPROJECT_GUIDE.md # Read specific guide
```

### 🛠️ Templates (`templates/`)

Ready-to-use templates for creating Agent Catalog components:

| Template | Purpose | Usage |
|---|---|---|
| **`prompt_template.yaml`** | Agent prompt templates | Create new prompts with proper structure |
| **`python_function_template.py`** | Python tool functions | Build custom tools and utilities |
| **`semantic_search_template.yaml`** | Couchbase vector search | Set up semantic search functionality |
| **`sqlpp_query_template.sqlpp`** | Database queries | Create SQL++ queries for Couchbase |
| **`http_request_template.yaml`** | HTTP/API requests | Build HTTP request tools |
| **`agentc_command_notes.txt`** | CLI command reference | AgentC command examples |

**Using templates:**
```bash
# Copy template for new component
cp templates/prompt_template.yaml prompts/my_new_prompt.yaml
cp templates/python_function_template.py tools/my_new_tool.py

# Edit with your specific requirements
$EDITOR prompts/my_new_prompt.yaml
```

### 🔧 Shared Resources (`shared/`)

Common utilities and configurations used across all agents:

| File | Purpose |
|---|---|
| **`agent_setup.py`** | Common agent initialization and setup utilities |
| **`couchbase_client.py`** | Couchbase database connection and client management |
| **`capella_model_services_langchain.py`** | LangChain integration with Capella model services |
| **`capella_model_services_llamaindex.py`** | LlamaIndex integration with Capella model services |
| **`__init__.py`** | Package initialization for shared utilities |

**Using shared resources:**
```python
# Import shared utilities in your agent
from shared.couchbase_client import get_couchbase_client
from shared.agent_setup import initialize_agent
from shared.capella_model_services_langchain import get_langchain_llm
```

### 📋 Scripts (`scripts/`)

Automation and setup scripts for the repository:

| Script | Purpose |
|---|---|
| **`setup.sh`** | Full repository setup and installation |
| **`setup_pipx.sh`** | Clean pipx-based installation (recommended) |
| **`scope_copy.py`** | Utility for copying agent scopes |

**Using scripts:**
```bash
# Run setup scripts
bash scripts/setup.sh --yes           # Full setup
bash scripts/setup_pipx.sh           # Clean pipx setup
python scripts/scope_copy.py         # Utility script
```

## Architecture

Each example agent follows this structure:

```
notebooks/agent_name/
├── main.py              # Main agent implementation
├── pyproject.toml       # Poetry dependencies (requires poetry install)
├── .env                 # Environment configuration
├── prompts/             # Agent prompts and templates
├── tools/               # Agent tools and functions
├── data/                # Data loading and processing
└── evals/               # Evaluation scripts
```

## Contributing

This is a quickstart repository. For contributing to the main Agent Catalog:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Ensure all poetry dependencies are installed
5. Commit changes (required for publishing)
6. Submit a pull request

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.
