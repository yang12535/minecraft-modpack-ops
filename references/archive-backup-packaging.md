# Archive, Backup, And Packaging

## Contents

- Package intent
- Pre-archive state
- Backup retention
- Public packages
- Private restore packages
- Compression and resource planning
- Verification
- Transfer and storage
- Failure routing

## Define Package Intent

Choose the package class before selecting files.

- **Public player package:** safe to share with players or publish as a friend-facing artifact after privacy review.
- **Private full restore package:** contains credentials or deployment material for the operator. Treat it as secret material.
- **Game-only archive:** contains only the Minecraft server directory or selected game files.
- **Root-relative restore archive:** preserves paths such as `root/server`, `etc/systemd/system`, `etc/frp`, `etc/minecraft`, and `usr/local/bin` so a compatible host can restore by extracting from `/`.

If the user says "public" or "for players", default to the public model. If the user says "private", "disaster recovery", "full restore", "keys included", or "extract and deploy", default to the private full-restore model.

## Check Pre-Archive State

Before stopping or packaging:

1. query online players through RCON or the server console
2. inspect recent join/leave logs when the user asks "if nobody is playing"
3. run `save-all flush` before disruptive work when the server is active
4. stop through the normal lifecycle command when a consistent world snapshot is required
5. confirm Java and tunnel processes/listeners are in the intended final state
6. check free disk space on the source host and destination host

Do not infer "nobody is playing" from a quiet terminal. Use the Minecraft online count or logs.

## Tune Backup Retention

Backup mods can consume most of a small VPS disk while still obeying their configured limits.

Inspect at least:

- backup type: full backups versus modified-only backups
- interval or timer
- count limit such as `backupsToKeep`
- size limit such as `maxDiskSize`
- whether backups run with no players online
- whether subdirectories are created per world, because limits may apply per subdirectory
- output path

On small 30-50 GB disks, a full-backup policy often needs both a low count limit and a low size limit. A practical starting point for private friend servers is 3-4 retained full backups and roughly 4-5 GB total backup space, adjusted to actual world size and disk budget.

When cleaning existing backups:

1. list backups with timestamp, size, and path
2. keep the newest files unless the user names a specific recovery point
3. remove only backup archives owned by the target server
4. re-check directory size and disk free space
5. preserve a copy of the old backup config before editing retention settings

Do not include routine backup directories in public packages. Include retained backups in a private restore package only when the package is meant to preserve rollback history.

## Build Public Packages

Public packages should be treated like publishable artifacts even when shared only with friends.

Usually include:

- `mods`, `config`, `defaultconfigs`, `kubejs`, datapacks, resource packs, custom assets, libraries, loader files, run scripts, and the intended `world`
- server-side pack manifests or installer metadata needed to reproduce the pack
- a sanitized `server.properties`

Usually exclude:

- `logs`, `crash-reports`, backup directories, debug dumps, caches, and generated installer logs
- `ops.json`, `whitelist.json`, `banned-players.json`, and `banned-ips.json`
- `usercache.json`, username caches, player source IP logs, and raw UUID mapping notes
- RCON configs, FRP configs, FRP tokens, SSH keys, provider metadata, and systemd operational units
- local helper scripts that contain proxy addresses, credentials, private endpoints, or one-off deployment state

Sanitize configuration before adding it to the archive:

- clear `rcon.password`
- disable RCON unless the public package intentionally documents how to set a fresh password
- clear `server-ip`
- replace web or config UI passwords with placeholders
- remove private proxy settings and private API tokens
- keep intentional gameplay settings such as difficulty, online mode, view distance, simulation distance, player cap, and pack-specific configs

Prefer creating temporary sanitized overlay files and adding those to the tar stream. Do not modify the live server config merely to build a public artifact.

Run a path-level and content-level review before transfer. Search for names such as:

```text
password token secret rcon frpc frps proxy private_key ops.json usercache whitelist banned logs simplebackups crash-reports
```

Treat keyword hits as review prompts, not automatic exclusions.

## Build Private Full Restore Packages

Private restore archives may include credentials, but only after the user explicitly asks for a private or full restore package.

Useful contents include:

- complete Minecraft server directory
- retained backup archives, if rollback history is part of the requested deliverable
- lifecycle commands such as `mcstart-*`, `mcstop-*`, and RCON helpers
- systemd units and environment files
- RCON password files
- FRPC configs, FRPS configs, token files, and tunnel binaries needed for the specific deployment
- a short restore note inside the archive explaining expected extraction path and post-restore checks

For root-relative restore packages, preserve ownership and modes when possible:

```sh
tar --xattrs --acls -cpf - root/server etc/frp etc/minecraft etc/systemd/system usr/local/bin
```

After restoring on another host, verify:

- service users and groups exist
- file ownership and modes still protect tokens and RCON config
- `systemctl daemon-reload` has run
- provider NAT, public endpoints, and firewall rules match the new host
- global start/stop commands still point at the restored server path

Never paste private archive manifests, token hashes, passwords, or endpoint secrets into a public issue or repository.

## Plan Compression Resources

`tar.zst` is a good default for Linux-to-Linux archival work. High-compression settings such as:

```sh
zstd --ultra -22 --long=31 -T0
```

can use many CPU cores and a large amount of memory. A 2 GB window with multiple worker threads can exceed 8 GB of RAM and may require temporary swap. This is expected behavior, not a Minecraft-specific problem.

Before using aggressive compression:

- check free disk space for both the output archive and any temporary swap
- check available memory and swap
- decide whether the machine is idle enough to tolerate CPU and I/O saturation
- monitor for OOM kills
- remove temporary swap after compression

If the host is memory constrained, reduce threads before lowering the compression level. If the host is disk constrained, compress on a larger machine or stream the tar output to another host.

Remember that already-compressed backup zips, jars, images, and region data may not shrink much. An archive that is smaller than expected is often explained by excluded backups, logs, or caches; verify by comparing source directory sizes and manifest counts rather than guessing.

## Verify The Archive

Verification should be layered:

1. generate a SHA-256 file next to the archive
2. run a decompression test, for example `zstd -t --long=31 archive.tar.zst`
3. create a manifest with `tar -tf`
4. check required paths are present
5. check forbidden public paths are absent for public packages
6. inspect sensitive files from the archive stream, not only from the live filesystem
7. compare world region counts when package size looks suspicious
8. after copying, verify the destination hash matches

For worlds, compare counts of region-like files such as `.mca` between source and archive. Public package size alone is not proof that the world was truncated.

## Transfer And Store

Transfer archives with tools that preserve bytes and report failures, such as `scp`, `rsync`, or a verified object-store upload. Verify hashes after transfer.

Keep private restore packages out of:

- public repositories
- issue trackers
- shared documentation folders
- normal player distribution folders
- logs or chat transcripts

Public packages can be distributed more freely after privacy review, but they may still contain player builds, coordinates, playerdata, and world history. Treat worlds as sensitive unless the user explicitly wants them public.

## Failure Routing

- **Archive is unexpectedly small:** compare source size by directory, confirm backups/logs were intentionally excluded, and compare world region counts.
- **Archive is unexpectedly large:** inspect backup directories, logs, crash reports, caches, and already-compressed assets.
- **Compression is killed:** inspect OOM logs, add temporary swap, reduce `zstd` threads, or stream to a larger machine.
- **Disk fills during compression:** remove partial archives, avoid local temporary swap, exclude intended bulky directories, or compress on another host.
- **Public archive contains a secret:** delete the artifact, rebuild from sanitized overlays, and rotate exposed credentials if the package left the trusted environment.
- **Private restore package fails after extraction:** check service users, file modes, systemd reload, Java version, provider mappings, and path assumptions.

## Completion Report

Report:

- package class: public, private, game-only, or full-restore
- final archive path, size, and hash status
- backup retention changes and remaining backup count/size
- whether the server and tunnels were left running or stopped
- which validation stages passed: decompression, manifest, required paths, forbidden paths, hash after transfer
- unresolved restore requirements such as provider NAT recreation or missing service users

Do not report credentials, token hashes, player IPs, raw UUID mappings, or private endpoint details unless the user explicitly requests them inside the authorized environment.
