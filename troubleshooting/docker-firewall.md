# Docker Firewall Troubleshooting Case Study

## Problem

While reviewing firewall behavior on my Ubuntu homelab server, I learned that Docker-published ports do not always behave the way I expected when relying only on UFW.

The server used UFW for host-level firewall rules, but Docker also creates its own iptables rules for container networking.

That meant a service could be published by Docker even when I expected UFW alone to control access.

## Environment

### Host

- Ubuntu Server
- Docker
- Docker Compose
- UFW
- iptables
- Tailscale

### Services

The server hosts several Docker services, including monitoring and self-hosted applications.

Some of these services publish ports to the host.

## Initial Assumption

My initial assumption was:

```text
UFW rule
   |
   v
Controls access to published Docker port
```

That model was incomplete.

Docker manages its own forwarding and NAT rules, so traffic to published container ports may be handled before the normal UFW rules behave the way I expected.

## Investigation

I reviewed the host firewall configuration with:

```bash
sudo ufw status verbose
```

I also checked which services were listening:

```bash
sudo ss -lntup
```

Then I inspected Docker-related firewall behavior through iptables.

```bash
sudo iptables -S DOCKER-USER
```

The important discovery was that Docker has its own firewall processing path.

## Key Concept

Docker creates and manages iptables rules for container networking.

The `DOCKER-USER` chain provides a place for administrator-defined filtering rules before Docker continues processing forwarded traffic.

Conceptually:

```text
Incoming traffic
      |
      v
Linux networking
      |
      v
DOCKER-USER
      |
      v
Docker forwarding rules
      |
      v
Container
```

This made `DOCKER-USER` the appropriate place to enforce additional filtering for Docker-published services.

## Goal

The firewall policy I wanted was:

```text
Allow trusted LAN traffic
Allow Tailscale traffic
Allow established/related traffic
Drop unwanted Docker-forwarded traffic
```

The goal was not to block Docker completely.

The goal was to keep container services reachable from trusted networks while preventing unintended access.

## Implementation

I created a persistent firewall script at:

```text
/usr/local/sbin/homelab-docker-firewall.sh
```

The script manages the required `DOCKER-USER` rules.

A systemd service was also created:

```text
homelab-docker-firewall.service
```

This allows the firewall policy to be restored automatically during startup instead of relying on manual commands after every reboot.

## Why systemd Was Used

A firewall rule that only works until the next reboot is not a complete infrastructure solution.

The systemd service gives the configuration a repeatable lifecycle:

```text
System boot
    |
    v
Docker networking available
    |
    v
homelab-docker-firewall.service
    |
    v
DOCKER-USER rules applied
```

This makes the firewall behavior persistent and easier to verify.

## Verification

I verified the firewall using:

```bash
sudo iptables -S DOCKER-USER
```

I also checked UFW:

```bash
sudo ufw status verbose
```

And reviewed listening sockets:

```bash
sudo ss -lntup
```

The configuration was also reboot-tested to confirm that the custom rules returned automatically.

## Important Lesson: Listening vs Reachable

One of the most useful concepts from this work was the difference between a service listening on an interface and that service actually being reachable.

For example:

```text
127.0.0.1:PORT
```

means the service is bound to the local loopback interface.

It is normally reachable only from the host itself.

By contrast:

```text
0.0.0.0:PORT
```

means the service is listening on all IPv4 interfaces.

However, that still does not automatically mean the service is reachable from the public internet.

Actual reachability also depends on:

- firewall rules
- routing
- NAT
- router port forwarding
- Docker rules
- the network the client is on

## Troubleshooting Model

The incident helped me build a better mental model for network troubleshooting.

```text
Application
    |
Container
    |
Docker port publishing
    |
DOCKER-USER / iptables
    |
Host firewall
    |
Host network interface
    |
Router / network
    |
Client
```

When a service is unexpectedly reachable or unreachable, each layer needs to be checked separately.

## Commands Used

```bash
sudo ufw status verbose

sudo iptables -S DOCKER-USER

sudo ss -lntup
```

Additional Docker checks can include:

```bash
docker ps

docker compose ps
```

These help confirm which ports Docker is publishing.

## What I Learned

### UFW and Docker Are Not the Same Firewall Layer

UFW manages host firewall policy, while Docker creates additional iptables rules for container networking.

Understanding both is important when publishing container ports.

### `DOCKER-USER` Is an Important Control Point

The `DOCKER-USER` chain provides a place to apply administrator-defined filtering before Docker's forwarding rules continue processing traffic.

### Persistent Infrastructure Matters

A manual rule is useful for testing.

A systemd-managed rule that survives reboot is much closer to an actual infrastructure solution.

### Verification Must Include Reboot Testing

The configuration was not considered complete until the rules were verified after a reboot.

That confirmed the solution was persistent rather than temporary.

## Result

The server now uses a persistent Docker firewall policy based on the `DOCKER-USER` chain.

This allows trusted LAN and Tailscale access while providing additional control over traffic forwarded to Docker-published services.

The project helped me understand the interaction between:

- Docker networking
- iptables
- UFW
- systemd
- interface binding
- routing
- service reachability
