# FRP And NAT Exposure

## Contents

- Network path mapping
- Binary acquisition
- FRPS hardening
- FRPC configuration
- Start/stop coupling
- Validation
- Failure routing

## Map Every Hop

Document the path before configuration:

```text
player -> public game endpoint -> provider NAT -> frps remote port
frpc -> public control endpoint -> provider NAT -> frps bind port
frpc -> local Minecraft address and port
```

Provider NAT ports and FRP ports are separate layers. A provider may map a public control port to VPS `7000` and a public game port to VPS `25565`.

Determine:

- relay OS and architecture
- internal FRPS bind port
- FRPS remote game port
- provider public mappings
- local Minecraft address and port
- firewall and listener conflicts
- outbound reachability from FRPC to the public control endpoint

FRP TCP proxying does not require enabling IP forwarding.

## Acquire FRP Reliably

- identify the current official release
- download the exact architecture build
- verify the published checksum
- upload binaries from a trusted machine when the relay has no Internet egress
- record the installed version on both sides

Do not install an unverified binary from an arbitrary mirror.

## Harden FRPS

Run FRPS as an unprivileged systemd service.

Configure:

- a fixed bind address and control port
- file-backed random token authentication
- forced TLS
- `allowPorts` restricted to the required Minecraft remote port
- a low per-client port limit
- bounded log retention
- dashboard disabled unless explicitly required

Protect:

- token file: mode `0600`
- configuration: root-owned and only group-readable when required
- log directory: writable only by the service account

Use systemd hardening such as:

- `NoNewPrivileges=true`
- `PrivateTmp=true`
- `ProtectHome=true`
- `ProtectSystem=strict`
- a narrow `ReadWritePaths`

Enable FRPS across relay reboots. The relay service is infrastructure and normally remains active even when Minecraft is stopped.

## Configure FRPC

Run FRPC under a separate unprivileged account and systemd service.

Set:

- public FRPS address
- provider-mapped control port
- identical file-backed token bytes
- TLS
- local Minecraft bind address and port
- FRPS remote game port
- TCP health check against Minecraft
- bounded log retention

Health checks prevent FRPS from advertising a proxy before the local game listener exists.

Keep FRPC disabled for independent boot activation when the user requires it to follow Minecraft start/stop commands.

## Couple FRPC To Minecraft

Preserve existing PID, RCON, logging, and detach behavior.

Back up scripts, edit narrowly, run `bash -n`, then install.

The start command should:

- start or detect Minecraft
- start FRPC after a process exists
- repair FRPC if Minecraft was already running but the tunnel was missing
- report tunnel failure without losing local server control
- print local and public endpoints, not credentials

The stop command should:

- save and stop Minecraft gracefully
- stop FRPC after the game exits
- stop FRPC even when Minecraft is already absent
- leave FRPS running on the relay

Do not let FRPC's process hold the SSH invocation open.

## Transfer Credentials Safely

- generate the token on a server
- stream it over SSH directly to the other server
- do not write it to the local workspace
- do not echo it into chat or logs
- disable shell tracing before handling credentials
- avoid passing credentials in command arguments or environment dumps
- compare SHA-256 hashes inside the authorized session afterward
- confirm the files have identical bytes, including line endings

Authentication succeeding once is not a reason to ignore a byte mismatch; normalize it to avoid version-dependent parsing behavior.
Do not copy the token or its hash into the completion report, repository, issue tracker, or other persistent artifact.

## Validate Four States

### FRPS Alone

- FRPS service is active
- control port listens
- Minecraft proxy port does not listen

### Minecraft Plus FRPC

- FRPC logs in successfully
- health check changes to success
- proxy starts
- FRPS opens the remote game port

### Public Endpoint

Use the Minecraft status protocol or a real client. Verify:

- protocol/version
- player count and cap
- MOTD
- reasonable response latency

A plain TCP connect is insufficient. Provider NAT may accept a TCP handshake while the backend is absent.

### Lifecycle

With no players online:

1. run the global stop command
2. confirm all dimensions save
3. confirm FRPC becomes inactive
4. confirm FRPS closes the game proxy port
5. confirm Minecraft status fails publicly
6. run the global start command
7. wait with bounded polling
8. confirm RCON readiness
9. confirm FRPC health success
10. confirm public Minecraft status recovers

Leave the server in the requested final state.

## Failure Routing

- control public port unreachable: inspect provider NAT, outbound filtering, and FRPS bind listener.
- FRPC authentication failure: compare token hashes and TLS/auth settings.
- FRPC logs in but proxy is rejected: inspect `allowPorts`, port limit, and existing listener conflicts.
- proxy starts only after a delay: inspect local TCP health checks and Minecraft startup time.
- public TCP connects but status times out: NAT edge is reachable but FRPS proxy or backend is absent.
- FRPS remote port remains after FRPC stop: inspect stale client/control state and service logs.
- FRPC starts at boot unexpectedly: disable systemd enablement and retain explicit command coupling.
- Minecraft stops but tunnel remains: ensure every early-return path in the stop script calls the FRPC stop helper.

## Guardrails

- Do not expose RCON through FRP unless explicitly required and strongly restricted.
- Do not expose the FRPS dashboard by default.
- Do not allow arbitrary remote ports when one game port is sufficient.
- Do not run FRPS or FRPC as root without a concrete need.
- Do not put tokens directly in world-readable TOML.
- Do not validate only from the relay itself; test the player-facing public endpoint.
- Treat relay IPs, provider mappings, client source addresses, and FRP logs as sensitive infrastructure data. Disclose exact values only to the authorized requester.

## Completion Report

Report FRP version, relay and local service states, mapping status, TLS and token-file authentication, health-check behavior, global command coupling, and public Minecraft status result. Reveal exact endpoints only when needed by the authorized requester. Never report the token or its hash.
