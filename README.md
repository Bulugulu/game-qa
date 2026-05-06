# game-qa

A Claude Code plugin that catches the bugs your unit tests can't see — the "felt slow", "click did nothing", "HUD never updated" kind that only show up when a real player drives the game.

## What This Does

**game-qa** turns Claude into a real QA process for your browser-based game. When you ask Claude to verify a feature, it doesn't just check the code — it spins up the running game, drives it as a player would, captures screenshots, and only tells you "done" when every flow actually feels right.

Built on a single rule: **a case that passes mechanically but feels broken to a player is a fail.**

### Key Features

- **Three-layer verification** — Unit tests, in-game state, and visual/UX. Not one. All three.
- **Parallel QA agents** — Multiple subagents verify different domains of a feature at once; the orchestrator only surfaces when everything is green.
- **Player-perspective reporting** — Results come back in player-flow language ("player respawns within 30s and can move"), not `expect(x).toBe(y)`.
- **Persistent QA sheet** — Every run writes screenshots and verdicts to `qa-runs/` so you can review and reproduce flows manually.
- **Cheat menu included** — The same debug actions the agent uses are exposed to you behind `~` or `Ctrl+Shift+D`, so you can validate any flow manually in one click.
- **Tool-agnostic browser driver** — Uses Playwright MCP today, but the contract is open — Chrome DevTools MCP or any future MCP works the same way.

## Installation

```bash
/plugin marketplace add Bulugulu/game-qa
/plugin install game-qa@game-qa
```

Then, once per project, set up the debug-system foundation:

```
/game-qa:bootstrap-game-qa-system
```

Claude reads the prompt and gap-fills your project's debug system to satisfy the framework's contract — action registry, `window.debug` proxy, `getState`/`screenshot` primitives, structured event log, and the user-facing cheat menu.

## Usage

### Verify a feature before you merge

```
> "QA the combat respawn before I merge"
```

Claude pulls in `verifying-browser-games`, dispatches a test-case author (player journeys + edge cases + adversarial inputs), then dispatches one execution subagent per domain in parallel. They drive the game as players, capture screenshots, write `verdict.json`. The orchestrator aggregates, fixes any failures, and surfaces only when everything is green.

### Validate a flow yourself

Hit `~` or `Ctrl+Shift+D` to open the cheat menu. Every action the agent has access to shows up as a button. Click "Kill Self" to test respawn, click "Set HP=1" to test low-health UX, click "Force Dawn" to skip the night cycle.

### Check the QA sheet

After every run, look in `qa-runs/<timestamp>_<feature>/`:

- `summary.md` — what was verified, in player-flow language
- `manifest.json` — pass/fail per domain
- `domains/<name>/screenshots/` — visual evidence
- `domains/<name>/verdict.json` — per-case observations

## What's in the bundle

| | |
|---|---|
| `/game-qa:bootstrap-game-qa-system` | One-shot setup. Adds the debug-system foundation to your project. Run once. |
| `verifying-browser-games` | The orchestrator skill. Strategy, dispatch, reporting. Auto-triggers when you ask Claude to verify or QA. |
| `enumerating-game-test-cases` | Test-case authoring subagent. Translates a feature into player journeys + edges. |
| `running-game-qa-pass` | Execution subagent. Drives the browser for one domain, returns a verdict. |

## Status

**v0.1.0 — pre-validation.** Designed against a real three.js multiplayer game; the debug-system foundation is implemented and shipping there. The skills haven't survived wide-cycle real-world use yet. Expect rough edges; PRs welcome.

## Manual install

If you'd rather copy directly:

```bash
git clone https://github.com/Bulugulu/game-qa.git
mkdir -p ~/.claude/skills ~/.claude/commands
cp -r game-qa/plugins/game-qa/skills/* ~/.claude/skills/
cp game-qa/plugins/game-qa/commands/bootstrap-game-qa-system.md ~/.claude/commands/
```

## License

MIT — see [LICENSE](./LICENSE).
