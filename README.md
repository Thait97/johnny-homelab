# Johnny's Homelab

A hands-on homelab environment I use to build practical experience with Linux administration, Docker, networking, automation, monitoring, storage, security, and infrastructure troubleshooting.

This repository documents systems I have actually built, configured, tested, broken, repaired, and improved while developing my IT and infrastructure skills.

## Featured Projects

Here are some of the strongest projects and troubleshooting case studies from my homelab:

### Architecture

- [Current Homelab Architecture](architecture/current-homelab.md)
- [Homelab Architecture Diagrams](architecture/diagram.md)
- [Future NAS Architecture](architecture/future-nas.md)

### Automation

- [Ansible Health Check](automation/ansible-health-check.md)
- [Immich Backup Automation](automation/immich-backup.md)
- [Immich Burst Review Automation](automation/immich-burst-review.md)
- [Network Watchdog Automation](automation/network-watchdog.md)

### Networking & Services

- [Docker Networking](networking/docker-networking.md)
- [Tailscale Remote Access](networking/tailscale.md)
- [Pi-hole DNS Filtering](services/pihole.md)
- [Homelab Monitoring Stack](services/monitoring-stack.md)

### Troubleshooting

- [Ansible SSH-Agent Troubleshooting](troubleshooting/ansible-ssh-agent.md)
- [Docker Firewall Troubleshooting](troubleshooting/docker-firewall.md)
- [fwupd Refresh Troubleshooting](troubleshooting/fwupd-refresh.md)
- [Storage Mount Troubleshooting](troubleshooting/storage-mounts.md)
- [Nextcloud Compose Troubleshooting](troubleshooting/nextcloud-compose.md)

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
- Docker
- Docker Compose
- systemd
- Ansible
- Bash
- SSH
- Tailscale
- UFW / iptables
- Samba
- rsync

## Services

My current homelab includes several self-hosted services used for storage, monitoring, administration, and learning.

- **Immich** — self-hosted photo management
- **Nextcloud** — file and cloud services
- **Uptime Kuma** — service availability monitoring
- **Netdata** — system performance monitoring
- **Portainer** — Docker management
- **Cockpit** — Linux server administration
- **Pi-hole** — DNS-level filtering
- **Samba** — SMB file sharing
- **Tailscale** — secure remote network access

## Automation

I am gradually automating routine homelab operations rather than relying entirely on manual administration.

Current projects include:

- Read-only Ansible health checks
- Weekly Immich backups with `systemd` timers and `rsync`
- Automated Immich burst-photo review
- Persistent Docker firewall rules
- Docker network repair automation
- Network health monitoring
- Mac dashboard launcher for homelab services

## Monitoring

I use multiple tools because each provides a different view of the environment.

**Uptime Kuma** monitors whether services are reachable.

**Netdata** provides live host metrics such as CPU, memory, disk, and network activity.

**Portainer** provides visibility into Docker containers and Compose workloads.

**Cockpit** provides host-level Linux administration.

One of the most important lessons from operating the lab has been that:

> A running container does not necessarily mean the application is healthy.

Application endpoints, logs, storage mounts, databases, caches, networking, and dependencies still need to be verified.

## Networking & Security

Current networking and security work includes:

- UFW firewall configuration
- Docker `DOCKER-USER` firewall rules
- Tailscale remote access
- SSH key authentication
- User-defined Docker networks
- Docker DNS/service-name communication
- Pi-hole DNS filtering
- LAN service exposure review
- Port and listening-socket auditing

I also built persistent firewall rules to control traffic reaching Docker-published services because Docker's networking rules can bypass assumptions made when using UFW alone.

## Storage & Backups

Current storage includes separate disks for application data, photos, and backups.

Important mount points include:

```text
/mnt/photos
/mnt/storage
/mnt/immich-backup
```

The Immich photo library is backed up weekly using a custom script, `rsync`, and a `systemd` timer.

The backup process includes:

- mount verification
- available-space checks
- synchronization
- retention handling
- service-health verification

I also verify storage using tools such as:

```bash
lsblk
df -h
```

because the existence of a mount-point directory does not prove that the underlying drive is actually mounted.

## Troubleshooting

A major goal of this repository is documenting troubleshooting methodology rather than only documenting successful configurations.

Examples include:

- diagnosing an Ansible `Permission denied (publickey)` failure
- identifying an unloaded SSH key in `ssh-agent`
- investigating failed `systemd` units
- repairing Docker networking between monitoring services
- understanding Docker firewall behavior
- diagnosing missing storage mounts
- investigating DNS port conflicts
- debugging Docker Compose configuration

For each larger incident, I focus on:

1. Problem
2. Environment
3. Investigation
4. Root cause
5. Fix
6. Verification
7. Lessons learned

## Portfolio Status

The core Version 1 documentation is now in place. Future improvements will focus more on evidence and presentation than adding documentation for its own sake.

Planned improvements include:

- sanitized screenshots of monitoring and administration tools
- command-output examples where they strengthen a case study
- diagrams updated as the rack and NAS move from planning to deployment
- configuration examples with secrets removed
- additional troubleshooting incidents as they happen naturally

## Future Architecture

I am also designing a more modular rack-based homelab.

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

The future storage design is centered around a dedicated NAS using:

- Mini/Tiny compute system
- PCIe HBA in IT mode
- External multi-drive storage
- ZFS
- 2.5 GbE networking

Persistent storage will remain centralized while application compute is separated where practical.

This architecture is still being designed and is documented separately from the systems that are currently deployed.

## What I'm Learning

This homelab is helping me develop practical experience with:

- Linux system administration
- Networking fundamentals
- Docker and container networking
- Infrastructure monitoring
- Troubleshooting methodology
- Storage and backups
- Automation
- Ansible
- Bash
- systemd
- Firewall configuration
- Remote administration
- Infrastructure design

My goal is not simply to deploy services, but to understand how they work, how they fail, and how to troubleshoot them methodically.

---

> This repository contains sanitized documentation from my personal homelab. Credentials, API keys, authentication tokens, public IP addresses, private keys, and other sensitive information are intentionally excluded.
