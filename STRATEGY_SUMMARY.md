# OpenAI Evals - Strategy Support Quick Reference

This is a quick reference guide for which evaluation strategies are supported by the OpenAI Evals harness. For detailed information, see [HARNESS_STRATEGIES_ANALYSIS.md](HARNESS_STRATEGIES_ANALYSIS.md).

## 📝 Key Updates (Based on Official Documentation)

**Three major capabilities were initially underestimated:**

1. **Local Model Inference** ✅ - Full support via custom completion functions (transformers, vLLM, llama.cpp)
2. **Synthetic Data Generation** ✅ - OpenAI Platform can generate test data with GPT-4
3. **Dashboard & Visualization** ✅ - OpenAI Platform Dashboard with report_url, execution tracing, and visual result exploration

**Updated Coverage**: 25 out of 38 strategies (66%)

## ✅ Fully Supported Strategies

### Installation & Setup
- PyPI package installation (`pip install evals`)
- Git clone with Git LFS for large datasets
- Docker containers (for specific evals like multistep_web_tasks)

### Authentication
- OpenAI API (GPT-3.5, GPT-4)
- Anthropic (Claude)
- Google Gemini
- Together AI
- HuggingFace Hub (via LangChain)
- Custom API endpoints

### Model/System Types
- Remote API-based models (OpenAI, Anthropic, Google, Together)
- **Local models via custom completion functions** (transformers, vLLM, llama.cpp, local HuggingFace checkpoints)
- Local models via LangChain wrappers
- Stateful agents and policies (via Solver framework)
- Multi-turn interactive systems
- Tool-using agents

### Datasets & Benchmarks
- JSONL format datasets (local and cloud storage)
- Git LFS for large datasets
- **AI-generated test data** (via OpenAI Platform using GPT-4 for diverse test cases and edge cases)
- Simulated environments (WebArena, Docker-based)
- Ground truth with reference answers
- LLM-as-judge configurations

### Execution Modes
- Batch inference (parallel or sequential)
- Interactive multi-turn dialogues
- Head-to-head model comparisons (arena battles)
- Multi-agent scenarios (e.g., influencer vs voter)

### Scoring Methods
- Exact string matching
- Substring/fuzzy matching
- JSON structural equality
- Embedding-based similarity
- LLM-as-judge (subjective evaluation)
- Accuracy, F-score, precision, recall
- Bootstrap uncertainty quantification

### Recording & Logging
- Local JSON Lines files
- Snowflake database
- HTTP POST to custom endpoints
- Console-only (dry-run)
- Cloud storage (GCS, Azure Blob)

### Visualization & Reporting
- **OpenAI Platform Dashboard** with interactive UI for viewing evaluation results
- **Execution tracing** via JSONL logs and Dashboard Traces view
- **report_url** in API responses linking to visual dashboard
- Third-party visualization tools (logviz for log visualization)
- Integration with Matplotlib/Seaborn for custom charts

## ⚠️ Partially Supported Strategies

### Limited Container Support
- Docker used for specific evals (multistep_web_tasks)
- Not a general-purpose container strategy

### Limited Performance Measurement
- Token usage tracking via API
- No built-in latency/throughput profiling
- No energy/carbon measurement

## ❌ Not Supported Strategies

### Installation
- Binary packages (standalone executables)
- Node.js/npm packages

### Authentication
- Evaluation platform/leaderboard authentication

### System Types
- Specialized algorithms (ANN, knowledge graphs) without wrapping in LLM interface

### Data Sources
- Real-time production traffic sampling
- Online stream processing

### Execution
- Continuous production monitoring

### Reporting
- Automated leaderboard submission
- Regression detection/alerting
- Demographic/domain subgroup analysis

## 🔍 Undocumented but Functional Features

These features work but aren't prominently documented:

1. **HttpRecorder** - POST evaluation results to HTTP endpoints
2. **Multi-model evaluations** - Multiple models in one eval (e.g., ballots)
3. **Docker environments** - For simulated internet/web tasks
4. **Cloud storage paths** - GCS and Azure Blob support
5. **Solver postprocessors** - Output transformation pipeline
6. **HumanCliSolver** - Interactive human evaluation
7. **External registries** - Load custom eval/completion function registries
8. **Meta-evaluations** - Evaluate the evaluators themselves
9. **Nested solvers** - CoT, few-shot, self-consistency wrappers
10. **Rate limiting & retries** - Automatic API error handling

## Key Strengths

1. **Flexible model support**: Remote APIs + **full local model inference** (transformers, vLLM, llama.cpp) via custom completion functions
2. **Rich evaluation types**: From simple string matching to complex LLM-judged criteria
3. **Interactive capabilities**: Multi-turn dialogues, stateful agents, tool use
4. **Extensible framework**: Custom completion functions, solvers, and evals
5. **Multiple recording backends**: Local files, Snowflake, HTTP, cloud storage
6. **AI-assisted dataset creation**: Generate test data with GPT-4 via OpenAI Platform
7. **Visualization & dashboards**: OpenAI Platform Dashboard with report_url, execution tracing

## Key Gaps

1. **No production monitoring**: Not designed for continuous production evaluation
2. **No leaderboard integration**: Manual submission required
3. **Limited performance profiling**: No latency/throughput measurement
4. **No subgroup analysis**: Can't stratify by demographics/domains automatically
5. **Limited built-in charts**: Relies on third-party tools (logviz) or custom Matplotlib/Seaborn integration for detailed charting

## Comparison to Problem Statement

According to the problem statement taxonomy, OpenAI Evals supports:

- **Phase 0 (Provisioning)**: 2 out of 5 strategies (40%)
- **Phase I (Specification)**: 6 out of 9 strategies (67%)
- **Phase II (Execution)**: 1 out of 4 strategies (25%)
- **Phase III (Assessment)**: 4 out of 6 strategies (67%)
- **Phase IV (Reporting)**: 4 out of 6 strategies (67%)

**Overall: 25 out of 38 strategies (66%)**

The harness is strongest in specification (dataset prep, model config, judging), assessment (scoring methods), and reporting (dashboards, tracing, charts). It's weakest in production monitoring and specialized execution modes.

## Recommended Use Cases

### ✅ Good Fit
- Academic benchmark evaluation
- Model comparison studies
- Prompt engineering experiments
- Agent capability testing
- RAG system evaluation
- Multi-turn dialogue assessment
- Tool use evaluation
- Custom evaluation logic

### ❌ Poor Fit
- Production model monitoring
- Real-time continuous evaluation
- Automated regression detection with alerting
- Public leaderboard submission (automated)
- Performance benchmarking (latency/throughput)
- Energy efficiency measurement
- Demographic/domain subgroup analysis
