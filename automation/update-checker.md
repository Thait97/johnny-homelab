# Ubuntu Update Checker + Notifications

**Skills demonstrated:** Linux | Bash | systemd | Monitoring | Notifications | Safe automation

## Project Overview

I built a read-only update-check workflow for my Ubuntu homelab server so I can see when package updates or reboot attention are needed without automatically changing the server.

The workflow connects Linux automation to monitoring and phone notifications:

```text
systemd timer/service
        |
        v
homelab-update-check.sh
        |
        v
Uptime Kuma
        |
        v
ntfy
        |
        v
iPhone notification
```

## Script

The update-check script is located at:

```text
/usr/local/sbin/homelab-update-check.sh
```

Its role is to inspect update/reboot state and report the result rather than install packages automatically.

## Why Read-Only

For a server, automatically applying every update can create unnecessary risk. I prefer to separate:

```text
Detection
from
Remediation
```

The checker tells me that attention is needed. I can then review the updates and choose when to apply them, especially for changes that could affect networking, SSH, Docker, or require a reboot.

## systemd Design

The workflow uses systemd so the check is repeatable and does not depend on me remembering to run it manually.

```text
systemd timer
     |
     v
systemd service
     |
     v
update-check script
     |
     v
monitoring / notification
```

The timer controls when the check runs, while the service defines the command that runs.

## Notification Path

The result is connected to monitoring and notifications so update attention can reach my phone.

I keep notification URLs, webhook values, and private ntfy topics out of the public repository.

## Safety

This public documentation intentionally does not include:

- secret webhook URLs
- private ntfy topics
- credentials
- tokens
- public WAN addresses

## What I Learned

### Monitoring Can Be Actionable Without Making Changes

A useful automation does not have to modify the system. Read-only checks can reduce risk while still improving operations.

### Detection and Remediation Are Different Responsibilities

The workflow reports when action may be needed, but package installation and reboot decisions remain deliberate administrative actions.

### systemd Makes Small Scripts Operational

A script becomes much more useful when it has a repeatable schedule, logs, and service state that can be inspected with normal Linux tools.

## Separate November 13 Security Update Automation

This existing script is still a **read-only update-awareness workflow**. A separate `unattended-upgrades` configuration now automatically installs eligible Ubuntu security updates from approved origins without automatically rebooting. Third-party infrastructure updates remain manually controlled. The November 13 logs verified a completed unattended `libc6`-related package batch. See [Ubuntu unattended security maintenance](../security/ubuntu-unattended-upgrades-2026-11-13.md).

## Result

The server has a repeatable update-awareness workflow that checks for maintenance needs and routes that status through monitoring and notifications without automatically modifying the host.
