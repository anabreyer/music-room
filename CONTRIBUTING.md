# Contributing Guide

---

## Tech Stack

| Layer | Choice | Justification |
|---|---|---|
| Backend | Python + FastAPI | REST + JSON native, Swagger auto-generated via `/docs`, async support for WebSocket |
| Package manager | uv | Fast dependency resolution, virtual environment management, `uv sync` for any environment setup |
| Database | PostgreSQL | Strong concurrency handling (transactions / locks); required for Track Vote and Playlist Editor conflict resolution |
| Real-time | WebSocket | Built into FastAPI; required for Track Vote live ranking and Playlist Editor sync |
| Auth | JWT | Stateless, mobile-friendly |
| ORM | SQLAlchemy 2.0 | Async support, mature ecosystem, FastAPI compatible |
| Frontend | React Native (Expo) | Single codebase for iOS, Android, and web (React Native Web) |
| State management | Zustand | Lightweight, minimal boilerplate |
| HTTP client | axios | — |
| OAuth | expo-auth-session | Google and Facebook OAuth support |
| Local dev | Docker + uv | Unified environment across all team members; uv manages Python deps inside the container |
| Deployment | Railway (Cloud) | Used for testing and production; GitLab CI auto-deploy, built-in PostgreSQL plugin |
| CI | GitLab CI | Auto-run tests on every push and MR |
| Version control | GitLab | — |

---

## Local Development

Docker is used for local development to ensure a consistent environment across all team members. uv manages Python dependencies inside the container.

```
# Start local dev environment (Docker)
make dev

# Stop
make down

# View logs
make logs
```

For running without Docker (e.g. quick local testing):

```
# Install dependencies
make install    # runs: uv sync

# Start dev server
make run        # runs: uv run uvicorn main:app --reload
```

---

## Deployment

Railway is used for testing and production deployment. GitLab CI triggers a deploy to Railway on every merge to `main`.

```
# Manual deploy (if needed)
make deploy     # runs: railway up
```

Railway builds the app using `uv sync` to install dependencies from `pyproject.toml`.

---

## Makefile Reference

| Command | What it does |
|---|---|
| `make dev` | Start local Docker environment (`docker compose up --build`) |
| `make down` | Stop Docker environment (`docker compose down`) |
| `make logs` | Stream Docker logs (`docker compose logs -f`) |
| `make install` | Install Python dependencies locally (`uv sync`) |
| `make run` | Start FastAPI server locally without Docker (`uv run uvicorn main:app --reload`) |
| `make test` | Run all tests (`uv run pytest`) |
| `make deploy` | Deploy to Railway (`railway up`) |

---

## Commit Message Rules

All commit messages must follow this format — enforced via GitLab Push Rules:

```
[keyword]: commit message
```

| Keyword | When to use |
|---------|-------------|
| `feature` | Adding a new feature |
| `fix` | Bug fix |
| `chore` | Dependency updates, config changes, non-code tasks |
| `docs` | Documentation only changes |
| `refactor` | Code restructuring without behavior change |
| `test` | Adding or updating test code |
| `style` | Formatting, whitespace, lint fixes |

Examples:
```
[feature]: add track voting endpoint
[fix]: resolve concurrency issue on simultaneous votes
[test]: add unit tests for playlist license tier 2
[docs]: update Swagger spec for auth endpoints
```

GitLab enforcement: Settings → Repository → Push Rules → Commit message regex:
```
^\[(feature|fix|chore|docs|refactor|test|style)\]: .+
```

---

## Merge Request Rules

- All MRs must target the `main` branch.
- At least 1 approval from a non-author is required before merging.
- All CI tests must pass before merging.
- MR title must clearly describe the change.

---

## Testing Policy

> Each owner is responsible for writing and maintaining tests for their own code.

- Every new feature must include corresponding test code in the same MR.
- Every change to existing functionality must include updates to the related tests.
- Tests must pass in CI before the MR can be merged.
- Test files must be committed using the `[test]` keyword.

Scope per layer:

| Layer | What to test |
|-------|-------------|
| API endpoints | Correct HTTP responses, error handling, auth enforcement |
| Service logic | Business rules, license tiers, visibility control |
| DB | Query correctness, transaction integrity, concurrency handling |
| Frontend | Component rendering, navigation, API integration |

---

## Environment Variables

- All credentials, API keys, and environment-specific config must be stored in a `.env` file.
- `.env` must be listed in `.gitignore` and never committed to the repository.
- A `.env.example` file with placeholder values must be kept up to date for onboarding.

---

## Dependencies

- No third-party libraries may be committed directly to the repository.
- Backend dependencies must be declared in `pyproject.toml` and installed via `uv sync`.
- Frontend dependencies must be declared in `package.json` and installed via `npm install` or `yarn`.
- The project must be fully installable from a fresh clone via `make` or equivalent.
