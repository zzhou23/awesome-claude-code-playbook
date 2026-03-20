# Claude Code Recipe: Backend API (Python / Node.js / Go)

> Turn Claude Code into a senior backend engineer that writes secure, tested, production-ready APIs.

## Quick Start

### CLAUDE.md

Drop this into your project root as `CLAUDE.md`. Adjust the framework-specific sections to match your stack.

```markdown
# Project: <your-api-name>

## Stack
- Language: Python 3.12
- Framework: FastAPI + Uvicorn
- Database: PostgreSQL 16 via SQLAlchemy 2.0 (async)
- Migrations: Alembic
- Testing: pytest + pytest-asyncio + httpx
- Auth: JWT via python-jose

## Project Structure
src/
  app/
    main.py              # FastAPI app factory, lifespan events
    api/
      v1/
        routes/          # Route handlers (thin — delegate to services)
        dependencies.py  # Dependency injection (DB sessions, auth, etc.)
    models/              # SQLAlchemy ORM models
    schemas/             # Pydantic request/response schemas
    services/            # Business logic (one service per domain)
    repositories/        # Database access layer
    core/
      config.py          # Settings via pydantic-settings
      security.py        # Auth helpers, password hashing
      exceptions.py      # Custom exception classes
  migrations/            # Alembic migration scripts
  tests/
    conftest.py          # Shared fixtures (test DB, client, auth)
    unit/                # Pure logic tests (services, utils)
    integration/         # Tests that hit the DB or external APIs
    e2e/                 # Full request cycle tests

## API Conventions
- RESTful endpoints: plural nouns, nested resources where natural
- Response envelope: { "data": ..., "meta": { "page", "total" } }
- Error responses: { "error": { "code": "NOT_FOUND", "message": "..." } }
- Pagination: cursor-based for lists, query params `?cursor=&limit=`
- Versioning: URL path prefix /api/v1/

## Database Rules
- NEVER write raw SQL without parameterized queries — use SQLAlchemy ORM or text() with bind params
- All schema changes go through Alembic migrations — never modify tables manually
- Use repository pattern: routes → services → repositories → DB
- Always use async sessions via `async with get_session() as session:`

## Error Handling
- Raise domain-specific exceptions in services (e.g., UserNotFoundError)
- Map domain exceptions to HTTP responses in a single exception handler
- Never leak stack traces or internal details in API responses
- Log full context server-side with structlog

## Security
- Validate ALL request inputs via Pydantic schemas (strict mode)
- Apply auth middleware globally, whitelist public endpoints explicitly
- Rate limit all endpoints (slowapi or custom middleware)
- Sanitize log output — never log passwords, tokens, or PII
- Use CORS allowlist, not wildcard

## Testing
- Test command: `pytest tests/ -v --tb=short`
- Lint command: `ruff check src/ tests/`
- Format command: `ruff format src/ tests/`
- Minimum coverage: 80% (`pytest --cov=src/app --cov-fail-under=80`)
- Every new endpoint needs: unit test (service logic) + integration test (HTTP round trip)

## Code Style
- Type hints on all function signatures
- Keep route handlers under 10 lines — delegate to services
- One file per model, one file per router, one file per service
- Immutable patterns: return new objects, never mutate arguments
```

**Alternatives for other stacks:**

| Concern | Node.js | Go |
|---|---|---|
| Framework | Express / Fastify | chi / gin |
| ORM / DB | Prisma / Drizzle | sqlc / GORM |
| Migrations | Prisma Migrate / knex | golang-migrate / goose |
| Testing | Jest + supertest | go test + testify |
| Validation | zod / joi | validator / custom |
| Test command | `npm test` | `go test ./...` |

### settings.json

Save as `.claude/settings.json` to pre-approve safe commands:

```json
{
  "permissions": {
    "allow": [
      "pytest tests/ -v --tb=short",
      "pytest tests/**",
      "ruff check src/ tests/",
      "ruff format src/ tests/",
      "alembic revision --autogenerate -m *",
      "alembic upgrade head",
      "alembic downgrade -1",
      "npm test",
      "npm run lint",
      "npx jest **",
      "go test ./...",
      "go vet ./...",
      "make lint",
      "make test",
      "docker compose up -d",
      "docker compose down",
      "docker compose logs --tail=50 *",
      "curl http://localhost:*"
    ],
    "deny": [
      "rm -rf /",
      "alembic downgrade base",
      "docker system prune"
    ]
  }
}
```

### Recommended MCP Servers

- [server-postgres](https://github.com/modelcontextprotocol/servers/tree/main/src/postgres) - Read-only PostgreSQL access with schema inspection. **When to use:** Claude needs to understand your real DB schema to write accurate queries, models, and migrations.

- [server-sqlite](https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite) - Local SQLite operations with built-in analysis. **When to use:** Prototyping or working with SQLite test databases during local development.

- [server-github](https://github.com/modelcontextprotocol/servers/tree/main/src/github) - GitHub API integration for issues, PRs, and code search. **When to use:** Linking implementation work to issues, creating PRs, or searching upstream repos for patterns.

- [server-fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) - Web content fetching with robots.txt compliance. **When to use:** Reading framework docs, third-party API specs, or OpenAPI schema URLs while implementing integrations.

- [mcp-server-docker](https://github.com/ckreiling/mcp-server-docker) - Docker container and image management. **When to use:** Debugging containerized services, inspecting logs, or managing your local dev environment (DB, Redis, etc.).

## Recommended Workflow

### Step 1: API Design

Start by having Claude design the contract before writing any code.

> "Design a REST API for a bookmarks service. I need CRUD for bookmarks, tagging support, and search by title. Output the endpoint list with request/response schemas as Pydantic models."

Review the schemas Claude proposes. Lock them down in `src/app/schemas/` before moving on. This prevents rework later when the shape of responses changes mid-implementation.

### Step 2: Database Schema + Migrations

Once schemas are agreed, move to the data layer.

> "Create SQLAlchemy models for the bookmark and tag tables based on the schemas we defined. Then generate an Alembic migration."

Always review the generated migration file manually. Check column types, nullability, indexes, and foreign key constraints. Claude gets these right most of the time, but "most of the time" is not good enough for migrations that run against production.

### Step 3: TDD Implementation

Work endpoint by endpoint. For each one, follow the red-green-refactor cycle:

1. Ask Claude to write the test first (it should fail)
2. Ask Claude to implement the handler and service logic
3. Run tests to confirm green
4. Ask Claude to refactor if needed

> "Write an integration test for POST /api/v1/bookmarks that creates a bookmark and verifies the 201 response. Then implement the route handler and service to make it pass."

This keeps implementation focused and prevents Claude from generating large blocks of untested code.

### Step 4: Integration Testing

After individual endpoints work, test cross-cutting concerns:

> "Add tests for: auth middleware rejects expired tokens, rate limiter returns 429 after 100 requests/min, and the error handler formats validation errors correctly."

This is where bugs in middleware ordering and error handling surface. Run the full test suite after each change.

### Step 5: Documentation and Deployment Prep

Finish with operational readiness:

> "Generate an OpenAPI description for all endpoints. Add a health check at /healthz that verifies DB connectivity. Write a Dockerfile for production with multi-stage build."

Claude is excellent at generating OpenAPI docs and Dockerfiles. These are high-value, low-risk tasks to delegate.

## Tips and Pitfalls

1. **Specify your ORM and DB in CLAUDE.md.** Without this, Claude will guess — and it might generate raw SQL when you use SQLAlchemy, or Prisma syntax when you use Drizzle. Be explicit.

2. **Pin your framework version.** Add `FastAPI==0.115.x` (or your version) to CLAUDE.md. Framework APIs change between versions, and Claude may generate code for a different version than you are running.

3. **Always remind Claude about security.** Even with rules in CLAUDE.md, explicitly mention auth, input validation, and SQL injection prevention when asking for new endpoints. Security is too important to rely on implicit context.

4. **Keep route handlers thin.** If Claude puts business logic in a route handler, ask it to extract a service. The pattern `route → service → repository` keeps code testable and prevents fat controllers.

5. **Review migration files manually every time.** Claude generates correct migrations most of the time, but an incorrect migration deployed to production is painful to fix. Spend 60 seconds reading the SQL it produces.

6. **Specify your API response format up front.** If you use a response envelope like `{ "data": ..., "meta": ... }`, put it in CLAUDE.md. Otherwise Claude will return bare objects in some endpoints and wrapped objects in others.

7. **Use Claude for boilerplate, not for crypto.** Let Claude generate CRUD endpoints, tests, and Dockerfiles all day. Do not let it implement custom authentication, encryption, or token generation without thorough review.

8. **Test with a real database, not mocks.** Ask Claude to set up fixtures with a test database (SQLite in-memory or a Dockerized Postgres). Mocking the DB layer hides real query bugs and schema mismatches.

## Starter Prompt Templates

### Scaffold a New CRUD Endpoint

```
Create a full CRUD endpoint for the "Project" resource at /api/v1/projects.

Include:
- Pydantic schemas (CreateProject, UpdateProject, ProjectResponse)
- SQLAlchemy model with id, name, description, created_at, updated_at
- Service layer with create, get_by_id, list (paginated), update, delete
- Route handlers using dependency injection for DB session and auth
- Integration tests for all 5 operations (happy path + 404 on missing)

Follow the project structure in CLAUDE.md. Use the repository pattern.
```

### Add Authentication Middleware

```
Add JWT authentication to the API:

1. Create a /api/v1/auth/login endpoint that accepts email+password and returns access+refresh tokens
2. Create a dependency `get_current_user` that validates the JWT from the Authorization header
3. Apply it globally, but whitelist: /healthz, /docs, /api/v1/auth/login
4. Add tests: valid token succeeds, expired token returns 401, missing token returns 401

Use python-jose for JWT. Store user passwords with bcrypt via passlib.
Hash tokens, never store them in plain text.
```

### Write a Database Migration

```
I need to add a "workspace" concept to the app:

- New table: workspaces (id UUID, name, slug UNIQUE, created_at)
- New join table: workspace_members (workspace_id, user_id, role ENUM('owner','member','viewer'))
- Add workspace_id FK to the projects table (nullable for now, will backfill later)

Generate the SQLAlchemy models and an Alembic migration.
Include indexes on: workspaces.slug, workspace_members(workspace_id, user_id), projects.workspace_id.
```

### Profile a Slow Endpoint

```
The GET /api/v1/projects endpoint is slow (~800ms) when returning 50 items with nested tags.

Help me diagnose and fix it:
1. Add SQL query logging to see what queries are running
2. Identify N+1 query problems or missing indexes
3. Add eager loading or query optimization
4. Add a benchmark test that asserts response time < 200ms for 100 projects

Show before/after query counts.
```

### Add Comprehensive Error Handling

```
Improve error handling across the API:

1. Create domain exceptions in core/exceptions.py: NotFoundError, ConflictError, ValidationError, AuthenticationError, PermissionError
2. Create a global exception handler that maps each to the correct HTTP status (404, 409, 422, 401, 403)
3. Response format: { "error": { "code": "NOT_FOUND", "message": "Project not found", "details": {} } }
4. Never leak stack traces — log them server-side with structlog, return generic message to client
5. Add tests for each exception type to verify correct status codes and response format
```
