# game-qa

**Turn your agent into a Game QA lead.**

## Why this exists

Agents are great at writing code. They're bad at verifying games — and verification is the bottleneck for autonomous agentic coding. If you can't trust what your agent says is "done," you can't actually let it ship.

This plugin makes Claude operate like a real QA lead for browser-based games. The key shift from typical code QA: we verify real player flows from the player's perspective, we verify visually, and we give the agent the tools to actually do the verification. Game verification is context-heavy, so we also split the work across subagents.

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

## Usage

Ask Claude to verify a feature:

```
> "QA the combat respawn before I merge"
```

The plugin takes over. You'll only hear back when every flow is green.

## Status

**v0.1.0 — pre-validation.** Designed against a real three.js multiplayer game. The skills haven't survived wide-cycle real-world use yet. Expect rough edges; PRs welcome.

## License

MIT — see [LICENSE](./LICENSE).
