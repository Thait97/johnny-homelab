# Johnny's Homelab

A hands-on homelab I use to build practical experience with Linux administration, Docker, networking, automation, monitoring, storage, security, and infrastructure troubleshooting.

This repository documents systems I have **built and operated**, along with clearly labeled **future architecture** I am designing as the lab grows.

The main portfolio story is not simply that I installed self-hosted services. It is that I built infrastructure, encountered real failures, isolated the affected layer, fixed the issue, made solutions persistent where appropriate, and verified the result.

## Featured Troubleshooting

1. [Immich External-Library Re-Indexing — Validation Pending](troubleshooting/immich-external-library-reindexing.md) — PostgreSQL asset IDs, Docker mounts, scheduled scans, automation isolation
2. [Docker Firewall / DOCKER-USER](troubleshooting/docker-firewall.md) — Docker, UFW, iptables, systemd, reboot verification
3. [Ansible SSH-Agent Failure](troubleshooting/ansible-ssh-agent.md) — SSH authentication, `ssh-agent`, Ansible verification
4. [fwupd systemd Troubleshooting](troubleshooting/fwupd-refresh.md) — service status, logs, direct command testing, verification
5. [Storage Mount Troubleshooting](troubleshooting/storage-mounts.md) — `lsblk`, `df`, mount-state reasoning
6. [Nextcloud Compose Troubleshooting](troubleshooting/nextcloud-compose.md) — Docker Compose/YAML debugging
7. [Pi-hole Port 53 Troubleshooting](troubleshooting/pihole-port-53.md) — DNS listeners, socket binding, `ss`

## Featured Automation

1. [Immich Backup Automation](automation/immich-backup.md) — weekly rsync-based backup/synchronization, retention, systemd
2. [Ansible Health Check](automation/ansible-health-check.md) — read-only infrastructure checks over SSH
3. [Ubuntu Update Checker + Notifications](automation/update-checker.md) — systemd, monitoring, ntfy/iPhone notification path
4. [Immich Burst Review Automation](automation/immich-burst-review.md) — Bash, SQL, PostgreSQL, Python, REST API, state tracking
5. [Docker Monitoring Network Repair](networking/docker-networking.md) — Docker DNS, user-defined networks, persistent systemd repair
6. [Mac Homelab Dashboard Launcher](automation/mac-dashboard.md) — shell scripting, symlinks, PATH, workflow automation

## Current Architecture & Services

- [Current Homelab Architecture](architecture/current-homelab.md)
- [Homelab Architecture Diagrams](architecture/diagram.md)
- [Homelab Monitoring Stack](services/monitoring-stack.md)
- [Pi-hole DNS Filtering](services/pihole.md)
- [Tailscale Remote Access](networking/tailscale.md)

## Future / In-Progress Architecture

- [Lenovo M720q NAS Controller — TrueNAS Installed; DAS Pending](projects/lenovo-m720q-nas/README.md) — [November 6 installation milestone](projects/lenovo-m720q-nas/progress-2026-11-06.md); no data pool or external storage
- [Future NAS Architecture](architecture/future-nas.md) — TrueNAS controller configured for initial LAN access; **NAS storage expansion and production migration not deployed**
- [Network Watchdog Design — In Progress](automation/network-watchdog.md) — **in development, not yet presented as fully deployed**
- Future Immich/Jellyfin compute and 2.5 GbE rack design remain part of the planned architecture

## Current Environment

### Main Server

- Ubuntu Server
- Intel Core i5-9100F
- 8 GB RAM
- 120 GB SSD — operating system
- 1 TB HDD — Immich photo storage
- 256 GB SSD — additional application/storage use
- 1 TB HDD — Immich backup

### Core Technologies

- Linux / Ubuntu Server
- Docker / Docker Compose
- systemd
- Ansible
- Bash
- SSH
- Tailscale
- UFW / iptables
- Samba
- rsync

## Current Services

- **Immich** — self-hosted photo management
- **Uptime Kuma** — service availability monitoring
- **Netdata** — system performance monitoring
- **Portainer** — Docker management
- **Cockpit** — Linux host administration
- **Pi-hole** — DNS-level filtering
- **Samba** — SMB file sharing
- **Tailscale** — remote network access

## Previously Deployed Services

- **Nextcloud** — previously operated as a Docker Compose stack, removed from the Ubuntu homelab on November 6, 2026 during the transition toward dedicated storage and compute nodes. The [Nextcloud Compose troubleshooting case study](troubleshooting/nextcloud-compose.md) remains as historical project evidence.

## Current Automation

- Read-only Ansible health checks
- Weekly rsync-based Immich backup/synchronization with `systemd` scheduling and retention
- Immich burst-photo review workflow
- Persistent Docker firewall rules
- Persistent Docker monitoring-network repair
- Ubuntu update/reboot attention checks with monitoring/notification integration
- Service and network health monitoring
- Mac dashboard launcher

The custom **Network Watchdog** is documented separately as **In Progress** rather than as a completed deployment.

## November 6, 2026 — Final Docker Health Check

After restoring Ethernet connectivity to the Ubuntu server, a later verification confirmed **all 9 Docker containers healthy**, including Immich components, Pi-hole, Uptime Kuma, Portainer, and both OpenGym containers. This replaces the earlier intermediate observation that the OpenGym containers were still in `health: starting`.

See the [November 6 NAS and Ubuntu verification report](projects/lenovo-m720q-nas/progress-2026-11-06.md).

## Monitoring Philosophy

I use multiple tools because each answers a different question:

- **Uptime Kuma:** Is the service reachable?
- **Netdata:** What is the host doing right now?
- **Portainer:** What is happening inside Docker?
- **Cockpit:** What is happening on the Linux host?

A running container does not necessarily mean the application inside it is healthy. Logs, storage, databases, networking, dependencies, and the user-facing endpoint may still need verification.

## Storage & Backups

Important current mount points include:

```text
/mnt/photos
/mnt/storage
/mnt/immich-backup
```

I verify storage with tools such as:

```bash
lsblk
df -h
```

because a mount-point directory existing does not prove the intended disk is actually mounted.

The Immich workflow currently runs Sunday at 03:00 with 30-day deleted-file retention. Longer retention, optional daily copy-only synchronization, and periodic restore testing are documented as **future improvements**, not current deployment.

## Troubleshooting Approach

For larger incidents I document:

1. Problem
2. Environment
3. Investigation
4. Root cause
5. Fix
6. Verification
7. Lessons learned

Examples of sanitized proof already included in case studies include:

```text
ok=12
changed=0
unreachable=0
failed=0
```

and:

```text
0 loaded units listed.
```

The next presentation improvement is adding selected sanitized screenshots/output where they provide stronger evidence than prose alone.

## Future Architecture

The future storage/rack design is clearly separated from the current lab.

Current direction includes:

```text
Router
   |
2.5 GbE Switch
   |
   +-- NAS
   +-- Immich / Jellyfin Compute
   +-- Network Infrastructure
   +-- Monitoring / Management
```

The purchased Lenovo M720q now boots **TrueNAS SCALE 25.10.x** from a verified SanDisk SATA SSD and was accessible through its web interface over Ethernet on November 6. **No data pool, SAS HBA, external drives, or NAS shares exist yet.** The current preferred *research* design uses a shelf-mounted **five-bay hot-swap cage** with optional later expansion. See the [NAS controller project](projects/lenovo-m720q-nas/README.md) and [November 6 progress report](projects/lenovo-m720q-nas/progress-2026-11-06.md). Purchases for the remaining hardware remain on hold.

## What I'm Learning

This homelab is helping me develop practical experience with:

- Linux system administration
- SSH and remote administration
- Docker and container networking
- systemd services and timers
- Ansible
- Bash automation
- monitoring and logs
- storage and mounts
- backup design
- firewall configuration
- DNS
- layered troubleshooting
- infrastructure design
- verifying changes instead of assuming they worked

My goal is not to present myself as a senior systems engineer. The goal is to show credible hands-on junior infrastructure/IT experience and a methodical approach to learning and troubleshooting.

---

> This repository contains sanitized documentation from my personal homelab. Credentials, private keys, API keys, tokens, secret webhook URLs, private ntfy topics, Wi-Fi/router credentials, public WAN addresses, and secret-bearing `.env` files are intentionally excluded.
