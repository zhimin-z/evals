# S1C7: Analysis Plan Specification

**Grade: Partial (37.5% coverage)**

## Supports:
- Plan pre-specification requirement ✓
- Sample size justification ✗
- Primary/secondary metric specification ✓
- Significance thresholds ✗
- Multiple comparison corrections ✗
- Uncertainty quantification method ✓
- Stratification variables ✗
- Plan adherence enforcement ✗

**Coverage calculation:** 3/8 = 37.5%

## Documentation: Minimal

Analysis plan specification is implicit in YAML eval definitions and code comments, but there is no comprehensive guide for pre-specification, power analysis, or statistical planning. Most statistical capabilities must be inferred from code examples rather than documented best practices.

---

## Evidence

### 1. Plan Pre-specification Requirement ✓
- **YAML-based configuration**: Evals are pre-specified before execution in `evals/registry/evals/*.yaml` files
  ```yaml
  # Example: ab.yaml
  ab:
    id: ab.dev.v0
    description: "Evaluation description..."
    metrics: [accuracy]

  ab.dev.v0:
    class: evals.elsuite.modelgraded.classify:ModelBasedClassify
    args:
      samples_jsonl: ab/samples.jsonl
      eval_type: cot_classify
      modelgraded_spec: fact
  ```
  - All eval parameters specified before any evaluation runs
  - Metrics list specified in advance (`metrics: [accuracy]`)

- **Metric pre-specification**: `evals/record.py:291-294` reads metrics from pre-defined spec
  ```python
  base_eval_spec = registry.get_base_eval(self.run_spec.base_eval)
  if base_eval_spec and len(base_eval_spec.metrics) >= 1:
      primary_metric = base_eval_spec.metrics[0]
  else:
      primary_metric = "accuracy"
  ```
  - Enforces that metrics must be defined before evaluation
  - Default to accuracy if not specified

- **Documentation**: `docs/build-eval.md:57-62` documents the eval naming and specification process
  - Encourages versioning when eval changes: "when you change your eval, you should bump the version"
  - Pre-specification of data splits and eval parameters before running

### 2. Sample Size Justification ✗
- **Not implemented**: No power analysis or sample size calculation tools
- **No effect size specification**: No mechanism to specify target effect size $\delta$
- **No power specification**: No way to specify desired power $1-\beta$ or significance $\alpha$
- **Manual only**: Users must manually justify sample sizes (if at all)
- **Implicit constraints**: Sample sizes determined by dataset size and `--max_samples` CLI flag, not statistical power analysis

### 3. Primary/Secondary Metric Specification ✓
- **First metric is primary**: `evals/record.py:292` designates first metric in list as primary
  ```python
  primary_metric = base_eval_spec.metrics[0]
  ```
  - Convention: Metrics list order indicates importance
  - First metric used for result highlighting/coloring

- **Example specification**: `evals/registry/evals/function-deduction.yaml:1-3`
  ```yaml
  metrics: [adjusted_avg_rounds,          # PRIMARY (listed first)
           solved_ratio, solved, samples,  # SECONDARY
           avg_success_rounds, sem_*, ...]  # EXPLORATORY
  ```
  - 18 metrics listed; only first is designated primary
  - Others are secondary/exploratory measures

- **Multiple metric support**: Any number of metrics can be specified in list
  - No explicit "primary" vs "secondary" tags (implicit via ordering)
  - No designation of "exploratory" metrics

### 4. Significance Thresholds ✗
- **Not implemented**: No configurable alpha level specification
- **No threshold enforcement**: System does not enforce significance thresholds during execution
- **P-values reported as-is**: When p-values are computed, they are reported without interpretation against predetermined alpha
  - Example: `evals/elsuite/function_deduction/eval.py:246` computes Mann-Whitney U p-value
  ```python
  _, p_value = scipy.stats.mannwhitneyu(solved, not_solved, alternative="less")
  result["solved_or_not_mann_whitney_u_p_value"] = p_value
  ```
  - P-value recorded in result but never compared to significance threshold
  - No documentation of pre-determined alpha level

### 5. Multiple Comparison Corrections ✗
- **Not implemented**: No Bonferroni, FDR, Holm-Bonferroni, or other corrections
- **Multiple metrics without adjustment**: Function deduction eval reports 18 metrics without multiple comparison adjustment
  - File: `evals/registry/evals/function-deduction.yaml:1-3`
  - Includes `solved_or_not_mann_whitney_u_p_value` among many other metrics
  - No adjustment for family-wise error rate

- **P-hacking risk**: When multiple metrics are computed, conducting tests on each without correction increases false positive rate
  - Probability of spurious significance: $1-(1-\alpha)^m$ for $m$ tests
  - Example: 18 metrics × 0.05 alpha → ~60% chance of at least one false positive

- **No framework support**: Evals can implement corrections manually in eval code, but no built-in utilities

### 6. Uncertainty Quantification Method ✓
- **Bootstrap resampling**: `evals/metrics.py:21-23` implements bootstrap accuracy std
  ```python
  def get_bootstrap_accuracy_std(events, num_samples=1000):
      vals = [m.data["correct"] for m in events]
      return np.std([np.mean(random.sample(vals, len(vals)//2))
                     for _ in range(num_samples)])
  ```
  - Resamples 50% of samples (half-sampling)
  - Default 1000 bootstrap samples for variance estimation
  - Returns standard deviation of bootstrap means

- **Standard error computation**: `evals/elsuite/twenty_questions/scripts/make_plots.py:72-77`
  ```python
  def compute_sem(x):
      sem = x.std() / (len(x) ** 0.5)
      sem2 = sem * 2  # 95% confidence interval (±1.96 ≈ 2 SEM)
      lower = max(0, (x.mean() - sem2).round(2))
      upper = (x.mean() + sem2).round(2)
      return lower, upper
  ```
  - Parametric 95% CI via ±2 standard errors
  - Assumes normally distributed means (CLT justified for large n)

- **Confusion matrix metrics**: `evals/metrics.py:26-73` supports precision, recall, F-score computation
  - Enables detailed classification metrics beyond accuracy
  - No confidence intervals for these metrics

- **No BCa (bias-corrected accelerated) intervals**: Bootstrap only provides percentile-based CIs
- **No explicit CI method documentation**: No specification of CI coverage or methodology

### 7. Stratification Variables ✗
- **Not implemented**: No framework support for subgroup analysis
- **Per-eval implementation only**: Individual evals can stratify by task/difficulty
  - Example: `evals/elsuite/function_deduction/eval.py:257-263` computes per-complexity metrics
  ```python
  def _get_per_complexity_metrics(self, all_metrics):
      result = {}
      for complexity in sorted_complexities:
          metrics = [x for x in all_metrics if x["complexity"] == complexity]
          result[f"complexity_{complexity}"] = self._get_success_metrics(metrics)
      return result
  ```
  - Manual stratification by complexity level
  - No framework-level stratification API

- **No demographic variables**: No support for demographic stratification (if applicable)
- **No difficulty weighting**: No weighted metrics by difficulty level
- **No interaction terms**: No framework support for interaction analysis

### 8. Plan Adherence Enforcement ✗
- **No locking mechanism**: Plans (YAML files) can be modified before/during evaluation
- **No validation of deviations**: System does not check if execution follows pre-specified plan
- **No plan hash/signature**: No way to verify that evaluation followed a specific locked plan
- **Version tracking via naming**: Only mechanism is `<eval>.<split>.v0` → `v1` versioning
  - If eval changes, user should bump version
  - But nothing prevents running without version bump

- **Manual enforcement only**: Relies on developer discipline to follow pre-specified plans
  - Documentation in `docs/build-eval.md:62` mentions bumping version when eval changes
  - Not enforced by system

---

## Key Strengths

1. **Declarative metric specification** - Metrics pre-specified in YAML before execution
2. **Bootstrap uncertainty** - Proper resampling-based confidence intervals
3. **Parametric CIs available** - SEM-based intervals for simple cases
4. **Per-sample event recording** - All data available for custom post-hoc analysis
5. **Hypothesis testing support** - Statistical tests (Mann-Whitney U) implemented in evals
6. **Primary metric convention** - First metric treated as primary (implicit)

---

## Key Limitations

1. **No power analysis** - Cannot justify sample sizes based on effect size and power
2. **No multiple comparison corrections** - 18 metrics reported without adjustment
3. **No significance thresholds** - Alpha levels not configurable or enforced
4. **No stratification API** - Subgroup analysis requires custom eval code
5. **No plan enforcement** - No mechanism to lock or verify adherence to pre-specification
6. **Minimal documentation** - No guide for statistical pre-specification best practices

---

## Configuration Examples

### Metric Pre-specification
```yaml
# In evals/registry/evals/my_eval.yaml
my_eval:
  id: my_eval.dev.v0
  description: "My evaluation description"
  metrics: [accuracy,           # PRIMARY (first in list)
           f1_score,            # SECONDARY
           precision, recall,   # SECONDARY
           conf_matrix]         # EXPLORATORY

my_eval.dev.v0:
  class: evals.elsuite.basic.match:Match
  args:
    samples_jsonl: my_eval/samples.jsonl
```

### Stratified Analysis (Manual Implementation)
```python
# In custom eval class
def _get_per_difficulty_metrics(self, all_metrics):
    """Compute metrics stratified by difficulty level"""
    difficulties = ["easy", "medium", "hard"]
    result = {}

    for difficulty in difficulties:
        difficulty_metrics = [
            m for m in all_metrics
            if m["difficulty"] == difficulty
        ]
        result[f"{difficulty}_accuracy"] = (
            sum(m["correct"] for m in difficulty_metrics) /
            len(difficulty_metrics)
        )

    return result
```

### Bootstrap Confidence Intervals
```python
# From evals/metrics.py
def get_bootstrap_accuracy_std(events, num_samples=1000):
    vals = [m.data["correct"] for m in events]
    bootstrap_means = [
        np.mean(random.sample(vals, len(vals)//2))
        for _ in range(num_samples)
    ]
    mean = np.mean(vals) / len(vals)
    ci_lower = np.percentile(bootstrap_means, 2.5)
    ci_upper = np.percentile(bootstrap_means, 97.5)
    return mean, ci_lower, ci_upper
```

### Hypothesis Testing (Manual)
```python
# From evals/elsuite/function_deduction/eval.py:246
import scipy.stats

# Test if solved problems have lower complexity
if solved and not_solved:
    _, p_value = scipy.stats.mannwhitneyu(
        solved, not_solved, alternative="less"
    )
    # NOTE: No multiple comparison correction applied!
    result["solved_vs_notsolved_p_value"] = p_value
```

---

## Statistical Analysis Best Practices (Framework Support)

**Supported**:
- ✅ Pre-specification of metrics via YAML
- ✅ Bootstrap confidence intervals
- ✅ Standard error and parametric CIs
- ✅ Multiple statistical tests (Mann-Whitney U, etc.)
- ✅ Per-sample event recording for custom analysis

**Not Supported**:
- ❌ Power analysis and sample size justification
- ❌ Multiple comparison correction
- ❌ Significance thresholds
- ❌ Subgroup stratification API
- ❌ Pre-specification locking/enforcement
- ❌ Deviation documentation

---

## Comparison with Statistical Standards

The framework is optimized for **exploratory benchmark evaluation** rather than **confirmatory statistical analysis**:

| Aspect | Standard | Framework | Gap |
|--------|----------|-----------|-----|
| Pre-specification | Required (OSF) | YAML-based | ✓ Partial |
| Power analysis | Required | Not supported | ✗ Critical |
| Multiple comparisons | Required (Bonferroni/FDR) | Not supported | ✗ Critical |
| Significance thresholds | Pre-specified | Not enforced | ✗ Major |
| Uncertainty quantification | Documented | Bootstrap + SEM | ✓ Good |
| Stratification | Pre-specified | Manual only | ✗ Major |
| Plan enforcement | Locked before execution | No enforcement | ✗ Critical |

---

## Recommendations for Enhancement

1. **Power analysis**: Implement `calculate_sample_size(effect_size, alpha, power)` utility
2. **Plan locking**: Add `plan_hash` field to run specification, computed before execution
3. **Multiple comparison correction**: Implement Bonferroni, Holm, and Benjamini-Hochberg
4. **Significance thresholds**: Add `alpha`, `correction_method` to eval specification
5. **Stratification API**: Support `stratify_by` parameter in eval definition
6. **BCa bootstrap CIs**: Use `scipy.stats.bootstrap` for bias-corrected intervals
7. **Pre-specification validation**: Check metrics against recorded events; flag unexpected metrics
8. **Statistical test suite**: Document and validate all hypothesis tests used in evals

---

## Summary

The harness provides **metric pre-specification and uncertainty quantification** but lacks comprehensive analysis plan infrastructure for confirmatory statistical evaluation. Metrics can be declared in YAML before execution, and bootstrap/parametric confidence intervals are available, but there is no support for power analysis, multiple comparison correction, significance thresholds, stratification APIs, or plan enforcement. The framework is appropriate for exploratory LLM benchmark evaluation but would require substantial additions to support rigorous statistical pre-registration workflows (e.g., OSF-style pre-registration).
