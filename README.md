# game-qa

A Claude Code plugin for verifying browser-based game features at three layers — **unit**, **programmatic in-game**, **visual in-game** — framed as **player journeys, not assert-equals**.

Built on the discipline that **a feature that passes mechanically but feels broken to a player is a fail.**

## What's in the bundle

| Artifact | What it does |
|---|---|
| `/game-qa:bootstrap-game-qa-system` | One-shot setup. Run once per project. Builds the debug-system foundation (server-authoritative action registry, `window.debug` proxy, `getState`/`screenshot` primitives, structured `[debug-event]` log, user-facing cheat menu). |
| `verifying-browser-games` *(skill)* | Orchestrator strategy. The agent reads this when verifying a feature. Three layers, parallel-by-domain dispatch, player-flow reporting, only-surface-when-green discipline. |
| `enumerating-game-test-cases` *(skill)* | Test-case authoring subagent. Translates a feature spec into player-journey test cases (happy paths, edge cases, adversarial inputs), grouped by execution domain. |
| `running-game-qa-pass` *(skill)* | Execution subagent. Drives the browser as a player would for one domain, returns a verdict, persists screenshots and `verdict.json` to the artifact directory. |

## Lifecycle for a feature

```
[setup, once per project]
  /game-qa:bootstrap-game-qa-system
  (the agent gap-fills your debug system to satisfy the plugin's contract)

[per feature]
  verifying-browser-games dispatches:
    enumerating-game-test-cases       → test-cases.json
    running-game-qa-pass × N domains  → verdict.json + screenshots (parallel)
  → orchestrator aggregates verdicts  → manifest.json + summary.md
  → on failure: orchestrator fixes and re-dispatches (cap: 2 cycles)
  → on green: surfaces to the user with player-flow language + cheat-menu instructions

The user only hears about the feature when it's truly green.
```

Persistent artifacts land in `qa-runs/<timestamp>_<feature>/` so you can review the QA sheet and reproduce flows manually via the cheat menu.

## Install

```
/plugin marketplace add Bulugulu/game-qa
/plugin install game-qa@game-qa
```

After install, the bootstrap command and three skills become available. The skill descriptions trigger automatically when you ask Claude to verify, QA, or test a game feature.

## Why this exists

Game QA isn't code QA. A passing unit test for `respawnDuration(t)` doesn't prove the player respawns correctly. A working `debug.killBoss()` doesn't prove the kill button is wired. A nice screenshot doesn't prove the underlying state is correct. None of those tell you whether the player would *feel* the feature works.

This plugin enforces all three layers, plus the player-perspective rule: every test case is a player journey, every observation is a player observation, every report is in player-flow language.

## Requirements

- The bootstrap command (artifact A) assumes a browser-based game with a server (any transport — Colyseus, WS, REST). It adapts to whatever's there.
- A browser-driving tool the QA subagents can use. Today: Playwright MCP. The skill is contract-based, not tool-locked — Chrome DevTools MCP, a successor MCP, or any future tool meeting the contract works (open URL, scrape console, real clicks/typing, screenshots).

## Status

`v0.1.0` — pre-validation. Designed against a real game project (a three.js multiplayer survival-MOBA jam entry) and the debug-system contract is implemented and shipping there. The skills haven't survived wide-cycle real-world use yet. Expect rough edges; PRs welcome.

## Manual install

If you'd rather copy directly without the plugin system:

```bash
git clone https://github.com/Bulugulu/game-qa.git
mkdir -p ~/.claude/skills ~/.claude/commands
cp -r game-qa/plugins/game-qa/skills/* ~/.claude/skills/
cp game-qa/plugins/game-qa/commands/bootstrap-game-qa-system.md ~/.claude/commands/
```

## License

MIT — see [LICENSE](./LICENSE).
