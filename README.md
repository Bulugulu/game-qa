# game-qa

**Turn your agent into a Game QA lead.**

## Why this exists

Agents are great at writing code. They're bad at verifying games — and verification is the bottleneck for autonomous agentic coding. If you can't trust what your agent says is "done," you can't actually let it ship.

This plugin makes Claude operate like a real QA lead for browser-based games. The key shift from typical code QA: we verify real player flows from the player's perspective, we verify visually, and we give the agent the tools to actually do the verification. Game verification is context-heavy, so we also split the work across subagents.

## Why browser games specifically

Browser-based games (three.js, canvas, WebGL, DOM) are uniquely well-suited to fully agent-authored development. There's no game engine UI to click through, no proprietary editor, no asset pipeline. Claude can write the whole game in code — runtime, rendering, gameplay, networking — and you can stay out of the loop.

That makes the missing piece *verification*. If the agent can write the game but can't prove a feature works the way a player would experience it, you can't actually let it ship. This plugin closes the loop: the agent codes the feature, the agent verifies the feature, and only then does it tell you "done."

## How it works — two pieces

game-qa has two parts that come together. They're separate concerns and worth understanding separately.

### 1. The skills

A skill family that teaches Claude how to verify games like a real QA lead. **These trigger automatically.** When the agent is working in a game repo and the plugin is installed, it recognizes that game QA applies — you don't have to prompt it.

- **A QA lead's mindset.** Enumerate player journeys, walk every journey to its edge cases, dispatch subagents, report in player language — not assert language. *Without this, agents claim "tests pass" once unit tests go green. That's the false-completion bug.*
- **Visual validation.** The agent sees the game the way the player would. When visual targets exist (mockups, design specs, prior screenshots), the agent compares against them. *A unit test can't see the screen. A function-checker can't notice a button looks pressable but isn't wired to anything.*
- **Subagent split.** Test authoring and per-domain execution dispatched to subagents in parallel. *One mega-agent doing end-to-end QA gets stuck or runs out of context, and you lose every observation it made.*
- **A persistent QA sheet.** Every run writes screenshots, per-case verdicts, and a human-readable summary to `qa-runs/`. *"Trust me, I tested it" doesn't scale. A real QA process leaves a paper trail.*
- **Only-when-green reporting.** The agent runs dispatch → fix → re-dispatch internally and surfaces only when every flow passes. *You should see "done" once, not 12 messages of dispatch noise.*

### 2. The agentic QA interface

A foundation the agent builds into *your game* so it can actually drive the verification. The skills above need real tools to do their job — a way to trigger states, capture screenshots, emit structured events. The plugin's bootstrap command tells Claude how to add this interface to your project.

You run it once per project:

```
/game-qa:bootstrap-game-qa-system
```

Claude adapts the interface to whatever stack you already have. No new dependencies. What you end up with:

- **Programmatic state triggers.** Set HP to 1, kill a player, force dawn, spawn a boss, end a match — one call to set up any moment under test. *Without this, the agent has to play through the game to reach the state. Slow, brittle, burns context on irrelevant gameplay.*
- **Visual capture.** A `debug.screenshot()` primitive that pauses animation and time-of-day for clean, deterministic captures. *A visual diff against an animated frame is noise. Pause first.*
- **Structured event log.** Every debug action emits a stable `[debug-event]` line. *The agent's browser-driving tool greps for these instead of parsing arbitrary logs.*
- **A cheat menu for you.** The same actions the agent uses, exposed behind a key combo (`~` or `Ctrl+Shift+D`). Click "Kill Self" and reproduce what the agent claimed. *The agent's verification is only useful if you can independently reproduce it.*

**Without piece 2, the skills are an instruction manual for tools that don't exist. Without piece 1, the interface is just a debug system. Together, the agent can verify features autonomously.**

## What a run looks like

The ideal flow is invisible. You're working on combat respawn with Claude. It finishes implementing the feature. Without you prompting, it recognizes that the QA skills apply (it's a game repo, the skills are installed) and runs the verification process itself.

You don't see:

- The test-case author returning 9 cases across 2 domains
- 2 execution subagents running in parallel
- Domain 2 failing — the respawn timer never appears on screen
- Claude reading the failure, fixing the HUD wiring, re-dispatching
- Verdicts aggregating, manifest and summary written to `qa-runs/`

You see one message:

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

You can also prompt explicitly — *"QA the combat respawn before I merge"* — but the whole point is you don't have to.

## Installation

```bash
/plugin marketplace add Bulugulu/game-qa
/plugin install game-qa@game-qa
```

Then, once per project:

```
/game-qa:bootstrap-game-qa-system
```

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
