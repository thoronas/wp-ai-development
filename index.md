---
title: WP AI Development Workflow
last_updated: 2026-05-22
---

The end-to-end workflow for AI-assisted WordPress development using Claude Code and the [WP Scaffold](https://github.com/thoronas/wp-claude-scaffolding). Covers first-time machine setup, project bootstrapping (new and existing), and daily session habits.

---

## Part 1 — Global Installation (Once Per Machine)

This step installs the skills, agents, and rules into `~/.claude/` so they are available in every project automatically. Do this once when you first clone `wp-scaffold-global`.

### 1. Clone wp-scaffold-global

```bash
cd ~/Sites
git clone https://github.com/thoronas/wp-scaffold-global.git
```

### 2. Run install.sh

```bash
cd ~/Sites/wp-scaffold-global
chmod +x install.sh
./install.sh
```

The script symlinks each skill subdirectory, agent file, and rule file from the repo into `~/.claude/`. Output should look like:

```text
Skills:
  ✓ skills/blueprint
  ✓ skills/wordpress-router
  ✓ skills/wp-abilities-api
  ✓ skills/wp-block
  ✓ skills/wp-debug
  ✓ skills/wp-feature
  ✓ skills/wp-interactivity-api
  ✓ skills/wp-migrate
  ✓ skills/wp-performance
  ✓ skills/wp-phpstan
  ✓ skills/wp-playground
  ✓ skills/wp-plugin-directory-guidelines
  ✓ skills/wp-project-triage
  ✓ skills/wp-review
  ✓ skills/wp-theme
  ✓ skills/wp-wpcli-and-ops
  ✓ skills/wpds
Agents:
  ✓ agents/performance-profiler.md
  ✓ agents/security-auditor.md
  ✓ agents/wpcs-enforcer.md
Rules:
  ✓ rules/behavioral-standards.md
  ✓ rules/js-standards.md
  ✓ rules/php-standards.md
  ✓ rules/template-standards.md
  ✓ rules/test-standards.md
```

### 3. Verify

```bash
ls ~/.claude/skills/
# blueprint  wordpress-router  wp-abilities-api  wp-block  wp-debug  wp-feature
# wp-interactivity-api  wp-migrate  wp-performance  wp-phpstan  wp-playground
# wp-plugin-directory-guidelines  wp-project-triage  wp-review  wp-theme
# wp-wpcli-and-ops  wpds

ls ~/.claude/rules/
# behavioral-standards.md  js-standards.md  php-standards.md
# template-standards.md  test-standards.md
```

### Keeping skills up to date

Because `install.sh` creates symlinks (not copies), any edit to a file in `~/Sites/wp-scaffold-global/` is picked up by `~/.claude/` immediately — no re-running needed. Only run `install.sh` again if you add a new skill, agent, or rule file.

---

## Part 2 — Bootstrapping an Existing Project (Drop-in Mode)

Use this when adding Claude Code to a project that is already in development. Works regardless of repo structure — the target can be a wp-content root, a single theme directory, or a themes directory.

### What you need to know first

Before running the script, have these ready:

| Value           | Example                 | What it's for                                          |
| --------------- | ----------------------- | ------------------------------------------------------ |
| Target path     | `/path/to/your-project` | Absolute path to the repo root                         |
| Theme slug      | `clf8`                  | Matches your existing `themes/clf8/` directory         |
| Plugin slug     | `clf8-plugin`           | Matches your existing `plugins/clf8-plugin/` directory |
| Composer vendor | `clarifyfirst`          | Used in `composer.json` `name` field                   |
| PHP namespace   | `ClarifyFirst`          | PSR-4 autoload namespace prefix                        |

### Step 1 — Clone wp-scaffold-project

```bash
cd ~/Sites
git clone https://github.com/thoronas/wp-claude-scaffolding.git
cd wp-scaffold-project
```

### Step 2 — Run bin/init.sh in drop-in mode

```bash
bin/init.sh \
  --mode dropin \
  --target /path/to/your-project \
  --theme your-theme-slug \
  --plugin your-plugin-slug \
  --vendor your-vendor \
  --namespace YourNamespace
```

Or run without flags and answer the interactive prompts:

```bash
bin/init.sh
# Choice [1/2]: 1  (drop-in)
# Target directory: /path/to/your-project
# ...
```

**What the script copies into the target:**

- `.claude/` — settings.json (pre-configured permissions) and reference directory
- `CLAUDE.md` — project memory template, pre-filled with your theme/plugin slugs
- `PROJECT-SPEC.md` — feature specification template
- `DECISIONS.md` — architectural decision log template
- `.editorconfig`, `.mcp.json` — editor and MCP config
- `composer.json` — PHP tooling (WPCS, PHPUnit, PSR-4 autoload) with your namespace filled in
- `phpcs.xml.dist`, `phpunit.xml.dist` — WPCS and test config with your text domains filled in
- `docs/DEVELOPMENT-PROMPTS.md` — prompt reference library

**What the script does NOT touch:** your existing theme, plugin, `.gitignore` (it only appends the Claude-specific block), or any other code.

### Step 3 — Review the placeholder warning

The script runs a post-copy grep and warns about any placeholder strings it couldn't substitute automatically. Review the listed files — most will be documentation examples, not real gaps.

### Step 4 — Fill in the three context files

These are the files Claude reads to understand your project. Open each one and complete it:

**`CLAUDE.md`** — the highest-leverage file. Loaded every session automatically.

- Project name and one-sentence description
- "Current Focus" — 1–2 sentences on what you're working on right now
- "Known Issues / Gotchas" — project-specific traps Claude should avoid
- Keep it under 800 words. Detail goes in `PROJECT-SPEC.md`, not here.

**`PROJECT-SPEC.md`** — loaded on demand when you reference it.

- List your existing features and data model
- Add planned features as you identify them
- Claude reads this when you say "Read PROJECT-SPEC.md, Feature N. Build it."

**`DECISIONS.md`** — loaded on demand.

- Document architectural choices that have already been made
- Prevents Claude from proposing alternatives you've already ruled out
- Example: "We use CPT for team members, not ACF Repeater — see DECISIONS.md"

### Step 5 — Update .claude/settings.json permission globs

The default settings.json scopes read/write permissions to `themes/**` and `plugins/**`. If your repo root is a theme directory (not a wp-content directory), update the globs to match:

```json
"allow": [
  "Read(**)",
  "Write(src/**)",
  "Write(inc/**)",
  "Write(templates/**)"
]
```

Match the paths to where your actual code lives.

### Step 6 — Run composer install

```bash
cd /path/to/your-project
composer install
```

Installs WPCS, PHPUnit, and the Composer autoloader. Verify with `composer phpcs` — it should report zero violations on a fresh codebase, or surface real issues to address.

### Step 7 — Commit the scaffold files

```bash
git add CLAUDE.md PROJECT-SPEC.md DECISIONS.md .claude/ .editorconfig .mcp.json \
        composer.json composer.lock phpcs.xml.dist phpunit.xml.dist docs/
git commit -m "chore: add Claude Code scaffold (wp-scaffold-project drop-in)"
```

The project is now Claude-ready. Open Claude Code in the project directory and start a session.

---

## Part 3 — Bootstrapping a New Project (Fresh Build Mode)

Use this when starting a project from scratch. Fresh build mode copies the Claude layer plus the full WordPress scaffold structure, then optionally generates a complete theme skeleton via the `/wp-theme` skill.

### Before you start

Same values as drop-in mode, plus a decision about the theme:

- Do you want Claude to generate the full theme skeleton now (FSE or hybrid)?
- If yes, be ready to describe the theme when `/wp-theme` launches — layout intent, whether it needs classic PHP parts, block patterns needed, etc.

### Step 1 — Create the target directory and initialize git

```bash
mkdir -p ~/Sites/your-project
cd ~/Sites/your-project
git init
```

### Step 2 — Run bin/init.sh in fresh mode

```bash
cd ~/Sites/wp-scaffold-project

bin/init.sh \
  --mode fresh \
  --target ~/Sites/your-project \
  --theme your-theme-slug \
  --plugin your-plugin-slug \
  --vendor your-vendor \
  --namespace YourNamespace
```

The script will ask: `Generate theme skeleton via wp-theme skill? [Y/n]:`

- **Y** — Claude Code launches immediately in the target directory and runs `/wp-theme` interactively. Have your theme requirements ready.
- **n / `--no-theme`** — skip for now; run `/wp-theme` manually later from inside the project.

**What gets copied into the target (in addition to the Claude layer):**

- `plugins/your-plugin/` — full plugin skeleton, renamed and pre-filled with your slug, namespace, and constants
- `mu-plugins/`, `languages/`, `tests/Unit/`, `tests/Integration/` — directory structure with `.gitkeep` placeholders
- `.wp-env.json` — local WordPress environment config, pointing to your theme and plugin
- `.gitignore` — comprehensive WordPress `.gitignore` including the Claude-specific entries

### Step 3 — Fill in the context files

Same as drop-in mode (see Part 2, Step 4). For a new project, `CLAUDE.md`'s "Current Focus" should be the very first thing you're building.

### Step 4 — Review the plugin bootstrap file

The plugin file at `plugins/your-plugin/your-plugin.php` has substitutable fields already filled in (text domain, constants, namespace) and fields that require manual completion:

- `Plugin Name:` — human-readable display name
- `Author:` and `Author URI:`
- `Plugin URI:`
- `Description:`

Open the file and fill these in before doing any other work.

### Step 5 — Run composer install and start wp-env

```bash
cd ~/Sites/your-project
composer install
npx wp-env start
```

`wp-env` spins up a local WordPress instance with your theme and plugin activated. Visit `http://localhost:8888` to confirm the environment is up.

### Step 6 — If you skipped the theme, generate it now

Open Claude Code in the project directory:

```bash
claude
```

Then run:

```text
/wp-theme
```

The skill walks through a series of questions (FSE or hybrid, layout structure, patterns needed, etc.) and generates a complete theme skeleton conforming to WPCS.

### Step 7 — Initial commit

```bash
git add -A
git commit -m "feat: initial project scaffold (wp-scaffold-project fresh build)"
```

---

## Part 4 — Daily Session Workflow

### Starting a session

1. **Update "Current Focus" in `CLAUDE.md`** — 1–2 sentences on what this session is for. This is the single highest-impact habit.
2. Open Claude Code in the project directory: `claude` or via the IDE extension.
3. `CLAUDE.md` is auto-loaded. Global skills, agents, and rules are already available from the machine install.

### Starting a feature

```text
Update Current Focus in CLAUDE.md
→ Open Claude Code
→ "Read PROJECT-SPEC.md, Feature N. Then build it."
→ Claude auto-invokes /wp-feature
→ Triage phase — Claude checks for signals (interactive UI, performance risk,
   new block, Playground demo needed, WP.org submission, etc.) and asks whether
   to invoke a relevant extended skill first. Simple features skip triage automatically.
→ Review the plan
→ Approve → Claude implements with WPCS enforcement
→ composer phpcbf && composer phpcs (auto-fixed + zero violations)
→ Run tests
→ Commit
```

**Shorthand for iteration:**

| Situation | Say |
| --------- | --- |
| Mostly right, one issue | "Revise only [part], the rest is correct" |
| Wrong assumption | "You assumed [X], actually [Y]. Here's the code." |
| Continue large implementation | "Plan approved. Implement [next file]." |
| Doesn't work | "Here's the error: [paste]. Diagnose and correct." |
| Explore alternative | "Propose alternative for [part] prioritizing [tradeoff]." |

### Session handoffs

```text
"Write a handoff note to Current Focus in CLAUDE.md."
```

The next session picks up automatically. Replaces manual context-pasting at session start.

### Using reference material

1. Drop file into `.claude/reference/[category]/`
2. Point Claude at it: `"Use the pattern in .claude/reference/plugins/acf-example.php for our settings page."`
3. Reference files are not auto-loaded — always pointed at explicitly to keep context lean.
4. Strip `.git` from any cloned reference repos before committing: `rm -rf .claude/reference/repo/.git`

### Running a full WPCS audit

```text
"Run a full WPCS audit on the plugin."
```

→ `wpcs-enforcer` agent  
→ `phpcbf` (auto-fix)  
→ `phpcs` (check remaining)  
→ Manual fixes for unfixable violations  
→ Report

### Debugging

```text
"Here's the error: [paste]. Diagnose and correct."
```

→ `/wp-debug` skill auto-invokes  
→ Traces across theme and plugin packages  
→ Proposes fix with WPCS validation

### Available skills — quick reference

#### Core development

| Skill | When to invoke |
| ----- | -------------- |
| `/wp-feature` | Add a feature, settings page, post type, or REST endpoint — includes a triage phase that routes to extended skills as needed |
| `/wp-block` | Create a Gutenberg block from scratch |
| `/wp-debug` | Bug, error, or unexpected behavior |
| `/wp-migrate` | WP/PHP version upgrades, deprecation replacements |
| `/wp-review` | Security and WPCS code review before merge |
| `/wp-theme` | Generate a full theme skeleton — FSE or hybrid |

#### Extended skills

| Skill | When to invoke |
| ----- | -------------- |
| `/blueprint` | Create or edit WordPress Playground blueprint JSON files |
| `/wordpress-router` | Classify a WordPress repo and route to the right workflow |
| `/wp-abilities-api` | Work with the WordPress Abilities API and REST exposure |
| `/wp-interactivity-api` | Build or debug Interactivity API features (`data-wp-*` directives) |
| `/wp-performance` | Profile and optimize queries, caching, cron, and HTTP calls |
| `/wp-phpstan` | Configure and run PHPStan static analysis |
| `/wp-playground` | Spin up disposable WP instances locally or in-browser |
| `/wp-plugin-directory-guidelines` | Review a plugin for WordPress.org compliance |
| `/wp-project-triage` | Inspect a repo and produce a structured diagnostic report |
| `/wp-wpcli-and-ops` | WP-CLI operations: search-replace, db, cron, cache, multisite |
| `/wpds` | Build UIs using the WordPress Design System |

---

## Part 5 — Development Prompt Library

The scaffold ships a prompt reference library at `docs/DEVELOPMENT-PROMPTS.md`. It contains 17 structured prompt templates covering the full range of WordPress development tasks.

### Skills vs. prompts — when to use which

| Situation | Use |
| --------- | --- |
| Inside Claude Code on a project with scaffold files | Skills (`/wp-feature`, `/wp-debug`, `/wp-performance`, etc.) — auto-load project context |
| Outside Claude Code (Claude.ai, another AI tool, API) | DEVELOPMENT-PROMPTS.md — self-contained, no project context assumed |
| Need tight control over exactly what's requested | DEVELOPMENT-PROMPTS.md — fill every field explicitly |
| Quick iteration on a known problem | Skills — faster to invoke, context already loaded |

### What's in the library

| Prompt | When to reach for it |
| ------ | -------------------- |
| WordPress Feature Builder | New feature in an existing plugin or theme — the everyday workhorse |
| Existing Codebase Feature Integration | Adding to a codebase you haven't fully mapped yet |
| Production Bug Investigation | Structured diagnosis with stack trace, environment, reproduction steps |
| Performance Investigation | N+1 queries, slow admin screens, caching gaps |
| Codebase Audit + Targeted Refactor | Before a major change — understand what's there first |
| WordPress: Custom Block Builder | Full block with `block.json`, edit/save components, PHP render callback |
| WordPress: WP-CLI Command | CLI command with argument validation and output formatting |
| Code Review (PR/Diff Review) | Structured multi-perspective review before merge |
| Migration + Upgrade | PHP version bumps, WP major versions, deprecated API replacement |
| Multi-Perspective Engineering Review | Architecture decisions — gets security, performance, and maintainability angles |
| Documentation Generator | Generate inline docs, README sections, or hook reference from existing code |

### How to use them

Every prompt follows the same three-phase pattern:

1. **Context in** — fill every bracketed field before sending. "I don't have profiling data yet" is more useful than leaving a field blank.
2. **Output out** — each prompt specifies its required deliverables so the model knows what format to produce.
3. **Iterate** — each prompt ends with a checklist of what to verify first and how to feed corrections back in.

For large implementations: `"Produce the full implementation plan with file-by-file specifications first. Then implement the highest-priority file. I will ask for subsequent files one at a time."`

---

## Part 6 — Adding a New Global Skill

1. Create `~/Sites/wp-scaffold-global/skills/[name]/SKILL.md`
2. Run `~/Sites/wp-scaffold-global/install.sh` once to create the symlink
3. The skill is immediately available in all projects — no changes to any project repo

To update an existing skill, edit the file directly. The symlink means `~/.claude/` reflects the change instantly.

---

## Key Guardrails

The scaffold's `settings.json` pre-denies at the tool-permission level — not relying on model judgment:

**Denied reads:** `.env`, `.env.*`, `wp-config.php`, `credentials*`, `secrets*`  
**Denied commands:** `rm -rf *`, `curl *`, `wget *`, `ssh *`, `scp *`

These blocks apply in every project that uses the scaffold's `settings.json` without any per-session configuration needed.

### Global Behavioral Rules

`rules/behavioral-standards.md` (active in every project via `~/.claude/rules/`) enforces four principles derived from Karpathy's LLM coding guidelines:

| Rule | What it means in practice |
| ---- | ------------------------- |
| **Think before coding** | State assumptions explicitly; surface ambiguity before building |
| **Simplicity first** | Minimum code that solves the problem — no speculative abstractions |
| **Surgical changes** | Touch only what the request requires; mention unrelated issues rather than fixing them |
| **Goal-driven execution** | Transform tasks into verifiable goals with a test or check per stage |
