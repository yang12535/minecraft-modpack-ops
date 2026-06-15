# Minecraft Modpack Ops

> **Language:** This is the English version. For Chinese (default), open the repository's [`zh/` directory](https://github.com/yang12535/minecraft-modpack-ops).

A product-neutral operational skill for deploying and maintaining modded Minecraft dedicated servers.

The repository is documentation-first. It does not depend on Codex or a particular model provider. Any automation agent can use it if the agent can:

- read Markdown files
- inspect and edit local files
- run local shell commands
- connect to authorized hosts over SSH
- perform network and application-level validation

The agent must still have explicit authorization for every target system.

## Scope

The skill covers four related workflows:

- modpack deployment, conservative client-only pruning, administration, and performance validation
- world replacement with online/offline UUID migration and mod-specific player-data preservation
- FRP and provider-NAT exposure with secure lifecycle coupling to Minecraft
- historical stability analysis and fair multi-exit comparison across FRP, network, and Minecraft layers

`SKILL.md` is intentionally short. It routes the agent to only the relevant file under `references/`.

## Repository Layout

```text
.
|-- README.md          (Chinese, default)
|-- SKILL.md           (Chinese routing, default)
|-- zh/                (Chinese full text)
|-- en/                (English full text)
|-- references/        (kept for backward compatibility)
`-- agents/            (kept for backward compatibility)
```

The default language is Chinese. For English documentation, use the files under `en/`. The root `references/` and `agents/` directories are retained for backward compatibility with existing setups.

## Use With Any Shell Agent

Give the agent access to this repository and instruct it to begin with `en/SKILL.md`:

```text
Read /absolute/path/to/minecraft-modpack-ops/en/SKILL.md and follow its routing
instructions for this Minecraft server task. Use the available shell and SSH
tools, preserve unrelated services, and complete the relevant validation stages.
```

Agents or frameworks with a skill-directory convention should copy or symlink only the `en/` subdirectory into that directory; after installation, its entrypoint is the copied directory's root `SKILL.md`. Do not install the whole repository where skills are discovered recursively, because the root, `zh/`, and `en/` entrypoints share the same skill name. Otherwise, reference `en/SKILL.md` from the agent's system instructions, project instructions, or task prompt.

The workflow assumes the agent can adapt commands to the operating system and existing service conventions. It intentionally does not provide a universal deployment script.

## Examples

```text
Deploy this CurseForge export as a Forge dedicated server. Remove only mods
proven to be client-only, add graceful global start/stop commands, and validate
a real client login.
```

```text
Replace the stopped server's world with this archive. The target will use
offline mode, so migrate all player and mod UUID references before startup.
```

```text
Expose this private Minecraft server through an FRP relay. Harden FRPS, couple
FRPC to the existing Minecraft lifecycle commands, and validate the public
Minecraft status protocol.
```

```text
Compare the recent stability of these two public exits. Separate passive logs
from active probes, exclude any retired relay, and report Minecraft timeouts,
lag, memory, GC, swap, and confidence in exit attribution.
```

## Security And Privacy

This repository contains no production endpoints, player identities, UUIDs, credentials, or server-specific paths.

When using the skill:

- keep SSH keys, RCON passwords, FRP tokens, and proxy credentials on authorized hosts
- never place secrets in command arguments, shell tracing, process listings, logs, chat, or repository files
- treat player names, UUIDs, IP addresses, coordinates, inventories, worlds, logs, `ops.json`, and `usercache.json` as sensitive data
- keep UUID mapping tables and extracted worlds ephemeral
- redact identity-bearing log excerpts before storing or sharing them
- disclose exact infrastructure endpoints only to the authorized requester
- validate credential equality without publishing the credential or its hash
- clean temporary archives, extraction trees, and downloaded binaries after validation

The included `.gitignore` is defense in depth, not a substitute for reviewing staged changes before every commit.

## Operational Expectations

- Inspect before changing.
- Preserve user content and unrelated services.
- Prefer structured parsers over raw text or binary replacement.
- Keep long-running services detached from the agent's shell session.
- Use RCON for graceful saves and stops when available.
- Validate Minecraft itself, not merely a process or TCP listener.
- State clearly when a real client login or other acceptance stage remains untested.

## Review Status

The initial repository was reviewed for:

- embedded IP addresses, usernames, UUIDs, emails, and local machine paths
- credentials, tokens, passwords, and private-key references
- instance-specific service names and commands
- Codex-only assumptions in the core workflow
- accidental inclusion of worlds, logs, archives, binaries, or player data

The core documentation is platform-neutral; only `agents/openai.yaml` is provider-specific and optional.
