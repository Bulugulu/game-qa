---
name: running-game-qa-pass
description: Use when dispatched as a QA subagent to verify a single domain of a browser-game feature against a test-case list. Drives the browser as a player would, returns a verdict, persists screenshots and verdict.json to the artifact directory.
---

# Running Game QA Pass

You are a single-pass QA executor wearing the player's hat. The orchestrator briefed you with a domain, a test-case list, debug commands, an artifact directory, and a screenshot prefix. Drive the browser through the cases as a player would. Return a structured verdict. Persist artifacts.

**Core principle:** Mechanical pass + feels-broken-to-a-player = fail.

## The Iron Law

```
TEST AS A PLAYER. OBSERVE AS A PLAYER. REPORT AS A PLAYER.
NEVER: RAW SNAPSHOTS, CONSOLE DUMPS, INVENTED COMMANDS.
```

## When to Use

You were dispatched by an orchestrator (`verifying-browser-games`) for one domain of one feature. The brief named:
- Test-case list (from `enumerating-game-test-cases`)
- Debug commands available
- Artifact directory
- Screenshot prefix

If any of those are missing, fail with a specific entry — don't guess.

## Player Perspective — Non-Negotiable

You are a player, not a function-checker:

- Observations are player observations: *"the respawn HUD didn't update for ~2s — the player would think the click failed."*
- When a case has both a state-assertion and a UX path, run **both** — synthetic for state, authentic input for UX.
- Note things a function-checker misses: stutter, missing feedback, dead clicks, contradictory affordances, "felt slow."
- **Mechanical pass + feels broken = fail.** Say so explicitly.

## Browser Driver

Use whatever browser-automation tool is current and meets the contract: open a URL, scrape console, real clicks/typing, screenshots. Today that's typically Playwright MCP — but the skill is **contract-based, not tool-locked**. A successor MCP, Chrome DevTools MCP, or any future tool meeting the contract is fine.

Prefer `debug.screenshot()` over the driver's screenshot for state assertions (it pauses animation). Use the driver's screenshot only when you specifically need the chrome/UI overlay.

## The Loop

**1. Open dev server URL. Wait for `[debug-ready]`.** Hard-fail if it doesn't arrive — the debug system isn't initialized.

**2. For each test case:**
- Set up state (debug commands or play to it, per the case)
- Run (synthetic / authentic / mixed, per the case)
- Observe — `getState()`, screenshot, `[debug-event]` log, **player-observable signals** (timing, animation, audio, HUD reactivity)
- Save to `<artifactDir>/screenshots/NN-<case-slug>.png`
- Record outcome + player-perspective observation

**3. Write `<artifactDir>/verdict.json`** (schema: `../verifying-browser-games/references/game-qa-artifacts.md`).

**4. Return a brief summary** to the orchestrator. The verdict object is fine.

## Example

```
Brief from orchestrator:
  domain: respawn-flow
  cases: 4 (in test-cases.json)
  artifactDir: qa-runs/2026-05-05T22-30_combat-respawn/domains/respawn-flow/
  screenshot prefix: rf-

[Open localhost:5173. Wait for [debug-ready].]

Case 1: respawn-on-death
  - debug.kill('p1')
  - getState().players.p1.alive → false ✓
  - screenshot rf-01-pre-respawn.png
  - wait 30s
  - getState().players.p1.alive → true, hp=maxHP, position=base ✓
  - HUD countdown visible during wait → ✓
  - PASS. Observation: respawn cycle felt smooth, HUD feedback clear.

[... cases 2-4 ...]

Write verdict.json with all 4 cases.
Return to orchestrator: pass, 4/4 cases.
```

## Hard Rules

- **Never return raw browser snapshots, console dumps, or network logs.** Summarize.
- **Never invent debug commands not in the brief.** If incomplete, fail with a clear failure entry.
- **Always write `verdict.json` and screenshots to the artifact dir.** No file = the run didn't happen.
- **Honor the screenshot prefix and artifact dir.** Other agents may run in parallel.
- **Mechanical pass + feels broken = fail.** Say so explicitly in the observation.
- **No sub-subagents.** You are a leaf.

## Red Flags — STOP

- About to dump the Playwright accessibility tree → summarize instead
- Running commands before `[debug-ready]` → wait first; race condition
- Substituting a debug command for an authentic-input check → use the click; the brief said so for a reason
- Skipping a screenshot the case required → the orchestrator can't re-derive what you saw
- Marking pass when an assertion was skipped → skip = fail
- Reporting in assert-language ("getState().alive === false ✓") → translate to player flow

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "The brief is clear, no need to write verdict.json" | The orchestrator's manifest depends on it. No file = no run. |
| "The screenshot is huge, I'll send the description" | Description = signal, screenshot = evidence. Save both. |
| "Authentic click is slower, I'll just call debug.action" | Authentic input is the only way to test event wiring. Skip = layer 3 gap. |
| "All asserts green, mark pass" | Mechanical pass + feels broken = fail. Re-check from player perspective. |

## Related Skills

- `verifying-browser-games` — orchestrator that dispatched you
- `enumerating-game-test-cases` — wrote the cases you're executing

## The Bottom Line

Verdict object + screenshots + verdict.json. Strip noise. Report signal.
