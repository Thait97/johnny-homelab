# Ansible Health Check

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

For this type of playbook, `changed=0` is a positive result.

It means Ansible gathered information and performed checks without changing the server.

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

The exact checks can evolve as the homelab grows.

## Why Ansible

I could run each command manually, but Ansible gives me a repeatable way to perform the same health checks in the same order.

Conceptually:

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

This reduces the chance of forgetting a routine check and gives me experience with configuration-management tooling in a low-risk workflow.

## Interpreting the Recap

The Ansible recap is one of the fastest ways to understand whether the run completed cleanly.

### `ok`

Tasks that completed successfully without making a change.

### `changed`

Tasks that modified the managed host.

For this read-only playbook:

```text
changed=0
```

is expected.

### `unreachable`

Indicates Ansible could not establish the required connection to the host.

For example, an SSH authentication problem can produce an unreachable result.

### `failed`

Indicates that a task ran but failed.

A clean health-check result should end with:

```text
unreachable=0
failed=0
```

## Troubleshooting Value

The health check is useful not only when everything passes, but also when it identifies one abnormal layer.

For example, one run reported:

```text
Systemd: ALERT - 1 failed
```

I then narrowed that alert to `fwupd-refresh.service` rather than assuming the entire server was unhealthy.

Another incident involved Ansible itself failing with:

```text
Permission denied (publickey)
```

Manual SSH testing showed the server and key were valid, and the actual problem was that the key was not loaded into the local SSH agent.

These incidents turned the health check into a useful troubleshooting entry point.

## Troubleshooting Flow

```text
Run Ansible health check
        |
        v
Review recap
        |
        +-- unreachable > 0
        |      |
        |      v
        |   Check SSH / network / authentication
        |
        +-- failed > 0
        |      |
        |      v
        |   Identify failed task or service
        |
        +-- alerts in output
        |      |
        |      v
        |   Investigate the affected subsystem
        |
        +-- changed > 0
               |
               v
        Confirm whether a change was expected
```

## Safety Principle

Routine health checks should avoid making changes unless a real issue has been identified and a change is intentionally approved.

That separation matters because monitoring and remediation are different jobs.

The check should first tell me what is wrong.

A separate troubleshooting or repair step can then address the issue.

## What I Learned

### Automation Does Not Have to Be Destructive

Ansible is often used to configure systems, but it can also be useful for safe, read-only inspection.

### `changed=0` Can Be the Desired Result

In a health-check workflow, success does not mean Ansible changed something.

It means the system was inspected successfully without unnecessary modification.

### A Health Check Should Narrow the Problem

The goal is not just to print information.

A useful health check should help determine which layer needs attention next.

### Verification Still Matters

Even when Ansible reports success, application-specific verification may still be needed.

For example, a running Docker container does not guarantee that the application inside it is healthy.

## Result

The Ansible health-check playbook gives me a repeatable, read-only way to assess the homelab from my Mac.

It also provides a consistent starting point for troubleshooting SSH, systemd, Docker, storage, and network problems while helping me build practical Ansible experience.
