# Cybervaldez Playbook - Upgrade Guide

> For existing projects that want structured AI-assisted development without starting over.
>
> **What you'll get:** 14 skills layered on top of your existing codebase — planning, building, testing, and consultation. Your code stays untouched. The playbook adds structure around your development process.

---

## CRITICAL CONSTRAINTS

- Do NOT modify existing project files (code, configs, tests)
- Do NOT restructure the project's directory layout
- Do NOT replace existing tooling (test frameworks, CI/CD, linters)
- The playbook layers ON TOP of existing work — it does not replace it
- Best suited for alpha/prototype-stage projects — be honest if the project is too mature
- Playbook files (UPGRADE.md, README.md, LICENSE, skills/) are moved into `playbook/` after upgrade

---

## Pre-Flight Check

Before proceeding, check if `.claude/skills/research/SKILL.md` exists at the current directory.

If it does, **STOP** — this is already an upgraded/kickstarted project or the playbook source repo.

Tell the user:
> "This directory already has skills installed (`.claude/skills/research/SKILL.md` found). UPGRADE is for projects that haven't adopted the playbook yet."

---

## Step 1: Project Scan

**Automatic detection — no user input needed yet.**

### 1.1: Detect Project Type

```bash
# Detect project type from files
ls package.json next.config.* vite.config.* app.py requirements.txt manage.py Cargo.toml go.mod 2>/dev/null

# Scan structure
find . -maxdepth 3 -type f | head -100
ls -la

# Count complexity signals
find . -type f -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/venv/*' | wc -l
```

Map to project type:

| Files Found | Project Type |
|-------------|-------------|
| `next.config.*` | `nextjs-with-cli` |
| `vite.config.*` + React in dependencies | `react-with-cli` |
| `app.py` or `requirements.txt` with `webui/` | `python-cli-with-webui` |
| None of above | `unsupported` |

**If `unsupported`:** **HARD STOP.** Tell the user:

> "The playbook is designed for `python-cli-with-webui`, `nextjs-with-cli`, and `react-with-cli` project types. Your project doesn't match any of these. The playbook's skills, testing patterns, and conventions won't apply correctly to your stack."

Do not proceed. Do not offer workarounds.

### 1.2: Detect Existing Infrastructure

Also detect and record:

| Signal | How to Detect |
|--------|---------------|
| Test framework | Look for jest.config.*, vitest.config.*, pytest.ini, conftest.py, etc. |
| Existing `.claude/` folder | `ls -la .claude/ 2>/dev/null` |
| CI/CD pipelines | `ls .github/workflows/ .gitlab-ci.yml Jenkinsfile 2>/dev/null` |
| Package dependencies | Parse `package.json` dependencies or `requirements.txt` |
| Git status | `git status --porcelain` |
| Git history depth | `git rev-list --count HEAD 2>/dev/null` |
| Total file count | Count from above |
| Dependency count | `jq '.dependencies + .devDependencies | length' package.json 2>/dev/null` or `wc -l < requirements.txt 2>/dev/null` |

### 1.3: Gate Checks

**Dirty git working tree:** If `git status --porcelain` returns output, **ABORT:**

> "Your working tree has uncommitted changes. Please commit or stash your changes before running the upgrade. This ensures the backup in Step 4 captures a clean state."

**No git repo:** If `git rev-list --count HEAD` fails, **WARN** but don't block:

> "This directory is not a git repository. The upgrade will proceed, but some skills use `git diff` to track changes. You can run `git init` later."

**Existing `.claude/skills/`:** If `.claude/skills/` exists but `.claude/skills/research/SKILL.md` does not (pre-flight passed), **WARN:**

> "Found existing `.claude/skills/` directory. The upgrade will add playbook skills to this directory, potentially overwriting files with the same names. Proceed?"

User must confirm before continuing.

---

## Step 2: Tech Stack Research

For each major technology detected in Step 1 (framework, UI library, state management, etc.):

1. Run the **full `/research` workflow** for each detected tech
2. Process techs **one at a time** so user can correct classifications individually

### Workflow per tech:

1. Web search for current best practices
2. Classify into domains per `skills/TECH_CONTEXT.md` Domain Classification Table
3. Present findings to user for confirmation (same Present & Confirm pattern as `/research` Step 4)
4. Save confirmed findings to `techs/{tech}/README.md`

### Example flow:

```
Detected technologies: Next.js, React, TypeScript, Tailwind

Researching: Next.js
{... full /research workflow ...}
Is this correct? Any corrections?

> User confirms

✓ Saved techs/nextjs/README.md

Researching: React
{... full /research workflow ...}
```

This pre-populates the research directory so skills are tech-aware from day one.

---

## Step 3: Compatibility Assessment

**Present an honest, structured assessment. Bias toward questioning adoption.**

### Complexity Indicators

Gathered from Step 1 detection:

| Signal | Value | Impact |
|--------|-------|--------|
| Total files | N | >200 = warning |
| Dependencies | N | >30 = warning |
| Existing test files | N | >0 = potential conflict |
| CI/CD pipelines | Y/N | If yes = needs manual reconciliation |
| Existing `.claude/` config | Y/N | If yes = will be merged carefully |
| Git history depth | N commits | >500 = mature project warning |

### Pros of Adopting

- Structured skill pipeline (UX → Design → Build → Test)
- Tech-aware AI assistance via research docs
- Quality gates catch issues early
- E2E testing patterns with agent-browser

### Cons / Honest Warnings

- **Testing separation:** Playbook uses E2E-only testing (bash + curl + agent-browser). If your project has mature unit/integration tests, these are SEPARATE concerns — the playbook does not replace them
- **Convention conflicts:** Playbook conventions may conflict with existing team conventions
- **Code assumptions:** Skills assume certain patterns (`data-testid`, `window.appState`, etc.) that existing code won't have
- **Maturity mismatch:** For projects past prototype stage, adaptation cost may exceed benefit
- **Python dependency:** Browser skills require Python 3.10+ even for JS projects

### Verdict

Present one of three verdicts:

```
ASSESSMENT: GOOD FIT

{2-3 sentence summary}. Low file count, few dependencies,
no existing test infrastructure to conflict with. The playbook
will add structure without friction.

Proceed with upgrade? (This will create a backup first)
```

```
ASSESSMENT: PROCEED WITH CAUTION

{2-3 sentence summary}. Existing test suite and CI/CD mean
you'll have two parallel quality systems. The playbook's E2E
tests supplement but don't replace your existing tests.

Proceed with upgrade? (This will create a backup first)
```

```
ASSESSMENT: NOT RECOMMENDED

{2-3 sentence summary}. This project has 500+ commits, 50+
dependencies, and mature CI/CD. The adaptation cost likely
exceeds the benefit at this stage.

If you still want to proceed, here's what would need attention:
- {specific concern 1}
- {specific concern 2}

Proceed anyway? (This will create a backup first)
```

**User must explicitly confirm.** If "NOT RECOMMENDED", explain what would need to change and let user decide anyway.

---

## Step 4: Backup

**Non-negotiable if proceeding. This happens before any modifications.**

```bash
# Create backup (exclude node_modules, venv, .git, build artifacts)
mkdir -p .backup
rsync -a --exclude='node_modules' --exclude='venv' --exclude='.git' \
  --exclude='dist' --exclude='build' --exclude='__pycache__' \
  --exclude='.backup' --exclude='skills' --exclude='KICKSTART.md' \
  --exclude='UPGRADE.md' --exclude='README.md' --exclude='LICENSE' \
  . .backup/

# Add .backup to .gitignore (append, don't replace)
grep -q '^\.backup/' .gitignore 2>/dev/null || echo '.backup/' >> .gitignore
```

Tell user:

```
Backup created at .backup/
Your original project files are preserved. If anything goes wrong:
  rm -rf .claude/skills playbook techs
  cp -r .backup/* .
```

---

## Step 5: Install Skills

Same skill installation as KICKSTART but with existing-project awareness.

### 5.1: Handle Existing `.claude/` Folder

```bash
# Preserve existing .claude/ contents — only add skills/ subdirectory
mkdir -p .claude/skills
```

If `.claude/settings.json` or `.claude/settings.local.json` exist, leave them untouched.

### 5.2: Copy Core Skills

```bash
cp -r skills/core/* .claude/skills/
```

This includes:
- ux-planner (universal UX principles)
- ui-planner (visual design advisor)
- ui-review (AI slop guard)
- create-task (implementation workflow)
- coding-guard (code quality)
- cli-first (observability)
- kaizen (diverse persona feedback)
- team (expert persona collaboration)

### 5.3: Copy Shared References + Research Skill

```bash
cp skills/TECH_CONTEXT.md .claude/skills/
cp skills/SKILL_INDEX.md .claude/skills/
cp -r skills/research .claude/skills/
```

### 5.4: Create techs/ Directory

If not already created from Step 2:

```bash
mkdir -p techs
# Only create techs/README.md if it doesn't exist
[ -f techs/README.md ] || cat > techs/README.md << 'EOF'
# Technologies

Research artifacts for technologies used in this project.

Run `/research {tech}` to research a technology and create reference docs for skills.

**Note:** Research happens here in your project, not in the playbook source. This ensures research has real project context (file structure, conventions, existing techs) for deterministic, useful reference docs.

## Researched Technologies

| Technology | Status | Domain | Description |
|------------|--------|--------|-------------|
EOF
```

### 5.5: Copy Browser Skills

Only for supported project types (`python-cli-with-webui`, `nextjs-with-cli`, `react-with-cli`):

```bash
cp -r skills/browser/* .claude/skills/
```

This includes:
- ux-review (visual UX verification via screenshots)
- e2e-guard (auto-generate missing E2E tests)
- e2e (full test orchestration)
- e2e-investigate (root cause analysis for failures)
- agent-browser (browser automation utility)

### 5.6: Replace Placeholders

Replace these placeholders in all skill `.md` files. Values are **detected from the existing project**, not defaults:

| Placeholder | Detection Logic |
|-------------|----------------|
| `{{PROJECT_NAME}}` | `basename "$PWD"` |
| `{{SERVER_PORT}}` | Detect from existing config: `package.json` scripts (look for `--port` flags), `app.py` (look for `PORT` env or hardcoded port), or fall back to type default (`8080` for python, `3000` for nextjs, `5173` for react) |
| `{{SERVER_START}}` | Detect from `package.json` scripts (`dev` or `start`) or existing `start.sh`. Fall back to type default (`./start.sh` for python, `npm run dev` for nextjs/react) |
| `{{VENV_PYTHON}}` | `./venv/bin/python` if `venv/` exists, else `python3` |
| `{{WEBUI_PATH}}` | First of `src/`, `webui/`, `app/`, `public/` that exists |
| `{{TESTS_PATH}}` | First of `tests/`, `test/`, `__tests__/`, `spec/` that exists, or create `tests/` |
| `{{API_BASE}}` | Detect from existing routes (look for `/api` patterns in code) or default `/api` |
| `{{OUTPUTS_PATH}}` | `public/outputs` if `public/` exists, else `outputs` |
| `{{TEST_JOB}}` | `test-fixtures` |

### 5.7: Generate `.claude/PROJECT_CONFIG.md`

**Before generating, capture the playbook source URL:**

```bash
git remote get-url origin 2>/dev/null
```

Strip any `.git` suffix (e.g., `https://github.com/user/repo.git` → `https://github.com/user/repo`). Use this as the `PLAYBOOK_SOURCE` value. If no remote exists, leave it empty.

```markdown
# Project Configuration

Generated by Cybervaldez Playbook Upgrade

## Settings

| Setting | Value |
|---------|-------|
| Project Name | PROJECT_NAME |
| Project Type | PROJECT_TYPE |
| Server Port | SERVER_PORT |
| Server Start | SERVER_START |
| API Base | API_BASE |
| WebUI Path | WEBUI_PATH |
| Tests Path | TESTS_PATH |
| Outputs Path | OUTPUTS_PATH |
| Playbook Source | PLAYBOOK_SOURCE |
| Upgraded From | {detected project description, e.g. "Next.js 14 app with 45 files, jest tests, GitHub Actions CI"} |
| Backup Location | .backup/ |

## Skills Installed

Core skills from cybervaldez-playbook are installed in `.claude/skills/`.

## Next Steps

1. Run `/ux-planner` to plan your next feature
2. Run `/create-task` to implement with tests
3. Run `/e2e` to verify everything works
```

---

## Step 6: Ensure Supporting Structure

```bash
# Ensure tests directory exists (use detected path from 5.6)
mkdir -p {{TESTS_PATH}}

# Ensure .gitignore has playbook entries (append, don't replace)
grep -q '^playbook/' .gitignore 2>/dev/null || echo 'playbook/' >> .gitignore
grep -q '^\.backup/' .gitignore 2>/dev/null || echo '.backup/' >> .gitignore
grep -q '^tests/e2e-runs/' .gitignore 2>/dev/null || echo 'tests/e2e-runs/' >> .gitignore
```

---

## Step 7: Organize Playbook Files

Move playbook source files to `playbook/` to keep the project root clean. Skills have already been copied to `.claude/skills/` in Step 5.

### 7.1: Handle README.md Conflict

The playbook's `README.md` (in the repo root alongside KICKSTART.md/UPGRADE.md) must not overwrite the project's own `README.md`.

**Detection:** Check if a non-playbook `README.md` exists at root. The playbook's README.md contains "Cybervaldez Playbook" — if the root README.md does NOT contain this string, it's the project's own README and must be preserved.

```bash
# Check if project has its own README.md (not the playbook's)
if [ -f README.md ] && ! grep -q "Cybervaldez Playbook" README.md 2>/dev/null; then
    # Project has its own README — it stays untouched
    # The playbook README was already in root from copy-paste; skip moving it
    echo "Project README.md preserved at root."
fi
```

### 7.2: Move Playbook Files

```bash
mkdir -p playbook

# Move playbook README (rename to avoid conflict)
# Only move if it's the playbook's README (contains "Cybervaldez Playbook")
if [ -f README.md ] && grep -q "Cybervaldez Playbook" README.md 2>/dev/null; then
    mv README.md playbook/PLAYBOOK_README.md
fi

# Move other playbook files
[ -f LICENSE ] && mv LICENSE playbook/LICENSE
[ -f KICKSTART.md ] && mv KICKSTART.md playbook/KICKSTART.md
[ -f UPGRADE.md ] && mv UPGRADE.md playbook/UPGRADE.md

# Move entire skills source into playbook (already copied to .claude/skills/)
[ -d skills/ ] && mv skills/ playbook/skills/
```

**Result after upgrade:**
```
PROJECT_NAME/                    # Existing project root (UNTOUCHED)
├── {existing project files}     # All original files preserved
├── techs/                       # Tech research (new)
│   └── {tech}/README.md
├── playbook/                    # Playbook files tucked away (new)
│   ├── UPGRADE.md               # This file (reference)
│   ├── KICKSTART.md             # Kickstart (reference)
│   ├── PLAYBOOK_README.md       # Playbook documentation
│   ├── LICENSE                  # Playbook license
│   └── skills/                  # Full source skills preserved
│       ├── core/
│       ├── browser/
│       └── research/
├── .backup/                     # Pre-upgrade backup (new)
└── .claude/
    ├── PROJECT_CONFIG.md        # Project config (new)
    ├── {existing .claude files}  # Preserved
    └── skills/                  # Active skills (new)
        ├── SKILL_INDEX.md
        ├── TECH_CONTEXT.md
        ├── ux-planner/
        ├── create-task/
        ├── research/
        └── ...
```

---

## Step 8: Report & Next Steps

```
UPGRADE COMPLETE: {PROJECT_NAME}

Skills installed: .claude/skills/ (14 skills)
Tech research: techs/ ({N} technologies documented)
Backup: .backup/ (original project preserved)
Playbook reference: playbook/

IMPORTANT: Restart Claude Code to pick up new skills.

Open a new terminal and restart Claude Code, then:

  /ux-planner "I want to add [feature] to {PROJECT_NAME}"
  /research {tech}          — research additional technologies
  /coding-guard             — audit existing code patterns
  /e2e                      — set up E2E testing

Your existing code is untouched. The playbook adds structure
around your development process — it doesn't replace your work.

Remember: AI is AI, and AI hallucinates.
Always verify generated code and outputs before shipping.
```

---

## Troubleshooting & Edge Cases

### Dirty Git Working Tree

Detected at Step 1. Upgrade aborts with:
> "Your working tree has uncommitted changes. Please commit or stash first."

### Existing `.claude/skills/`

Detected at Step 1. Warns user that skills will be overwritten. User must confirm.

### No Git Repo

Detected at Step 1. Warns but doesn't block. Skills use `git diff` but project can `git init` later.

### Unsupported Project Type

**HARD STOP** at Step 1. Refuses to upgrade. Explains playbook only supports `python-cli-with-webui`, `nextjs-with-cli`, `react-with-cli`.

### Project README.md Conflict

Handled at Step 7.1. If project has its own README.md at root, it stays untouched. The playbook's README.md is moved to `playbook/PLAYBOOK_README.md`.

### Partial Upgrade (Failed Midway)

No special handling needed. User restores from `.backup/` (backup is created before any modifications):

```bash
# Restore from backup
rm -rf .claude/skills playbook techs
cp -r .backup/* .
```

### Port Detection Failure

If server port can't be detected from existing config, falls back to type defaults:
- `python-cli-with-webui`: `8080`
- `nextjs-with-cli`: `3000`
- `react-with-cli`: `5173`
