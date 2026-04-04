---
description: "Kaizen: continuous self-improvement audit for your Claude Code setup. Surfaces actionable improvements, fixes on approval, tracks progress across sessions."
user-invocable: true
effort: high
---

# Kaizen: Continuous Self-Improvement for Claude Code

You are running a kaizen cycle — a structured audit of the user's Claude Code setup based on best practices from **The Ultimate Claude Code Guide** by Florian Bruniaux:
https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/ultimate-guide.md

Original audit methodology:
https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/tools/audit-prompt.md

## Principles

1. **No walls of text.** The user sees a short numbered list (max 5 items) and nothing else until they respond.
2. **No repeats.** Check the kaizen log. Skip items resolved, deferred (< 7 days), or skipped (< 3 days).
3. **Do the work.** When the user approves, execute the fix. Create files, write configs, add hooks. Do not just describe what to do.
4. **One at a time.** After each fix, confirm in 1-2 lines and move to the next.
5. **Track everything.** Update the kaizen log after each action.
6. **Be relevant.** Only flag issues that matter for this project's actual tech stack and workflows.
7. **Reference the guide.** Each finding links to the relevant Ultimate Guide section.
8. **Safe edits only.** When modifying existing files, use targeted edits (Edit tool) — never overwrite an entire config file. Never delete user files. Kaizen only creates new files or adds to existing ones.

---

## Phase 1: Discovery

Use Claude's native tools — **Read**, **Glob**, **Grep** — not bash commands. This is faster, more reliable, and follows Claude Code conventions.

Do all discovery silently. Do NOT show scan output to the user.

### 1.1 Configuration Files

Read these files (skip any that don't exist):

- `./CLAUDE.md`
- `./.claude/CLAUDE.md`
- `~/.claude/CLAUDE.md`
- `./.claude/settings.json`
- `./.claude/settings.local.json`
- `~/.claude/settings.json`

For each, note: does it exist? Approximate size (line count)? Any obvious issues (secrets, outdated content, bloat over 200 lines)?

### 1.2 Security Hooks and Permissions

From the settings files already read, check:

- **PreToolUse hooks**: Are there guards for secrets, dangerous commands, or domain-specific safety?
- **PostToolUse hooks**: Any auto-formatting or validation?
- **Permissions**: Are sensitive files (`.env`, credentials) denied? Are dangerous operations soft-denied?
- **Gitignore**: Read `.gitignore` and check if `.claude/settings.local.json` and other sensitive `.claude/` files are excluded.
- **Secrets in CLAUDE.md**: Grep all CLAUDE.md files for patterns like `api.key`, `secret`, `token`, `password`, `credential`, `private.key`.

### 1.3 MCP Servers

Check for MCP configuration in:
- `./.mcp.json`
- `./.claude/mcp.json`

Read the files to see which MCP servers are configured. Check for hardcoded secrets (tokens, API keys) — these should use environment variable references instead.

### 1.4 Commands, Agents, Skills, Rules

Use Glob to discover what exists:

- `.claude/commands/*.md` — list all slash commands
- `.claude/agents/*.md` — list all agents
- `.claude/skills/*/SKILL.md` — list all skills
- `.claude/rules/*.md` — list all rules

Don't check against a hardcoded list. Instead, evaluate whether the discovered commands cover the project's common workflows (testing, reviewing, deploying, quality checks).

### 1.5 Auto Memory

Find the project-specific MEMORY.md file. The path follows the pattern:
`~/.claude/projects/<project-path-slug>/memory/MEMORY.md`

Where `<project-path-slug>` is the working directory path with `/` replaced by `-`.

Check:
- How many memory files are referenced?
- Are any entries likely stale?
- Is MEMORY.md approaching the 200-line display limit?

### 1.6 Tech Stack Detection

Read `package.json`, `tsconfig.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`, or other manifest files to understand the project's tech stack. Check for framework config files (`next.config.*`, `vite.config.*`, `angular.json`, etc.). This informs which best practices are relevant.

---

## Phase 2: Analysis Against Best Practices

Using discovery results, evaluate against the Ultimate Guide criteria. Do this internally — do not show analysis to the user.

### What to Check

**Security (Critical):**
- Missing or weak PreToolUse hooks (no secret blocking, no dangerous command blocking)
- No permissions config (or overly permissive)
- `.claude/settings.local.json` not in `.gitignore`
- Secrets or API tokens hardcoded in CLAUDE.md files or MCP config
- No context management patterns documented

**Workflow (Important):**
- Missing slash commands for detected workflows (testing, PR review, deployment, quality checks)
- No custom agents for repeated multi-step tasks
- Incomplete MCP setup for detected tech stack
- No PostToolUse hooks for auto-formatting (e.g., eslint, prettier, rustfmt after edits)
- No `.claude/rules/` directory for composable, glob-scoped conventions
- CLAUDE.md over 200 lines (context bloat — consider extracting into rules files)

**Polish (Nice to Have):**
- Tool/command descriptions not optimized for Claude's tool selection
- Missing MCP servers that would benefit the stack (e.g., Context7 for library docs)
- No per-skill effort levels configured
- Accumulated junk permissions in `settings.local.json` from auto-approvals

### Guide References

Each finding must include a guide section reference:
- CLAUDE.md issues → "Guide: Section 2 - Memory Files"
- Commands → "Guide: Section 3 - Slash Commands"
- Agents → "Guide: Section 4 - Custom Agents"
- Skills → "Guide: Section 5 - Skills"
- Hooks → "Guide: Section 6 - Hooks"
- MCP → "Guide: Section 7 - MCP Servers"
- Security → "Guide: Section 8 - Security Hardening"
- Context → "Guide: Section 9 - Context Management"

---

## Phase 3: Filter Against Kaizen Log

Determine the kaizen log path:
`~/.claude/projects/<project-path-slug>/kaizen-log.json`

Where `<project-path-slug>` is the project working directory with `/` replaced by `-` and leading `-` preserved.

Load the log if it exists. If it doesn't exist (first run), create the directory and seed the log now:

```json
{
  "last_run": null,
  "session_count": 0,
  "total_resolved": 0,
  "items": []
}
```

Then filter:

- **SKIP** items with status `"resolved"`
- **SKIP** items with status `"deferred"` where `defer_until` is in the future
- **SKIP** items with status `"skipped"` where skip date < 3 days ago
- **RE-SURFACE** deferred items past their `defer_until` date, skipped items after 3 days
- **LIMIT** to 5 items max per session

---

## Phase 4: Present to User

Show ONLY this. No preamble. No methodology explanation. No scan output.

```
Kaizen — [date] — Session #[n]

1. [one-line description] [Guide: Section X]
2. [one-line description] [Guide: Section X]
...

Ready to start with #1? (yes / skip / done)
```

**Session impact warning:** If any finding's fix would modify shared config files (`.claude/settings.json`, `.claude/settings.local.json`, `CLAUDE.md`, `.claude/commands/*.md`, `.claude/rules/*.md`, or MCP config), add a line after the list:

```
⚠ Items [list numbers] modify shared config — changes will affect other active Claude sessions.
```

Only show this warning when at least one finding involves shared config. Omit it entirely otherwise.

If zero new items:

```
Kaizen — [date] — Session #[n]
All clear. No new findings.
```

Then stop and hand control to the user for normal work.

---

## Phase 5: Execute Fixes

When the user says **YES**:
- Execute the fix immediately. Create the file, write the config, add the hook.
- Use Ultimate Guide example templates as reference for structure and content.
- Show 1-2 line confirmation of what was done.
- Update kaizen log: status `"resolved"`, date, session number.
- Prompt: `Next: #[n]. [description]. Go? (yes / skip / done)`

When the user says **SKIP**:
- Update kaizen log: status `"skipped"`, today's date.
- Move to next item.

When the user says **LATER** or **DEFER**:
- Update kaizen log: status `"deferred"`, `defer_until` = today + 7 days.
- Move to next item.

When the user says **DONE** or **STOP** or anything else:
- Save kaizen log.
- Say `Kaizen done. [n] items fixed this session.` and nothing else.
- Hand control back to user.

---

## Phase 6: Behavioral Pattern Detection (Every 5th Session)

Check `session_count` in the kaizen log. Every 5th session, also analyze:

Read the project-specific MEMORY.md index file. Look at feedback-type memories for patterns:

- Repeated corrections that should become CLAUDE.md rules or `.claude/rules/` files
- Frequent workflows that should become slash commands
- Missing hooks suggested by patterns (e.g., user always formats manually → PostToolUse hook)
- Oversized memory files needing consolidation (>200 lines)
- Missing agents for repeated delegation patterns

Add findings as kaizen items with priority `"important"` and source `"behavioral"`.

---

## Phase 7: Update Kaizen Log

After each action, update the kaizen log with this structure:

```json
{
  "last_run": "2026-01-15T14:30:00Z",
  "session_count": 1,
  "total_resolved": 0,
  "items": []
}
```

Each item in the items array:

```json
{
  "id": "short-kebab-id",
  "description": "One-line description of the issue",
  "priority": "critical|important|green",
  "guide_ref": "Section X: Name",
  "status": "found|resolved|deferred|skipped",
  "source": "config_scan|behavioral",
  "date_found": "2026-01-15",
  "date_resolved": null,
  "date_deferred": null,
  "defer_until": null,
  "session_found": 1,
  "session_resolved": null
}
```

If the log file doesn't exist on first run, create it at the project-specific path with `session_count: 0` and an empty `items` array.

---

## Manual Overrides

```
/kaizen              — Normal kaizen cycle
/kaizen --full       — Ignore log, recheck everything
/kaizen --log        — Show session history and item statuses
/kaizen --reset      — Clear deferred/skipped, re-surface everything
/kaizen --behavioral — Force behavioral scan regardless of session count
```

Handle by checking `$ARGUMENTS` for the flag.
