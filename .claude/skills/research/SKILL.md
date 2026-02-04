---
name: research
description: Research technologies and produce reference documents that make base skills tech-aware.
argument-hint: <technology name>
---

# /research — Technology Research

Research a technology and produce reference documents for playbook skills. Evaluates each base skill and produces a reference doc only where the tech genuinely adds context.

## What It Does

1. Identify and research the technology via web search
2. Walk through each base skill — evaluate if {tech} changes how it operates
3. Produce reference docs in relevant skill `references/` directories
4. Produce a skill recommendations document

The tech name (e.g., "xstate") is used in the filename for greppability. Prefix/suffix allowed for context (e.g., `xstate-testing.md`).

## Usage

/research xstate
/research remotion
/research {any-technology-name}

---

## Step 1: Check for Existing Research

Search for `{tech}` in all `references/` directories across `skills/core/` and `skills/browser/`:

```
skills/core/*/references/*{tech}*
skills/browser/*/references/*{tech}*
```

- If reference docs exist → report what's already there. Ask user: update existing docs or done?
- If none exist → proceed to Step 2.

---

## Step 2: Identify the Technology

Web search for "{tech} javascript library" or "{tech} npm package" to find matches.

If multiple matches:

Which {tech}?
  1. {Best match} — {one-line description}
  2. {Second match} — {one-line description}
  3. {Third match} — {one-line description}
  4. Something else

If one clear match, confirm directly:
{tech}: {one-line description}. Correct?

---

## Step 3: Research

Using the confirmed identity, web search for:
- Core concepts and API patterns
- Ecosystem (packages, tools, integrations)
- Testing approaches and patterns
- Common gotchas and anti-patterns
- How it integrates with React/Next.js projects

---

## Step 4: Evaluate Each Base Skill

Read each skill's SKILL.md and ask: "Does {tech} change how this skill operates? Would a developer need {tech}-specific guidance when running this skill?"

Walk through each — **skip if no genuine context to add**:

| Skill | Evaluate for |
|-------|-------------|
| `skills/core/create-task/` | Scaffolding patterns, file structures, conventions |
| `skills/core/coding-guard/` | Code quality concerns, anti-patterns to check |
| `skills/core/ux-planner/` | UX planning, component design implications |
| `skills/core/cli-first/` | CLI tooling, terminal workflows |
| `skills/browser/e2e/` | E2E test structure or execution |
| `skills/browser/e2e-guard/` | Test coverage semantics |
| `skills/browser/e2e-investigate/` | Unique failure modes to diagnose |
| `skills/browser/ux-review/` | Visual review criteria |

### Present suggestions to the user

For each skill where {tech} adds genuine context, present the suggested reference doc with:
- **Skill name** and proposed **filename**
- **Purpose** — what the doc will cover and why this skill needs it

Example format:

```
Suggested reference docs for {tech}:

1. create-task/references/{tech}.md
   Purpose: Scaffolding patterns for {tech} — file structure conventions,
   boilerplate templates, naming patterns.

2. coding-guard/references/{tech}.md
   Purpose: Anti-patterns specific to {tech} — common misuse patterns
   to flag during code review.

3. (skipped) ux-planner — {tech} doesn't change UX planning guidance.
4. (skipped) cli-first — no {tech}-specific CLI patterns.
...

Proceed with all, or discuss any of these first?
```

The user may:
- **Proceed** — produce all suggested docs
- **Discuss** — ask questions, suggest improvements, request changes to scope or content for specific docs
- **Skip** — remove specific docs from the list

Wait for user confirmation before producing any docs.

---

## Step 5: Produce Reference Docs

For each relevant skill, create a file in its `references/` directory:

- **Filename:** tech name with optional descriptive suffix
  (e.g., `xstate.md`, `xstate-testing.md`, `remotion-video.md`)
- **One file per skill** — all context for that skill+tech in one document
- **Content:** practical and actionable — code examples, shell commands, conventions
- **Style:** follow existing references (e.g., `video-testing-conventions.md`)
- **No theory/overview** — just patterns the skill needs

---

## Step 6: Skill Recommendations

Produce a `{tech}-recommendations.md` in the techs directory: `techs/{tech}-recommendations.md`

Contents:
- Which base skills have reference docs and what they cover
- Example prompts showing how to use base skills with this tech
  (e.g., "create a checkout wizard using xstate", "run e2e on the auth flow")
- Which skills were evaluated and skipped, with rationale
- Compatible project types

---

## After Research

Report to the user:
- Reference docs produced and their locations
- Skills evaluated and skipped
- Example prompts for using the tech with base skills
