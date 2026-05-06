# game-qa-artifacts — schemas and folder layout

Reference for `verifying-browser-games`, `running-game-qa-pass`, and `enumerating-game-test-cases`.

## Folder layout

```
qa-runs/
  <YYYY-MM-DDTHH-mm>_<feature-slug>/
    test-cases.json         # output of enumerating-game-test-cases
    manifest.json           # orchestrator's aggregate
    summary.md              # human-readable report
    domains/
      <domain-slug>/
        verdict.json         # output of running-game-qa-pass (one per domain)
        screenshots/
          01-<step>.png
          02-<step>.png
```

JSON for machine-readable, single MD for human-readable. Screenshots are the heavy stuff; everything else is small.

## test-cases.json — output of enumerating-game-test-cases

```json
{
  "feature": "combat-respawn",
  "createdAt": "2026-05-05T22:25:00Z",
  "domains": [
    {
      "name": "respawn-flow",
      "happyPath": [
        {
          "id": "respawn-on-death",
          "playerFlow": "Player dies, respawns at base within 30s with full HP, can move",
          "setup": "debug.kill('p1')  OR  enemy attack with dmg > maxHP",
          "checks": [
            "state.players.p1.alive becomes false",
            "respawn HUD shows countdown",
            "after 30s, alive=true and HP=maxHP, position=base",
            "movement input works"
          ]
        }
      ],
      "edgeCases": [
        {
          "id": "respawn-near-enemy-spawn",
          "playerFlow": "Player dies inside enemy territory — should respawn at base, not death position",
          "setup": "debug.move('p1', enemySpawnPos) → debug.kill('p1')",
          "checks": ["respawn position is base, not death position"]
        }
      ],
      "adversarial": [
        {
          "id": "rapid-death-respawn",
          "playerFlow": "Player dies repeatedly within respawn window — system shouldn't double-fire",
          "setup": "kill, attempt second kill before respawn",
          "checks": ["only one respawn event per death"]
        }
      ]
    }
  ],
  "questions": [
    "If player disconnects during the respawn timer, what's the expected behavior on reconnect?"
  ]
}
```

## verdict.json — output of running-game-qa-pass (one per domain)

```json
{
  "domain": "respawn-flow",
  "overall": "fail",
  "ranAt": "2026-05-05T22:30:00Z",
  "cases": [
    {
      "id": "respawn-from-turret-kill",
      "playerFlow": "Player killed by a turret should respawn at base within 30s with HUD feedback",
      "outcome": "fail",
      "observation": "Player respawned at correct position, but [debug-event] kill not emitted; HUD respawn timer never appeared. From the player's seat, no feedback that they died or are about to respawn.",
      "evidence": ["screenshots/04-pre-kill.png", "screenshots/05-post-kill.png"]
    }
  ],
  "screenshotsDir": "screenshots/"
}
```

## manifest.json — orchestrator's aggregate

```json
{
  "feature": "combat-respawn",
  "ranAt": "2026-05-05T22:30:00Z",
  "overall": "pass",
  "domains": [
    { "name": "combat-damage", "verdict": "pass", "verdictPath": "domains/combat-damage/verdict.json" },
    { "name": "respawn-flow", "verdict": "pass", "verdictPath": "domains/respawn-flow/verdict.json" }
  ],
  "playerFlowsValidated": [
    "Player dies at match start, respawns within 30s, can move and attack",
    "Player killed by a turret takes the same flow as PvP death"
  ],
  "iterations": 2
}
```

## summary.md — orchestrator's human-readable report

Free-form markdown for the user. Recommended sections: headline, player flows validated (bullet list in player language), failures-and-fixes (only if surfaced), manual-validation instructions (cheat menu / debug commands / manual play), links to per-domain `verdict.json`.
