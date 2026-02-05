# Tech Context Detection

Shared reference for all skills to detect and handle technology-specific patterns.

---

## Two-Phase Research

Skills use a lazy evaluation approach to tech-specific documentation:

1. **Phase 1 (On tech mention):** `/research` creates `techs/{tech}/README.md` with domain classification
2. **Phase 2 (On skill invocation):** Each skill produces `references/{tech}.md` when needed

---

## Domain Classification Table

| Domain | Examples | Skills Affected |
|--------|----------|-----------------|
| **State Management** | XState, Redux, Zustand, Jotai, MobX | coding-guard, cli-first, create-task |
| **UI Components** | Radix, Shadcn, MUI, Chakra, Ant Design | ux-planner, ux-review, create-task |
| **Data Fetching** | TanStack Query, SWR, tRPC, Apollo | coding-guard, e2e-guard, create-task |
| **Form Handling** | React Hook Form, Formik, Zod, Yup | ux-planner, coding-guard, e2e-guard |
| **Animation** | Framer Motion, GSAP, React Spring | ux-review, e2e (wait patterns) |
| **Routing** | React Router, TanStack Router, Next.js | create-task, e2e, e2e-guard |
| **Testing Tools** | Playwright, Vitest, Jest, Cypress | e2e, e2e-guard, e2e-investigate |
| **Build Tools** | Vite, Turbopack, esbuild, Webpack | create-task, e2e (server startup) |
| **Styling** | Tailwind, CSS Modules, Styled Components | ux-review, ux-planner |
| **Auth** | NextAuth, Clerk, Auth0, Supabase Auth | coding-guard, e2e, create-task |

---

## Skill Concern Matrix

Each skill has fixed concerns. When producing a reference doc, evaluate these against the tech:

### create-task
| Concern | Question |
|---------|----------|
| File structure | Does this tech require specific folder organization? (e.g., `machines/`, `stores/`) |
| Scaffolding | Does this tech have boilerplate patterns or generators? |
| Test organization | Does this tech change where/how tests should be structured? |
| Debug containers | Does this tech require special debug visualization? |

### coding-guard
| Concern | Question |
|---------|----------|
| Anti-patterns | Does this tech have known misuse patterns to flag? |
| Silent failures | Does this tech have failure modes that don't throw errors? |
| State mutation | Does this tech have immutability or mutation gotchas? |
| Error handling | Does this tech have unconventional error patterns? |

### cli-first
| Concern | Question |
|---------|----------|
| State exposure | Does this tech require special patterns to expose state for AI verification? |
| TestID conventions | Does this tech have recommended testID patterns? |
| Verification commands | Does this tech have unique ways to verify state via CLI? |
| Token costs | Does this tech affect the cost of different verification methods? |

### ux-planner
| Concern | Question |
|---------|----------|
| Component constraints | Does this tech limit or enable specific UI patterns? |
| Async feedback | Does this tech affect how loading/error states should be shown? |
| Accessibility | Does this tech have accessibility implications or built-in a11y? |
| Form patterns | Does this tech change how forms should be designed? |

### e2e-guard
| Concern | Question |
|---------|----------|
| Coverage patterns | Does this tech require specific test coverage patterns? |
| Element selection | Does this tech affect how elements should be selected in tests? |
| API verification | Does this tech change how API responses should be verified? |
| State assertions | Does this tech require special state verification patterns? |

### e2e
| Concern | Question |
|---------|----------|
| Server startup | Does this tech affect how the dev server starts? |
| Artifact paths | Does this tech produce artifacts in non-standard locations? |
| Timing/waits | Does this tech require special wait patterns? |
| Cleanup | Does this tech require special cleanup between tests? |

### e2e-investigate
| Concern | Question |
|---------|----------|
| Failure patterns | Does this tech have common failure signatures? |
| Log formats | Does this tech produce logs in a specific format? |
| Reproduction | Does this tech require specific steps to reproduce issues? |
| Debug tools | Does this tech have built-in debugging tools to leverage? |

### ux-review
| Concern | Question |
|---------|----------|
| Visual patterns | Does this tech have visual conventions to verify? |
| Animations | Does this tech affect timing or transition verification? |
| Design system | Does this tech integrate with specific design systems? |
| Responsive | Does this tech affect responsive behavior verification? |

---

## Threshold

**Produce a reference doc if 2+ concerns are relevant.**

---

## Tech Context Detection Preamble

Each skill should include this check before executing:

```markdown
## Tech Context Detection

Before executing:
1. Scan task for technology mentions
2. For each tech detected:
   a. Check `techs/{tech}/README.md` exists — if not, run `/research {tech}` first
   b. Check `references/{tech}.md` exists — if not AND domain affects this skill:
      - Read `skills/TECH_CONTEXT.md` for concern matrix
      - Evaluate concerns (see matrix above)
      - If 2+ concerns relevant, produce reference doc
3. Read relevant reference docs and apply patterns
```

---

## Reference Doc Style

Follow existing patterns (see `testing-conventions.md`, `ux-patterns.md`, `cli-patterns.md`):

- **Practical and actionable** — code examples, shell commands, conventions
- **Tables for quick reference** — patterns, anti-patterns, checklists
- **No theory/overview** — just patterns the skill needs
- **Anti-patterns clearly marked** — GOOD vs BAD examples
- **Greppable section headers** — easy to search
