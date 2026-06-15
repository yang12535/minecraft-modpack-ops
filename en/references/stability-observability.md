# Stability And Multi-Exit Observability

## Contents

- Observation scope
- Node lifecycle
- Layered health
- Historical event counting
- Controlled active tests
- Minecraft runtime stability
- Multi-exit comparison
- Failure routing

## Define The Observation Window

State the exact start and end time, timezone, and reason for the window before counting events.

Determine:

- which Minecraft process lifetime is in scope
- which FRPS and FRPC services were expected to be active
- when each relay was created, stopped, replaced, or destroyed
- whether configuration or provider NAT mappings changed
- whether test probes, uploads, backups, or maintenance occurred
- whether clocks and journal timestamps are aligned

Do not compare a live relay with a relay that was stopped or destroyed during the window. Split the window at lifecycle changes or mark the retired relay as not comparable.

## Verify Node Lifecycle First

Before explaining failures:

1. confirm the instance still exists in the provider control plane when available
2. confirm its expected public address and NAT mappings are current
3. check boot time and service start time
4. identify intentional shutdown, deletion, replacement, or expired billing
5. annotate the effective retirement time

Connection refusal, timeout, SSH banner stalls, and endless FRPC retries are expected after relay retirement. Do not attribute these samples to carrier quality or FRP reliability.

## Distinguish Layered Health

Treat each layer as separate evidence:

1. provider instance exists
2. public NAT edge accepts traffic
3. relay OS responds
4. FRPS is listening and authenticated
5. FRPC has an established control session
6. the proxy is registered and healthy
7. the local Minecraft listener is ready
8. the public Minecraft status protocol succeeds
9. a matching client reaches the world

These signals are insufficient on their own:

- `systemctl active`: a retrying FRPC process can remain active indefinitely
- process exists: it may be disconnected or wedged
- ICMP succeeds: the game or control mapping may still be broken
- TCP connects: a provider NAT edge may accept the handshake without a working backend
- stale established sockets: old work connections may survive after control health is lost

Require recent FRPC login success, proxy registration, health-check success, and an application-level Minecraft response before calling an exit healthy.

## Count Historical Events Carefully

Keep passive historical counts separate from active tests.

For every count, record:

- source log and host
- observation window and timezone
- event pattern used
- sample count
- success and failure definitions
- exclusions and known blind spots

Useful FRP events include:

- FRPC control connection attempts
- successful logins
- authentication or dial errors
- proxy start and close events
- health-check transitions
- FRPS accepted connections
- work-connection failures
- service starts and restarts

Useful Minecraft events include:

- joins
- login-stage timeouts
- runtime timeouts
- clean client disconnects
- kicks and handshake failures
- server restarts and crashes

Do not assume FRPS accepted connections equal successful player logins. Status probes, scanners, and failed handshakes also create connections. Minecraft behind FRPC may see the proxy address instead of the player's source address, so backend logs may not identify which public exit a player used.

Avoid double-counting:

- one timeout may produce both `lost connection` and `left the game`
- one active probe may appear in FRPS and Minecraft logs
- service retry loops may produce repeated attempts without independent user traffic

## Run Controlled Active Tests

Use the same probe implementation, source host, timing, timeout, and sample count for every live exit.

Prefer the Minecraft status protocol over raw TCP. Report:

- attempts
- successes and failures
- success rate
- average, minimum, maximum, and P95 latency
- protocol or response mismatches

Run enough samples to expose intermittent failure, but bound the rate to avoid affecting play. Label active test traffic so it is not later mistaken for passive history.

When players are online, avoid disruptive lifecycle tests. Prefer status probes, read-only service inspection, and log analysis.

## Inspect Minecraft Runtime Stability

Check:

- process uptime and restart count
- current and peak resident memory
- configured heap and actual heap occupancy
- young, concurrent, and full GC behavior
- host available memory and swap use
- TPS or MSPT under representative exploration
- `Can't keep up` frequency and worst delay
- OOM, watchdog, crash, and failed-save records
- disk free space and world growth
- join, timeout, and disconnect patterns

Do not infer a memory leak from low Linux `free` memory alone. Compare process RSS, heap occupancy after GC, native overhead, page cache, and swap trend over time.

No Full GC or OOM does not mean memory headroom is adequate. A host with almost no available memory and growing swap can still be operational while having poor recovery margin. Recommend more host memory, a smaller heap, or lower distances only after accounting for native JVM and OS reserve.

Treat isolated `Can't keep up` records during startup or chunk generation differently from sustained high MSPT. Report count, worst delay, workload context, and whether the condition persisted.

## Compare Multiple Exits

Build one row per exit with a common window:

| Field | Meaning |
| --- | --- |
| Lifecycle | live, stopped, retired, destroyed, or unknown |
| FRPS state | listener, start time, restarts |
| FRPC state | recent login, retries, health status |
| Passive connections | accepted or observed without generated probes |
| Passive failures | explicit errors within the same layer |
| Active status probes | successes, failures, latency summary |
| Player evidence | joins, timeouts, or reports attributable to the exit |
| Confidence | confirmed, inferred, or unavailable |

Never rank a destroyed or otherwise inactive exit against a live one. Mark it excluded and explain why.

Avoid combining unlike numbers. Examples:

- FRPS accepted connections are not directly comparable with Minecraft joins.
- FRPC retry failures are not player connection failures.
- active probes must not be added to passive historical totals.

## Failure Routing

- process active but repeated dial errors: FRPC is alive in a retry loop; inspect FRPS, provider NAT, and relay lifecycle.
- public TCP connects but SSH banner or Minecraft status times out: NAT edge may be accepting traffic without a responsive backend.
- ICMP works but FRP control is refused: inspect the mapped control port, FRPS listener, firewall, and instance lifecycle.
- status succeeds but players cannot join: inspect mod channels, online mode, UUID handling, login-stage logs, and client versions.
- timeouts cluster during uploads: correlate router queueing, upstream saturation, retransmissions, and bufferbloat.
- available memory is near zero with swap growth: inspect RSS and heap after GC, then preserve OS/native headroom.
- many warnings but few gameplay symptoms: classify warnings by subsystem instead of using the raw warning count as a stability metric.

## Completion Report

Report:

- exact observation window and timezone
- lifecycle state of every compared node
- passive and active results in separate sections
- success/failure definitions and exclusions
- Minecraft restart, timeout, lag, memory, GC, swap, and disk findings
- confidence level for exit attribution
- unresolved gaps, including missing provider history or unavailable logs

Redact player identities, source IPs, credentials, UUIDs, and unnecessary raw log lines.
