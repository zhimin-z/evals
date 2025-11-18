# S2C1: Data Loading & Preprocessing

**Grade: Present (87.5% coverage)**

## Supports:
- ✓ Multiple format support
- ✗ Multi-modal data support (limited to images only)
- ✓ Streaming support
- ✓ Preprocessing pipeline
- ✓ Schema standardization
- ✓ Split reproducibility
- ✓ Data validation
- ✓ Preprocessing configuration

---

## Detailed Assessment

### 1. Multiple Format Support ✓
**Coverage**: 5+ formats supported

The framework supports diverse data formats through the central data loading module (`evals/data.py:47-231`):

- **JSONL** (line-delimited JSON):
  - `get_jsonl(path)` - bulk loading
  - `iter_jsonls(paths)` - streaming with memory efficiency
  - `_stream_jsonl_file(path)` - cloud streaming
  - Used as primary format across 473+ registry datasets

- **JSON**: `get_json(path)` with file validation

- **CSV**: `get_csv(path)` with DictReader support for flexible schema

- **Compressed formats**: Automatic decompression for `.gz`, `.lz4`, `.zst` extensions

- **REST APIs**: `requests` library support for REST endpoints
  - Example: `/evals/elsuite/multistep_web_tasks/webarena/` REST API integration

- **Cloud Storage**: Google Cloud Storage (GCS) via `blobfile` with transparent I/O
  - Paths like `gs://bucket/path` handled seamlessly
  - Streaming support: `bf.BlobFile(path, "r", streaming=True)`

- **SQL Database**: Snowflake connector with full retry logic
  - File: `/evals/utils/snowflake.py:1-128`
  - Features: connection pooling, session keep-alive, pandas integration

- **HuggingFace Datasets**: Used in multiple evals (MMMU, text_compression, LAMBADA)

**Evidence**: `/evals/data.py`, `/evals/utils/snowflake.py`, registry data directory structure

---

### 2. Multi-Modal Data Support ✗ (Limited)
**Coverage**: Images only; audio, video, and mixed modalities absent

**Supported**:
- **Images (JPEG, PNG, TIFF)**:
  - PIL/Pillow integration in MMMU eval
  - File: `/evals/elsuite/mmmu/eval.py:21-36`
  - Base64 encoding for API transmission: `data:image/png;base64,...`
  - Up to 7 images per sample in multi-modal understanding evaluation

**Not Supported**:
- Audio (WAV, MP3, FLAC) - No librosa, scipy.io, or audio file handling
- Video files - No ffmpeg, OpenCV, or video processing
- Mixed modalities beyond text + images
- Structured data specifically (XML, Protobuf, etc.)

**Evidence**: `/evals/elsuite/mmmu/eval.py`, `/evals/elsuite/multistep_web_tasks/webarena/browser_env/browser_utils.py`

---

### 3. Streaming Support ✓
**Coverage**: Fully implemented for large datasets

**Implementation**:
- **Iterator-based streaming**: `iter_jsonls(paths)` in `/evals/data.py:146-165`
  - Uses `itertools.islice()`  for memory-efficient processing
  - Supports optional `line_limit` parameter for batch control
  - Recursive directory traversal without loading entire dataset

- **Cloud streaming**: `_stream_jsonl_file(path)` in `/evals/data.py:105-109`
  - `bf.BlobFile(path, "r", streaming=True)` for GCS streaming
  - Line-by-line JSON parsing prevents memory overflow

- **Batching for recording events**: `/evals/record.py:381-400`
  - Configurable batch size (default 100)
  - Batch processing for HTTP/Snowflake recording

- **Sampling control**: `/evals/eval.py:26-38`
  - `_MAX_SAMPLES` global variable for limiting samples
  - Thread pool configuration: `EVALS_THREADS` environment variable (default 10)

**Evidence**: `/evals/data.py`, `/evals/record.py`, `/evals/eval.py`

---

### 4. Preprocessing Pipeline ✓
**Coverage**: Multiple transformation types implemented

**Tokenization & Normalization**:
- `normalize(s: str)` in `/evals/elsuite/utils.py`
  - Lowercases text, removes punctuation, articles, and extra whitespace
- `fuzzy_match()` for text similarity
- Text processing utilities across multiple eval suites

**Data Augmentation**:
- Noise injection in `/evals/elsuite/identifying_variables/renderers/tabular.py`
  - `apply_noise()` with signal-to-noise ratio control
  - `sparsify_data()` for creating missing values
- Sampling and filtering in dataset generation scripts

**Format Conversion**:
- CSV to JSONL: `/evals/elsuite/steganography/scripts/dataset/csv2jsonl.py` and `/evals/elsuite/text_compression/scripts/dataset/csv2jsonl.py`
- JSON parsing with error reporting: `_decode_json()` in `/evals/data.py:82-90`
- Dataclass/Pydantic serialization: `_to_py_types()` in `/evals/data.py:174-199`

**Data Generation & Processing**:
- Chess dataset preparation: `/evals/elsuite/cant_do_that_anymore/scripts/dataset_creation.py`
  - Move filtering, validation, decompression
- Error recovery data: `/evals/elsuite/error_recovery/scripts/dataset_creation.py`
  - Data filtering, column selection, subsetting
- Theory of Mind: `/evals/elsuite/theory_of_mind/scripts/data_generation.py`
  - Context building, format standardization
- Word association: `/evals/elsuite/already_said_that/scripts/gen_data.py`
  - Corpus processing, embeddings-based generation

**Evidence**: `/evals/data.py`, `/evals/elsuite/utils.py`, preprocessing scripts across registry/data/

---

### 5. Schema Standardization ✓
**Coverage**: Per-eval schema validation with automatic normalization

**Type Validation**:
- Pydantic BaseModel validation:
  - File: `/evals/elsuite/multiple_choice.py:14-48`
  - File: `/evals/elsuite/mmmu/eval.py:21-36`
  - Automatic field type checking and conversion

- Dataclass definitions:
  - File: `/evals/elsuite/identifying_variables/structs.py:8-49`
  - Typed fields with optional defaults

**Format Normalization**:
- `_to_py_types()` function in `/evals/data.py:174-199`
  - Converts Pydantic models, dataclasses, Paths to standard Python types
  - Handles recursive structures (dicts, lists)
  - Supports `exclude_keys` for field filtering

- `EnhancedJSONEncoder` in `/evals/data.py:202-208`
  - Custom JSON serialization for non-standard types
  - `jsondumps()` and `jsondump()` for standardized output

**Sample Schema Enforcement**:
- Basic evals require standard keys: `"input"`, `"ideal"`
  - `/evals/elsuite/basic/match.py:30-36` validates sample structure
  - `/evals/data.py` enforces format through loader type hints

- Model-graded evals: Flexible schema with validation
  - Keys defined by evaluation prompt templates
  - Schema union support (superset of keys supported)

**Evidence**: `/evals/data.py`, `/evals/elsuite/` eval implementations, data_test.py

---

### 6. Split Reproducibility ✓
**Coverage**: Deterministic seeding and split management

**Deterministic Sampling**:
- Fixed shuffle seed: `SHUFFLE_SEED = 123` in `/evals/eval.py:26`
- `_index_samples()` function in `/evals/eval.py:30-38`:
  ```python
  random.Random(SHUFFLE_SEED).shuffle(indices)
  ```
  - Uses reproducible seed for all sample shuffling
  - Indices paired with samples for deterministic ordering

**Configurable Seeds**:
- Eval seed parameter in `/evals/eval.py:56-74`:
  - Default seed: `20220722`
  - Per-sample seed generation: `f"{sample_id}:{self.seed}".encode("utf-8")`
  - Allows exact reproduction across runs

**Split Naming Convention**:
- Format: `<eval_name>.<split>.<version>` documented in `/docs/build-eval.md:57-61`
- Split examples: "val", "test", "dev"
- Version bumping ensures reproducible evaluation across changes

**Documentation**:
- `/docs/build-eval.md:62`: "running the same eval name against the same model should always give similar results so that others can reproduce it"

**Evidence**: `/evals/eval.py`, `/docs/build-eval.md`, eval naming pattern

---

### 7. Data Validation ✓
**Coverage**: Comprehensive multi-layer validation

**File Access Validation**:
- `/evals/data.py:47-79` - `open_by_file_pattern()`:
  - File existence checking
  - Local vs. registry path resolution
  - Multiple compression format support with fallback
  - Detailed error messages: `RuntimeError(f"Failed to open: {filename}")`

- `/evals/data.py:127-143` - Directory validation:
  - `bf.isdir()` checks
  - Raises `ValueError` for unsupported path types

**JSON Parsing Validation**:
- `/evals/data.py:82-90` - `_decode_json()`:
  - JSON structure validation with line/column error reporting
  - Error format: `f"Error parsing JSON on line {line_number}: {e.msg} at {path}:{line_number}:{e.colno}"`

**Schema Validation**:
- Pydantic BaseModel validation across evals
- Dataclass field type checking
- Sample structure assertions:
  - `/evals/elsuite/basic/match.py:30-36`:
    ```python
    assert isinstance(sample, dict), "sample must be a dict"
    assert "input" in sample, "sample must have an 'input' key"
    ```

**Registry Integrity**:
- `/evals/registry.py:273-285` - Duplicate detection:
  - `assert name not in registry, f"duplicate entry: {name} from {path}"`
- Reserved keyword validation
- Resource lookup with fuzzy matching using `difflib`

**Sample Count Validation**:
- Multiple evals validate sample counts before processing
- `/evals/elsuite/function_deduction/eval.py:293-302`:
  - Ensures dataset has sufficient samples
  - Assertion: `assert len(samples) >= self.n_samples`

**Configuration Validation**:
- Parameter range checks (e.g., `n_samples > 0`)
- Enum validation (target_language in valid set)
- Completion function count assertions

**File Existence Checks**:
- `/evals/elsuite/skill_acquisition/eval.py:276-279`:
  - Directory and file existence validation before processing

**Evidence**: `/evals/data.py`, `/evals/registry.py`, `/evals/eval.py`, `/evals/elsuite/` test files

---

### 8. Preprocessing Configuration ✓
**Coverage**: Configuration files and customizable parameters

**YAML Configuration**:
- Eval specifications in `/evals/registry/evals/` - define data sources and preprocessing parameters
- Solver configurations in `/evals/registry/solvers/` - specify model parameters
- ModelGraded specifications in `/evals/registry/modelgraded/` - define evaluation types

**Example Configuration Hierarchy**:
- `/docs/build-eval.md:42-53` shows eval YAML format:
  ```yaml
  <eval_name>:
    id: <eval_name>.dev.v0
    description: <description>
    metrics: [accuracy]

  <eval_name>.dev.v0:
    class: evals.elsuite.basic.match:Match
    args:
      samples_jsonl: <eval_name>/samples.jsonl
  ```

**Customizable Parameters**:
- Environment variables for preprocessing:
  - `EVALS_THREADS` - thread count (default 10)
  - `EVALS_SHOW_EVAL_PROGRESS` - progress display
  - `EVALS_SEQUENTIAL` - sequential vs. parallel mode
  - `EVALS_GENTLE_INTERRUPT` - graceful shutdown
- Per-eval configuration in class constructors
- Seed configuration for reproducibility

**Preprocessing Disabling**:
- Streaming can be disabled via `streaming=False` in `get_csv()`
- Individual transformations can be skipped in custom eval implementations
- Selective field inclusion via `exclude_keys` parameter in JSON serialization

**Documentation**:
- `/docs/eval-templates.md` - documents template parameters and customization
- `/docs/build-eval.md` - documents data formatting and registration
- `/docs/custom-eval.md` - custom eval implementation patterns

**Evidence**: `/evals/registry/` YAML files, `/docs/` documentation, eval class implementations

---

## Documentation Assessment

**Rating: Moderate**

The framework provides solid documentation with room for improvement:

**Strengths**:
- Clear data format specifications (JSONL, JSON, CSV)
- Split naming conventions documented
- Template parameter documentation in eval-templates.md
- Examples in docs/ and examples/ directories
- Inline code comments explaining complex logic

**Gaps**:
- Limited documentation on preprocessing pipeline customization
- Minimal guidance on streaming for very large datasets
- No dedicated guide for multi-modal data handling
- Limited schema validation examples beyond simple types
- API documentation could be more comprehensive

---

## Coverage Summary

| Sub-capability | Status | Evidence |
|---|---|---|
| Multiple format support | ✓ | `/evals/data.py`, `/evals/utils/snowflake.py` |
| Multi-modal data support | ✗ (limited) | `/evals/elsuite/mmmu/eval.py` |
| Streaming support | ✓ | `iter_jsonls()`, `_stream_jsonl_file()` |
| Preprocessing pipeline | ✓ | `normalize()`, data generators, format converters |
| Schema standardization | ✓ | Pydantic models, dataclasses, `_to_py_types()` |
| Split reproducibility | ✓ | `SHUFFLE_SEED`, seed parameter, naming convention |
| Data validation | ✓ | Multi-layer: file, JSON, schema, registry, config |
| Preprocessing configuration | ✓ | YAML registry, environment variables, parameters |

**Coverage**: 7 out of 8 capabilities = **87.5%**

**Grade**: **Present** (80-100% coverage threshold met)

---

## Key Implementation Files

| File | Purpose | Key Functions |
|---|---|---|
| `/evals/data.py` | Core data I/O | `get_jsonl()`, `iter_jsonls()`, `get_csv()`, `get_json()` |
| `/evals/eval.py` | Eval base class | `_index_samples()`, `eval_all_samples()` |
| `/evals/registry.py` | Registry management | `_load_directory()`, `_validate_reserved_keywords()` |
| `/evals/utils/snowflake.py` | Database integration | `SnowflakeConnection.query()` |
| `/evals/elsuite/utils.py` | Preprocessing utils | `normalize()`, `fuzzy_match()` |
| `/evals/record.py` | Event recording/batching | Batch processing, HTTP/Snowflake output |

---

## Notable Limitations

1. **Multi-modal limitations**:
   - Image support only (no audio/video)
   - No comprehensive mixed-modality pipelines
   - Limited structured data type handling

2. **Database support**:
   - Snowflake only; no PostgreSQL, MySQL, MongoDB
   - No native query abstraction layer

3. **API integration**:
   - REST only; no GraphQL support
   - Limited to requests library patterns

4. **Format support**:
   - No Parquet support
   - No Protocol Buffers or Avro

These limitations align with the framework's focus on LLM evaluation rather than general-purpose data engineering.

---

## Conclusion

OpenAI Evals provides **comprehensive data loading and preprocessing capabilities** with strong support for multiple formats (JSONL, JSON, CSV, REST, GCS, Snowflake), streaming for large datasets, deterministic reproducibility, and robust data validation. The main gap is limited multi-modal support (images only). The framework is well-suited for LLM evaluation workflows, with clear data format conventions and reproducible sampling mechanisms.
