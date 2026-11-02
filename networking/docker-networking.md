# Docker Networking

## Project Overview

I use Docker and Docker Compose for several homelab services. One of the most useful networking lessons came from connecting monitoring services reliably without depending on changing container IP addresses.

I created a user-defined Docker network named:

```text
monitoring
```

and connected services such as:

```text
uptime-kuma
portainer
```

This allows Uptime Kuma to reach Portainer using Docker DNS:

```text
portainer:9443
```

instead of using a container IP that may change after restarts.

## Problem

Using container IP addresses directly is fragile.

A container might have an address such as:

```text
172.x.x.x
```

but Docker can assign a different address after a container is recreated or restarted.

If a monitoring target is configured with that temporary IP, the monitor can fail even though the application itself is healthy.

## Better Approach

A user-defined Docker network provides built-in DNS resolution between attached containers.

Conceptually:

```text
Uptime Kuma
     |
     | Docker DNS lookup
     v
portainer
     |
     v
Port 9443
```

The monitoring target can therefore use:

```text
portainer:9443
```

rather than a hard-coded container IP.

## Why User-Defined Networks Matter

A user-defined bridge network gives containers a predictable logical relationship even when their individual IP addresses change.

This is useful because application configuration can reference service names instead of infrastructure details that Docker manages dynamically.

## Troubleshooting Scenario

When Uptime Kuma could no longer resolve or reach Portainer by hostname, the first place to check was the shared Docker network.

The troubleshooting question became:

```text
Are both containers still attached to the monitoring network?
```

rather than immediately changing firewall rules or application configuration.

## Useful Checks

General container status:

```bash
docker ps
```

Compose stack status:

```bash
docker compose ps
```

Available Docker networks:

```bash
docker network ls
```

Inspect a specific network:

```bash
docker network inspect monitoring
```

That inspection can show which containers are attached to the network.

## Persistent Repair

I also created a persistent repair mechanism to reconnect the monitoring relationship after startup if needed.

This reinforced an important infrastructure lesson:

> A manual fix is useful for diagnosis, but a production-style solution should survive restart and reboot.

Conceptually:

```text
System startup
     |
     v
Docker starts
     |
     v
Containers start
     |
     v
Monitoring network relationship verified/repaired
     |
     v
Uptime Kuma can resolve Portainer by name
```

## Docker DNS vs Container IP

The key distinction is:

```text
Container IP
= implementation detail that may change

Docker service/container name on a shared user-defined network
= stable logical target for service-to-service communication
```

This made the monitoring configuration more reliable and easier to understand.

## Relationship to Host Networking

Docker networking is only one layer of connectivity.

A service may still depend on:

- host interface binding
- published ports
- UFW
- `DOCKER-USER` iptables rules
- LAN routing
- Tailscale
- application-level health

That means a network failure should be isolated layer by layer rather than assuming every connectivity issue is caused by Docker itself.

## Troubleshooting Model

```text
Application
    |
Container
    |
Docker network / DNS
    |
Published port if needed
    |
Host firewall
    |
Host interface
    |
LAN / Tailscale / client
```

## What I Learned

### Do Not Depend on Dynamic Container IPs

Container IP addresses can change, especially after recreation.

### Docker Provides Internal DNS

Containers on the same user-defined network can communicate using names, which is more stable than hard-coded IP addresses.

### Check the Simplest Shared Dependency First

If one monitor fails while the rest of the server is healthy, the problem may be the specific container-to-container path rather than the entire host or network.

### Persistence Matters

A repair that has to be manually repeated after every startup is not a complete infrastructure solution.

## Result

The monitoring services now use a shared Docker network so Uptime Kuma can reach Portainer by name instead of depending on changing container IP addresses.

This project gave me practical experience with Docker bridge networks, internal DNS, service-to-service communication, startup persistence, and layered network troubleshooting.
