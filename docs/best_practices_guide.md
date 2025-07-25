# Best Practices Guide

This guide summarizes the best practices for developing, maintaining, and contributing to the Photo Post-Processing Pipeline. It is based on modern Python standards, reproducibility, security, and code quality.

---

## 1. Project Structure

- Source code in `src/`
- Tests in `tests/`
- Documentation in `docs/`
- All configuration in `pyproject.toml`
- `.pre-commit-config.yaml` for code quality automation
- `Makefile` for common developer tasks (optional)

---

## 2. Dependency Management

- Pin all dependencies in `pyproject.toml` for reproducibility
- Use [uv](https://github.com/astral-sh/uv) for fast, reliable installs and lockfile management
- Avoid `requirements.txt` and legacy `setup.cfg`/`setup.py` unless needed for legacy compatibility

---

## 3. Linting, Formatting, and Type Checking

- Use [Ruff](https://pypi.org/project/ruff/) for linting and formatting (`ruff check`, `ruff format`)
- Use [mypy](https://pypi.org/project/mypy/) for static type checking
- Enforce code style and type safety in CI and pre-commit hooks

---

## 4. Testing

- Write all tests in the `tests/` directory
- Use [pytest](https://pypi.org/project/pytest/) for all test discovery and execution
- Use [pytest-cov](https://pypi.org/project/pytest-cov/) for coverage reporting
- Ensure all new features and bugfixes include tests
- Run tests locally with `uv run pytest` and in CI

---

## 5. Security

- Use [bandit](https://pypi.org/project/bandit/) for static security analysis
- Use [safety](https://pypi.org/project/safety/) to check for vulnerable dependencies
- Run security checks in CI and before releases

---

## 6. Pre-commit Hooks

- Use [pre-commit](https://pre-commit.com/) to automate code quality checks before every commit
- Install hooks with `uv run pre-commit install`
- Run all hooks on demand with `uv run pre-commit run --all-files`

---

## 7. Continuous Integration (CI)

- Use GitHub Actions for automated testing, linting, type checking, and security
- Ensure all checks pass before merging PRs
- Upload coverage and security reports as artifacts

---

## 8. Documentation

- Keep `README.md` and all guides in `docs/` up to date
- Document all public functions and modules with docstrings
- Use clear, concise language and provide usage examples

---

## 9. Contribution Guidelines

- Fork the repo and create feature branches for changes
- Use `uv sync --all-extras` to install all dependencies
- Use the provided `uv` commands or `Makefile` for all dev tasks
- Ensure all tests, lint, type, and security checks pass before submitting a PR
- Follow the code style and structure of the project

---

## 10. General Python Best Practices

- Prefer explicit over implicit code
- Use type annotations everywhere
- Handle exceptions gracefully and log errors
- Avoid global state and side effects
- Write small, testable functions and classes

---

## References

- [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [PEP 621 – Storing project metadata in pyproject.toml](https://peps.python.org/pep-0621/)
- [Ruff Documentation](https://docs.astral.sh/ruff/)
- [mypy Documentation](https://mypy.readthedocs.io/en/stable/)
- [pytest Documentation](https://docs.pytest.org/en/stable/)
- [pre-commit Documentation](https://pre-commit.com/)
- [bandit Documentation](https://bandit.readthedocs.io/en/latest/)
- [safety Documentation](https://pyup.io/safety/)
# Project Best Practices Implementation Guide

## 📁 1. ENHANCED PROJECT STRUCTURE

### Current Structure (Good):
```
photo_post_processing/
├── src/
│   ├── utils/
│   └── process_photos.py
├── tests/
├── docs/
├── assets/
├── requirements.txt
└── README.md
```

### Recommended Structure (Better):
```
photo_post_processing/
├── src/
│   ├── photo_processor/           # Main package
│   │   ├── __init__.py
│   │   ├── core/                  # Core processing logic
│   │   │   ├── __init__.py
│   │   │   ├── image_processor.py
│   │   │   └── pipeline.py
│   │   ├── utils/                 # Utilities
│   │   ├── models/                # Data models/classes
│   │   └── config/                # Configuration
│   ├── cli.py                     # Command line interface
│   └── main.py                    # Entry point
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── docs/
├── scripts/                       # Development scripts
├── .github/                       # GitHub workflows
├── requirements/                  # Multiple requirement files
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── Dockerfile
├── docker-compose.yml
├── pytest.ini
├── setup.py or pyproject.toml
└── Makefile
```

## ⚙️ 2. CONFIGURATION MANAGEMENT

### Issues with Current Config:
- Hardcoded paths
- No environment separation
- Mixed concerns

### Recommended Improvements:
```python
# config/settings.py
from pathlib import Path
from typing import Dict, Any
import os
from dataclasses import dataclass

@dataclass
class ProcessingConfig:
    """Processing configuration settings"""
    jpeg_quality: int = 90
    watermark_opacity: float = 0.9
    watermark_scale: float = 0.15
    enable_brightness_adjust: bool = False

@dataclass
class PathConfig:
    """Path configuration"""
    base_dir: Path = Path.cwd()
    input_dir: Path = None
    output_dir: Path = None
    assets_dir: Path = None

    def __post_init__(self):
        if self.input_dir is None:
            self.input_dir = self.base_dir / "input"
        if self.output_dir is None:
            self.output_dir = self.base_dir / "output"
        if self.assets_dir is None:
            self.assets_dir = self.base_dir / "assets"

class Config:
    """Main configuration class"""
    def __init__(self, env: str = "development"):
        self.env = env
        self.processing = ProcessingConfig()
        self.paths = PathConfig()

    @classmethod
    def from_env(cls) -> "Config":
        env = os.getenv("PHOTO_PROCESSOR_ENV", "development")
        return cls(env)
```

## 🧪 3. TESTING FRAMEWORK

### Current Testing Issues:
- Custom test runner instead of pytest
- No test configuration
- Mixed test types

### Recommended Testing Setup:
```python
# pytest.ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts =
    --strict-markers
    --strict-config
    --verbose
    --tb=short
    --cov=src
    --cov-report=html
    --cov-report=term-missing
markers =
    unit: Unit tests
    integration: Integration tests
    slow: Slow running tests
    requires_images: Tests that need sample images
```

## 📦 4. DEPENDENCY MANAGEMENT

### Current Requirements Issues:
- Minimal dependencies
- No version pinning strategy
- No development dependencies

### Recommended Approach:
```
requirements/
├── base.txt          # Core dependencies
├── dev.txt          # Development dependencies
├── test.txt         # Testing dependencies
└── docs.txt         # Documentation dependencies
```

## 🔍 5. CODE QUALITY TOOLS

### Essential Tools to Add:
- **Black**: Code formatting
- **isort**: Import sorting
- **flake8**: Linting
- **mypy**: Type checking
- **pre-commit**: Git hooks

### Configuration Files:
```ini
# .flake8
[flake8]
max-line-length = 88
select = E,W,F
ignore = E203,E501,W503
exclude = .git,__pycache__,docs/source/conf.py,old,build,dist
```

## 🚀 6. CI/CD PIPELINE

### GitHub Actions Workflow:
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9, "3.10", "3.11"]

    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v3
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements/dev.txt
    - name: Run tests
      run: pytest
    - name: Run linting
      run: flake8 src tests
    - name: Type checking
      run: mypy src
```

## 📚 7. DOCUMENTATION

### Current Documentation Issues:
- Basic README
- No API documentation
- No usage examples

### Recommended Documentation:
```
docs/
├── README.md
├── API.md
├── CONTRIBUTING.md
├── CHANGELOG.md
├── examples/
│   ├── basic_usage.py
│   ├── advanced_processing.py
│   └── batch_processing.py
└── architecture/
    ├── design_decisions.md
    └── system_overview.md
```

## 🐳 8. CONTAINERIZATION

### Dockerfile:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements/base.txt .
RUN pip install -r base.txt

COPY src/ .
COPY assets/ ./assets/

CMD ["python", "main.py"]
```

## 🔧 9. DEVELOPMENT TOOLS

### Makefile for Common Tasks:
```makefile
.PHONY: install test lint format clean

install:
    pip install -r requirements/dev.txt

test:
    pytest

lint:
    flake8 src tests
    mypy src

format:
    black src tests
    isort src tests

clean:
    find . -type d -name __pycache__ -delete
    find . -name "*.pyc" -delete
    rm -rf .coverage htmlcov/ .pytest_cache/
```

## 🛡️ 10. ERROR HANDLING & LOGGING

### Current Issues:
- Minimal error handling
- No logging system
- Silent failures

### Recommended Improvements:
```python
# utils/logging_config.py
import logging
import sys
from pathlib import Path

def setup_logging(level: str = "INFO", log_file: Path = None):
    """Configure logging for the application"""

    formatters = {
        'detailed': logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        ),
        'simple': logging.Formatter('%(levelname)s - %(message)s')
    }

    handlers = [
        logging.StreamHandler(sys.stdout)
    ]

    if log_file:
        handlers.append(logging.FileHandler(log_file))

    logging.basicConfig(
        level=getattr(logging, level.upper()),
        handlers=handlers,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
```

## 📊 11. MONITORING & METRICS

### Performance Tracking:
```python
# utils/metrics.py
import time
import psutil
from functools import wraps

def track_performance(func):
    """Decorator to track function performance"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        start_memory = psutil.Process().memory_info().rss

        result = func(*args, **kwargs)

        end_time = time.time()
        end_memory = psutil.Process().memory_info().rss

        logger.info(f"{func.__name__} took {end_time - start_time:.2f}s")
        logger.info(f"Memory usage: {(end_memory - start_memory) / 1024 / 1024:.2f}MB")

        return result
    return wrapper
```

## 🎯 IMMEDIATE PRIORITY RECOMMENDATIONS

### High Priority (Implement First):
1. **Add proper testing with pytest**
2. **Set up code formatting (Black + isort)**
3. **Add proper logging throughout the application**
4. **Create environment-based configuration**
5. **Add CI/CD pipeline with GitHub Actions**

### Medium Priority:
1. **Containerize the application**
2. **Add comprehensive documentation**
3. **Implement proper error handling**
4. **Add performance monitoring**

### Low Priority (Future Enhancements):
1. **Add API endpoints (Flask/FastAPI)**
2. **Database integration for processing history**
3. **Web interface for batch processing**
4. **Cloud deployment automation**
