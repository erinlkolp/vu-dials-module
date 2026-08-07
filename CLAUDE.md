# CLAUDE.md

## Project Overview

`vudials_client` is a Python client library for Streacom's VU1 Dial hardware. It exposes two main classes:

- **`VUDial`** — controls individual dials (value, color, background image, name, easing)
- **`VUAdmin`** — manages the VU1 server (API key lifecycle, dial provisioning)

The library communicates with a local VU1 server over plain HTTP using a key-in-URL query-parameter authentication scheme.

**Package name (PyPI):** `vudials-client`
**Import path:** `from vudials_client import vudialsclient`
**Versioning scheme:** calendar versioning, `YYYY.M.patch`

`pyproject.toml` is the single source of truth for the current version, the
supported Python range (`requires-python`), and all runtime and `[dev]`
dependency pins. Read them from there rather than restating them here — a
duplicated value in this file is a value that will silently go stale.

---

## Repository Layout

```
src/vudials_client/
    __init__.py          # empty — intentional
    vudialsclient.py     # VUUtil, VUAdminUtil, VUDial, VUAdmin

tests/
    __init__.py
    conftest.py          # pytest fixtures: vudial, vuadmin
    test_vudialsclient.py

.github/workflows/
    ci.yml               # matrix CI across the supported Python versions

pyproject.toml           # build system, deps, pytest & coverage config
pydoc-markdown.yml       # generates docs/api.md from docstrings
requirements.txt         # legacy; mirrors the runtime dep in pyproject.toml
```

---

## Development Setup

```bash
git clone git@github.com:erinlkolp/vu1-dial-python-module.git
cd vu1-dial-python-module
pip install -e ".[dev]"
```

The `[dev]` extra installs the test tooling; see `[project.optional-dependencies]` in `pyproject.toml` for the current list.

---

## Running Tests

```bash
# Run all tests with coverage
pytest tests/ -v --cov=vudials_client --cov-report=term-missing

# Run a specific test class
pytest tests/test_vudialsclient.py::TestVUDialSetDialColor -v

# Run without coverage (faster)
pytest tests/ -v
```

Tests use the `responses` library to mock all HTTP calls — no live server is needed. The two shared fixtures in `conftest.py` are `vudial` (a `VUDial` pointed at `localhost:5340`) and `vuadmin` (a `VUAdmin` pointed at the same address).

---

## Class Architecture

### Utility base classes (internal)

| Class | Purpose |
|---|---|
| `VUUtil` | Builds `key=` query-param URIs and dispatches GET / multipart-POST |
| `VUAdminUtil` | Builds `admin_key=` query-param URIs and dispatches GET / POST |

### Public classes

| Class | Inherits | Key concern |
|---|---|---|
| `VUDial` | `VUUtil` | Dial control: value, color, image, name, easing |
| `VUAdmin` | `VUAdminUtil` | Server admin: provision dials, CRUD on API keys |

Every method returns a raw `requests.Response`. Callers must parse `.json()` or check `.status_code` themselves. All methods call `raise_for_status()` internally, so HTTP 4xx/5xx raise `requests.exceptions.HTTPError`.

---

## Security Notes

These are pre-existing design constraints of the VU1 server API — do not silently "fix" them:

1. **Plain HTTP only.** The server only listens on HTTP; HTTPS is not supported. Keep traffic on trusted local interfaces.
2. **Key-in-URL authentication.** Both `key=` and `admin_key=` parameters appear in the query string and will be recorded in server access logs, proxy logs, and HTTP client history. This is intentional until the upstream server adds header-based auth.
3. **No input validation in the library.** The library does not validate value ranges (e.g., RGB 0–100) or UID format; that is the server's responsibility.

---

## Code Conventions

- **No `logging.basicConfig()` in library code.** The module registers a named logger (`logging.getLogger(__name__)`) but never configures the root logger. Leave this pattern in place.
- **`urllib.parse.quote` for user-supplied strings** in URL path segments (`uid`) and most query parameters to prevent URL injection. Do not remove these calls.
- **Type annotations on all public methods.** Parameters and return types (`-> requests.Response`) must be annotated.
- **Docstrings on all public methods** using the `:param name: description` / `:return:` style already present.

---

## Adding New API Endpoints

1. Identify whether the new endpoint belongs to the dial API (use `VUDial` + `VUUtil`) or the admin API (use `VUAdmin` + `VUAdminUtil`).
2. URL-encode any user-supplied path segment with `quote(value, safe="")`.
3. URL-encode query-parameter values with `quote(value)`.
4. Coerce numeric parameters with `int()` before interpolating into the query string.
5. Return the raw `requests.Response` without unwrapping JSON.
6. Add a corresponding test class in `tests/test_vudialsclient.py` following the existing pattern: at minimum test the success path, the correct endpoint, key parameters, and HTTP error propagation.

---

## Versioning & Publishing

The project uses **calendar versioning** (`YYYY.M.patch`). To release a new version:

1. Update `version` in `pyproject.toml`.
2. Ensure the changelog / README reflects the change.
3. Tag the commit and push; PyPI publishing is done manually via `hatch build && hatch publish` or the equivalent `twine upload`.

---

## CI/CD

GitHub Actions runs `.github/workflows/ci.yml` on every push to `main` or any `claude/**` branch, and on PRs targeting `main`.

- Matrix: `ubuntu-latest` across the Python versions listed in `ci.yml` (`jobs.test.strategy.matrix.python-version`), which must stay consistent with `requires-python` in `pyproject.toml`
- Command: `pytest tests/ -v --cov=vudials_client --cov-report=term-missing --cov-report=xml`
- Coverage XML is uploaded as a build artifact (Python 3.12 run only)

---

## Documentation Generation

API docs in `docs/api.md` are generated from docstrings via `pydoc-markdown`:

```bash
pip install pydoc-markdown
pydoc-markdown
```

Configuration is in `pydoc-markdown.yml` (source: `src/vudials_client/`, output: `docs/api.md`).
