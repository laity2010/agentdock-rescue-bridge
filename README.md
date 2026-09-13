# AgentDock Rescue Bridge

Out-of-band recovery bridge for the owner's Mac.

Public health endpoint (does not trigger repair and contains no credential):

https://vast-broken-indie-never.trycloudflare.com/health

The rescue endpoint is separately protected by a rotating one-time token and is intentionally not published here.

## Unattended recovery

This repository contains the cloud-side watchdog for the Mac AgentDock rescue bridge.

- `rescue-watchdog.yml` calls `https://rescue.laity.xx.kg/rescue` every 5 minutes. The Mac bridge performs the actual local health decision; healthy AgentDock returns a no-op, while repeated local health failures start the fixed local Codex recovery routine.
- `keepalive.yml` makes one tiny monthly heartbeat commit so GitHub does not disable scheduled workflows after prolonged repository inactivity.
- The Mac also has an independent local `launchd` watchdog that calls the loopback rescue endpoint every 60 seconds. The GitHub workflow is a second fault domain, not the primary detector.

The public rescue endpoint accepts no arbitrary command or prompt. Recovery behavior is fixed on the Mac and guarded by health checks, a single-run lock, and a cooldown.
