---
name: verifying-browser-games
description: Use when verifying any feature in a browser-based interactive runtime (three.js, canvas, WebGL, DOM game) where behavior depends on game state, render output, or user input — and unit tests alone won't prove it works in the browser.
---

# Verifying Browser Games

Game QA is not code QA. A passing unit test for `respawnDuration(t)` doesn't prove the player respawns correctly. A working `debug.killBoss()` doesn't prove the kill button is wired. None of those tell you whether the player would feel the feature works.

**Core principle:** A case that passes mechanically but feels broken to a player is a fail.

**REQUIRED BACKGROUND:** This skill specializes `superpowers:verification-before-completion` for browser games. Same "evidence before claims" discipline; this layer adds game-specific scaffolding.

## When to Use

**Mandatory:**
- Implementing or fixing any in-game system with both code logic and a rendered/interactive surface
- Before claiming a game feature is "done" or merging
- When a bug reproduces in-browser but tests are green (or vice versa)

**Don't use for:**
- Pure tooling, build scripts, doc edits
- Systems with no rendered surface

## The Iron Law

```
NO "DONE" CLAIM WITHOUT GREEN AT ALL THREE LAYERS,
FRAMED AS PLAYER JOURNEYS, WITH PERSISTED ARTIFACTS
```

## Three Layers, Three Scaffolds

| Layer | Scaffold | What it gives you |
|---|---|---|
| Unit | Standalone `qa-<system>.ts` scripts | Pure-logic feedback; no game required |
| Programmatic in-game | Debug registry + `window.debug` proxy *(set up by the `bootstrap-game-qa-system` command)* | Trigger any state in one RPC, on the same code path as real events |
| Visual in-game | Browser-driving tool (e.g. Playwright MCP today) + `[debug-event]` log + `debug.screenshot()` | Visual regression + scriptable user simulation |

If the debug system isn't built (no `[debug-ready]` marker, no `window.debug`), **offer to run `bootstrap-game-qa-system` first**; otherwise proceed with layers 1+3 and flag the layer-2 gap when reporting.

## How to Verify a Feature

**1. Plan.** Dispatch `enumerating-game-test-cases` to author `test-cases.json`. Skip if trivial (one obvious flow, no edges).

**2. Execute.** Dispatch `running-game-qa-pass` × N — one per execution domain — in parallel, single message. Each gets:
- Domain it owns
- Test-case subset (from D's output)
- Debug commands available
- Artifact dir: `qa-runs/<run>/domains/<domain>/`
- Unique screenshot prefix

Sequential if domains share global game state.

**3. Aggregate.** Read each `verdict.json`; write `manifest.json` + `summary.md`. Schemas: `references/game-qa-artifacts.md`.

**4. Fix or report.** Failures → fix and re-dispatch the failing domains. Cap: 2 fix cycles, then escalate. Otherwise report (see "Reporting" below).

## Synthetic vs. Authentic — Both, Not One

| Question | Technique |
|---|---|
| "Does the system behave correctly given state X?" | `debug.<action>()` → `getState()` → `debug.screenshot()` |
| "Is the user-input → state pipeline wired correctly?" | Real `page.click()` / `page.keyboard.press()` |

Synthetic = logic. Authentic = UX. **Both, not one.**

## Reporting to the User

**Don't surface incremental progress.** Run dispatch → aggregate → fix → re-dispatch internally. Surface only when **all green** (or stuck after 2 cycles).

When you do surface:

1. **Headline.** *"Combat respawn verified across 4 player flows + 7 edge cases. All green."*
2. **Player flows validated** — bullets in human language, not asserts.
3. **Link to QA sheet** — `qa-runs/<run>/summary.md` and `manifest.json`.
4. **Manual-validation instructions**, ranked: cheat menu → debug commands → manual play. Tell the user *exactly* how to reach each flow.
5. **Offer to drill in.**

Withhold per-domain raw verdicts, console dumps, and fixed-along-the-way failures unless asked.

## Example

```
[Just implemented combat respawn. Ready for QA.]

[Dispatch enumerating-game-test-cases]
  feature: combat-respawn
  artifactDir: qa-runs/2026-05-05T22-30_combat-respawn/

[D returns]:
  9 cases across 2 domains (respawn-flow, hud-feedback)

[Dispatch running-game-qa-pass × 2 in parallel]
  agent-1: respawn-flow, prefix "rf-"
  agent-2: hud-feedback, prefix "hud-"

[Returns]:
  agent-1: pass
  agent-2: fail (HUD respawn timer never appears)

[Fix HUD wiring. Re-dispatch agent-2.]
  agent-2: pass

[Aggregate. Write manifest.json + summary.md.]

To user:
  Combat respawn verified across 9 cases. All green.
  Player flows validated:
    - Player dies at match start, respawns within 30s
    - Killed by turret takes same flow as PvP death
    - HUD respawn countdown visible from death to revive
  QA sheet: qa-runs/2026-05-05T22-30_combat-respawn/summary.md
  To validate manually: open cheat menu (Ctrl+Shift+D), click "Kill Self".
```

## Red Flags — STOP

- "Tests pass, shipping it" → did you verify in the running game?
- "Looks right in the screenshot" → what does `getState()` say? What did the player observe?
- "I'll just run Playwright in the main thread quickly" → no, dispatch a subagent.
- "I'll surface progress so the user knows I'm working" → no. Run the loop. Surface when done.
- **Mechanical pass + screenshot fine but feels broken** → player perspective overrides. Mark fail.

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "Unit tests pass, that's enough" | Unit tests don't render, don't click, don't show the player anything |
| "I can run Playwright myself, faster" | Snapshots/console flood your context. Dispatch a subagent. |
| "D is overkill for this feature" | If there are >3 cases or any non-obvious edges, you're past D-skip territory |
| "The user wants to see progress" | The user wants done. Surfacing intermediate failures = noise. |
| "Cheat-menu instructions are obvious" | They aren't. Tell the user the exact key combo and exact action. |

## Related Skills

- `superpowers:verification-before-completion` — parent discipline
- `enumerating-game-test-cases` — dispatched once per feature
- `running-game-qa-pass` — dispatched per domain
- `bootstrap-game-qa-system` — slash command that builds the debug-system foundation

## The Bottom Line

The user only hears about a feature when it's truly green. Until then, you run the loop yourself.
