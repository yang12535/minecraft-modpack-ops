# World Replacement And UUID Migration

## Contents

- Archive inspection
- UUID model selection
- Complete data conversion
- Staged replacement
- World defaults and OP records
- Validation
- Guardrails

## Treat It As A Data Migration

World replacement is not merely copying a directory. Player identity may appear in filenames, text stores, standalone NBT, region/entity NBT, and mod-specific databases.

## Inspect The Archive

Before touching the live world:

1. Verify the checksum and test every ZIP member.
2. Reject absolute paths, traversal, and unexpected links.
3. Detect a direct world root versus one wrapper directory.
4. Reject ambiguous multiple roots.
5. Confirm `level.dat`.
6. Inventory player-keyed data.

Inspect at least:

- `playerdata`, advancements, and stats
- GameStages and cosmetic armor
- FTB Quests and FTB Teams
- economy, claims, parties, waystones, backpacks, and starter-kit tracking
- scoreboards and persistent mod stores
- ownership inside entities and block entities

Do not assume UUIDs appear only in filenames.

## Choose One UUID Model

Derive the target model from the final `online-mode`.

### Online Mode

Use the authenticated Mojang UUID for the canonical current player name. Query authoritative profile data and record the evidence.

### Offline Mode

Compute Java's name UUID over the exact case-sensitive bytes:

```text
OfflinePlayer:<player-name>
```

This is an MD5 version-3 UUID. Paid accounts also receive this offline UUID when joining an `online-mode=false` server.

Build an explicit mapping table:

```text
player name | source UUID | target UUID | source mode | target mode | evidence
```

Preserve the exact name casing players will use. Keep this table ephemeral or inside the authorized administration environment. Report mapping counts by default, not raw player names and UUIDs.

## Convert All Player References

Renaming `playerdata/<uuid>.dat` or changing `ops.json` alone is insufficient.

Convert:

- filenames and directories keyed by UUID
- textual UUIDs with and without hyphens
- NBT `IntArray[4]` UUIDs
- NBT `Most`/`Least` long pairs
- owner, thrower, trusted-player, claim, party, team, quest, economy, and notification references
- entity and block-entity ownership inside `.mca` files

Use structured JSON/TOML/SNBT/NBT parsers. Use an NBT-aware region writer rather than binary search-and-replace.

When rewriting region files:

- preserve each chunk's compression type
- preserve timestamps where practical
- rebuild the location table correctly
- leave unmodified chunks semantically unchanged
- reject external or unsupported chunk formats unless handled deliberately

Run the transformation twice on copies. The second pass must report:

- zero UUID value changes
- zero path renames
- zero region chunk rewrites

This idempotence check is a strong signal that old UUID forms were not missed.

## Replace Through Staging

1. Stop Minecraft gracefully.
2. Confirm Java and game/RCON listeners are gone.
3. Extract into a staging directory on the same filesystem.
4. Transform UUIDs and world defaults in staging.
5. Validate `level.dat`, expected player files, counts, and target UUIDs.
6. Remove stale `session.lock`.
7. Apply the user's retention decision:
   - keep a timestamped backup when requested or valuable
   - delete directly only when the user explicitly says the old world is disposable
8. Rename staging into the configured `level-name`.

Use a same-filesystem atomic rename such as `os.replace` for the final move. Never extract over a running or live world.

## Set Authentication And Defaults

Keep these consistent:

- `online-mode`
- `enforce-secure-profile`
- whitelist policy
- `op-permission-level`
- difficulty
- hardcore
- gamerules such as `keepInventory`

An imported `level.dat` can carry difficulty and gamerules that override assumptions from `server.properties`. Modify world NBT with a parser when required.

Verify runtime values through RCON:

```text
difficulty
gamerule keepInventory
```

## Build OP Level Four Correctly

For each known target player:

- use the target-mode UUID
- write the name and UUID to `ops.json`
- set `"level": 4`
- set `op-permission-level=4`
- update `usercache.json` consistently
- run idempotent `op <name>` after startup

The desired response is that the player is already an operator.

Offline mode permits username impersonation. Before public exposure, require a trusted network, strict whitelist, or a compatible authentication layer.

Treat the source world, transformed world, profile responses, `ops.json`, `usercache.json`, logs, coordinates, inventories, claims, and economy records as private player data. Do not copy them into a skill repository or retain temporary extraction directories after validation.

## Validate The Migrated World

### Static

- ZIP passes integrity checks
- world root is unambiguous
- all expected target UUID files exist
- old UUID filenames are absent
- mapping is idempotent

### Boot

- server reaches `Done (...)`
- no destructive world, registry, or player-data load failure appears
- expected dimensions load

Obsolete recipe or advancement warnings can be nonfatal; classify them separately.

### Runtime

- RCON reports intended difficulty and gamerules
- `save-all flush` succeeds
- OP commands are idempotent
- logs show target UUIDs at login

### Player Acceptance

Join with representative converted accounts and verify:

- inventory and selected slot
- ender chest
- position and dimension
- advancements and stats
- quests and teams
- economy and claims
- pets, mounts, backpacks, and owned entities

Do not declare identity migration fully accepted without a real player login when a matching client is available.

## Failure Routing

- empty inventory after login: target mode and playerdata UUID disagree.
- vanilla data survives but mod data resets: UUIDs remain in mod-specific filenames or stores.
- pets or claims lose ownership: embedded NBT or region data was not converted.
- OP entry ignored: `ops.json` UUID follows the wrong authentication mode or permission level is lower than four.
- server regenerates a world: staging root or `level-name` is wrong.
- `session.lock` error: stale lock copied from the source.

## Guardrails

- Never mix online and offline UUIDs after choosing the target mode.
- Never infer a paid player's UUID in offline mode from their Mojang profile.
- Never perform UUID migration with raw binary replacement.
- Never overwrite the live world while Minecraft is running.
- Never discard an old world unless the user explicitly permits it.
- Never expose an offline-mode server publicly without explaining name impersonation risk.
- Never include raw UUID maps, player inventories, coordinates, profile properties, or full identity-bearing logs in a public report.

## Completion Report

Report the source world generically, target authentication mode, player mapping count, converted data categories, backup/delete decision, difficulty/gamerules, OP result, boot result, and player-login acceptance status. Reveal player names or UUIDs only when the authorized requester needs them. Do not print credentials or private profile properties.
