# Claude Code Recipe: Multi-Repo Orchestration

> Turn Claude Code into a multi-repo architect that navigates, syncs, and coordinates work across 10+ repositories.

## Quick Start

### CLAUDE.md (Meta Orchestration Repo)

This goes into the root repo that acts as the "hub" for all other repos. It declares the ecosystem so Claude understands the landscape.

```markdown
# Project: <your-org>-hub

## Overview
Meta orchestration repo for a multi-repo ecosystem. This repo does not contain application code — it holds the registry, sync scripts, shared templates, and documentation for all child repositories.

## Ecosystem

### Repository Registry
Source of truth: `hub/projects.yaml`
Every repository, its layer, category, and status is defined here. No repo exists outside this file.

### Architecture Layers
- **Foundation** — Knowledge bases, data sources, reference material
- **Build** — Code generators, scaffolders, enablement tools
- **Validate** — Benchmarks, testing, competitive analysis
- **Publish** — Content authoring, reporting, tracking

### Data Flow
Foundation → Build → Validate → Publish
Each layer consumes the output of the layer before it.

## Key Paths
- `hub/projects.yaml`         — Canonical repo registry (single source of truth)
- `scripts/sync_repos.sh`     — Clone/pull all repos from registry
- `shared/templates/`         — Standardized file templates for all repos
- `docs/progress.md`          — Current status of the overall ecosystem

## Port Allocation
| Port Range | Service Category |
|-----------|-----------------|
| 8000-8009 | Foundation services |
| 8010-8019 | Build services |
| 8020-8029 | Validation services |
| 8030-8039 | Publishing services |
| 9000-9009 | Dev tools and dashboards |

## Conventions
- Repo naming: `{prefix}-{category}-{verb}` (e.g., `acme-data-ingest`)
- All repos live in a flat workspace directory (no nesting)
- All repos share the same file template structure (see shared/templates/)
- Language: English for all documentation and code

## Related Repos
All repos are listed in `hub/projects.yaml`. Run `bash scripts/sync_repos.sh` to clone/update them all.

## Commands
- Sync all repos: `bash scripts/sync_repos.sh`
- List repos: `cat hub/projects.yaml`
- Check status: `for dir in ../my-prefix-*/; do echo "$dir: $(git -C "$dir" status -s | wc -l) changes"; done`
```

### CLAUDE.md (Child Repos)

Each child repo gets a lightweight CLAUDE.md (~50 lines max, 5 sections):

```markdown
# Project: <your-prefix>-<category>-<verb>

## Overview
One-sentence description of what this repo does.

Part of the <your-org> ecosystem (<layer> layer). Consumes data from <upstream-repo>, produces output for <downstream-repo>.

## Commands
- Install: `pip install -e ".[dev]"`
- Test: `pytest tests/ -v --tb=short`
- Lint: `ruff check src/ tests/`
- Run: `uvicorn src.app.main:app --port 8010`
- Dev: `uvicorn src.app.main:app --port 8010 --reload`

## Key Paths
- `src/app/main.py`      — Application entry point
- `src/app/services/`    — Business logic
- `data/analyzed/`       — Output data (dated JSON: YYYY-MM-DD.json)
- `tests/`               — Unit and integration tests
- `docs/progress.md`     — Current status and next steps

## Conventions
- Python 3.12, FastAPI, pytest
- Immutable patterns: frozen dataclasses, never mutate arguments
- File size limit: 400 lines max, extract when larger
- Test coverage: 80%+ minimum

## Related Repos
- `<prefix>-hub` — Central orchestration (meta)
- `<prefix>-<upstream>` — Provides input data (reads from `data/analyzed/`)
- `<prefix>-<downstream>` — Consumes our output
```

### projects.yaml (Registry)

The heart of multi-repo management — a single YAML file that is the source of truth:

```yaml
# hub/projects.yaml — Single source of truth for all repositories
org: your-org-name
workspace: ~/workspace
naming: "{prefix}-{category}-{verb}"

repositories:
  - name: acme-hub
    layer: meta
    category: orchestration
    status: active
    port: null
    description: Central orchestration and registry

  - name: acme-data-ingest
    layer: foundation
    category: data
    status: active
    port: 8000
    description: Raw data ingestion and normalization

  - name: acme-data-enrich
    layer: foundation
    category: data
    status: active
    port: 8001
    description: Data enrichment and classification pipeline

  - name: acme-api-generate
    layer: build
    category: api
    status: active
    port: 8010
    description: API scaffolding and code generation

  - name: acme-test-benchmark
    layer: validate
    category: testing
    status: active
    port: 8020
    description: Automated performance benchmarking

  - name: acme-report-publish
    layer: publish
    category: reporting
    status: active
    port: 8030
    description: Report generation and publishing
```

### sync_repos.sh (Sync Script)

Reads the registry and clones or pulls every repo:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Reads projects.yaml and syncs all repos to the workspace directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
REGISTRY="$SCRIPT_DIR/../hub/projects.yaml"
WORKSPACE=$(grep '^workspace:' "$REGISTRY" | awk '{print $2}')
WORKSPACE="${WORKSPACE/#\~/$HOME}"
ORG=$(grep '^org:' "$REGISTRY" | awk '{print $2}')

echo "Syncing repos for org: $ORG → $WORKSPACE"

# Parse repo names from YAML (lightweight, no yq dependency)
grep '^\s*- name:' "$REGISTRY" | awk '{print $3}' | while read -r repo; do
  target="$WORKSPACE/$repo"
  if [ -d "$target/.git" ]; then
    echo "  Pulling $repo..."
    git -C "$target" pull --rebase --quiet
  else
    echo "  Cloning $repo..."
    gh repo clone "$ORG/$repo" "$target" -- --quiet
  fi
done

echo "Done. $(grep -c '^\s*- name:' "$REGISTRY") repos synced."
```

### Shared File Templates

Store canonical templates in the hub repo. All child repos use these:

```
shared/templates/
├── CLAUDE.md.template       # 5-section structure (~50 lines)
├── README.md.template       # Overview / Quick Start / Architecture / Env Vars / License
├── progress.md.template     # Current Status / Key Decisions / Known Issues / Next Steps
└── TODO.md.template         # High / Medium / Low Priority tiers
```

Example `progress.md.template`:

```markdown
# Progress

## Current Status
Phase: Planning | In Development | Alpha | Beta | Production

## Key Decisions
- YYYY-MM-DD: <decision and rationale>

## Known Issues
- <issue description and workaround if any>

## Next Steps
- [ ] <next task>
```

### settings.json

```json
{
  "permissions": {
    "allow": [
      "bash scripts/sync_repos.sh",
      "cat hub/projects.yaml",
      "git -C * status",
      "git -C * pull --rebase",
      "git -C * log --oneline -10",
      "git -C * diff --stat",
      "pytest tests/ -v --tb=short",
      "ruff check src/ tests/",
      "ruff format src/ tests/",
      "curl http://localhost:*"
    ],
    "deny": [
      "rm -rf *",
      "git push --force *"
    ]
  }
}
```

### Recommended MCP Servers

- [server-github](https://github.com/modelcontextprotocol/servers/tree/main/src/github) - GitHub API for repos, issues, PRs across the org. **When to use:** Creating repos, managing cross-repo issues, or batch operations across the ecosystem.

- [server-filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) - Secure file access across repo boundaries. **When to use:** Claude needs to read/write files in sibling repos outside the current working directory.

- [server-memory](https://github.com/modelcontextprotocol/servers/tree/main/src/memory) - Persistent knowledge graph for ecosystem context. **When to use:** Storing cross-repo relationships, data flow maps, and architectural decisions across sessions.

- [server-fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) - Web content fetching. **When to use:** Checking upstream dependency docs or API specs referenced by multiple repos.

## Recommended Workflow

### Step 1: Bootstrap the Ecosystem

Start by defining the registry and syncing all repos.

> "Read hub/projects.yaml and sync all repositories. Then give me a status report: which repos exist locally, which are missing, and which have uncommitted changes."

This gives you and Claude a shared understanding of the current state before doing any work.

### Step 2: Standardize File Structure

Apply templates to ensure consistency across all repos.

> "Check each repo in the workspace for missing standard files (CLAUDE.md, README.md, docs/progress.md, docs/TODO.md). For any that are missing, generate them using the templates in shared/templates/. Customize the Overview section for each repo based on its entry in projects.yaml."

This is high-value batch work that Claude excels at — repetitive but needs per-repo customization.

### Step 3: Cross-Repo Data Mapping

Document where data flows between repos so Claude can navigate directly.

> "For each repo that produces output data, find the output directory and file format. Then for each repo that consumes that data, verify the import path matches. Create a data flow map in docs/data-flow.md."

Store this in Claude's memory file so every future session knows where to find data without searching.

### Step 4: Implement Features Across Repos

When a feature touches multiple repos, use sub-agents for parallel work.

> "I need to add a new 'quality-score' field. It originates in the data-ingest repo (new YAML field), flows through the api-generate repo (included in API response), and is displayed in the report-publish dashboard. Plan the changes needed in each repo, then implement them."

Claude can launch sub-agents to work on each repo simultaneously using git worktrees for isolation.

### Step 5: Batch Operations

Use the sync script and registry for ecosystem-wide operations.

> "Run linting across all Python repos in the ecosystem. Report which repos have lint errors and fix them."

This leverages the registry to know which repos exist and the standardized commands to know how to lint each one.

## Tips and Pitfalls

1. **Maintain a single source of truth.** The registry YAML file must be the only place where repos are defined. If you create a new repo, add it to the registry first. If you rename a repo, update the registry first. Everything else derives from it.

2. **Use flat directory structure.** Put all repos side by side in one workspace directory. Nesting repos inside other repos creates git confusion and makes cross-repo paths fragile.

3. **Standardize file templates aggressively.** When every repo has the same CLAUDE.md structure, progress.md format, and TODO.md layout, Claude can navigate any repo in the ecosystem without re-learning the structure.

4. **Map cross-repo data paths in memory.** Store a table of "repo X produces data at path Y, consumed by repo Z at path W" in Claude's memory file. This eliminates the most common time-waster: Claude searching for where data comes from.

5. **Assign non-conflicting ports.** Use a port allocation table in the hub CLAUDE.md. Without this, Claude will pick random ports and you will get collisions when running multiple services simultaneously.

6. **Batch-rename with care.** Renaming repos in a multi-repo ecosystem is a multi-step operation: rename on GitHub, update the registry, update all cross-references in CLAUDE.md and README.md files, delete and re-clone local directories. Script this process.

7. **Use sub-agents for cross-repo features.** When a feature touches 3+ repos, launch parallel sub-agents with git worktree isolation. Each agent works on one repo independently, then you integrate the results.

8. **Keep child CLAUDE.md files short.** The hub CLAUDE.md can be detailed (ecosystem overview, data flow, port table), but each child repo CLAUDE.md should be ~50 lines max with 5 sections: Overview, Commands, Key Paths, Conventions, Related Repos.

## Starter Prompt Templates

### Bootstrap a New Repo in the Ecosystem

```
Add a new repository to the ecosystem:

Name: <prefix>-<category>-<verb>
Layer: <foundation|build|validate|publish>
Purpose: <one sentence>
Port: <assigned port>
Upstream: <repo that provides input data>
Downstream: <repo that consumes our output>

Steps:
1. Add it to hub/projects.yaml
2. Create the GitHub repo (internal visibility)
3. Clone it locally
4. Generate CLAUDE.md, README.md, docs/progress.md, docs/TODO.md from shared templates
5. Customize each file based on the purpose and layer
6. Initial commit and push
```

### Cross-Repo Feature Implementation

```
I need to implement a new feature that spans multiple repos:

Feature: <description>
Repos involved:
- <repo-1>: <what changes here>
- <repo-2>: <what changes here>
- <repo-3>: <what changes here>

For each repo:
1. Plan the changes needed
2. Write tests first
3. Implement the feature
4. Run tests and lint

Then verify the data flows correctly end-to-end from the first repo to the last.
```

### Ecosystem Health Check

```
Run a health check across the entire ecosystem:

1. Read hub/projects.yaml to get the full repo list
2. For each repo, check:
   - Does the local directory exist?
   - Are there uncommitted changes?
   - Is the local branch up to date with remote?
   - Do standard files exist (CLAUDE.md, README.md, docs/progress.md)?
   - Do tests pass?
   - Does lint pass?
3. Report results as a table: repo | local | clean | up-to-date | files | tests | lint
```

### Batch Update Across All Repos

```
Apply this change to all repos in the ecosystem:

Change: <description of what to update>
Affected files: <which files in each repo>

For each repo:
1. Make the change
2. Run tests to verify nothing broke
3. Commit with message "<type>: <description>"
4. Push to remote

Report any repos where the change failed or tests broke.
```
