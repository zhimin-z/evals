# S1C1: Benchmark Loading & Validation

**Grade: Present (100% coverage)**

## Supports:
- Multiple format support ✓
- Completeness validation ✓
- Construct preservation ✓
- Validity evidence retention ✓
- Multi-benchmark support ✓
- Integrity verification ✓
- Clear validation errors ✓
- Version consistency ✓

**Coverage calculation:** 8/8 = 100%

## Documentation: Comprehensive

The harness provides thorough documentation across multiple resources including a detailed README with setup/usage instructions, 6 dedicated documentation files covering eval creation workflows, and extensive inline code documentation.

---

## Evidence

### 1. Multiple Format Support ✓
- **JSONL** (primary): `evals/data.py:120-133` with `get_jsonl()` and `iter_jsonls()` for streaming and recursive directory handling
- **CSV**: `evals/data.py:168` via `get_csv()`
- **Cloud Storage**: `evals/data.py:47-79` supports GCS and S3 via `blobfile` library
- **Compression**: Auto-detects and handles GZIP (`.gz`), LZ4 (`.lz4`), and Zstandard (`.zst`) formats

### 2. Completeness Validation ✓
- **Eval name validation**: `evals/eval.py:66-67` enforces `<base_eval>.<split>` format with explicit error: `"Eval name must at least have <base_eval>.<split>. Got name {name}"`
- **Sample field validation**: Requires `input` and `ideal` fields in sample dictionaries
- **JSON parsing with detailed errors**: `evals/data.py:82-90` provides line and column information for malformed JSON: `"Error parsing JSON on line {line_number}: {e.msg} at {path}:{line_number}:{e.colno}"`

### 3. Construct Preservation ✓
- **EvalSpec structure**: `evals/base.py:51-61` defines dataclass with `cls`, `registry_path`, `args`, `key`, `group`
- **YAML registry storage**: 470+ benchmark definitions in `evals/registry/evals/*.yaml`
- **Example**: `evals/registry/evals/ab.yaml` shows construct with id, description, metrics, class specification, and arguments

### 4. Validity Evidence Retention ✓
- **Sample files**: Stored in `evals/registry/data/{eval_name}/samples.jsonl` with validity indicators (`ideal`, `criteria`, `choice` fields depending on eval type)
- **Git LFS integrity tracking**: Sample files tracked with SHA256 hashes (e.g., `oid sha256:e81f4c32f139ece2734460931fdc3269883baafd2f666de530e5ff87bcb11d94`)
- **Accessible metadata**: Benchmark specs preserve reference data throughout evaluation lifecycle

### 5. Multi-Benchmark Support ✓
- **EvalSetSpec**: `evals/base.py:64-72` enables grouping multiple evals with shared metadata (`key`, `group`)
- **oaievalset CLI**: `evals/cli/oaievalset.py` supports batch evaluation with progress tracking and resume capability
- **Eval sets registry**: 20+ YAML configurations in `evals/registry/eval_sets/*.yaml` define benchmark collections

### 6. Integrity Verification ✓
- **Git LFS integration**: Binary data files tracked with SHA256 hashes in `.gitattributes`
- **Version hashing**: Versioning convention `{base_eval}.{split}.{version}` (e.g., `ab.dev.v0`, `afrikaans-lexicon.dev.v0`)
- **Data corruption detection**: Git LFS pointer files contain size metadata for drift detection; multiple versions coexist for reproducibility

### 7. Clear Validation Errors ✓
- **Formatted error messages**: `evals/data.py:82-90` with syntax highlighting information
- **Descriptive validation failures**: Format constraints explicitly stated in error messages
- **Eval name validation**: Points user to required format with actual vs. expected
- **File access errors**: Wrapped with context: `"Failed to open: {filename}"`

### 8. Version Consistency ✓
- **Semantic versioning**: Format `{base_eval}.{split}.{version}` maintained across all registry entries
- **Multiple coexisting versions**: System supports `v0`, `v1`, etc. variants simultaneously
- **Metadata preservation**: Version information stored in YAML spec (`id: ab.dev.v0`) and propagated to results

---

## Key Strengths

1. **Robust data format ecosystem** - Supports local files, cloud storage, and multiple compression schemes transparently
2. **Production-grade error handling** - Detailed, actionable error messages with file location and syntax details
3. **Integrity-first design** - Git LFS integration ensures data authenticity and enables reproducible runs
4. **Schema-based validation** - Pydantic dataclasses (`EvalSpec`, `EvalSetSpec`, `BaseEvalSpec`) provide runtime type checking
5. **Scalable architecture** - Registry-based loading enables managing 470+ benchmarks efficiently
6. **Documentation coverage** - README, 6 specialized guides, and inline docstrings support multiple user workflows

---

## Supporting Resources

- **README**: Comprehensive setup and usage guide with links to 6 detailed documentation files
- **Code documentation**: `evals/base.py` defines component specifications; `evals/data.py` documents loading mechanisms
- **Example evals**: 470+ YAML specifications in registry demonstrate specification patterns
- **Templates**: `docs/eval-templates.md` provides benchmark creation patterns
- **Build guides**: `docs/build-eval.md` and `docs/custom-eval.md` walk through complete workflows
