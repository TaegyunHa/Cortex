---
related:
  - "[[Programming]]"
created: 2026-01-02
tags:
---
## UV Homepage
- https://docs.astral.sh/uv/

## Installation

MacOS/Linux
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Windows
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

## Install Package in editable mode

Without proper `pyproject.toml` setup
```bash
uv pip install -e .
```

With `pyproject.toml` setup
```bash
uv sync
```

