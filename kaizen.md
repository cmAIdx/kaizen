---
description: "Kaizen: continuous self-improvement audit based on The Ultimate Claude Code Guide. Surfaces actionable improvements, fixes on approval, tracks progress across sessions."
user-invocable: true
effort: high
---

# Kaizen: Continuous Self-Improvement

You are running a kaizen cycle based on best practices from The Ultimate Claude Code Guide and Anthropic's official Claude Code documentation.

### Attribution & Sources

This command draws from and is informed by:

| Source | Author / Org | URL |
|--------|-------------|-----|
| The Ultimate Claude Code Guide | Florian Bruniaux | https://github.com/FlorianBruniaux/claude-code-ultimate-guide |
| Claude Code Official Docs | Anthropic | https://code.claude.com/docs/en/overview |
| Claude Code Mastery | TheDecipherist | https://github.com/TheDecipherist/claude-code-mastery |
| Awesome Claude Code Workflows | ithiria894 | https://github.com/ithiria894/awesome-claude-code-workflows |
| Claude Code Examples (hooks, settings) | Anthropic | https://github.com/anthropics/claude-code/tree/main/examples |

Specific doc pages referenced: [Hooks](https://code.claude.com/docs/en/hooks), [Hooks Guide](https://code.claude.com/docs/en/hooks-guide), [Memory](https://code.claude.com/docs/en/memory), [Settings](https://code.claude.com/docs/en/settings), [Permissions](https://code.claude.com/docs/en/permissions), [Sandboxing](https://code.claude.com/docs/en/sandboxing), [MCP](https://code.claude.com/docs/en/mcp), [Sub-agents](https://code.claude.com/docs/en/sub-agents), [Context Window](https://code.claude.com/docs/en/context-window), [Best Practices](https://code.claude.com/docs/en/best-practices).

## Principles

1. **No walls of text.** The user sees a short numbered list (max 5 items) and nothing else until they respond.
2. **No repeats.** Check the kaizen log. Skip items resolved, deferred (< 7 days), or skipped (< 3 days).
3. **Do the work.** When the user approves, execute the fix. Create files, write configs, add hooks. Do not just describe what to do.
4. **One at a time.** After each fix, confirm in 1-2 lines and move to the next.
5. **Track everything.** Update the kaizen log after each action.
6. **Be relevant.** Only flag issues that matter for this project's actual tech stack and workflows.
7. **Reference the guide.** Each finding links to the relevant Ultimate Guide section.
8. **Never expose secrets.** Finding descriptions must reference file and line number only — never include actual secret values.

---

## Phase 0: Arguments & Behavioral Gate

### 0.1 Parse Arguments

Check `$ARGUMENTS` for flags and set mode:

| Flag | Behavior |
|------|----------|
| (none) | Normal kaizen cycle (Phases 1-7) |
| `--full` | Skip all Phase 3 filtering — recheck everything including resolved items |
| `--log` | Show log summary (see format below), then stop |
| `--reset` | Change all `"deferred"` and `"skipped"` items back to `"found"`, null out their `date_skipped`/`date_deferred`/`defer_until` fields, then run normal cycle |
| `--behavioral` | Force behavioral analysis regardless of session count |

If an unrecognized flag is provided, inform the user and list valid flags, then stop.

If multiple valid flags are given, `--log` takes priority (display and stop). Otherwise, flags combine: `--full --behavioral` rechecks everything and includes behavioral analysis.

**`--log` display format:**

```
Kaizen Log — [n] sessions, [r] resolved, [o] open

| # | ID | Priority | Status | Found | Resolved |
|---|-----|----------|--------|-------|----------|
| 1 | [id] | [priority] | [status] | [date] | [date/-] |
...
```

### 0.2 Load Kaizen Log

Load the log from `~/.claude/projects/<project-path-slug>/kaizen-log.json`.

Where `<project-path-slug>` is the project working directory with `/` replaced by `-` and leading `-` preserved (e.g., `-Users-cm-Documents-dev-myproject`).

If the file does not exist, start with a fresh log (`session_count: 0`, empty `items` array). If the JSON fails to parse, back up the corrupted file (rename to `kaizen-log.json.bak`) and start fresh.

**If `--log` flag is set, do NOT increment `session_count`.** Display the log (Phase 0.1 format) and stop — `--log` is a read-only operation.

**Otherwise, increment `session_count` now**, before any other logic. This ensures the behavioral gate check uses the correct session number.

### 0.3 Behavioral Gate

If this is a 5th session (`session_count % 5 === 0`), prompt:

```
This is session #[n] — would you like to include a user usage analysis?
This scans your feedback memories and workflow patterns for improvements.
(y / n)
```

On non-5th sessions, prompt with a shorter message:

```
Include user usage analysis? (y / n — auto-suggested every 5 sessions)
```

If the user says **y** or **yes** (or if `--behavioral` flag is set), set a flag to run behavioral analysis after Phase 5.

If **n**, **no**, or any other response (e.g., "go", "start"), skip behavioral analysis.

---

## Phase 1: Discovery

Use Claude's native tools — **Read**, **Glob**, **Grep** — not bash commands. This is faster, more reliable, and follows Claude Code conventions.

Do all discovery silently. Do NOT show scan output to the user.

### 1.1 Configuration Files

Read these files (skip any that don't exist):

- `./CLAUDE.md`
- `./.claude/CLAUDE.md`
- `~/.claude/CLAUDE.md`
- `./CLAUDE.local.md` (gitignored, user-specific project overrides)
- `./.claude/settings.json`
- `./.claude/settings.local.json`
- `~/.claude/settings.json`

For each, note: does it exist? Approximate size (line count)? Any obvious issues (secrets, outdated content, bloat over 200 lines)? Check for `@import` syntax (`@path/to/file`) that pulls in additional files — follow imports to understand the full context chain.

### 1.2 Security, Hooks, and Permissions

From the settings files already read (both project and global), check:

- **Hooks** (all events, not just PreToolUse/PostToolUse):
  - `PreToolUse`: Guards for secrets, dangerous commands, domain-specific safety (e.g., multi-tenant isolation)?
  - `PostToolUse`: Auto-formatting, linting, validation?
  - `SessionStart`/`PostCompact`: Context re-injection of critical rules after compaction?
  - `Stop`: Quality gate hooks (agent-based test verification before Claude finishes)?
  - `Notification`: Desktop alert hooks for unattended sessions?
  - Note any other configured events (`UserPromptSubmit`, `SubagentStart`, `ConfigChange`, etc.)
- **Permissions**: Are `.env` and credential files denied? Are dangerous operations soft-denied? Note: Read/Edit deny rules do NOT block Bash subprocesses (`cat .env` bypasses `Read(.env)` deny). Flag this gap if no Bash-level guard exists.
- **Sandboxing**: Is OS-level sandboxing enabled? Check for `sandbox.enabled`, filesystem write restrictions, network domain filtering. Especially important for projects handling sensitive data.
- **Auto mode**: Check `autoMode.environment`, `autoMode.allow`, and `autoMode.soft_deny` for quality and completeness. Are environment descriptions accurate? Are soft denies covering critical safety policies?
- **Gitignore**: Read `.gitignore` and check if `.claude/settings.local.json` and other sensitive `.claude/` files are excluded.
- **Secrets scan**: Grep CLAUDE.md files AND MCP config files (`.mcp.json`, `.claude/mcp.json`) for patterns like `api.key`, `secret`, `token`, `password`, `credential`, `private.key`. Exclude any value that is a variable reference rather than a literal (e.g., `${VAR_NAME}`, `$VAR_NAME`, `process.env.VAR`, `env:VAR`).

### 1.3 MCP Servers

Check for MCP configuration in:
- `./.mcp.json`
- `./.claude/mcp.json`

Read the files to see which MCP servers are configured. Cross-reference with the `enabledMcpjsonServers` array in `.claude/settings.local.json` to see which are active.

### 1.4 Commands, Agents, Skills, Rules

Use Glob to discover what exists:

- `.claude/commands/*.md` — list all slash commands
- `.claude/agents/*.md` — list all agents
- `.claude/skills/*/SKILL.md` — list all skills
- `.claude/rules/*.md` — list all rules

Don't require specific command names. Instead, evaluate whether the discovered commands cover the project's common workflows (testing, reviewing, deploying, quality checks) based on the detected tech stack. Note if agents directory is empty — complex projects with multi-step workflows may benefit from custom agents.

For `.claude/rules/*.md` files, check if they use `paths:` frontmatter to scope to relevant file globs (e.g., `paths: ["src/app/api/**"]`). Path-scoped rules only load when Claude works with matching files, reducing context window usage.

### 1.5 Auto Memory

Read the MEMORY.md index from the project-specific memory directory at:
`~/.claude/projects/<project-path-slug>/memory/MEMORY.md`

(Same `<project-path-slug>` as the kaizen log path.)

Check:

- How many memory files are referenced?
- Are any entries likely stale (referenced files that no longer exist)?
- Is MEMORY.md approaching the 200-line display limit?
- Are there memory files on disk not referenced in MEMORY.md? (List files in the memory directory and compare.)

### 1.6 Tech Stack Detection

Read `package.json`, `tsconfig.json`, and check for framework config files (`next.config.*`, `vite.config.*`, etc.) to understand the project's tech stack. This informs which best practices are relevant.

---

## Phase 2: Analysis Against Best Practices

Using discovery results, evaluate against the Ultimate Guide criteria. Do this internally — do not show analysis to the user.

### What to Check

**Security (Critical):**
- Missing or weak PreToolUse hooks (no secret blocking, no dangerous command blocking)
- No permissions config (or overly permissive)
- `.claude/` sensitive files not in `.gitignore`
- Secrets found in CLAUDE.md or MCP config files
- Permission deny rules for `.env` without matching Bash guards (Read/Edit deny does NOT block `cat .env` via Bash — flag this gap)
- No OS-level sandboxing on projects handling sensitive data (check `sandbox.enabled` in settings)
- `autoMode.soft_deny` missing critical safety policies, or `autoMode.environment` inaccurate/stale

**Workflow (Important):**
- Missing slash commands for detected workflows (testing, PR review, deployment)
- No custom agents for repeated multi-step tasks
- Incomplete MCP setup for detected tech stack
- No PostToolUse hooks for auto-formatting/linting
- No rules directory for team conventions
- CLAUDE.md over 200 lines (context bloat)
- No Stop hook quality gate (agent-based test verification before Claude finishes)
- No SessionStart or PostCompact hook to re-inject critical context after compaction

**Context Optimization (Important):**
- `.claude/rules/*.md` files without `paths:` frontmatter (loads into every session instead of only when matching files are touched)
- CLAUDE.md content that could be moved to path-scoped rules to reduce baseline context usage
- No context management patterns documented in CLAUDE.md

**Polish (Nice to Have / "nit"):**
- Tool descriptions not optimized for Claude's tool selection
- Missing MCP servers that would benefit the stack (e.g., Context7 for library docs)
- No per-skill effort levels
- Custom PR review configuration missing
- No Notification hook for desktop alerts during unattended sessions

### Guide References

Each finding must include a guide section reference. For items covered by the Ultimate Guide, use that reference. For items based on Anthropic's official docs, use the docs reference instead:

- CLAUDE.md issues → "Guide: Section 2 - Memory Files"
- Commands → "Guide: Section 3 - Slash Commands"
- Agents → "Guide: Section 4 - Custom Agents"
- Skills → "Guide: Section 5 - Skills"
- Hooks → "Guide: Section 6 - Hooks"
- MCP → "Guide: Section 7 - MCP Servers"
- Security → "Guide: Section 8 - Security Hardening"
- Context → "Guide: Section 9 - Context Management"
- Sandboxing → "Docs: Sandboxing"
- Permissions → "Docs: Permissions"
- Auto mode → "Docs: Permissions (auto mode)"
- Path-scoped rules → "Docs: Memory (rules)"

---

## Phase 3: Filter Against Kaizen Log

Using the log loaded in Phase 0, filter findings:

- **SKIP** items with status `"resolved"` (unless `--full` flag is set)
- **SKIP** items with status `"deferred"` where `defer_until` is in the future (i.e., fewer than 7 days since deferral)
- **SKIP** items with status `"skipped"` where fewer than 3 days have elapsed since `date_skipped`
- **RE-SURFACE** deferred items past their `defer_until` date, and skipped items where 3+ days have elapsed. When re-surfacing, change the item's status back to `"found"` and null out `date_skipped`/`date_deferred`/`defer_until`.
- **LIMIT** to 5 items max per session

If `--full` is set, skip all filtering — recheck everything including previously resolved items.

**Deduplication:** When matching new Phase 2 findings against existing log items, compare by the target file/config and nature of the issue — not by exact description text. If a new finding addresses the same gap as an existing item, reuse the existing item's ID rather than creating a duplicate.

---

## Phase 4: Present to User

Show ONLY this. No preamble. No methodology explanation. No scan output.

**Never include actual secret values in finding descriptions.** Reference file and line number only (e.g., "Hardcoded token in .mcp.json:5", not the token itself).

```
Kaizen — [date] — Session #[n]

1. [one-line description] [Guide: Section X]
2. [one-line description] [Guide: Section X]
...

Ready to start with #1? (y / skip / done)
```

**Shared config items:** If a finding's fix would modify shared config files (`.claude/settings.json`, `.claude/settings.local.json`, `CLAUDE.md`, `~/.claude/CLAUDE.md`, `~/.claude/settings.json`, `.claude/commands/*.md`, `.claude/rules/*.md`, `.claude/agents/*.md`, `.claude/skills/*/SKILL.md`, or MCP config), mark it in the list with red text:

```
1. [one-line description] [Guide: Section X]
2. **🔴 [one-line description] — modifies shared config** [Guide: Section X]
3. [one-line description] [Guide: Section X]
```

When prompting for a red-flagged item, show the warning inline and offer skip:

```
Next: #2. [description]
🔴 This modifies shared config and will affect other active Claude sessions.
Go? (y / skip / done)
```

If zero new items:

```
Kaizen — [date] — Session #[n]
All clear. No new findings.
```

Then stop and hand control to the user for normal work.

---

## Phase 5: Execute Fixes

**Before executing any fix, verify the issue still exists.** If the fix was already applied (e.g., by a previous interrupted session or manual change), mark it resolved without re-applying.

When the user says **Y** or **YES**:
- Execute the fix immediately. Create the file, write the config, add the hook.
- Use Ultimate Guide example templates as reference for structure and content.
- Show 1-2 line confirmation of what was done.
- Update kaizen log: status `"resolved"`, date, session number.
- Prompt: `Next: #[n]. [description]. Go? (y / skip / done)`

When the user says **SKIP**:
- Update kaizen log: status `"skipped"`, `date_skipped` = today's date.
- Move to next item.

When the user says **LATER** or **DEFER**:
- Update kaizen log: status `"deferred"`, `defer_until` = today + 7 days.
- Move to next item.

When the user says **DONE** or **STOP** or **EXIT**:
- Save kaizen log.
- Say `Kaizen done. [n] items fixed this session.` and nothing else.
- Hand control back to user.

**For any other response** (questions, clarifications, multi-word input):
- Treat it as a question about the current item. Respond helpfully, then re-prompt with the same item: `Go? (y / skip / done)`

**After the last item** (all items processed):
- Save kaizen log.
- Say `Kaizen done. [n] items fixed this session.` and nothing else.
- Hand control back to user.

---

## Phase 6: Behavioral Analysis & User Coaching

This phase runs only if the behavioral flag was set in Phase 0.3 (user said yes, or `--behavioral` flag).

### 6.1 Data Gathering

Read the project-specific MEMORY.md index file. Read ALL referenced memory files (not just feedback-type). Also read the kaizen log history. Gather:

- **Feedback memories** (`type: feedback` in YAML frontmatter, or `feedback_*` prefix): Corrections, confirmations, workflow preferences
- **User memories** (`type: user` or `user_*` prefix): Role, expertise, goals
- **Project memories** (`type: project` or `project_*` prefix): Current initiatives, deadlines, context
- **Reference memories** (`type: reference` or `reference_*` prefix): External tools, dashboards, docs
- **Kaizen log history**: What's been fixed, skipped, deferred across sessions — patterns in what the user engages with vs. skips

### 6.2 Analysis Categories

Analyze across five dimensions. Do this internally — build the full picture before presenting.

**A) Anti-patterns — things the user does that hurt productivity or safety:**
- Repeated corrections in feedback memories that keep recurring (same mistake across sessions)
- Manual steps that should be automated (formatting, testing, deployment gates)
- Overriding safety guards or skipping recommended practices
- Context window waste (asking Claude to re-read files already in context, not using `/compact`)
- Not using subagents for research-heavy tasks (polluting main context)

**B) Good practices — things the user does well (reinforce these):**
- Effective use of existing hooks, commands, rules
- Good memory hygiene (keeping memories relevant and updated)
- Consistent workflow patterns that work well
- Appropriate use of tools (e.g., using Glob instead of `find`)

**C) Automation opportunities — repetitive tasks that should become hooks or commands:**
- Workflows the user triggers manually every session or frequently
- Multi-step sequences that could be a single slash command
- Quality checks done manually that could be PostToolUse or Stop hooks
- Delegation patterns that should become custom agents

**D) Configuration gaps — settings that would improve their setup:**
- Missing path-scoped rules for domain-specific conventions
- Permissions that could be tightened or loosened based on actual usage
- MCP servers that would benefit their workflow
- Auto-mode environment descriptions that are stale or incomplete

**E) Interaction style coaching — how to get better results from Claude Code:**
- Are prompts too vague? (leads to wasted iterations)
- Are prompts too prescriptive? (prevents Claude from finding better solutions)
- Over-use of "fix it" without context? (causes shallow fixes)
- Not using `/clear` between unrelated tasks? (context pollution)
- Not leveraging `/btw` for side questions? (unnecessarily adds to conversation history)
- Missing use of effort levels or model routing for different task types?

### 6.3 Present the Coaching Report

Present a concise coaching report. This is NOT the Phase 4 numbered-list format — it's a brief narrative with clear sections. Keep each section to 1-3 bullet points. Skip any section with nothing noteworthy.

```
## Usage Analysis — Session #[n]

**Working well:**
- [1-2 things they do effectively — be specific, cite the pattern]

**Watch out for:**
- [1-2 anti-patterns or habits that cost time — cite the feedback memory or pattern]

**Suggested automations:**
- [concrete suggestion: "Create a /deploy command for your push-to-both workflow"]
- [concrete suggestion: "Add a PostCompact hook to re-inject multi-tenant isolation rules"]

**Quick wins:**
- [small config/setting changes that would help immediately]
```

### 6.4 Offer to Implement

After the coaching report, prompt:

```
Want me to implement any of these? I can create the hooks, commands, or config changes now.
List the items you'd like (e.g., "1 and 3") or say "all" / "skip".
```

For each item the user selects:
- Execute the fix (create the command, add the hook, update the config)
- Show 1-2 line confirmation
- Add to kaizen log with `source: "behavioral"` and `priority: "important"`
- Mark resolved

If the user says **skip** or **done**, move on without logging unselected items (they're coaching observations, not persistent findings — only log items the user acts on).

### 6.5 Behavioral Memory Hygiene

While analyzing memories, also check:
- Oversized memory files needing consolidation (>200 lines)
- Stale feedback memories that have been addressed by rules or hooks
- Duplicate or contradictory memories
- Memory files on disk not referenced in MEMORY.md

Flag these as separate kaizen items in the Phase 4/5 format if any are found.

---

## Appendix: Kaizen Log Schema

This section defines the JSON format used by the kaizen log. The log is updated incrementally during Phases 0, 5, and 6 — this is not a separate execution step.

**`total_resolved` is always computed, never manually tracked.** Before writing the log, set `total_resolved = items.filter(i => i.status === "resolved").length`.

```json
{
  "last_run": "2026-04-03T14:30:00Z",
  "session_count": 1,
  "total_resolved": 0,
  "items": []
}
```

**Fresh log** (when no file exists): `{ "last_run": null, "session_count": 0, "total_resolved": 0, "items": [] }`. The session_count is then incremented in Phase 0.2, so the first session is #1.

Each item in the items array:

```json
{
  "id": "short-kebab-id",
  "description": "One-line description of the issue",
  "priority": "critical|important|nit",
  "guide_ref": "Section X: Name",
  "status": "found|resolved|deferred|skipped",
  "source": "config_scan|behavioral",
  "date_found": "2026-04-03",
  "date_resolved": null,
  "date_deferred": null,
  "date_skipped": null,
  "defer_until": null,
  "session_found": 1,
  "session_resolved": null
}
```

Item IDs must be unique across all items in the log. Use short kebab-case descriptors (e.g., `gitignore-settings-local`, `no-dangerous-command-hook`).

**Log hygiene:** If the items array exceeds 50 entries, archive resolved items older than 90 days to `kaizen-log-archive.json` in the same directory.

---

## Manual Overrides

```
/kaizen              — Normal kaizen cycle
/kaizen --full       — Ignore log filters, recheck everything (including resolved)
/kaizen --log        — Show session history and item statuses, then stop
/kaizen --reset      — Set all deferred/skipped items back to "found", then run normal cycle
/kaizen --behavioral — Force behavioral scan regardless of session count
```

Handle by checking `$ARGUMENTS` for the flag in Phase 0.
