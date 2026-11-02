# Homelab Monitoring Stack

## Project Overview

I use several monitoring and administration tools in my homelab because each one answers a different question.

The goal is not to install multiple dashboards for the sake of having more software.

The goal is to separate:

- service availability
- host performance
- container visibility
- Linux system administration

The main tools are:

- Uptime Kuma
- Netdata
- Portainer
- Cockpit

## Monitoring Philosophy

One of the most important lessons from operating the homelab is:

> A running container does not automatically mean the application is healthy.

A complete health check may require verifying:

```text
Host
→ Storage
→ Network
→ Container
→ Application
→ Dependencies
→ User access
```

That is why I use multiple tools rather than relying on a single dashboard.

## Uptime Kuma

Uptime Kuma is used for service availability monitoring.

Its main job is answering:

```text
Is this service reachable?
```

Examples include checking whether services such as:

- Immich
- Portainer
- Cockpit
- Pi-hole
- other internal services

are available.

Uptime Kuma is useful for quickly identifying when one monitored service becomes unreachable while the rest of the environment remains healthy.

## Netdata

Netdata provides live system performance monitoring.

I use it to inspect:

- CPU usage
- memory usage
- disk activity
- storage utilization
- network activity
- system load

Its main job is answering:

```text
What is the host doing right now?
```

For example, if a service feels slow, Netdata can help determine whether CPU, memory, disk, or network activity is unusually high.

## Portainer

Portainer provides visibility into Docker.

I use it to inspect:

- containers
- container status
- images
- networks
- Docker resources
- Compose-managed workloads

Its main job is answering:

```text
What is happening inside Docker?
```

Portainer is useful when the Linux host is healthy but a containerized service has a problem.

## Cockpit

Cockpit provides Linux host administration and system visibility.

It helps with host-level tasks such as:

- system status
- services
- storage
- logs
- resource usage
- general server administration

Its main job is answering:

```text
What is happening on the Linux host?
```

## How the Tools Fit Together

Each tool provides a different layer of visibility.

```text
                  Homelab Monitoring

                        |
        +---------------+---------------+
        |               |               |
        v               v               v
   Availability       Host            Docker
        |            Metrics            |
        v               |               v
  Uptime Kuma       Netdata         Portainer
                        |
                        v
                 Linux Administration
                        |
                        v
                     Cockpit
```

Another way to think about it is:

```text
Uptime Kuma
"Can I reach it?"

Netdata
"Is the system under load?"

Portainer
"Is the container running?"

Cockpit
"What is happening on the host?"
```

## Example Troubleshooting Workflow

If Immich becomes unavailable, I can troubleshoot in layers.

### Step 1 — Uptime Kuma

Check whether the service is actually unreachable.

If Uptime Kuma reports Immich down, continue investigating.

### Step 2 — Portainer or Docker

Check whether the container is running.

```bash
docker ps
```

or:

```bash
docker compose ps
```

A running container does not end the investigation.

### Step 3 — Application Logs

Check the application logs.

```bash
docker compose logs
```

This can reveal application-level failures even when the container itself is running.

### Step 4 — Netdata

Check system resources.

Possible issues include:

- high CPU usage
- low available memory
- heavy disk activity
- abnormal network activity

### Step 5 — Cockpit / Linux Host

Inspect the Linux host, services, storage, or system logs.

### Step 6 — Storage

Verify required storage is mounted.

```bash
lsblk
df -h
```

This is especially important for services that depend on external or separate storage drives.

## Docker Monitoring Network

I also created a user-defined Docker network called:

```text
monitoring
```

This network is used by:

```text
uptime-kuma
portainer
```

It allows Uptime Kuma to reach Portainer using Docker DNS:

```text
portainer:9443
```

instead of depending on changing container IP addresses.

This was a useful lesson in Docker networking and service-name
