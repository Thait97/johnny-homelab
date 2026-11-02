# Pi-hole DNS Filtering

## Project Overview

Pi-hole is used in my homelab for DNS-level filtering and as a practical way to learn how DNS affects client devices and network services.

This project also helped me understand the limitations of DNS filtering, the importance of allowlisting, and how port conflicts can affect DNS services.

## Role in the Homelab

Pi-hole provides DNS filtering for selected devices rather than being forced onto every device in the household.

This allows me to test and learn without disrupting other users who may need access to domains that are blocked by my lists.

## DNS

Pi-hole uses DNS on port:

```text
53
```

The web interface is exposed separately for administration.

## What Pi-hole Can and Cannot Do

Pi-hole can block many domains at the DNS level.

It is useful for:

- blocking known advertising domains
- blocking tracking domains
- seeing DNS query activity
- testing allowlists and blocklists
- learning DNS behavior

However, DNS filtering has limits.

For example, Pi-hole does not reliably remove YouTube ads because the video and advertisement traffic may come from the same or closely related delivery infrastructure.

## Allowlisting

One of the practical lessons from using Pi-hole was learning how to restore access when a legitimate site is blocked.

The workflow is:

```text
Site fails to load
      |
      v
Check Pi-hole query activity
      |
      v
Identify blocked domain
      |
      v
Allowlist if appropriate
      |
      v
Retest the site
```

This reinforced that a blocklist should not be treated as automatically correct.

## Port 53 Conflict

A DNS-related issue was investigated when another system service was already using DNS-related sockets.

This introduced an important Linux networking concept:

> Two services cannot independently bind to the same address and port combination at the same time.

Useful checks include:

```bash
sudo ss -lntup
```

and inspecting the system resolver configuration.

The troubleshooting process focused on identifying which process was listening on port 53 before changing configuration.

## Client Scope

I intentionally do not require every family device to use Pi-hole.

That gives me a safer test environment:

```text
Selected devices
      |
      v
   Pi-hole
      |
      v
Upstream DNS
```

while other household devices can continue using their normal DNS path.

## Future Direction

The planned DNS architecture is:

```text
Client
  |
  v
Pi-hole
  |
  v
Unbound
  |
  v
Recursive DNS
```

Unbound is planned as a future addition once the network-infrastructure node is finalized.

## What I Learned

This project reinforced:

- how DNS requests work
- DNS-level filtering
- allowlisting and troubleshooting
- port binding and conflicts
- why DNS filtering cannot block every type of advertisement
- how client DNS configuration changes network behavior
- why infrastructure changes should be introduced gradually in a shared home network

## Result

Pi-hole is now part of my homelab as a controlled DNS filtering service and learning platform rather than simply an ad blocker.
