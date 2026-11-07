# Nextcloud Docker Compose Troubleshooting — Historical Project

**Project status:** Previously deployed; Nextcloud removed from the active Ubuntu homelab on November 6, 2026. This document preserves a real earlier troubleshooting experience, not a currently running service.

## Problem

While working with my Nextcloud Docker Compose stack, I encountered a configuration issue caused by a duplicate YAML key.

This was a useful reminder that container failures are not always application failures. Sometimes the problem is in the configuration file that defines the stack.

## Environment

The former Nextcloud stack included:

- Nextcloud
- MariaDB
- Redis
- cron
- Docker Compose

The historical configuration used storage under:

```text
/mnt/storage/nextcloud
```

## Troubleshooting Approach

The first goal was to determine whether the problem was:

- Docker itself
- a container
- Nextcloud
- the database
- or the Compose configuration

Reviewing the Compose file revealed a duplicate YAML key.

## Why Duplicate YAML Keys Matter

YAML is indentation-sensitive and expects keys within the same mapping to be unique.

A duplicate key can cause parsing problems or unexpected configuration behavior.

Conceptually:

```yaml
services:
  nextcloud:
    environment:
      EXAMPLE: value1
      EXAMPLE: value2
```

This is ambiguous and should be corrected rather than assuming the application is at fault.

## Verification Workflow

After correcting the configuration, the stack should be validated and restarted in a controlled way.

Useful checks include:

```bash
docker compose config
```

This helps validate and render the Compose configuration before applying it.

Then:

```bash
docker compose up -d
```

followed by:

```bash
docker compose ps
```

and, if needed:

```bash
docker compose logs
```

## Troubleshooting Model

```text
Stack problem
    |
    v
Validate Compose file
    |
    +-- syntax/config issue --> fix YAML first
    |
    +-- config valid
            |
            v
      inspect containers
            |
            v
       inspect logs
            |
            v
      verify application
```

## What I Learned

This incident reinforced:

- configuration files are part of the application stack
- YAML structure and indentation matter
- duplicate keys can create subtle failures
- `docker compose config` is useful before restarting services
- troubleshooting should separate configuration, container, and application layers
- changing application settings is unnecessary if the real problem is the Compose definition

## Result

The issue became a practical example of configuration-file debugging and helped me build a better habit of validating Compose files before assuming a service itself is broken.
