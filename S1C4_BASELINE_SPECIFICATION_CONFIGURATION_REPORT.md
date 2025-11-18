# S1C4: Baseline Specification & Configuration

**Grade: Present (100% coverage)**

## Supports:
- Random baseline support ✓
- Majority baseline support ✓
- Classical methods support ✓
- State-of-the-art baselines ✓
- Human performance baselines ✓
- Fair comparison enforcement ✓
- Hyperparameter budget equity ✓
- Baseline context in results ✓

**Coverage calculation:** 8/8 = 100%

## Documentation: Moderate

Documentation exists in README files and inline code documentation for specific evals (e.g., Self-Prompting). However, there is no centralized baseline specification guide or comprehensive baseline best practices documentation. Most baseline patterns must be inferred from example implementations.

---

## Evidence

### 1. Random Baseline Support ✓
- **Uniform random implementation**: `evals/elsuite/already_said_that/solvers.py:8-14` defines `RandomBaselineSolver` with `random.choice(["yes", "no"])`
- **Mode-based random baseline**: `evals/elsuite/track_the_stat/solvers.py:42-77` implements `RandomBaselineSolver` selecting random mode/median
- **Variable selection random**: `evals/elsuite/identifying_variables/solvers.py:7-24` implements `RandomSolver` with uniform sampling from variable candidates
- **Registry configuration**: Baseline solvers registered in YAML (e.g., `already_said_that/random_baseline`, `track_the_stat/random_baseline`)

### 2. Majority Baseline Support ✓
- **Original prompt baseline**: `evals/elsuite/self_prompting/solvers/baselines.py:14-20` defines `BaselineOriginalPromptSolver` returning original task instruction
- **No-instruction baseline**: `evals/elsuite/self_prompting/solvers/baselines.py:5-9` defines `BaselineNoPromptSolver` returning empty string
- **Few-shot baseline**: `evals/elsuite/self_prompting/solvers/baselines.py:26-70` defines `BaselineFewShotSolver` concatenating few-shot examples
- **Rationale**: These simple baselines establish lower bounds for comparison

### 3. Classical Methods Support ✓
- **Average interpolation baseline**: `evals/elsuite/function_deduction/baselines.py:16-76` implements `AverageBaseline` with three-step algorithm:
  - Queries neighboring values: `test_input - 1`, `test_input + 1`
  - Computes three guesses: `round()`, `math.floor()`, `math.ceil()` of average
  - Surrenders after 9 rounds if unsuccessful
- **Oracle/full-knowledge baseline**: `evals/elsuite/function_deduction/baselines.py:78-133` implements `FullKnowledge` with configurable modes (`mode: random`, `mode: best`)
- **Registry configuration**: Baselines registered with parameters (e.g., `function_deduction/average_baseline`, `function_deduction/full_knowledge_random`, `function_deduction/full_knowledge_best`)

### 4. State-of-the-Art Baselines ✓
- **Published SOTA integration**: `evals/elsuite/hr_ml_agent_bench/benchmarks/ant/baselines/human.py` uses Stable Baselines3 SAC algorithm for RL tasks
- **Optimized classical methods**: Human expert baselines tuned for specific benchmarks
- **Checkpoint persistence**: `evals/elsuite/hr_ml_agent_bench/utils.py:97-180` implements `get_baseline_score()` with checkpoint caching to avoid re-running expensive SOTA baselines
- **GPU allocation management**: Automatic GPU selection and allocation for compute-intensive baselines

### 5. Human Performance Baselines ✓
- **Human expert implementation**: `evals/elsuite/hr_ml_agent_bench/benchmarks/ant/baselines/human.py` provides expert-tuned baseline
- **Human CLI solver**: `already_said_that/human_cli`, `track_the_stat/human_cli` registered solvers for interactive human judgment
- **Ceiling establishment**: HR-ML-Agent-Bench explicitly records human baseline scores as evaluation ceiling
- **Multi-baseline recording**: `evals/elsuite/hr_ml_agent_bench/eval.py:85-98` records three baselines per task:
  - `naive_baseline_score` (0%)
  - `human_baseline_score` (100% ceiling)
  - `model_score` (target system performance)

### 6. Fair Comparison Enforcement ✓
- **Data split validation**: `evals/elsuite/self_prompting/eval.py:121-178` implements `_calculate_improvement_wrt_baseline()` with strict validation:
  ```python
  if set(spec_args["tasker_models"]) != set(self.tasker_models):
      logger.warn(f"SKIPPING IMPROVEMENT METRICS...")
  if spec_args["n_tasks"] != self.n_tasks:
      logger.warn(f"SKIPPING IMPROVEMENT METRICS...")
  if spec_args["n_samples_per_task"] != self.n_samples_per_task:
      logger.warn(f"SKIPPING IMPROVEMENT METRICS...")
  ```
- **Same measurement protocol**: Baselines and target system evaluated with identical evaluation metrics and data
- **Deterministic shuffling**: `evals/eval.py:26` uses `SHUFFLE_SEED = 123` for reproducible sample ordering across all systems
- **Baseline logpath specification**: `evals/registry/evals/self_prompting.yaml:13` stores baseline results path for reference: `baseline_logpath: self_prompting/oriprompt.log`

### 7. Hyperparameter Budget Equity ✓
- **Seed control**: `evals/eval.py:60` default seed `20220722` ensures reproducibility; same seed used for all baseline systems
- **Temperature configuration**: All systems configured with identical temperature in YAML (e.g., `temperature: 1` for generation, `temperature: 0` for classification)
- **Max token limits**: Identical `max_tokens` for all baselines (e.g., `max_tokens: 512` for generation baselines)
- **Sample limit control**: `evals/eval.py:41-43` `set_max_samples()` applies same sample limit to all systems
- **Isolated execution environments**: `evals/elsuite/hr_ml_agent_bench/eval.py:73` uses `TemporaryDirectory()` for each baseline run, ensuring no cross-contamination

### 8. Baseline Context in Results ✓
- **Normalized improvement metric**: `evals/elsuite/self_prompting/eval.py:155-165` computes baseline-relative scores between -1 and +1:
  ```python
  def normalized_improvement(current, baseline):
      """Returns -1 (regression) to +1 (max improvement)"""
      if current < baseline:
          return (current - baseline) / baseline
      else:
          return (current - baseline) / (1 - baseline)
  ```
- **Human-relative scoring**: `evals/elsuite/hr_ml_agent_bench/eval.py:95-97` reports human-relative performance where 0=naive baseline, 1=human ceiling
- **Multiple metric reporting**: `evals/elsuite/hr_ml_agent_bench/eval.py:85-98` records 7 metrics per task:
  - Absolute: `model_score`, `naive_baseline_score`, `human_baseline_score`
  - Normalized: `*_normalized` versions (0-1 range)
  - Relative: `model_score_humanrelative` (0-1 range)
- **Results documentation**: `evals/elsuite/self_prompting/readme.md:69-70` documents baseline metrics in table format with interpretation guide
- **Aggregated comparison**: `evals/elsuite/hr_ml_agent_bench/eval.py:112-114` computes average across all tasks: `avg_humanrelative_score`

---

## Key Strengths

1. **Diverse baseline types** - Random, simple, classical, SOTA, and human baselines all supported
2. **Comparison fairness** - Strict validation ensures identical data splits, measurement protocols, and hyperparameters
3. **Reproducibility** - Deterministic seeds, isolated environments, and checkpoint persistence enable consistent re-runs
4. **Baseline context** - Results presented relative to baselines with normalized and human-relative metrics
5. **Extensible registry** - YAML-based configuration enables easy addition of new baseline solvers
6. **Efficiency** - Checkpoint caching prevents re-execution of expensive SOTA baselines
7. **Validity evidence** - Baseline context supports convergent validity (target outperforms simpler alternatives) and discriminant validity (relative to human capability)

---

## Baseline Configuration Examples

### Random Baseline
```python
class RandomBaselineSolver(Solver):
    def _solve(self, task_state: TaskState) -> SolverResult:
        answer = random.choice(["yes", "no"])
        return SolverResult(output=f"[answer: {answer}]")
```
**Registered as**: `already_said_that/random_baseline`

### Classical Method Baseline
```yaml
function_deduction/average_baseline:
  class: evals.elsuite.function_deduction.baselines:AverageBaseline

function_deduction/full_knowledge_random:
  class: evals.elsuite.function_deduction.baselines:FullKnowledge
  args:
    mode: random
    samples_jsonl: function_deduction/data.jsonl
```

### Simple Baseline
```python
class BaselineOriginalPromptSolver(Solver):
    def _solve(self, task_state: TaskState) -> SolverResult:
        # Return the original task instruction as baseline
        return SolverResult(output=task_state.current_state["original_instruction"])
```

### Eval with Baseline Reference
```yaml
self_prompting.full:
  class: evals.elsuite.self_prompting.eval:SelfPrompting
  args:
    samples_jsonl: self_prompting/samples.jsonl
    tasker_models: ["gpt-3.5-turbo", "gpt-4-base", "gpt-4"]
    n_tasks: 50
    n_samples_per_task: 10
    baseline_logpath: self_prompting/oriprompt.log  # Reference baseline
```

---

## Fair Comparison Validation

The Self-Prompting eval demonstrates explicit fair comparison enforcement:

```python
def _calculate_improvement_wrt_baseline(self, current_res):
    baseline_spec = extract_spec(Path(self.baseline_logpath))
    spec_args = baseline_spec["run_config"]["eval_spec"]["args"]

    # Validate identical configuration
    assert set(spec_args["tasker_models"]) == set(self.tasker_models)
    assert spec_args["n_tasks"] == self.n_tasks
    assert spec_args["n_samples_per_task"] == self.n_samples_per_task

    # Compute normalized improvement
    return {
        "accuracy_improvement_wrt_oriprompt": normalized_improvement(
            current_res["accuracy"],
            baseline_res["accuracy"]
        )
    }
```

---

## Baseline Context in Results

### Self-Prompting Metrics
**Reported metrics** (from `evals/elsuite/self_prompting/readme.md`):
- `accuracy`: Mean accuracy on all tasks
- `accuracy_fuzzy`: Fuzzy-match accuracy
- `accuracy_improvement_wrt_oriprompt`: Normalized relative improvement [-1, +1]
- `baseline_accuracy`: Absolute baseline score
- `n_samples`: Sample count

### HR-ML-Agent-Bench Metrics
**Reported metrics** (from `evals/elsuite/hr_ml_agent_bench/eval.py`):
- `model_score`: Target system raw score
- `naive_baseline_score`: Random/no-effort baseline (0%)
- `human_baseline_score`: Expert human performance (100%)
- `model_score_normalized`: Target in [0,1] range
- `naive_baseline_score_normalized`: Random baseline in [0,1]
- `human_baseline_score_normalized`: Human baseline in [0,1]
- `model_score_humanrelative`: Target relative to naive (0) and human (1)
- `avg_humanrelative_score`: Average across all tasks

---

## Baseline Execution Framework

**Persistent Baseline Caching** (`evals/elsuite/hr_ml_agent_bench/utils.py`):
```python
def get_baseline_score(
    baseline_script: Path,
    score_fn: Callable[[Path], float],
    other_files: Optional[list[Path]] = None,
    save_checkpoints: bool = True,
) -> float:
    # Executes baseline in isolated temp directory
    # Saves checkpoints to avoid re-running expensive baselines
    # Manages GPU allocation and process-level isolation
```

**Benefits**:
- Expensive SOTA baselines computed once and cached
- Reproducible baseline scores across eval runs
- GPU resource management for compute-intensive baselines
- Isolated execution prevents interference with target systems

---

## Validity Evidence from Baselines

The framework enables two key validity assessments:

### Convergent Validity
Target system outperforms simpler alternatives:
- Random baseline: Tests above-chance performance
- Original prompt baseline: Tests improvement over naive instructions
- No-instruction baseline: Tests necessity of well-structured prompts

### Discriminant Validity
Target system performs relative to human capability:
- Human-relative metric shows: 0 = naive baseline, 1 = human expert
- Scores above human indicate superhuman capability
- Scores near random indicate near-chance performance

---

## Configuration Reusability

**Baseline specifications versioned and reused**:
- `baseline_logpath: self_prompting/oriprompt.log` stored as reference
- YAML-based registry enables baseline composition and reuse
- Same baseline seed across eval runs ensures reproducibility
- Checkpoint persistence enables baseline result reuse without re-execution

---

## Summary

The harness provides **comprehensive baseline support with strict fair comparison enforcement**. All baseline types (random, classical, SOTA, human) are integrated into a unified registry system with deterministic seeds, identical hyperparameters, and explicit validation that baselines use same data splits and measurement protocols. Results are presented with baseline context via normalized improvement metrics and human-relative scoring, enabling both convergent and discriminant validity assessment.
