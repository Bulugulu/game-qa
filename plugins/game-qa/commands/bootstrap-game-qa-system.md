---
description: One-shot prompt to bootstrap a project's QA debug system — server-authoritative action registry, window.debug client proxy, getState/screenshot primitives, structured event log, and user-facing cheat menu. Run once per project.
---

You are bootstrapping a QA debug system for this browser/three.js project. The system is the foundation of the project's three-layer game-QA framework (`verifying-browser-games`). When you're done, the project will have a server-authoritative debug action registry, a `window.debug` proxy, `getState`/`screenshot`/structured-event-log primitives, and a user-facing cheat menu.

If parts of this already exist, gap-fill rather than rebuild. Don't break existing debug actions.

## Build, in this order

**1. Server-authoritative dispatcher.** A registry mapping action names to handlers. Each handler routes through the same canonical mutators that handle real game events. *If you'd write the same action twice (once for real, once for debug), the architecture is wrong — refactor the real path so debug can reuse it.*

**2. Reflection-driven catalog.** `debug.help()` queries the server registry and returns `{ actions: [{ name, args, doc }, …] }`. New action on server → automatically callable from the browser console. **No client-side action list to keep in sync.**

**3. Client `window.debug.<action>(args)` proxy.** A `Proxy` whose `get` trap returns a function for any property name. The function forwards `(actionName, args)` over whatever transport the project already has (Colyseus message, plain WS, REST). Don't introduce a new transport.

**4. `debug.getState()`.** Returns a JSON snapshot of whatever an agent would assert against — players, resources, time, match state. Pick a sensible default for this project; document how to extend it.

**5. `debug.screenshot()`.** Pauses animation/time-of-day for one frame, calls `renderer.render`, returns a canvas data URL.

**6. Structured event log.** Every dispatched debug action emits `console.log("[debug-event]", payload)` with `{ action, args, result, ok }`. Emit `console.log("[debug-ready]")` once the registry is wired. **These two markers are the contract** that the browser-driving QA tool (Playwright MCP today) scrapes for.

**7. User-facing cheat menu.** A floating panel behind a key combo (e.g. `~` or `Ctrl+Shift+D`). Lists actions from `debug.help()`. Each action gets a button or small form. Clicking calls the same action via the same transport — **same code path as the agent**. The user must be able to independently reproduce anything the agent verified.

## Don't

- **Don't decide what feature-specific actions to add.** Land 2–3 sentinel actions per major system as starter content (`setHP`, `kill`, `match.end`). The rest grows organically as features land.
- **Don't replace the project's stack.** Adapt to whatever's there.
- **Don't run continuously.** This is a setup prompt, not a watcher.

## When you're done

- All seven items above shipped (or gap-filled if some existed)
- Starter actions callable from both browser console and cheat menu
- Short README at `docs/qa/debug-system.md` showing how to extend the registry
- Tell the user: *"Debug system ready. Try `debug.help()` in the browser console, or hit the cheat-menu key combo. From here, run QA via the `verifying-browser-games` skill."*

Then exit.
