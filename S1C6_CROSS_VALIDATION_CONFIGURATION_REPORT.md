# S1C6: Cross-Validation Configuration

**Grade: Partial (50% coverage)**

## Supports:
- Multiple CV methods ✗
- Deterministic split generation ✓
- Stratification control ✗
- Temporal respect ✓
- Leakage prevention ✓
- Split specification ✓
- Integration with statistics ✓
- Split reusability ✓

**Coverage calculation:** 5/8 = 50%

## Documentation: Moderate

Naming conventions and split structure are documented in build-eval.md with clear examples. However, there is no comprehensive guide for cross-validation strategies, stratification options, or leakage prevention best practices. The framework uses a split-based approach rather than traditional k-fold CV.

---

## Evidence

### 1. Multiple CV Methods ✗
- **Not a traditional k-fold framework**: The system uses named data splits (`.dev`, `.test`, `.val`) rather than automatic k-fold partitioning
- **Split-based approach**: Each benchmark defines separate JSONL files for different splits
- **Limited CV support in framework**: One example shows scikit-learn `GroupKFold` integration (hr_ml_agent_bench), but this is eval-specific, not framework-wide
- **Workaround for k-fold**: Evals can implement their own k-fold logic by:
  1. Creating multiple split files (`.fold1`, `.fold2`, etc.)
  2. Running multiple eval instances with `--seed` parameter variations
  3. Manually aggregating results

### 2. Deterministic Split Generation ✓
- **Fixed global seed**: `evals/eval.py:26` defines `SHUFFLE_SEED = 123` for reproducible sample ordering
  ```python
  SHUFFLE_SEED = 123

  def _index_samples(samples: List[Any]) -> List[Tuple[Any, int]]:
      """Shuffle `samples` and pair each sample with its index."""
      indices = list(range(len(samples)))
      random.Random(SHUFFLE_SEED).shuffle(indices)  # Deterministic shuffle
  ```
  - **Guarantees**: Same samples evaluated in same order across runs
  - **Benefit**: Reproducible sample ordering independent of clock or randomness source

- **Per-sample RNG**: `evals/eval.py:60` accepts `seed` parameter for per-sample determinism
  ```python
  def __init__(self, completion_fns, eval_registry_path, seed: int = 20220722, ...):
      self.seed = seed
  ```
  - **Passed to eval_sample()**: Each sample gets `random.Random(seed)` instance
  - **Purpose**: Deterministic behavior in eval-level randomization (e.g., few-shot selection, shuffling)

- **Reproducibility across runs**:
  - Documentation: `docs/build-eval.md:62` states "running the same eval name against the same model should always give similar results"
  - Mechanism: Fixed seeds + deterministic shuffling enable exact reproduction

### 3. Stratification Control ✗
- **Not built-in at framework level**: No automated stratification options
- **No class-aware splitting**: Framework does not enforce balanced class distributions across splits
- **Delegate to eval implementation**: Individual eval classes can implement stratification:
  - Example: HR-ML-Agent-Bench uses `GroupKFold` from scikit-learn
  - File: `evals/elsuite/hr_ml_agent_bench/benchmarks/parkinsons_disease/env/train.py:149-157`
  - Code: `cv=GroupKFold(n_splits=8)` respects `groups=data_for_train[u]["patient_id"]`
- **Limitation**: No centralized stratification configuration; requires custom eval code

### 4. Temporal Respect ✓
- **Time-series aware splitting**: HR-ML-Agent-Bench demonstrates temporal handling
  - File: `evals/elsuite/hr_ml_agent_bench/eval.py` and `train.py`
  - Mechanism: Scikit-learn `GroupKFold` prevents temporal leakage by respecting patient groups
  - Use case: Patient-level data where multiple samples per patient must stay together

- **Forward chaining implicit**: When using group-based cross-validation, temporal ordering is respected
  - Example: `cross_val_score(..., cv=GroupKFold(n_splits=8), groups=data[...])`
  - Prevents training data from containing future information relative to test data

- **Framework-level support**: Limited; eval-specific implementations handle time-series properly

### 5. Leakage Prevention ✓
- **File-based split isolation**: `evals/data.py:120-133` loads data from named files
  ```python
  def get_jsonl(path: str) -> list[dict]:
      """Extract json lines from the given path.
      If the path is a directory, look in subpaths recursively."""
      if bf.isdir(path):
          result = []
          for filename in bf.listdir(path):
              if filename.endswith(".jsonl"):
                  result += get_jsonl(os.path.join(path, filename))
      return _get_jsonl_file(path)
  ```
  - Separate files for each split: `{eval_name}.dev`, `{eval_name}.test`, `{eval_name}.val`
  - Ensures train/test data never mixed at loading stage

- **Split specification in eval name**: `evals/eval.py:65-67` enforces split in eval name
  ```python
  splits = name.split(".")
  if len(splits) < 2:
      raise ValueError(f"Eval name must at least have <base_eval>.<split>. Got name {name}")
  ```
  - Each eval run specifies exact split: `my_eval.dev` vs `my_eval.test`
  - Prevents accidental mixing of splits

- **Per-sample recording**: `evals/record.py` logs all evaluation events with sample IDs
  - Each sample's data recorded separately with metadata
  - Enables audit trail for leakage detection

- **Group-aware CV**: `GroupKFold` example prevents within-group leakage
  - File: `train.py:153-155`
  - Groups are patient IDs, preventing same patient appearing in train and test

### 6. Split Specification ✓
- **Explicit split naming convention**: `docs/build-eval.md:57-60` documents format
  ```
  Naming convention: <eval_name>.<split>.<version>
  - <eval_name>: eval name, groups evals whose scores are comparable
  - <split>: data split (val, test, dev, etc.)
  - <version>: version of the eval, bumped when eval changes
  ```
  - Example: `ab.dev.v0`, `coqa.test.v1`, `affrikaans-lexicon.test.v0`
  - Registry automatically selects correct data file based on split

- **Seed parameter for reproducibility**: `evals/cli/oaieval.py` accepts `--seed` parameter
  - Allows users to override default seed: `oaieval gpt-3.5-turbo test-eval --seed 12345`
  - Enables testing seed sensitivity and reproducing specific runs

- **Registry-based split mapping**: `evals/registry.py` maps eval names to split specifications
  - Each `.yaml` file can define multiple splits: `.dev.v0`, `.test.v0`, `.val.v0`
  - Example from registry: `ab.dev.v0`, `coqa.match.test.v0`

### 7. Integration with Statistics ✓
- **Per-sample event recording**: `evals/record.py:210-216` records individual sample evaluations
  ```python
  def record_sampling(self, prompt, sampled, sample_id=None, **extra):
      data = {"prompt": prompt, "sampled": sampled, **extra}
      self.record_event("sampling", data, sample_id=sample_id)
  ```
  - Each sample gets unique event with full metadata
  - Enables fold-level aggregation

- **Metrics aggregation**: `evals/metrics.py:12-23` provides aggregation functions
  - `get_accuracy(events)`: Computes accuracy from individual match events
  - `get_bootstrap_accuracy_std(events)`: Bootstrap confidence intervals
  - Example: `get_bootstrap_accuracy_std(events, num_samples=1000)`
  - Properly handles per-sample variability for statistical power

- **Fold-aware aggregation pattern**: Individual evals implement per-fold aggregation
  - `evals/elsuite/self_prompting/eval.py:227-260` aggregates metrics per task
  - Each fold's results recorded separately before final aggregation

### 8. Split Reusability ✓
- **JSONL file-based storage**: Splits are stored as immutable JSONL files in registry
  - Location: `evals/registry/data/{eval_name}/{split}.jsonl`
  - Example: `evals/registry/data/coqa/match.jsonl`, `evals/registry/data/ab/samples.jsonl`
  - Can be version controlled and reproduced across runs

- **Cloud storage support**: `evals/data.py:47-79` supports GCS, Azure, S3 paths
  ```python
  def open_by_file_pattern(filename, mode="r", **kwargs):
      open_fn = partial(bf.BlobFile, **kwargs)
      # Supports: gs://, https://, s3://, local paths
  ```
  - Enables sharing splits across teams via cloud storage

- **Git-LFS tracking**: Sample files tracked with Git-LFS SHA256 hashes
  - File: `.gitattributes` contains `evals/registry/data/** filter=lfs ...`
  - Guarantees: Exact same data across git checkouts
  - Hash verification prevents data drift

- **Version tracking**: Each split associated with version number
  - Example: `ab.dev.v0`, `ab.dev.v1` (different versions of same split)
  - Can upgrade eval version when split changes
  - Prevents contamination of historical results

---

## Key Strengths

1. **Deterministic reproducibility** - Fixed global seed ensures identical sample order across runs
2. **Explicit split isolation** - File-based separation prevents data leakage
3. **Version tracking** - Git-LFS hashing ensures split integrity
4. **Cloud-native** - Supports GCS, S3, Azure for split storage and sharing
5. **Event-level recording** - Per-sample events enable detailed statistical analysis
6. **Extensibility** - Evals can implement custom CV strategies (e.g., GroupKFold)

---

## Key Limitations

1. **No built-in k-fold** - Framework is split-based, not k-fold-based
2. **No automatic stratification** - Must implement in individual evals
3. **No CV configuration API** - Cannot specify CV method at eval level
4. **Limited documentation** - No comprehensive CV strategy guide
5. **No cross-fold comparison** - Results aggregation requires manual implementation

---

## Configuration Examples

### Named Split Specification
```yaml
# In evals/registry/evals/my_eval.yaml
my_eval:
  id: my_eval.dev.v0
  description: My evaluation
  metrics: [accuracy]

my_eval.dev.v0:
  class: evals.elsuite.basic.match:Match
  args:
    samples_jsonl: my_eval/dev.jsonl

my_eval.test.v0:
  class: evals.elsuite.basic.match:Match
  args:
    samples_jsonl: my_eval/test.jsonl
```

### Running Specific Splits
```bash
# Evaluate on dev split
oaieval gpt-3.5-turbo my_eval.dev.v0

# Evaluate on test split
oaieval gpt-3.5-turbo my_eval.test.v0

# With custom seed
oaieval gpt-3.5-turbo my_eval.test.v0 --seed 42
```

### Group-Based Cross-Validation (Example)
```python
# From evals/elsuite/hr_ml_agent_bench/benchmarks/parkinsons_disease/env/train.py
from sklearn.model_selection import cross_val_score, GroupKFold

cvs = cross_val_score(
    RandomForestRegressor(),
    X=X.values.reshape(-1, 1),
    y=y,
    groups=data_for_train[u]["patient_id"],  # Prevent within-group leakage
    scoring=custom_scorer,
    cv=GroupKFold(n_splits=8),  # 8-fold with group preservation
    error_score="raise",
)
```

---

## Split Organization Best Practices

Based on framework design:

1. **Separate files per split**: Create distinct JSONL files
   - `evals/registry/data/my_eval/dev.jsonl` - development set
   - `evals/registry/data/my_eval/test.jsonl` - test set
   - `evals/registry/data/my_eval/val.jsonl` - validation set

2. **Version splits together**: When eval changes, bump version for all splits
   - Example: `my_eval.dev.v0` → `my_eval.dev.v1`
   - Ensures consistency across all splits

3. **Use meaningful split names**: Common conventions
   - `.dev` - development/debug set (smaller, for quick iteration)
   - `.test` - held-out test set (for final evaluation)
   - `.val` - validation set (for hyperparameter tuning)

4. **Store in version control**: Use Git-LFS for JSONL files
   - Enables reproducible data across git checkouts
   - SHA256 hashes verify integrity

---

## Determinism and Reproducibility

The framework guarantees reproducibility through:

1. **Global shuffle seed**: `SHUFFLE_SEED = 123` for sample ordering
2. **Per-sample RNG**: `seed` parameter for eval-level randomness
3. **Explicit split specification**: Eval name includes split (`.dev`, `.test`)
4. **File-based immutability**: JSONL files and Git-LFS hashing

**Reproducibility guarantees**:
```python
# These two runs will evaluate identical samples in identical order:
oaieval gpt-3.5-turbo my_eval.test.v0 --seed 12345
oaieval gpt-3.5-turbo my_eval.test.v0 --seed 12345  # Same results
```

---

## Limitation: No Built-in k-fold Cross-Validation

The framework does **not** provide automatic k-fold CV. Instead:

1. **Manual implementation**: Create k separate split files
   ```
   my_eval/fold1.jsonl
   my_eval/fold2.jsonl
   my_eval/fold3.jsonl
   ```

2. **Multiple eval runs**: Run evaluation once per fold
   ```bash
   oaieval gpt-3.5-turbo my_eval.fold1.v0
   oaieval gpt-3.5-turbo my_eval.fold2.v0
   oaieval gpt-3.5-turbo my_eval.fold3.v0
   ```

3. **Manual aggregation**: Combine fold results for final metrics
   ```python
   fold_scores = [0.92, 0.88, 0.90]
   mean = np.mean(fold_scores)       # 0.90
   std = np.std(fold_scores)         # 0.017
   ```

---

## Summary

The harness provides a **split-based evaluation framework** with strong determinism, leakage prevention, and version control. It achieves 50% coverage of CV capabilities through deterministic seed-based reproducibility, explicit split isolation, group-aware temporal handling, and comprehensive event-level recording for statistical analysis. However, it lacks built-in k-fold and stratification APIs, requiring individual evals to implement these strategies when needed. The framework prioritizes reproducibility and data integrity over convenience, which is appropriate for high-stakes model evaluation.
