# KubeJS Server Automation

## Contents

- Decide whether to reuse KubeJS
- Design session-scoped delayed tasks
- Send private notices
- Register player commands
- Reload and validate
- Failure routing

## Reuse The Existing Script Runtime

Before adding another mod, inspect the pack for:

- KubeJS and its required JavaScript runtime
- `kubejs/server_scripts/`
- existing server scripts and naming conventions
- the exact Minecraft, loader, KubeJS, and runtime versions
- script logs and reload commands

Prefer an existing, healthy server-side scripting framework for a small policy or notification feature. Adding a new administration mod increases dependency and compatibility surface.

Do not remove or rewrite pack-authored scripts. Add a narrowly named file and keep configurable text, delays, and command names near the top.

## Design A Login Session

A delayed login action must belong to one connection session, not merely to a player identity.

On login:

1. capture the player's UUID as a string
2. generate a unique session token for in-process identity, such as a timestamp plus random value
3. store the token in the player's persistent data under a namespaced key
4. schedule one task for the requested delay

When the task runs:

1. look up the currently online player by UUID
2. stop if the player is offline
3. compare the current stored token with the captured token
4. stop if they differ, because the player reconnected and created a newer session
5. perform the action once
6. remove the token or mark the session complete

Do not retain the original player object inside a long delayed callback. Re-resolve the online player when the task executes.

This model naturally handles:

- many players with independent timers
- disconnect before the delay
- rapid reconnects
- an old timer firing after a new session begins
- exactly one action per completed session

It does not require a tick-by-tick polling loop.

## Send A Private Notice

Use the player message API, not a server-wide broadcast.

Keep the notice in one helper so automatic delivery and an on-demand command render identical content. Use multiple message components for readable lines and formatting.

Do not include:

- private infrastructure details unless players must use them
- credentials or management ports
- player-specific data
- temporary operational notes that will become misleading without an owner and expiry plan

If the notice includes a public endpoint, label primary, standby, and retirement status clearly and update it as part of relay lifecycle changes.

## Register An On-Demand Command

Register a literal player command that calls the same notice helper.

Decide explicitly:

- whether every player can run it
- whether console execution is supported
- whether aliases are useful
- whether the command should have a cooldown

If the implementation assumes a player command source, reject or handle console execution cleanly instead of dereferencing a missing player.

Command registration can depend on the data-pack or command-registry reload stage. Reloading only server scripts may parse the file without rebuilding the command tree.

## Reload And Validate

Back up the existing file when replacing one, then:

1. run a JavaScript syntax check when a compatible local runtime is available
2. install the script with normal server ownership and non-executable permissions
3. reload KubeJS server scripts
4. inspect the KubeJS server log for the named file
5. require a complete script count with zero new errors
6. run the normal server resource reload when command registration or data-pack events require it
7. test the command as a player context
8. verify Minecraft and any public tunnels remain active

Pack-authored recipe warnings may appear during a full resource reload. Compare timestamps and classify them separately from errors introduced by the new script.

For the delayed path, the strongest acceptance test is:

1. join after the script is active
2. remain connected for the configured delay
3. confirm only that player receives one notice
4. reconnect and confirm the timer restarts
5. disconnect before expiry and confirm the old session does not receive a later notice

Do not shorten the production delay silently just to make testing convenient. Use a temporary clearly marked test value only with authorization, then restore and reload it.

## Failure Routing

- script parses but command is unknown: run the required server resource reload and inspect command-registry errors.
- old session still sends after reconnect: store and compare a per-login token rather than checking only UUID.
- offline task throws an exception: re-resolve the player and return when absent.
- every player receives the notice: replace server broadcast APIs with the target player's message API.
- notice repeats continuously: schedule one callback instead of using a server tick loop.
- edited text does not appear: confirm the remote file changed, reload server scripts, and inspect the new log timestamp.
- unrelated recipe warnings appear: compare with prior logs and report them separately unless the new script touches recipes.

## Guardrails

- Keep server-only automation out of client script directories.
- Do not add a new mod when the existing script runtime safely covers the requirement.
- Do not identify sessions by player name.
- Do not persist session tokens outside the authorized server.
- Do not restart or disconnect active players when a supported hot reload is sufficient.
- Do not claim the delayed path is fully tested from a successful script reload alone.

## Completion Report

Report the script path, delay behavior, reconnect semantics, command name and permissions, reload result, new script error count, service state, and any acceptance stage that still requires a real player session.
