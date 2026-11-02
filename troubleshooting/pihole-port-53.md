# Pi-hole Port 53 Troubleshooting

**Skills demonstrated:** Linux networking | DNS | Pi-hole | Socket inspection | Troubleshooting

## Problem

While working with Pi-hole DNS, I investigated a port 53 conflict caused by another resolver/service already listening on DNS-related sockets.

A key error pattern for this type of issue is:

```text
address already in use
```

The correct response is not to randomly stop services. The first step is to identify which process is already listening.

## Investigation

I used:

```bash
sudo ss -lntup
```

to inspect listening TCP/UDP sockets and determine what was bound to DNS port 53.

## Binding Matters

The address a service binds to changes where it can receive traffic.

For example:

```text
0.0.0.0:53
```

means the service is listening on port 53 across all IPv4 interfaces.

A binding such as:

```text
192.168.1.x:53
```

is different because it targets a specific LAN interface/address.

Likewise, loopback-only bindings are not equivalent to LAN-facing DNS service.

## Troubleshooting Model

```text
Pi-hole cannot bind to port 53
          |
          v
Inspect listeners with ss
          |
          v
Identify existing resolver/service
          |
          v
Compare interface/address bindings
          |
          v
Adjust configuration deliberately
          |
          v
Verify DNS service
```

## What I Learned

### “Address Already in Use” Is a Clue

It means another socket already owns the requested address/port combination. Identify that listener before changing anything.

### Port Number Alone Is Not Enough

`0.0.0.0:53`, `127.0.0.1:53`, and `192.168.1.x:53` describe different bindings and can have different reachability.

### DNS Problems Should Be Isolated Layer by Layer

A Pi-hole failure can involve the application, Linux socket binding, an existing resolver, interface selection, or client DNS configuration.

## Result

The incident gave me practical experience inspecting DNS listeners and understanding how Linux address/interface binding affects whether Pi-hole can claim port 53.
