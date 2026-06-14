---
name: minecraft-modpack-ops
description: Deploy, update, migrate, slim, expose, and troubleshoot modded Minecraft dedicated servers over SSH. Use for CurseForge or Modrinth exports, Forge/Fabric/NeoForge packs, client-only mod pruning, proxy-assisted downloads, world replacement with online/offline UUID migration, global start/stop commands, RCON, FRP tunnels, NAT port mapping, player handshake failures, and performance validation.
---

# Minecraft Modpack Ops

## Purpose

Use this skill as the routing entrypoint for modded Minecraft server operations. It is product-neutral and can be followed by any automation agent with suitable local shell, filesystem, network, and SSH access. Read only the reference files required by the current task; do not load every guide by default.

## Route The Task

- **Deploy, update, repair, slim, tune, or operate a modpack:** read [references/modpack-deployment.md](references/modpack-deployment.md).
- **Replace a world, change online/offline mode, migrate UUIDs, preserve player/mod data, or rebuild OP records:** read [references/world-migration.md](references/world-migration.md).
- **Expose Minecraft through a VPS, provider NAT, FRPS/FRPC, or couple a tunnel to start/stop commands:** read [references/frp-tunneling.md](references/frp-tunneling.md).
- **Mixed task:** read each relevant reference, then combine their validation requirements. For example, a world replacement followed by public exposure requires both the world and FRP guides.

## Shared Principles

- Inspect the artifact, host, running services, ports, and existing conventions before changing anything.
- Preserve content and user data by default. Remove or rewrite only what evidence supports.
- Run Minecraft and tunnel processes independently of the SSH session.
- Keep download proxies scoped to download commands.
- Keep RCON passwords, FRP tokens, SSH keys, and other credentials on the servers and out of chat or workspace files.
- Treat player names, UUIDs, IP addresses, public endpoints, logs, world archives, `ops.json`, and `usercache.json` as sensitive operational data. Do not commit or persist them outside the authorized environment.
- Avoid placing secrets in command arguments, shell history, debug tracing, process listings, or logs.
- Separate boot, network, client handshake, lifecycle, and runtime acceptance tests.
- Prefer exact versions and verify available hashes.
- Back up existing scripts before modifying them and preserve unrelated services or user changes.
- Use structured parsers for manifests, JSON, TOML, NBT, and region data.
- Never claim success from a process existing or a TCP connect alone; validate the actual Minecraft behavior.

## Completion

Report only high-signal operational facts:

- what was installed or changed
- local and public endpoint status; reveal exact addresses only to the authorized requester and never in public artifacts
- authentication and permission mode
- start/stop/RCON/tunnel behavior
- boot, status handshake, login, graceful stop, and restart results
- unresolved warnings or untested acceptance stages

Never include credentials, token hashes, player IPs, raw UUID mappings, or unnecessary log excerpts.
