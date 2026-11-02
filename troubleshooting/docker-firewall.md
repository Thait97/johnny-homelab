# Docker Firewall Troubleshooting Case Study

**Skills demonstrated:** Linux networking | Docker | iptables | UFW | systemd | Security | Verification

## Problem

While reviewing firewall behavior on my Ubuntu homelab server, I learned that Docker-published ports do not always behave the way I expected when relying only on UFW.

Docker creates its own iptables rules for container networking, so host firewall assumptions need to be verified rather than trusted blindly.

## Environment

- Ubuntu Server
- Docker / Docker Compose
- UFW
- iptables
- Tailscale

## Investigation

I reviewed UFW:

```bash
sudo ufw status verbose
```

Checked listening sockets:

```bash
sudo ss -lntup
```

And inspected Docker's user-controlled filtering chain:

```bash
sudo iptables -S DOCKER-USER
```

## Key Concept

Docker manages forwarding/NAT rules for published container ports. The `DOCKER-USER` chain provides a place for administrator-defined filtering before Docker continues processing forwarded traffic.

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
Docker forwarding
      |
      v
Container
```

## Goal

```text
Allow trusted LAN traffic
Allow Tailscale traffic
Allow established/related traffic
Drop unwanted Docker-forwarded traffic
```

## Persistent Implementation

Script:

```text
/usr/local/sbin/homelab-docker-firewall.sh
```

systemd service:

```text
homelab-docker-firewall.service
```

This makes the policy persistent instead of requiring manual rule entry after reboot.

## Verification

I verified the configuration with:

```bash
sudo iptables -S DOCKER-USER
sudo ufw status verbose
sudo ss -lntup
```

The configuration was also reboot-tested to confirm that the custom rules returned automatically.

## Listening vs Reachable

A major lesson from this project was that:

```text
0.0.0.0:PORT
```

means a service is listening on all IPv4 interfaces, but **does not automatically mean it is exposed to the public internet**.

Actual reachability also depends on:

- firewall rules
- routing
- NAT
- router port forwarding
- Docker rules
- client network location

Likewise:

```text
127.0.0.1:PORT
```

is loopback-only and is normally reachable only from the host itself.

## What I Learned

- UFW and Docker operate across different firewall/networking layers.
- `DOCKER-USER` is an important administrator control point for Docker-forwarded traffic.
- Manual rules are useful for testing, but persistence matters.
- Reboot verification is part of proving that an infrastructure fix is complete.
- Listening on `0.0.0.0` is not the same thing as public internet exposure.

## Result

The server now uses a persistent Docker firewall policy based on `DOCKER-USER`, allowing trusted LAN/Tailscale access while adding control over forwarded traffic to Docker-published services.
