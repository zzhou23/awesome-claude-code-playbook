# Claude Code Recipe: Windows Development

> Turn Claude Code into a Windows-savvy developer that avoids the platform-specific pitfalls that silently break builds, corrupt files, and waste hours of debugging.

## Quick Start

### CLAUDE.md

Drop this into your project root. The **Platform Constraints** section is the key differentiator — it teaches Claude the Windows-specific rules that most AI tools get wrong.

```markdown
# Project: <your-project>

## Stack
- Language: Python 3.12
- Framework: FastAPI / PySide6 / (your framework)
- Testing: pytest
- Platform: Windows 11

## Platform Constraints (CRITICAL)

### Line Endings
- Configure `.gitattributes` with `* text=auto` in every repo
- Set `git config core.autocrlf true` on Windows machines
- In Python, use `newline=""` for CSV files and `encoding="utf-8"` everywhere
- NEVER assume `\n` — always be explicit about line ending handling

### Virtual Environments
- Activation: `.\venv\Scripts\activate` (NOT `source venv/bin/activate`)
- pip location: `venv\Scripts\pip.exe` (NOT `venv/bin/pip`)
- Python location: `venv\Scripts\python.exe`
- When creating: `python -m venv venv` (use `py -3.12 -m venv venv` if multiple versions installed)

### File Encoding
- ALWAYS pass `encoding="utf-8"` to `open()`, `Path.read_text()`, `subprocess.run()`
- Windows default encoding is cp1252, NOT UTF-8 — this causes silent data corruption
- Example: `open("file.txt", "r", encoding="utf-8")`
- Example: `subprocess.run(cmd, capture_output=True, text=True, encoding="utf-8")`

### File Locking
- Windows locks files that are open by any process
- ALWAYS close file handles before renaming, moving, or deleting
- Use context managers (`with open(...)`) — never use bare `open()` without closing
- If a file operation fails with PermissionError, check what process holds the lock
- Retry pattern for locked files: wait 0.5s, retry up to 3 times, then fail with clear error

### Bash Command Safety
- NEVER use `cd X && command` or `cd X && command 2>/dev/null` — this can trigger Claude Code safety intercepts on Windows
- ALWAYS use absolute paths: `ls /c/Users/me/project` not `cd /c/Users/me/project && ls`
- Use forward slashes in paths, even on Windows (bash interprets them correctly)

### Process Management
- Use `taskkill /F /PID <pid>` instead of `kill -9 <pid>`
- `subprocess.Popen` on Windows does NOT support `preexec_fn` — use `creationflags` instead
- Signal handling differs: `SIGTERM` is not reliably delivered on Windows
- Watch for process race conditions: check `returncode` before reading stdout/stderr

### Path Handling
- Use `pathlib.Path` for ALL path operations — never concatenate strings with `/` or `\\`
- Forward slashes work in Python and bash; backslashes need escaping
- Max path length: 260 chars by default — enable long paths in registry if needed
- Network paths (UNC): `\\server\share` — use `pathlib.PureWindowsPath` if parsing

## Commands
- Install: `pip install -e ".[dev]"`
- Test: `pytest tests/ -v --tb=short`
- Lint: `ruff check src/ tests/`
- Type check: `npx tsc --noEmit` (for TypeScript projects)
- Dev server: `uvicorn src.app.main:app --port 8000 --reload`

## Code Style
- Type hints on all function signatures
- Immutable patterns: frozen dataclasses, return new objects
- Max file size: 400 lines
- Always validate at system boundaries (user input, file I/O, subprocess output)
```

### .gitattributes

Essential for consistent line endings across platforms:

```
# Set default behavior to automatically normalize line endings
* text=auto

# Force specific line endings for certain file types
*.py text eol=lf
*.js text eol=lf
*.ts text eol=lf
*.json text eol=lf
*.yaml text eol=lf
*.yml text eol=lf
*.md text eol=lf
*.sh text eol=lf

# Windows-specific files keep CRLF
*.bat text eol=crlf
*.cmd text eol=crlf
*.ps1 text eol=crlf

# Binary files — never touch
*.png binary
*.jpg binary
*.ico binary
*.woff2 binary
*.zip binary
```

### settings.json

Pre-approve Windows-safe commands:

```json
{
  "permissions": {
    "allow": [
      "pytest tests/ -v --tb=short",
      "pytest tests/**",
      "ruff check src/ tests/",
      "ruff format src/ tests/",
      "pip install -e *",
      "pip install *",
      "python -m *",
      "py -3* -m *",
      "npx tsc --noEmit",
      "npm test",
      "npm run *",
      "curl http://localhost:*",
      "tasklist /FI *",
      "netstat -ano | findstr *"
    ],
    "deny": [
      "rm -rf /",
      "del /s /q *",
      "format *:",
      "taskkill /F /IM *"
    ]
  }
}
```

### Recommended MCP Servers

- [server-filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) - Secure file operations with path validation. **When to use:** Claude needs to access files outside the working directory, especially when dealing with Windows path complexities.

- [server-fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) - Web content fetching. **When to use:** Looking up Windows API docs, PowerShell references, or platform-specific library documentation.

- [server-github](https://github.com/modelcontextprotocol/servers/tree/main/src/github) - GitHub API integration. **When to use:** Checking upstream issues for Windows-specific bugs or finding platform-compatible alternatives.

## Recommended Workflow

### Step 1: Environment Setup and Verification

Before writing any code, confirm the Windows environment is correctly configured.

> "Verify my development environment: check Python version, confirm venv is activated (look for Scripts/ not bin/), verify git line ending settings (core.autocrlf), and confirm .gitattributes exists. Report any issues."

This catches the most common Windows setup problems before they cause subtle bugs later.

### Step 2: Platform-Aware Project Scaffolding

When starting a new project, ensure all files are created with correct encoding and line endings.

> "Scaffold a new FastAPI project. Make sure all Python files use UTF-8 encoding headers, the .gitattributes file is included, and the venv activation command in README uses the Windows Scripts/ path. Include both Windows and Unix commands where applicable."

Claude often defaults to Unix conventions. Being explicit here prevents the first round of "it works on my machine" issues.

### Step 3: File I/O with Explicit Encoding

Every file operation should specify encoding.

> "Add a CSV export feature. Use `encoding='utf-8-sig'` for Excel compatibility, `newline=''` for correct line endings, and context managers for all file handles. The file must be readable by both Excel on Windows and command-line tools on Linux."

The `utf-8-sig` BOM is a Windows-specific requirement for Excel to correctly interpret UTF-8 CSV files.

### Step 4: Subprocess and Process Management

When Claude needs to run external commands, ensure Windows compatibility.

> "Add a feature that runs an external tool as a subprocess. Use `subprocess.run()` with `encoding='utf-8'`, handle `FileNotFoundError` (command not found on Windows is different from Unix), and use `creationflags=subprocess.CREATE_NO_WINDOW` if it should run without a console window."

Windows subprocess behavior differs significantly from Unix — the error types, signal handling, and process creation flags are all different.

### Step 5: Testing Across Platforms

When writing tests, account for platform differences.

> "Add tests for the file export feature. Use `tmp_path` fixture for temporary files, test with both ASCII and Unicode content, verify line endings in output files are correct, and mock `pathlib.Path` operations instead of hardcoding OS-specific paths."

Using pytest's `tmp_path` avoids hardcoded temp directory paths that differ between Windows (`%TEMP%`) and Unix (`/tmp`).

## Tips and Pitfalls

1. **Always specify encoding.** The single most common Windows bug is `open("file.txt")` without `encoding="utf-8"`. Windows defaults to cp1252, which silently corrupts non-ASCII characters. Add this to your CLAUDE.md as a CRITICAL rule.

2. **Use `.gitattributes` from day one.** Do not rely on `core.autocrlf` alone — it is a per-machine setting. `.gitattributes` travels with the repo and enforces consistent behavior for all contributors.

3. **Never use `cd X && command` in Claude Code.** This pattern can trigger safety intercepts when combined with output redirection on Windows. Use absolute paths for all commands: `ls /c/Users/me/project` instead of `cd /c/Users/me/project && ls`.

4. **Watch for file lock errors.** If Claude gets a `PermissionError` when trying to write or delete a file, the file is locked by another process. Common culprits: running dev server, open IDE, antivirus scanning. Check with `tasklist` before retrying.

5. **Use `pathlib.Path` everywhere.** String concatenation with `\\` or `/` will eventually break. `Path("src") / "app" / "main.py"` works correctly on every platform without thinking about separators.

6. **Test Unicode early.** If your app handles any user-generated content, test with Unicode characters (CJK, emoji, accented characters) on Windows early. Encoding bugs are much harder to fix once they have corrupted stored data.

7. **Virtual env paths are different.** If Claude generates a `Makefile` or shell script that references `venv/bin/python`, it will fail silently on Windows where the path is `venv/Scripts/python`. Include both paths in any automation scripts, or use `python -m` instead.

8. **Subprocess signals don't work the same way.** `SIGTERM` and `SIGINT` behave differently on Windows. If your code sends signals to child processes, use `process.terminate()` (which uses `TerminateProcess` on Windows) instead of `os.kill(pid, signal.SIGTERM)`.

## Starter Prompt Templates

### Set Up a Windows-Compatible Python Project

```
Create a new Python project with full Windows compatibility:

1. Project structure with src/ layout
2. .gitattributes with correct line ending rules
3. pyproject.toml with dev dependencies (pytest, ruff, black)
4. A conftest.py that sets UTF-8 encoding globally for tests
5. A Makefile that works on both Windows (via Git Bash) and Unix
6. README with both Windows and Unix setup instructions

All file operations must use encoding="utf-8" explicitly.
```

### Fix Encoding Issues in Existing Project

```
I'm seeing mojibake (garbled characters) in my application on Windows.

Help me fix it:
1. Find all open() calls without explicit encoding
2. Find all subprocess calls without encoding="utf-8"
3. Find all Path.read_text() / write_text() without encoding
4. Fix each one to use explicit UTF-8 encoding
5. Add a .gitattributes if missing
6. Add tests with Unicode content (CJK characters, emoji) to prevent regression
```

### Add Cross-Platform File Operations

```
I need a utility module for file operations that works correctly on Windows:

Requirements:
- Read/write with explicit UTF-8 encoding
- Retry logic for PermissionError (file locked by another process)
- Atomic writes (write to temp file, then rename) that handle Windows file locking
- CSV export compatible with Excel on Windows (UTF-8-BOM)
- Path operations using pathlib only (no string concatenation)
- Tests covering: ASCII content, Unicode content, locked file retry, concurrent access
```

### Set Up Notification Hooks on Windows

```
Set up Claude Code hooks for Windows desktop notifications:

1. Create a PowerShell script that sends Windows toast notifications
2. Configure the Notification hook in settings.json to call the script
3. The notification should show: task summary, duration, success/failure
4. Make it async so it does not block Claude Code
5. Log all notifications to a file for reference

Use PowerShell's BurntToast module or the built-in New-BurntToastNotification cmdlet.
```
