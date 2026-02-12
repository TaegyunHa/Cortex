# Building a Smart CI/CD Pipeline for FastAPI + Vue.js Monorepos: A Complete Guide

**Learn advanced GitHub Actions patterns including path filtering, schema-driven development, smart caching, and dependency orchestration**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Project Architecture Overview](#project-architecture-overview)
3. [Smart Path Filtering](#smart-path-filtering)
4. [Modern Python Tooling with uv](#modern-python-tooling-with-uv)
5. [Linting & Type Checking: Fast Feedback Loops](#linting--type-checking-fast-feedback-loops)
6. [System Dependencies: Caching APT Packages](#system-dependencies-caching-apt-packages)
7. [Schema-Driven Development: The OpenAPI Pattern](#schema-driven-development-the-openapi-pattern)
8. [Concurrency Control: Preventing Duplicate Runs](#concurrency-control-preventing-duplicate-runs)
9. [Artifact Management: Sharing Build Outputs](#artifact-management-sharing-build-outputs)
10. [Testing Strategy: Fast Feedback First](#testing-strategy-fast-feedback-first)
11. [Complete Workflow Example](#complete-workflow-example)
12. [Tips, Tricks & Gotchas](#tips-tricks--gotchas)
13. [Measuring Success: Before & After](#measuring-success-before--after)
14. [Adapting to Your Project](#adapting-to-your-project)
15. [Further Reading](#further-reading)
16. [Appendix: Complete Code Examples](#appendix-complete-code-examples)

---

## Introduction

### The Challenge

If you've ever worked with a monorepo containing both backend and frontend code, you've likely experienced the pain of slow, wasteful CI/CD pipelines. Every small frontend fix triggers the entire backend test suite. Every backend refactor runs frontend builds. Your CI minutes disappear, developers wait longer for feedback, and the cost keeps climbing.

### A Real-World Scenario

Imagine a full-stack application:
- **Backend:** FastAPI application with comprehensive test coverage, type checking, and linting
- **Frontend:** Vue.js application with TypeScript, requiring build-time type generation from the backend API
- **Problem:** Any commit triggers 15 minutes of CI across both stacks, even if only one line changed in a README file

The challenges compound:
- **Wasted Resources:** Running all tests for every change
- **Slow Feedback:** Developers wait 15 minutes to know if their 2-line fix works
- **Cache Misses:** Reinstalling dependencies on every run
- **Race Conditions:** Multiple pushes to the same PR create duplicate, conflicting CI runs
- **Type Drift:** Frontend types fall out of sync with backend API changes

### What You'll Learn

This guide walks through battle-tested patterns for building an efficient, intelligent CI/CD pipeline for FastAPI + Vue.js monorepos. These patterns apply to any full-stack monorepo setup.

**Core Patterns Covered:**
1. **Smart Path Filtering** - Run only relevant jobs based on changed files
2. **Modern Python Tooling** - Leverage `uv` package manager for 10-100x faster dependency management
3. **Schema-Driven Development** - Auto-generate TypeScript types from OpenAPI schemas
4. **Multi-Layer Caching** - Cache Python packages, npm dependencies, and system libraries
5. **Dependency Orchestration** - Coordinate backend schema generation with frontend builds
6. **Concurrency Control** - Prevent duplicate CI runs and wasted resources

### Who This Guide Is For

- **Full-stack developers** building applications with separate backend and frontend codebases
- **DevOps engineers** optimizing CI/CD pipelines for monorepos
- **Teams** looking to reduce CI costs and improve developer experience
- **Anyone** wanting to learn advanced GitHub Actions patterns

### What We're Building

By the end of this guide, you'll have a CI pipeline that:
- Runs in ~5 minutes instead of 15 minutes (for typical changes)
- Only executes relevant jobs (backend changes don't run frontend tests unnecessarily)
- Catches type mismatches between backend and frontend at build time
- Caches aggressively to speed up repeated runs
- Provides clear, actionable error messages
- Scales efficiently as your team and codebase grow

---

## Project Architecture Overview

### Monorepo Structure

Here's the typical structure we're working with:

```
my-app/
├── .github/
│   └── workflows/
│       └── ci.yml                    # Main CI workflow
├── backend/
│   ├── pyproject.toml                # Python dependencies and tool config
│   ├── uv.lock                       # Locked dependencies (reproducibility)
│   ├── src/
│   │   └── app/
│   │       └── main.py               # FastAPI application
│   ├── scripts/
│   │   └── export_openapi_schema.py  # Schema export for type generation
│   └── public/
│       └── openapi.json              # Generated schema (gitignored)
└── frontend/
    ├── package.json                  # npm dependencies and scripts
    ├── package-lock.json             # Locked dependencies
    ├── vite.config.ts                # Vite build configuration
    ├── openapi-ts.config.ts          # Type generation configuration
    ├── src/
    │   ├── generated/                # Auto-generated types (gitignored)
    │   └── components/               # Vue components
    └── scripts/
        └── generate-types.js         # TypeScript generation script
```

### Tech Stack Breakdown

**Backend:**
- **FastAPI** - Modern Python web framework with automatic OpenAPI schema generation
- **Python 3.11+** - Latest Python with improved performance
- **uv** - Ultra-fast package manager (Rust-based, 10-100x faster than pip)
- **Ruff** - Lightning-fast Python linter and formatter (replaces Black, isort, Flake8)
- **mypy** - Static type checker for Python
- **pytest** - Testing framework with async support

**Frontend:**
- **Vue 3** - Progressive JavaScript framework
- **Vite** - Next-generation frontend build tool (fast HMR and builds)
- **TypeScript** - Type-safe JavaScript
- **ESLint** - Linting for JavaScript/TypeScript
- **vue-tsc** - Type-checking for Vue components
- **@hey-api/openapi-ts** - Generates TypeScript types from OpenAPI schemas

### Key Architectural Decision: Schema-Driven Development

The most powerful pattern in this setup is using the backend's OpenAPI schema as the single source of truth for API contracts:

```
Backend Code → OpenAPI Schema → TypeScript Types → Frontend Code
```

**Benefits:**
- Type safety across the entire stack
- Impossible for frontend types to drift from backend reality
- Compile-time errors when API changes break frontend code
- No manual type duplication or synchronization

### Why Separate Backend/Frontend?

You might wonder: why not nest `frontend/` inside `backend/`? Or use a monorepo tool like Nx or Turborepo?

**Reasons for this structure:**
1. **Clear separation of concerns** - Each part has its own dependencies, configs, and workflows
2. **Flexibility** - Can deploy backend and frontend independently
3. **Tooling compatibility** - Backend and frontend tools don't interfere with each other
4. **Simplicity** - No need for complex monorepo orchestration tools
5. **Path-based filtering** - Easy to detect which part of the codebase changed

---

## Smart Path Filtering

### The Problem

Traditional CI workflows run all jobs for every commit:

```yaml
# Naive approach - always runs everything
jobs:
  backend-test:
    runs-on: ubuntu-latest
    steps: [...]

  frontend-test:
    runs-on: ubuntu-latest
    steps: [...]
```

**Result:** Change one line in `frontend/README.md` → backend tests run for 10 minutes → wasted time and money.

### The Solution: dorny/paths-filter

The `dorny/paths-filter` action analyzes which files changed and outputs boolean flags. We use these flags to conditionally run jobs.

**Complete Example:**

```yaml
jobs:
  # Step 1: Detect which parts of the codebase changed
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.filter.outputs.backend }}
      frontend: ${{ steps.filter.outputs.frontend }}
      workflow: ${{ steps.filter.outputs.workflow }}
    steps:
      - uses: actions/checkout@v4

      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            backend:
              - 'backend/**'
            frontend:
              - 'frontend/**'
            workflow:
              - '.github/workflows/**'

  # Step 2: Run backend jobs only if backend or workflow files changed
  backend-lint:
    needs: changes
    if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # ... linting steps ...

  # Step 3: Run frontend jobs only if frontend or workflow files changed
  frontend-lint:
    needs: changes
    if: needs.changes.outputs.frontend == 'true' || needs.changes.outputs.workflow == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # ... linting steps ...
```

### How It Works

**1. The `changes` Job**

This job runs quickly (< 5 seconds) and outputs which areas changed:

```yaml
changes:
  runs-on: ubuntu-latest
  outputs:
    backend: ${{ steps.filter.outputs.backend }}   # 'true' or 'false'
    frontend: ${{ steps.filter.outputs.frontend }} # 'true' or 'false'
    workflow: ${{ steps.filter.outputs.workflow }} # 'true' or 'false'
```

**2. Path Filters**

Define glob patterns for each area:

```yaml
filters: |
  backend:
    - 'backend/**'           # Any file in backend/ directory
  frontend:
    - 'frontend/**'          # Any file in frontend/ directory
  workflow:
    - '.github/workflows/**' # Any workflow file change
```

**3. Conditional Job Execution**

Use `needs` and `if` to create dependencies:

```yaml
backend-test:
  needs: changes  # Wait for changes job to complete
  if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
```

### Why Include Workflow Changes?

Notice we check for `workflow == 'true'` in every condition:

```yaml
if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
```

**Reason:** When you modify the CI workflow itself, you want to test it against all jobs to ensure it still works. This prevents you from breaking CI accidentally.

### Real-World Impact

**Before path filtering:**
- Frontend-only change: Runs backend + frontend jobs = 15 minutes
- Backend-only change: Runs backend + frontend jobs = 15 minutes

**After path filtering:**
- Frontend-only change: Runs frontend jobs only = 5 minutes (67% faster)
- Backend-only change: Runs backend jobs only = 8 minutes (47% faster)
- Workflow change: Runs all jobs = 15 minutes (testing the changes)

**Monthly savings for a team with 100 commits:**
```
Average savings: 7 minutes per commit
100 commits × 7 minutes = 700 minutes/month saved
```

### String Comparison Gotcha

> **Important:** Outputs are always strings, not booleans:

```yaml
# Correct
if: needs.changes.outputs.backend == 'true'

# Wrong - will always be false
if: needs.changes.outputs.backend == true

# Wrong - will always be truthy (non-empty string)
if: needs.changes.outputs.backend
```

---

## Modern Python Tooling with uv

### Why uv Instead of pip/poetry/pipenv?

`uv` is a modern Python package manager written in Rust that's 10-100x faster than traditional tools:

**Comparison:**

| Tool | Install 100 packages | Lock dependencies | Written in |
|------|---------------------|-------------------|------------|
| pip | ~60 seconds | N/A (no lock file) | Python |
| poetry | ~45 seconds | ~30 seconds | Python |
| **uv** | **~2 seconds** | **~1 second** | Rust |

**Key Benefits:**
- **Speed:** Parallel downloads and Rust-based resolution
- **Lock files:** `uv.lock` ensures reproducible installs
- **Virtual environments:** Built-in venv management
- **Compatible:** Works with existing `pyproject.toml` files
- **GitHub Actions integration:** Official `setup-uv` action with caching

### Installation & Caching in CI

```yaml
- name: Install uv
  uses: astral-sh/setup-uv@v5
  with:
    enable-cache: true
    cache-dependency-glob: backend/uv.lock
```

**What this does:**
1. Installs `uv` CLI tool
2. Enables automatic caching using GitHub Actions cache
3. Creates cache key based on `backend/uv.lock` content
4. Restores cached packages if lock file hasn't changed

**Cache Hit Performance:**
- First run: ~15 seconds (download and install packages)
- Cached run: ~2 seconds (restore from cache)

### Running Commands with uv

**1. Installing Dependencies**

```yaml
- name: Install dependencies
  run: uv sync
  working-directory: backend
```

`uv sync`:
- Reads `pyproject.toml` and `uv.lock`
- Creates virtual environment
- Installs exact versions from lock file
- Fast and reproducible

**2. Running Tools Without Installation (uvx)**

```yaml
- name: "Ruff lint (fix: cd backend && uvx ruff check --fix)"
  run: uvx ruff check
  working-directory: backend
```

`uvx` (like `npx` for Python):
- Runs tools in isolated environments
- No need to `uv sync` first
- Always uses compatible version
- Perfect for linters and formatters

**3. Running Commands in Virtual Environment (uv run)**

```yaml
- name: Run tests
  run: uv run pytest
  working-directory: backend
```

`uv run`:
- Executes command in the project's virtual environment
- Ensures correct Python and dependencies
- Use for application code and tests

### Pattern Comparison

```yaml
# Linting: Use uvx (no dependencies needed)
- run: uvx ruff check

# Type checking: Use uv run (needs installed dependencies for stubs)
- run: uv run mypy src

# Tests: Use uv run (needs application code and dependencies)
- run: uv run pytest

# Application: Use uv run
- run: uv run uvicorn app.main:app
```

### Developer-Friendly Error Messages

Include fix commands in step names:

```yaml
- name: "Ruff lint (fix: cd backend && uvx ruff check --fix)"
  run: uvx ruff check
  working-directory: backend

- name: "Ruff format (fix: cd backend && uvx ruff format)"
  run: uvx ruff format --check
  working-directory: backend
```

When CI fails, developers see exactly how to fix it locally:

```
Ruff lint (fix: cd backend && uvx ruff check --fix)
   Error: Found 3 linting errors
```

Developer copies the fix command: `cd backend && uvx ruff check --fix`

### Configuration in pyproject.toml

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-app"
version = "1.0.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.113.0",
    "uvicorn>=0.34.2",
    "pydantic>=2.11.7",
]

[dependency-groups]
dev = [
    "pytest>=7.0.0",
    "pytest-asyncio>=0.23.5",
    "ruff>=0.1.0",
    "mypy>=1.0.0",
]

[tool.ruff]
line-length = 120
target-version = "py311"

[tool.ruff.lint]
select = [
    "E",  # pycodestyle
    "F",  # Pyflakes
    "W",  # warning
    "I",  # isort
]
ignore = [
    "E501",  # Line too long (handled by formatter)
]

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
check_untyped_defs = true
```

---

## Linting & Type Checking: Fast Feedback Loops

### The Strategy: Separate Parallel Jobs

Instead of one monolithic "test everything" job, split into focused parallel jobs:

```
┌─────────────┐
│  changes    │
└──────┬──────┘
       │
   ┌───┴────────────────────────┐
   │                            │
   ▼                            ▼
┌──────────────┐        ┌──────────────┐
│ backend-lint │        │ frontend-lint│
└──────────────┘        └──────────────┘
   ▼                            ▼
┌──────────────┐        ┌──────────────┐
│backend-      │        │frontend-     │
│typecheck     │        │build         │
└──────────────┘        └──────────────┘
   ▼
┌──────────────┐
│ backend-test │
└──────────────┘
```

**Why separate jobs?**
1. **Parallel execution** - Multiple jobs run simultaneously on different runners
2. **Isolated failures** - Linting failure doesn't block type checking
3. **Clear attribution** - Immediately see which check failed
4. **Faster feedback** - Get lint errors in 30 seconds, not after 10-minute test suite

### Backend Linting with Ruff

```yaml
backend-lint:
  needs: changes
  if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Install uv
      uses: astral-sh/setup-uv@v5
      with:
        enable-cache: true
        cache-dependency-glob: backend/uv.lock

    - name: "Ruff lint (fix: cd backend && uvx ruff check --fix)"
      run: uvx ruff check
      working-directory: backend

    - name: "Ruff format (fix: cd backend && uvx ruff format)"
      run: uvx ruff format --check
      working-directory: backend
```

**Key points:**
- No `uv sync` needed - `uvx` handles tool installation
- Two separate steps: linting and formatting
- Each step shows fix command in the name

**Typical runtime:** 15-30 seconds

### Backend Type Checking with mypy

```yaml
backend-typecheck:
  needs: changes
  if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Install uv
      uses: astral-sh/setup-uv@v5
      with:
        enable-cache: true
        cache-dependency-glob: backend/uv.lock

    - name: Install dependencies
      run: uv sync
      working-directory: backend

    - name: "Mypy (fix: cd backend && uv run mypy src)"
      run: uv run mypy src
      working-directory: backend
```

**Why mypy needs `uv sync`:**
- Type checkers need type stubs from dependencies
- Can't use `uvx` for type checking (needs the actual packages)
- Use `uv run` to execute in the virtual environment

**Typical runtime:** 45-90 seconds (first run), 10-20 seconds (cached)

### Frontend Linting with ESLint

```yaml
frontend-lint:
  needs: changes
  if: needs.changes.outputs.frontend == 'true' || needs.changes.outputs.workflow == 'true'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json

    - name: Install dependencies
      run: npm ci
      working-directory: frontend

    - name: "ESLint (fix: cd frontend && npm run lint:fix)"
      run: npm run lint
      working-directory: frontend
```

**Key points:**
- `cache: 'npm'` enables automatic npm caching
- `cache-dependency-path` points to lock file
- `npm ci` (not `npm install`) for reproducible CI builds

### npm ci vs npm install

| Command | Use Case | Behavior |
|---------|----------|----------|
| `npm install` | Local development | Updates package-lock.json, installs latest compatible versions |
| `npm ci` | CI/CD | Deletes node_modules, installs exact versions from lock file, fails if lock file is out of sync |

**Always use `npm ci` in CI:**
- Faster (skips dependency resolution)
- Reproducible (exact versions)
- Catches lock file issues

---

## System Dependencies: Caching APT Packages

### The Problem

Some applications require system libraries that aren't available in standard GitHub Actions runners:
- Qt/GTK for GUI applications
- OpenGL/EGL for graphics processing
- Database client libraries (PostgreSQL, MySQL)
- Media processing tools (ffmpeg, ImageMagick)
- Scientific computing libraries (BLAS, LAPACK)

Installing with `apt-get` on every run is slow:

```yaml
# Slow: Runs on every build
- run: |
    sudo apt-get update
    sudo apt-get install -y libegl1 libgl1 libxkbcommon0 libdbus-1-3
```

**Time:** 30-90 seconds per run

### The Solution: cache-apt-pkgs-action

```yaml
- name: Cache system dependencies
  uses: awalsh128/cache-apt-pkgs-action@v1
  with:
    packages: libegl1 libgl1 libxkbcommon0 libdbus-1-3
```

**What this does:**
1. First run: Installs packages via apt-get and caches them
2. Subsequent runs: Restores from cache (2-5 seconds)
3. Cache key: Based on package list and runner OS
4. Automatic invalidation: Cache updates when package list changes

### Performance Impact

| Scenario | Without Cache | With Cache | Speedup |
|----------|---------------|------------|---------|
| First run | 45 seconds | 45 seconds (builds cache) | - |
| Subsequent runs | 45 seconds | 3 seconds | **15x faster** |

### Real-World Example

```yaml
backend-typecheck:
  needs: changes
  if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    # Cache system libraries needed for Python packages
    - name: Cache system dependencies
      uses: awalsh128/cache-apt-pkgs-action@v1
      with:
        packages: libegl1 libgl1 libxkbcommon0 libdbus-1-3

    - name: Install uv
      uses: astral-sh/setup-uv@v5
      with:
        enable-cache: true
        cache-dependency-glob: backend/uv.lock

    - name: Install dependencies
      run: uv sync
      working-directory: backend

    - name: "Mypy (fix: cd backend && uv run mypy src)"
      run: uv run mypy src
      working-directory: backend
```

### When to Use This Pattern

**Good use cases:**
- Projects using GUI frameworks (PySide6, PyQt, Tkinter)
- Browser automation (Playwright, Puppeteer)
- Image/video processing (OpenCV, Pillow with system codecs)
- Database clients requiring system libraries
- Scientific computing with native dependencies

**Don't use if:**
- You don't need system packages (pure Python/JavaScript projects)
- The packages are already installed on GitHub runners
- You're using Docker (install in Dockerfile instead)

---

## Schema-Driven Development: The OpenAPI Pattern

This is the most powerful pattern in the entire workflow. By generating frontend TypeScript types from the backend's OpenAPI schema, you achieve **compile-time type safety across the entire stack**.

### The Big Idea

```
Backend FastAPI Code
    ↓
OpenAPI Schema (JSON)
    ↓
TypeScript Types + API Client
    ↓
Frontend Vue/React Code
```

**Benefits:**
- Frontend and backend types can never drift out of sync
- API changes cause frontend build failures (catches breaking changes early)
- Autocomplete for API requests in frontend code
- Type-safe request/response handling
- No manual type duplication

### Step 1: Export OpenAPI Schema (Backend)

**File: `backend/scripts/export_openapi_schema.py`**

```python
#!/usr/bin/env python3
"""
Export OpenAPI schema from FastAPI application without starting the server.
This script generates the openapi.json file for frontend type generation.
"""

import json
import sys
from pathlib import Path

BACKEND_DIR = Path(__file__).parent.parent

def export_openapi_schema(output_path: Path) -> None:
    """Export the OpenAPI schema to a JSON file."""
    # Add backend src to path
    backend_src = BACKEND_DIR / "src"
    sys.path.insert(0, str(backend_src))

    # Import FastAPI app
    from app.main import app

    # Get OpenAPI schema from FastAPI
    openapi_schema = app.openapi()

    # Write to file
    output_path.parent.mkdir(parents=True, exist_ok=True)
    with open(output_path, "w", encoding="utf-8") as f:
        json.dump(openapi_schema, f, indent=2)

    print(f"OpenAPI schema exported to: {output_path}")

if __name__ == "__main__":
    output_path = BACKEND_DIR / "public" / "openapi.json"
    export_openapi_schema(output_path)
```

**Why this approach?**
- No need to start the server (works in CI without networking)
- Fast execution (< 1 second)
- Deterministic output (same schema every time)
- Can run offline

**Add to .gitignore:**
```gitignore
# backend/.gitignore
public/openapi.json
```

The schema is generated, not committed.

### Step 2: Configure Type Generation (Frontend)

**File: `frontend/openapi-ts.config.ts`**

```typescript
import { defineConfig } from '@hey-api/openapi-ts'

export default defineConfig({
  input: 'http://backend:8000/openapi.json',  // For local dev
  output: './src/generated',
  plugins: [
    {
      name: '@hey-api/client-fetch',
      throwOnError: true,
    },
    {
      enums: 'typescript',  // Generate TypeScript enums (runtime values)
      name: '@hey-api/typescript',
    },
    {
      name: '@hey-api/sdk',
      asClass: true,
      responseStyle: 'data',
    },
  ],
})
```

**What gets generated:**
- `src/generated/types.gen.ts` - TypeScript interfaces for all models
- `src/generated/client.gen.ts` - Fetch-based API client
- `src/generated/sdk.gen.ts` - Class-based SDK (optional)

### Step 3: Generate Types Script (Frontend)

**File: `frontend/scripts/generate-types.js`**

```javascript
#!/usr/bin/env node
/**
 * Generate TypeScript types from OpenAPI schema
 *
 * Usage:
 *   npm run generate-types              # Use live backend
 *   node scripts/generate-types.js 1    # Use local schema file
 */

import { execSync } from 'child_process'
import fs from 'fs'
import path from 'path'
import { fileURLToPath } from 'url'

const __filename = fileURLToPath(import.meta.url)
const __dirname = path.dirname(__filename)

const BACKEND_DIR = path.join(__dirname, '../../backend/')
const USE_LOCAL_SCHEMA = process.argv[2]
const LOCAL_SCHEMA_PATH = path.join(BACKEND_DIR, 'public/openapi.json')

console.log('Generating TypeScript types from OpenAPI schema...')

if (USE_LOCAL_SCHEMA) {
    if (!fs.existsSync(LOCAL_SCHEMA_PATH)) {
        console.error(`Schema file not found: ${LOCAL_SCHEMA_PATH}`)
        console.error('Run: cd backend && uv run python scripts/export_openapi_schema.py')
        process.exit(1)
    }
    console.log(`Using local schema: ${LOCAL_SCHEMA_PATH}`)

    // Generate with explicit input path
    execSync(
        `npx @hey-api/openapi-ts --input ${LOCAL_SCHEMA_PATH} --output ./src/generated`,
        { stdio: 'inherit' }
    )
} else {
    // Use config file (default to live backend)
    execSync('npx @hey-api/openapi-ts', { stdio: 'inherit' })
}

console.log('Type generation complete!')
```

**Add to .gitignore:**
```gitignore
# frontend/.gitignore
src/generated
```

Types are always generated, never committed.

### Step 4: Orchestrate in CI

This is the critical part - the frontend build **must** depend on backend schema generation:

```yaml
frontend-build:
  needs: changes
  if: |
    needs.changes.outputs.frontend == 'true' ||
    needs.changes.outputs.backend == 'true' ||   # ← Key: backend changes affect frontend!
    needs.changes.outputs.workflow == 'true'
  runs-on: ubuntu-latest
  steps:
    # 1. Install backend dependencies
    - name: Install uv
      uses: astral-sh/setup-uv@v5
      with:
        enable-cache: true
        cache-dependency-glob: backend/uv.lock

    - name: Install backend dependencies
      run: uv sync
      working-directory: backend

    # 2. Export OpenAPI schema
    - name: Export OpenAPI schema
      run: uv run python scripts/export_openapi_schema.py
      working-directory: backend

    # 3. Install frontend dependencies
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json

    - name: Install frontend dependencies
      run: npm ci
      working-directory: frontend

    # 4. Generate TypeScript types from schema
    - name: Generate TypeScript types
      run: node scripts/generate-types.js true  # Use local schema
      working-directory: frontend

    # 5. Build frontend (with type checking)
    - name: "Type check and build"
      run: npm run build  # Runs: vue-tsc --build && vite build
      working-directory: frontend
```

**Critical Pattern: Cross-Project Dependencies**

Notice the condition:

```yaml
if: |
  needs.changes.outputs.frontend == 'true' ||
  needs.changes.outputs.backend == 'true' ||   # ← Backend changes trigger frontend build!
  needs.changes.outputs.workflow == 'true'
```

**Why?** Because backend API changes can break frontend types. This catches breaking changes before deployment.

### Example: Catching Breaking Changes

**Scenario:** You change a backend API endpoint:

```python
# Before
class UserResponse(BaseModel):
    id: int
    name: str
    email: str

# After (remove email field)
class UserResponse(BaseModel):
    id: int
    name: str
```

**Without schema-driven development:**
- Backend deploys successfully
- Frontend code still references `user.email`
- Runtime error in production

**With schema-driven development:**
- Backend changes → OpenAPI schema changes
- TypeScript type generation updates types
- Frontend build fails with error: "Property 'email' does not exist on type 'UserResponse'"
- Breaking change caught before merge

### Developer Workflow

**Local development:**

```bash
# Terminal 1: Backend
cd backend
uv run python scripts/export_openapi_schema.py
uv run uvicorn app.main:app --reload

# Terminal 2: Frontend
cd frontend
npm run generate-types
npm run dev
```

**package.json scripts:**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --build && vite build",
    "generate-types": "node scripts/generate-types.js",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts",
    "lint:fix": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix"
  }
}
```

---

## Concurrency Control: Preventing Duplicate Runs

### The Problem

Multiple pushes to the same PR trigger multiple CI runs:

```
Developer pushes commit A → CI run #1 starts
Developer pushes commit B → CI run #2 starts
Developer pushes commit C → CI run #3 starts

Result: 3 CI runs active, only #3 matters
Waste: Run #1 and #2 consume resources unnecessarily
```

**Costs:**
- Wasted CI minutes
- Confusing notification spam
- Slower feedback (runners are busy with old runs)

### The Solution: Concurrency Groups

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true
```

**What this does:**
1. Creates a unique concurrency group per PR or branch
2. When a new run starts, cancels any in-progress runs in the same group
3. Saves CI minutes and reduces noise

### Understanding the Pattern

```yaml
group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
```

**Breakdown:**
- `github.workflow`: Workflow file name (e.g., "CI")
- `github.event.pull_request.number`: PR number if triggered by PR (e.g., "123")
- `github.ref`: Git ref for direct pushes (e.g., "refs/heads/main")
- `||`: Fallback operator (use PR number if available, else use ref)

**Examples:**

| Trigger | Group Name | Behavior |
|---------|------------|----------|
| PR #42 opened | `CI-42` | Multiple pushes to PR #42 cancel each other |
| Push to `main` | `CI-refs/heads/main` | Pushes to main cancel previous main builds |
| Push to `feature/api` | `CI-refs/heads/feature/api` | Branch-specific cancellation |

### When NOT to Use Cancel-in-Progress

Some workflows should never cancel:

```yaml
# Don't cancel deployment workflows
deploy-production:
  concurrency:
    group: production-deploy
    cancel-in-progress: false  # Let deploys finish

# Don't cancel release builds
create-release:
  concurrency:
    group: release-${{ github.ref }}
    cancel-in-progress: false  # Complete all release steps
```

### Advanced Pattern: Conditional Cancellation

Cancel on feature branches, but not on main:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}
```

**Result:**
- Feature branches: Aggressive cancellation for fast feedback
- Main branch: All runs complete (for deployment/metrics)

---

## Artifact Management: Sharing Build Outputs

### Use Cases

**Build artifacts** are files generated during CI that you want to preserve:
- Compiled frontend builds (for deployment)
- Test coverage reports
- Build logs for debugging
- Screenshots from E2E tests
- Binary releases

### Basic Pattern

```yaml
- name: Upload build artifact
  uses: actions/upload-artifact@v4
  with:
    name: frontend-dist
    path: frontend/dist/
    retention-days: 7
```

**Parameters:**
- `name`: Unique artifact identifier
- `path`: File or directory to upload
- `retention-days`: How long to keep (max 90 days)

**Why 7 days?** Balance between debugging needs and storage costs.

### Complete Example

```yaml
frontend-build:
  needs: changes
  if: needs.changes.outputs.frontend == 'true' || needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
  runs-on: ubuntu-latest
  steps:
    # ... build steps ...

    - name: "Type check and build"
      run: npm run build
      working-directory: frontend

    - name: Upload build artifact
      uses: actions/upload-artifact@v4
      with:
        name: frontend-dist
        path: frontend/dist/
        retention-days: 7
```

### Downloading Artifacts in Later Jobs

```yaml
deploy:
  needs: frontend-build
  runs-on: ubuntu-latest
  steps:
    - name: Download build artifact
      uses: actions/download-artifact@v4
      with:
        name: frontend-dist
        path: dist/

    - name: Deploy to server
      run: |
        # dist/ contains the frontend build
        rsync -avz dist/ server:/var/www/app/
```

### Conditional Upload: Debug on Failure

Upload test artifacts only when tests fail:

```yaml
- name: Run tests
  run: uv run pytest --junit-xml=test-results/junit.xml
  working-directory: backend

- name: Upload test results on failure
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: test-failures-${{ github.run_id }}
    path: |
      backend/test-results/
      backend/.coverage
    retention-days: 14
```

**Benefits:**
- Saves storage (only uploads when needed)
- Unique naming with `${{ github.run_id }}`
- Longer retention for debugging (14 days)

---

## Testing Strategy: Fast Feedback First

### The Philosophy: Job Ordering Matters

**Optimal order:**
1. **Linting** (fastest, catches common issues) - 15-30 seconds
2. **Type checking** (fast, catches type errors) - 30-60 seconds
3. **Tests** (slower, comprehensive validation) - 2-5 minutes
4. **Build** (slowest, final validation) - 3-7 minutes

**Why this order?**
- Developers get fast feedback on simple mistakes
- Expensive operations (tests, builds) only run if linting/typing passes
- Parallel execution means fastest jobs complete first

### Backend Testing

```yaml
backend-test:
  needs: changes
  if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Cache system dependencies
      uses: awalsh128/cache-apt-pkgs-action@v1
      with:
        packages: libegl1 libgl1 libxkbcommon0 libdbus-1-3

    - name: Install uv
      uses: astral-sh/setup-uv@v5
      with:
        enable-cache: true
        cache-dependency-glob: backend/uv.lock

    - name: Install dependencies
      run: uv sync
      working-directory: backend

    - name: Run tests
      run: uv run pytest
      working-directory: backend
```

### Parallel vs Sequential Execution

**These jobs run in parallel (no dependencies):**

```yaml
jobs:
  changes:
    # Runs first, outputs filters

  # All of these run simultaneously after changes completes:
  backend-lint:
    needs: changes

  backend-typecheck:
    needs: changes

  backend-test:
    needs: changes

  frontend-lint:
    needs: changes

  frontend-build:
    needs: changes
```

**Visualization:**

```
changes (5s)
    ↓
┌───┴────────┬───────────┬──────────┬─────────────┐
│            │           │          │             │
backend-     backend-    backend-   frontend-    frontend-
lint (20s)   typecheck   test       lint (25s)   build
             (45s)       (3m)                     (4m)
│            │           │          │             │
└────────────┴───────────┴──────────┴─────────────┘
                Total: 4 minutes
```

**Without parallelization:** 20s + 45s + 3m + 25s + 4m = **~8.5 minutes**

**With parallelization:** max(20s, 45s, 3m, 25s, 4m) = **~4 minutes**

---

## Complete Workflow Example

Here's the full anonymized workflow with all patterns combined:

```yaml
name: CI

on:
  push:
    branches: [master, dev]
  pull_request:
    branches: [master, dev]

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.filter.outputs.backend }}
      frontend: ${{ steps.filter.outputs.frontend }}
      workflow: ${{ steps.filter.outputs.workflow }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            backend:
              - 'backend/**'
            frontend:
              - 'frontend/**'
            workflow:
              - '.github/workflows/**'

  backend-lint:
    needs: changes
    if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
          cache-dependency-glob: backend/uv.lock

      - name: "Ruff lint (fix: cd backend && uvx ruff check --fix)"
        run: uvx ruff check
        working-directory: backend

      - name: "Ruff format (fix: cd backend && uvx ruff format)"
        run: uvx ruff format --check
        working-directory: backend

  backend-typecheck:
    needs: changes
    if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache system dependencies
        uses: awalsh128/cache-apt-pkgs-action@v1
        with:
          packages: libegl1 libgl1 libxkbcommon0 libdbus-1-3

      - name: Install uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
          cache-dependency-glob: backend/uv.lock

      - name: Install dependencies
        run: uv sync
        working-directory: backend

      - name: "Mypy (fix: cd backend && uv run mypy src)"
        run: uv run mypy src
        working-directory: backend

  backend-test:
    needs: changes
    if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache system dependencies
        uses: awalsh128/cache-apt-pkgs-action@v1
        with:
          packages: libegl1 libgl1 libxkbcommon0 libdbus-1-3

      - name: Install uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
          cache-dependency-glob: backend/uv.lock

      - name: Install dependencies
        run: uv sync
        working-directory: backend

      - name: Run tests
        run: uv run pytest
        working-directory: backend

  frontend-lint:
    needs: changes
    if: needs.changes.outputs.frontend == 'true' || needs.changes.outputs.workflow == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci
        working-directory: frontend

      - name: "ESLint (fix: cd frontend && npm run lint:fix)"
        run: npm run lint
        working-directory: frontend

  frontend-build:
    needs: changes
    if: needs.changes.outputs.frontend == 'true' || needs.changes.outputs.backend == 'true' || needs.changes.outputs.workflow == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache system dependencies
        uses: awalsh128/cache-apt-pkgs-action@v1
        with:
          packages: libegl1 libgl1 libxkbcommon0 libdbus-1-3

      - name: Install uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
          cache-dependency-glob: backend/uv.lock

      - name: Install backend dependencies
        run: uv sync
        working-directory: backend

      - name: Export OpenAPI schema
        run: uv run python scripts/export_openapi_schema.py
        working-directory: backend

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install frontend dependencies
        run: npm ci
        working-directory: frontend

      - name: Generate TypeScript types
        run: node scripts/generate-types.js true
        working-directory: frontend

      - name: "Type check and build (fix: cd frontend && npm run build)"
        run: npm run build
        working-directory: frontend

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: frontend-dist
          path: frontend/dist/
          retention-days: 7
```

---

## Tips, Tricks & Gotchas

### 1. working-directory vs cd

**Recommended:**
```yaml
- run: npm run build
  working-directory: frontend
```

**Avoid:**
```yaml
- run: cd frontend && npm run build
```

**Why?** `working-directory` is explicit, easier to read, and less error-prone.

### 2. Include Fix Commands in Step Names

**Good:**
```yaml
- name: "ESLint (fix: cd frontend && npm run lint:fix)"
  run: npm run lint
  working-directory: frontend
```

**Result:** When CI fails, developers immediately see how to fix it locally.

### 3. String Comparison for Outputs

**Correct:**
```yaml
if: needs.changes.outputs.backend == 'true'
```

**Wrong:**
```yaml
if: needs.changes.outputs.backend == true  # Always false
if: needs.changes.outputs.backend          # Always truthy
```

Outputs are strings, not booleans!

### 4. Cache Keys

Each cache needs a unique, stable key:

```yaml
# uv cache - key based on lock file
- uses: astral-sh/setup-uv@v5
  with:
    enable-cache: true
    cache-dependency-glob: backend/uv.lock

# npm cache - key based on lock file
- uses: actions/setup-node@v4
  with:
    cache: 'npm'
    cache-dependency-path: frontend/package-lock.json

# APT cache - key based on package list
- uses: awalsh128/cache-apt-pkgs-action@v1
  with:
    packages: libegl1 libgl1 libxkbcommon0 libdbus-1-3
```

### 5. Path Filter Edge Cases

Always include workflow changes in filters:

```yaml
workflow:
  - '.github/workflows/**'
```

**Why?** Changing CI configuration should trigger all jobs to test the changes.

### 6. npm ci vs npm install

In CI, always use `npm ci`:

```yaml
# CI: Fast, reproducible, validates lock file
- run: npm ci

# CI: Slow, can modify lock file, less reproducible
- run: npm install
```

### 7. Debugging CI Issues

Add debug output when needed:

```yaml
- name: Debug outputs
  run: |
    echo "Backend changed: ${{ needs.changes.outputs.backend }}"
    echo "Frontend changed: ${{ needs.changes.outputs.frontend }}"
    echo "Workflow changed: ${{ needs.changes.outputs.workflow }}"
```

### 8. Cost Optimization Tips

- Use path filtering (avoid unnecessary job runs)
- Cancel in-progress runs (don't waste CI minutes on outdated code)
- Cache aggressively (faster runs = fewer runner minutes)
- Use `ubuntu-latest` (cheapest GitHub-hosted runner)
- Optimize job ordering (fail fast on linting before expensive tests)

### 9. Security Considerations

```yaml
# Use minimal tokens
- uses: actions/checkout@v4
  with:
    token: ${{ secrets.GITHUB_TOKEN }}

# Never log secrets
- run: echo ${{ secrets.API_KEY }}  # DANGEROUS!

# Pin action versions for security
- uses: dorny/paths-filter@v3  # or use full SHA
```

### 10. Matrix Testing (Advanced)

Test across multiple Python or Node versions:

```yaml
backend-test:
  strategy:
    matrix:
      python-version: ['3.11', '3.12']
  steps:
    - uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
```

---

## Measuring Success: Before & After

### Metrics to Track

**Before Optimization:**
```
Average CI time: 15 minutes
Jobs per commit: 6 jobs
Total runtime: 90 job-minutes per commit
CI runs per day: 50 commits
Monthly cost: 135,000 CI minutes
Developer wait time: High frustration
Failed run attribution: Unclear which check failed
```

**After Optimization:**
```
Average CI time: 5 minutes (path filtering)
Jobs per commit: 3 jobs (average, due to filtering)
Total runtime: 15 job-minutes per commit
CI runs per day: 30 effective runs (canceled duplicates)
Monthly cost: 13,500 CI minutes
Developer wait time: Low - fast feedback on lint errors
Failed run attribution: Clear - separate jobs for each concern
```

### Cost Analysis

**Monthly savings:**
```
Before: 135,000 minutes/month
After:  13,500 minutes/month
Savings: 121,500 minutes/month (90% reduction)

GitHub Actions pricing: ~$0.008 per minute
Monthly savings: ~$972
Annual savings: ~$11,664
```

**Developer productivity:**
```
Before: 15 min wait × 50 commits/day = 750 developer-minutes/day
After:  5 min wait × 50 commits/day = 250 developer-minutes/day
Saved: 500 developer-minutes/day = ~8.3 developer-hours/day

For a 10-person team: Almost 1 full day of productivity regained per day
```

### Developer Experience Improvements

| Aspect | Before | After |
|--------|--------|-------|
| Lint feedback | 15 minutes | 30 seconds |
| Type error feedback | 15 minutes | 1 minute |
| Test feedback | 15 minutes | 5 minutes |
| Build feedback | 15 minutes | 7 minutes |
| Noise from duplicate runs | High | Low (auto-canceled) |
| Clear error attribution | No - monolithic job | Yes - separate jobs |
| Fix guidance | Generic error messages | Fix commands in step names |

---

## Adapting to Your Project

### Implementation Checklist

**Phase 1: Path Filtering (Biggest Impact)**
- [ ] Identify logical boundaries in your monorepo
- [ ] Add `dorny/paths-filter` to workflow
- [ ] Define filter patterns for each area
- [ ] Add conditional execution to existing jobs
- [ ] Test with commits to different areas

**Phase 2: Caching**
- [ ] Enable uv cache (or poetry/pip cache)
- [ ] Enable npm/yarn/pnpm cache
- [ ] Cache system dependencies (if needed)
- [ ] Monitor cache hit rates in Actions logs

**Phase 3: Schema Generation (if applicable)**
- [ ] Create OpenAPI export script
- [ ] Configure TypeScript generation tool
- [ ] Update frontend build to generate types
- [ ] Make frontend-build depend on backend changes
- [ ] Add generated files to .gitignore

**Phase 4: Optimization**
- [ ] Add concurrency control
- [ ] Split monolithic jobs into focused parallel jobs
- [ ] Add artifact uploads for build outputs
- [ ] Include fix commands in step names
- [ ] Measure CI time improvements

### Common Variations

**Different Python Dependency Managers:**

```yaml
# Poetry instead of uv
- uses: actions/setup-python@v4
  with:
    python-version: '3.11'
    cache: 'poetry'
- run: poetry install

# pip with requirements.txt
- uses: actions/setup-python@v4
  with:
    python-version: '3.11'
    cache: 'pip'
- run: pip install -r requirements.txt
```

**Different Frontend Frameworks:**

```yaml
# React instead of Vue
- run: npm run build  # Creates build/ instead of dist/

# Next.js
- run: npm run build  # Creates .next/ directory

# SvelteKit
- run: npm run build  # Creates .svelte-kit/output/
```

The pattern stays the same: Schema → Types → Build

**Different Monorepo Structures:**

**Multiple Backends:**
```
my-app/
├── services/
│   ├── api/
│   ├── workers/
│   └── admin/
└── frontend/
```

**Multiple Frontends:**
```
my-app/
├── backend/
└── apps/
    ├── web/
    ├── mobile/
    └── admin-dashboard/
```

**Adapt path filters:**
```yaml
filters: |
  api:
    - 'services/api/**'
  workers:
    - 'services/workers/**'
  web:
    - 'apps/web/**'
  mobile:
    - 'apps/mobile/**'
```

---

## Further Reading

### Official Documentation

- [GitHub Actions Documentation](https://docs.github.com/en/actions) - Complete reference
- [dorny/paths-filter](https://github.com/dorny/paths-filter) - Path filtering action
- [astral-sh/setup-uv](https://github.com/astral-sh/setup-uv) - uv installation action
- [uv Documentation](https://docs.astral.sh/uv/) - Modern Python package manager
- [@hey-api/openapi-ts](https://heyapi.vercel.app/) - TypeScript type generation
- [FastAPI Documentation](https://fastapi.tiangolo.com/) - Backend framework
- [Vite Documentation](https://vitejs.dev/) - Frontend build tool

### Related Topics

**Advanced CI/CD Patterns:**
- Semantic versioning and automated releases (semantic-release)
- Preview deployments for pull requests (Vercel, Netlify)
- Database migration testing in CI
- End-to-end testing with Playwright/Cypress
- Performance regression testing

**Monorepo Tools:**
- Turborepo - High-performance build system
- Nx - Smart monorepo tooling
- Bazel - Google's build system for massive codebases

**Type Safety:**
- tRPC - End-to-end typesafe APIs
- GraphQL Code Generator - Generate types from GraphQL schemas
- Prisma - Type-safe database client with codegen

### Community Resources

- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions) - Discover community actions
- [GitHub Community Forum](https://github.com/orgs/community/discussions) - Ask questions and share solutions
- Stack Overflow tags: `github-actions`, `monorepo`, `fastapi`, `vue.js`

---

## Appendix: Complete Code Examples

### A. pyproject.toml (Backend Configuration)

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-app-backend"
version = "1.0.0"
description = "FastAPI backend application"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.113.0",
    "uvicorn>=0.34.2",
    "pydantic>=2.11.7",
]

[dependency-groups]
dev = [
    "pytest>=7.0.0",
    "pytest-asyncio>=0.23.5",
    "ruff>=0.1.0",
    "mypy>=1.0.0",
]

[tool.ruff]
line-length = 120
target-version = "py311"

[tool.ruff.lint]
select = [
    "E",  # pycodestyle errors
    "F",  # Pyflakes
    "W",  # pycodestyle warnings
    "I",  # isort
]
ignore = [
    "E501",  # Line too long (handled by formatter)
]

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
check_untyped_defs = true
disallow_untyped_decorators = false
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
strict_equality = true
show_error_codes = true
```

### B. package.json (Frontend Configuration)

```json
{
  "name": "my-app-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --build && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts",
    "lint:fix": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix",
    "generate-types": "node scripts/generate-types.js"
  },
  "dependencies": {
    "vue": "^3.5.13",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "@hey-api/openapi-ts": "^0.87.5",
    "@vitejs/plugin-vue": "^6.2.0",
    "eslint": "^9.39.2",
    "typescript": "^5.7.2",
    "vite": "^6.2.0",
    "vue-tsc": "^2.0.0"
  }
}
```

### C. vite.config.ts (Frontend Build Configuration)

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 5173,
    proxy: {
      // Proxy API requests to backend during development
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
})
```

### D. Using Generated Types in Frontend

```typescript
// src/services/api.ts
import { DefaultService } from '@/generated'

// The API client is fully typed based on your backend OpenAPI schema
export const api = new DefaultService()

// Example: Calling a typed API endpoint
async function getUsers() {
  // TypeScript knows the exact shape of the response
  const users = await api.getUsers()

  // Autocomplete works for all properties
  users.forEach(user => {
    console.log(user.id, user.name, user.email)
  })
}

// Example: Type-safe request with validation
async function createUser(name: string, email: string) {
  // TypeScript validates the request body matches the backend schema
  const newUser = await api.createUser({
    body: {
      name,
      email,
      // If you add a field that doesn't exist in the backend,
      // TypeScript will show an error!
    }
  })

  return newUser
}
```

---

## Conclusion

### Key Takeaways

1. **Path Filtering is Essential** - Don't waste CI minutes running irrelevant jobs. Use `dorny/paths-filter` to detect changes and run only what's needed.

2. **Modern Tools Make CI Faster** - `uv` for Python and `Vite` for frontend aren't just local development niceties—they dramatically speed up CI runs.

3. **Schema-Driven Development Ensures Type Safety** - Automatically generating frontend types from backend schemas catches breaking changes at build time, not in production.

4. **Smart Caching Reduces Build Times** - Cache Python packages, npm dependencies, and system libraries. The first run is slow, but subsequent runs are fast.

5. **Clear Error Messages Improve Developer Experience** - Include fix commands in step names. When CI fails, developers should immediately know how to fix it locally.

6. **Concurrency Control Saves Money** - Cancel duplicate CI runs automatically. Only the latest code matters.

7. **Parallel Jobs Provide Fast Feedback** - Split linting, type checking, testing, and building into separate jobs. Developers get lint errors in 30 seconds, not 15 minutes.

### The Bigger Picture

These patterns aren't just about faster CI—they're about:
- **Developer Happiness:** Fast feedback loops and clear error messages reduce frustration
- **Cost Efficiency:** 90% reduction in CI minutes translates to real cost savings
- **Shipping with Confidence:** Type safety across the stack catches bugs before production
- **Team Scalability:** Efficient CI scales with team growth without linear cost increases

### What We Built

Our optimized CI pipeline:
- Runs only relevant jobs (path filtering)
- Provides feedback in seconds for common errors (linting)
- Catches type mismatches between backend and frontend (schema-driven development)
- Caches aggressively (uv, npm, apt packages)
- Prevents duplicate runs (concurrency control)
- Scales efficiently as the team grows

**Result:** 5-minute CI runs instead of 15 minutes, with better signal and lower costs.

### Your Turn

Start with the highest-impact patterns:
1. **Add path filtering** (biggest time savings)
2. **Enable caching** (fastest runs)
3. **Add concurrency control** (prevent waste)
4. **Implement schema-driven development** (if applicable)

Measure the improvements. Share learnings with your team. Iterate.

### Final Thought

Great CI/CD isn't about perfect coverage or zero failures—it's about **fast, reliable feedback that helps teams ship better software faster**.

Your developers shouldn't dread waiting for CI. With these patterns, CI becomes a helpful assistant, not a bottleneck.

Now go build something amazing!
