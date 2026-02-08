# Cybervaldez Playbook - Upgrade Guide

> For existing projects that want structured AI-assisted development without starting over.
>
> **What you'll get:** 14 skills layered on top of your existing codebase — planning, building, testing, and consultation. Your code stays untouched. The playbook adds structure around your development process.

---

## Setup

Copy your existing project into the playbook root as a subfolder:

```bash
# From the playbook root
cp -r /path/to/my-app .
```

The upgrade process auto-detects your project folder — no special naming required.

---

## CRITICAL CONSTRAINTS

- Do NOT modify existing project files (code, configs, tests)
- Do NOT restructure the project's directory layout
- Do NOT replace existing tooling (test frameworks, CI/CD, linters)
- The playbook layers ON TOP of existing work — it does not replace it
- The playbook is a **factory** — the playbook root stays untouched, projects live as subfolders
- Best suited for alpha/prototype-stage projects — be honest if the project is too mature

---

## Pre-Flight: Auto-Detect Project Folder

1. **Scan ALL subdirectories** in the playbook root — do not stop at the first match.
2. **Skip known playbook dirs:** `skills/`, `.git/`, `.claude/`, `node_modules/`, `.github/`, `__pycache__/`, `test-kickstart/`
3. **Find dirs** that contain project files (`package.json`, `app.py`, `requirements.txt`, `manage.py`, `Cargo.toml`, `go.mod`, `tsconfig.json`, `vite.config.*`, `next.config.*`) but do **NOT** have `.claude/skills/research/SKILL.md`
4. **If one match** → that's the project. Use `{project-name}` as the detected folder name in all paths below. All steps run from the playbook root.
5. **If multiple matches** → ask the user which one to upgrade.
6. **If no matches** → **STOP:**
   > "No un-upgraded project found. Copy your project into this directory first — see the Setup section above."
7. **If matched folder already has `.claude/skills/research/SKILL.md`** → **STOP:**
   > "This project already has skills installed (`.claude/skills/research/SKILL.md` found). UPGRADE is for projects that haven't adopted the playbook yet."

---

## Step 1: Project Scan

**Automatic detection — no user input needed yet.** Uses detected `{project-name}/` as prefix. Runs from playbook root.

### 1.1: Detect Project Type

```bash
# Detect project type from files
ls {project-name}/package.json {project-name}/next.config.* {project-name}/vite.config.* {project-name}/app.py {project-name}/requirements.txt {project-name}/manage.py {project-name}/Cargo.toml {project-name}/go.mod 2>/dev/null

# Check for webui directory (required for python-cli-with-webui classification)
ls -d {project-name}/webui/ 2>/dev/null

# Scan structure
find {project-name}/ -maxdepth 3 -type f -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/venv/*' -not -path '*/__pycache__/*' | head -100
ls -la {project-name}/

# Count complexity signals
find {project-name}/ -type f -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/venv/*' -not -path '*/__pycache__/*' | wc -l
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
| Existing `.claude/` folder | `ls -la {project-name}/.claude/ 2>/dev/null` |
| CI/CD pipelines | `ls {project-name}/.github/workflows/ {project-name}/.gitlab-ci.yml {project-name}/Jenkinsfile 2>/dev/null` |
| Package dependencies | `jq '.dependencies + .devDependencies | length' {project-name}/package.json 2>/dev/null` or `wc -l < {project-name}/requirements.txt 2>/dev/null` (approximate — may include comments/blanks) |
| Git status | `git -C {project-name} status --porcelain` |
| Git history depth | `git -C {project-name} rev-list --count HEAD 2>/dev/null` |
| Total file count | Count from above |

### 1.3: Gate Checks

**Git check:** First check if the project has its own git repo: `[ -d {project-name}/.git ]`

- **No `.git` directory** → **WARN** but don't block (do NOT run `git status` — it would inherit the parent repo):
  > "No git repo found in `{project-name}/`. The upgrade will proceed, but some skills use `git diff` to track changes. You can run `git init` later."

- **Has `.git` directory** → Check for dirty tree: `git -C {project-name} status --porcelain`
  - If output → **ABORT:**
    > "Your working tree has uncommitted changes. Please commit or stash your changes before running the upgrade."
  - If clean → proceed

**Existing `.claude/skills/`:** If `{project-name}/.claude/skills/` exists but `{project-name}/.claude/skills/research/SKILL.md` does not (pre-flight passed), **note for Step 3 assessment:**

> "Existing `.claude/skills/` found — playbook skills will be added alongside existing files."

(No separate confirmation — this is factored into the Step 3 assessment.)

---

## Step 2: Tech Stack Research

### Where to find techs

Identify major technologies from these sources (in order):

1. **Direct dependencies:** `requirements.txt` entries / `package.json` `dependencies` (not `devDependencies`)
2. **README tech stack sections:** Any "Tech Stack" or "Built With" table in the project README
3. **Key imports:** Scan Python/JS source for third-party imports not covered by 1-2

**Skip:** Standard library modules (`os`, `json`, `http.server`, `path`), language runtimes (`Python`, `Node.js`), and shell tools (`Bash`, `curl`). These don't need tech research — they're assumed knowledge.

**Note:** This step reads the `/research` workflow from the **playbook root** (`skills/research/SKILL.md`), not from the project's `.claude/skills/` (which isn't installed until Step 4). Use it as a **reference for research structure** (web search, domain classification, README format) — but **skip the per-tech user confirmation** (research SKILL.md Step 4). During upgrade, tech findings are reviewed collectively in the Step 3 assessment instead.

### Verify and research

1. **Verify each tech exists in actual source code** — not just README or docs:
   - Python: `grep -r "import {tech}\|from {tech}" {project-name}/ --include="*.py"`
   - JS/TS: `grep -r "require.*{tech}\|import.*{tech}" {project-name}/ --include="*.js" --include="*.ts" --include="*.tsx"`
   - If a tech appears in README but not in code imports, **skip it** and log:
     > "Skipped {tech}: mentioned in README but not found in imports."
2. For each verified tech, follow the `/research` workflow (Steps 2-3 only: web search + domain classification). **Do not run research Step 4** (per-tech user confirmation) — tech findings are presented collectively in Step 3 assessment.
3. Save findings to `{project-name}/techs/{tech}/README.md`

**Implementation note:** Research each tech sequentially using a single agent rather than launching parallel agents per tech. This preserves full research depth while avoiding redundant context loading across agents (~60% token reduction). Each tech builds on the accumulated context from prior researches.

### Workflow per tech:

1. Web search for current best practices
2. Classify into domains per `skills/TECH_CONTEXT.md` Domain Classification Table
3. Save findings to `{project-name}/techs/{tech}/README.md` (reviewed as part of Step 3 assessment)

### Example flow:

```
Detected technologies: Next.js, React, TypeScript, Tailwind
Skipped Prisma: mentioned in README but not found in imports.

Researching: Next.js...
Researching: React...
Researching: TypeScript...
Researching: Tailwind...

--- Tech Research Summary ---

Next.js (framework): App Router, RSC, API routes...
React (ui-library): Component model, hooks, JSX...
TypeScript (language): Strict mode, type inference...
Tailwind (styling): Utility-first, JIT, responsive...

(Findings saved to techs/. Summary included in Step 3 assessment.)
```

**Phantom tech advisory:** If any techs were skipped (mentioned in docs but absent from imports), they'll appear in the Step 7 final report so you can update outdated references in your README.

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
| Existing `.claude/skills/` | Y/N | If yes = playbook skills added alongside existing files |
| Git history depth | N commits | >500 = mature project warning |

### Tech Stack Summary

Include the tech research summary from Step 2 in the assessment:

| Technology | Domain | Key Finding |
|------------|--------|-------------|
| {tech} | {domain} | {1-line summary} |

If any techs were auto-skipped: "{tech}: mentioned in README but not in imports"

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

Proceed with upgrade?
```

```
ASSESSMENT: PROCEED WITH CAUTION

{2-3 sentence summary}. Existing test suite and CI/CD mean
you'll have two parallel quality systems. The playbook's E2E
tests supplement but don't replace your existing tests.

Proceed with upgrade?
```

```
ASSESSMENT: NOT RECOMMENDED

{2-3 sentence summary}. This project has 500+ commits, 50+
dependencies, and mature CI/CD. The adaptation cost likely
exceeds the benefit at this stage.

If you still want to proceed, here's what would need attention:
- {specific concern 1}
- {specific concern 2}

Proceed anyway?
```

**User must explicitly confirm.** If "NOT RECOMMENDED", explain what would need to change and let user decide anyway.

---

## Step 4: Install Skills

Same skill installation as KICKSTART, copying from `skills/` in the playbook root, with existing-project awareness.

### 4.1: Handle Existing `.claude/` Folder

```bash
# Preserve existing .claude/ contents — only add skills/ subdirectory
mkdir -p {project-name}/.claude/skills
```

If `{project-name}/.claude/settings.json` or `{project-name}/.claude/settings.local.json` exist, leave them untouched.

### 4.2: Copy Core Skills

```bash
cp -r skills/core/* {project-name}/.claude/skills/
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

### 4.3: Copy Shared References + Research Skill

```bash
cp skills/TECH_CONTEXT.md {project-name}/.claude/skills/
cp skills/SKILL_INDEX.md {project-name}/.claude/skills/
cp -r skills/research {project-name}/.claude/skills/
```

### 4.4: Create techs/ Directory

If not already created from Step 2:

```bash
mkdir -p {project-name}/techs
# Only create techs/README.md if it doesn't exist
[ -f {project-name}/techs/README.md ] || cat > {project-name}/techs/README.md << 'EOF'
# Technologies

Research artifacts for technologies used in this project.

Run `/research {tech}` to research a technology and create reference docs for skills.

**Note:** Research happens here in your project, not in the playbook source. This ensures research has real project context (file structure, conventions, existing techs) for deterministic, useful reference docs.

## Researched Technologies

| Technology | Status | Domain | Description |
|------------|--------|--------|-------------|
EOF
```

### 4.5: Copy Browser Skills

Only for supported project types (`python-cli-with-webui`, `nextjs-with-cli`, `react-with-cli`):

```bash
cp -r skills/browser/* {project-name}/.claude/skills/
```

This includes:
- ux-review (visual UX verification via screenshots)
- e2e-guard (auto-generate missing E2E tests)
- e2e (full test orchestration)
- e2e-investigate (root cause analysis for failures)
- agent-browser (browser automation utility)

### 4.6: Replace Placeholders

Replace these placeholders in all skill `.md` files. Values are **detected from the existing project**, not defaults:

| Placeholder | Detection Logic |
|-------------|----------------|
| `{{PROJECT_NAME}}` | `{project-name}` (already known from pre-flight detection) |
| `{{SERVER_PORT}}` | Detect from existing config: `package.json` scripts (look for `--port` flags), `app.py` (look for `PORT` env or hardcoded port), or fall back to type default (`8080` for python, `3000` for nextjs, `5173` for react) |
| `{{SERVER_START}}` | Detect from `package.json` scripts (`dev` or `start`) or existing `start.sh`. Fall back to type default (`./start.sh` for python, `npm run dev` for nextjs/react) |
| `{{VENV_PYTHON}}` | `./venv/bin/python` if `venv/` exists, else `python3` |
| `{{WEBUI_PATH}}` | First directory (checking `webui/`, `app/`, `src/`, `public/` in order) that contains a `js/` or `templates/` subdirectory. If the match is nested (e.g., `webui/prompty/`), use the nested path. |
| `{{TESTS_PATH}}` | First of `tests/`, `test/`, `__tests__/`, `spec/` that exists, or create `tests/` |
| `{{API_BASE}}` | Detect the common API route prefix from actual route definitions: `grep -rh "path.*'/api\|route.*'/api\|@app.*('/api" {project-name}/ --include="*.py" --include="*.js" --include="*.ts"` and extract the longest shared prefix. Default `/api` if no routes found. |
| `{{OUTPUTS_PATH}}` | `public/outputs` if `public/` exists, else `outputs` |
| `{{TEST_JOB}}` | `test-fixtures` |

**Run the replacement:**

```bash
find {project-name}/.claude/skills -type f \( -name '*.md' -o -name '*.sh' \) -exec sed -i \
  -e 's|{{PROJECT_NAME}}|{project-name}|g' \
  -e 's|{{SERVER_PORT}}|{SERVER_PORT}|g' \
  -e 's|{{SERVER_START}}|{SERVER_START}|g' \
  -e 's|{{VENV_PYTHON}}|{VENV_PYTHON}|g' \
  -e 's|{{WEBUI_PATH}}|{WEBUI_PATH}|g' \
  -e 's|{{TESTS_PATH}}|{TESTS_PATH}|g' \
  -e 's|{{API_BASE}}|{API_BASE}|g' \
  -e 's|{{OUTPUTS_PATH}}|{OUTPUTS_PATH}|g' \
  -e 's|{{TEST_JOB}}|{TEST_JOB}|g' \
  {} +
```

**Note:** `{{BASE_URL}}` and ui-planner design tokens (e.g. `{{AESTHETIC}}`, `{{LAYOUT}}`) are runtime templates — do NOT replace them.

### 4.7: Generate `.claude/PROJECT_CONFIG.md`

**Before generating, capture the playbook source URL:**

```bash
# Run from the playbook root
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

## Skills Installed

Core skills from cybervaldez-playbook are installed in `.claude/skills/`.

## Next Steps

Pick your starting point:

| If you want to... | Run |
|-------------------|-----|
| Plan a new feature | `/ux-planner "I want to add [feature]"` |
| Redesign current UI | `/ui-planner "review current design of PROJECT_NAME"` |
| Get expert feedback | `/team "review PROJECT_NAME and suggest improvements"` |
| Get diverse user perspectives | `/kaizen "evaluate PROJECT_NAME from a user perspective"` |
| Implement something specific | `/create-task` |
| Research a technology | `/research {tech}` |
```

---

## Step 5: README Consolidation

**Merge playbook development workflow into the project's existing README.**

1. **Read** `{project-name}/README.md` (if it exists)
2. **Identify** what's already documented (setup, architecture, conventions, etc.)
3. **Add missing sections** from the template below — do NOT duplicate what already exists
4. **Present** the merged README to the user for approval before saving
5. **Save** only after user confirms

### Sections to add (if missing):

```markdown
## Development Workflow

This project uses the [Cybervaldez Playbook](PLAYBOOK_SOURCE) for structured AI-assisted development.

### Available Skills

| Skill | Purpose |
|-------|---------|
| `/ux-planner` | Plan features with UX tradeoffs |
| `/ui-planner` | Establish visual identity |
| `/create-task` | Build with tests baked in |
| `/coding-guard` | Audit for anti-patterns |
| `/e2e` | End-to-end test verification |
| `/research` | Research new technologies |

See `.claude/skills/SKILL_INDEX.md` for full details.

### Workflow

1. `/ux-planner` → plan the feature
2. `/ui-planner` → design the visuals
3. `/create-task` → implement with tests
4. `/coding-guard` + `/e2e` → verify quality
```

**Guidelines for merging:**
- Preserve the project's existing voice and content
- Add the Development Workflow section in a natural location (typically near the end, before Contributing/License)
- If the project already has a "Development" or "Contributing" section, integrate the skills table into it rather than creating a duplicate
- Replace `PLAYBOOK_SOURCE` with the detected playbook URL

---

## Step 6: Ensure Supporting Structure

```bash
# Use the TESTS_PATH value resolved in Step 4.6 (e.g., tests/)
mkdir -p {project-name}/{TESTS_PATH}

# If no .gitignore exists, seed one with project-type defaults
if [ ! -f {project-name}/.gitignore ]; then
  case {PROJECT_TYPE} in
    python-cli-with-webui)
      cat > {project-name}/.gitignore << 'EOF'
venv/
__pycache__/
*.pyc
.env
EOF
      ;;
    nextjs-with-cli)
      cat > {project-name}/.gitignore << 'EOF'
node_modules/
.next/
.env.local
EOF
      ;;
    react-with-cli)
      cat > {project-name}/.gitignore << 'EOF'
node_modules/
dist/
.env.local
EOF
      ;;
  esac
fi

# Ensure e2e-runs is ignored (append, don't replace)
grep -q '^tests/e2e-runs/' {project-name}/.gitignore 2>/dev/null || echo 'tests/e2e-runs/' >> {project-name}/.gitignore
```

---

## Step 7: Report & Next Steps

**Final step:** Add `{project-name}/` to the **playbook root** `.gitignore` (only after full success):

```bash
echo '{project-name}/' >> .gitignore
```

**Tell the user:**

```
UPGRADE COMPLETE: {PROJECT_NAME}

Skills installed: {project-name}/.claude/skills/ (14 skills)
Tech research: {project-name}/techs/ ({N} technologies documented)
Skipped (mentioned in docs but not in code): {comma-separated list, or "none"}
  → The upgrade does not modify existing project files.
    Update these references manually after the upgrade if they're outdated.

IMPORTANT: Open a new Claude Code session from inside the project folder.

What do you want to do first?

  /ux-planner "I want to add [feature]"           — plan a new feature
  /ui-planner "review current design of {PROJECT_NAME}"  — redesign or establish visual identity
  /team "review {PROJECT_NAME} and suggest improvements"  — get expert feedback
  /kaizen "evaluate {PROJECT_NAME} from a user perspective"  — diverse persona feedback
  /create-task                                     — implement something specific
  /research {tech}                                 — research a technology

Your existing code is untouched. The playbook adds structure
around your development process — it doesn't replace your work.

The playbook is reusable — you can upgrade another project
any time from the playbook root.

Remember: AI is AI, and AI hallucinates.
Always verify generated code and outputs before shipping.
```

---

## Troubleshooting & Edge Cases

### Dirty Git Working Tree

Detected at Step 1. Only checked if `{project-name}/.git` exists (prevents false positives from parent repos). Upgrade aborts with:
> "Your working tree has uncommitted changes. Please commit or stash first."

### Existing `.claude/skills/`

Detected at Step 1. Logged as a note and factored into the Step 3 compatibility assessment. No separate confirmation — playbook skills are added alongside existing files.

### No Git Repo

Detected at Step 1 by checking for `{project-name}/.git`. Warns but doesn't block — `git status` is NOT run (it would inherit the parent repo's state). Skills use `git diff` but project can `git init` later.

### Unsupported Project Type

**HARD STOP** at Step 1. Refuses to upgrade. Explains playbook only supports `python-cli-with-webui`, `nextjs-with-cli`, `react-with-cli`.

### Project README.md Conflict

Handled at Step 5 (README Consolidation). The AI merges playbook content into the existing README, presenting the result for user approval before saving. The project's original content is always preserved.

### Partial Upgrade (Failed Midway)

1. **Check what was installed:**
   ```bash
   ls {project-name}/.claude/skills/ 2>/dev/null  # Skills copied?
   ls {project-name}/techs/ 2>/dev/null             # Tech research done?
   cat {project-name}/.claude/PROJECT_CONFIG.md 2>/dev/null  # Config generated?
   ```

2. **Resume from failure point:**
   - If skills missing → Re-run from Step 4
   - If placeholders unreplaced → Re-run the `sed` command from Step 4.6
   - If README not merged → Re-run Step 5
   - If PROJECT_CONFIG missing → Re-run Step 4.7

3. **Start fresh** (only if project source exists elsewhere):
   ```bash
   rm -rf {project-name}/.claude/skills {project-name}/techs
   # Re-run upgrade from Step 4
   ```

### Port Detection Failure

If server port can't be detected from existing config, falls back to type defaults:
- `python-cli-with-webui`: `8080`
- `nextjs-with-cli`: `3000`
- `react-with-cli`: `5173`
