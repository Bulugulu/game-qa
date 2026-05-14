---
name: bootstrap-game-qa-system
description: Use ONCE per browser-game project to set up the game-qa infrastructure. Copies the shipped runner template, the adapter skeleton, and the journey schema; gap-fills the project's existing debug system if one is present. Skip for non-game projects.
---

# Bootstrap Game QA System

You are setting up `game-qa` infrastructure for a browser-game project. The plugin ships three reference artifacts you'll copy and adapt:

- `references/runner.ts` — project-agnostic runner (copy as-is, do not modify)
- `references/qa-adapter.template.ts` — adapter skeleton (copy and fill in)
- `references/journey-schema.md` — YAML contract the runner consumes (don't copy; engineers read this to author journeys)

**The seam:** the runner is **framework-fixed**, the adapter is **project-specific**. If you need new behavior, add a primitive to the adapter or extend the journey schema — never fork the runner.

**Run once per project.** If parts exist, gap-fill — don't rebuild.

## The Iron Law

```
GAME PROJECTS ONLY.
RUN ONCE. GAP-FILL, DON'T REPLACE.
ARRANGE/PROBE PRIMITIVES ONLY — NEVER ACTIONS-UNDER-TEST.
```

## What to Build (in order)

1. **Debug system.** An action registry (server-authoritative for multiplayer projects; client-side for single-player), reflection-driven catalog (`debug.help()`), `window.debug.<action>` proxy on the client, `[debug-ready]` + `[debug-event]` console contract. If the project has these, verify and move on. If not, build the minimum: a `window.debug` object exposing project actions, plus a one-shot `console.log("[debug-ready]")` once it's wired.

2. **Drop the runner.** Copy `references/runner.ts` to `<project>/qa/runner.ts`. Start unmodified. The runner is extensible — add new `perform:` types or `capture: at:` kinds as your game needs them (see "Extending the framework" below) — but don't silently change existing semantics. If your project is plain JS, run via `tsx` / `ts-node` / `vite-node`, or strip type annotations after copy. The runner imports `./adapter.ts` — the next step creates it.

3. **Adapt the adapter.** Copy `references/qa-adapter.template.ts` to `<project>/qa/adapter.ts`. Then fill in:
   - `url` — your project's dev URL
   - `readyMarker` — the console string your project emits when `window.debug` is wired
   - `hudSelector` — optional CSS selector for the HUD if you want `capture: at: [hud]` to work
   - `arrange.*` — wrappers around your `window.debug.*` for state setup (see arrange-vs-shortcut discipline below)
   - `probe.*` — wrappers for read-only state queries; always include `snapshot` as the default fallback
   - `events.subscribe` / `events.drain` — wire to your project's event source (DOM events, EventEmitter, WebSocket messages, etc.)
   - `reset` — idempotent between-journey reset
   - `clearByTag` — sweep entities this runner created (use the `spawnTracker` pattern in the template)

4. **Sentinel primitives.** Land a handful of starter `arrange.*` and `probe.*` per major system — enough to prove the wiring works. The full catalog grows organically as features land; engineers add what they need via capability-gap tickets, you don't pre-build it.

5. **Arrange-vs-shortcut taxonomy doc** at `<project>/docs/qa/arrange-vs-shortcut.md`. The discipline: `arrange.*` sets state; it never bypasses the chain the *current case* is verifying. The same primitive may be allowed in one case and forbidden in another — it's the case that defines what's a shortcut. Setup-flavored verbs (`set`, `spawn`, `give`, `grant`, `clear`) are usually safe; ban anything that bypasses the action under test for this case.

6. **Smoke-test the rig.** Author a 3-step `qa-runs/smoke/journeys/smoke/journey.yaml` (capture initial → waitMs 500 → capture after). Run the runner against it. Confirm the evidence bundle appears (`output.json`, `screenshots/`). If this fails, the rest of the framework can't work — fix it before declaring done.

7. **`qa-runs/` dir** at the project root with a stub README pointing at `requesting-game-qa`.

## Extending the framework

The plugin ships a skeleton. **Extend it for your game.** The cross-project contract is the *shape*:

- The four-namespace adapter (`arrange` / `probe` / `events` / `reset`)
- The step-vocabulary categories (`arrange`, `perform`, `capture`, `waitForEvent`, `waitForState`, `waitMs`, `expect`, `loop`, `js`)
- The evidence-bundle layout (`output.json` + screenshots + dom + console + tickets)

What's *not* fixed — extend in your project copy as your game needs:

- New `perform:` action types in `runner.ts` (pointer-lock-aware mouse-delta, scroll-wheel, key-hold, gamepad, touch — whatever your game uses)
- New `capture: at:` kinds (regional screenshots, audio events, frame strips, performance traces)
- Project-specific `arrange.*` and `probe.*` primitives — always, that's the adapter's whole point
- Project-specific waits, predicates, time conventions (tick-based stepping, sim-clock advance)

If an extension feels broadly useful, propose it upstream. Otherwise live with the local extension — that's the framework working as designed.

## When You're Done

- All seven items above shipped (or gap-filled)
- Smoke journey runs end-to-end and emits an evidence bundle
- Sentinel `arrange.*` and `probe.*` callable from both browser console and runner
- Tell the user: *"Game-QA bootstrapped. The runner at `qa/runner.ts` is framework code — don't modify it; extend `qa/adapter.ts` instead. Engineers can now request QA via `requesting-game-qa`. Add new `arrange.*` / `probe.*` primitives as features land. Never add an action that shortcuts an action-under-test — see `docs/qa/arrange-vs-shortcut.md`."*

Then exit.

## Hard Rules

- **GAME projects only.** Refuse for tools, build infra, generic web apps.
- **Don't break existing debug actions.** Gap-fill, don't rebuild.
- **Never add primitives that bypass the action under test.** A primitive that performs what a case is supposed to verify is not a test — it's a shortcut. Only `arrange.*` (preconditions) and `probe.*` (observation). (See taxonomy doc.)
- **Extend, don't fork.** Adding new `perform:` types or `capture: at:` kinds for your game is encouraged. Silently changing existing semantics (e.g., making `arrange:` do real player input) is forking — don't.
- **Don't reconstruct the runner from scratch.** The plugin ships one. Copy `references/runner.ts` and extend in place.
- **Land starter content, not the full catalog.** 2-3 sentinel primitives per system. Features add the rest.
- **Don't run continuously.** Setup, not watcher.

## Red Flags — STOP

- Replacing the project's existing transport → gap-fill instead
- Adding an arrangement whose name verbs the current case's action under test → arrange-vs-shortcut violation
- Forking the runner for project-specific reasons → use the adapter
- Continuing past 2-3 sentinel primitives per system → over-anticipates

## Related Skills

- `requesting-game-qa` — engineer's entry once bootstrap is done
- `game-qa` — Lead orchestrator
- `references/adapter-contract.md` — full adapter shape
- `references/arrange-vs-shortcut.md` — the discipline
- `references/journey-schema.md` — what the runner accepts

## The Bottom Line

Debug + adapter + runner + taxonomy + dir. Sentinel primitives only. Done.
