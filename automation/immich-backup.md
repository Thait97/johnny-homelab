# Immich Backup Automation

**Skills demonstrated:** Linux | Bash | rsync | systemd | Storage | Backup design | Verification

## Project Overview

I built an automated **weekly rsync-based Immich backup/synchronization workflow with retention and systemd scheduling**.

The goal is to synchronize my self-hosted Immich photo library to a separate physical drive, verify storage before copying data, retain deleted files for a recovery window, and make the workflow repeatable without relying on manual commands.

## Environment

Source:

```text
/mnt/photos/Immich
```

Destination:

```text
/mnt/immich-backup/Immich
```

The source and backup destination are on separate physical drives.

## Backup Script

```text
/usr/local/sbin/immich-weekly-backup.sh
```

The workflow includes:

- source mount verification
- backup mount verification
- free-space checks
- `rsync` synchronization
- deleted-file retention handling
- final synchronization
- service-health verification

## Why Mount Checks Matter

A mount-point directory can still exist even if the intended disk is not mounted.

I verify storage using:

```bash
lsblk
df -h
```

This reduces the risk of accidentally writing backup data into an ordinary directory on the OS filesystem when the intended backup disk is unavailable.

## Synchronization

```text
/mnt/photos/Immich
        |
        | rsync
        v
/mnt/immich-backup/Immich
```

`rsync` efficiently synchronizes changed data instead of blindly recreating every file on every run.

## systemd Scheduling

Service:

```text
immich-weekly-backup.service
```

Timer:

```text
immich-weekly-backup.timer
```

Current schedule:

```text
Sunday 03:00
```

Current deleted-file retention:

```text
30 days
```

The service defines what runs. The timer defines when it runs.

## Testing and Verification

I manually tested the workflow before relying on the schedule, and the first scheduled run also completed successfully.

Useful checks include:

```bash
lsblk
df -h
systemctl status immich-weekly-backup.timer
systemctl status immich-weekly-backup.service
journalctl -u immich-weekly-backup.service
```

A configured timer is not enough by itself; the source drive, backup drive, synchronization, service result, and logs all need verification.

## Failure Scenarios Considered

### Backup Drive Missing

The workflow should not continue normally when the intended destination disk is not mounted.

### Source Drive Missing

The workflow should not treat an empty mount-point directory as valid source data.

### Low Free Space

Available capacity is checked before a large synchronization.

### Service Failure

systemd and `journalctl` provide a consistent place to inspect failed runs.

## Peer Review and Future Improvements

After getting peer feedback from a friend with homelab/IT experience, I reviewed the backup strategy and identified possible future improvements:

- keep the existing Sunday 03:00 weekly rsync-based workflow
- consider a daily copy-only `rsync` later
- increase deleted-file retention from 30 days toward approximately 60–90 days
- periodically test restores rather than assuming synchronized data is recoverable

I chose to keep the current weekly schedule and 30-day retention for now, with **60 days** as the preferred next retention target.

The implementation and troubleshooting are my own; the outside feedback was advisory and helped me review the design more critically.

## What I Learned

### Automation Needs Safety Checks

Reliable automation validates the environment before doing work.

### A Directory Is Not the Same as a Mounted Disk

A path existing does not prove that the storage device behind it is mounted.

### Backups Need Recoverability Testing

Successful synchronization is useful evidence, but periodic restore testing is a future improvement so validation includes recoverability.

### Peer Review Improves Design

Outside review helped identify retention, synchronization frequency, and restore validation as areas worth revisiting without changing implementation ownership.

## Result

The current implementation is a Sunday 03:00 **rsync-based backup/synchronization workflow** with mount checks, storage validation, systemd scheduling, logging, and 30-day deleted-file retention.

Future improvements remain clearly separate from the deployed design: a longer retention window, optional daily copy-only synchronization, and periodic restore testing.
