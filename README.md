# Cybervaldez Playbook

**My approach to AI-assisted development - battle-tested patterns from real projects.**

## What This Repo Is For

This playbook serves two purposes:

### 1. For Evaluating My Work Approach
If you're a founder, tech lead, or recruiter considering working with me, this repo demonstrates:
- How I structure development workflows
- The quality guardrails I build into projects
- My philosophy on testing, observability, and maintainability
- Real patterns from production projects

### 2. For Founders & Developers Ready to Kickstart An Idea
- 14 skills guide the AI through planning, building, testing, and consultation*
- Not sure what to build yet? Optionally discover trending ideas to spark your direction
- No manual setup—everything configured automatically**

*Need help at any stage? I'm available for consulting and development work.
**All you need: an AI subscription (Claude/Gemini) and 2 minutes. Quick Start guide below.

## A Note on Philosophy

These patterns represent **my true north** - lessons learned from building projects with AI assistance. They're guidelines shaped by experience, not universal rules. Your mileage may vary, and that's okay. Take what works, adapt what doesn't.

---

## Quick Start

**Requirements:**
- Claude Code or compatible AI assistant (Claude, Gemini, etc.)

**Setup:**

1. Clone this repo — it's a **factory** that produces projects as subfolders:
```bash
git clone https://github.com/cybervaldez/cybervaldez-playbook.git
cd cybervaldez-playbook
```

2. **New project?** Tell your AI assistant:
```
Kickstart a new project using KICKSTART.md
```
The AI creates a project subfolder (e.g., `my-app/`) with scaffold, skills, and its own git history. The playbook root stays untouched and reusable.

3. **Existing project?** Copy it in and upgrade:
```bash
cp -r /path/to/my-app .
```
Then tell your AI assistant:
```
Upgrade my project using UPGRADE.md
```
The AI auto-detects the project folder, layers skills on top without modifying your code.

4. Start building (from inside the project folder):
```bash
cd my-app
# Restart Claude Code, then:
/ux-planner "I want to build [your idea]"
```

The playbook is reusable — kickstart or upgrade multiple projects from the same clone.

---

## Start Here: Recommended Reading Order

New to the playbook? Follow this path:

### 1. Understand the Philosophy (5 min)
- [The Origin Story](#the-origin-story) - Why these patterns exist
- [The Philosophy](#the-philosophy) - Forced pauses and quality gates
- [CLI-First = AI-First Debugging](#cli-first--ai-first-debugging) - Why text over screenshots

### 2. Learn the Core Skills (10 min)
- [Skills Reference](#skills-reference) - Overview of all 14 skills
- [The Execution Flow](#the-execution-flow) - Visual pipeline diagram
- [Skill Prerequisite Matrix](#skill-prerequisite-matrix) - What depends on what
- [SKILL_INDEX.md](skills/SKILL_INDEX.md) - Quick reference for finding skills

### 3. First Feature Workflow
```
/ux-planner "I want to build [your idea]"   # Plan the UX
    ↓
/ui-planner                                  # Design the visuals (optional)
    ↓
/create-task "Implement [feature]"           # Build it
    ↓
/coding-guard + /cli-first + /ux-review      # Quality gates (parallel)
    ↓
/e2e                                         # Verify it works
```

### 4. When Things Go Wrong
- [Error Recovery Guide](#error-recovery-guide) - What to do when skills fail
- `/e2e-investigate` - Debug failing tests
- [Project Type Compatibility Matrix](#project-type-compatibility-matrix) - Which skills work for your project

### 5. Deep Dives (As Needed)
- [Technology Research](#technology-research) - Make skills tech-aware
- [The War on Dirty Data](#the-war-on-dirty-data) - Why explicit failures matter
- [Design for Greppability](#design-for-greppability) - Naming conventions

---

### Manual Installation (Alternative)

If you prefer manual setup:

1. Copy the `skills/` folder to your project's `.claude/skills/` directory:
```bash
cp -r skills/* your-project/.claude/skills/
```

2. Customize the placeholders (see [Customization Guide](#customization-guide))

3. Start using the skills

---

## The Origin Story

AI coding assistants are remarkably powerful - but that power needs structure.
Working with AI is a bit like taming a dragon: exhilarating when it works,
and a learning experience when it doesn't.

Models change with every iteration, and so does how we work with them. Here's
what I've discovered along the way:

- **Regressions happen without guardrails**: Without explicit checks, it's easy
  for changes to quietly undo previous work. I learned to build verification
  into the workflow, not hope for consistency.

- **Feature parity requires attention**: New capabilities can arrive while
  existing behavior drifts. I learned to track what should stay the same,
  not just what should change.

- **UIs need to be made observable**: AI doesn't have native visibility into
  user interfaces. I learned to expose UI state through CLI tools and structured
  output - making the invisible debuggable.

- **Context needs to stay clean**: Fallback values and "sensible defaults"
  accumulate into polluted context that misleads future interactions. I learned
  to be explicit and let things fail loudly rather than silently guess.

- **True norths help, but it's an ongoing journey**: I've developed guiding
  principles that help channel outputs in the right direction. But working
  effectively with AI is continuous learning - there's always more to discover.

There's a method to the madness. The discipline I practiced in full-stack
engineering - explicit contracts, clear boundaries, verifiable state -
makes working with AI deterministic, not hopeful.

The problem was never capability - it was direction. Here's what I use now to
stay pointed at true norths. **The magic isn't in any single skill. It's in
the transitions between them.**

**Pipeline Skills** (connected, sequential):
- `/research` - make skills tech-aware
- `/ux-planner` - think before building
- `/ui-planner` - establish visual identity
- `/ui-review` - catch AI slop patterns
- `/create-task` - implement with tests baked in
- `/coding-guard` - catch anti-patterns early
- `/cli-first` - make code observable
- `/ux-review` - verify from user's eyes
- `/e2e-guard` - ensure coverage exists
- `/e2e` - prove it works end-to-end
- `/e2e-investigate` - diagnose when it doesn't

**Consultation Skills** (standalone, any time):
- `/team` - expert personas for strategic advice
- `/kaizen` - diverse user personas for real-world feedback

---

## The Philosophy

### "The In-Between Is Where Quality Lives"

Traditional AI coding goes straight from idea to implementation:

```
Idea -> Code -> Done
```

This approach fails because it skips the critical thinking steps. My workflow forces structured pauses:

```
Idea -> UX Planning -> Visual Design -> Implementation -> Parallel Verification -> Full Testing -> Done
           |               |                                  |                        |
      FORCED PAUSE    FORCED PAUSE                     QUALITY GATES             RECOVERY LOOPS
                     (optional)
```

Each arrow represents a **structured handoff** - output from one skill becomes input for the next.

### Why This Works

| Principle | What It Means |
|-----------|---------------|
| **Forced pause points** | Can't skip from idea to implementation without UX planning |
| **Parallel verification** | Multiple quality perspectives checked simultaneously |
| **Recovery loops** | Failures don't end the workflow - they feed back into it |
| **Handoff formats** | Each skill's output is designed for the next skill's input |

---

## CLI-First = AI-First Debugging

### The Core Insight

AI assistants are text-native. They read text, write text, and reason about text. When you design your development workflow around CLI tools, you're designing for AI debuggability by default.

```
CLI Output → Text → AI Can Parse → AI Can Debug
GUI Output → Pixels → AI Needs Vision → Expensive & Lossy
```

This isn't about preferring terminals over GUIs for aesthetic reasons. It's about **making your system debuggable by the most capable debugging partner you have**.

### Why CLI Tools Win

| Aspect | CLI Approach | GUI Approach |
|--------|--------------|--------------|
| **Output format** | Text (native to AI) | Pixels (requires vision) |
| **Token cost** | ~200 tokens (curl response) | ~2000+ tokens (screenshot) |
| **Composability** | Pipe, grep, jq | None |
| **Reproducibility** | Copy-paste command | Click sequence |
| **Scriptability** | Immediate | Requires automation framework |

### The Debugging Stack

Every verification method has a cost. Use the cheapest one that answers the question:

```
curl + jq        (~200 tokens)   → API responses, data verification
snapshot + grep  (~500 tokens)   → DOM structure, element presence
JS eval          (~100 tokens)   → State values, internal variables
screenshot       (~2000+ tokens) → Visual layout only (last resort)
```

When something breaks, the AI can:
```bash
# Reproduce the issue
curl -sf "$URL/api/endpoint" | jq '.'

# Check the DOM state
agent-browser snapshot -c | grep "expected-element"

# Inspect internal state
agent-browser eval "window.app.state"
```

No screenshots needed. No vision tokens burned. Just text that AI can read, analyze, and act on.

### Greppability Is Debuggability

If AI can't find your code, it can't fix your code. CLI-first design means:

- **Greppable names**: `test_v4_stage_ordering.sh` not `test_1.sh`
- **Greppable IDs**: `data-testid="bake-generate-btn"` not `data-testid="btn-1"`
- **Greppable errors**: Explicit error messages, not silent fallbacks

When the AI runs `grep -r "stage" webui/`, it should find everything related to stages. No abstractions hiding the truth.

### The Anti-Pattern: Screenshot-Driven Debugging

```bash
# WRONG: Screenshot to check if button exists
agent-browser screenshot /tmp/check.png  # 2000+ tokens, AI has to "look"

# RIGHT: Text query to check if button exists
agent-browser snapshot -c | grep -q "Generate"  # 500 tokens, direct answer
```

Screenshots are reserved for visual verification (`/ux-review`, `/e2e` failure artifacts). Everything else uses text.

### This Shapes Everything

The CLI-first philosophy influences every skill in this pipeline:

| Skill | CLI-First Approach |
|-------|-------------------|
| `/create-task` | Verifies with curl + snapshot, never screenshots |
| `/coding-guard` | Analyzes code with grep patterns |
| `/e2e-guard` | Generates tests using CLI commands |
| `/e2e-investigate` | Reproduces issues with curl + snapshot before analyzing screenshots |
| `/ux-review` | Only skill that screenshots (visual judgment required) |

---

## Why E2E Tests Over Unit Tests

This is a deliberate, experience-driven choice. Here's why:

### The Problem with Unit Tests

Unit tests with mocks create a **false sense of security**:

```
Unit Test Reality:
┌─────────────────────────────────────────────────────────┐
│  You write mock → Mock passes → API changes → BROKEN   │
│  (static)         (false confidence)  (silent failure) │
└─────────────────────────────────────────────────────────┘
```

Mocks are lies we tell ourselves. They test what we *think* the system does, not what it *actually* does. When you mock an API response, you're betting that:
1. The API still returns that exact format
2. The integration between components still works
3. The real database behaves like your mock

These bets fail silently. The tests pass, but the system is broken.

### E2E Tests Are Ground Truth

E2E tests verify what the **user actually experiences**:

```bash
# This test cannot lie - it uses the real system
curl -sf "$BASE_URL/api/endpoint" | jq '.field'
agent-browser open "$URL" && agent-browser snapshot -c | grep "expected text"
```

If the API changes format, the E2E test fails. If the UI doesn't render, the E2E test fails. There's no mock to hide the truth.

### CLI-First + AI = Dynamic Verification

The `/cli-first` skill takes this philosophy further. Instead of writing static tests with frozen assumptions, make your code **observable** so AI can verify it dynamically:

```
CLI-First Reality:
┌─────────────────────────────────────────────────────────┐
│  AI reads system → AI verifies live → System changes → │
│  (dynamic)         (real truth)        AI ADAPTS       │
└─────────────────────────────────────────────────────────┘
```

**The Equation:**
```
Unit Tests + Mocks = Static verification of frozen assumptions
CLI-First + AI    = Dynamic verification of live reality
```

You don't write tests. You make code observable. AI does the rest. See `/cli-first` for details.

### The Integration Gap

The bugs that hurt most are **integration bugs** - where two components that work perfectly in isolation fail when combined:

| Test Type | Catches | Misses |
|-----------|---------|--------|
| Unit tests | Logic errors in isolated functions | Integration failures, API contract changes, race conditions |
| E2E tests | Everything the user can experience | Nothing that matters to users |

I'd rather have fewer tests that tell the truth than many tests that create false confidence.

---

## The War on Dirty Data

### The Problem: Context Pollution

Every fallback value, every "sensible default", every backwards-compatibility shim creates **dirty data** that pollutes the system:

```javascript
// This looks helpful but creates chaos
const composition = params.get('composition') || 42;
```

When something breaks, you're now debugging two possibilities:
1. The real value is wrong
2. The fallback kicked in and hid the real problem

Multiply this by every component, and you have a system where **nothing is trustworthy**. The AI assistant sees dirty data and generates solutions based on lies.

### The False Positive/False Negative Trap

| Pattern | What Happens | Why It's Dangerous |
|---------|--------------|-------------------|
| **Fallback defaults** | System "works" with wrong data | Bug is hidden until it compounds |
| **Silent catch blocks** | Errors disappear | Root cause is never found |
| **Backwards compatibility** | Old and new code paths coexist | Double the complexity, double the bugs |
| **Migration shims** | "Temporary" code becomes permanent | Zombie code that nobody understands |

Each of these creates either:
- **False positives**: Tests pass but system is broken
- **False negatives**: Tests fail for the wrong reason, wasting debugging time

### The Solution: Explicit Failure

```javascript
// WRONG: Silent fallback
const composition = params.get('composition') || 42;

// RIGHT: Explicit failure
const composition = params.get('composition');
if (!composition) {
    return { error: 'composition parameter required', status: 400 };
}
```

When something is wrong, the system should **scream**, not whisper. Explicit errors are:
- Debuggable (you know exactly what failed)
- Actionable (the error message tells you what to fix)
- Honest (no hidden state, no guessing)

---

## Design for Greppability

### Why This Matters for AI

AI assistants use tools like `grep` and `glob` to understand codebases. The more greppable your code, the better the AI can help you.

### Naming Conventions That Help

**File names should be self-descriptive and searchable:**

```
# GOOD: Clear, greppable names
test_v4_bake_modal.sh
test_v4_stage_ordering.sh
webui/v4/js/modal/bake.js
webui/v4/server/api/variants.py

# BAD: Ambiguous, hard to find
test_1.sh
utils.js
helpers.py
```

**Function names should include their purpose:**

```javascript
// GOOD: grep "getComposition" finds this immediately
function getCompositionFromURL() { ... }

// BAD: grep "get" returns 500 results
function get() { ... }
```

**Test IDs should be greppable:**

```html
<!-- GOOD: grep "bake-generate" finds this -->
<button data-testid="bake-generate-btn">Generate</button>

<!-- BAD: grep "btn" returns everything -->
<button data-testid="btn-1">Generate</button>
```

### The Explicitness Principle

Every piece of code should answer these questions without context:

| Question | How to Answer |
|----------|---------------|
| What does this file do? | File name: `test_v4_stage_ordering.sh` |
| What component is this? | Path: `webui/v4/js/modal/bake.js` |
| What does this test verify? | Function name: `test_stages_show_correct_counts()` |
| What element is this? | Test ID: `data-testid="stage-manager-total"` |

### Self-Documenting Structure

```
project/
├── webui/
│   └── v4/
│       ├── js/
│       │   ├── modal/           # Modal components
│       │   │   ├── bake.js      # Bake modal
│       │   │   └── myimages.js  # My Images modal
│       │   ├── sidebar/         # Sidebar components
│       │   └── home/            # Home page components
│       └── server/
│           └── api/             # API endpoints
│               ├── variants.py
│               └── stages.py
└── tests/
    └── v4/
        ├── ui/                  # UI tests
        │   ├── test_v4_bake_modal.sh
        │   └── test_v4_stages.sh
        └── api/                 # API tests
            └── test_v4_variants.sh
```

When an AI needs to find "the stage manager code", it can:
```bash
grep -r "stage" webui/v4/js/          # Find JS
grep -r "stage" webui/v4/server/api/  # Find API
ls tests/v4/ui/test_*stage*.sh        # Find tests
```

No guessing. No digging through abstractions. Direct path to the answer.

---

## The Guardrail Philosophy

### Why Gatekeeping Matters

Without guardrails, AI generates code that:
1. Works in isolation but breaks integration
2. Passes tests that don't test what matters
3. Accumulates technical debt that compounds

The skills in this pipeline act as **gates** that must be passed:

```
/create-task
     |
     v
+----+----+----+----+
|    |    |    |    |
v    v    v    v    |
/coding-guard       |   <- Gate 1: No anti-patterns
/cli-first          |   <- Gate 2: Code is AI-verifiable
/ux-review          |   <- Gate 3: User can actually use it
/e2e-guard          |   <- Gate 4: Tests exist and cover changes
+----+----+----+----+
     |
     v
   /e2e             <- Gate 5: Full system works end-to-end
     |
     v
   DONE
```

Each gate catches a different class of problems:

| Gate | Catches | Examples |
|------|---------|----------|
| `/coding-guard` | Anti-patterns | Fallback defaults, silent catches, state pollution |
| `/cli-first` | Observability gaps | Missing testids, closure state, token-heavy tests |
| `/ux-review` | UX regressions | Broken interactions, missing feedback, confusion |
| `/e2e-guard` | Missing coverage | New code without tests, untested paths |
| `/e2e` | Integration failures | Components that don't work together |

### No Backwards Compatibility Theater

"Backwards compatibility" often means keeping broken behavior working:

```javascript
// This is not backwards compatibility - this is technical debt
if (oldFormat) {
    // Old code path nobody understands
    return legacyTransform(data);
} else {
    // New code path
    return modernTransform(data);
}
```

Now you have:
- Two code paths to maintain
- Two code paths to test
- Two code paths where bugs can hide
- Confusion about which path runs when

**Better approach**: Fix it once, fix it right, remove the old path.

### No Migration Shims

"We'll migrate later" is a lie. Temporary code becomes permanent:

```javascript
// TODO: Remove after migration (added 2 years ago)
function temporaryShim() { ... }
```

Either:
1. Migrate now and delete the old code
2. Don't migrate and keep one code path

Never both.

---

## Making Manual Tests Automatable

### The Problem: AI Can't See

Traditional "manual testing" requires a human to:
1. Open a browser
2. Look at the screen
3. Click things
4. Verify visually

AI can't do this... unless you give it eyes and hands.

### agent-browser: The AI's Eyes and Hands

The `agent-browser` tool makes UI **discoverable** to AI:

```bash
# AI can "see" the page structure
agent-browser snapshot -c
# Output: Text representation of DOM - buttons, inputs, text, testids

# AI can "click" elements
agent-browser click @e1

# AI can "read" state
agent-browser eval "window.state.selectedItem"

# AI can "fill" forms
agent-browser fill @e2 "test value"
```

This transforms "manual testing" into **automated testing that AI can perform**.

### The Screenshot Budget: Context Costs Money

Screenshots are expensive - they consume significant context tokens:

| Method | Context Cost | Use Case |
|--------|--------------|----------|
| `agent-browser snapshot -c` | ~500 tokens | Text verification, element discovery |
| `curl` API response | ~200 tokens | Data verification |
| `agent-browser screenshot` | ~2000+ tokens | Visual verification only |

**Screenshots are reserved for skills that need visual judgment:**

| Skill | Can Screenshot? | Why |
|-------|-----------------|-----|
| `/ux-review` | YES | Needs to see visual hierarchy, spacing, colors |
| `/e2e` | YES | Captures visual state for failure analysis |
| `/create-task` | NO | Uses snapshot + curl (text is enough) |
| `/coding-guard` | NO | Analyzes code, not visuals |
| `/e2e-guard` | NO | Generates tests, uses snapshot for verification |
| `/e2e-investigate` | NO | Prefers live debugging (curl + snapshot); reads /e2e screenshots as fallback |

### Why This Matters

By restricting screenshots to visual verification skills:

1. **Context stays clean** - Implementation skills don't waste tokens on images
2. **Faster execution** - Text operations are faster than image processing
3. **Clear responsibilities** - Visual skills do visual work, others don't pretend to

### The Verification Stack

```
+------------------+------------------+------------------+
|     METHOD       |      COST        |     VERIFIES     |
+------------------+------------------+------------------+
| curl + jq        | Lowest (~200)    | API responses    |
| snapshot -c      | Low (~500)       | DOM structure    |
| snapshot + grep  | Low (~500)       | Text presence    |
| JS eval          | Low (~100)       | State values     |
| screenshot       | High (~2000+)    | Visual layout    |
+------------------+------------------+------------------+
```

**Rule**: Use the cheapest method that answers the question.

```bash
# WRONG: Screenshot to check if button exists
agent-browser screenshot /tmp/check.png  # 2000+ tokens

# RIGHT: Snapshot to check if button exists
agent-browser snapshot -c | grep -q "Generate"  # 500 tokens

# WRONG: Screenshot to check API response
agent-browser screenshot /tmp/api.png

# RIGHT: curl to check API response
curl -sf "$URL/api/status" | jq '.success'  # 200 tokens
```

### Test IDs Make AI Testing Possible

Without test IDs, AI has to guess:

```bash
# BAD: Fragile, depends on text that might change
agent-browser find text "Submit Form" click

# GOOD: Stable, greppable, explicit
agent-browser click '[data-testid="form-submit-btn"]'
```

Test IDs are the **contract** between implementation and testing. They make the UI:
- **Discoverable** - AI can find elements reliably
- **Stable** - Changes to text don't break tests
- **Greppable** - `grep "form-submit"` finds both implementation and tests

---

## The Execution Flow

```
+--------------------------------------------------------------------------+
|                     CYBERVALDEZ SKILLS WORKFLOW                           |
|                  "The In-Between Is Where Quality Lives"                  |
+--------------------------------------------------------------------------+
|                                                                           |
|  +------------------+                                                     |
|  |   /ux-planner    |  <- FORCED PAUSE: Think before coding              |
|  +--------+---------+                                                     |
|           |                                                               |
|           | ==> (optional) /cli-first --design for observability guidance |
|           |                                                               |
|           | ==> HANDOFF: UX flows ready for visual design                 |
|           v                                                               |
|  +------------------+                                                     |
|  |   /ui-planner    |  <- VISUAL DESIGN: Aesthetic + layout + tokens     |
|  +--------+---------+    (optional but recommended for UI-heavy projects) |
|           |                                                               |
|           | ==> Generates preview files                                   |
|           v                                                               |
|  +------------------+                                                     |
|  |   /ui-review     |  <- SLOP GUARD: Catch AI-generic patterns          |
|  +--------+---------+                                                     |
|           |                                                               |
|           | ==> FAIL? Loop back to /ui-planner with suggestions           |
|           | ==> PASS? Proceed to implementation                           |
|           v                                                               |
|  +------------------+                                                     |
|  |   /create-task   |  <- IMPLEMENTATION: Code + tests                   |
|  +--------+---------+                                                     |
|           |                                                               |
|           | ==> PARALLEL VERIFICATION (non-negotiable)                    |
|           v                                                               |
|  +------------------------------------------------------------------+    |
|  |  /coding-guard  |  /cli-first   |  /ux-review  |  /e2e-guard    |    |
|  |  (code quality) | (observability)| (user POV)  |  (test coverage)|    |
|  +--------+--------+-------+-------+------+-------+---------+------+    |
|           |                |              |                 |            |
|           +----------------+--------------+-----------------+            |
|                            |                                             |
|           | ==> ISSUES? Loop back to /create-task with specific tasks    |
|           v                                                              |
|  +------------------+                                                    |
|  |      /e2e        |  <- FULL VERIFICATION: Complete test suite         |
|  +--------+---------+                                                    |
|           |                                                              |
|           | ==> FAILURE? /e2e-investigate -> diagnose -> /create-task    |
|           v                                                              |
|       DONE                                                               |
|                                                                          |
|  UTILITY: /agent-browser (powers browser automation across all skills)   |
|                                                                          |
+--------------------------------------------------------------------------+
```

---

## Skills Reference

### Core Skills (All Project Types)

| Skill | Purpose | Connects To |
|-------|---------|-------------|
| `/research` | Research technologies and classify their pipeline impact | Enables tech-aware skills |
| `/ux-planner` | Conversational UX advisor - asks questions, explores options, plans interactions | Outputs to `/ui-planner` or `/create-task` |
| `/ui-planner` | Visual design advisor - establishes aesthetic, layout, and design tokens | Outputs to `/ui-review` |
| `/ui-review` | Visual design quality guard - detects AI slop and generic patterns | Feeds back to `/ui-planner` or outputs to `/create-task` |
| `/create-task` | Implementation with built-in e2e testing requirements | Triggers `/coding-guard`, `/cli-first`, `/ux-review`, `/e2e-guard` |
| `/coding-guard` | Scans code changes for anti-patterns and convention violations | Reports issues to fix |
| `/cli-first` | Enforces code observability for AI-powered dynamic verification | Complements `/coding-guard` |
### Browser-Based Skills (python-cli-with-webui, nextjs-with-cli, react-with-cli)

| Skill | Purpose | Connects To |
|-------|---------|-------------|
| `/ux-review` | Takes screenshots, verifies from user perspective | Identifies UX gaps |
| `/e2e-guard` | Ensures test coverage exists for changes | Auto-generates missing tests |
| `/e2e` | Full end-to-end test orchestration | On failure, triggers `/e2e-investigate` |
| `/e2e-investigate` | Diagnoses test failures, generates actionable tasks | Outputs to `/create-task` |

### Consultation Skills (Non-Pipeline)

These skills provide feedback and advice but don't connect to the pipeline. Use them anytime for perspective.

| Skill | Purpose | Key Difference |
|-------|---------|----------------|
| `/team` | Summon expert personas (Marketing, UX, Product, etc.) for strategic advice | Professionals giving expert opinions |
| `/kaizen` | Gather feedback from diverse fictional personas representing real users | Regular people giving real-world reactions |

**When to use which:**
- `/team` for strategic questions: "How should I position this feature?" "Is my pricing strategy sound?"
- `/kaizen` for user perspective: "Will my grandmother understand this?" "Can a colorblind user see my errors?"

### /team vs /kaizen Decision Tree

```
                    ┌─────────────────────────────┐
                    │   What kind of feedback?    │
                    └─────────────┬───────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              v                   v                   v
      Expert opinion?     User perspective?    Accessibility?
              │                   │                   │
              v                   v                   v
           /team              /kaizen             /kaizen
```

| Question Type | Use | Example Personas |
|---------------|-----|------------------|
| Pricing strategy | `/team` | Business, Marketing |
| Feature prioritization | `/team` | Product, Technical |
| "Will this confuse users?" | `/kaizen` | Grandma Dorothy, Frustrated Frank |
| "Can colorblind users use this?" | `/kaizen` | Marcus (colorblind) |
| "How should we position this?" | `/team` | Marketing, Content |
| "Is this form usable?" | `/kaizen` | Grandma Dorothy, Priya (motor) |
| "Is this architecture scalable?" | `/team` | Technical |
| "What do stakeholders think?" | `/kaizen internal` | Tommy (intern), Sales, Support |

### Utility Skills

| Skill | Purpose |
|-------|---------|
| `/agent-browser` | Browser automation - powers all UI testing and verification |

### About agent-browser

**agent-browser** is a CLI tool that gives AI the ability to control a browser programmatically. It's developed by Vercel Labs and provides:

- **Accessibility-tree snapshots** - Text representation of the page structure
- **Element references** - Stable `@e1`, `@e2` refs for clicking/filling
- **JavaScript evaluation** - Direct access to page state
- **Form automation** - Fill, click, select, check operations
- **Screenshot capture** - Visual verification when needed

The `skills/browser/agent-browser/SKILL.md` included here is a **quick reference copy** for convenience. For full documentation, installation instructions, and updates:

**GitHub: [github.com/vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser)**

```bash
# Installation (see repo for latest instructions)
npx agent-browser

# Basic usage
agent-browser open "https://example.com"
agent-browser snapshot -i          # Interactive elements with refs
agent-browser click @e1            # Click by reference
agent-browser eval "document.title" # Execute JS
agent-browser close
```

---

## Skill Prerequisite Matrix

Understanding which skills depend on others helps avoid errors and ensures proper workflow.

| Skill | Hard Prerequisites | What Happens | **Next Step** |
|-------|-------------------|--------------|---------------|
| `/ui-review` | Preview files exist | Error: "No preview files found" | **Run `/ui-planner` first** |
| `/ux-review` | Server running | Error: "Server not running" | **Run `{{SERVER_START}}`** |
| `/e2e` | Test files exist, server running | Error: "No test files found" | **Run `/e2e-guard` to generate tests** |
| `/e2e-investigate` | `/e2e` failed with artifacts | Nothing to investigate | **Run `/e2e` first** |
| `/coding-guard` | Git diff available | Uses `--cached` for staged | **Run `git add .` then retry** |
| `/e2e-guard` | Git diff available | Uses `--cached` for staged | **Run `git add .` then retry** |

---

## Project Type Compatibility Matrix

Not all skills apply to all project types. Browser-based skills require UI.

| Skill | python-cli-with-webui | nextjs-with-cli | react-with-cli | CLI-only | API-only |
|-------|----------------------|-----------------|----------------|----------|----------|
| `/ux-planner` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/ui-planner` | ✓ | ✓ | ✓ | ✗ | ✗ |
| `/ui-review` | ✓ | ✓ | ✓ | ✗ | ✗ |
| `/create-task` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/coding-guard` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/cli-first` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `/ux-review` | ✓ | ✓ | ✓ | ✗ | ✗ |
| `/e2e-guard` | ✓ | ✓ | ✓ | Partial | Partial |
| `/e2e` | ✓ | ✓ | ✓ | ✗ | ✗ |
| `/e2e-investigate` | ✓ | ✓ | ✓ | ✗ | ✗ |
| `/research` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/team` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/kaizen` | ✓ | ✓ | ✓ | ✓ | ✓ |

**Notes:**
- **CLI-only**: Projects without web UI (e.g., command-line tools)
- **API-only**: Backend services with no frontend
- **Partial**: `/e2e-guard` can generate API tests via `curl` but not browser tests

---

## Error Recovery Guide

What to do when skills fail or find issues.

### /e2e Fails Repeatedly

| Failure Count | Action |
|---------------|--------|
| 1-2 failures | Fix the issue, re-run `/e2e` |
| 3+ failures | Stop retrying. Run `/e2e-investigate` for root cause analysis |
| Flaky tests | Check for timing issues (insufficient `sleep`), race conditions |

**Circuit breaker**: If `/e2e` fails 3+ times on the same test, `/e2e-investigate` is mandatory before continuing.

### /coding-guard Finds Issues

```
/coding-guard finds violations
       │
       v
┌──────────────────────────────────────┐
│ Review each violation in report      │
│                                      │
│ For each issue:                      │
│   1. Understand WHY it's flagged     │
│   2. Fix the code                    │
│   3. Stage the fix (git add)         │
│   4. Re-run /coding-guard            │
│                                      │
│ Loop until: No violations remain     │
└──────────────────────────────────────┘
```

### /ui-review Finds AI Slop

```
/ui-review → NEEDS ITERATION
       │
       v
Option A: Fix in preview file
   → Edit webui/previews/preview-*.html directly
   → Re-run /ui-review

Option B: Regenerate with /ui-planner
   → /ui-planner with specific feedback
   → e.g., "regenerate with sharper corners"
```

### Preview Files Deleted

If preview files are deleted before `/ui-review`:
1. Run `/ui-planner` again to regenerate
2. Or manually create preview file based on styleguide

### Git History Empty (First Commit)

For `/coding-guard` or `/e2e-guard` on a new repo:
```bash
# Use staged changes instead of committed diff
git add .
/coding-guard  # Will use --cached flag automatically
```

### Git State Assumptions

Skills that use `git diff` handle various states:

| Git State | What Happens |
|-----------|--------------|
| Normal (has commits + changes) | Uses `git diff HEAD` |
| First commit ever | Uses `git diff --cached` (staged files) |
| Clean working tree | Nothing to audit - immediate success |
| No git repo | Error with instructions to `git init` |
| Untracked files only | Stage first with `git add`, then run skill |

**If git is not configured:**
```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

### Server Not Running

All browser-based skills require a running server:
```bash
# Start server
{{SERVER_START}}

# Wait for ready (check every 2s, timeout 30s)
for i in {1..15}; do
  curl -sf http://localhost:{{SERVER_PORT}} && echo "✓ Server ready" && break
  sleep 2
done

# Then run skill
/ux-review
```

---

## Technology Research

> **IMPORTANT:** Technology research happens in your **kickstarted project**, not the playbook repo. This is by design:
> - Projects have real context (file structure, conventions, existing techs)
> - Reference docs are project-specific, not universal
> - Without project context, research is speculative

The playbook uses **two-phase research** to make skills tech-aware without slowing developers down:

> **Note:** The playbook assumes a single-project context. For monorepos, scope your tasks to specific subdirectories. See `skills/TECH_CONTEXT.md` for full assumptions.

### Phase 1: Tech Assessment (On Install/Mention)

When you mention a new technology in your **kickstarted project**, run `/research {tech}` to create a lightweight assessment:

```
/research xstate
/research tanstack-query
```

**The research flow includes user confirmation:**
1. AI researches the technology via web search
2. AI presents findings (domain, pipeline impact, key patterns)
3. **You confirm or correct** the classification
4. Only after confirmation: `techs/{tech}/README.md` is saved

This confirmation step ensures domain classification matches your actual use case, not assumptions.

### Phase 2: Skill Reference (Automatic)

When a skill is invoked with tech context, it automatically:
1. Checks for `references/{tech}.md` in its directory
2. If missing AND the tech's domain affects the skill, produces the reference doc
3. Applies tech-specific patterns during execution

This lazy evaluation means docs are produced when needed, based on actual usage context.

### Domain Classification Table

| Domain | Examples | Skills Affected |
|--------|----------|-----------------|
| **State Management** | XState, Redux, Zustand, Jotai | coding-guard, cli-first, create-task |
| **UI Components** | Radix, Shadcn, MUI, Chakra | ux-planner, ui-planner, ui-review, ux-review, create-task |
| **Data Fetching** | TanStack Query, SWR, tRPC, Apollo | coding-guard, e2e-guard, create-task |
| **Form Handling** | React Hook Form, Formik, Zod | ux-planner, coding-guard, e2e-guard |
| **Animation** | Framer Motion, GSAP, React Spring | ux-review, ui-planner, ui-review, e2e (wait patterns) |
| **Routing** | React Router, TanStack Router | create-task, e2e, e2e-guard |
| **Testing Tools** | Playwright, Vitest, Jest | e2e, e2e-guard, e2e-investigate |
| **Build Tools** | Vite, Turbopack, esbuild | create-task, e2e (server startup) |
| **Styling** | Tailwind, CSS Modules | ux-review, ux-planner, ui-planner, ui-review |
| **Auth** | NextAuth, Clerk, Auth0 | coding-guard, e2e, create-task |

### Skill Concern Matrix

Each skill evaluates specific concerns when deciding whether to produce a tech reference:

| Skill | Concerns |
|-------|----------|
| **create-task** | File structure? Scaffolding? Test organization? Debug containers? |
| **coding-guard** | Anti-patterns? Silent failures? State mutation? Error handling? |
| **cli-first** | State exposure? TestID conventions? Verification commands? Token costs? |
| **ux-planner** | Component constraints? Async feedback? Accessibility? Form patterns? |
| **ui-planner** | Design tokens? Animation constraints? Component theming? Layout systems? |
| **ui-review** | Typography validation? Color system check? Animation capabilities? Brand alignment? |
| **e2e-guard** | Coverage patterns? Element selection? API verification? State assertions? |
| **e2e** | Server startup? Artifact paths? Timing/waits? Cleanup? |
| **e2e-investigate** | Failure patterns? Log formats? Reproduction? Debug tools? |
| **ux-review** | Visual patterns? Animations? Design system? Responsive? |

**Threshold:** If 2+ concerns are relevant, the skill produces a reference doc.

See `skills/TECH_CONTEXT.md` for the complete reference.

---

## The In-Between Actions

### Transition 1: UX Planning -> Visual Design (Optional)

```
/ux-planner produces:
  - User goal description
  - Recommended approach with rationale
  - Interaction flow diagram
  - Key UX requirements checklist

                    |
                    v

/ui-planner consumes this and establishes visual identity:
  - Aesthetic direction (from gallery selection)
  - Layout patterns
  - Design tokens (styleguide.css)
  - HTML preview files

                    |
                    v

/ui-review validates the design:
  - Styleguide compliance check
  - AI slop pattern detection
  - Context-specific audit

                    |
         PASS?      |      FAIL?
          |         |         |
          v         |         v
    /create-task <--|-- /ui-planner (iterate)
```

**The ASCII Gallery Drill-Down Process**

`/ui-planner` forces explicit visual decisions via interactive ASCII galleries:

```
STEP 1: Aesthetic Selection (A-H) — visual thumbnails of each style
┌─────────────────────────────────────────────────────────────────┐
│  [A] BRUTALIST        [B] NEO-MINIMAL      [C] MAXIMALIST       │
│  +---------------+    +---------------+    +---------------+    │
│  | ############  |    |               |    | ***  +++  @@@ |    │
│  | ##  TEXT  ##  |    |       o       |    | +++  @@@  *** |    │
│  | ############  |    |    ______     |    | @@@  ***  +++ |    │
│  +---------------+    +---------------+    +---------------+    │
│  Raw, bold, no        Whitespace-focused,  Layered, textured,   │
│  curves               breathing room       rich density         │
└─────────────────────────────────────────────────────────────────┘
→ User selects ONE letter (e.g., "D")

STEP 2: Layout Selection (1-8) — visual wireframes
┌─────────────────────────────────────────────────────────────────┐
│  [1] SIDEBAR          [2] DASHBOARD        [3] CARD GRID        │
│  +---------------+    +---------------+    +---------------+    │
│  | +--+--------+ |    | +--+--+-----+ |    | +--+ +--+ +--+|    │
│  | |  |        | |    | |  |  |     | |    | |  | |  | |  ||    │
│  | |  |        | |    | +--+--+-----+ |    | +--+ +--+ +--+|    │
│  | +--+--------+ |    | +-----------+ |    | +--+ +--+ +--+|    │
│  +---------------+    +---------------+    +---------------+    │
│  Nav left,            Metrics top,         Uniform grid of      │
│  content right        content below        repeating items      │
└─────────────────────────────────────────────────────────────────┘
→ User multi-selects (e.g., "1, 3")

STEP 3: Generate HTML Previews → each aesthetic+layout combo
STEP 4: Iterate → "like X but with Y's header" → hybrid generated
STEP 5: "this one" → design tokens committed to styleguide.css
```

The visual ASCII art makes each option immediately scannable - users see the
structure before committing. This prevents defaulting to generic AI aesthetics.

The ui-planner/ui-review loop ensures designs are distinctive, not generic AI output. Skip this for non-UI-heavy features.

### Transition 2: Implementation -> Quality Gates

```
/create-task produces:
  - Code changes (git diff)
  - E2E test files

                    |
                    v

/coding-guard, /cli-first, /ux-review, /e2e-guard run in parallel:
  - coding-guard: Checks anti-patterns, conventions
  - cli-first: Verifies code is observable for AI testing
  - ux-review: Takes screenshots, verifies user experience
  - e2e-guard: Ensures test coverage exists
```

These run simultaneously as non-negotiable checkpoints. All three must pass.

### Transition 3: Failure -> Recovery

```
/e2e finds failures:
  - Screenshots of failed states
  - Console errors
  - Test output

                    |
                    v

/e2e-investigate produces:
  - Root cause analysis
  - Specific tasks for /create-task
  - Suggested fixes

                    |
                    v

/create-task implements the fix

Loop continues until all tests pass
```

Failures aren't dead ends - they're feedback that feeds back into the system.

---

## Project Types

### python-cli-with-webui

**Best for:**
- Python backend (Flask, FastAPI, Django, custom)
- Vanilla JavaScript frontend (no framework)
- CLI-first development workflow
- Bash + agent-browser testing

**Testing stack:**
- E2E: Bash scripts with `agent-browser` + `curl`
- No unit tests (CLI-first philosophy)

**Examples:**
- Flask + jQuery apps
- FastAPI + HTMX applications
- Python tools with web dashboards

**Key conventions:**
- Python: `snake_case` naming
- JavaScript: `camelCase` naming
- Tests: `test_v4_feature.sh` (bash)
- TestIDs: `component-element-action`
- Debug containers for state visibility

### nextjs-with-cli

**Best for:**
- Next.js applications
- TypeScript or modern JavaScript
- Modern React patterns (hooks, function components)
- CLI-first development workflow

**Testing stack:**
- E2E: Bash scripts with `agent-browser` + `curl`
- No unit tests (CLI-first philosophy)
- No mocks (tests against real server)

**Examples:**
- Next.js applications
- Next.js + API routes

**Key conventions:**
- Components: `PascalCase.tsx`
- Hooks: `useCamelCase.ts`
- Tests: `test_feature.sh` (bash)
- TestIDs: `component-element-action`
- Debug containers for state visibility
- Expose state via `window.appState`

**Observability:**
- Debug containers in JSX
- Window object for state exposure
- React DevTools compatibility

### react-with-cli

**Best for:**
- React SPAs with Vite
- React + Express apps
- TypeScript or modern JavaScript
- CLI-first development workflow

**Testing stack:**
- E2E: Bash scripts with `agent-browser` + `curl`
- No unit tests (CLI-first philosophy)
- No mocks (tests against real server)

**Examples:**
- React SPAs with Vite
- React + Express apps

**Key conventions:**
- Components: `PascalCase.tsx`
- Hooks: `useCamelCase.ts`
- Tests: `test_feature.sh` (bash)
- TestIDs: `component-element-action`
- Debug containers for state visibility
- Expose state via `window.appState`

**Observability:**
- Debug containers in JSX
- Window object for state exposure
- React DevTools compatibility

---

### Why All Project Types Use Bash Tests?

All project types share the same testing philosophy:

| Aspect | Reason |
|--------|--------|
| **Bash scripts** | Text-native, AI can read/write/debug |
| **No unit tests** | E2E tests verify user experience, not implementation |
| **No mocks** | Tests run against real system, catch integration bugs |
| **agent-browser + curl** | CLI tools compose (pipe, grep, jq) |
| **Same patterns** | Skills work identically across both types |

**The only difference:** Python vs React/Next.js implementation. Testing approach is identical.

---

### Tooling Requirements

All project types require **Node.js and Python** installed:

| Project Type | Node.js | Python | Why |
|--------------|---------|--------|-----|
| **python-cli-with-webui** | Optional (for npm packages) | Required | Backend runtime + agent-browser |
| **nextjs-with-cli** | Required | Required | Next.js runtime + agent-browser |
| **react-with-cli** | Required | Required | React runtime + agent-browser |

**Why Python for a React project?**

The `agent-browser` CLI tool is written in Python. This is intentional:

1. **AI models are trained more on Python** - Python dominates AI/ML training data, making AI agents significantly better at reading, writing, and debugging Python code compared to JavaScript
2. **CLI tools benefit from Python's ecosystem** - Browser automation, text processing, and system scripting are where Python excels
3. **Consistency across project types** - Using the same testing tool (agent-browser) for both Python and React projects means identical test patterns

**For AI development teams:**

Most AI projects already use Python (torch, transformers, langchain), so having both runtimes isn't additional overhead. The bash + agent-browser approach provides consistent testing whether your backend is Python or Node.js.

**Tool installation:**

```bash
# Node.js (if using nextjs-with-cli or react-with-cli)
# Install from nodejs.org or via nvm

# Python 3.10+ (all project types)
# Install from python.org or via pyenv

# agent-browser (installed during kickstart)
pip install agent-browser
```

### Adding More Project Types

Want to add your own project type? Edit `KICKSTART.md` to add a new project type with:
- Detection logic and defaults
- Project-specific file structure
- Conventions and patterns

---

## Customization Guide

### Automatic Customization (Recommended)

Using KICKSTART.md automatically:
1. Detects your project structure
2. Replaces all placeholders with your values
3. Generates `.claude/PROJECT_CONFIG.md` with your settings

After kickstart completes, check:
- `.claude/PROJECT_CONFIG.md` - Your configured values
- `.claude/skills/` - All 14 skills installed and configured

### Manual Customization

If you installed manually, the skills use these placeholders that you'll need to customize:

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `{{PROJECT_NAME}}` | Your project name | `MyApp` |
| `{{TEST_JOB}}` | Test job/fixture name | `test-fixtures` |
| `{{SERVER_PORT}}` | Default server port | `8080` |
| `{{SERVER_START}}` | Server startup command | `./start.sh` |
| `{{VENV_PYTHON}}` | Python path | `./venv/bin/python` |
| `{{API_BASE}}` | API base path | `/api` |
| `{{WEBUI_PATH}}` | WebUI source path | `src/frontend` |
| `{{TESTS_PATH}}` | Test files path | `tests/` |
| `{{OUTPUTS_PATH}}` | Generated outputs path | `outputs/` |

### Skills Requiring Customization

| Skill | What to Customize |
|-------|-------------------|
| `create-task` | Project structure, test patterns, coding conventions |
| `coding-guard` | Anti-patterns specific to your stack |
| `e2e-guard` | File paths, test discovery patterns |
| `e2e` | Server startup, cleanup paths, test phases |
| `e2e-investigate` | Artifact paths, log locations |
| `testing-conventions.md` | Your project's testing patterns |

### Generic Skills (No Customization Needed)

These work as-is across any web project:

- `ux-planner/` - Universal UX principles and patterns
- `ui-planner/` - Visual design advisor and preview generation
- `ui-review/` - AI slop detection and design quality guard
- `ux-review/` - Generic UX verification workflow
- `agent-browser/` - Browser automation utility

See `KICKSTART.md` "Post-Kickstart: Optional Customizations" section for customization examples.

---

## Reference Files

### Included References

| File | Purpose |
|------|---------|
| `skills/TECH_CONTEXT.md` | Preflight assumptions, domain classification, and skill concern matrix |
| `skills/PROJECT_CONTEXT.md` | Content archetype taxonomy for project-aware skill recommendations |
| `skills/core/ux-planner/references/ux-patterns.md` | UX laws, principles, and patterns database |
| `skills/core/ui-planner/references/ui-patterns.md` | Design tokens, typography, color theory, and visual patterns |
| `skills/core/ui-review/references/slop-research.md` | AI slop research methodology and detection patterns |
| `skills/core/create-task/references/testing-conventions.md` | Testing patterns and best practices |
| `skills/core/cli-first/references/cli-patterns.md` | Token cost table and universal CLI-first patterns |
| `skills/core/team/references/expert-personas.md` | Expert persona definitions and interaction patterns |
| `skills/core/kaizen/references/persona-roster.md` | Diverse user persona definitions and feedback patterns |

### What's Universal

The UX patterns reference includes timeless principles that apply to any web project:
- Fitts's Law, Hick's Law, Miller's Law
- Progressive disclosure, feedback loops, error prevention
- Modal patterns, toast notifications, loading states
- Accessibility guidelines (WCAG)
- Mobile considerations

---

## Project Structure

```
.
├── .claude/
│   └── skills/
│       └── research/
│           └── SKILL.md               # /research — technology research
├── README.md                           # This file
├── KICKSTART.md                        # AI prompt guide for kickstarting projects
├── LICENSE                             # MIT License
├── skills/
│   ├── TECH_CONTEXT.md                 # Domain classification & skill concern matrix
│   ├── PROJECT_CONTEXT.md              # Content archetype taxonomy
│   ├── core/                           # Always installed (all project types)
│   │   ├── ux-planner/
│   │   │   ├── SKILL.md
│   │   │   └── references/
│   │   │       └── ux-patterns.md
│   │   ├── ui-planner/
│   │   │   ├── SKILL.md
│   │   │   ├── references/
│   │   │   │   └── ui-patterns.md
│   │   │   └── templates/
│   │   │       └── preview-base.html
│   │   ├── ui-review/
│   │   │   ├── SKILL.md
│   │   │   └── references/
│   │   │       └── slop-research.md
│   │   ├── create-task/
│   │   │   ├── SKILL.md
│   │   │   └── references/
│   │   │       └── testing-conventions.md
│   │   ├── coding-guard/
│   │   │   └── SKILL.md
│   │   ├── cli-first/
│   │   │   ├── SKILL.md
│   │   │   └── references/
│   │   │       └── cli-patterns.md
│   │   ├── team/                        # Expert consultation (non-pipeline)
│   │   │   ├── SKILL.md
│   │   │   └── references/
│   │   │       └── expert-personas.md
│   │   ├── kaizen/                      # User feedback (non-pipeline)
│   │   │   ├── SKILL.md
│   │   │   └── references/
│   │   │       └── persona-roster.md
│   │
│   ├── browser/                        # For browser-testing projects
│   │   ├── references/
│   │   │   └── element-operations.md
│   │   ├── ux-review/
│   │   │   └── SKILL.md
│   │   ├── e2e-guard/
│   │   │   └── SKILL.md
│   │   ├── e2e/
│   │   │   └── SKILL.md
│   │   ├── e2e-investigate/
│   │   │   └── SKILL.md
│   │   └── agent-browser/
│   │       └── SKILL.md
```

**After kickstart, your project is a subfolder:**
```
cybervaldez-playbook/                   # Factory (untouched, reusable)
├── KICKSTART.md
├── UPGRADE.md
├── README.md
├── skills/                             # Source skills
└── your-project/                       # Created by kickstart (gitignored)
    ├── README.md                       # Project-specific README
    ├── techs/                          # Technology research output
    │   └── README.md
    ├── .claude/
    │   ├── PROJECT_CONFIG.md           # Your configured values
    │   └── skills/                     # Installed skills (copied from ../skills/)
    │       ├── TECH_CONTEXT.md
    │       ├── SKILL_INDEX.md
    │       ├── research/
    │       ├── ux-planner/
    │       ├── ui-planner/
    │       ├── create-task/
    │       ├── coding-guard/
    │       └── ...
    └── .git/                           # Own git history
```

---

## License

MIT License - see [LICENSE](LICENSE)

---

## About Cyber Valdez

I'm a developer who believes AI should make us more productive, not just faster at creating technical debt. These skills represent my current approach to AI-assisted development.

The key insights:

1. **Structure and guardrails don't slow you down** - they prevent the rework that really slows you down.

2. **E2E tests are honest** - they test what users experience, not what you hope the system does.

3. **Dirty data is the enemy** - fallbacks, shims, and "temporary" code create bugs that hide for months.

4. **Greppability is a feature** - when AI can find your code, it can help you fix it.

5. **The in-between matters** - the handoffs between skills are where quality is enforced.

Questions? Issues? Find me at:
- LinkedIn: [linkedin.com/in/cybervaldez](https://www.linkedin.com/in/cybervaldez)
- Email: cybervaldez@gmail.com

---

*"The in-between is where quality lives."*
