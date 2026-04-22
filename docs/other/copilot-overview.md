---
layout: page
title: Github Copilot - Overview
parent: Other
---

# GitHub Copilot – How It Works, Files & Configuration

--- 

***Note:** Written with Copilot - As of: April 2026 – Copilot evolves rapidly, details may change.*

---

## 1. Temporary Files & Data Processing

### Where does Copilot write temporary files?

Copilot itself does **not write persistent temporary files** to disk in the traditional sense. The data flows are as follows:

| Aspect | Details |
|---|---|
| **Context Collection** | The editor (VS Code / VS 2026) collects the current file content, open tabs, cursor position, and prompt text **in memory (RAM)**. |
| **API Communication** | The collected context is sent via HTTPS to the GitHub Copilot API (hosted on Azure). The response (completion/chat) is streamed back. |
| **Extension Cache** | VS Code stores extension data at:<br>`%USERPROFILE%\.vscode\extensions\github.copilot-*\`<br>`%APPDATA%\Code\User\globalStorage\github.copilot-chat\` |
| **Chat History (Session Logs)** | Chat sessions are stored locally in `workspaceStorage`:<br>`%APPDATA%\Code\User\workspaceStorage\<workspace-id>\GitHub.copilot-chat\` |
| **Memory System (Agent Mode)** | In Agent Mode, Copilot can create memory files at:<br>`%APPDATA%\Code\User\memories\` (User scope, persistent)<br>`%APPDATA%\Code\User\memories\session\` (Session scope, temporary)<br>`.github/copilot-memories/` in the repo (Repo scope) |

### What happens to the data?

| Phase | Behavior |
|---|---|
| **During the session** | Context is held in RAM. A new API call is made for each prompt. |
| **After the session** | Chat history remains locally in `workspaceStorage` (until VS Code cleans it up or the workspace is deleted). |
| **Server-side** | GitHub does **not store prompts or code** for Copilot Individual/Business by default. For Copilot Enterprise, admins can configure retention. Code snippets are **not** used for model training (Business/Enterprise). |
| **Telemetry** | Usage data (acceptance rates, latency, feature usage) is collected anonymously. Can be restricted in settings. |
| **Deletion** | There is no automatic cleanup. Chat logs remain locally. Memory files in session scope are deleted after the session ends. |

### Upload Behavior

- **Code is NOT uploaded or stored** (Business/Enterprise).
- Only the **necessary context** (current file, neighboring tabs, explicitly referenced files) is sent per request.
- Transmission is encrypted (TLS 1.2+).
- For Copilot Individual: Snippets can optionally be used for improvements (configurable in GitHub Settings > Copilot).

---

## 2. VS Code, Visual Studio 2026 & Copilot CLI

### VS Code

| Feature | Description |
|---|---|
| **Extensions** | `GitHub Copilot` (Inline Completions) + `GitHub Copilot Chat` (Chat Panel, Agent Mode) |
| **Inline Completions** | Ghost text while typing, Tab to accept |
| **Chat Panel** | Side chat window, supports Ask/Edit/Agent mode |
| **Agent Mode** | Can create/edit files, run terminal commands, run tests, work iteratively |
| **Participants** | `@workspace`, `@terminal`, `@vscode` – specialized chat participants |
| **Edits Mode** | Multi-file editing with diff preview, more targeted than Agent Mode |
| **Context Variables** | `#file`, `#selection`, `#codebase`, `#terminalLastCommand` etc. |

### Visual Studio 2026

| Feature | Description |
|---|---|
| **Integration** | Natively built-in (not an extension), deeper IDE integration |
| **Inline Completions** | Similar to VS Code, but with better C#/C++/.NET context |
| **Chat Window** | Integrated chat window with solution context |
| **Agent Mode** | Available from VS 2022 17.14+ / VS 2026, similar to VS Code |
| **Specialty** | Uses Solution/Project structure as additional context (`.sln`, `.csproj` are automatically included) |
| **Debugging Integration** | Copilot can assist with exceptions and debugging sessions, directly in debug context |

### Copilot CLI

| Feature | Description |
|---|---|
| **Installation** | `gh extension install github/gh-copilot` (via GitHub CLI) |
| **Commands** | `gh copilot suggest` – command suggestions for Shell/Git/gh<br>`gh copilot explain` – explanation of commands |
| **Context** | Has no file/project context, works only with the provided prompt |
| **No Agent Mode** | Purely interactive, no file editing |

### How the Tools Work Together

```
┌─────────────┐    ┌──────────────────┐    ┌─────────────┐
│  VS Code    │    │ Visual Studio    │    │ Copilot CLI │
│  (Extension)│    │ 2026 (native)    │    │ (gh copilot)│
└──────┬──────┘    └────────┬─────────┘    └──────┬──────┘
       │                    │                      │
       └────────────┬───────┘──────────────────────┘
                    │
          ┌─────────▼──────────┐
          │  GitHub Copilot    │
          │  API (Azure/Cloud) │
          └─────────┬──────────┘
                    │
          ┌─────────▼──────────┐
          │  LLM Backend       │
          │  (GPT-4o, Claude,  │
          │   Gemini, etc.)    │
          └────────────────────┘
```

- All three use the **same GitHub Copilot API** on the backend.
- The **license** is per GitHub user, valid for all clients simultaneously.
- There is **no shared state** between clients (no shared chat history).
- Repo-based configuration files (`.github/copilot-instructions.md` etc.) take effect in **all** IDE clients.

---

## 3. VS Code Settings

### Model Selection

The model can be changed at the top of the Chat Panel:

| Model | Characteristics |
|---|---|
| **GPT-4o** | Default, fast, good all-rounder |
| **GPT-4.1** | Latest OpenAI model, stronger for complex code |
| **Claude Sonnet 4** | Anthropic, strong with long contexts and reasoning |
| **Claude Opus 4** | Anthropic, strongest reasoning model |
| **Gemini 2.5 Pro** | Google, large context window |
| **o3/o4-mini** | OpenAI reasoning models, for complex logic |

### Chat Modes (Chat Panel)

| Mode | Description |
|---|---|
| **Ask** | Only answer questions, no file changes |
| **Edit** | Multi-file editing with diff preview, user accepts/rejects individually |
| **Agent** | Fully automatic: read/write files, use terminal, work iteratively, run tests |

### Should You Always Leave Agent Mode On?

**Yes – it is perfectly fine to leave Agent Mode permanently selected**, even when you're just asking questions. Here's why:

| Aspect | Ask Mode | Agent Mode (with a plain question) |
|---|---|---|
| **System Prompt** | Contains instruction "only answer, don't change anything" | Contains instruction "you may use tools, edit files, run terminal" |
| **Tool Access** | Read-only tools (search, read file) | All tools available (write, terminal, etc.) |
| **Behavior on a plain question** | Answers directly | **Also answers directly** – tools are only used when needed |
| **Context Gathering** | Manual (`#file`, `@workspace`) | Can proactively search the codebase for better answers |
| **Token Usage** | Slightly lower (smaller system prompt) | Slightly higher due to tool definitions in the system prompt |

**Key point:** Agent Mode is a **capability extension**, not a requirement to act. When you ask a plain question, the agent will simply answer it – without touching any files. It only takes action (editing, running terminal) when the task requires it.

**Advantages of keeping Agent Mode on:**
- It can **proactively** search the codebase to answer questions better (instead of requiring manual `@workspace` or `#file` references)
- Seamless transition: If a question turns into a task ("Ah, then please fix that"), it can start immediately
- No mode switching needed

**Only downside:**
- With `autoRunTerminalCommands: true`, there's a minimal risk of unintended execution on a misunderstood question. This setting defaults to `false` – it asks before running commands.

**Bottom line:** Leaving Agent Mode on is the most pragmatic choice. The extra token cost from tool definitions in the system prompt is negligible.

### Agent Execution Environment (Local / Cloud / Copilot CLI)

In the Chat Panel there is a dropdown for the **agent execution environment**. This controls **where** the agent's tools and actions run – the LLM itself always runs in the cloud (GitHub API) regardless of this setting.

| | Local | Copilot CLI | Cloud |
|---|---|---|---|
| **LLM (thinking)** | Cloud (GitHub API) | Cloud (GitHub API) | Cloud (GitHub API) |
| **Tool execution** (file edits, terminal) | Your PC, via VS Code | Your PC, via `gh` CLI | GitHub Codespace (remote VM) |
| **Toolset** | Full: file search, editor diffs, terminal, browser, MCP servers, extensions | Limited: terminal-oriented, no editor UI, no diff preview | Full (but remote) |
| **UI integration** | Inline diffs, accept/reject per file, editor highlighting | Minimal UI, results are text-based | Like Local, but in a Codespace |
| **Async work** | No – requires VS Code to stay open | No | Yes – you can close VS Code, it works and creates a PR |
| **Use case** | Normal development (default) | Fallback / lightweight / CI scenarios | Let Copilot work asynchronously, get a PR later |

**When is "Local" grayed out?** This can happen when:
- You're in a **remote workspace** (SSH, WSL, Codespace, Dev Container) – no local file access available
- A prerequisite is missing (e.g. Docker for certain agent features)
- Your Copilot plan doesn't support the option

**Recommendation:** For local development, **always use Local**. It provides the richest integration and full tool access.

### Local Models (Ollama etc.) – Separate Setting

Independently from the execution environment dropdown, VS Code also supports **local LLM models**:

- **VS Code supports local models** via Ollama or other local LLM servers.
- Setting: `Settings > Copilot > Language Models > Local`
- Uses the **VS Code Language Model API** to integrate local models.
- Models run entirely on your own machine – **no data sent to the cloud**.
- Useful for: Air-gapped environments, data privacy, offline work.
- Limitation: Quality depends on the local model, significantly weaker than cloud models.

> **Note:** This is a completely different setting from the "Local/Cloud/CLI" execution environment dropdown. The execution environment controls *where tools run*. Local models control *which LLM does the thinking*.

### IDE (VS Code / VS 2026) vs. Copilot CLI – When to Use What

| | IDE (VS Code / VS 2026) | Copilot CLI (`gh copilot`) |
|---|---|---|
| **Writing/editing code** | Far superior (diffs, multi-file, agent) | Cannot do this |
| **Forgot a shell command?** | Can do it too, but overkill | Perfect: `gh copilot suggest "find large files"` |
| **Explain a command** | Can do it too | Perfect: `gh copilot explain "tar -xzf"` |
| **CI/CD pipeline** | Not available | Only headless option |
| **Quick question in terminal** | Need to open VS Code / use chat | Directly in terminal, no IDE switch |
| **Agent Mode** | Full | Not available |
| **Codebase context** | Full (open files, workspace, references) | None (only the prompt you provide) |

**Summary:**
- **For development work:** The IDE always wins. There is no scenario where the CLI is better for writing code.
- **Copilot CLI is a terminal helper**, not a development tool. It answers "What was the git command for X?" or "What does this command do?" – directly in the terminal without context switching.
- If you have VS Code open anyway, you practically never need Copilot CLI, because the chat can do the same and has more context.

**When CLI makes sense:**
- You're SSH'd into a server with no IDE
- You want to use Copilot programmatically in a script or CI
- You're in a terminal and don't want to switch to the IDE for a quick question

### Important VS Code Settings (`settings.json`)

```jsonc
{
  // Copilot inline completions on/off
  "github.copilot.enable": {
    "*": true,
    "markdown": true,
    "plaintext": false
  },

  // Copilot Chat: Default model
  // (selected via UI, not directly in settings.json)

  // Agent Mode: Automatically run terminal commands
  "github.copilot.chat.agent.autoRunTerminalCommands": false,

  // Which tools the agent is allowed to use
  "chat.agent.tools": { ... },

  // Configure MCP servers (external tool integration)
  "mcp.servers": { ... },

  // Code reference filter (blocks suggestions resembling public code)
  "github.copilot.advanced.codeReferenceFilter": true,

  // Context provider
  "github.copilot.chat.codeGeneration.useReferencedFiles": true,

  // Local models (Ollama etc.)
  "github.copilot.chat.models.local": { ... }
}
```

### Additional Useful Settings

| Setting | Description |
|---|---|
| `github.copilot.chat.localeOverride` | Override the language of chat responses |
| `github.copilot.chat.scopeSelection` | Scope for `@workspace` search |
| `github.copilot.chat.temporalContext.enabled` | Include recently edited files as context |
| `github.copilot.nextEditSuggestions.enabled` | Next Edit Suggestions (NES) – proactive suggestions |
| `github.copilot.chat.agent.autoFix` | Automatically fix errors after edits |
| `chat.agent.maxRequests` | Max number of tool calls per agent turn |

---

## 4. Supporting Files in the Repository

### Overview of Copilot-Relevant Files

| File | Path | Purpose | Format |
|---|---|---|---|
| **Copilot Instructions** | `.github/copilot-instructions.md` | Global instructions for all Copilot interactions | Markdown |
| **Prompt Files** | `.github/prompts/*.prompt.md` | Reusable prompt templates | Markdown with YAML frontmatter |
| **Code Instructions** | `.github/instructions/*.instructions.md` | Context-dependent rules (per glob pattern) | Markdown with YAML frontmatter |
| **VS Code Settings** | `.vscode/settings.json` | Workspace-specific Copilot settings | JSON |
| **MCP Config** | `.vscode/mcp.json` | MCP servers for the project | JSON |
| **GitHub Copilot Config** | `.github/copilot-config.yml` | Content Exclusions (Enterprise) | YAML |

### Yes, Almost Always Markdown – But Not Exclusively

Most Copilot-specific files are **Markdown** (`.md`) because:
- LLMs natively understand Markdown well
- It's easily readable and version-controllable
- YAML frontmatter can be used for metadata

Exceptions: `.vscode/settings.json`, `.vscode/mcp.json`, `.github/copilot-config.yml`

---

### 4.1 `.github/copilot-instructions.md`

The **most important file** – automatically included in every chat interaction.

```markdown
# Copilot Instructions

## Project Overview
This project is an ASP.NET Core 9 Web API with an Angular 19 frontend.

## Technology Stack
- Backend: C# / .NET 9 / ASP.NET Core Minimal APIs
- Frontend: Angular 19 / TypeScript / Tailwind CSS
- Database: PostgreSQL with Entity Framework Core
- Tests: xUnit (Backend), Jest (Frontend)

## Code Conventions
- Code comments always in English
- Use `PascalCase` for public members, `camelCase` for private
- Do not use regions (#region)
- Use async/await consistently, never .Result or .Wait()
- Always define DTOs as records
- Error handling via ProblemDetails (RFC 7807)

## Architecture
- Clean Architecture: Domain → Application → Infrastructure → API
- CQRS with MediatR
- Repository Pattern only for complex queries

## Security
- All endpoints must have [Authorize], unless explicitly marked as [AllowAnonymous]
- Input validation via FluentValidation
- No secrets in code, always use User Secrets or Azure Key Vault

## Testing
- Unit tests for all service methods
- Integration tests with WebApplicationFactory
- At least happy path + one error case per method
```

### 4.2 `.github/prompts/*.prompt.md` (Prompt Files)

Reusable prompts, callable via the Chat Panel (paperclip icon or `/` command).

**Example: `.github/prompts/new-api-endpoint.prompt.md`**

```markdown
---
description: "Creates a new API endpoint following project standards"
mode: "agent"
tools: ["codebase", "terminal", "file"]
---

Create a new API endpoint for the feature: {{ input }}

Follow these steps:
1. Create a Request/Response record in `/src/Application/Features/`
2. Create a MediatR handler
3. Create a FluentValidation validator
4. Register the endpoint in the corresponding endpoint group in `/src/API/Endpoints/`
5. Create xUnit tests in `/tests/`
6. Use existing patterns from #file:src/Application/Features/Users/GetUser.cs as a template
```

**Example: `.github/prompts/code-review.prompt.md`**

```markdown
---
description: "Code review according to team standards"
mode: "ask"
---

Perform a code review of the current changes. Check:

1. **Security**: SQL Injection, XSS, missing authorization
2. **Performance**: N+1 queries, missing indexes, unnecessary allocations
3. **Patterns**: Does the code follow our architecture rules?
4. **Tests**: Are tests missing? Are edge cases covered?
5. **Naming**: Consistent with the rest of the codebase?

Provide concrete feedback with file references.
```

### 4.3 `.github/instructions/*.instructions.md` (Code Instructions)

Context-dependent rules that only apply when certain files are involved.

**Example: `.github/instructions/angular.instructions.md`**

```markdown
---
applyTo: "src/frontend/**/*.ts"
---

# Angular Rules

- Use Standalone Components (no NgModule)
- Signals instead of RxJS for state management
- OnPush Change Detection for all components
- Lazy loading for all feature routes
- Use the inject() pattern instead of constructor injection
- All HTTP calls through a typed API service
```

**Example: `.github/instructions/ef-migrations.instructions.md`**

```markdown
---
applyTo: "**/Migrations/**"
---

# EF Core Migrations

- NEVER modify existing migrations
- Always add a new migration
- Use `HasData()` for seed data
- Every migration needs a descriptive name
- Check the generated SQL before committing
```

### 4.4 `.vscode/mcp.json` (MCP Server)

```jsonc
{
  "servers": {
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${env:DATABASE_URL}"
      }
    },
    "github": {
      "command": "gh",
      "args": ["copilot", "mcp-server"]
    }
  }
}
```

### 4.5 Recommended Repo Structure

```
.github/
├── copilot-instructions.md          # Global instructions
├── copilot-config.yml               # Content Exclusions (Enterprise)
├── prompts/
│   ├── new-api-endpoint.prompt.md   # Reusable prompts
│   ├── code-review.prompt.md
│   ├── fix-bug.prompt.md
│   └── write-tests.prompt.md
└── instructions/
    ├── angular.instructions.md      # Frontend-specific rules
    ├── dotnet.instructions.md       # Backend-specific rules
    ├── ef-migrations.instructions.md
    └── testing.instructions.md

.vscode/
├── settings.json                    # Workspace settings incl. Copilot
└── mcp.json                         # MCP server config
```

---

## 5. Tips & Best Practices

### Do's
- **Maintain `copilot-instructions.md`** – this is the most effective lever for consistent Copilot output
- **Create Prompt Files for recurring tasks** – saves time and standardizes workflows
- **Use Instructions Files for technology-specific rules** – applied automatically based on file patterns
- **Use `#file:` references** in prompts to point Copilot to templates
- **Agent Mode** for complex, multi-step tasks
- **Edit Mode** when you want control over every individual change

### Don'ts
- No secrets in instruction files
- Don't overload instructions (too many rules → LLM ignores some)
- Don't expect Copilot CLI to have the same context as the IDE
- Don't treat Copilot instructions as a replacement for good documentation

---

## 6. Privacy Summary

| Plan | Code Retention | Training | Telemetry |
|---|---|---|---|
| **Individual** | Not stored (opt-out available for snippets) | Opt-in | Yes, anonymized |
| **Business** | Not stored | No | Yes, anonymized |
| **Enterprise** | Not stored | No | Yes, admin-controlled |

Official reference: [GitHub Copilot Trust Center](https://resources.github.com/copilot-trust-center/)
