# Ansible SSH-Agent Troubleshooting Case Study

## Problem

While running a read-only Ansible health check against my Ubuntu homelab server, Ansible failed with an SSH authentication error:

```text
Permission denied (publickey)
```

The important part was determining whether the problem was:

- the server
- SSH itself
- the SSH key
- the SSH agent
- or Ansible

Instead of changing multiple things at once, I tested each layer separately.

## Environment

### Ansible Controller

- macOS
- Ansible
- OpenSSH
- Ed25519 SSH key

### Managed Host

- Ubuntu Server
- SSH enabled
- key-based authentication
- Ansible-managed health checks

The Mac acts as the Ansible controller and connects to the Ubuntu server over SSH.

## Troubleshooting

### 1. Test SSH Without Ansible

The first step was to remove Ansible from the troubleshooting path and test SSH directly.

```bash
ssh johnnyt@192.168.1.x
```

The SSH connection worked, but macOS asked for the SSH private key passphrase.

That proved several things:

- the Ubuntu server was reachable
- the SSH service was working
- the SSH key was valid
- the server accepted the corresponding public key

This narrowed the problem to the client side.

## 2. Check the SSH Agent

I checked whether the key was already loaded into the local SSH agent:

```bash
ssh-add -l
```

The result showed:

```text
The agent has no identities.
```

This was the key finding.

The SSH private key existed and worked, but it was not loaded into the running SSH agent.

## Root Cause

The Ansible connection was failing because the SSH key had not been loaded into the macOS SSH agent.

Manual SSH still worked because I could enter the key's passphrase interactively.

Ansible expected SSH authentication to proceed automatically, but the required key was not available through the agent.

The problem was therefore not:

- the Ubuntu SSH server
- the public key
- the network
- the Ansible playbook

The issue was the local SSH authentication state on the Ansible controller.

## Fix

I added the private key to the SSH agent:

```bash
ssh-add ~/.ssh/id_ed25519
```

After entering the key's passphrase, the key was loaded into the agent.

I verified it with:

```bash
ssh-add -l
```

The command now displayed an identity instead of reporting that the agent had no identities.

## Verification

I ran the Ansible health check again.

The playbook completed successfully with results similar to:

```text
ok=12
changed=0
unreachable=0
failed=0
```

The important results were:

```text
unreachable=0
failed=0
```

This confirmed that Ansible could authenticate to the Ubuntu server successfully.

Because the playbook was designed as a read-only health check:

```text
changed=0
```

was also the expected result.

## Troubleshooting Flow

The incident reinforced the value of isolating each layer instead of immediately changing configuration.

```text
Ansible fails
     |
     v
Test SSH manually
     |
     +-- SSH fails
     |      |
     |      └── investigate network, SSH server, keys, firewall
     |
     +-- SSH works
            |
            v
       Check ssh-agent
            |
            v
       Key not loaded
            |
            v
     ssh-add private key
            |
            v
       Retry Ansible
            |
            v
          Success
```

## Commands Used

```bash
ssh johnnyt@192.168.1.x

ssh-add -l

ssh-add ~/.ssh/id_ed25519
```

I intentionally use a sanitized LAN address in this public documentation.

## What I Learned

This incident helped reinforce several important infrastructure concepts.

### SSH and Ansible Are Separate Layers

Ansible uses SSH for transport, so an Ansible authentication failure can actually be an SSH problem.

Testing SSH independently helps determine which layer is failing.

### A Valid SSH Key Is Not Enough

A private key can exist on disk and still not be available to automated tools.

The SSH agent needs to know about the key before applications can use it without repeatedly prompting for the passphrase.

### Troubleshoot From the Bottom Up

Instead of changing the Ansible configuration immediately, I verified:

1. network reachability
2. SSH service
3. SSH authentication
4. local SSH agent state
5. Ansible

This avoided unnecessary changes and made the root cause easier to identify.

### Verification Matters

A fix is not complete until the original workflow succeeds again.

The final successful Ansible run confirmed that the actual problem had been resolved.

## Result

The issue was resolved without changing the server or the Ansible playbook.

The root cause was a missing SSH identity in the controller's SSH agent, and loading the existing key restored Ansible connectivity.
