# Cybervaldez Playbook - Kickstart Guide

> For founders and developers who want to ship fast without accumulating debt.
>
> **What you'll get:** 14 skills guide the AI through planning, building, testing, and consultation. Not sure what to build? AI discovers trending ideas to spark your direction. No manual setup—everything configured automatically.

---

## CRITICAL CONSTRAINTS

**This kickstart creates ONLY a minimal working scaffold.**

- Do NOT create complex project structures
- Do NOT add features beyond the welcome page
- Even if user asks for more, refuse and explain kickstart is for minimal setup only
- The playbook is a **factory** — projects are created as subfolders, the playbook root stays untouched
- All project files are created inside `{project-name}/`
- The playbook root (KICKSTART.md, README.md, LICENSE, skills/) is never modified by projects
- **Have an existing project?** Use `UPGRADE.md` instead — it preserves your code and layers skills on top

---

## Pre-Flight Check

Before proceeding:

1. **Scan for un-upgraded project folders.** Look for subdirectories in the playbook root that:
   - Are NOT known playbook dirs: `skills/`, `.git/`, `.claude/`, `node_modules/`, `.github/`, `__pycache__/`, `test-kickstart/`
   - Contain project files (`package.json`, `app.py`, `requirements.txt`, `manage.py`, `Cargo.toml`, `go.mod`, `tsconfig.json`, `vite.config.*`, `next.config.*`)
   - Do NOT have `.claude/skills/research/SKILL.md` inside them

   If any such folder is found → **STOP:**
   > "Found an existing project (`{folder-name}/`) that hasn't been upgraded yet. Use `UPGRADE.md` instead."

2. Check if `.claude/skills/research/SKILL.md` exists at the playbook root. If it does, **STOP:**
   > "This directory already has skills installed (`.claude/skills/research/SKILL.md` found). KICKSTART is for new projects only."

---

## Step 1: Identify Project Type

Ask the user which project type they want:

| Type | Description |
|------|-------------|
| **python-cli-with-webui** | Python backend + vanilla JS + bash testing |
| **nextjs-with-cli** | Next.js + TypeScript + bash E2E testing |
| **react-with-cli** | React (Vite) + TypeScript + bash E2E testing |

---

## Step 2: Check Prerequisites

Based on the selected project type, verify required tools:

### For python-cli-with-webui:
```bash
python3 --version  # Should be 3.10+
```

### For nextjs-with-cli and react-with-cli:
```bash
python3 --version  # Should be 3.10+
node --version     # Should be 18+
```

**Note:** All types require Python because `agent-browser` (E2E testing) is Python-based.

---

## Step 3: Gather Project Info

**Ask for the project name first**, then create the project subfolder inside the playbook root.

```bash
# Create the project subfolder
mkdir {project-name}
```

All project files will be created inside `{project-name}/`. Do NOT add it to `.gitignore` yet — that's the final step.

Auto-detect or use defaults for:

| Placeholder | Detection Logic | Default (python) | Default (nextjs) | Default (react) |
|-------------|-----------------|------------------|-----------------|-----------------|
| `VENV_PYTHON` | `./venv/bin/python` if exists | `python3` | `python3` | `python3` |
| `WEBUI_PATH` | First of `webui/`, `src/`, `app/` that exists | `webui` | `src` | `src` |
| `TESTS_PATH` | `tests/` | `tests/` | `tests/` | `tests/` |
| `SERVER_START` | Based on project type | `./start.sh` | `npm run dev` | `npm run dev` |
| `SERVER_PORT` | Based on project type | `8080` | `3000` | `5173` |
| `API_BASE` | `/api` | `/api` | `/api` | `/api` |
| `OUTPUTS_PATH` | Based on project type | `outputs` | `public/outputs` | `public/outputs` |
| `TEST_JOB` | `test-fixtures` | `test-fixtures` | `test-fixtures` | `test-fixtures` |

---

## Step 4: Create Project Files

**All files are created inside `{project-name}/`.** This step creates the project structure, discovers trending ideas, and generates the welcome page content.

### 4.1: Project Structure

#### For python-cli-with-webui:

```
{project-name}/
├── .gitignore          # Git ignore rules (see template below)
├── app.py              # Flask app serving static files
├── start.sh            # Server start script
├── webui/
│   └── index.html      # Welcome page
└── tests/
    └── test_welcome.sh # Bash E2E test with agent-browser
```

**`.gitignore` template (used by all project types):**
```
# Environment & secrets
.env
.env.*

# Claude Code local config
.claude/settings.local.json

# OS files
.DS_Store
Thumbs.db

# Python
__pycache__/
*.py[cod]
venv/

# Node
node_modules/

# Build outputs
dist/
build/

# Test artifacts
tests/e2e-runs/
```

**app.py template:**
```python
from flask import Flask, send_from_directory
import os

app = Flask(__name__, static_folder='webui', static_url_path='')

@app.route('/')
def index():
    return send_from_directory('webui', 'index.html')

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 8080))
    app.run(host='0.0.0.0', port=port, debug=True)
```

**start.sh template:**
```bash
#!/bin/bash
cd "$(dirname "$0")"
source venv/bin/activate 2>/dev/null || true
python app.py
```

**Note:** Replace `PROJECT_NAME` in these templates with the actual project name when creating the files. These are NOT `{{}}` skill placeholders — they are resolved at file creation time, not by the Step 5 `sed` sweep.

**tests/test_welcome.sh template:**
```bash
#!/bin/bash
# tests/test_welcome.sh
set -euo pipefail

BASE_URL="${BASE_URL:-http://localhost:8080}"

cleanup() {
    agent-browser close 2>/dev/null || true
}
trap cleanup EXIT

agent-browser open "$BASE_URL"
sleep 1

# Check welcome message
agent-browser snapshot -c | grep -q "Welcome to PROJECT_NAME"

# Verify window state
STATE=$(agent-browser eval "window.appState?.initialized" 2>/dev/null)
[ "$STATE" = "true" ] && echo "  [PASS] App state initialized"

echo "PASS: Welcome page loads successfully"
```

#### For nextjs-with-cli:

```
{project-name}/
├── .gitignore          # Git ignore rules (see template below)
├── package.json
├── tsconfig.json
├── next.config.js
├── pages/
│   └── index.tsx       # Entry point
├── src/
│   └── App.tsx         # Welcome page component
└── tests/
    └── test_welcome.sh # Bash E2E test with agent-browser
```

**package.json template:**
```json
{
  "name": "PROJECT_NAME",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "test:e2e": "bash tests/test_welcome.sh"
  },
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}
```

**tests/test_welcome.sh template:**
```bash
#!/bin/bash
# tests/test_welcome.sh
set -euo pipefail

BASE_URL="${BASE_URL:-http://localhost:3000}"

cleanup() {
    agent-browser close 2>/dev/null || true
}
trap cleanup EXIT

agent-browser open "$BASE_URL"
sleep 1

# Check welcome message
agent-browser snapshot -c | grep -q "Welcome to PROJECT_NAME"

# Verify window state
STATE=$(agent-browser eval "window.appState?.initialized" 2>/dev/null)
[ "$STATE" = "true" ] && echo "  [PASS] App state initialized"

echo "PASS: Welcome page loads successfully"
```

**pages/index.tsx template:**
```tsx
import App from '../src/App';

export default function Home() {
  return <App />;
}
```

**next.config.js template:**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
};

module.exports = nextConfig;
```

**tsconfig.json template:**
```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

#### For react-with-cli:

```
{project-name}/
├── .gitignore          # Git ignore rules (see template above)
├── package.json
├── tsconfig.json
├── vite.config.ts
├── index.html            # Vite entry HTML
├── src/
│   ├── App.tsx           # Welcome page component
│   └── main.tsx          # Entry point
└── tests/
    └── test_welcome.sh
```

**package.json template:**
```json
{
  "name": "PROJECT_NAME",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test:e2e": "bash tests/test_welcome.sh"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0"
  }
}
```

**tests/test_welcome.sh template:**
```bash
#!/bin/bash
# tests/test_welcome.sh
set -euo pipefail

BASE_URL="${BASE_URL:-http://localhost:5173}"

cleanup() {
    agent-browser close 2>/dev/null || true
}
trap cleanup EXIT

agent-browser open "$BASE_URL"
sleep 1

# Check welcome message
agent-browser snapshot -c | grep -q "Welcome to PROJECT_NAME"

# Verify window state
STATE=$(agent-browser eval "window.appState?.initialized" 2>/dev/null)
[ "$STATE" = "true" ] && echo "  [PASS] App state initialized"

echo "PASS: Welcome page loads successfully"
```

**src/main.tsx template:**
```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

**index.html template (Vite root):**
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>PROJECT_NAME</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

**vite.config.ts template:**
```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
```

**tsconfig.json template:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

---

### 4.2: Discover Trending Ideas

**The AI discovers what's trending to spark your direction.**

The welcome page features **3 cyclable themes**, each with CONSTELLATIONS (trending) and UNCHARTED (blue ocean) ideas. Users click the footer to cycle through themes.

Before generating the welcome page, search the web for **3 distinct theme categories**:

#### For python-cli-with-webui, nextjs-with-cli, and react-with-cli:

**Theme 1: Rising Stars** — Hot trending ideas
- Example queries: "trending micro-SaaS ideas for solo founders 2026", "most profitable indie hacker tools 2026"
- nextHint: "drift toward hidden nebulae"

**Theme 2: Hidden Nebulae** — Obscure but profitable niches
- Example queries: "boring business software ideas 2026", "unsexy SaaS profitable niches"
- nextHint: "chart distant galaxies"

**Theme 3: Distant Galaxies** — Blue ocean opportunities
- Example queries: "underserved micro-SaaS niches 2026", "blue ocean SaaS no competition 2026"
- nextHint: "return to rising stars"

**For each theme, capture 3 CONSTELLATIONS and 3 UNCHARTED ideas:**
- **Title**: Short name (3-5 words)
- **Pitch**: One-line elevator pitch explaining the value (no numbers/percentages)

**All ideas MUST be playbook-compatible:**
- Has UI components (testable via agent-browser)
- Has API endpoints (testable via curl)
- Has observable state (window.appState or DOM)
- Behavior is deterministic (same input → same output)

**These AI-discovered ideas populate the cyclable themes in the welcome page.**

---

### 4.3: Welcome Page Content

Generate the welcome page with dynamic trending content from 4.2.

#### For python-cli-with-webui (webui/index.html):

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PROJECT_NAME</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        @keyframes warm-pulse {
            0%, 100% { color: #ffd700; }
            25% { color: #f0883e; }
            50% { color: #ff6b6b; }
            75% { color: #ffd700; }
        }
        @keyframes cool-shift {
            0%, 100% { color: #58a6ff; }
            25% { color: #bc8cff; }
            50% { color: #79c0ff; }
            75% { color: #d2a8ff; }
        }
        body {
            font-family: 'SF Mono', 'Monaco', 'Inconsolata', monospace;
            background: #0d1117;
            color: #c9d1d9;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 2rem;
        }
        .container { width: 600px; max-width: 100%; padding: 2rem; }
        .constellation {
            color: #58a6ff;
            white-space: pre;
            font-size: 0.85rem;
            line-height: 1.4;
            text-align: center;
            margin-bottom: 1rem;
            background: linear-gradient(135deg, #58a6ff, #7ee787, #58a6ff);
            -webkit-background-clip: text;
            background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        .constellation .star { -webkit-text-fill-color: #f0883e; }
        .constellation .star-bright {
            -webkit-text-fill-color: initial;
            animation: warm-pulse 2s infinite;
        }
        .constellation .star-cool {
            -webkit-text-fill-color: initial;
            animation: cool-shift 1.5s infinite;
        }
        .title { text-align: center; margin-bottom: 0.25rem; }
        .title h1 { color: #c9d1d9; font-size: 1.1rem; font-weight: normal; }
        .section {
            border: 1px solid #30363d;
            padding: 1rem;
            margin-bottom: 1rem;
        }
        .section-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: #8b949e;
            font-size: 0.7rem;
            letter-spacing: 0.15em;
            margin-bottom: 0.75rem;
            border-bottom: 1px solid #30363d;
            padding-bottom: 0.5rem;
        }
        .header-art {
            font-size: 0.75rem;
            letter-spacing: 0.3em;
        }
        .header-art .star { color: #f0883e; }
        .header-art .wave { color: #58a6ff; }
        .status-line {
            text-align: center;
            font-size: 0.85rem;
            margin-bottom: 1.5rem;
            color: #7ee787;
        }
        .status-line .dim { color: #8b949e; }
        .explore-list {
            list-style: none;
            padding: 0;
            min-height: 5.5rem;
        }
        .explore-list li {
            color: #c9d1d9;
            margin: 0.3rem 0;
            font-size: 0.85rem;
            line-height: 1.4;
        }
        .explore-list .star { color: #f0883e; }
        .explore-list .pitch {
            color: #8b949e;
            font-size: 0.75rem;
            margin-left: 1rem;
        }
        .nav-list {
            list-style: none;
            padding: 0;
        }
        .nav-list li {
            margin: 0.3rem 0;
            font-size: 0.85rem;
        }
        .nav-list .cmd { color: #7ee787; }
        .nav-list .desc { color: #8b949e; }
        .next-waypoint {
            border: 1px solid #f0883e;
            padding: 1rem;
            margin-bottom: 1rem;
            text-align: center;
        }
        .next-waypoint .header {
            color: #f0883e;
            font-size: 0.7rem;
            letter-spacing: 0.15em;
            margin-bottom: 0.5rem;
        }
        .next-waypoint code {
            color: #7ee787;
            font-size: 0.85rem;
        }
        .next-waypoint .hint {
            color: #8b949e;
            font-size: 0.75rem;
            margin-top: 0.5rem;
        }
        .theme-cycle {
            text-align: center;
            margin-top: 0.75rem;
            font-size: 0.7rem;
            color: #6e7681;
            cursor: pointer;
            transition: color 0.2s;
        }
        .theme-cycle:hover { color: #8b949e; }
        .theme-cycle .arrow { color: #58a6ff; }
    </style>
</head>
<body>
    <div class="container">
        <pre class="constellation">            <span class="star-bright">★</span>
       TRUE NORTH
            │
    ┌───────┼───────┐
    <span class="star-cool">☆</span>       <span class="star-cool">☆</span>       <span class="star-cool">☆</span>
guardrails cli-first e2e-truth</pre>

        <div class="title">
            <h1>Welcome to PROJECT_NAME</h1>
        </div>
        <p class="status-line">☆ Kickstarted <span class="dim">— scaffold complete, ready to navigate</span></p>

        <div class="section">
            <div class="section-header">
                <span>── CONSTELLATIONS ──</span>
                <span class="header-art"><span class="star">✦</span> · <span class="star">☆</span> · <span class="star">✦</span></span>
            </div>
            <ul class="explore-list" id="constellations"></ul>
        </div>

        <div class="section">
            <div class="section-header">
                <span>── UNCHARTED ──</span>
                <span class="header-art"><span class="wave">~</span> <span class="wave">≈</span> <span class="wave">~</span> <span class="wave">≈</span></span>
            </div>
            <ul class="explore-list" id="uncharted"></ul>
        </div>

        <div class="section">
            <div class="section-header">
                <span>── NAVIGATION ──</span>
                <span class="header-art"><span class="star">☆</span></span>
            </div>
            <ul class="nav-list">
                <li><span class="cmd">/ux-planner</span>   <span class="desc">— chart your first feature</span></li>
                <li><span class="cmd">/create-task</span>  <span class="desc">— build with tests baked in</span></li>
                <li><span class="cmd">/research</span>     <span class="desc">— evaluate tech for your stack</span></li>
                <li><span class="cmd">/coding-guard</span> <span class="desc">— check for anti-patterns</span></li>
                <li><span class="cmd">/e2e</span>          <span class="desc">— prove it works end-to-end</span></li>
            </ul>
        </div>

        <div class="next-waypoint">
            <div class="header">★ NEXT WAYPOINT</div>
            <code>/ux-planner "I want to build [your idea]"</code>
            <p class="hint">chart your course before you build</p>
        </div>
        <p class="theme-cycle" id="theme-toggle">
            <span id="theme-hint">THEME_1_NEXT_HINT</span> <span class="arrow">→</span>
        </p>

        <!-- Debug container for tests -->
        <div id="app-debug" style="display: none;">
            <pre id="debug-state"></pre>
            <div id="debug-log"></div>
        </div>
    </div>

    <script>
        const themes = [
            {
                name: 'THEME_1_NAME',
                nextHint: 'THEME_1_NEXT_HINT',
                constellations: [
                    { title: 'THEME_1_CONSTELLATION_1_TITLE', pitch: 'THEME_1_CONSTELLATION_1_PITCH' },
                    { title: 'THEME_1_CONSTELLATION_2_TITLE', pitch: 'THEME_1_CONSTELLATION_2_PITCH' },
                    { title: 'THEME_1_CONSTELLATION_3_TITLE', pitch: 'THEME_1_CONSTELLATION_3_PITCH' },
                ],
                uncharted: [
                    { title: 'THEME_1_UNCHARTED_1_TITLE', pitch: 'THEME_1_UNCHARTED_1_PITCH' },
                    { title: 'THEME_1_UNCHARTED_2_TITLE', pitch: 'THEME_1_UNCHARTED_2_PITCH' },
                    { title: 'THEME_1_UNCHARTED_3_TITLE', pitch: 'THEME_1_UNCHARTED_3_PITCH' },
                ]
            },
            {
                name: 'THEME_2_NAME',
                nextHint: 'THEME_2_NEXT_HINT',
                constellations: [
                    { title: 'THEME_2_CONSTELLATION_1_TITLE', pitch: 'THEME_2_CONSTELLATION_1_PITCH' },
                    { title: 'THEME_2_CONSTELLATION_2_TITLE', pitch: 'THEME_2_CONSTELLATION_2_PITCH' },
                    { title: 'THEME_2_CONSTELLATION_3_TITLE', pitch: 'THEME_2_CONSTELLATION_3_PITCH' },
                ],
                uncharted: [
                    { title: 'THEME_2_UNCHARTED_1_TITLE', pitch: 'THEME_2_UNCHARTED_1_PITCH' },
                    { title: 'THEME_2_UNCHARTED_2_TITLE', pitch: 'THEME_2_UNCHARTED_2_PITCH' },
                    { title: 'THEME_2_UNCHARTED_3_TITLE', pitch: 'THEME_2_UNCHARTED_3_PITCH' },
                ]
            },
            {
                name: 'THEME_3_NAME',
                nextHint: 'THEME_3_NEXT_HINT',
                constellations: [
                    { title: 'THEME_3_CONSTELLATION_1_TITLE', pitch: 'THEME_3_CONSTELLATION_1_PITCH' },
                    { title: 'THEME_3_CONSTELLATION_2_TITLE', pitch: 'THEME_3_CONSTELLATION_2_PITCH' },
                    { title: 'THEME_3_CONSTELLATION_3_TITLE', pitch: 'THEME_3_CONSTELLATION_3_PITCH' },
                ],
                uncharted: [
                    { title: 'THEME_3_UNCHARTED_1_TITLE', pitch: 'THEME_3_UNCHARTED_1_PITCH' },
                    { title: 'THEME_3_UNCHARTED_2_TITLE', pitch: 'THEME_3_UNCHARTED_2_PITCH' },
                    { title: 'THEME_3_UNCHARTED_3_TITLE', pitch: 'THEME_3_UNCHARTED_3_PITCH' },
                ]
            }
        ];

        let currentTheme = 0;

        function renderTheme() {
            const theme = themes[currentTheme];
            document.getElementById('constellations').innerHTML = theme.constellations.map(i =>
                `<li><span class="star">✦</span> ${i.title}<br><span class="pitch">↳ ${i.pitch}</span></li>`
            ).join('');
            document.getElementById('uncharted').innerHTML = theme.uncharted.map(i =>
                `<li><span class="star">✦</span> ${i.title}<br><span class="pitch">↳ ${i.pitch}</span></li>`
            ).join('');
            document.getElementById('theme-hint').textContent = theme.nextHint;
        }

        document.getElementById('theme-toggle').addEventListener('click', () => {
            currentTheme = (currentTheme + 1) % themes.length;
            renderTheme();
        });

        renderTheme();

        window.appState = {
            view: 'welcome',
            initialized: true
        };
    </script>
</body>
</html>
```

#### For nextjs-with-cli and react-with-cli (src/App.tsx):

**Note:** The `react-with-cli` and `nextjs-with-cli` project types share the same App.tsx welcome page component and HTML template below.

```tsx
import React, { useEffect, useState } from 'react';

const s = {
  body: {
    fontFamily: "'SF Mono', 'Monaco', 'Inconsolata', monospace",
    background: '#0d1117',
    color: '#c9d1d9',
    minHeight: '100vh',
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    padding: '2rem',
  },
  container: { width: '600px', maxWidth: '100%', padding: '2rem' },
  constellation: {
    color: '#58a6ff',
    whiteSpace: 'pre' as const,
    fontSize: '0.85rem',
    lineHeight: 1.4,
    textAlign: 'center' as const,
    marginBottom: '1rem',
  },
  starBright: { color: '#ffd700' },
  starCool: { color: '#58a6ff' },
  star: { color: '#f0883e' },
  title: { textAlign: 'center' as const, marginBottom: '0.25rem' },
  h1: { color: '#c9d1d9', fontSize: '1.1rem', fontWeight: 'normal' as const },
  statusLine: {
    textAlign: 'center' as const,
    fontSize: '0.85rem',
    marginBottom: '1.5rem',
    color: '#7ee787',
  },
  section: {
    border: '1px solid #30363d',
    padding: '1rem',
    marginBottom: '1rem',
  },
  sectionHeader: {
    display: 'flex',
    justifyContent: 'space-between',
    alignItems: 'center',
    color: '#8b949e',
    fontSize: '0.7rem',
    letterSpacing: '0.15em',
    marginBottom: '0.75rem',
    borderBottom: '1px solid #30363d',
    paddingBottom: '0.5rem',
  },
  headerArt: {
    fontSize: '0.75rem',
    letterSpacing: '0.3em',
  },
  dim: { color: '#8b949e' },
  list: { listStyle: 'none' as const, padding: 0, margin: 0, minHeight: '5.5rem' },
  listItem: { color: '#c9d1d9', margin: '0.3rem 0', fontSize: '0.85rem', lineHeight: 1.4 },
  navItem: { margin: '0.3rem 0', fontSize: '0.85rem' },
  cmd: { color: '#7ee787' },
  desc: { color: '#8b949e' },
  nextWaypoint: {
    border: '1px solid #f0883e',
    padding: '1rem',
    marginBottom: '1rem',
    textAlign: 'center' as const,
  },
  nextHeader: {
    color: '#f0883e',
    fontSize: '0.7rem',
    letterSpacing: '0.15em',
    marginBottom: '0.5rem',
  },
  nextCode: { color: '#7ee787', fontSize: '0.85rem' },
  nextHint: { color: '#8b949e', fontSize: '0.75rem', marginTop: '0.5rem' },
  wave: { color: '#58a6ff' },
  pitch: { color: '#8b949e', fontSize: '0.75rem', marginLeft: '1rem' },
  themeCycle: {
    textAlign: 'center' as const,
    marginTop: '0.75rem',
    fontSize: '0.7rem',
    color: '#6e7681',
    cursor: 'pointer',
    transition: 'color 0.2s',
  },
  arrow: { color: '#58a6ff' },
};

const themes = [
  {
    name: 'THEME_1_NAME',
    nextHint: 'THEME_1_NEXT_HINT',
    constellations: [
      { title: 'THEME_1_CONSTELLATION_1_TITLE', pitch: 'THEME_1_CONSTELLATION_1_PITCH' },
      { title: 'THEME_1_CONSTELLATION_2_TITLE', pitch: 'THEME_1_CONSTELLATION_2_PITCH' },
      { title: 'THEME_1_CONSTELLATION_3_TITLE', pitch: 'THEME_1_CONSTELLATION_3_PITCH' },
    ],
    uncharted: [
      { title: 'THEME_1_UNCHARTED_1_TITLE', pitch: 'THEME_1_UNCHARTED_1_PITCH' },
      { title: 'THEME_1_UNCHARTED_2_TITLE', pitch: 'THEME_1_UNCHARTED_2_PITCH' },
      { title: 'THEME_1_UNCHARTED_3_TITLE', pitch: 'THEME_1_UNCHARTED_3_PITCH' },
    ]
  },
  {
    name: 'THEME_2_NAME',
    nextHint: 'THEME_2_NEXT_HINT',
    constellations: [
      { title: 'THEME_2_CONSTELLATION_1_TITLE', pitch: 'THEME_2_CONSTELLATION_1_PITCH' },
      { title: 'THEME_2_CONSTELLATION_2_TITLE', pitch: 'THEME_2_CONSTELLATION_2_PITCH' },
      { title: 'THEME_2_CONSTELLATION_3_TITLE', pitch: 'THEME_2_CONSTELLATION_3_PITCH' },
    ],
    uncharted: [
      { title: 'THEME_2_UNCHARTED_1_TITLE', pitch: 'THEME_2_UNCHARTED_1_PITCH' },
      { title: 'THEME_2_UNCHARTED_2_TITLE', pitch: 'THEME_2_UNCHARTED_2_PITCH' },
      { title: 'THEME_2_UNCHARTED_3_TITLE', pitch: 'THEME_2_UNCHARTED_3_PITCH' },
    ]
  },
  {
    name: 'THEME_3_NAME',
    nextHint: 'THEME_3_NEXT_HINT',
    constellations: [
      { title: 'THEME_3_CONSTELLATION_1_TITLE', pitch: 'THEME_3_CONSTELLATION_1_PITCH' },
      { title: 'THEME_3_CONSTELLATION_2_TITLE', pitch: 'THEME_3_CONSTELLATION_2_PITCH' },
      { title: 'THEME_3_CONSTELLATION_3_TITLE', pitch: 'THEME_3_CONSTELLATION_3_PITCH' },
    ],
    uncharted: [
      { title: 'THEME_3_UNCHARTED_1_TITLE', pitch: 'THEME_3_UNCHARTED_1_PITCH' },
      { title: 'THEME_3_UNCHARTED_2_TITLE', pitch: 'THEME_3_UNCHARTED_2_PITCH' },
      { title: 'THEME_3_UNCHARTED_3_TITLE', pitch: 'THEME_3_UNCHARTED_3_PITCH' },
    ]
  }
];

export default function App() {
  const [themeIndex, setThemeIndex] = useState(0);
  const theme = themes[themeIndex];

  useEffect(() => {
    (window as any).appState = {
      view: 'welcome',
      initialized: true
    };
  }, []);

  return (
    <div style={s.body}>
      <div style={s.container}>
        <style>{`
          html, body { margin: 0; padding: 0; background: #0d1117; }
          @keyframes warm-pulse {
            0%, 100% { color: #ffd700; }
            25% { color: #f0883e; }
            50% { color: #ff6b6b; }
            75% { color: #ffd700; }
          }
          @keyframes cool-shift {
            0%, 100% { color: #58a6ff; }
            25% { color: #bc8cff; }
            50% { color: #79c0ff; }
            75% { color: #d2a8ff; }
          }
          .constellation-art {
            background: linear-gradient(135deg, #58a6ff, #7ee787, #58a6ff);
            -webkit-background-clip: text;
            background-clip: text;
            -webkit-text-fill-color: transparent;
          }
          .constellation-art .star-icon {
            -webkit-text-fill-color: #f0883e;
          }
          .constellation-art .star-bright-icon {
            -webkit-text-fill-color: initial;
            animation: warm-pulse 2s infinite;
          }
          .constellation-art .star-cool-icon {
            -webkit-text-fill-color: initial;
            animation: cool-shift 1.5s infinite;
          }
        `}</style>
        <pre style={s.constellation} className="constellation-art">
{`            `}<span style={s.starBright} className="star-bright-icon">★</span>{`
       TRUE NORTH
            │
    ┌───────┼───────┐
    `}<span style={s.starCool} className="star-cool-icon">☆</span>{`       `}<span style={s.starCool} className="star-cool-icon">☆</span>{`       `}<span style={s.starCool} className="star-cool-icon">☆</span>{`
guardrails cli-first e2e-truth`}
        </pre>

        <div style={s.title}>
          <h1 style={s.h1}>Welcome to PROJECT_NAME</h1>
        </div>
        <p style={s.statusLine}>☆ Kickstarted <span style={s.dim}>— scaffold complete, ready to navigate</span></p>

        <div style={s.section}>
          <div style={s.sectionHeader}>
            <span>── CONSTELLATIONS ──</span>
            <span style={s.headerArt}><span style={s.star}>✦</span> · <span style={s.star}>☆</span> · <span style={s.star}>✦</span></span>
          </div>
          <ul style={s.list}>
            {theme.constellations.map((item, i) => (
              <li key={i} style={s.listItem}>
                <span style={s.star}>✦</span> {item.title}<br/>
                <span style={s.pitch}>↳ {item.pitch}</span>
              </li>
            ))}
          </ul>
        </div>

        <div style={s.section}>
          <div style={s.sectionHeader}>
            <span>── UNCHARTED ──</span>
            <span style={s.headerArt}><span style={s.wave}>~</span> <span style={s.wave}>≈</span> <span style={s.wave}>~</span> <span style={s.wave}>≈</span></span>
          </div>
          <ul style={s.list}>
            {theme.uncharted.map((item, i) => (
              <li key={i} style={s.listItem}>
                <span style={s.star}>✦</span> {item.title}<br/>
                <span style={s.pitch}>↳ {item.pitch}</span>
              </li>
            ))}
          </ul>
        </div>

        <div style={s.section}>
          <div style={s.sectionHeader}>
            <span>── NAVIGATION ──</span>
            <span style={s.headerArt}><span style={s.star}>☆</span></span>
          </div>
          <ul style={s.list}>
            <li style={s.navItem}><span style={s.cmd}>/ux-planner</span>   <span style={s.desc}>— chart your first feature</span></li>
            <li style={s.navItem}><span style={s.cmd}>/create-task</span>  <span style={s.desc}>— build with tests baked in</span></li>
            <li style={s.navItem}><span style={s.cmd}>/research</span>     <span style={s.desc}>— evaluate tech for your stack</span></li>
            <li style={s.navItem}><span style={s.cmd}>/coding-guard</span> <span style={s.desc}>— check for anti-patterns</span></li>
            <li style={s.navItem}><span style={s.cmd}>/e2e</span>          <span style={s.desc}>— prove it works end-to-end</span></li>
          </ul>
        </div>

        <div style={s.nextWaypoint}>
          <div style={s.nextHeader}>★ NEXT WAYPOINT</div>
          <code style={s.nextCode}>/ux-planner "I want to build [your idea]"</code>
          <p style={s.nextHint}>chart your course before you build</p>
        </div>
        <p style={s.themeCycle} onClick={() => setThemeIndex((themeIndex + 1) % themes.length)}>
          <span>{theme.nextHint}</span> <span style={s.arrow}>→</span>
        </p>

        {/* Debug container for tests */}
        <div id="app-debug" style={{ display: 'none' }}>
          <pre id="debug-state"></pre>
          <div id="debug-log"></div>
        </div>
      </div>
    </div>
  );
}
```

**Replace placeholders:**
- `PROJECT_NAME` - User's project name
- For each theme (1, 2, 3), replace from web search results (4.2):
  - `THEME_N_NAME` - Theme name (e.g., "rising-stars", "hidden-nebulae", "distant-galaxies")
  - `THEME_N_NEXT_HINT` - Hint for next theme (e.g., "drift toward hidden nebulae", "chart distant galaxies", "return to rising stars")
  - `THEME_N_CONSTELLATION_1/2/3_TITLE` - Trending idea titles for this theme
  - `THEME_N_CONSTELLATION_1/2/3_PITCH` - Elevator pitches for trending ideas
  - `THEME_N_UNCHARTED_1/2/3_TITLE` - Blue ocean idea titles for this theme
  - `THEME_N_UNCHARTED_1/2/3_PITCH` - Elevator pitches for blue ocean ideas

---

## Step 5: Install Skills

Copy skills from `skills/` (in the playbook root) into the project's `.claude/skills/`:

```bash
mkdir -p {project-name}/.claude/skills
```

### For ALL project types:

**Step 5.1: Copy core skills:**
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

**Step 5.2: Copy shared references and research skill:**
```bash
cp skills/TECH_CONTEXT.md {project-name}/.claude/skills/
cp skills/SKILL_INDEX.md {project-name}/.claude/skills/
cp -r skills/research {project-name}/.claude/skills/
```

This includes:
- TECH_CONTEXT.md (domain classification & skill concern matrix)
- SKILL_INDEX.md (skill discovery index — run `/help` or read this to discover all 14 skills)
- research (technology research skill)

**Step 5.3: Create techs/ directory for project-specific research:**
```bash
mkdir -p {project-name}/techs
cat > {project-name}/techs/README.md << 'EOF'
# Technologies

Research artifacts for technologies used in this project.

Run `/research {tech}` to research a technology and create reference docs for skills.

**Note:** Research happens here in your project, not in the playbook source. This ensures research has real project context (file structure, conventions, existing techs) for deterministic, useful reference docs.

## Researched Technologies

| Technology | Status | Domain | Description |
|------------|--------|--------|-------------|
EOF
```

### For python-cli-with-webui, nextjs-with-cli, and react-with-cli:

**Step 5.4: Copy browser skills:**
```bash
cp -r skills/browser/* {project-name}/.claude/skills/
```

This includes:
- ux-review (visual UX verification via screenshots)
- e2e-guard (auto-generate missing E2E tests)
- e2e (full test orchestration)
- e2e-investigate (root cause analysis for failures)
- agent-browser (browser automation utility)

**After copying, replace these placeholders in all skill .md files:**

| Placeholder | Replace With |
|-------------|--------------|
| `{{PROJECT_NAME}}` | User's project name |
| `{{TEST_JOB}}` | `test-fixtures` (or detected value) |
| `{{SERVER_PORT}}` | `3000` or `8080` (based on project type) |
| `{{SERVER_START}}` | `npm run dev` or `./start.sh` |
| `{{VENV_PYTHON}}` | `./venv/bin/python` or `python3` |
| `{{API_BASE}}` | `/api` |
| `{{WEBUI_PATH}}` | `src` or `webui` |
| `{{TESTS_PATH}}` | `tests/` |
| `{{OUTPUTS_PATH}}` | `public/outputs` or `outputs` |

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

**Verify no placeholders were missed** (should return nothing — only runtime templates remain):

```bash
grep -r '{{' {project-name}/.claude/skills/ --include='*.md' | grep -v '{{BASE_URL}}\|{{AESTHETIC}}\|{{LAYOUT}}'
```

If any lines appear, a placeholder was not replaced. Check the `sed` values and re-run.

**Note:** `{{BASE_URL}}` and ui-planner design tokens (e.g. `{{AESTHETIC}}`, `{{LAYOUT}}`) are runtime templates — do NOT replace them.

**Note:** Run `/research {tech}` in your project to add technology-specific reference docs to skills.

**Before generating PROJECT_CONFIG.md, capture the playbook source URL:**

```bash
# Run from the playbook root
git remote get-url origin 2>/dev/null
```

Strip any `.git` suffix from the result (e.g., `https://github.com/user/repo.git` → `https://github.com/user/repo`). If the URL starts with `git@` (SSH format), convert to HTTPS: `git@github.com:user/repo.git` → `https://github.com/user/repo`. If no remote exists, use an empty string. Use this as the `PLAYBOOK_SOURCE` value below.

**Generate `{project-name}/.claude/PROJECT_CONFIG.md`:**

```markdown
# Project Configuration

Generated by Cybervaldez Playbook Kickstart

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

## Skills Installed

Core skills from cybervaldez-playbook are installed in `.claude/skills/`.

## Next Steps

1. Run `/ux-planner` to plan your first feature
2. Run `/create-task` to implement with tests
3. Run `/e2e` to verify everything works
```

---

## Step 6: Generate README

Generate a `README.md` inside `{project-name}/` with project-specific context. **All commands run from the playbook root using `{project-name}/` prefixes.**

```markdown
# {PROJECT_NAME}

{One-line description — user provides or AI suggests based on trending ideas}

## Quick Start

\`\`\`bash
{SERVER_START}
# Open http://localhost:{SERVER_PORT}
\`\`\`

## Tech Stack

- {detected/chosen technologies}

## Development Workflow

This project uses the [Cybervaldez Playbook]({PLAYBOOK_SOURCE}) for structured AI-assisted development.

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

## Project Structure

\`\`\`
{generated from actual file tree}
\`\`\`

## Conventions

_To be established as the project grows. Run `/coding-guard` to audit patterns._
```

---

## Step 7: Initialize Git

**Initialize a fresh git repo inside the project subfolder.** The playbook's git history stays untouched. All commands run from the playbook root using `git -C`.

```bash
git -C {project-name} init
git -C {project-name} add .
git -C {project-name} commit -m "Initial kickstart: {project-name}"
```

This gives the project its own git history (including the README) in a single initial commit, independent of the playbook repo.

---

## Step 8: Launch & Verify

**All commands run from the playbook root.** Add `{project-name}/` to the playbook's `.gitignore`, install dependencies, start the server, and verify.

```bash
echo '{project-name}/' >> .gitignore
```

**For python-cli-with-webui:**
```bash
chmod +x {project-name}/start.sh
(cd {project-name} && ./start.sh) &
sleep 2
curl -sf http://localhost:8080 | head -20
```

**For nextjs-with-cli:**
```bash
npm install --prefix {project-name}
npm run dev --prefix {project-name} &
sleep 5
curl -sf http://localhost:3000 | head -20
```

**For react-with-cli:**
```bash
npm install --prefix {project-name}
npm run dev --prefix {project-name} &
sleep 5
curl -sf http://localhost:5173 | head -20
```

**Tell the user:**

```
★ {project-name} is kickstarted!

Your welcome page is running at http://localhost:{PORT}
Open it in your browser to see your constellation map.

⚠ IMPORTANT: Keep this server running, then restart Claude Code
from inside the project folder.

  cd {project-name}

Then restart Claude Code so it picks up the new skills
from .claude/skills/:

14 skills installed — run /help or see .claude/skills/SKILL_INDEX.md

Start here:
  /ux-planner   — chart your first feature
  /create-task  — build with tests baked in
  /e2e          — prove it works end-to-end

Explore more:
  /ui-planner   — establish visual identity
  /research     — evaluate tech for your stack
  /coding-guard — check for anti-patterns
  /kaizen       — get diverse persona feedback
  /team         — summon expert consultants

After restarting, chart your course:
  /ux-planner "I want to build [your idea]"

The playbook is reusable — you can kickstart another project
any time from the playbook root.

Run /research {tech} in your project to add technology-specific
reference docs.

Remember: AI is AI, and AI hallucinates.
Always verify generated code and outputs before shipping.
```

---

## Troubleshooting & Edge Cases

### Partial Kickstart (Failed Midway)

If kickstart fails partway through:

1. **Check what was created:**
   ```bash
   ls -la {project-name}/.claude/skills/ 2>/dev/null  # Skills installed?
   ls {project-name}/.git/ 2>/dev/null                 # Git initialized?
   ```

2. **Resume from failure point:**
   - If skills missing → Re-run Step 5 (copy skills)
   - If README missing → Re-run Step 6
   - If git not initialized → Re-run Step 7

3. **Start fresh (nuclear option):**
   ```bash
   rm -rf {project-name}
   # Re-run kickstart from step 1
   ```

### Port Conflicts

If default port is already in use:

```bash
# Check if port is in use
lsof -i :8080  # or :3000 for nextjs, :5173 for react

# Options:
# 1. Kill the process using the port
# 2. Change SERVER_PORT during kickstart
# 3. Edit .claude/PROJECT_CONFIG.md after kickstart
```

**Post-kickstart port change:**
```bash
# Update all skill files
sed -i 's/{{SERVER_PORT}}/8081/g' {project-name}/.claude/skills/**/SKILL.md
# Update PROJECT_CONFIG.md manually
```

### Missing Python venv

If `./venv/bin/python` doesn't exist:

```bash
# Create venv
python3 -m venv venv

# Or use system Python (update during kickstart)
# VENV_PYTHON will be set to "python3" instead
```

### Wrong Python Version

Requires Python 3.10+:
```bash
python3 --version  # Check version

# If too old:
# 1. Install Python 3.10+ via pyenv or system package manager
# 2. Update VENV_PYTHON to point to correct Python
```

### Node.js Not Installed (for nextjs/react)

```bash
node --version  # Should be 18+

# If not installed:
# 1. Install from nodejs.org
# 2. Or use nvm: nvm install 18 && nvm use 18
```

### Project Type Doesn't Match Options

If your project isn't `python-cli-with-webui`, `nextjs-with-cli`, or `react-with-cli`:

1. **Choose closest match** and customize after
2. **Or add custom type** to KICKSTART.md:
   - Add detection logic in Step 1
   - Add file templates in Step 4
   - Add placeholder defaults in Step 3

---

## Post-Kickstart: Optional Customizations

### Skills Customization Levels

**For python-cli-with-webui, nextjs-with-cli, and react-with-cli:**

| Skill | Level | Notes |
|-------|-------|-------|
| `create-task` | High | Project structure, conventions, test patterns |
| `e2e` | High | Server startup, test phases, cleanup |
| `e2e-guard` | Medium | File paths, test discovery |
| `coding-guard` | Medium | Anti-patterns specific to your stack |
| `ux-planner` | Low | Universal UX principles |
| `agent-browser` | Low | Just server startup command |

### Adding Project-Specific Anti-Patterns

Edit `.claude/skills/coding-guard/SKILL.md`:

```markdown
## Project-Specific Anti-Patterns

| Pattern | Why Bad |
|---------|---------|
| Direct DOM manipulation | Use framework refs |
| Any type | Use proper TypeScript types |
```

### Adding Project-Specific Test Patterns

Edit `.claude/PROJECT_CONVENTIONS.md`:

```markdown
## Project Test Patterns

- All API tests use `supertest`
- Database reset between tests
- Screenshots saved to `test-results/`
```
