# 改善 Kaizen for Claude Code

A self-improving audit system for your Claude Code setup. Run `/kaizen` and it scans your configuration, surfaces actionable improvements, executes fixes on approval, and tracks progress across sessions.

It gets quieter as your setup improves. That is the point.

## What it does

Each `/kaizen` session:

1. **Discovers** your Claude Code configuration using native tools (Read, Glob, Grep)
2. **Analyzes** against best practices — security hooks, permissions, CLAUDE.md quality, slash commands, MCP setup, rules
3. **Filters** against a persistent log — resolved items never resurface, skipped items return after 3 days, deferred after 7
4. **Presents** up to 5 actionable findings with guide references
5. **Executes** fixes on your approval — creates files, writes configs, adds hooks
6. **Tracks** everything in a session log outside your repo

Every 5th session it also runs **behavioral pattern detection** — analyzing your memory files for repeated corrections that should become rules, frequent workflows that should become commands, and missing hooks suggested by your usage patterns.

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
/kaizen --full       — Ignore log, recheck everything
/kaizen --log        — Show session history and item statuses
/kaizen --reset      — Clear deferred/skipped, re-surface everything
/kaizen --behavioral — Force behavioral scan regardless of session count
```

When findings are presented, respond with:
- **yes** — execute the fix
- **skip** — skip this item (resurfaces after 3 days)
- **later** / **defer** — defer this item (resurfaces after 7 days)
- **done** / **stop** — end the kaizen session

## How it evolves

| Sessions | What happens |
|---|---|
| 1 | First scan. Fix what matters, skip the rest. |
| 2-4 | Resolved items filtered out. List shrinks each run. |
| 5 | First behavioral scan. Repeated corrections become rules. |
| 6+ | Mostly workflow and polish items left. |
| Eventually | "All clear." Straight to work. |

## What it checks

**Security (Critical):**
- Missing PreToolUse hooks for secrets and dangerous commands
- Weak or missing permissions config
- `.claude/settings.local.json` not gitignored
- Hardcoded secrets in CLAUDE.md or MCP config

**Workflow (Important):**
- Missing slash commands for detected workflows
- No custom agents for repeated tasks
- No PostToolUse hooks for auto-formatting
- No `.claude/rules/` directory for composable conventions
- CLAUDE.md over 200 lines (context bloat)

**Polish (Nice to Have):**
- MCP servers that would benefit the tech stack
- Accumulated junk permissions from auto-approvals
- Command descriptions not optimized for Claude's tool selection

## Design decisions

- **Manual, not automatic.** Runs when you want it, not on every session start. No wasted tokens.
- **Native tools, not bash.** Uses Read/Glob/Grep instead of shell commands — more reliable, better UX, cross-platform.
- **Log stored outside the repo.** Kaizen tracking lives in `~/.claude/projects/...`, not in your git history.
- **Actionable items only.** No scores, no dashboards. Just a list of things worth fixing and the ability to fix them now.
- **Session impact warnings.** When a fix modifies shared config, kaizen warns that other active Claude sessions may be affected.

## Attribution

The discovery and analysis methodology is adapted from the audit prompt in **The Ultimate Claude Code Guide** by [Florian Bruniaux](https://github.com/FlorianBruniaux):

- [Audit Prompt](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/tools/audit-prompt.md)
- [Full Guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/ultimate-guide.md)

Kaizen wraps the audit methodology into a persistent, incremental improvement loop — so the value compounds across sessions instead of being a one-shot scan.

## License

MIT
