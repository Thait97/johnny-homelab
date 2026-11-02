# Ansible Health Check

**Skills demonstrated:** Linux | SSH | Ansible | systemd | Docker | Storage checks | Troubleshooting

## Project Overview

I use my Mac as an Ansible controller to run a read-only health check against my Ubuntu homelab server.

The goal is to collect a consistent snapshot of the server's health without making configuration changes.

This helps me practice infrastructure automation while keeping routine checks safe and repeatable.

## Environment

### Controller

- macOS
- Ansible
- OpenSSH

### Managed Host

- Ubuntu Server
- SSH key authentication
- Docker / Docker Compose
- systemd
- Tailscale
- mounted storage

The Ansible project lives under approximately:

```text
~/ansible-homelab/
```

## Read-Only Design

The health check is intentionally designed to inspect the system rather than modify it.

A successful run looks similar to:

```text
ok=12
changed=0
unreachable=0
failed=0
```

For this type of playbook, `changed=0` is a positive result. It means Ansible gathered information and performed checks without changing the server.

## What the Check Is For

The playbook is intended to help answer questions such as:

- Is the server reachable over SSH?
- Are expected services running?
- Are any systemd units failed?
- Are Docker workloads healthy?
- Is storage mounted and available?
- Is there enough disk space?
- Is the network path functioning?
- Are there obvious alerts that need deeper investigation?

## Why Ansible

I could run each command manually, but Ansible gives me a repeatable way to perform the same health checks in the same order.

```text
Mac
 |
 | Ansible over SSH
 v
Ubuntu Server
 |
 +-- systemd checks
 +-- Docker checks
 +-- storage checks
 +-- network checks
 +-- other health checks
```

## Interpreting the Recap

### `ok`
Tasks that completed successfully without making a change.

### `changed`
Tasks that modified the managed host. For this read-only playbook, `changed=0` is expected.

### `unreachable`
Ansible could not establish the required connection to the host.

### `failed`
A task ran but failed.

A clean result should end with:

```text
unreachable=0
failed=0
```

## Troubleshooting Value

One run reported:

```text
Systemd: ALERT - 1 failed
```

I narrowed that alert to `fwupd-refresh.service` rather than assuming the entire server was unhealthy.

Another incident involved:

```text
Permission denied (publickey)
```

Manual SSH testing showed the server and key were valid, and the actual problem was that the key was not loaded into the local SSH agent.

## Safety Principle

Routine health checks should avoid making changes unless a real issue has been identified and a change is intentionally approved.

Monitoring and remediation are different jobs: first identify the problem, then perform a deliberate repair.

## What I Learned

- Automation can be useful without being destructive.
- For a read-only health check, `changed=0` is a good result.
- A health check should narrow the failure domain instead of only printing data.
- Application-specific verification still matters even when infrastructure checks pass.

## Result

The playbook gives me a repeatable, read-only way to assess the homelab from my Mac and a consistent starting point for troubleshooting SSH, systemd, Docker, storage, and network problems.
