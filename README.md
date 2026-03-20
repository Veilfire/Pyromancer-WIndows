# Pyromancer for Windows 11

**A security-first, AI-powered terminal emulator for Windows 11 with GPU-accelerated rendering.**

Pyromancer is a native Windows 11 terminal application by [Veilfire](https://veilfire.dev) that pairs a high-performance GPU-rendered terminal with an autonomous AI agent system. Every feature is designed with a privacy-first philosophy: secrets never leave Windows DPAPI, all AI actions are auditable, and destructive commands require explicit human approval.

---

## Table of Contents

- [Philosophy](#philosophy)
- [Security & Privacy Architecture](#security--privacy-architecture)
- [Terminal Features](#terminal-features)
- [AI Assistant](#ai-assistant)
- [Autonomous Agent System](#autonomous-agent-system)
- [File System Access Rules](#file-system-access-rules)
- [Intelligent Autocomplete](#intelligent-autocomplete)
- [Tab System](#tab-system)
- [Macro System](#macro-system)
- [Slash Commands](#slash-commands)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Preferences](#preferences)
- [Status Bar](#status-bar)
- [Data Storage](#data-storage)

---

## Philosophy

Pyromancer is built on three principles:

1. **Security is not optional.** Every AI interaction passes through permission gates, risk classifiers, and a tamper-proof audit log. You are always in control of what runs in your terminal.

2. **Privacy by architecture.** API keys live exclusively in Windows DPAPI-protected storage. Terminal output sent to AI providers is sanitized to remove secrets, tokens, and credentials before transmission. Autocomplete data is AES-256-GCM encrypted at rest.

3. **Performance without compromise.** The terminal renderer runs entirely on the GPU via Direct3D 11. The AI agent operates asynchronously without blocking your terminal. Autocomplete suggestions appear in under 5ms.

---

## Security & Privacy Architecture

### Human-In-The-Loop (HITL) Permission System

Every command the AI wants to execute is classified by risk before it runs. The permission system uses 180+ patterns covering Windows-native tools, PowerShell cmdlets, cmd.exe built-ins, and cross-platform utilities:

| Level | Examples | Behavior |
|-------|----------|----------|
| **Safe** (0) | `dir`, `Get-ChildItem`, `type`, `ipconfig`, `Get-Process`, `git log`, `winget list` | Configurable: auto-approve or require approval |
| **Moderate** (1) | `copy`, `move`, `mkdir`, `Set-Content`, `robocopy`, `dotnet build`, `git push`, `winget install` | Configurable: auto-approve or require approval |
| **Dangerous** (2) | `del /s`, `Remove-Item -Recurse`, `taskkill`, `git reset --hard`, `Stop-Service`, `cleanmgr` | Configurable: auto-approve or require approval |
| **Critical** (3) | `reg add`, `sc config`, `schtasks`, `runas`, `wmic`, `Set-ExecutionPolicy`, `Invoke-Expression`, `New-Service` | Always requires approval + warning (unless Critical auto-approve enabled) |

The permission dialog presents:
- The exact command to be executed
- The AI's stated intent
- Risk level badge with color coding (green/yellow/orange/red)
- Four approval scopes: **Deny**, **Allow Once**, **Allow for Session**, **Allow Forever**

Session grants are revocable at any time from the status bar.

### Auto-Approve Threshold

A configurable slider in **Preferences > Security** lets you set how much risk the agent can auto-approve without prompting:

| Threshold | Behavior |
|-----------|----------|
| **None** (default) | Every command requires approval |
| **Safe** | Auto-approve safe commands (read-only operations) |
| **Moderate** | Auto-approve safe + moderate commands |
| **Dangerous** | Auto-approve safe + moderate + dangerous commands |
| **Critical** | Auto-approve all commands including critical system-level operations (requires explicit opt-in) |

Every threshold level requires confirmation through a warning dialog with escalating severity. A persistent disclaimer is displayed whenever any non-None level is active, covering liability and risk acceptance. The Dangerous and Critical levels show a red confirmation button. By enabling any auto-approve threshold, the user acknowledges and accepts all risks and liability for actions taken by the agent at or below that level.

### Secret Redaction

All terminal output is sanitized before it reaches any AI provider:

- **Pattern-based detection (38 patterns)**: AWS keys, GitHub/GitLab tokens, Azure storage keys, JWTs, Bearer tokens, Slack/Stripe/SendGrid tokens, database connection strings, private key blocks, and more
- **Credential cross-reference**: Any string matching a stored DPAPI-protected secret is automatically redacted
- **Entropy analysis**: High-entropy strings near keywords like `password`, `secret`, `token` are flagged
- **Environment variable filtering**: Sensitive environment variables are detected and redacted

Redacted content is replaced with `[REDACTED:type]` markers. The original content never leaves the local machine.

### Tamper-Proof Audit Log

Every security-relevant event is recorded in an HMAC-SHA256 signed JSONL log:

- AI provider invocations (provider, model, query hash)
- Permission decisions (command, risk level, grant/deny)
- Credential access events
- Command executions with source attribution
- File access rule violations
- Workflow events (captured, accepted, declined, reset, deviation blocked)

Each log entry is individually HMAC-signed, making per-entry tampering detectable.

### DPAPI-Protected Secret Storage

Pyromancer never stores API keys, tokens, or credentials in plaintext. All sensitive material is protected using Windows Data Protection API (DPAPI) with user-bound keys. Credentials cannot be transferred between users or machines. This includes:

- AI provider API keys (Anthropic, OpenAI, OpenRouter)
- User-defined secrets (configured in Preferences > Security)
- Autocomplete encryption key (AES-256-GCM)
- Audit log HMAC signing key

### User-Defined Secrets

Define custom secrets in **Preferences > Security > Secrets** and optionally inject them as environment variables into terminal sessions:

- **DPAPI-backed**: Secret values are stored exclusively using Windows DPAPI
- **Environment variable mapping**: Optionally map any secret to an environment variable name (e.g., `$env:GITHUB_TOKEN`)
- **Per-secret toggle**: Enable or disable environment injection independently for each secret with an inline toggle
- **Stealth injection**: Toggling a secret injects it into all running shell sessions without the value appearing in terminal output, scrollback, or AI context
- **Agent-safe**: Secrets injected as env vars are still subject to redaction -- the AI never sees their values
- **Shell history safe**: Injection commands use techniques that prevent the secret from entering shell command history

---

## Terminal Features

### GPU-Accelerated Rendering

The terminal display is rendered entirely on the GPU using Direct3D 11:

- **Per-cell instanced rendering**: Thousands of cells rendered in a single draw call
- **Dynamic glyph atlas**: Font texture atlas built at runtime with lazy rasterization
- **Post-processing effects**: Chromatic aberration, bloom, scanlines, vignette, and color grading
- **Ghost text**: Inline autocomplete suggestions rendered as semi-transparent overlay glyphs with configurable animations
- **HLSL shader pipeline**: Six dedicated shaders compiled at runtime

### Terminal Emulation

- **Full ANSI support**: 14-state parser, 256-color + 24-bit true color, bold/dim/italic/underline/strikethrough
- **Cursor styles**: Block, underline, and beam with configurable animation
- **Scroll regions**: Full DECSTBM support for `vim`, `less`, `htop`
- **Scrollback buffer**: Up to 100,000 lines (default: 10,000) with content reflow on resize
- **Alternate screen buffer**: Full support for modes 47, 1047, 1049
- **Mouse tracking**: Modes 1000, 1002, 1003
- **Bracketed paste mode**: Mode 2004

### Shell Integration

- **ConPTY**: Native Windows Pseudo Console API for full-fidelity terminal I/O
- **Shell selection**: PowerShell or Command Prompt (configurable in Preferences)
- **OSC 133 support**: Semantic prompt detection for precise command boundary tracking
- **Remote session detection**: Automatic SSH connection awareness with hostname display
- **Password prompt detection**: Recognizes password prompts to suppress logging
- **UTF-8 support**: Full multi-byte character handling

### Themes

Four theme options, each with its own dedicated ANSI color palette:

- **Veilfire Stealth** (default): Warm orange text on void-black with ice cyan accents — the signature Veilfire look
- **Cyberpunk Neon**: Neon cyan text on deep indigo with hot pink cursor — electric and vivid
- **Ember Dark**: Bright amber text on charred black with molten red cursor — everything burns
- **Custom**: User-defined text, background, cursor, and selection colors with full color picker

Selecting a theme changes not just the foreground/background but the entire 16-color ANSI palette, so `ls --color`, git diffs, syntax highlighting, and all terminal output feel distinct per theme.

All themes support configurable background opacity and glass effects (None, Clear Glass, Mica, Mica Alt).

---

## AI Assistant

### Provider Support

Pyromancer supports three AI providers, all authenticated via DPAPI-protected storage:

| Provider | Models | Authentication |
|----------|--------|----------------|
| **Anthropic** | Claude Opus 4, Sonnet 4, Haiku 4 | API key |
| **OpenAI** | GPT-4o, GPT-4o-mini, o1-preview, o1-mini | API key |
| **OpenRouter** | 100+ models (Claude, GPT-4, Gemini, Llama, etc.) | API key or OAuth sign-in |

### AI Sidebar

The AI panel opens via `Ctrl+Space` or the animated sparkles button in the tab bar:

- **Resizable sidebar** with drag handle
- **Multi-line input** with auto-expansion
- **Streaming responses** with real-time token display
- **Terminal context**: Last N lines automatically included (configurable, default: 200)
- **Secret-safe**: All terminal context is redacted before transmission
- **Chat/Agent toggle**: Switch between conversational mode and autonomous agent mode

---

## Autonomous Agent System

When in agent mode, the AI autonomously executes multi-step workflows in your terminal.

### Tools

| Tool | Purpose |
|------|---------|
| `execute_command` | Run a shell command and capture output |
| `send_input` | Send raw text/keystrokes for interactive programs |
| `wait_for_output` | Wait for a regex pattern to appear in terminal output |
| `read_terminal` | Read current visible terminal output |
| `task_complete` | Signal that the task is finished |
| `store_memory` | Store observations in the 3-tier memory system |
| `recall_memory` | Retrieve memories with keyword or semantic search |
| `list_memories` | List available memories across tiers |
| `delete_memory` | Remove specific memories |
| `manage_secret` | Store/retrieve secrets from the encrypted credential store |

### Interactive Workflow Support

The agent handles interactive terminal sessions (SSH, password prompts, interactive programs) using `send_input` and `wait_for_output` together:

```
1. send_input("ssh user@host\n")      -- start SSH
2. wait_for_output("password:")        -- detect password prompt
3. (user enters password manually)
4. wait_for_output("\\$\\s*$")         -- detect remote shell prompt
5. execute_command("Get-Process")      -- run commands on remote host
6. execute_command("whoami")           -- continue with next command
7. task_complete("Done.")              -- finish
```

### Execution Modes

| Mode | Description |
|------|-------------|
| **Strict Step-by-Step** | Every tool call requires individual user approval |
| **Strict Workflow** | Entire workflow presented for single approval |
| **Autonomous** | Auto-approves up to a configurable risk level; HITL above that |

### Task Budget System

Every agent task operates within a budget to prevent runaway execution:

- **Command limit**: Maximum tool calls per task (default: 50, configurable: 5-500)
- **Time limit**: Maximum wall-clock time (default: 10 minutes, configurable: 1-60 minutes)
- **Visual tracking**: Progress counter in the agent status display
- **Graceful degradation**: When budget runs low, the agent prioritizes remaining steps

### Context Window Management

- **Configurable window size**: 4,096 to 200,000 tokens (default: 128,000)
- **Real-time tracking**: Estimated token usage displayed as a pie chart ring in the status bar
- **Color coding**: Changes from orange to red as context fills
- **Output truncation**: Tool results exceeding 2,000 characters are intelligently truncated

### Agent States

| State | Description |
|-------|-------------|
| `idle` | No task running |
| `thinking` | Waiting for LLM response (thinking text displayed in real-time) |
| `awaitingPermission` | Waiting for user to approve a command |
| `executing` | Running a tool |
| `observing` | Processing tool output |
| `paused` | User paused the agent |
| `completed` | Task finished successfully |
| `failed` | Task failed or was stopped |

### Per-Tab Agents

Each terminal tab has its own independent agent instance, enabling multiple concurrent agent workflows. Tab indicators show a pulsing lightning bolt icon and animated color-cycling glow border when an agent is active.

### Memory System

The agent includes a three-tier memory architecture:

- **Task Memory**: In-memory scratchpad for the active task, cleared on completion
- **Session Memory**: Persists across tasks within a session, cleared on app restart
- **Long-Term Memory**: Persistent database with full-text search and semantic search

### Background Reflection

An optional reflection engine reviews completed tasks and extracts learnings for long-term memory. Configurable intervals: 15 minutes, 30 minutes, 1 hour, or manual only.

### Single Tool Call Enforcement

The agent executes exactly one tool call per LLM response. If the model returns multiple tool calls, only the first is executed and the rest are discarded. This prevents redundant commands, reduces cost, and ensures the agent observes each result before deciding the next action.

### Customizable System Prompt

The agent persona is fully editable via **Preferences > Agent**. The default persona enforces autonomous execution behavior.

### Task Scheduler

Schedule agent tasks to run automatically on a recurring basis. Managed from the **floating scheduler panel** (clock icon in the tab bar).

| Schedule Type | Description |
|---------------|-------------|
| **Interval** | Every N minutes (5-1440) |
| **Daily** | At a specific time each day |
| **Weekdays** | Monday through Friday at a specific time |
| **Weekly** | On a specific weekday at a specific time |

Each scheduled task has:
- **Name and prompt**: What the agent should do
- **Per-task budget**: Independent max commands and time limits
- **Enable/disable toggle**: Pause individual schedules without deleting them
- **Run tracking**: Last run result, total run count, next fire date
- **Manual trigger**: "Run Now" button with inline confirmation dialog
- **Collapsible cards**: Click to expand/collapse task details; tasks needing attention auto-expand

When a scheduled task fires (automatically or via "Run Now"), the AI sidebar opens in Agent mode and displays the full execution — thinking, commands, permissions, and results — just like a manually submitted agent task.

#### Workflow Capture & Locking

After a task's first run, Pyromancer captures the exact sequence of commands the agent executed and presents them for review in the scheduler panel. This enables **workflow locking** — a safety-first design: capture → review → accept → lock → validate.

| Workflow State | Badge | Description |
|----------------|-------|-------------|
| **Pending First Run** | Yellow | Task has never been executed — no workflow captured yet |
| **Running** | Cyan | Task is currently executing its first run |
| **Review** | Blue | First run completed — captured steps shown for acceptance |
| **Locked (N)** | Green | Workflow accepted — agent constrained to captured patterns |
| **Needs Review** | Red | First run failed (after one auto-retry) — manual intervention needed |

**Workflow acceptance**: When a first run completes, the scheduler panel expands to show each captured step with its tool name and command template. Accept to lock the workflow, or Decline to reset for a fresh first run.

**Command templates**: For shell commands, the template locks the base command and subcommand while allowing flexible arguments. For PowerShell commands (`powershell -Command "..."`), the parser extracts the inner cmdlet for tight binding — e.g., `powershell -Command New-Object Microsoft.Update.Session *` rather than a blanket `powershell *`.

**Per-task auto-execute**: Once a workflow is accepted, toggle auto-execute to skip permission prompts for commands that pass workflow validation AND fall within the global auto-approve threshold.

**Locked execution validation**: During accepted workflow runs, every tool call is validated against the captured template before execution. Deviations are blocked, logged to the audit trail, and the task fails with a clear explanation.

**Reset & Re-run**: At any point, reset an accepted or failed workflow to capture a fresh one. One-click "Reset & Re-run" clears the workflow and immediately triggers a new first run.

**State guards**: Tasks in `Review` or `Needs Review` states are automatically skipped by the scheduler timer — they won't auto-fire until the user takes action.

---

## File System Access Rules

Restrict the agent's file system access with per-path rules:

| Level | Behavior |
|-------|----------|
| **Read Only** | Agent can view files but cannot modify, delete, or create |
| **Read & Write** | Agent can both read and write files in the path |

Manage rules in **Preferences > Agent > File System Access**. Rules are injected into the agent's system prompt so it knows its boundaries upfront.

---

## Intelligent Autocomplete

### Suggestion Sources

1. **Git context** -- Branch names, modified files, remotes
2. **Command history** -- Previously executed commands
3. **Learned tokens** -- Commands, paths, and arguments learned from terminal output
4. **Directory entries** -- Per-host directory listings
5. **Command knowledge base** -- 1,000+ subcommands for `git`, `docker`, `kubectl`, `npm`, `dotnet`, `winget`, and more
6. **PATH binaries** -- All executables in `$env:PATH`
7. **File path completion** -- Context-aware file and directory completion
8. **Fuzzy matching** -- Typo correction via Damerau-Levenshtein distance

### Inline Ghost Text

Completions appear as semi-transparent "ghost text" after the cursor. Press **Right Arrow** to accept. Configurable appearance with multiple animation modes (wave, pulse, rainbow).

### Encrypted Persistence

All learned autocomplete data is encrypted at rest with AES-256-GCM.

---

## Tab System

Each tab maintains independent state:

- Terminal emulator instance with own ConPTY session
- Independent agent instance
- Shell type detection and display
- Remote session awareness (SSH detection)
- Running command indicator and unread output tracking
- Custom tab names (double-click to rename)

### Navigation

| Action | Shortcut |
|--------|----------|
| New Tab | `Ctrl+T` |
| Close Tab | `Ctrl+W` |
| Next Tab | `Ctrl+Tab` |
| Previous Tab | `Ctrl+Shift+Tab` |

---

## Macro System

- **Record**: `/macro record <name>` -- captures keystrokes with timing
- **Play**: `/macro play <name>` -- replays with original timing (delays capped at 2s)
- **Manage**: `/macro list` or **Preferences > Macros** -- assign hotkeys for quick access

---

## Slash Commands

| Command | Description |
|---------|-------------|
| `/help` | Show all available slash commands |
| `/search <query>` | Search terminal output using AI |
| `/explain [text]` | AI explains last terminal output |
| `/fix [description]` | AI suggests fix for last error |
| `/macro record\|play\|list\|stop [name]` | Macro operations |
| `/clear` | Clear AI conversation history |
| `/model [provider] [model]` | Switch AI provider/model |

---

## Keyboard Shortcuts

All shortcuts are customizable in **Preferences > Key Mappings**.

| Action | Default |
|--------|---------|
| Toggle AI Sidebar | `Ctrl+Space` |
| New Tab | `Ctrl+T` |
| Close Tab | `Ctrl+W` |
| Next Tab | `Ctrl+Tab` |
| Previous Tab | `Ctrl+Shift+Tab` |
| Preferences | `Ctrl+,` |
| Start Macro Recording | `Ctrl+Shift+R` |

---

## Preferences

Pyromancer's preferences are organized into nine tabs:

- **General**: Shell selection (PowerShell / Command Prompt), startup directory, font, cursor style and animation, scrollback limit
- **Appearance**: Theme selection (4 built-in with dedicated palettes + custom), color picker with OK/Cancel, background opacity, glass effects, selection opacity
- **AI**: Provider and model selection with OAuth for OpenRouter, context settings, permission border customization (mode, colors, speed, width, softness)
- **Agent**: Execution mode, task budgets, file access rules, memory system, background reflection, customizable system prompt
- **Autocomplete**: Ghost text appearance, animation modes (wave/pulse/rainbow), data management
- **Macros**: Record, play, and manage macros with hotkey assignments
- **Key Mappings**: Customize keybindings with key capture UI
- **Security**: Auto-approve threshold with tiered warnings and persistent disclaimer, user-defined secrets with per-secret env var injection toggle, redaction, audit logging, audit event viewer
- **Debug**: Per-subsystem logging with toggles and live viewer

---

## Status Bar

| Indicator | Description |
|-----------|-------------|
| **System/hostname** | Globe icon for remote sessions, PC icon for local |
| **Username** | Current Windows user |
| **Path** | Current working directory |
| **Permission grant** | Active "Allow" grants with revoke button |
| **AI status** | "Autonomous AI" with context pie chart when running, "AI Ready" when idle |
| **Learning indicator** | Count of learned tokens + command history entries |
| **Hotkey hint** | `Ctrl+Space: AI` |

---

## Data Storage

All user data is stored under `%LOCALAPPDATA%\Veilfire\Pyromancer\`:

| File/Directory | Purpose |
|---|---|
| `settings.json` | User preferences |
| `credentials/` | DPAPI-encrypted API keys & secrets |
| `autocomplete_state.enc` | AES-256-GCM encrypted learned tokens |
| `audit.jsonl` | HMAC-signed audit trail |
| `macros/*.json` | Recorded macro files |
| `memory.db` | Long-term memory database |
| `debug.log` | Debug log output (when enabled) |

---

## System Requirements

- **Windows 11** (version 22H2 or later) or Windows 10 (version 21H1 or later)
- **DirectX 11** compatible GPU
- Internet connection required for AI features

---

## License

Pyromancer is proprietary software by Veilfire. All rights reserved.
