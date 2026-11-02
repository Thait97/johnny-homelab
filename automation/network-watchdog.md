# Network Watchdog Design — In Progress

**Status:** In development; not yet presented as a fully deployed production service.

**Skills being developed:** Linux networking | systemd | Bash | journalctl | Layered troubleshooting

## Project Overview

I am designing a lightweight systemd-based network watchdog to make routine network health checks repeatable.

The goal is to test the network path in layers and record the results in system logs so intermittent failures are easier to investigate.

## Planned Checks

The watchdog is designed around a simple sequence:

```text
Network interface
      |
      v
Home router
      |
      v
Internet reachability
      |
      v
Tailscale
```

This makes it easier to determine where a network failure begins instead of treating every connectivity problem as the same issue.

## Planned systemd Design

```text
systemd timer
      |
      v
systemd service
      |
      v
network-check script
      |
      v
journal logs
```

The timer would control when the check runs. The service would define what is executed, and the script would perform the health checks.

## Intended Frequency

The design uses a short recurring interval, such as every five minutes, so transient failures can be captured without requiring constant manual checking.

## Logging

Results are intended to be written to the system journal with a recognizable tag.

Example inspection workflow:

```bash
journalctl -t homelab-watch
```

## Read-Only Design

The watchdog is intended primarily to observe and report. It should not automatically restart networking, reboot the server, or change configuration simply because one probe fails.

This keeps monitoring separate from remediation and reduces the chance that an automated response makes a temporary problem worse.

## Potential Future Enhancements

- repeated-failure thresholds
- Uptime Kuma integration
- notification hooks
- DNS checks
- gateway latency tracking
- packet-loss tracking
- Tailscale peer checks

These should be added only after the basic watchdog is implemented and tested.

## What I Am Learning

This project is reinforcing:

- systemd services and timers
- layered network troubleshooting
- Linux logging with `journalctl`
- monitoring vs remediation
- safe automation design

## Status

This project remains **in progress**. It is part of the homelab automation roadmap and should not be read as a fully deployed or production-tested service yet.
