# XState — Research & Development Journal

> Actor-based state management & orchestration for complex app logic using state machines, statecharts, and the actor model.

## Status

| Phase | Status |
|-------|--------|
| Research | Done |
| Reference docs | Not started |

---

## What Is XState?

**XState** is a state management and orchestration library for JavaScript/TypeScript that uses event-driven programming, state machines, statecharts, and the actor model to handle complex logic in predictable, robust, and visual ways.

- **GitHub:** [statelyai/xstate](https://github.com/statelyai/xstate) — 29.2k stars, MIT license
- **Docs:** [stately.ai/docs/xstate](https://stately.ai/docs/xstate)
- **npm:** ~1.5M weekly downloads
- **Current version:** v5.25.0 (requires TypeScript 5.0+)
- **Zero dependencies** in the core library
- **Visual editor:** [Stately Studio](https://state.new) for creating and editing machines visually

### How It Differs from Redux/Zustand/Context

| Aspect | Redux / Zustand | XState |
|--------|-----------------|--------|
| Model | Global store + reducers/actions | State machines + actors |
| Invalid states | Possible (multiple booleans can conflict) | Impossible by design (one state at a time) |
| Side effects | Middleware (thunks, sagas) | Built-in actions, invoked actors |
| Visualization | DevTools only | Visual editor + inspector |
| Learning curve | Lower for simple cases | Higher, but enforces correctness |
| Concurrency | Manual | Built-in parallel states + actor spawning |
| When to reach for it | Simple shared state, UI state | Complex workflows, multi-step forms, async orchestration |

**Key insight:** XState is not a replacement for Zustand/Redux — it solves a different problem. Use XState when your state has *rules* (valid transitions, guards, effects). Use Zustand when your state is *data* (counters, lists, UI toggles).

---

## Core Concepts

1. **State Machines** — Define finite states and the events that transition between them. A machine can only be in one state at a time.

```typescript
import { createMachine } from 'xstate';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'Inactive',
  states: {
    Inactive: { on: { toggle: 'Active' } },
    Active: { on: { toggle: 'Inactive' } },
  },
});
```

2. **Actors** — Independent units of computation that can receive events, maintain internal state, and spawn child actors. In v5, actors are the central abstraction.

```typescript
import { createActor } from 'xstate';

const actor = createActor(toggleMachine);
actor.subscribe((snapshot) => console.log(snapshot.value));
actor.start();
actor.send({ type: 'toggle' }); // logs 'Active'
```

3. **Context** — Extended state (data) stored alongside the finite state. Updated via the `assign` action.

```typescript
const countMachine = createMachine({
  context: { count: 0 },
  on: {
    INC: { actions: assign({ count: ({ context }) => context.count + 1 }) },
  },
});
```

4. **Guards** — Conditional logic that determines whether a transition should occur.

```typescript
states: {
  active: {
    on: {
      submit: {
        guard: ({ context }) => context.isValid,
        target: 'submitted',
      },
    },
  },
}
```

5. **Actions** — Side effects that execute during transitions (logging, API calls, context updates).

6. **Invoked Actors** — Child actors spawned by a machine to handle async work (promises, callbacks, other machines).

7. **Statecharts** — Extended state machines with hierarchical (nested) states, parallel states, and history states for modeling complex behavior.

---

## The XState Ecosystem

| Package | Purpose | Install |
|---------|---------|---------|
| `xstate` | Core state machine library + actor model | `npm i xstate` |
| `@xstate/react` | React hooks (`useMachine`, `useActor`) | `npm i @xstate/react` |
| `@xstate/store` | Lightweight event-driven store (Zustand alternative) | `npm i @xstate/store` |
| `@xstate/vue` | Vue composition functions | `npm i @xstate/vue` |
| `@xstate/svelte` | Svelte utilities | `npm i @xstate/svelte` |
| `@xstate/solid` | Solid hooks | `npm i @xstate/solid` |
| `@statelyai/inspect` | Visual state inspector | `npm i @statelyai/inspect` |
| [Stately Studio](https://state.new) | Visual editor for machines | Web app (free tier) |

### @xstate/store — The Lightweight Option

For simpler state needs, `@xstate/store` provides event-driven state management comparable to Zustand or Pinia, but with XState's event vocabulary. Key features:

- Atoms (lightweight reactive values)
- Selectors with custom equality
- Built-in undo/redo
- Immer integration for immutable updates
- Can be converted to full XState machines via `fromStore()` when complexity grows

This provides a migration path: start with `@xstate/store`, graduate to full `xstate` when you need state machines.

---

## Ideal Use Cases

| Use Case | Why XState Fits |
|----------|-----------------|
| Multi-step forms / wizards | States enforce valid step sequences, guards prevent skipping |
| Authentication flows | Login → MFA → success/failure modeled as explicit states |
| Data fetching orchestration | Loading/error/success states with retry logic |
| Complex UI interactions | Drag-and-drop, modals, multi-panel coordination |
| Workflow engines | Approval chains, document lifecycle |
| Game logic | Turn-based state, animation sequencing |
| Real-time features | WebSocket connection lifecycle + reconnection |

### When NOT to Use XState

- **Simple UI toggles** — `useState` or Zustand is sufficient for booleans and counters
- **Global shared data** — If you just need a shared store without transition rules, use Zustand or Context
- **Prototyping** — The upfront modeling adds friction when requirements are still unclear
- **Small teams unfamiliar with state machines** — Learning curve can slow down delivery if the team hasn't bought in
- **Server-only CRUD** — If state lives in the database and the frontend just renders it, a state machine adds unnecessary complexity

---

## Testing with XState

XState machines are highly testable because they are **deterministic** — the same sequence of events always produces the same result.

### Unit Testing (Arrange-Act-Assert)

```typescript
import { createActor } from 'xstate';
import { describe, it, expect } from 'vitest';

describe('toggle machine', () => {
  it('transitions to Active on toggle', () => {
    const actor = createActor(toggleMachine);
    actor.start();
    actor.send({ type: 'toggle' });
    expect(actor.getSnapshot().value).toBe('Active');
  });
});
```

### Mocking Side Effects

```typescript
const mockLogger = vi.fn();
const machine = setup({
  actions: { logMessage: mockLogger },
}).createMachine({ /* ... */ });

// Assert mockLogger was called during transition
```

### Model-Based Testing

XState provides `xstate/graph` utilities to automatically generate test paths from a machine, ensuring all reachable states and transitions are covered. This complements the playbook's E2E-first philosophy — use model-based testing to generate the scenarios, then validate them with Playwright.

### Fit with E2E-First Philosophy

- State machines define *what* the user can do — E2E tests validate *that they can do it*
- Machine definitions serve as living documentation of valid user flows
- Model-based test generation ensures no reachable state is untested
- Guards and transitions map directly to test assertions

---

## Fit Assessment

| Criterion | Assessment |
|-----------|------------|
| Compatible project types | nextjs-with-cli, react-with-cli |
| Runtime requirement | Node.js 18+, TypeScript 5.0+ |
| Testing integration | Model-based test generation complements Playwright E2E; deterministic machines simplify unit tests |
| Complexity appropriate? | Yes for projects with complex state logic; overkill for simple CRUD |

---

## Decisions

| Decision | Rationale |
|----------|-----------|
| Include `@xstate/store` alongside `xstate` | Provides a lightweight entry point; teams can start simple and graduate to full machines |
| Include `@xstate/react` | Primary framework target for the playbook |
| Skip `@xstate/test` (deprecated) | Testing utilities moved into `xstate/graph` in v5 |
| Target React projects only | Vue/Svelte/Solid are not playbook project types |

## Open Questions

- Which base skills benefit from xstate-specific reference docs?
- Should reference docs include Stately Studio integration guidance or keep it pure code?
- What's the ideal directory structure for machines in a project? (`src/machines/` vs co-located with features?)
- How should `@xstate/store` and full `xstate` coexist in the same project?
- Machine composition: parent machines invoking child machines
- React integration patterns: `useMachine` vs `useActor` vs `useSelector`
- `@xstate/store` for simple state vs full machine for complex flows — where's the crossover?
- TypeScript inference quality: how well does v5 infer event and context types?
- Testing workflow: unit test machines in isolation, then E2E test the UI that uses them
- DevTools / inspector integration for debugging during development

---

## Sources

- [XState Documentation](https://stately.ai/docs/xstate)
- [XState GitHub](https://github.com/statelyai/xstate)
- [XState Quick Start](https://stately.ai/docs/quick-start)
- [@xstate/store Documentation](https://stately.ai/docs/xstate-store)
- [@xstate/react Documentation](https://stately.ai/docs/xstate-react)
- [XState Testing Guide](https://stately.ai/docs/testing)
- [XState v5 Announcement](https://stately.ai/blog/2023-12-01-xstate-v5)
- [React State Management in 2025 — Makers' Den](https://makersden.io/blog/react-state-management-in-2025)
