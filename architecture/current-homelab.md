# Current Homelab Architecture

This document describes the current state of my homelab and how the main systems, services, storage, monitoring, and remote-access components fit together.

The environment is actively evolving, but this page reflects the systems I currently operate rather than future planned infrastructure.

## Main Server

The primary homelab server runs Ubuntu Server and hosts most of the current workloads.

### Hardware

- Intel Core i5-9100F
- 8 GB RAM
- 120 GB SSD — Ubuntu Server operating system
- 1 TB HDD — Immich photo storage
- 256 GB SSD — additional application/storage use
- 1 TB HDD — Immich backup storage

### Operating System

- Ubuntu Server
- Docker
- Docker Compose
- systemd
- SSH
- UFW

## Storage Layout

Current storage is split across multiple drives based on workload.

```text
120 GB SSD
└── Ubuntu Server OS

1 TB HDD
└── /mnt/photos
    └── Immich photo library

256 GB SSD
└── /mnt/storage
    └── application/storage data

1 TB HDD
└── /mnt/immich-backup
    └── Immich backup
```

Important mount points:

```text
/mnt/photos
/mnt/storage
/mnt/immich-backup
```

I verify that storage is actually mounted using:

```bash
lsblk
df -h
```

This is important because the existence of a mount-point directory does not necessarily mean that the underlying drive is mounted.

## Docker Workloads

Several self-hosted applications run using Docker and Docker Compose.

Current services include:

- Immich **v3.4.1** (latest recorded November 13, 2026)
- Uptime Kuma
- Netdata
- Portainer
- Pi-hole

Docker Compose is used to manage application stacks and their supporting components.

Common administrative commands include:

```bash
docker ps
docker compose ps
docker compose pull
docker compose up -d
docker compose logs
```

One lesson I have learned is that a running container does not automatically mean the application itself is healthy.

I also verify:

- application web interfaces
- logs
- storage mounts
- databases
- caches
- network connectivity
- dependent services

## Immich

Immich is one of the main applications in the environment.

It provides self-hosted photo management and stores its main photo data on the dedicated photo drive.

```text
/mnt/photos/Immich
```

I have worked with:

- Docker Compose updates
- database backups
- storage verification
- container health checks
- application verification
- photo-loading verification
- custom automation

Immich is also protected by a separate backup process using another physical drive.

The [November 10 upgrade case study](../troubleshooting/immich-3.4.0-ghcr-ipv6-2026-11-10.md) documents resolution of an IPv6-related GHCR image-pull failure; IPv6 was restored after upgrading and four Immich containers were healthy at verification.

## Immich Backup

The Immich photo library is backed up to:

```text
/mnt/immich-backup/Immich
```

The backup process uses:

- a custom shell script
- `rsync`
- a `systemd` service
- a `systemd` timer

The current schedule is weekly.

The backup design includes:

- mount checks
- free-space checks
- synchronization
- retention handling
- service verification

I manually tested the backup process before relying on the scheduled job. The [November 10 verification](../automation/immich-backup-verification-2026-11-10.md) additionally confirmed a successful November 8 scheduled backup, matching sampled PostgreSQL backup hashes, and a successful manual run. A full restoration test remains outstanding.

## November 11–13 Backup and Patch Safeguards

The Sunday full-library backup is supplemented by a new daily database replication timer at 03:30. The replication uses UUID/mount validation, a shared maintenance lock, gzip/source-copy checks and verified temporary-file renaming. Manual service execution succeeded; the first scheduled run after November 13 has not yet been observed. See [DB replication](../automation/immich-database-replication.md).

The Wednesday 04:00 Immich patch updater is a **preparation-only** service: it checks eligible stable patch releases and backup prerequisites, then prepares images for later manual deployment. Dry-run/service tests passed but the first real scheduled image-preparation run remains to be observed. See [patch preparation](../automation/immich-patch-preparation.md). **Immich was still v3.4.1 after the reboot**.

## Previous Service — Nextcloud (Removed November 6, 2026)

I previously operated Nextcloud through a Docker Compose stack with MariaDB, Redis, and cron. I removed Nextcloud from the active Ubuntu deployment on November 6, 2026 while reorganizing the homelab around dedicated storage and compute systems.

The earlier [Nextcloud Compose troubleshooting case study](../troubleshooting/nextcloud-compose.md) is retained as historical learning evidence. Nextcloud is **not currently running** on this Ubuntu server.

## Monitoring

I use several monitoring and administration tools because each provides a different view of the system.

### Uptime Kuma

Used to monitor service availability and determine whether applications are reachable.

### Netdata

Used for live monitoring of:

- CPU
- memory
- disk activity
- network activity
- system performance

### Portainer

Used for Docker visibility and container management.

### Cockpit

Used for Linux host administration and system visibility.

Together, these tools provide both application-level and host-level monitoring.

## Docker Networking

I use a user-defined Docker network for monitoring-related services.

The network allows services such as Uptime Kuma and Portainer to communicate using Docker DNS names rather than depending on changing container IP addresses.

For example:

```text
uptime-kuma
    |
    └── portainer:9443
```

This helped reinforce the difference between container IP addresses and Docker service-name resolution.

## Firewall and Security

The server uses UFW for host-level firewall management.

I also learned that Docker-published ports require additional consideration because Docker creates its own iptables rules.

A persistent firewall solution was added using the:

```text
DOCKER-USER
```

chain.

The general policy is:

```text
Allow trusted LAN traffic
Allow Tailscale traffic
Allow established/related traffic
Drop unwanted forwarded Docker traffic
```

I verify firewall and listening behavior with commands such as:

```bash
sudo ufw status verbose
sudo iptables -S DOCKER-USER
sudo ss -lntup
```

A [read-only security audit on November 10](../security/audit-2026-11-10.md) inspected UFW, IPv4/IPv6 DOCKER-USER chains, SSH access policy, and container privileges. This was a local configuration assessment, not proof of complete external security.

## Remote Access

Tailscale provides remote network connectivity to the homelab.

I use Tailscale together with SSH for remote administration.

These are separate layers:

```text
Tailscale = secure network path
SSH       = remote shell
```

This separation has been important when troubleshooting remote-access problems.

## Ansible

My Mac acts as an Ansible controller for the homelab.

I currently use Ansible for read-only server health checks.

A healthy read-only run should return results similar to:

```text
changed=0
unreachable=0
failed=0
```

For a health-check playbook, `changed=0` is desirable because it means the playbook inspected the system without modifying it.

## November 13, 2026 — Controlled Reboot Verification

Following staged Docker/containerd and Tailscale upgrades and a `libc6` reboot-required marker, a controlled reboot succeeded. SSH remained accessible via enabled `ssh.socket`, even though `ssh.service` was disabled. Docker, containerd, and Tailscale recovered. All nine containers ran; **eight reported healthy while Portainer was running without a health check**. All five maintenance timers remained active. [Maintenance report](../maintenance/ubuntu-maintenance-2026-11-13.md).

## November 6, 2026 — Docker Verification

After reconnecting the Ubuntu server to Ethernet, an initial check found the two OpenGym containers still starting. A subsequent final check confirmed **all nine Docker containers healthy**, including both OpenGym containers and the Immich, Pi-hole, Uptime Kuma, and Portainer components. This is a point-in-time health-status result, not a claim that every application passed an independent end-to-end test.

## Current Architecture

```text
                     Internet
                        |
                     Router
                        |
                  Local Network
                        |
                  Ubuntu Server
                        |
        +---------------+---------------+
        |               |               |
      Docker          Storage        Monitoring
        |               |               |
   +----+----+      +----+----+      +---+---+
   |         |      |         |      |       |
 Immich   Pi-hole   Photos   Backup  Kuma  Netdata
   |                   |       |      |
   |                   |       |   Portainer
   |                   |       |
   +-------------------+-------+
               Local data

Remote administration:
Mac / iPhone
     |
  Tailscale
     |
    SSH
     |
Ubuntu Server
```

## What This Environment Has Taught Me

Operating this homelab has helped me develop practical experience with:

- Linux administration
- Docker and Docker Compose
- container networking
- storage and mount management
- backup automation
- systemd services and timers
- SSH
- Ansible
- monitoring
- firewall configuration
- remote access
- troubleshooting methodology

The biggest lesson has been that maintaining infrastructure requires more than getting a service to start.

A healthy system requires verifying the entire path:

```text
Host
→ Storage
→ Network
→ Container
→ Application
→ Dependencies
→ User access
```

That troubleshooting mindset is one of the main skills I am developing through this homelab.
