---
title: Changelog
permalink: /changelog/
last_updated: 2026-07-25
---

Notable changes to the WP AI Development Workflow documentation and the scaffold it describes.

This file records changes that **require action on your machine or in your projects**, not just documentation edits. Anything under "Action required" affects setups created before that date.

---

## 2026-07-25 — Harness audit

A full audit of `wp-scaffold-global` and the project scaffold against current model guidance. Two live defects were found, both invisible to a read-through review and both found only by executing the harness.

### Action required

**1. Replace `Write(...)` permission rules with `Edit(...)` in every project.**

Claude Code matches file-write permissions against `Edit(path)` rules only. A `Write(path)` rule never matches — it silently does nothing, so you keep getting permission prompts for paths you believed were pre-approved. The scaffold template shipped four such rules, so **every project bootstrapped before this date is affected**.

```bash
grep '"Write(' .claude/settings.json    # any output = dead rules
```

Change `Write(` to `Edit(` for `themes/**`, `plugins/**`, `mu-plugins/**`, `tests/**`. Earlier versions of this documentation also taught the broken form in Part 2, Step 5 — now corrected.

**2. Re-run `install.sh` on every machine, not just `git pull`.**

`install.sh` now prunes symlinks whose target has been deleted upstream. Two rules were removed in this release; without the prune step they leave dangling symlinks in `~/.claude/rules/`.

```bash
cd ~/Sites/wp-scaffold-global && git pull && ./install.sh
```

**3. Verify your agents actually register.**

```bash
claude -p "List every subagent type available to you via the Agent tool. \
Output ONLY their exact names, one per line. Do not use any tools." 2>&1 | tail -30
```

If `wpcs-enforcer`, `security-auditor`, and `performance-profiler` do not appear, your local agent files predate the frontmatter fix — pull and re-run `install.sh`.

### Fixed

- **Three agents were silently unregistered for six weeks.** `wpcs-enforcer`, `security-auditor`, and `performance-profiler` had no YAML frontmatter, so Claude Code never registered them despite the symlinks resolving correctly. Anything delegating to them had been quietly falling back to inline work in the main session — meaning the generator/evaluator separation the scaffold is built around was not actually running. Fixed by adding `name`, `description`, and `tools` frontmatter.
- **Four dead permission rules** in the project template and in every project derived from it (see Action required above).

### Changed

- **`security-auditor` is now read-only** (`Read, Grep, Glob, Bash`). Its own instructions say to report findings and show corrected code rather than apply it, so a security review can no longer silently rewrite the code it is reviewing.
- **`install.sh` prunes deleted entries** in addition to creating links, so deletions propagate across machines.
- **Path-scoped rules replace `**`-globbed ones.** No rule is now loaded unconditionally.

### Removed

- **`rules/behavioral-standards.md`** (400 words, globbed `**`). Every instruction had stopped earning its place: `<use_parallel_tool_calls>` was an exact duplicate of Claude Code's own system prompt; "do not add docstrings, comments, or type annotations to code you did not change" **directly contradicted** it, since the current system prompt instructs matching the surrounding code's comment density; "match existing style" had become default behaviour; and the remainder was already stated, with WordPress specifics, in the project `CLAUDE.md`. The one part with independent value, `<investigate_before_answering>`, survives as a path-scoped rule.
- **`rules/workflow-improvement.md`** (904 words, globbed `**`). Duplicated the `/workflow-improvement` skill, which was already installed and invocable. Its only unique content was a hand-maintained cache of reference material that had already gone stale.

**Net effect: context loaded before any task fell from ~3,032 words to ~1,642 (−46%).** A non-WordPress session now loads no scaffold rules at all.

### Added

- **`bin/audit-drift.sh`** — reports two kinds of silent divergence: between `skills/` and the vendor-neutral `portable-skills/` fork, and between the project template and already-bootstrapped projects. It reports rather than reconciles, because some divergence is deliberate; intentional cases go in `.sync-allowlist` with a required reason.
- **`rules/investigate-before-answering.md`** — path-scoped to PHP and JS under `plugins/`, `mu-plugins/`, `themes/`.
- **Part 7 — Verification and Maintenance** in the documentation: the smoke test, the drift audit, and the per-release harness review.
- **Agents section** documenting all three agents, their tool scopes, and the frontmatter requirement.
- **`/workflow-improvement`** added to the skills reference — it had been installed but never documented here.

### Known gaps

Tracked and not yet addressed:

- **`CLAUDE.md` is 1,642 words against its own documented 800-word limit.** It also conditionally mandates applying `docs/DEVELOPMENT-PROMPTS.md`, a 4,879-word library — an effectively mandatory read of a large file, which is what progressive disclosure exists to avoid.
- **No propagation path from template to existing projects.** `CLAUDE.md` and `.claude/settings.json` are copied at bootstrap and never sync. `bin/audit-drift.sh` makes the drift visible but does not fix it; template fixes still have to be applied per project by hand.
- **Skills with heavy few-shot examples** (`blueprint` at 1,943 words, `wp-theme` at 1,806) may over-constrain current models, which do better with well-designed interfaces than with worked examples. Flagged by size, not yet verified by inspection.
- **`/doctor` has not been run** against a scaffold-bootstrapped project to cross-check this list.

---

## 2026-05-22 — Initial release

- Initial GitHub Pages site for the WP AI Development Workflow
- Covers global installation, drop-in and fresh-build bootstrapping, daily session workflow, the 17-prompt development library, and adding new global skills
