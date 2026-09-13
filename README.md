# AgentDock Rescue Bridge

Out-of-band recovery bridge for the owner's Mac.

## Public endpoints

The rescue bridge uses a fixed Cloudflare named tunnel that is independent of the AgentDock production tunnel:

- Health: `https://rescue.laity.xx.kg/health`
- Status: `https://rescue.laity.xx.kg/status`
- Rescue gate: `https://rescue.laity.xx.kg/rescue`

The rescue endpoint accepts no command, prompt, token, or arbitrary payload. It only asks the Mac-side bridge to evaluate a fixed health contract and, when that contract fails repeatedly, start a fixed local Codex recovery routine.

## Recovery health contract

AgentDock is considered healthy only when all three gates pass together:

1. Local core health: `http://127.0.0.1:8765/healthz` returns HTTP 200 with `ok=true`.
2. The registered macOS launchd job `com.uvwt.agentdock.tunnel` is running.
3. Public production health through Cloudflare, `https://agentdock.laity.xx.kg/healthz`, returns HTTP 200 with `ok=true`.

A local core HTTP 200 by itself is not enough. This prevents a half-recovered state where the core is alive but the external connector still returns 502/530.

## Unattended recovery

- `rescue-watchdog.yml` calls `https://rescue.laity.xx.kg/rescue` every 5 minutes from a GitHub-hosted runner. This is the external backup detector.
- A Mac `launchd` watchdog calls the loopback rescue endpoint every 60 seconds. This is the primary detector and does not depend on GitHub or ChatGPT.
- If the core is unhealthy, local Codex uses the installed AgentDock Desktop lifecycle to recover it.
- If the core is healthy but the production tunnel/public health is unhealthy, Codex restarts only the existing registered tunnel job and leaves the core untouched.
- Recovery is declared successful only after the same three health gates all pass again.
- `keepalive.yml` makes one tiny monthly heartbeat commit so GitHub does not disable scheduled workflows after prolonged repository inactivity.

The Mac bridge also uses a single-run lock and cooldown to prevent overlapping recovery attempts. No replacement AgentDock launch service is created by the rescue system.
