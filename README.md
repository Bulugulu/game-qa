# game-qa

**Turn your agent into a Game QA lead.**

## Why this exists

Agents are great at writing code. They're bad at verifying games — and verification is the bottleneck for autonomous agentic coding. If you can't trust what your agent says is "done," you can't actually let it ship.

This plugin makes Claude operate like a real QA team for browser-based games. The key shift from typical code QA: we verify real player flows from the player's perspective, we verify visually, and we give the agent the tools to actually do the verification. Game verification is context-heavy, so we split the work across subagents in a PM-driven loop and stream findings live.

## Why browser games specifically

Browser-based games (three.js, canvas, WebGL, DOM) are uniquely well-suited to fully agent-authored development. There's no game engine UI to click through, no proprietary editor, no asset pipeline. Claude can write the whole game in code — runtime, rendering, gameplay, networking — and you can stay out of the loop.

That makes the missing piece *verification*. If the agent can write the game but can't prove a feature works the way a player would experience it, you can't actually let it ship. This plugin closes the loop: the agent codes the feature, the agent verifies the feature, and only then does it tell you "done."

## How it works — two pieces

game-qa has two parts that come together. They're separate concerns and worth understanding separately.

### 1. The skills — a PM-driven QA loop

A skill family that teaches Claude how to verify games like a real QA team. **These trigger automatically.** When the agent finishes a feature in a game repo, it recognizes that game QA applies — you don't have to prompt it.

The loop has five roles, each a separate skill, all spawned as subagents so observation-heavy work stays out of your main context:

- **Feature owner** (`requesting-game-qa`) — the agent that built the feature, now wearing a PM hat. Writes a brief, signs off the test plan, watches tickets stream in, fixes as findings land.
- **Enumerator** (`enumerating-game-test-cases`) — translates the brief into player-journey test cases. One happy path per player promise, walked to its edges, grouped by execution domain so journeys can run in parallel. Declares per case: input mode, review mode, evidence requirements, arrangement primitives needed.
- **QA Lead** (`game-qa`) — validates adapter coverage *first* (gaps surface before engineers waste time), plans journeys, dispatches engineers and reviewers in parallel, rolls up a single verdict.
- **QA Engineer** (`running-game-qa-pass`) — authors a journey YAML, runs it via the project's runner, returns evidence-bundle pointers. Captures; never judges.
- **QA Reviewer** (`reviewing-game-qa`) — owns the cases, judges each from the evidence, files tickets immediately so the feature owner can fix in parallel.

The whole loop in one paragraph: *Engineer signals ready → brief → plan signed off → Lead validates adapter coverage → engineers + reviewers run in parallel → tickets stream live to the feature owner who fixes as they land → one verdict comes back.*

Why it's structured this way:

- **A QA lead's mindset.** Enumerate player journeys, walk every journey to its edge cases, report in player language — not assert language. *Without this, agents claim "tests pass" once unit tests go green. That's the false-completion bug.*
- **Visual validation.** The agent sees the game the way the player would. When visual targets exist (mockups, design specs, prior screenshots), the agent compares against them. *A unit test can't see the screen.*
- **Role separation, parallelism by default.** Authoring, execution, and review are separate subagents — engineers and reviewers run in parallel. *One mega-agent doing end-to-end QA gets stuck or runs out of context, and you lose every observation it made.*
- **Streaming tickets.** Findings file immediately to `tickets/`, not batched at the end. *The feature owner fixes capability-gaps first (they block QA), then bugs at their own pace — no waiting on a final report to start fixing.*
- **A persistent QA sheet.** Every run writes a journey YAML, screenshots, per-case review JSONs, and a human-readable summary to `qa-runs/`. *"Trust me, I tested it" doesn't scale.*
- **Only-when-green reporting.** The feature owner runs dispatch → fix → re-dispatch internally and surfaces only when every flow passes. *You should see "done" once, not 12 messages of dispatch noise.*

### 2. The agentic QA interface

A foundation the agent builds into *your game* so the QA loop can actually drive the verification. The skills above need real tools to do their job — a way to arrange state, capture evidence, observe events. **The plugin ships the framework as templates:** a project-agnostic runner you copy verbatim, an adapter skeleton you fill in against your project's `window.debug.*`, and a journey-schema doc that pins down the YAML contract. The bootstrap skill walks the agent through copying and adapting — not reconstruction from natural language.

You run it once per project (just ask Claude — the `bootstrap-game-qa-system` skill auto-triggers):

> Bootstrap the game-qa system for this project.

Claude adapts the interface to whatever stack you already have. No new dependencies. What you end up with:

- **Arrange primitives** (`arrange.*`). One call to set up any moment under test — your project's state-setup actions wrapped behind a stable namespace. *Without this, the agent has to play through the game to reach the state. Slow, brittle, burns context on irrelevant gameplay.*
- **Probe primitives** (`probe.*`). Named state queries — small wrappers around your project's read-only debug actions. Reviewers check returned shapes against expected values. *Reading global state ad-hoc is fragile; named probes make the contract explicit.*
- **Event accumulator** (`events.subscribe` / `events.drain`). Subscribe to a channel before the action; drain after. *Captures evidence the screenshot can't.*
- **A project-agnostic journey runner** at `qa/runner.ts`. Reads journey YAML, drives Playwright as a library, emits an evidence bundle. *Engineers author data, not code — the runner is the runtime contract.*
- **Arrange-vs-shortcut taxonomy.** Allowed: setup verbs (`set*`, `spawn*`, `give*`, `grant*`, `clear*`). Forbidden: any `debug.*` whose name describes what the *current case* is verifying. *The discipline that prevents synthetic shortcuts — a primitive that bypasses the chain being verified isn't a test, it's a lie.*
- **A cheat menu for you.** The same actions the agent uses, exposed behind a key combo (`~` or `Ctrl+Shift+D`). Click any cheat-menu action to reproduce what the agent claimed. *The agent's verification is only useful if you can independently reproduce it.*

**Without piece 2, the skills are an instruction manual for tools that don't exist. Without piece 1, the interface is just a debug system. Together, the agent can verify features autonomously.**

## What a run looks like

The ideal flow is invisible. You finish a feature with Claude. Without you prompting, it recognizes that the QA skills apply (it's a game repo, the skills are installed) and runs the verification process itself.

You don't see:

- The feature owner writing a brief and spawning the enumerator
- The enumerator returning N player-journey cases across M execution domains, each declaring input mode, review mode, evidence requirements
- The feature owner signing off the plan
- The QA Lead validating adapter coverage and dispatching engineers + reviewers in parallel
- Reviewers filing tickets live as findings emerge — capability gaps first, then bugs
- The feature owner fixing as tickets land, re-dispatching affected slices
- Per-case reviews aggregating, summary written to `qa-runs/`

You see one message:

> Feature verified across N player flows + M edge cases. All green.
>
> **Player flows validated:** (a short bulleted list, in player language — what the player would feel, not what an assertion would check)
>
> QA sheet: `qa-runs/<date>_<feature>/summary.md`
>
> To validate manually: hit `~`, trigger the relevant cheat-menu action, watch the flow.

You can also prompt explicitly — *"QA this before I merge"* — but the whole point is you don't have to.

## Installation

```bash
/plugin marketplace add Bulugulu/game-qa
/plugin install game-qa@game-qa
```

Then, once per project, ask Claude:

> Bootstrap the game-qa system for this project.

The `bootstrap-game-qa-system` skill handles the rest.

## Browse the skills

The plugin is plain Markdown — read each skill directly:

- [`requesting-game-qa`](./plugins/game-qa/skills/requesting-game-qa/SKILL.md) — feature owner's entry. Compile a brief, drive test-plan sign-off, watch tickets, fix as they land.
- [`enumerating-game-test-cases`](./plugins/game-qa/skills/enumerating-game-test-cases/SKILL.md) — author the test plan: player journeys per execution domain, with input mode, review mode, and evidence requirements declared per case.
- [`game-qa`](./plugins/game-qa/skills/game-qa/SKILL.md) — QA Lead orchestrator. Validates adapter coverage, dispatches engineers and reviewers in parallel, rolls up a single verdict.
- [`running-game-qa-pass`](./plugins/game-qa/skills/running-game-qa-pass/SKILL.md) — QA engineer: author a journey YAML, run it, return evidence pointers. Never judges.
- [`reviewing-game-qa`](./plugins/game-qa/skills/reviewing-game-qa/SKILL.md) — QA reviewer: own the cases, judge each from the evidence, file tickets immediately.
- [`bootstrap-game-qa-system`](./plugins/game-qa/skills/bootstrap-game-qa-system/SKILL.md) — copy-and-adapt onboarding. Ships a project-agnostic [runner template](./plugins/game-qa/skills/bootstrap-game-qa-system/references/runner.ts), an [adapter skeleton](./plugins/game-qa/skills/bootstrap-game-qa-system/references/qa-adapter.template.ts), and a [journey schema](./plugins/game-qa/skills/bootstrap-game-qa-system/references/journey-schema.md). The agent copies the runner verbatim, fills in the adapter against your `window.debug.*`, and smoke-tests the rig.

## Status

**v0.1.1 — pre-validation.** Designed against and exercised on a real three.js multiplayer game. The framework is engine- and architecture-agnostic by design (single-player, multiplayer, any browser game) but hasn't been stress-tested outside that origin context. Expect rough edges; PRs welcome.

## License

MIT — see [LICENSE](./LICENSE).
