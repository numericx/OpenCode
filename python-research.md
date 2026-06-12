# Python Development Best Practices

## Executive Summary

The Python ecosystem has undergone its most significant tooling consolidation in a decade. **Astral** (creators of Ruff) has become the dominant force, and `uv` and `ruff` form the de facto standard for new Python projects in 2026. Type hinting is mainstream, `pyproject.toml` (PEP 621) is universal, and the monolithic toolchain has been replaced by fast, focused Rust-based tools.

---

## 1. Package & Dependency Management

### Recommendation: `uv` (primary) for new projects

| Aspect | Recommendation | Rationale |
|--------|----              |-----               |
| **New projects** | `uv init` / `uv add` / `uv lock` / `uv sync` | Single binary, cargo-style DX, deterministic lockfiles (`uv.lock`), 10-100x faster than pip/poetry |
| **Python version mgmt** | `uv python install` / `uv python pin` | Built-in, no separate pyenv needed |
| **Script dependencies** | PEP 723 inline scripts via `# /// script` | No project scaffolding needed for quick scripts |
| **CI pipelines** | `astral-sh/setup-uv` GitHub Action + `uv run` | Cuts CI setup times from minutes to seconds |
| **Data science** | `conda` / `mamba` | Handles non-Python deps (MKL, CUDA) that `uv` cannot |
| **Existing Poetry** | `poetry 2.0+` (released Jan 2025) | Now supports PEP 621 `[project]` table; migration to `uv` recommended for new work |
| **Avoid** | `pipenv` | Largely abandoned |

### Example `pyproject.toml` (2026 standard)

```toml
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "httpx>=0.28",
    "pydantic>=2.10",
    "structlog>=24.1",
]

[dependency-groups]          # PEP 735
dev = ["pytest>=8", "ruff", "mypy", "pytest-cov", "hypothesis"]
docs = ["mkdocs-material"]

[build-system]
requires = ["hatchling >= 1.26"]
build-backend = "hatchling.build"
```

---

## 2. Linting, Formatting & Import Sorting

### Recommendation: `ruff` for everything

**Why Ruff replaces the entire legacy toolchain:**

| What Ruff Replaces | Rule Prefix | Function |
|--------------------|-------------|----------|
| flake8             | E, W        | Basic syntax errors |
| pyflakes           | F           | Unused imports, undefined names |
| pyupgrade          | UP          | Modernize to latest Python syntax |
| isort              | I           | Import sorting |
| pycodestyle        | E, W        | Style conventions |
| pylint             | PL          | Code smell detection |
| mccabe             | C90         | Complexity |
| flake8-bugbear     | B           | Common bugs |
| flake8-simplify    | SIM         | Simplify expressions |
| pydocstyle         | D           | Docstring style |
| flake8-quotes      | Q           | Quote conventions |

**Recommended configuration:**

```toml
[tool.ruff]
target-version = "py312"
line-length = 88
exclude = [".git", ".venv", "__pycache__", "build", "dist"]

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "F",    # pyflakes
    "I",    # isort
    "UP",   # pyupgrade (modernize syntax)
    "B",    # flake8-bugbear (common bugs)
    "SIM",  # flake8-simplify
    "PIE",  # flake8-pie (idiomatic Python)
    "RUF",  # ruff-specific rules
]
ignore = ["E501"]  # formatter handles line length

[tool.ruff.lint.isort]
known-first-party = ["myproject"]
combine-as-imports = true

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
docstring-code-format = true
```

### Black vs `ruff format`

- Use `ruff format` for **new projects** (one binary, identical Black-compatible output, 10-50x faster)
- Use `black` only if legacy repo requires exact compatibility or org policy mandates it

---

## 3. Type Hinting

### Recommendation: mypy (CI) + pyright (local) combined approach

| Aspect | Recommendation | Notes |
|--------|----              |-----               |
| **CI type checking** | `mypy src/ --config-file pyproject.toml` | Most mature, granular config, rich plugin ecosystem |
| **Local / IDE** | `pyright` via Pylance in VS Code or built-in PyCharm | Instant feedback, faster for large repos |
| **Strict mode** | `--disallow-untyped-defs --disallow-any-generics` | Use for libraries; gradual typing for large codebases |
| **Runtime validation** | `pydantic` (standard) / `typeguard` (lightweight) | Type hints are not enforced at runtime |
| **Python version** | `>=3.12` minimum | Enables PEP 695 (Type parameter syntax), PEP 754 (`TypeIs`), PEP 742 (`TypeGuard`) |

**Key typing features enabled by Python 3.12+:**

- **PEP 695** (Type Parameter Syntax): `def identity[T](x: T) -> T`
- **PEP 742** (`TypeGuard`): Narrow types by subtype relationship
- **PEP 754** (`TypeIs`): Narrow types by identity check
- **PEP 646** (`TypeVarTuple`): Generic tuples with variable length
- **PEP 723** (Inline script metadata): Single-file scripts with dependency declarations

**Example mypy `pyproject.toml` config:**

```toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_untyped_decorators = true
ignore_missing_imports = false
```

---

## 4. Testing

### Recommendation: `pytest` + `pytest-cov` as the foundation

| Aspect | Recommendation | Notes |
|--------|----              |-----                      |
| **Framework** | `pytest` | Universal standard; nothing competes |
| **Coverage** | `pytest-cov` | `--cov-report=term-missing,html` and `--cov-report=xml` |
| **Property-based testing** | `hypothesis` | For parsing, validation, complex logic where edge cases matter |
| **Async testing** | `pytest-asyncio` | Mature; supports auto-marking for all async |
| **Parallel execution** | `pytest-xdist` | Essential for large suites (`-n auto`) |
| **Snapshot testing** | `syrupy` | Alternative to manual assertions |
| **Test data** | `factory-boy` (models) / `faker` (generic) | |
| **Multi-Python testing** | `tox` or `nox` | |
| **Target coverage** | 80-95% | Branch coverage recommended |

**Example pytest config:**

```toml
[tool.pytest.ini_options]
minversion = "8.0"
addopts = [
    "-ra",
    "--strict-markers",
    "--strict-config",
    "--cov=src/myproject",
    "--cov-report=term-missing",
    "--cov-report=html",
]
testpaths = ["tests"]
pythonpath = ["src"]
markers = [
    "slow: marks tests as slow (deselect with -m 'not slow')",
    "integration: marks test as integration test",
    "serial: marks test that must run serially",
]
```

---

## 5. Documentation

### Recommendation: MkDocs Material (most projects) or Sphinx (API-heavy libraries)

| Aspect | Recommendation | Notes |
|--------|----              |-----                         |
| **Most projects** | `mkdocs` + Material theme | Simplest setup; `mkdocs.yml` only; live reload; beautiful dark mode; search |
| **API-heavy libraries** | `sphinx` + `sphinx.ext.autodoc` + `sphinx.ext.napoleon` | Auto-generates API docs from docstrings; cross-referencing; used by pandas, numpy, celery |
| **Docstring style** | Google style (default) | Simple, readable; NumPy style for scientific computing; Sphinx style for advanced needs |
| **Enforcement** | `ruff D-rules` or `pydocstyle` | `ruff.lint.pydocstyle.convention = "google"` |

**MkDocs `mkdocs.yml` minimal config:**

```yaml
site_name: My Project
theme:
  name: material
  palette:
    - scheme: default
      toggle:
        icon: material/toggle-switch-off-outline
        name: Switch to dark mode
    - scheme: slate
      toggle:
        icon: material/toggle-switch
        name: Switch to light mode
  features:
    - content.code.copy
    - navigation.instant
    - navigation.tracking
repo_url: https://github.com/username/myproject
nav:
  - Home: index.md
  - API: api.md
```

---

## 6. Project Structure

### Recommended layout

```
myproject/
├── pyproject.toml          # ALL project metadata, deps, tool configs
├── uv.lock                 # Deterministic lockfile
├── src/                    # src/ layout (strongly recommended)
│   └── myproject/
│       ├── __init__.py
│       ├── cli.py
│       ├── config.py
│       └── api/
├── tests/
│   ├── conftest.py
│   └── api/
├── docs/
│   ├── mkdocs.yml
│   └── docs/
├── .pre-commit-config.yaml
├── Dockerfile
└── README.md
```

### Why `src/` layout

- Prevents accidental imports of uninstalled package
- Prevents shadowing stdlib modules
- Better for editable installs (`uv pip install -e .`)
- Forces proper package structure

### Key conventions

| Convention | Recommendation |
|------------|---------------|
| **Metadata** | `pyproject.toml` (PEP 621) exclusively — no `setup.py`, no `setup.cfg`, no `requirements.txt` |
| **Package name** | `my-project` (PyPI name) maps to `my_project` (import name) |
| **Build backend** | `hatchling` (recommended) or `setuptools` (universal) |
| **`__init__.py`** | Keep minimal; use lazy imports; omit for namespace packages (PEP 420) |
| **`py.typed`** | Include PEP 561 marker in package root if shipped with types |
| **Lockfile** | `uv.lock` (generated by `uv lock`) — never edit manually |
| **Dependencies** | PEP 508 strings in `[project.dependencies]`; dev deps in `[dependency-groups]` (PEP 735) |

### Build Backend Comparison

| Backend | PEP 621 | PEP 639 (SPDX) | Notes |
|---------|---------|-----            |----- |
| **hatchling** | Yes | 1.27.0+ | Plugin ecosystem, reproducible builds by default, FastAPI default |
| **setuptools** | Yes | 77.0.3+ | Legacy compatibility, widest adoption historically |
| **flit_core** | Yes | 3.12+ | Simple, lightweight, best for pure-Python packages |
| **uv_build** | Yes | 0.7.19+ | New by Astral, Rust-based |

---

## 7. CI/CD

### Recommended pipeline (GitHub Actions + `uv`)

```yaml
name: CI
on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync --group dev
      - run: uv run ruff check .
      - run: uv run ruff format --check .

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync --group dev
      - run: uv run mypy src/myproject

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.12", "3.13"]
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
        with: { python-version: ${{ matrix.python-version }} }
      - run: uv sync --group dev
      - run: uv run pytest --strict-markers -n auto
      - uses: codecov/codecov-action@v5

  publish:
    needs: [lint, typecheck, test]
    if: startsWith(github.ref, 'refs/tags/')
    runs-on: ubuntu-latest
    permissions: { id-token: write }
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv build
      - run: uv publish
```

### Pre-commit hooks (`.pre-commit-config.yaml`)

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.15.0
    hooks:
      - id: mypy
        additional_dependencies: [pydantic, types-requests]
```

### CI Best Practices

| Practice | Recommendation |
|----------|---------------|
| **Python version management** | Use `uv` (fastest setup) |
| **Parallel test execution** | `uv run pytest -n auto` (pytest-xdist) |
| **Multiple Python versions** | Matrix build (3.12, 3.13) |
| **Coverage upload** | `codecov/codecov-action@v5` |
| **Pre-commit** | Use locally + verify in CI |
| **Skip slow tests in CI** | Run only with `pytest -m "not slow"` unless needed |
| **Dependency caching** | Use `actions/cache` or `setup-uv` built-in caching |
| **Publish** | Trusted publishers (PEP 740, no credentials needed) |

---

## 8. Code Quality & Security

| Tool | Use | Command |
|------|-----|---------|
| **ruff** | Lint + format (unilateral) | `ruff check . && ruff format .` |
| **semgrep** | General SAST (optional but recommended for critical apps) | `semgrep --config "p/python"` |
| **bandit** | Python-specific security (optional) | `bandit -r src/` |
| **mypy** | Type-related bugs | `mypy src/` |
| **pytest** | Functional bugs | `pytest` |
| **hypothesis** | Edge cases | `pytest` (with hypothesis included) |
| **safety** | Dependency vulnerability scanning | `safety check -r requirements.txt` |

### Security scanning strategy

1. **`ruff`** catches basic security issues (`S` rules): hard-coded passwords, SQL injection, use of `eval()`, weak crypto
2. **`semgrep`** for deeper static analysis with pattern matching (AST-based, not regex) + custom rule support
3. **`bandit`** specifically for Python-focused security scanning when a dedicated scanner is needed
4. Dependency vulnerability checks: `safety check` against known vulnerability databases

---

## 9. Complete Toolchain Summary

| Category | New Projects (2026) | Notes |
|--        |---------          |----- |
| **Package manager** | `uv` | 10-100x faster than pip/poetry |
| **Linting** | `ruff check` | Replaces flake8, pylint, mccabe, etc. |
| **Formatting** | `ruff format` | Replaces black for new projects |
| **Import sorting** | `ruff lint --select I` | Replaces isort |
| **Type Checking (CI)** | `mypy` | Mature, granular |
| **Type Checking (local)** | `pyright` | Instant IDE feedback |
| **Testing** | `pytest` | Universal standard |
| **Coverage** | `pytest-cov` | |
| **Property testing** | `hypothesis` | Edge case discovery |
| **Async testing** | `pytest-asyncio` | |
| **Parallel testing** | `pytest-xdist` | `-n auto` |
| **Snapshot testing** | `syrupy` | |
| **Pre-commit** | `pre-commit` | |
| **Docs** | `mkdocs-material` / `sphinx` | Depends on project type |
| **Build backend** | `hatchling` | Or `setuptools` for universal compat |
| **CI/CD** | GitHub Actions + `uv` | Fastest setup |

---

## Key Trends (2025-2026)

1. **The Astral ecosystem dominates.** `uv` + `ruff` + `ty` (emerging type checker from Astral) is becoming the default toolchain. One binary for linting, formatting, imports, and docstring checking.

2. **PEP 735 dependency groups.** Native dev dependency groups in `pyproject.toml` eliminate legacy hacks with `extras` or separate `requirements-dev.txt` files.

3. **Type hinting is mainstream.** New projects should use type hints from day one. Python 3.12+ enables PEP 695 (Type Parameter Syntax) and PEP 754 (`TypeIs`).

4. **Python 3.12+ is the minimum.** All tooling targets 3.12+; 3.13 adoption is accelerating. No one should be starting new projects on anything older.

5. **`pyproject.toml` is the only config file.** `setup.py`, `setup.cfg`, `requirements.txt`, `tox.ini` are all legacy. All tool configurations live under `[tool.<name>]`.

6. **`uv.lock` replaces all lockfiles.** A single universal lockfile (Cargo-style) replaces `poetry.lock`, `Pipfile.lock`, `requirements.txt`, `pip-tools` output.

7. **PEP 639 standardizes SPDX licenses.** The `license` field is now a simple string (e.g., `license = "MIT"`) instead of a table with `file`/`text` keys.

8. **Trusted publishers replace credentials.** PEP 740 enables publishing to PyPI without any API tokens or credentials via OIDC.

9. **Monorepo support is first-class.** Cargo-style workspaces (`uv.lock` with workspace tables) make monorepos feel native in Python for the first time.

---

## Appendix: Quick Start Checklist

For a new Python project in 2026, the setup workflow should be:

```bash
# 1. Initialize project
uv init myproject --lib

# 2. Add runtime dependencies
uv add httpx pydantic

# 3. Add dev dependencies
uv add --dev ruff pytest mypy pytest-cov hypothesis
uv add --group docs mkdocs-material

# 4. Configure tools in pyproject.toml (see sections above)

# 5. Add pre-commit hooks
uv add --dev pre-commit
pre-commit install

# 6. Create src layout (uv may do this by default)
mkdir -p src myproject tests

# 7. Run tooling
uv run ruff check .          # Lint
uv run ruff format .          # Format
uv run mypy src/              # Type check
uv run pytest --cov=src/      # Test
uv run mkdocs serve           # Docs
```

---

*Research conducted May 2026. Sources: PEPs 621/735/639/740/695/754/742/646/723/561/420; Ruff docs; uv docs; Poetry 2.0 docs; PyPA packaging specs; Astral engineering blog; community adoption data.*
