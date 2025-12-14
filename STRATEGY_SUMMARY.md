# OpenAI Evals - Strategy Support Quick Reference

This document analyzes which evaluation strategies are supported by the **OpenAI Evals ecosystem**, which includes both:
1. **Open-Source Repository** (this repo) - CLI-based evaluation framework
2. **OpenAI Platform Dashboard** ([platform.openai.com/docs/guides/evals](https://platform.openai.com/docs/guides/evals)) - Web-based evaluation UI

## 🎯 Scope & Coverage

**Important Note**: The OpenAI Platform Dashboard provides additional capabilities beyond the open-source repository analyzed here. This document combines both to give a complete picture.

### Combined Coverage Statistics
- **Open-Source Repository Only**: 23/38 strategies (61%)
- **With OpenAI Platform Dashboard**: 28/38 strategies (74%)

### Key Dashboard Enhancements
The OpenAI Platform Dashboard adds support for:
1. **Dashboard Creation** ✅ - Interactive web UI for viewing results
2. **Chart Generation** ✅ - Visual metric plots and comparisons  
3. **Leaderboard Publication** ✅ - Model comparison and ranking views
4. **Subgroup Analysis** ✅ - Filtering and stratification capabilities
5. **Execution Tracing** ✅ - Enhanced trace visualization (beyond JSONL logs)

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

**Via OpenAI Platform Dashboard** ([Get started →](https://platform.openai.com/docs/guides/evals)):
- ✅ **Interactive Dashboard UI** - Web-based evaluation result viewer
- ✅ **Chart Generation** - Visual metric plots, comparisons, and trend analysis
- ✅ **Execution Tracing** - Enhanced trace visualization with drill-down capabilities
- ✅ **Leaderboard Views** - Model ranking and comparison tables
- ✅ **Subgroup Analysis** - Filter and stratify results by various dimensions
- ✅ **report_url** - Direct links from API responses to visual dashboard

**Via Open-Source Repository**:
- ✅ **JSONL Event Logs** - Detailed execution traces for programmatic analysis
- ✅ **Third-party tools** - Integration with logviz, Matplotlib, Seaborn for custom visualization

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
- ~~Evaluation platform/leaderboard authentication~~ (Now supported via OpenAI Platform Dashboard login)

### System Types
- Specialized algorithms (ANN, knowledge graphs) without wrapping in LLM interface

### Data Sources
- Real-time production traffic sampling
- Online stream processing

### Execution
- Continuous production monitoring (real-time streaming)

### Reporting
- ~~Automated leaderboard submission~~ (Now supported via OpenAI Platform Dashboard)
- Automated regression detection with alerting
- ~~Demographic/domain subgroup analysis~~ (Now supported via OpenAI Platform Dashboard filtering)

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

**Combined (Open-Source + Platform Dashboard):**
1. **Flexible model support**: Remote APIs + **full local model inference** (transformers, vLLM, llama.cpp) via custom completion functions
2. **Rich evaluation types**: From simple string matching to complex LLM-judged criteria
3. **Interactive capabilities**: Multi-turn dialogues, stateful agents, tool use
4. **Extensible framework**: Custom completion functions, solvers, and evals
5. **Multiple recording backends**: Local files, Snowflake, HTTP, cloud storage
6. **AI-assisted dataset creation**: Generate test data with GPT-4 via OpenAI Platform
7. **Full visualization suite**: OpenAI Platform Dashboard with charts, leaderboards, trace viewers, and subgroup analysis

**Open-Source Repository Strengths:**
- Maximum flexibility and customization
- Run evaluations offline or in private environments
- Full control over data and infrastructure
- Integration with custom tooling and workflows

## Key Gaps

**Even with OpenAI Platform Dashboard:**
1. **No production monitoring**: Not designed for continuous production evaluation or real-time streaming
2. **Limited performance profiling**: No built-in latency/throughput measurement
3. **No automated regression alerting**: Manual monitoring required
4. **No energy/carbon tracking**: Environmental impact metrics not available

**Open-Source Repository Only:**
5. ~~**No built-in visualization**~~ (Addressed by OpenAI Platform Dashboard)
6. ~~**No leaderboard views**~~ (Addressed by OpenAI Platform Dashboard)  
7. ~~**No subgroup analysis**~~ (Addressed by OpenAI Platform Dashboard filtering)

## Comparison to Problem Statement

According to the problem statement taxonomy:

### Open-Source Repository Coverage
- **Phase 0 (Provisioning)**: 4 out of 7 strategies (57%)
- **Phase I (Specification)**: 7 out of 10 strategies (70%)
- **Phase II (Execution)**: 3 out of 4 strategies (75%)
- **Phase III (Assessment)**: 5 out of 6 strategies (83%)
- **Phase IV (Reporting)**: 1 out of 6 strategies (17%)

**Repository Only: 23 out of 38 strategies (61%)**

### Combined (Repository + OpenAI Platform Dashboard) Coverage
- **Phase 0 (Provisioning)**: 5 out of 7 strategies (71%) - *+1 from platform auth*
- **Phase I (Specification)**: 7 out of 10 strategies (70%) - *unchanged*
- **Phase II (Execution)**: 3 out of 4 strategies (75%) - *unchanged*
- **Phase III (Assessment)**: 5 out of 6 strategies (83%) - *unchanged*
- **Phase IV (Reporting)**: 5 out of 6 strategies (83%) - *+4 from dashboard UI, charts, leaderboards, subgroup analysis*

**Combined Total: 28 out of 38 strategies (74%)**

### What OpenAI Platform Dashboard Adds
1. **Phase 0-B**: Platform authentication (evaluation platform login)
2. **Phase IV-A-2**: Subgroup analysis (dashboard filtering/stratification)
3. **Phase IV-A-3**: Chart generation (visual metric plots)
4. **Phase IV-A-4**: Dashboard creation (interactive web UI)
5. **Phase IV-A-5**: Leaderboard publication (model ranking views)

The ecosystem is strongest in assessment (83%) and execution (75%). With the Dashboard, reporting improves from 17% to 83%. Weakest areas remain production monitoring and real-time streaming.

## Recommended Use Cases

### ✅ Excellent Fit (Open-Source + Dashboard)
- Academic benchmark evaluation with public results
- Model comparison studies with visual dashboards
- Team collaboration on evaluation workflows
- Prompt engineering experiments with historical tracking
- Agent capability testing with trace visualization
- RAG system evaluation with metric charting
- Multi-turn dialogue assessment
- Tool use evaluation

### ✅ Good Fit (Open-Source Only)
- Private/offline evaluation workflows
- Custom evaluation logic and metrics
- Integration with proprietary systems
- Air-gapped or high-security environments
- Maximum customization requirements

### ❌ Poor Fit (Even with Dashboard)
- Real-time production model monitoring
- Continuous evaluation with live traffic
- Automated regression detection with alerting
- Performance benchmarking (latency/throughput)
- Energy efficiency measurement
- Compliance-driven demographic analysis

## 📚 Additional Resources

- **OpenAI Platform Dashboard**: [platform.openai.com/docs/guides/evals](https://platform.openai.com/docs/guides/evals)
- **Open-Source Repository**: [github.com/openai/evals](https://github.com/openai/evals)
- **Documentation**: See `docs/` folder in this repository
- **Third-Party Tools**: [logviz](https://github.com/naimenz/logviz) for log visualization
