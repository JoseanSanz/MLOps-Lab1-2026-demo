[![CI](https://github.com/JoseanSanz/MLOps-Lab1-demo/actions/workflows/CI.yml/badge.svg)](https://github.com/JoseanSanz/MLOps-Lab1-demo/actions/workflows/CI.yml)

# MLOps-Lab1-demo

Demo for the first lab of the MLOps subject. A simple calculator is used as the
application, so that the focus is on how the project is organized, tested and checked.
In this lab you will learn to:

* Program the **logic** of an application as a reusable library.
* Program a **Command Line Interface** (click) to interact with the logic.
* Program a **REST API** (FastAPI) to use the logic as a microservice (backend).
* Manage the project and its dependencies with **uv** (versions pinned in `uv.lock`)
  and automate the common tasks with a **Makefile**.
* **Test** all the functionalities with **pytest** and check the code with **black** and **pylint**.
* Build a **CI pipeline** with **GitHub Actions** that runs all the checks on every push.

## Project structure

```
├── mylib/calculator.py        # business logic (shared by the CLI and the API)
├── cli/cli.py                 # command-line interface (click)
├── api/api.py                 # REST API (FastAPI): /, /health, /calculate
├── templates/home.html        # home page of the API
├── tests/
│   ├── test_logic.py          # unit tests of the logic
│   ├── test_cli.py            # integration tests of the CLI (CliRunner)
│   └── test_api.py            # integration tests of the API (TestClient)
├── Makefile                   # common tasks
├── pyproject.toml / uv.lock   # dependencies (runtime and dev group)
└── .github/workflows/CI.yml   # CI pipeline
```

## Run it locally

Requirements: [uv](https://docs.astral.sh/uv/) (installed by `make install` if missing).

```bash
make install          # installs all the dependencies (runtime + dev) from uv.lock
make test             # tests with coverage (fails below 90 %)
make format           # formats the code with black
make lint             # runs pylint
make all              # install + format-check + lint + test (what the CI does)
```

CLI:

```bash
uv run python -m cli.cli --help
uv run python -m cli.cli add 1 2
uv run python -m cli.cli divide 6 3
```

API:

```bash
make run-api          # http://localhost:8000/docs (interactive documentation)
curl http://localhost:8000/health
curl -X POST http://localhost:8000/calculate -d "op=add" -d "a=2" -d "b=3"
```

| Method | Path | Input | Answer |
|---|---|---|---|
| GET | `/` | – | Home page with a link to `/docs` |
| GET | `/health` | – | `{"status": "ok"}` |
| POST | `/calculate` | form data: `op` (add, subtract, multiply, divide, power), `a`, `b` | `{"result": ...}`; 400 for an invalid operation, division by zero or a result too large; 422 for missing or non-numeric parameters |

When you change the dependencies, use `uv add <package>` (or `uv add --dev <package>`)
and commit the updated `uv.lock`: `make install` uses `uv sync --locked` and fails if the
lock file is out of date.

## CI pipeline

`.github/workflows/CI.yml` runs on every push and pull request to `main`:

1. `make install`: installs the dependencies (fails if `uv.lock` is out of date).
2. `make format-check`: fails if the code is not formatted with black.
3. `make lint`: pylint (errors and warnings).
4. `make test`: pytest with a minimum coverage of 90 %.

The badge at the top of this file shows the result of the last run.
