# Tailscale Remote Access

## Project Overview

Tailscale provides secure remote network connectivity to my homelab.

One of the most useful concepts I learned from this setup is that remote networking and remote shell access are separate layers.

```text
Tailscale = network path
SSH       = remote shell
```

Tailscale makes the server reachable across its private overlay network, while SSH is the service I use to administer the Linux host.

## Current Role

The Ubuntu server has both a local LAN address and a Tailscale address.

This gives me two different network paths depending on where I am connecting from.

```text
Home
Mac / iPhone
    |
    v
Local LAN
    |
    v
Ubuntu Server

Away from home
Mac / iPhone
    |
    v
Tailscale
    |
    v
Ubuntu Server
```

## Interfaces

Relevant interfaces include:

```text
enp2s0     = wired LAN interface
tailscale0 = Tailscale interface
```

Understanding the interface involved in a connection is useful when troubleshooting firewall rules and service reachability.

## Troubleshooting Model

If remote SSH fails, I do not assume SSH itself is broken.

I separate the problem into layers:

```text
Can the device reach Tailscale?
        |
        v
Can it reach the server over Tailscale?
        |
        v
Is SSH listening?
        |
        v
Does authentication work?
```

This distinction prevents network problems from being mistaken for SSH authentication problems.

## Firewall Integration

My Docker firewall policy explicitly accounts for trusted Tailscale traffic.

That is part of a broader policy that allows trusted LAN and Tailscale access while restricting unwanted forwarded Docker traffic.

## Remote Administration

Tailscale is used with tools such as:

- SSH
- homelab dashboards
- service web interfaces
- remote health checks

The overlay network lets me access internal services without exposing them directly through public router port forwarding.

## Security Principle

I avoid treating remote access as a reason to publish services broadly to the internet.

The preferred model is:

```text
Remote device
     |
     v
Tailscale private network
     |
     v
Internal homelab service
```

rather than exposing administrative services through public inbound ports.

## Future Direction

A future Raspberry Pi infrastructure node may also run Tailscale and potentially provide subnet-routing capability for selected internal networks.

That design will be evaluated separately from the current server-based remote-access setup.

## What I Learned

This project reinforced:

- overlay networking
- interface awareness
- network path vs application protocol
- remote administration
- firewall integration
- why private remote access can reduce the need for public port forwarding
- layered troubleshooting

## Result

Tailscale now provides a practical remote-access layer for my homelab while SSH and other services remain separate application-level tools on top of that network path.
