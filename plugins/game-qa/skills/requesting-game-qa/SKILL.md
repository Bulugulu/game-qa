---
name: requesting-game-qa
description: Use ONLY for QA on browser-game feature work. Triggers when the engineer signals readiness for QA: "request QA on X", "verify X", "QA this", "ready for QA on X", "this should be ready, can we make sure it works?" Compiles a brief, drives test-plan sign-off, then hands off to game-qa. Skip for tooling, build scripts, infra, or non-game code.
---

# Requesting Game QA

You are the feature owner. You built the feature; now you're the PM driving it through verification. This skill is **for browser-game features only** — not tools, not build scripts, not generic code review.

**Core principle:** You own the feature's quality bar. QA executes; you set scope, sign off the plan, and unblock as findings stream.

## The Iron Law

```
GAME FEATURES ONLY.
NO BRIEF, NO QA. NO SIGN-OFF, NO EXECUTION.
WATCH TICKETS LIVE — DON'T WAIT FOR THE VERDICT.
```

## What Will Happen

You don't run QA yourself — your context is full of feature code. You **spawn sub-agents** to do the work, while you stay in your own context to fix what they find.

1. **Write the brief.** One file, every field filled.
2. **Spawn enumerator sub-agent** with the brief → it invokes `enumerating-game-test-cases`, writes `test-cases.json`, terminates. Short interaction.
3. **Sign off the plan.** Read `test-cases.json`. Edit / cull / question. Confirm it matches what you actually built and care about.
4. **Spawn QA Lead sub-agent** with the signed-off plan → it invokes `game-qa`. The Lead validates adapter coverage, then spawns its own engineers and reviewers in parallel, rolls up tickets, returns one verdict.
5. **Watch `<artifactDir>/tickets/` continuously** during step 4. Fix bugs as they land — `capability-gap` tickets first (they block QA), then bugs at your own pace.
6. **Lead returns `verdict.json`.** You read it, decide ship / no-ship.

Two spawns from you. Inside the Lead's spawn, more sub-agents work — but you never see them. You see only the brief, the plan (for sign-off), tickets (streamed), and the final verdict.

## What to Produce (this skill)

`<artifactDir>/brief.md`:

```yaml
feature: <slug>
spec: <design-doc path + section anchor>
goals: <one paragraph — what player promise this ships, in your own words>
scope:
  in:  [<player promise>, <player promise>, ...]
  out: [<deferred>, <covered by unit tests>, ...]
unitCoverage:
  - <path>: <count> asserts (what unit tests already cover — don't re-test)
arrangePrimitivesNeeded: [<your best guess at adapter actions QA will need>]
severity: blocking-for-merge | nice-to-have | quick-spotcheck
artifactDir: qa-runs/<date>_<feature>/
```

## Hard Rules

- **GAME features only.** If the work is a tool, build script, or infra change, route elsewhere — this skill is wrong for that.
- **Every brief field filled.** "TBD" / "see spec" is rejection-worthy.
- **Scope is binary.** A promise is `in` or `out`. If unsure, clarify before requesting QA.
- **Sign off the plan before invoking `game-qa`.** Over-exhaustive plans waste tokens; under-scoped plans miss bugs. You are the only one who can judge.
- **Don't enumerate cases yourself.** `enumerating-game-test-cases` does that. You evaluate the output.
- **Watch tickets continuously during execution.** Capability-gap tickets block QA — fix them first. Bug tickets you fix at your own pace.

## Red Flags — STOP

- Skipping the sign-off step → QA runs the wrong cases, wastes a full cycle
- About to write test cases inline → that's the enumerator's job
- Refusing capability-gap tickets ("just test it some other way") → silent compromise; QA will mark `unverified-pending-tooling` and the verdict will be incomplete
- Treating non-game work as QA-able here → use a different skill

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "Small feature, brief is overkill" | Brief is cheap; mid-cycle context recovery isn't |
| "I trust the QA Lead's plan, I don't need to review" | Lead doesn't know your goals — only you do |
| "I'll just verify it myself" | Engineer-as-tester misses player-perspective gaps |

## Related Skills

- `enumerating-game-test-cases` — invoked by you to draft the plan
- `game-qa` — invoked by you after sign-off; orchestrates the rest
- `references/brief-schema.md` — full brief field reference
- `references/ticket-schema.md` — how to read incoming tickets

## The Bottom Line

You're the PM. Brief, sign off the plan, watch `tickets/`, fix as they land.
