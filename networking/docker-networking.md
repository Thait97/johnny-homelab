# Docker Networking and Persistent Monitoring Network Repair

**Skills demonstrated:** Docker | Docker Compose | Container networking | DNS | systemd | Troubleshooting

## Project Overview

I use Docker and Docker Compose for several homelab services. One of the most useful networking lessons came from connecting monitoring services reliably without depending on changing container IP addresses.

I created a user-defined Docker network named:

```text
monitoring
```

and connected:

```text
uptime-kuma
portainer
```

Uptime Kuma can then monitor Portainer using Docker DNS:

```text
portainer:9443
```

instead of a container IP that may change after a restart or recreation.

## Problem

Using container IP addresses directly is fragile. Docker may assign a different address after a container is recreated, so a monitor pointed at a hard-coded `172.x.x.x` address can fail even though the application is healthy.

## Better Approach

A user-defined Docker network provides DNS resolution between attached containers.

```text
Uptime Kuma
     |
     | Docker DNS
     v
portainer:9443
```

This gives the monitor a stable logical target.

## Troubleshooting Scenario

When Uptime Kuma could no longer resolve Portainer by hostname, I checked the shared network before changing firewalls or application settings.

Useful checks included:

```bash
docker ps
docker compose ps
docker network ls
docker network inspect monitoring
```

The main question was:

```text
Are both containers still attached to the monitoring network?
```

## Persistent Repair

I turned the repair into a persistent startup solution rather than leaving it as a manual command.

Script:

```text
/usr/local/sbin/homelab-monitoring-network.sh
```

systemd service:

```text
homelab-monitoring-network.service
```

Conceptually:

```text
System startup
     |
     v
Docker / containers start
     |
     v
homelab-monitoring-network.service
     |
     v
monitoring network relationship verified/repaired
     |
     v
Uptime Kuma resolves Portainer by name
```

This reinforced an important infrastructure lesson:

> A manual fix is useful for diagnosis, but a stronger solution should survive restart and reboot.

## Docker DNS vs Container IP

```text
Container IP
= implementation detail that may change

Container/service name on a shared user-defined network
= stable logical target
```

## Relationship to Host Networking

Docker networking is only one connectivity layer. A service may still depend on:

- host interface binding
- published ports
- UFW
- `DOCKER-USER` iptables rules
- LAN routing
- Tailscale
- application health

So I troubleshoot network failures layer by layer rather than assuming Docker is always the problem.

## What I Learned

- Do not depend on dynamic container IPs.
- User-defined Docker networks provide useful internal DNS.
- Check the smallest shared dependency first.
- Persistent systemd-managed repairs are more reliable than repeated manual fixes.

## Result

The monitoring services use a shared Docker network so Uptime Kuma can reach Portainer by name, and a persistent systemd-managed repair mechanism helps restore that relationship after startup when needed.
