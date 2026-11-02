# Ansible SSH-Agent Troubleshooting Case Study

**Skills demonstrated:** Linux | SSH | Ansible | Authentication | macOS | Layered troubleshooting

## Problem

While running a read-only Ansible health check against my Ubuntu homelab server, Ansible failed with:

```text
Permission denied (publickey)
```

I needed to determine whether the failure was in the server, SSH, the key, the SSH agent, or Ansible.

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

## Troubleshooting

### 1. Test SSH Without Ansible

```bash
ssh johnnyt@192.168.1.x
```

Manual SSH worked but asked for the private-key passphrase. That showed the server was reachable, SSH was working, the key was valid, and the server accepted the public key.

### 2. Check the SSH Agent

```bash
ssh-add -l
```

Result:

```text
The agent has no identities.
```

This was the key finding: the private key existed and worked, but it was not loaded into the running SSH agent.

## Root Cause

The Ansible connection failed because the SSH key had not been loaded into the macOS SSH agent.

The issue was not the Ubuntu SSH server, public key, network path, or playbook. It was the local SSH authentication state on the controller.

## Fix

```bash
ssh-add ~/.ssh/id_ed25519
```

After entering the key passphrase, I verified the agent again with:

```bash
ssh-add -l
```

## Verification

The Ansible health check then completed successfully:

```text
ok=12
changed=0
unreachable=0
failed=0
```

Because the playbook is read-only, `changed=0` was also the expected result.

## Troubleshooting Flow

```text
Ansible fails
     |
     v
Test SSH manually
     |
     +-- SSH fails -> investigate network / server / key
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

I use a sanitized private LAN address in public documentation.

## What I Learned

- Ansible uses SSH for transport, so an Ansible error can actually be an SSH-layer problem.
- A valid key on disk is not automatically available to automated tools.
- Testing each layer independently avoids unnecessary configuration changes.
- A fix is not complete until the original workflow succeeds again.

## Result

The issue was resolved without changing the server or playbook. Loading the existing key into the SSH agent restored Ansible connectivity.
