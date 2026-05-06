# game-qa

**Turn your agent into a Game QA lead.**

## Why this exists

Agents are great at writing code. They're bad at verifying games — and verification is the bottleneck for autonomous agentic coding. If you can't trust what your agent says is "done," you can't actually let it ship.

This plugin makes Claude operate like a real QA lead for browser-based games. The key shift from typical code QA: we verify real player flows from the player's perspective, we verify visually, and we give the agent the tools to actually do the verification. Game verification is context-heavy, so we also split the work across subagents.

## Why browser games specifically

Browser-based games (three.js, canvas, WebGL, DOM) are uniquely well-suited to fully agent-authored development. There's no game engine UI to click through, no proprietary editor, no asset pipeline. Claude can write the whole game in code — runtime, rendering, gameplay, networking — and you can stay out of the loop.

That makes the missing piece *verification*. If the agent can write the game but can't prove a feature works the way a player would experience it, you can't actually let it ship. This plugin closes the loop: the agent codes the feature, the agent verifies the feature, and only then does it tell you "done."

## What you get

### A QA lead's mindset

A skill that tells the agent how a real QA process works: enumerate the player journeys for a feature, walk every journey to its edge cases, dispatch subagents to drive the game, report back in **player language — not assert language**. Surface a feature as "done" only when every flow is truly green.

*Why it matters:* without this, agents claim "tests pass" once unit tests go green. That's the false-completion bug, and it's the #1 reason you can't trust an agent to ship game features autonomously.

### An agentic QA interface

A foundation the agent uses to programmatically trigger any game state — set HP to 1, kill a player, force dawn, spawn a boss, end a match — so it can jump straight to the moment it wants to test.

*Why it matters:* without it, the agent has to play through the game to reach the state under test. Slow, brittle, and burns context on irrelevant gameplay. With it, every test is one call to set up the moment, then verify.

### Visual validation

The skills require the agent to use visual validation tools — see the game the way the player would — to confirm elements are rendered, positioned correctly, and interactable. When visual targets exist (mockups, design specs, prior screenshots), the agent compares against them.

*Why it matters:* a unit test can't see the screen. A function-checker can't notice that a button looks pressable but isn't wired to anything. Visual validation is how you catch the gap between "the code is correct" and "the player can actually use the feature."

### Subagent split

Test authoring and per-domain execution are dispatched to subagents in parallel, keeping the orchestrator focused on strategy and reporting.

*Why it matters:* one big agent doing end-to-end QA gets stuck or runs out of context, and you lose every observation it made. Smaller, parallel, replaceable.

### A persistent QA sheet

Every run lands in `qa-runs/<timestamp>_<feature>/` — screenshots, per-case verdicts, a human-readable summary.

*Why it matters:* "trust me, I tested it" doesn't scale. A real QA process leaves a paper trail you can review and reproduce.

### A cheat menu for you

The same actions the agent uses are exposed to you behind a key combo. If the agent says "I verified the respawn flow," you hit `~`, click "Kill Self," and see for yourself.

*Why it matters:* the agent's verification is only useful if you can independently reproduce it. One click.

## What a run looks like

You ask:

> "QA the combat respawn before I merge"

The orchestrator runs internally — you don't see any of this:

1. Dispatches the test-case author → returns 9 cases across 2 domains (respawn-flow, hud-feedback)
2. Dispatches 2 execution subagents in parallel — one per domain
3. Domain 1 passes. Domain 2 fails — the respawn timer never appears on screen
4. Reads the failure observation, fixes the HUD wiring, re-dispatches domain 2
5. Domain 2 passes. Aggregates verdicts. Writes the QA sheet to `qa-runs/`

You see (only at the end):

> Combat respawn verified across 4 player flows + 7 edge cases. All green.
>
> **Player flows validated:**
> - Player dies at match start, respawns within 30s, can move and attack
> - Player killed by a turret takes the same flow as PvP death
> - HUD respawn countdown visible from death to revive
> - Player who dies near enemy spawn respawns at base
>
> QA sheet: `qa-runs/2026-05-05T22-30_combat-respawn/summary.md`
>
> To validate manually: hit `~`, click "Kill Self", watch the flow.

If you want to drill in, ask *"show me the screenshots for hud-feedback"* and the orchestrator pulls them from the QA sheet.

## Installation

```bash
/plugin marketplace add Bulugulu/game-qa
/plugin install game-qa@game-qa
```

Then, once per project:

```
/game-qa:bootstrap-game-qa-system
```

Claude adds the agentic QA interface to your project, adapting to whatever stack you already have. No new dependencies.

## Browse the skills

The plugin is plain Markdown — read each skill directly:

- [`verifying-browser-games`](./plugins/game-qa/skills/verifying-browser-games/SKILL.md) — the QA lead's playbook (orchestrator)
- [`enumerating-game-test-cases`](./plugins/game-qa/skills/enumerating-game-test-cases/SKILL.md) — how a subagent translates a feature into player-journey test cases
- [`running-game-qa-pass`](./plugins/game-qa/skills/running-game-qa-pass/SKILL.md) — how a subagent drives the browser to verify one domain
- [`bootstrap-game-qa-system`](./plugins/game-qa/commands/bootstrap-game-qa-system.md) — the one-shot prompt that sets up the agentic QA interface

## Status

**v0.1.0 — pre-validation.** Designed against a real three.js multiplayer game. The skills haven't survived wide-cycle real-world use yet. Expect rough edges; PRs welcome.

## License

MIT — see [LICENSE](./LICENSE).
