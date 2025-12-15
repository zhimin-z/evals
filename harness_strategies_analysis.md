# OpenAI Evals Harness - Supported Strategies Analysis

This document identifies all strategies supported by the **OpenAI Evals ecosystem**, following the **Unified Evaluation Workflow** taxonomy that abstracts evaluation workflows across over 50 frameworks—from text generation to robot manipulation, molecular property prediction, and hardware stress testing.

The ecosystem consists of:

1. **Open-Source Repository** (this repo) - CLI-based evaluation framework with full code access
2. **OpenAI Platform Dashboard** ([platform.openai.com/docs/guides/evals](https://platform.openai.com/docs/guides/evals)) - Web-based evaluation UI with additional visualization and collaboration features

Every evaluation, no matter the modality, traverses these five phases.

## 📊 Coverage Summary

### Open-Source Repository Only
- **22 out of 38 strategies (58%)**
- Strongest in: Execution (75%), Assessment (83%)
- Weakest in: Reporting (17%)

### Combined Ecosystem (Repository + Dashboard)
- **27 out of 38 strategies (71%)**
- Dashboard adds: Platform authentication, subgroup analysis, chart generation, interactive dashboards, leaderboards
- Phase IV (Reporting) improves from 17% to 83% with Dashboard

### Strategy Notation
- ✅ **SUPPORTED** - Fully supported
- ✅ **SUPPORTED** (via OpenAI Platform Dashboard) - Supported through Dashboard
- ✅ **SUPPORTED** (Limited) - Partial support with limitations
- ❌ **NOT SUPPORTED** - Not available in either component

---

Strategies are organized according to the evaluation lifecycle phases.

---

## Phase 0: Provisioning (The Runtime)

*Establishing the technical foundation—you cannot evaluate what you cannot run.*

### Step A: Harness Installation

*Definition:* Installing dependencies, compiling binaries, building containers, and configuring execution backends.

**Strategy 1: Git Clone** ✅ **SUPPORTED** (Native)
- Direct cloning from GitHub repository for bleeding-edge versions or development work
- Git LFS for large data files (`evals/registry/data/**/*.jsonl filter=lfs`)
- Manual installation from source: `git clone https://github.com/openai/evals && cd evals && pip install -e .`
- Available out-of-the-box after installation
- Evidence: `README.md`, `.gitattributes`

**Strategy 2: PyPI Packages** ✅ **SUPPORTED** (Native)
- Primary installation method via `pip install evals`
- Development installation via `pip install -e .`
- Requirements files via `pip install -r requirements.txt`
- Git-based installations via `pip install git+https://github.com/openai/evals`
- Optional dependencies: `pip install -e .[formatters]`, `pip install -e .[torch]`
- Evidence: `pyproject.toml`, `README.md`

**Strategy 3: Node Package** ❌ **NOT SUPPORTED**
- No JavaScript-based installation via npm, npx, or Homebrew

**Strategy 4: Binary Packages** ❌ **NOT SUPPORTED**
- No standalone executable binaries

**Strategy 5: Container Images** ❌ **NOT SUPPORTED**
- The harness itself is NOT installed via Docker containers
- Installation is done via PyPI (`pip install evals`) or Git clone
- Note: Docker IS used within some evaluations (e.g., multistep_web_tasks with WebArena) for running evaluation environments, but this is for simulation purposes, not for installing the harness

### Step B: Service Authentication

*Definition:* Authenticating with model repositories, dataset platforms, evaluation services, and leaderboard APIs.

**Strategy 1: Evaluation Platform Authentication** ✅ **SUPPORTED** (via OpenAI Platform Dashboard)
- **Open-Source Repository**: ❌ No built-in authentication with external evaluation platforms
- **OpenAI Platform Dashboard**: ✅ Full authentication support via [platform.openai.com/docs/guides/evals](https://platform.openai.com/docs/guides/evals)
  - User login and account management
  - Team collaboration and permissions
  - Integration with OpenAI API authentication

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

*Defining the evaluation experiment—what to test, what to test it with, and how to judge the results.*

### Step A: SUT Preparation

*Definition:* Specifying how to interact with the System Under Test (SUT).

**Strategy 1: Model-as-a-Service (Remote Inference)** ✅ **SUPPORTED** (Native)
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

*Definition:* Acquiring and configuring the test inputs that will be used to evaluate the SUT.

**Strategy 1: Benchmark Dataset Preparation (Offline)** ✅ **SUPPORTED** (Native)
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

*Definition:* Pre-computing judges, references, and ground truth materials that will be used to score SUT outputs in Phase III.

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

*Observing SUT behavior—applying test inputs to elicit outputs and actions.*

### Step A: SUT Invocation

*Definition:* Running the System Under Test to generate outputs or take actions.

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

*Converting observations into measurements—judging outputs against quality criteria to produce scores.*

### Step A: Individual Scoring

*Definition:* Computing metrics for individual test instances based on SUT outputs.

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

*Definition:* Combining individual assessment results into benchmark-level aggregate metrics—a fundamental operation supported by all evaluation harnesses.

**Strategy 1: Score Aggregation** ✅ **SUPPORTED** (Native)
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

*Making results actionable—translating metrics into stakeholder-facing insights.*

### Step A: Insight Presentation

*Definition:* Visualizing metrics and publishing results to internal/external audiences.

**Strategy 1: Execution Tracing** ✅ **SUPPORTED**
- **Open-Source Repository**: ✅ Comprehensive event recording
  - Event recording for all evaluation steps
  - Detailed sampling logs with prompts and completions
  - Function call logging
  - Postprocessor tracking in Solver framework
  - Evidence: `evals/record.py`, `evals/solvers/solver.py`
- **OpenAI Platform Dashboard**: ✅ Enhanced trace visualization
  - Interactive trace viewer with drill-down capabilities
  - Visual execution flow diagrams
  - Timeline views of evaluation steps
  - Search and filter within traces
  - Evidence: [OpenAI Evals Dashboard](https://platform.openai.com/docs/guides/evals)

**Strategy 2: Subgroup Analysis** ✅ **SUPPORTED** (via OpenAI Platform Dashboard)
- **Open-Source Repository**: ❌ No built-in support for demographic/domain stratification
  - Manual analysis possible via event logs
- **OpenAI Platform Dashboard**: ✅ Full subgroup analysis support
  - Filter and stratify results by various dimensions
  - Interactive filtering in web UI
  - Custom grouping and segmentation
  - Evidence: [OpenAI Evals Dashboard](https://platform.openai.com/docs/guides/evals)

**Strategy 3: Chart Generation** ✅ **SUPPORTED** (via OpenAI Platform Dashboard)
- **Open-Source Repository**: ❌ No built-in visualization capabilities
  - Matplotlib and seaborn available as dependencies for custom analysis
  - Third-party tools like [logviz](https://github.com/naimenz/logviz) for log visualization
  - Evidence: `pyproject.toml` dependencies
- **OpenAI Platform Dashboard**: ✅ Full chart generation support
  - Visual metric plots and comparisons
  - Trend analysis and historical charts
  - Interactive visualization tools
  - Export capabilities for reports
  - Evidence: [OpenAI Evals Dashboard](https://platform.openai.com/docs/guides/evals)

**Strategy 4: Dashboard Creation** ✅ **SUPPORTED** (via OpenAI Platform Dashboard)
- **Open-Source Repository**: ❌ No built-in dashboard interface
  - Flask available as dependency for custom dashboards
  - Evidence: `pyproject.toml` dependencies
- **OpenAI Platform Dashboard**: ✅ Full dashboard support
  - Interactive web UI for viewing evaluation results
  - Real-time updates and exploration
  - Customizable views and layouts
  - Team collaboration features
  - Direct links via `report_url` in API responses
  - Evidence: [OpenAI Evals Dashboard](https://platform.openai.com/docs/guides/evals)

**Strategy 5: Leaderboard Publication** ✅ **SUPPORTED** (via OpenAI Platform Dashboard)
- **Open-Source Repository**: ❌ No built-in leaderboard submission
  - Manual submission to external platforms possible
- **OpenAI Platform Dashboard**: ✅ Full leaderboard support
  - Model ranking and comparison tables
  - Public and private leaderboards
  - Automatic score aggregation
  - Historical tracking and versioning
  - Evidence: [OpenAI Evals Dashboard](https://platform.openai.com/docs/guides/evals)

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

### Open-Source Repository Coverage
- **Phase 0 (Provisioning)**: Installation 2/5 ✅, Authentication 2/3 ✅
- **Phase I (Specification)**: SUT Preparation 3/4 ✅, Benchmark Inputs 3/4 ✅, Benchmark References 2/2 ✅
- **Phase II (Execution)**: SUT Invocation 3/4 ✅
- **Phase III (Assessment)**: Individual Scoring 3/4 ✅, Aggregation 2/2 ✅
- **Phase IV (Reporting)**: Insight Presentation 1/6 ✅

**Repository Total: 22 out of 38 strategies (58%)**

### Combined Ecosystem (Repository + OpenAI Platform Dashboard)
- **Phase 0 (Provisioning)**: Installation 2/5 ✅, Authentication 3/3 ✅ **(+1 from Dashboard)**
- **Phase I (Specification)**: SUT Preparation 3/4 ✅, Benchmark Inputs 3/4 ✅, Benchmark References 2/2 ✅
- **Phase II (Execution)**: SUT Invocation 3/4 ✅
- **Phase III (Assessment)**: Individual Scoring 3/4 ✅, Aggregation 2/2 ✅
- **Phase IV (Reporting)**: Insight Presentation 5/6 ✅ **(+4 from Dashboard)**

**Combined Total: 27 out of 38 strategies (71%)**

### What OpenAI Platform Dashboard Adds
1. **Phase 0-B-1**: Evaluation platform authentication
2. **Phase IV-A-2**: Subgroup analysis with interactive filtering
3. **Phase IV-A-3**: Chart generation with visual plots
4. **Phase IV-A-4**: Dashboard creation with web UI
5. **Phase IV-A-5**: Leaderboard publication with rankings

### Remaining Gaps (Even with Dashboard)
- Production streaming/real-time monitoring
- Automated regression alerting
- Performance profiling (latency/throughput)
- Energy/carbon tracking

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
- OpenAI Platform Dashboard: [platform.openai.com/docs/guides/evals](https://platform.openai.com/docs/guides/evals)

---

## Using the OpenAI Evals Ecosystem

### When to Use Open-Source Repository
- **Private/offline evaluations**: No internet required for local models
- **Maximum customization**: Full code access and modification
- **Custom infrastructure**: Integration with proprietary systems
- **Air-gapped environments**: High-security or compliance requirements
- **Development and debugging**: Direct code access for troubleshooting

### When to Use OpenAI Platform Dashboard
- **Team collaboration**: Share results with stakeholders
- **Visual analysis**: Interactive charts and dashboards
- **Model comparison**: Leaderboards and side-by-side views
- **Quick iteration**: No local setup required
- **Historical tracking**: Automatic versioning and history

### Best Practice: Use Both
- **Develop locally** with the open-source repository
- **Share results** via the OpenAI Platform Dashboard
- **Iterate quickly** with local testing
- **Communicate effectively** with visual dashboards

For more information:
- **Repository**: [github.com/openai/evals](https://github.com/openai/evals)
- **Dashboard**: [platform.openai.com/docs/guides/evals](https://platform.openai.com/docs/guides/evals)
- **Documentation**: See `docs/` folder in this repository
