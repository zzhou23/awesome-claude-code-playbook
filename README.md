<!--lint disable awesome-git-repo-age-->

# Awesome Claude Code Playbook [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> Battle-tested patterns, tools, and configurations for [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) — Anthropic's CLI coding agent.

A curated collection of practical resources for getting the most out of Claude Code. Every entry explains **why** it matters and **when** to reach for it.

## Contents

- [Recipes by Role](#recipes-by-role)
- [CLAUDE.md Templates and Best Practices](#claudemd-templates-and-best-practices)
- [MCP Servers](#mcp-servers)
- [Skills and Hooks](#skills-and-hooks)
- [Workflow Patterns](#workflow-patterns)
- [Tips and Tricks](#tips-and-tricks)
- [IDE Integrations](#ide-integrations)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Learning Resources](#learning-resources)

## Recipes by Role

Complete, copy-paste-ready configurations for your role — CLAUDE.md, settings, MCP servers, workflows, and prompt templates in one place.

- [Frontend (React / Next.js)](recipes/frontend-react.md) - Component-driven development workflow with TDD, accessibility, and responsive design patterns. **When to use:** You're building UIs with React or Next.js and want a battle-tested Claude Code setup.
- [Backend API (Python / Node.js / Go)](recipes/backend-api.md) - API development workflow covering schema design, TDD, migrations, and security. **When to use:** You're building REST or GraphQL APIs and want Claude Code configured for backend best practices.
- [Data Science / ML](recipes/data-science.md) - Data exploration, modeling, and productionization workflow optimized for script-first development. **When to use:** You work with pandas, scikit-learn, or ML pipelines and want Claude Code tuned for data workflows.

## CLAUDE.md Templates and Best Practices

### Starter Templates

- [anthropic/claude-code CLAUDE.md](https://github.com/anthropics/claude-code/blob/main/CLAUDE.md) - Official Claude Code repository CLAUDE.md as a real-world reference. **When to use:** Starting a new project and need a solid baseline CLAUDE.md.
- [Anthropic Cookbook Codebase Context](https://github.com/anthropics/anthropic-cookbook/tree/main/misc/prompt_caching) - Patterns for providing rich codebase context via CLAUDE.md. **When to use:** Your codebase is large and Claude keeps losing track of architecture decisions.
- [Modular Rules Directory](https://docs.anthropic.com/en/docs/claude-code/memory#rules-files) - Split rules into `rules/testing.md`, `rules/git.md`, etc. instead of one giant CLAUDE.md. **When to use:** Your CLAUDE.md exceeds 200 lines and becomes hard to maintain.
- [Project-Scoped vs User-Scoped Rules](https://docs.anthropic.com/en/docs/claude-code/memory#claudemd) - Use `.claude/CLAUDE.md` for project rules and `~/.claude/CLAUDE.md` for personal preferences. **When to use:** You want personal coding style preferences without polluting the team config.
- [Conditional Rules Pattern](https://docs.anthropic.com/en/docs/claude-code/memory#memory-files) - Use markdown headings and sections to organize rules by context (e.g., "When writing tests", "When doing refactors"). **When to use:** Different tasks need different rules and you want Claude to pick the right ones.

## MCP Servers

### File and Knowledge Management

- [modelcontextprotocol/server-filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) - Secure file operations with configurable access controls. **When to use:** You need Claude to read/write files outside the working directory with explicit permission boundaries.
- [modelcontextprotocol/server-memory](https://github.com/modelcontextprotocol/servers/tree/main/src/memory) - Persistent knowledge graph for long-running projects. **When to use:** Multi-session projects where Claude needs to remember decisions, architecture, and context across conversations.
- [modelcontextprotocol/server-everything](https://github.com/modelcontextprotocol/servers/tree/main/src/everything) - Reference MCP server demonstrating all protocol features. **When to use:** Learning MCP or testing your MCP client implementation.

### Database and API

- [modelcontextprotocol/server-postgres](https://github.com/modelcontextprotocol/servers/tree/main/src/postgres) - Read-only PostgreSQL access with schema inspection. **When to use:** You want Claude to understand your database schema and write correct queries without giving write access.
- [modelcontextprotocol/server-sqlite](https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite) - SQLite database operations with built-in analysis. **When to use:** Working with local SQLite databases for development or testing.
- [kiliczsh/mcp-mongo-server](https://github.com/kiliczsh/mcp-mongo-server) - MongoDB integration for document inspection and queries. **When to use:** Your project uses MongoDB and Claude needs to understand collection schemas.
- [modelcontextprotocol/server-github](https://github.com/modelcontextprotocol/servers/tree/main/src/github) - GitHub API integration for repos, issues, PRs, and more. **When to use:** You want Claude to interact with GitHub beyond what the `gh` CLI offers, or need structured API access.

### Browser and Web

- [modelcontextprotocol/server-fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) - Web content fetching with robots.txt compliance. **When to use:** Claude needs to read documentation, API references, or web content during development.
- [modelcontextprotocol/server-puppeteer](https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer) - Browser automation via Puppeteer for testing and scraping. **When to use:** You need Claude to interact with web UIs, take screenshots, or run browser-based tests.
- [AeroNotix/mcp-server-playwright](https://github.com/AeroNotix/mcp-server-playwright) - Playwright-based browser automation with multi-browser support. **When to use:** You prefer Playwright over Puppeteer, or need cross-browser testing capabilities.

### DevOps and Infrastructure

- [ckreiling/mcp-server-docker](https://github.com/ckreiling/mcp-server-docker) - Docker container and image management. **When to use:** You want Claude to manage Docker containers, inspect logs, or debug containerized services.
- [stophobia/kubernetes-mcp-server](https://github.com/stophobia/kubernetes-mcp-server) - Kubernetes cluster management via MCP. **When to use:** You need Claude to inspect pods, deployments, or debug Kubernetes issues.
- [modelcontextprotocol/server-aws-kb-retrieval](https://github.com/modelcontextprotocol/servers/tree/main/src/aws-kb-retrieval-server) - AWS Bedrock Knowledge Base retrieval. **When to use:** Your team uses AWS Bedrock knowledge bases and wants Claude to search them.
- [pab1it0/mcp-server-prometheus](https://github.com/pab1it0/mcp-server-prometheus) - Query Prometheus metrics and alerts. **When to use:** You want Claude to analyze application metrics or investigate performance issues using Prometheus data.

### AI and Specialized

- [modelcontextprotocol/server-sequential-thinking](https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking) - Dynamic problem-solving through structured thought sequences. **When to use:** Complex debugging or architecture decisions where step-by-step reasoning helps.
- [modelcontextprotocol/server-brave-search](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search) - Web search via Brave Search API. **When to use:** Claude needs to research current information, find documentation, or look up error messages.
- [modelcontextprotocol/server-slack](https://github.com/modelcontextprotocol/servers/tree/main/src/slack) - Slack workspace integration for reading and posting messages. **When to use:** You want Claude to check Slack for context, post updates, or search conversation history.
- [modelcontextprotocol/server-linear](https://github.com/modelcontextprotocol/servers/tree/main/src/linear) - Linear project management integration. **When to use:** Your team uses Linear for issue tracking and you want Claude to read/create tickets.
- [modelcontextprotocol/server-sentry](https://github.com/modelcontextprotocol/servers/tree/main/src/sentry) - Sentry error tracking integration. **When to use:** You want Claude to look up recent errors, stack traces, and crash reports from Sentry while debugging.

## Skills and Hooks

### Built-in Skills

- [/compact command](https://docs.anthropic.com/en/docs/claude-code/cli-usage#compact) - Compress conversation context to free up tokens. **When to use:** Your session is getting long and Claude starts forgetting earlier context or behaving inconsistently.
- [/clear command](https://docs.anthropic.com/en/docs/claude-code/cli-usage#clear) - Reset conversation entirely. **When to use:** Starting a completely new task unrelated to the current conversation.
- [/init command](https://docs.anthropic.com/en/docs/claude-code/cli-usage#init) - Generate a CLAUDE.md for your project automatically. **When to use:** Bootstrapping Claude Code on an existing project that has no CLAUDE.md yet.
- [/review command](https://docs.anthropic.com/en/docs/claude-code/cli-usage#review) - Review code changes in the current branch. **When to use:** Before committing or creating a PR, to get Claude's feedback on your changes.

### Community Skills

- [Custom Slash Commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands) - Create project-specific or user-specific slash commands in `.claude/commands/`. **When to use:** You have repetitive tasks (deploy, lint, test) that benefit from a single-command shortcut.
- [Claude Code Skills Marketplace](https://github.com/anthropics/claude-code#skills) - Official skills examples covering TDD, debugging, and planning workflows. **When to use:** You want structured, repeatable workflows rather than ad-hoc prompting.

### Git and Command Hooks

- [Pre-Commit Hook Pattern](https://docs.anthropic.com/en/docs/claude-code/hooks#precommit) - Run linters or formatters automatically before Claude commits. **When to use:** You want to enforce code quality standards on every commit Claude makes.
- [Post-File-Edit Hook Pattern](https://docs.anthropic.com/en/docs/claude-code/hooks#postedit) - Trigger actions after Claude edits files (e.g., auto-format, type-check). **When to use:** You want immediate feedback when Claude modifies files, without waiting until commit time.
- [Notification Hook Pattern](https://docs.anthropic.com/en/docs/claude-code/hooks#notification) - Send desktop/Slack notifications when Claude completes long tasks. **When to use:** You're running Claude Code in the background and want to know when it finishes.
- [Permission Guard Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks#permissions) - Restrict which commands or file paths Claude can access. **When to use:** You need fine-grained security controls beyond the built-in permission system.

## Workflow Patterns

- [TDD Loop with Claude Code](https://docs.anthropic.com/en/docs/claude-code/tutorials#tdd) - Red-Green-Refactor cycle: write failing test, implement, refactor with Claude. **When to use:** Building new features or fixing bugs where you want test coverage from the start.
- [Systematic Debugging Pattern](https://docs.anthropic.com/en/docs/claude-code/tutorials#debugging) - Structured approach: reproduce, hypothesize, verify, fix, confirm. **When to use:** Facing a bug you cannot quickly identify — prevents random code changes.
- [Multi-Agent Planning](https://docs.anthropic.com/en/docs/claude-code/sub-agents#planning) - Use sub-agents for research, planning, and parallel implementation. **When to use:** Complex tasks with independent subtasks that can be worked on simultaneously.
- [Session Handoff Pattern](https://docs.anthropic.com/en/docs/claude-code/memory#session-handoff) - Use progress.md or CLAUDE.md notes to pass context between sessions. **When to use:** Long projects that span multiple Claude Code sessions to avoid losing progress.
- [Headless Mode for CI/CD](https://docs.anthropic.com/en/docs/claude-code/cli-usage#headless) - Run Claude Code non-interactively with `--print` flag in CI pipelines. **When to use:** Automating code reviews, migrations, or refactors as part of your CI/CD workflow.
- [Git Worktree Isolation](https://docs.anthropic.com/en/docs/claude-code/sub-agents#worktrees) - Use Git worktrees to give sub-agents isolated copies of the repo. **When to use:** Running multiple agents in parallel that might conflict if editing the same files.
- [Agentic Loop with Checkpoints](https://docs.anthropic.com/en/docs/claude-code/tutorials#checkpoints) - Break complex tasks into phases with explicit verification between each step. **When to use:** Mission-critical changes where you want to review each step before proceeding.

## Tips and Tricks

### Prompt Engineering

- [Be Specific About Output Format](https://docs.anthropic.com/en/docs/claude-code/best-practices#output-format) - Tell Claude exactly what format you want: "respond with only the code, no explanation." **When to use:** Claude is being too verbose or you need machine-parseable output.
- [Provide Examples in CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/best-practices#examples) - Include concrete code examples of preferred patterns in your rules files. **When to use:** Claude keeps using a pattern you dislike despite text instructions.
- [Use Negative Constraints](https://docs.anthropic.com/en/docs/claude-code/best-practices#constraints) - "Do NOT add comments to existing code" is clearer than "minimize comments." **When to use:** Claude keeps doing something you don't want despite softer instructions.

### Performance Optimization

- [Token-Efficient Sessions](https://docs.anthropic.com/en/docs/claude-code/best-practices#tokens) - Keep sessions focused on one task, use `/compact` proactively, avoid re-reading files. **When to use:** You're on a usage plan with token limits or sessions are getting sluggish.
- [Targeted File Reading](https://docs.anthropic.com/en/docs/claude-code/best-practices#file-reading) - Guide Claude to specific files instead of letting it explore the entire codebase. **When to use:** Large monorepos where exploration wastes tokens and time.
- [Pre-Index Your Codebase](https://docs.anthropic.com/en/docs/claude-code/best-practices#indexing) - List key files and their purposes in CLAUDE.md so Claude skips exploration. **When to use:** Your codebase has non-obvious file organization or naming conventions.

### Context Window Management

- [Modular Conversation Strategy](https://docs.anthropic.com/en/docs/claude-code/best-practices#modular) - One session per task; use `/clear` between unrelated tasks. **When to use:** You notice Claude confusing details from an earlier task in the same session.
- [Progress File Pattern](https://docs.anthropic.com/en/docs/claude-code/memory#progress) - Write progress to a file so new sessions can pick up where you left off. **When to use:** Multi-step projects that will definitely span multiple sessions.
- [Strategic Use of /compact](https://docs.anthropic.com/en/docs/claude-code/cli-usage#compact-strategy) - Compact with a focused summary prompt to retain only relevant context. **When to use:** Mid-session when context is filling up but the task is not yet complete.

### Shell and Environment

- [Custom Allowed Commands](https://docs.anthropic.com/en/docs/claude-code/settings#allowed-tools) - Pre-approve specific shell commands in settings to avoid permission prompts. **When to use:** You run the same safe commands (test, lint, build) repeatedly and want fewer interruptions.
- [Environment Variable Injection](https://docs.anthropic.com/en/docs/claude-code/settings#environment) - Pass API keys and config via environment variables for MCP servers. **When to use:** Setting up MCP servers that need authentication tokens.
- [Shell Profile Integration](https://docs.anthropic.com/en/docs/claude-code/cli-usage#shell-profile) - Ensure your shell profile loads correctly so Claude has access to your tools (nvm, pyenv, etc.). **When to use:** Claude cannot find commands that work in your regular terminal.

## IDE Integrations

- [VS Code Extension](https://docs.anthropic.com/en/docs/claude-code/ide-integrations#vscode) - Native VS Code integration with inline diffs and terminal panel. **When to use:** VS Code is your primary editor and you want Claude embedded in your workflow.
- [JetBrains Plugin](https://docs.anthropic.com/en/docs/claude-code/ide-integrations#jetbrains) - IntelliJ/WebStorm/PyCharm integration with tool window and diff view. **When to use:** You use JetBrains IDEs and want Claude without switching to a terminal.
- [Terminal Multiplexer Setup](https://docs.anthropic.com/en/docs/claude-code/cli-usage#terminal) - Run Claude Code in a dedicated tmux/zellij pane alongside your editor. **When to use:** You prefer terminal-based workflows or use Neovim/Vim.
- [Neovim Integration Patterns](https://github.com/anthropics/claude-code/issues) - Community patterns for using Claude Code within Neovim via terminal buffers. **When to use:** Neovim users who want quick access to Claude without leaving the editor.

## Common Issues and Solutions

<!--lint disable awesome-list-item-->

- **Claude forgets instructions mid-session** — Context window full. Fix: use `/compact` with a summary, or split into smaller sessions.
- **Claude edits the wrong file** — Ambiguous file names. Fix: be explicit, e.g., "edit `src/auth/login.ts`, not `tests/auth/login.test.ts`."
- **Permission denied errors** — Sandboxed commands need approval. Fix: add safe commands to `allowedTools` in `.claude/settings.json`.
- **Claude installs wrong package version** — No version constraint in instructions. Fix: pin versions in CLAUDE.md, e.g., "use React 18.x, not 19."
- **MCP server fails to connect** — Config or path issue. Fix: check `claude mcp list`, verify server command paths are absolute.
- **Claude creates huge files** — No file size guidance. Fix: add rule "files should not exceed 400 lines; split if larger."
- **Slow responses in large repos** — Too much exploration. Fix: add file index to CLAUDE.md and use targeted prompts.
- **Claude ignores CLAUDE.md rules** — Rules buried in long file. Fix: move critical rules to top; use bold/caps for must-follow rules.
- **Git hook failures block commits** — Pre-commit hooks too strict. Fix: fix the underlying issue; do not use `--no-verify`.
- **Claude loops on same error** — Stuck in retry pattern. Fix: interrupt and provide context; try a different approach.

<!--lint enable awesome-list-item-->

## Learning Resources

### Official Documentation

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code) - Comprehensive official guide covering setup, usage, and configuration. **When to use:** First-time setup or checking current feature documentation.
- [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook) - Official code examples and best practices. **When to use:** Looking for working code patterns and integration examples.
- [Model Context Protocol Docs](https://modelcontextprotocol.io/) - Official MCP specification and guides. **When to use:** Building custom MCP servers or understanding how MCP works.

### Articles and Tutorials

- [Claude Code Best Practices](https://docs.anthropic.com/en/docs/claude-code/best-practices) - Official guide on writing effective CLAUDE.md files and prompts. **When to use:** Optimizing your Claude Code setup after initial experimentation.
- [Building MCP Servers](https://modelcontextprotocol.io/quickstart) - Step-by-step guide to creating custom MCP servers. **When to use:** You need Claude to interact with an internal tool or API not covered by existing servers.

### Community

- [r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/) - Active Reddit community for Claude discussions and tips. **When to use:** Looking for real-world usage experiences and community-discovered patterns.
- [Anthropic Discord](https://discord.gg/anthropic) - Official Discord server with dedicated Claude Code channels. **When to use:** Real-time help, discussion, and community interaction.
- [Claude Code GitHub Repository](https://github.com/anthropics/claude-code) - Source code, bug reports, and feature requests. **When to use:** Reporting bugs, checking known issues, or requesting features.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on how to add entries to this list.
