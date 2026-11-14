# Ubuntu Unattended Security Updates — November 11–13, 2026

**Status:** Configured, enabled, and verified by a successful unattended package-installation log on November 13.

## Objective

Enable automatic Ubuntu security maintenance while retaining manual review of third-party infrastructure upgrades and all reboots.

## Implementation

Configured daily package-list refreshes and installation of eligible Ubuntu security updates from approved Ubuntu/security origins. Docker, Netdata, and Tailscale remain under manual maintenance.

Verified settings:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
Unattended-Upgrade::Automatic-Reboot "false";
```

The `apt-daily.timer` and `apt-daily-upgrade.timer` units were verified. **Automatic reboot is disabled.**

## Observed automatic update

On November 13, the unattended-upgrades log showed successful automatic installation of:

- `libc-bin`, `libc-dev-bin`, `libc-gconv-modules-extra`
- `libc6`, `libc6-dev`, `locales`

The log concluded with:

```text
All upgrades installed
```

A later reboot-required marker related to `libc6` was handled with a separately reviewed, controlled reboot. Automatic package installation does **not** imply that running processes have picked up every update without a restart.

## Scope and lessons

This demonstrates unattended Ubuntu security maintenance operating successfully at least once. Future scheduled updates still need monitoring and logs should be checked for errors. The read-only [update checker](../automation/update-checker.md) continues to be a separate detection/notification workflow; it is not itself the unattended installer.

**Skills:** Ubuntu APT, repository-origin restrictions, unattended-upgrades, systemd timers, logs, reboot-aware change management.
