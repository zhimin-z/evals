# OpenAI Evals - Strategy Support Quick Reference

This is a quick reference guide for which evaluation strategies are supported by the OpenAI Evals harness. For detailed information, see [HARNESS_STRATEGIES_ANALYSIS.md](HARNESS_STRATEGIES_ANALYSIS.md).

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
- Local models via HuggingFace + LangChain
- Stateful agents and policies (via Solver framework)
- Multi-turn interactive systems
- Tool-using agents

### Datasets & Benchmarks
- JSONL format datasets (local and cloud storage)
- Git LFS for large datasets
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

## ⚠️ Partially Supported Strategies

### Limited Container Support
- Docker used for specific evals (multistep_web_tasks)
- Not a general-purpose container strategy

### Limited Synthetic Data
- Helper scripts for generating eval data
- Not a runtime data generation feature

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
- Built-in visualization/charts
- Interactive dashboards
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

1. **Flexible model support**: Remote APIs + local models via LangChain
2. **Rich evaluation types**: From simple string matching to complex LLM-judged criteria
3. **Interactive capabilities**: Multi-turn dialogues, stateful agents, tool use
4. **Extensible framework**: Custom completion functions, solvers, and evals
5. **Multiple recording backends**: Local files, Snowflake, HTTP, cloud storage

## Key Gaps

1. **No production monitoring**: Not designed for continuous production evaluation
2. **No visualization**: No built-in charts, dashboards, or reports
3. **No leaderboard integration**: Manual submission required
4. **Limited performance profiling**: No latency/throughput measurement
5. **No subgroup analysis**: Can't stratify by demographics/domains automatically

## Comparison to Problem Statement

According to the problem statement taxonomy, OpenAI Evals supports:

- **15 out of 22 Phase 0-I strategies** (68%)
- **3 out of 4 Phase II strategies** (75%)
- **4 out of 6 Phase III strategies** (67%)
- **1 out of 6 Phase IV strategies** (17%)

**Overall: 23 out of 38 strategies (61%)**

The harness is strongest in model/system preparation, dataset management, execution, and scoring. It's weakest in reporting/visualization and production monitoring.

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
- Real-time quality dashboards
- Automated regression detection
- Public leaderboard submission
- Performance benchmarking (latency/throughput)
- Energy efficiency measurement
