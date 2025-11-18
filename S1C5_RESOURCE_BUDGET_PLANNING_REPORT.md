# S1C5: Resource Budget Planning

**Grade: Partial (37.5% coverage)**

## Supports:
- Compute budget specification ✗
- Cost limit specification ✗
- Time constraint specification ✓
- Cost estimation before run ✗
- Token-based cost modeling ✓
- Tradeoff analysis ✗
- Budget enforcement ✗
- Cost reporting ✓

**Coverage calculation:** 3/8 = 37.5%

## Documentation: Minimal

Cost tracking is mentioned in README files and documentation, but there is no comprehensive guide for budget planning, cost estimation, or tradeoff analysis. Cost reporting is automatic but requires manual interpretation and Excel/Python analysis for meaningful insights.

---

## Evidence

### 1. Compute Budget Specification ✗
- **Not implemented**: No GPU/CPU allocation specification parameters
- **Workaround**: Thread pool sizing via `EVALS_THREADS` environment variable (`evals/eval.py:124`)
- **Limitation**: Cannot specify maximum compute duration or resource limits
- **Current capability**: Only sample-level control via `--max_samples` CLI flag (`evals/cli/oaieval.py:40`)

### 2. Cost Limit Specification ✗
- **Not implemented**: No `--max_cost` or `--budget` parameter
- **No spending enforcement**: System continues running until all samples completed, regardless of cost
- **Manual tracking required**: Users must monitor their OpenAI account dashboard for costs
- **Workaround**: Use `--max_samples` to control dataset size and hence cost

### 3. Time Constraint Specification ✓
- **Thread timeout configuration**: `evals/utils/api_utils.py:6` with `EVALS_THREAD_TIMEOUT` environment variable
  ```python
  EVALS_THREAD_TIMEOUT = float(os.environ.get("EVALS_THREAD_TIMEOUT", "40"))
  ```
  - Default: 40 seconds per thread
  - Configurable: `EVALS_THREAD_TIMEOUT=600 oaievalset gpt-3.5-turbo test`
  - Purpose: Prevents hanging on individual API calls
  - **Documentation**: `docs/run-evals.md:33-34`

- **Thread concurrency control**: `evals/eval.py:124` with `EVALS_THREADS` environment variable
  - Default: 10 threads
  - Configurable: `EVALS_THREADS=42 oaievalset gpt-3.5-turbo test`
  - **Trade-off**: More threads = faster evaluation but higher concurrent API load

- **Sequential mode**: `EVALS_SEQUENTIAL=1` for serial execution (`evals/eval.py:140-142`)
  - Required for certain eval types (human input, agent bench)
  - Reduces concurrency to 1

- **Limitation**: No eval-level deadline specification; only per-thread timeouts

### 4. Cost Estimation Before Run ✗
- **Not implemented**: No pre-run cost calculator
- **Manual estimation provided**: Documentation includes token usage tables
  - **Theory of Mind** (`evals/elsuite/theory_of_mind/readme.md:10-15`): 250k-2.4m tokens depending on model/variant
  - **Identifying Variables** (`evals/elsuite/identifying_variables/README.md:110-131`): Detailed token breakdown by model
    ```
    GPT-4-base (corrset): 1,450,000 tokens/run
    GPT-3.5-turbo Direct: 518,000 tokens/run
    (multiply by 10 for -large datasets)
    ```
  - **HR ML Agent Bench** (`evals/elsuite/hr_ml_agent_bench/README.md`): Task-specific token ranges

- **Manual cost calculation**: Users must multiply tokens by current OpenAI pricing
  - Example: 1.45M tokens × $0.01/1M tokens = $14.50 per run

- **No automated prediction**: Cannot specify dataset size and get cost estimate

### 5. Token-Based Cost Modeling ✓
- **Token counting at pre-flight**: `evals/solvers/providers/openai/openai_solver.py:159-179` with tiktoken library
  ```python
  def _perform_prechecks(self, msgs):
      enc = tiktoken.encoding_for_model(self.model)
      ctx_len = n_ctx_from_model_name(self.model)
      n_tokens = sum(len(enc.encode(msg["content"])) for msg in msgs)

      if n_tokens >= ctx_len:
          return SolverResult(output=f"Request too large. Context: {ctx_len}. Requested: {n_tokens}.")
  ```
  - **Purpose**: Avoid wasting tokens on requests exceeding context length
  - **Library**: tiktoken for accurate token counting per model

- **Token aggregation**: `evals/cli/oaieval.py:269-295` in `add_token_usage_to_result()`
  ```python
  sampling_events = recorder.get_events("sampling")
  total_usage = {
      key: sum(u[key] if u[key] is not None else 0 for u in usage_events)
      for key in usage_events[0]
  }
  # Aggregates: prompt_tokens, completion_tokens, total_tokens
  ```

- **Usage capture**: OpenAI API response includes `usage` object
  - **File**: `evals/completion_fns/openai.py:125-130`
  - Data captured: `prompt_tokens`, `completion_tokens`, `total_tokens`
  - Recorded via `record_sampling(usage=result.raw_data.usage)`

- **Multi-provider support**: Anthropic usage converted to OpenAI format
  - **File**: `evals/solvers/providers/anthropic/anthropic_solver.py:68-73, 132-142`
  - Conversion function: `anth_to_openai_usage()` maps `input_tokens`, `output_tokens` to standard schema

- **Context length trimming**: Dynamic conversation history truncation
  - **File**: `evals/elsuite/multistep_web_tasks/solvers/strong_solver/strong_solver.py:126-150`
  - Calculation: `target_tokens = context_length - max_response_tokens - TOKEN_BUFFER`
  - Constants: `TOKENS_PER_MESSAGE=4`, `TOKEN_BUFFER=10`

### 6. Tradeoff Analysis ✗
- **Not implemented**: No built-in utilities for cost vs. accuracy analysis
- **No cost-benefit calculator**: Cannot answer "Which is better: 10 samples with human judges or 100 with LLM judge?"
- **Manual analysis possible**: JSONL logs contain both metrics and token usage per sample
  - Location: `/tmp/evallogs/{run_id}_{model}_{eval}.jsonl` (default)
  - Customizable: `--record_path` CLI flag
  - Can post-process logs with Python for custom analysis

### 7. Budget Enforcement ✗
- **No hard spending limits**: System cannot be configured to stop when cost threshold reached
- **Context length enforcement only**:
  - Pre-checks prevent requests exceeding model's context length (`openai_solver.py:174-177`)
  - Returns error result instead of API call: "Request too large for {model}. Context: {ctx_len}. Requested: {n_tokens}."
  - **Benefit**: Prevents wasting tokens on guaranteed-to-fail requests

- **Graceful interruption possible**:
  - File: `evals/eval.py:237-253`
  - Environment variable: `EVALS_GENTLE_INTERRUPT`
  - Allows early stopping with partial results via KeyboardInterrupt
  - Does not enforce spending limits automatically

- **No rate limiting**: No tracking against OpenAI TPM (tokens per minute) limits
  - Relies on OpenAI API to reject requests exceeding rate limits

### 8. Cost Reporting ✓
- **Console output**: Token usage printed at evaluation completion
  - **File**: `evals/cli/oaieval.py:285-286`
  - Format: Comma-separated integers for readability
  - Example output:
    ```
    Token usage from 100 sampling events:
    prompt_tokens: 1,234,567
    completion_tokens: 234,567
    total_tokens: 1,469,134
    ```

- **Result dictionary**: Token usage added to final report
  - **File**: `evals/cli/oaieval.py:287-290`
  - Keys added:
    - `usage_prompt_tokens`: Total input tokens
    - `usage_completion_tokens`: Total output tokens
    - `usage_total_tokens`: Sum of above
  - Accessible for downstream analysis or logging

- **JSONL event logs**: All sampling events with full metadata
  - **File**: `evals/record.py:210-216` (record_sampling method)
  - Recorded per-sample with: `prompt`, `sampled`, `usage`, `model`, `sample_id`
  - Enables: Per-sample cost analysis, cost per accuracy breakdown

- **Snowflake database logging** (optional):
  - **File**: `evals/record.py:514-560` (Recorder class)
  - Events inserted into remote database for long-term cost tracking
  - Can query historical costs across many runs

- **CSV export ready**: JSONL format directly convertible to pandas DataFrame
  - Enables cost analysis with: `df['usage']['total_tokens']` etc.

---

## Key Strengths

1. **Accurate token counting** - Uses tiktoken library for model-specific token counts
2. **Pre-flight validation** - Prevents wasting tokens on oversized requests
3. **Multi-provider support** - Normalizes usage across OpenAI and Anthropic
4. **Comprehensive logging** - All sampling events recorded with metadata for analysis
5. **Thread-level timeout control** - Prevents hanging on individual API calls
6. **Flexible parallelization** - Trade off evaluation speed vs. concurrent API load via `EVALS_THREADS`

---

## Key Limitations

1. **No cost prediction** - Cannot estimate total cost before running evaluation
2. **No spending limits** - Cannot prevent runs from exceeding budget
3. **No tradeoff tools** - Must manually analyze cost vs. accuracy
4. **No rate limiting** - Relies on API provider rate limits
5. **No deadline specification** - Only per-thread timeouts available
6. **No pricing integration** - Must manually look up and apply pricing formulas

---

## Cost Estimation Examples (Manual)

### Example 1: Theory of Mind Evaluation
**Configuration**:
- Dataset: SocialIQA with CoT prompting
- Estimated tokens: 900,000
- Model: GPT-3.5-turbo
- Current pricing: $0.0005/1K input, $0.0015/1K output

**Manual calculation** (from documentation):
```
Assume: 75% input tokens, 25% output tokens
Input: 675,000 tokens × $0.0005/1K = $0.34
Output: 225,000 tokens × $0.0015/1K = $0.34
Total: ~$0.68 per run
```

### Example 2: Identifying Variables with GPT-4
**Configuration** (from documentation):
- Dataset: Standard (500 samples)
- Model: GPT-4-base
- Tokens per run: 1,450,000
- Current pricing: $0.03/1K input, $0.06/1K output

**Manual calculation**:
```
Assume: 70% input, 30% output
Input: 1,015,000 × $0.03/1K = $30.45
Output: 435,000 × $0.06/1K = $26.10
Total: ~$56.55 per run
```

---

## Configuration Examples

### Thread and Timeout Control
```bash
# Fast parallel evaluation (higher cost, faster results)
EVALS_THREADS=42 EVALS_THREAD_TIMEOUT=600 oaievalset gpt-3.5-turbo test

# Slow serial evaluation (lower cost, slower results)
EVALS_SEQUENTIAL=1 oaievalset gpt-3.5-turbo test

# Limited dataset for cost control
oaieval gpt-3.5-turbo my-eval --max_samples 100
```

### Max Tokens Configuration (in YAML)
```yaml
my_eval/solver:
  class: evals.solvers.providers.openai.openai_solver:OpenAISolver
  args:
    completion_fn_options:
      model: gpt-4
      extra_options:
        temperature: 0.7
        max_tokens: 500  # Limits output tokens per request
```

---

## Potential Enhancements

1. **Cost limiting** - Add `--max_cost $100` parameter with spending checks
2. **Cost prediction** - Calculate estimated cost before run based on model, samples, and token estimates
3. **Pricing configuration** - Store API pricing in registry for automatic cost calculation
4. **Tradeoff analysis** - Utility function: `suggest_configuration(target_cost=100, metric='accuracy')`
5. **Cost dashboards** - Post-processing script to visualize costs vs. metrics
6. **Budget alerts** - Warn when approaching spending limits during evaluation
7. **Rate limit management** - Queue requests to respect TPM limits
8. **Historical cost tracking** - Dashboard of costs per eval over time

---

## Token Usage Patterns

The system records token usage in sampling events, enabling analysis like:

```python
import json

# Load JSONL log
with open('evallogs/run_id_model_eval.jsonl') as f:
    events = [json.loads(line) for line in f]

# Filter sampling events
sampling_events = [e for e in events if e['type'] == 'sampling']

# Calculate cost per sample
total_tokens = sum(e['data']['usage']['total_tokens'] for e in sampling_events)
samples = len(sampling_events)
tokens_per_sample = total_tokens / samples

# Cost calculation (manual)
price_per_1k = 0.002  # User provides
total_cost = total_tokens * price_per_1k / 1000
cost_per_sample = total_cost / samples
```

---

## Summary

The harness provides **token-based cost tracking and reporting** but lacks comprehensive budget planning capabilities. Token usage is accurately captured at pre-flight (preventing wasted calls) and post-flight (for reporting), enabling manual cost analysis. However, there is no automated cost prediction, spending limits, or tradeoff analysis tools. The framework supports efficient resource utilization via thread control and timeout configuration, but users must manually implement budget governance. Cost analysis requires post-processing JSONL logs or database queries.
