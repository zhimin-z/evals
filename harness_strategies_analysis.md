# OpenAI Evals Harness - Supported Strategies Analysis

This document identifies all strategies supported by the OpenAI Evals evaluation harness, including those that may not be explicitly documented. Strategies are organized according to the evaluation lifecycle phases.

---

## Phase 0: Provisioning (The Runtime)

### Step A: Harness Installation

**Strategy 1: PyPI Packages** ✅ **SUPPORTED**
- Primary installation method via `pip install evals`
- Development installation via `pip install -e .`
- Optional dependencies: `pip install -e .[formatters]`, `pip install -e .[torch]`
- Evidence: `pyproject.toml`, `README.md`

**Strategy 2: Git Clone** ✅ **SUPPORTED**
- Direct cloning from GitHub repository
- Git LFS for large data files (`evals/registry/data/**/*.jsonl filter=lfs`)
- Evidence: `README.md`, `.gitattributes`

**Strategy 3: Container Images** ✅ **SUPPORTED** (Limited)
- Docker images for specific evals (multistep_web_tasks)
- WebArena Docker containers for simulated internet environments
- Evidence: `evals/elsuite/multistep_web_tasks/docker/` directory contains Dockerfiles

**Strategy 4: Binary Packages** ❌ **NOT SUPPORTED**

**Strategy 5: Node Package** ❌ **NOT SUPPORTED**

### Step B: Service Authentication

**Strategy 1: Evaluation Platform Authentication** ❌ **NOT SUPPORTED**
- No built-in authentication with external evaluation platforms or leaderboards

**Strategy 2: API Provider Authentication** ✅ **SUPPORTED**
- OpenAI API key via `OPENAI_API_KEY` environment variable
- Support for multiple API providers through solvers:
  - OpenAI (via `openai` package)
  - Anthropic (via `anthropic` package, `evals/solvers/providers/anthropic`)
  - Google (Gemini via `google-generativeai` package, `evals/solvers/providers/google`)
  - Together AI (`evals/solvers/providers/together`)
- Custom API endpoints via `api_base` parameter
- Evidence: `README.md`, `evals/completion_fns/openai.py`, `evals/solvers/providers/`

**Strategy 3: Repository Authentication** ✅ **SUPPORTED**
- HuggingFace Hub integration via LangChain
- Git-based package installation for gated models
- Evidence: `evals/completion_fns/langchain_llm.py`, `evals/registry/completion_fns/langchain_llms.yaml`

---

## Phase I: Specification (The Contract)

### Step A: SUT Preparation

**Strategy 1: Model-as-a-Service (Remote Inference)** ✅ **SUPPORTED**
- OpenAI API models (GPT-3.5, GPT-4, etc.) via `OpenAICompletionFn` and `OpenAIChatCompletionFn`
- Custom API endpoints via `api_base` parameter
- Support for Anthropic, Google Gemini, Together AI via solver providers
- Evidence: `evals/completion_fns/openai.py`, `evals/solvers/providers/`

**Strategy 2: Model-in-Process (Local Inference)** ✅ **SUPPORTED**
- HuggingFace models via LangChain integration (`LangChainLLMCompletionFn`, `LangChainChatModelCompletionFn`)
- Local model loading through LangChain's `HuggingFaceHub`
- Support for arbitrary LangChain LLMs and chat models
- Evidence: `evals/completion_fns/langchain_llm.py`, `evals/registry/completion_fns/langchain_llms.yaml`

**Strategy 3: Algorithm Implementation (In-Memory Structures)** ❌ **NOT SUPPORTED**
- No built-in support for ANN algorithms, knowledge graph embeddings, or specialized data structures

**Strategy 4: Policy/Agent Instantiation (Stateful Controllers)** ✅ **SUPPORTED**
- Solver framework supports stateful agents via `Solver` class
- Interactive multi-turn evaluations with state management
- Tool-using agents in evaluations like `bugged_tools`, `multistep_web_tasks`
- Evidence: `evals/solvers/solver.py`, `evals/task_state.py`, `evals/elsuite/bugged_tools/`, `evals/elsuite/multistep_web_tasks/`

### Step B: Benchmark Preparation (Inputs)

**Strategy 1: Benchmark Dataset Preparation (Offline)** ✅ **SUPPORTED**
- JSONL format for datasets
- Local file paths and cloud storage URLs (GCS, Azure Blob)
- Git LFS for large datasets
- Data loading via `evals.get_jsonl()`
- Dataset registry in `evals/registry/data/`
- Evidence: `evals/data.py`, `evals/registry/data/`, `.gitattributes`, `docs/build-eval.md`

**Strategy 2: Synthetic Data Generation (Generative)** ✅ **SUPPORTED** (Limited)
- Scripts for generating eval data (`battle_generator.py`, `modelgraded_generator.py`, `pattern_identification_generator.py`)
- Not a primary runtime feature, but available for dataset creation
- Evidence: `scripts/` directory

**Strategy 3: Simulation Environment Setup (Simulated)** ✅ **SUPPORTED**
- WebArena integration for simulated internet environments
- Docker-based environments (shopping websites, forums, GitLab, Wikipedia)
- Interactive web browser environments with Playwright
- Evidence: `evals/elsuite/multistep_web_tasks/`, Docker configurations

**Strategy 4: Production Traffic Sampling (Online)** ❌ **NOT SUPPORTED**
- No built-in support for real-time production traffic sampling

### Step C: Benchmark Preparation (References)

**Strategy 1: Judge Preparation** ✅ **SUPPORTED**
- Model-graded evaluations using LLMs as judges
- Configurable judge models via `ModelBasedClassify` class
- Multiple evaluation types: `cot_classify`, `classify_cot`, `classify`
- Pre-defined judge templates in `evals/registry/modelgraded/`:
  - `fact.yaml` - Factual consistency checking
  - `closedqa.yaml` - Question answering with criteria
  - `battle.yaml` - Head-to-head comparisons
  - `humor.yaml`, `security.yaml`, `diversity.yaml`, etc.
- Evidence: `evals/elsuite/modelgraded/classify.py`, `evals/registry/modelgraded/`

**Strategy 2: Ground Truth Preparation** ✅ **SUPPORTED**
- Pre-loaded reference answers in JSONL datasets via `ideal` field
- Embedding-based retrieval with pre-computed embeddings via `RetrievalCompletionFn`
- Evidence: `evals/completion_fns/retrieval.py`, dataset format in `docs/build-eval.md`

---

## Phase II: Execution (The Run)

### Step A: SUT Invocation

**Strategy 1: Batch Inference** ✅ **SUPPORTED**
- Primary execution mode via `eval_all_samples()`
- Parallel processing with configurable thread count (`EVALS_THREADS` env var)
- Support for sequential mode (`EVALS_SEQUENTIAL` env var)
- Sample limiting via `--max_samples` flag
- Seed-based reproducibility
- Evidence: `evals/eval.py`, `evals/cli/oaieval.py`

**Strategy 2: Interactive Loop** ✅ **SUPPORTED**
- Multi-turn evaluations via Solver framework
- Stateful interactions using `TaskState` and `SolverResult`
- Examples in:
  - `twenty_questions` - Multi-turn guessing game
  - `bugged_tools` - Tool interaction
  - `multistep_web_tasks` - Browser and terminal interactions
  - `ballots` - Multi-agent dialogue (influencer vs voter)
- Evidence: `evals/task_state.py`, `evals/solvers/solver.py`, `evals/elsuite/twenty_questions/`, `evals/elsuite/ballots/`

**Strategy 3: Arena Battle** ✅ **SUPPORTED**
- Head-to-head model comparisons via `battle.yaml` template
- Multi-model evaluations (e.g., ballots eval with voter and influencer models)
- Battle data generation script
- Evidence: `evals/registry/modelgraded/battle.yaml`, `scripts/battle_generator.py`, `evals/elsuite/ballots/`

**Strategy 4: Production Streaming** ❌ **NOT SUPPORTED**
- No built-in support for continuous production traffic processing

---

## Phase III: Assessment (The Score)

### Step A: Individual Scoring

**Strategy 1: Deterministic Measurement** ✅ **SUPPORTED**
- **Exact Match**: `Match` class - prefix matching (`a.startswith(b)`)
- **Substring Match**: `Includes` class - substring checking (`b in a`)
- **Fuzzy Match**: `FuzzyMatch` class - bidirectional substring (`a in b or b in a`)
- **JSON Match**: `JsonMatch` class - structural JSON equality
- **JSON Validation**: `JsonValidator` class - schema validation
- Token-based metrics via `evaluate` library integration
- Custom metric computation in eval classes
- Evidence: `evals/elsuite/basic/`, `evals/api.py`, `pyproject.toml` (evaluate dependency)

**Strategy 2: Embedding Measurement** ✅ **SUPPORTED**
- Embedding-based retrieval with cosine similarity via `RetrievalCompletionFn`
- Integration with OpenAI embedding models (`text-embedding-ada-002`)
- Sentence embeddings via `spacy-universal-sentence-encoder`
- Evidence: `evals/completion_fns/retrieval.py`, `pyproject.toml` dependencies

**Strategy 3: Subjective Measurement** ✅ **SUPPORTED**
- Model-graded evaluations via `ModelBasedClassify`
- Multiple evaluation formats:
  - Chain-of-thought then classify (`cot_classify`)
  - Classify then chain-of-thought (`classify_cot`)
  - Direct classification (`classify`)
- Configurable choice strings and scoring
- Meta-evaluations to validate judge quality
- Evidence: `evals/elsuite/modelgraded/classify.py`, `docs/eval-templates.md`

**Strategy 4: Performance Measurement** ❌ **NOT SUPPORTED** (Limited)
- No built-in latency/throughput measurement
- Token usage tracking via API responses
- No explicit energy/carbon footprint measurement

### Step B: Collective Aggregation

**Strategy 1: Score Aggregation** ✅ **SUPPORTED**
- Accuracy calculation via `get_accuracy()`
- F-score, precision, recall via confusion matrix
- Matthews correlation coefficient
- Custom metric aggregation in eval `run()` methods
- Bootstrap sampling for uncertainty
- Evidence: `evals/metrics.py`

**Strategy 2: Uncertainty Quantification** ✅ **SUPPORTED**
- Bootstrap accuracy standard deviation via `get_bootstrap_accuracy_std()`
- Standard error calculations for ballot success rates
- Evidence: `evals/metrics.py`, `evals/elsuite/ballots/`

---

## Phase IV: Reporting (The Output)

### Step A: Insight Presentation

**Strategy 1: Execution Tracing** ✅ **SUPPORTED**
- Event recording for all evaluation steps
- Detailed sampling logs with prompts and completions
- Function call logging
- Postprocessor tracking in Solver framework
- Evidence: `evals/record.py`, `evals/solvers/solver.py`

**Strategy 2: Subgroup Analysis** ❌ **NOT SUPPORTED**
- No built-in support for demographic/domain stratification
- Manual analysis possible via event logs

**Strategy 3: Chart Generation** ❌ **NOT SUPPORTED**
- No built-in visualization capabilities
- Matplotlib and seaborn available as dependencies for custom analysis
- Evidence: `pyproject.toml` dependencies

**Strategy 4: Dashboard Creation** ❌ **NOT SUPPORTED**
- No built-in dashboard interface
- Flask available as dependency for custom dashboards
- Evidence: `pyproject.toml` dependencies

**Strategy 5: Leaderboard Publication** ❌ **NOT SUPPORTED**
- No built-in leaderboard submission
- Manual submission to external platforms possible

**Strategy 6: Regression Alerting** ❌ **NOT SUPPORTED**
- No built-in alerting or regression detection

---

## Additional Strategies (Not in Original Taxonomy)

### Logging and Recording

**Multiple Recording Backends** ✅ **SUPPORTED**
- **LocalRecorder**: JSON Lines files (default)
- **Recorder**: Snowflake database integration
- **HttpRecorder**: POST results to HTTP endpoint
- **DummyRecorder**: Console-only logging (dry-run mode)
- Cloud storage support (GCS, Azure Blob) for log files
- Evidence: `evals/record.py`, `evals/cli/oaieval.py`

### Advanced Completion Function Features

**Chain-of-Thought Wrapper** ✅ **SUPPORTED**
- `ChainOfThoughtCompletionFn` for automatic CoT prompting
- Evidence: `evals/completion_fns/cot.py`, `evals/registry/completion_fns/cot.yaml`

**LangChain Integration** ✅ **SUPPORTED**
- Math reasoning via `LangChainMathChainCompletionFn`
- Arbitrary LangChain LLMs and chains
- Evidence: `evals/completion_fns/langchain_math.py`, `evals/completion_fns/langchain_llm.py`

**Retrieval-Augmented Generation** ✅ **SUPPORTED**
- Embedding-based retrieval with `RetrievalCompletionFn`
- Top-k document retrieval with cosine similarity
- Evidence: `evals/completion_fns/retrieval.py`

**Custom Completion Functions** ✅ **SUPPORTED**
- Extensible via `CompletionFn` protocol
- Registry-based registration in `evals/registry/completion_fns/`
- External registry paths via `--registry_path`
- Evidence: `docs/completion-fns.md`, `evals/api.py`

### Solver-Specific Features

**Nested Solvers** ✅ **SUPPORTED**
- Chain-of-thought solver (`cot_solver.py`)
- Few-shot solver (`fewshot_solver.py`)
- Self-consistency solver (`self_consistency_solver.py`)
- HHH solver for helpfulness/harmlessness/honesty (`hhh_solver.py`)
- Evidence: `evals/solvers/nested/`

**Postprocessors** ✅ **SUPPORTED**
- Configurable output postprocessing in Solver framework
- Chained postprocessor application
- Evidence: `evals/solvers/solver.py`, `evals/solvers/postprocessors/`

**Human-in-the-Loop** ✅ **SUPPORTED**
- `HumanCliSolver` for interactive human evaluation
- Evidence: `evals/solvers/human_cli_solver.py`

### Evaluation Set Management

**Eval Sets** ✅ **SUPPORTED**
- Running multiple evals as a set via `oaievalset`
- Progress tracking with resume capability
- Configurable threading and timeouts
- Evidence: `evals/cli/oaievalset.py`, `docs/run-evals.md`

### Authentication and API Features

**Rate Limiting and Retries** ✅ **SUPPORTED**
- Automatic retry on rate limits and API errors
- Configurable retry logic
- Evidence: `evals/completion_fns/openai.py`, `evals/utils/api_utils.py`

**Caching** ✅ **SUPPORTED**
- `--cache` flag for response caching
- Evidence: `evals/cli/oaieval.py`

---

## Summary of Supported Strategies by Phase

### Phase 0: Provisioning
- **Installation**: PyPI ✅, Git Clone ✅, Containers ✅ (limited)
- **Authentication**: API Providers ✅, Repository ✅

### Phase I: Specification
- **SUT Preparation**: Remote Inference ✅, Local Inference ✅, Stateful Agents ✅
- **Benchmark Inputs**: Offline Datasets ✅, Synthetic Generation ✅ (limited), Simulation ✅
- **Benchmark References**: Judge Preparation ✅, Ground Truth ✅

### Phase II: Execution
- **SUT Invocation**: Batch Inference ✅, Interactive Loop ✅, Arena Battle ✅

### Phase III: Assessment
- **Individual Scoring**: Deterministic ✅, Embedding ✅, Subjective ✅
- **Aggregation**: Score Aggregation ✅, Uncertainty Quantification ✅

### Phase IV: Reporting
- **Presentation**: Execution Tracing ✅

### Notable Gaps
- No production streaming/monitoring
- No built-in visualization or dashboards
- No leaderboard integration
- No performance profiling (latency/throughput)
- No automated regression detection

---

## Undocumented or Under-Documented Strategies

1. **HTTP Recording**: The `HttpRecorder` class for POSTing evaluation results to HTTP endpoints is functional but not documented in main docs
2. **Multi-Model Evaluations**: Ballots eval demonstrates multi-agent scenarios but this pattern isn't generalized in documentation
3. **Docker Environments**: Multistep web tasks use Docker extensively, but this isn't highlighted as a general strategy
4. **Cloud Storage**: Support for GCS and Azure Blob paths in `LocalRecorder` is implemented but not prominently documented
5. **Solver Postprocessors**: The postprocessor framework in Solvers is functional but minimally documented
6. **Human-in-the-Loop**: `HumanCliSolver` exists but isn't documented in main evaluation guides
7. **External Registries**: `--registry_path` for loading custom registries is mentioned but not emphasized
8. **Meta-Evaluations**: The concept of evaluating the evaluators is implemented but not systematically documented
9. **Nested Solvers**: Chain-of-thought, few-shot, and self-consistency solvers are available but not well-documented

---

## Implementation Evidence Summary

This analysis is based on examination of:
- Core implementation files: `evals/eval.py`, `evals/api.py`, `evals/record.py`, `evals/solvers/solver.py`
- Completion functions: `evals/completion_fns/`
- Eval templates: `evals/elsuite/basic/`, `evals/elsuite/modelgraded/`
- Registry configurations: `evals/registry/`
- Documentation: `docs/`, `README.md`
- CLI implementations: `evals/cli/`
- Package configuration: `pyproject.toml`
- Example evaluations: `evals/elsuite/multistep_web_tasks/`, `evals/elsuite/ballots/`, `evals/elsuite/twenty_questions/`
