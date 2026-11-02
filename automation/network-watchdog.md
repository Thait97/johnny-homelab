# Network Watchdog Automation

## Project Overview

I designed a lightweight systemd-based network watchdog to make routine network health checks repeatable.

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

## systemd Design

The automation uses the same separation of responsibilities I use for other Linux automation:

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

The timer controls when the check runs.

The service defines what is executed.

The script performs the actual health checks.

## Frequency

The design uses a short recurring interval, such as every five minutes, so transient failures can be captured without requiring constant manual checking.

## Logging

Results are intended to be written to the system journal with a recognizable tag.

Example inspection workflow:

```bash
journalctl -t homelab-watch
```

Using the journal means the watchdog fits into the normal Linux administration workflow instead of creating an isolated logging system.

## Why Layered Checks Matter

A failed internet test does not automatically mean the network interface is down.

For example:

```text
Interface works
Router responds
Internet fails
Tailscale unavailable
```

suggests a different problem than:

```text
Interface fails
Router unreachable
Internet unreachable
Tailscale unreachable
```

The sequence helps narrow the failure domain.

## Read-Only Design

The watchdog is primarily intended to observe and report.

It should not automatically restart networking, reboot the server, or make configuration changes simply because one probe fails.

This keeps monitoring separate from remediation and reduces the risk of an automated recovery action making a temporary problem worse.

## Potential Future Enhancements

Possible additions include:

- repeated-failure thresholds
- Uptime Kuma integration
- notification hooks
- DNS checks
- gateway latency tracking
- packet-loss tracking
- Tailscale peer checks

These should be added only when the basic watchdog is stable and useful.

## What I Learned

This project reinforces:

- systemd services and timers
- layered network troubleshooting
- Linux logging with `journalctl`
- the difference between monitoring and remediation
- why automation should fail safely
- how recurring checks can create useful troubleshooting history

## Status

This project is part of the homelab automation roadmap and is being developed incrementally rather than deployed as an aggressive auto-repair system.
