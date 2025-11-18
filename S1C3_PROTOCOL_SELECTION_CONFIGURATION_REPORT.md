# S1C3: Protocol Selection & Configuration

**Grade: Present (87.5% coverage)**

## Supports:
- Human judgment support ✗
- LLM-as-judge support ✓
- Algorithmic metrics support ✓
- Ensemble combinations ✓
- Custom metric support ✓
- Standardized measurement schema ✓
- Aggregation of subjective judgments ✓
- Measurement validation ✓

**Coverage calculation:** 7/8 = 87.5%

## Documentation: Moderate

Documentation exists for core measurement concepts through code examples, YAML configuration files, and inline documentation. However, there is limited formal documentation for creating custom metrics or understanding the full measurement validation workflow. Most guidance must be inferred from example implementations.

---

## Evidence

### 1. Human Judgment Support ✗
- **Crowdsourcing integration**: Not implemented in core framework
- **Rationale**: Framework focuses on automated measurement (LLM-as-judge and algorithmic metrics); human judgment requires external platform integration not present in codebase
- **Potential workaround**: Custom crowdsourcing implementations could extend framework via custom metric classes, but no turnkey solution exists

### 2. LLM-as-Judge Support ✓
- **Main implementation**: `evals/elsuite/modelgraded/classify.py:14-127` defines `ModelBasedClassify` class
- **Judge model selection**: `evals/elsuite/modelgraded/classify.py:86` uses `eval_completion_fn` (separate from policy completion function)
- **Rubric design**: YAML-based specifications in `evals/registry/modelgraded/` with prompt templates
  - **Example rubric** (`fact.yaml`): Detailed comparison prompt with 5-choice options (A-E) for factuality assessment
  - **Example rubric** (`best.yaml`): Comparative evaluation prompt for N responses with `from_n` dynamic choice generation
- **Chain-of-thought reasoning**: `evals/elsuite/modelgraded/classify_utils.py:13-27` provides 4 prompt templates:
  - `classify`: Direct choice selection
  - `classify_cot`: Answer first, then reasoning
  - `cot_classify`: Step-by-step reasoning then answer
  - `cot_classify_jp`: Japanese-language chain-of-thought
- **Confidence calibration**: Score mapping via `choice_scores` parameter (e.g., `from_strings` for numeric choices)
- **Multi-completion support**: `evals/elsuite/modelgraded/classify.py:67-74` supports `multicomp_n` parameter to sample N completions and concatenate for comparison

### 3. Algorithmic Metrics Support ✓
- **Exact Match**: `evals/elsuite/basic/match.py:30-56` implements `Match` class with `startswith()` checking
- **Substring/Includes**: `evals/elsuite/basic/includes.py` checks `expected in sampled`
- **Fuzzy Match**: `evals/elsuite/basic/fuzzy_match.py:23-51` with normalized bidirectional substring and F1 score (`evals/elsuite/utils.py` contains `fuzzy_match()` and `f1_score()`)
- **JSON Structure Match**: `evals/elsuite/basic/json_match.py:12-37` recursive component-wise comparison
- **JSON Schema Validation**: `evals/elsuite/basic/json_validator.py` validates JSON structure against schema
- **Parameter configuration**: Each metric accepts `max_tokens`, task-specific parameters (e.g., `num_few_shot` for Match)

### 4. Ensemble Combinations ✓
- **Multiple completions from single model**: `evals/elsuite/modelgraded/classify.py:67-74` with `sample_and_concat_n_completions()` concatenates N generations
- **Multiple models, single task**: `evals/elsuite/modelgraded/classify.py:31-49` supports multiple `completion_fns` in list with `multicomp_n="from_models"`
- **Voting scheme**: Implicit in judge model's `classify()` function which selects single best choice from ensemble
- **Output template control**: `evals/elsuite/modelgraded/classify.py:71` uses `output_template` for formatting multiple responses (e.g., `best.yaml:12` shows `"{i}. {output}\n"`)

### 5. Custom Metric Support ✓
- **Extensible interface**: Subclass `evals.Eval` (base class at `evals/eval.py:46-166`)
- **Implement two methods**:
  - `eval_sample(sample: dict, rng: Random)`: Per-sample evaluation logic
  - `run(recorder: RecorderBase)`: Aggregation and return final metrics
- **Recording API**: `evals.record.record_metrics(**kwargs)` (line 184+ in `record.py`) allows arbitrary metric recording
- **Example**: `FuzzyMatch` class records both `accuracy` and `f1_score` in single `eval_sample()` call
- **Aggregation pattern**: Use `recorder.get_scores(key)` to retrieve per-sample scores for aggregation (e.g., `evals/elsuite/basic/fuzzy_match.py:58`)

### 6. Standardized Measurement Schema ✓
- **Event structure**: `evals/record.py:43-51` defines `Event` dataclass:
  ```python
  Event(run_id, event_id, sample_id, type, data, created_by, created_at)
  ```
- **Measurement output mapping**:
  - Score: `data["score"]` or `data["accuracy"]`, `data["f1_score"]`, etc.
  - Rationale: `data["sampled"]` for judge output, `data["prompt"]` for evaluation prompt
  - Metadata: `data["choice"]`, `data["invalid_choice"]`, `data["metascore"]`
  - Uncertainty: Not explicitly stored per-sample; aggregated via bootstrap (see #7)
- **Flexible schema**: `record_metrics(**kwargs)` accepts any metric names without predefined schema
- **Standardized recording methods**:
  - `record_match()`: For binary match/mismatch events
  - `record_metrics()`: For arbitrary metric values
  - `record_sampling()`: For model sampling traces

### 7. Aggregation of Subjective Judgments ✓
- **Preserves per-sample variability**: Events recorded before aggregation (line 55-72 in `record.py`)
- **Individual metric extraction**: `recorder.get_scores(key)` retrieves per-sample values (e.g., `evals/elsuite/basic/match.py:61`)
- **Bootstrap uncertainty**: `evals/metrics.py:21-23` implements `get_bootstrap_accuracy_std()` with 1000 resamples for confidence intervals
- **Multi-rater patterns supported**:
  - Same judge model evaluated multiple times (different seeds/temperatures) via ensemble
  - Multiple judge models via separate eval runs and comparison
- **No forced aggregation**: Metrics returned as aggregated results, but individual events available for further analysis

### 8. Measurement Validation ✓
- **Meta-evaluation pattern**: `evals/elsuite/modelgraded/classify.py:96-98` implements `metaeval=True` mode
  - **Validation method**: Create labeled dataset where judge's expected choice is provided
  - **Comparison**: Judge's predicted `choice` compared against reference `choice` field
  - **Output**: `metascore` metric where 1.0 indicates perfect judge agreement
- **Quality assurance**: Run meta-eval on small golden dataset before full-scale evaluation
- **Validator implementations**: `evals/elsuite/basic/json_validator.py` validates JSON against schema before matching
- **Example workflow**:
  1. Create validation set with known judge outputs
  2. Set `metaeval: true` in eval config
  3. Inspect `metascore` in results (expect ~1.0 for valid judge)

---

## Key Strengths

1. **Diverse algorithmic metrics** - 5+ built-in scoring functions covering exact/fuzzy/JSON matching, extensible to custom
2. **Production-grade LLM-as-judge** - Chain-of-thought prompting, multi-language support, score mapping, ensemble support
3. **Measurement validation** - Meta-evaluation framework validates judge quality before full runs
4. **Standardized recording schema** - Event-based system preserves per-sample data for statistical analysis
5. **Extensibility** - Custom metrics via simple `Eval` subclass + `record_metrics()` calls
6. **Bootstrap uncertainty** - Confidence intervals computed from per-sample results
7. **No-code configuration** - YAML-based rubric specification enables rapid judge iteration

---

## Measurement Modality Examples

### Exact Match (Algorithmic)
```python
class Match(evals.Eval):
    def eval_sample(self, sample, rng):
        result = self.completion_fn(prompt=sample["input"], temperature=0.0)
        sampled = result.get_completions()[0]
        evals.record_and_check_match(
            sampled=sampled,
            expected=sample["ideal"]
        )
```
**Returns**: Binary match event with accuracy metric

### Fuzzy Match (Algorithmic)
```python
class FuzzyMatch(evals.Eval):
    def eval_sample(self, test_sample, rng):
        sampled = self.completion_fn(prompt=test_sample["input"]).get_completions()[0]
        matches = [utils.fuzzy_match(sampled, ans) for ans in test_sample["ideal"]]
        evals.record.record_metrics(
            accuracy=float(True in matches),
            f1_score=utils.f1_score(sampled, test_sample["ideal"])
        )
```
**Returns**: Accuracy and F1 score with fuzzy matching

### LLM-as-Judge (Model-Graded)
```yaml
# fact.yaml - Judge rubric
fact:
  prompt: |
    Compare the factual content of submitted vs expert answer.
    (A) subset and consistent
    (B) superset and consistent
    (C) same details
    (D) disagreement
    (E) differences don't matter
  choice_strings: ABCDE
  input_outputs:
    input: completion
```
**Configuration**: YAML-based judge prompt with 5-point scale

### Meta-Evaluation (Quality Validation)
```python
class ModelBasedClassify(evals.Eval):
    def eval_sample(self, test_sample, rng):
        # ... run judge evaluation ...
        if self.metaeval:
            metrics["metascore"] = choice == test_sample["choice"]
        evals.record.record_metrics(**metrics)
```
**Validation**: Compares judge output against ground truth; metascore should be ~1.0

---

## Limitations & Notes

### Gap: Human Judgment Support
- **Not implemented**: No crowdsourcing integration (Mechanical Turk, Prolific, Scale AI, etc.)
- **Recommendation**: Custom `HumanJudgmentEval` class could wrap external platform APIs
- **Workaround**: Manual result collection + offline integration using custom CSV loader

### Output Schema Observations
- **Per-sample uncertainty**: Not explicitly recorded (only aggregated via bootstrap)
- **Structured rationale**: Judge reasoning preserved in `sampled` field but not separately extracted
- **Confidence scores**: Implicit in choice_scores mapping; no explicit confidence calibration

### Validation Scope
- **Meta-evaluation**: Only tests judge consistency on labeled data
- **Measurement drift**: No continuous monitoring for judge behavior change across evaluation run
- **Judge calibration**: No automatic bias mitigation (blinding, randomization) implemented

---

## Configuration Files

**ModelGraded Specifications** (18 rubrics in `evals/registry/modelgraded/`):
- `fact.yaml` - Factual comparison (5-point scale)
- `best.yaml` - Best response selection
- `closedqa.yaml` - Closed-ended Q&A evaluation
- `diversity.yaml` - Response diversity scoring
- `humor.yaml` - Humor evaluation
- `keywords.yaml` - Keyword presence checking
- `translation.yaml` - Translation quality
- `sql.yaml` - SQL correctness evaluation
- And 10+ others

**Metric Classes** (`evals/elsuite/basic/`):
- `match.py` - Exact string matching
- `includes.py` - Substring matching
- `fuzzy_match.py` - Fuzzy string matching with F1 score
- `json_match.py` - JSON structure matching
- `json_validator.py` - JSON schema validation

---

## Recording and Aggregation

**Event Flow**:
1. `eval_sample()` called per sample
2. `record_metrics(**kwargs)` stores event with metric values
3. `run()` method retrieves events via `recorder.get_scores(key)`
4. Final aggregation (mean, std, bootstrap) computed from per-sample values

**Aggregate Metrics Computed**:
- Mean accuracy: `sum(correct) / len(events)`
- Bootstrap std: Confidence interval from 1000 resampled means
- Custom aggregations: User-defined via numpy operations on score arrays

---

## Summary

The harness provides **production-grade measurement infrastructure** with strong support for LLM-as-judge evaluation, diverse algorithmic metrics, ensemble combinations, and measurement validation. The primary limitation is the absence of human judgment integration (requiring external platform coupling). The standardized event schema and per-sample recording enable rigorous statistical analysis and validation of measurement quality.
