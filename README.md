# 改善 Kaizen for CC

A self-improving audit system for your Claude Code setup. Run `/kaizen` and it scans your configuration, surfaces actionable improvements, executes fixes on approval, and tracks progress across sessions.

It gets quieter as your setup improves. That is the point.

## Why "Kaizen"?

[Kaizen](https://en.wikipedia.org/wiki/Kaizen) (改善, Japanese for "improvement") is a philosophy originating from post-war Japanese manufacturing — most famously the [Toyota Production System](https://en.wikipedia.org/wiki/Toyota_Production_System). The core idea: instead of large, disruptive overhauls, make small, continuous improvements as part of daily work. Every person in the process is empowered to identify waste and fix it incrementally. Over time, these small changes compound into transformative results. This tool applies that same principle to your Claude Code setup.

## What it does

Each `/kaizen` session:

1. **Discovers** your Claude Code configuration using native tools (Read, Glob, Grep) — config files, hooks, permissions, sandboxing, MCP servers, commands, agents, skills, rules, memory, auto-mode settings
2. **Analyzes** against best practices — security hooks, permissions, sandboxing, CLAUDE.md quality, slash commands, MCP setup, rules, context optimization, hook coverage across all event types
3. **Filters** against a persistent log — resolved items never resurface, skipped items return after 3 days, deferred after 7
4. **Presents** up to 5 actionable findings with guide references — shared config items flagged with 🔴 warning
5. **Executes** fixes on your approval — creates files, writes configs, adds hooks
6. **Tracks** everything in a session log outside your repo

Optionally, it also runs **behavioral analysis & user coaching** — analyzing your memory files across five dimensions to identify anti-patterns, reinforce good practices, suggest automations, flag config gaps, and coach on interaction style. It then offers to implement the suggestions on the spot.

## Install

Copy `kaizen.md` into your project's `.claude/commands/` directory:

```bash
mkdir -p .claude/commands
curl -o .claude/commands/kaizen.md https://raw.githubusercontent.com/cmAIdx/kaizen/main/kaizen.md
```

That's it. Run `/kaizen` in your next Claude Code session.

## Usage

```
/kaizen              — Normal kaizen cycle
/kaizen --full       — Ignore log filters, recheck everything (including resolved)
/kaizen --log        — Show session history and item statuses (read-only, no side effects)
/kaizen --reset      — Clear deferred/skipped items, re-surface everything
/kaizen --behavioral — Force behavioral analysis regardless of session count
```

Flags can be combined: `/kaizen --full --behavioral` rechecks everything and includes behavioral analysis.

When findings are presented, respond with:
- **y** — execute the fix
- **skip** — skip this item (resurfaces after 3 days)
- **later** / **defer** — defer this item (resurfaces after 7 days)
- **done** / **stop** — end the kaizen session
- **anything else** — treated as a question about the current item

## How it evolves

| Sessions | What happens |
|---|---|
| 1 | First scan. Likely finds security and config basics. |
| 2-4 | Resolved items filtered out. List shrinks each run. |
| 5 | First prompted behavioral analysis. Usage coaching report. |
| 6+ | Mostly workflow and polish items left. |
| Eventually | "All clear." Straight to work. |

## What it checks

**Security (Critical):**
- Missing PreToolUse hooks for secrets and dangerous commands
- Weak or missing permissions config
- `.claude/settings.local.json` not gitignored
- Hardcoded secrets in CLAUDE.md or MCP config
- Read/Edit deny rules without matching Bash guards (subprocess bypass gap)
- No OS-level sandboxing on projects with sensitive data
- Stale or incomplete auto-mode environment/soft_deny configuration

**Workflow (Important):**
- Missing slash commands for detected workflows
- No custom agents for repeated tasks
- No PostToolUse hooks for auto-formatting/linting
- No `.claude/rules/` directory for composable conventions
- CLAUDE.md over 200 lines (context bloat)
- No Stop hook quality gate
- No SessionStart/PostCompact hooks for context re-injection

**Context Optimization (Important):**
- `.claude/rules/` files without `paths:` frontmatter (always loaded vs. on-demand)
- CLAUDE.md content that could be path-scoped rules instead
- No context management patterns documented

**Polish (Nice to Have):**
- MCP servers that would benefit the tech stack
- Accumulated junk permissions from auto-approvals
- Command/tool descriptions not optimized for Claude's tool selection
- No Notification hook for desktop alerts during unattended sessions

**Behavioral Analysis (on request or every 5th session):**
- Anti-patterns: recurring corrections, manual steps, context waste
- Good practices: what's working well (positive reinforcement)
- Automation opportunities: concrete hook, command, and agent suggestions
- Configuration gaps: path-scoped rules, permissions, MCP, auto-mode
- Interaction coaching: prompting style, `/clear` usage, `/btw`, effort levels

## Design decisions

- **Manual, not automatic.** Runs when you want it, not on every session start. No wasted tokens.
- **Native tools, not bash.** Uses Read/Glob/Grep instead of shell commands — more reliable, better UX, cross-platform.
- **Log stored outside the repo.** Kaizen tracking lives in `~/.claude/projects/...`, not in your git history.
- **Actionable items only.** No scores, no dashboards. Just a list of things worth fixing and the ability to fix them now.
- **Session impact warnings.** When a fix modifies shared config, kaizen flags it with 🔴 and offers skip — changes to shared config affect other active Claude sessions.
- **Secrets never exposed.** Finding descriptions reference file:line only, never actual secret values.
- **Idempotent fixes.** Before executing any fix, verifies the issue still exists — prevents double-application from interrupted sessions.
- **Coaching, not just auditing.** Behavioral analysis doesn't just find problems — it reinforces what's working, suggests concrete automations, and offers to implement them.
- **Log hygiene.** Auto-archives resolved items >90 days when the log exceeds 50 entries. Computed totals (never manually tracked counters).

## Attribution

This tool draws from and is informed by:

| Source | Author / Org | URL |
|--------|-------------|-----|
| The Ultimate Claude Code Guide | Florian Bruniaux | [GitHub](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) |
| Claude Code Official Docs | Anthropic | [code.claude.com](https://code.claude.com/docs/en/overview) |
| Claude Code Mastery | TheDecipherist | [GitHub](https://github.com/TheDecipherist/claude-code-mastery) |
| Awesome Claude Code Workflows | ithiria894 | [GitHub](https://github.com/ithiria894/awesome-claude-code-workflows) |
| Claude Code Examples | Anthropic | [GitHub](https://github.com/anthropics/claude-code/tree/main/examples) |

The discovery and analysis methodology was originally adapted from the [audit prompt](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/tools/audit-prompt.md) in the Ultimate Claude Code Guide. Kaizen wraps that methodology into a persistent, incremental improvement loop with behavioral coaching — so the value compounds across sessions instead of being a one-shot scan.

## License

MIT
