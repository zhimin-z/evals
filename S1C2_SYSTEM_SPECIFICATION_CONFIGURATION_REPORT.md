# S1C2: System Specification & Configuration

**Grade: Present (100% coverage)**

## Supports:
- Multiple model provider support ✓
- Exact version specification ✓
- Inference parameter configuration ✓
- Parameter validation ✓
- Resource specification ✓
- Resource validation ✓
- Multi-model comparative setup ✓
- Configuration reusability ✓

**Coverage calculation:** 8/8 = 100%

## Documentation: Comprehensive

The harness provides thorough documentation across completion function guides, CLI documentation, example solver configurations, and clear examples of parameter configuration in YAML format with best practices and use cases.

---

## Evidence

### 1. Multiple Model Provider Support ✓
- **OpenAI**: `evals/registry/solvers/defaults.yaml:16-361` with GPT-4, GPT-3.5-turbo, GPT-4-turbo-preview, GPT-4-base configurations
- **Anthropic**: `evals/registry/solvers/anthropic.yaml:1-126` supports Claude 3 Opus/Sonnet/Haiku, Claude 2.x, and Claude Instant variants with exact versioning (e.g., `claude-3-opus-20240229`)
- **Google Gemini**: `evals/registry/solvers/gemini.yaml` with Gemini Pro configurations
- **LangChain Integration**: `evals/completion_fns/langchain_llm.py` enables HuggingFace, custom, and other LangChain-supported models
- **OpenAI Assistants API**: `evals/registry/solvers/defaults.yaml:308-361` with tools support (code_interpreter, retrieval)

### 2. Exact Version Specification ✓
- **Pin by model ID**: `evals/registry/solvers/anthropic.yaml:8,29,49,71,91,111` shows exact model versioning (e.g., `model_name: claude-3-opus-20240229`)
- **OpenAI model variants**: `evals/registry/solvers/defaults.yaml:20,89,158` pinned versions: `gpt-3.5-turbo`, `gpt-4`, `gpt-4-turbo-preview`, `gpt-4-base`
- **Version mapping**: `evals/registry.py` provides `n_ctx_from_model_name()` function for context window lookup by exact model version
- **API version control**: `evals/completion_fns/openai.py:88` accepts optional `api_base` for custom endpoint versioning

### 3. Inference Parameter Configuration ✓
- **Temperature control**: `evals/registry/solvers/defaults.yaml:22,34,53,66` shows temperature values (generation: temperature=1, classification: temperature=0)
- **Max tokens**: `evals/registry/solvers/defaults.yaml:23,35,54,75` configures `max_tokens: 512` for generation, `max_tokens: 1` for classification
- **Extra options dict**: `evals/completion_fns/openai.py:90,97,122` passes arbitrary parameters via `extra_options: dict` (temperature, max_tokens, stop sequences, frequency_penalty, presence_penalty)
- **Sampling strategy**: Nested solver support for chain-of-thought and HHH reasoning patterns: `evals/registry/solvers/defaults.yaml:25-43,94-112,126-145`

### 4. Parameter Validation ✓
- **Type checking**: Pydantic dataclasses enforce type validation on solver specifications (solver metadata in registry)
- **OpenAI SDK validation**: `evals/completion_fns/openai.py:19-24` defines timeout exception catching with typed exception handling
- **Model validation**: Registry lookup validates model names against registered completion functions
- **Implicit API validation**: Parameters passed to OpenAI API are validated by the OpenAI Python SDK

### 5. Resource Specification ✓
- **Thread concurrency**: `docs/run-evals.md:31-36` documents `EVALS_THREADS` environment variable for thread pool configuration
- **Per-thread timeout**: `docs/run-evals.md:31-36` shows `EVALS_THREAD_TIMEOUT` environment variable (default 40 seconds, configurable to higher values for long-running samples)
- **Max samples**: `evals/cli/oaieval.py:40` provides `--max_samples` flag to limit evaluation size
- **Batch processing**: HTTP mode with `--http-batch-size` parameter in `evals/cli/oaieval.py:79-82` for batch result sending

### 6. Resource Validation ✓
- **Timeout handling**: `evals/completion_fns/openai.py:19-24` defines `OPENAI_TIMEOUT_EXCEPTIONS` with typed catching of rate limits, connection errors, timeout errors, and server errors
- **Retry mechanism**: `evals/completion_fns/openai.py:27-38` implements `openai_completion_create_retrying()` with `create_retrying()` utility
- **Error detection**: `evals/api_utils.py` implements exponential backoff retry with 60-second max wait
- **HTTP failure threshold**: `evals/cli/oaieval.py:84-89` allows configurable `--http-fail-percent-threshold` with validation (default 5%)

### 7. Multi-Model Comparative Setup ✓
- **Comma-separated models**: `evals/cli/oaieval.py:28-30` accepts "One or more CompletionFn URLs, separated by commas"
- **Example usage**: `oaieval "gpt-3.5-turbo,gpt-4" test-match` enables head-to-head comparison
- **Nested solvers for chains**: `evals/registry/solvers/defaults.yaml:25-43` shows CoTSolver with different models for thinking vs. extraction (chain-of-thought ablation)
- **Model-graded evaluation**: Multiple models in single eval (grader + solver models)

### 8. Configuration Reusability ✓
- **YAML registry system**: `evals/registry/solvers/defaults.yaml` (20+ configurations), `anthropic.yaml`, `gemini.yaml` enable save/load/version of complete solver specifications
- **CLI parameter overrides**: `evals/cli/oaieval.py:35-39` provides `--completion_args` flag for runtime parameter modification: `--completion_args "temperature=0.5,max_tokens=1024"`
- **Custom registry paths**: `evals/cli/oaieval.py:50-55` accepts `--registry_path` for external configuration management
- **Extra eval parameters**: `evals/cli/oaieval.py:33` supports `--extra_eval_params` for eval-specific customization
- **External registry example**: `docs/completion-fns.md:42-49` demonstrates registering completion functions outside core codebase via `--registry_path ~/my_project`

---

## Key Strengths

1. **Declarative configuration** - YAML-based solver registry enables reproducible, version-controlled system specifications
2. **Multi-provider flexibility** - Unified `CompletionFn` protocol abstracts across OpenAI, Anthropic, Google, LangChain, and custom endpoints
3. **Fine-grained control** - Temperature, max_tokens, and arbitrary parameters configurable at both registry and CLI levels
4. **Comparative evaluation** - Comma-separated model syntax enables easy head-to-head and ablation studies
5. **Robust error handling** - Typed exception catching with exponential backoff for transient failures
6. **Resource management** - Thread pooling with per-thread timeouts prevents resource exhaustion on large eval runs
7. **Extensibility** - Custom registry paths allow external solver/completion function definitions without modifying core codebase

---

## Configuration Examples

### Single Model Evaluation
```bash
oaieval gpt-3.5-turbo test-match
```

### Multi-Model Comparison
```bash
oaieval "gpt-3.5-turbo,gpt-4" test-match
```

### Parameter Override
```bash
oaieval gpt-4 test-match --completion_args "temperature=0.7,max_tokens=2048"
```

### Custom Registry
```bash
oaieval my_completion_fn test-match --registry_path ~/my_evals
```

### Resource Configuration
```bash
EVALS_THREADS=42 EVALS_THREAD_TIMEOUT=600 oaievalset gpt-3.5-turbo test
```

---

## Solver Configuration Files

- **OpenAI defaults**: `evals/registry/solvers/defaults.yaml` - 15+ generation/classification solver configs for GPT models
- **Anthropic**: `evals/registry/solvers/anthropic.yaml` - 10+ configs for Claude family
- **Google Gemini**: `evals/registry/solvers/gemini.yaml`
- **Specialized**: `function_deduction.yaml`, `make-me-pay.yaml`, `multistep_web_tasks.yaml` for domain-specific solver chains

---

## Supporting Documentation

- **Run evals guide**: `docs/run-evals.md` - CLI usage, threading configuration, resume capability
- **Completion functions**: `docs/completion-fns.md` - How to register and implement custom completion functions
- **Completion function protocol**: `docs/completion-fn-protocol.md` - Interface specification for custom implementations
- **API documentation**: `evals/api.py` - `CompletionFn` protocol interface definition
- **Completion function examples**: `evals/completion_fns/` directory with OpenAI, LangChain implementations

---

## Version Control & Reproducibility

- Exact model versions pinned in YAML (e.g., `claude-3-opus-20240229`)
- Configuration files tracked in git for audit trail
- Runtime parameter overrides via CLI with explicit command logging
- JSONL-based result recording preserves all sampling metadata
- Environment variable configuration enables consistent infrastructure setup
