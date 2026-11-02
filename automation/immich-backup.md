# Immich Backup Automation

## Project Overview

I built an automated backup process for my self-hosted Immich photo library using Linux, Bash, `rsync`, and `systemd`.

The goal was to create a backup process that could run automatically, verify storage before copying data, and survive reboots without relying on manual commands.

## Environment

### Source Storage

```text
/mnt/photos/Immich
```

### Backup Destination

```text
/mnt/immich-backup/Immich
```

The source and backup are stored on separate physical drives.

## Backup Script

The backup script is located at:

```text
/usr/local/sbin/immich-weekly-backup.sh
```

The script is responsible for validating the environment and synchronizing the Immich data.

The backup process includes:

- verifying the source mount
- verifying the backup mount
- checking available storage space
- running `rsync`
- handling retention
- performing final synchronization
- checking service health

## Why Mount Checks Matter

A mount-point directory can still exist even if the actual drive is not mounted.

For example:

```text
/mnt/immich-backup
```

existing does not prove that the backup disk is actually attached and mounted.

I verify storage using:

```bash
lsblk
df -h
```

This prevents the backup process from accidentally writing into an empty local directory on the OS drive when the intended backup disk is unavailable.

## Synchronization

The backup process uses:

```text
rsync
```

to synchronize Immich data from the primary photo drive to the backup drive.

Conceptually:

```text
Primary Photo Drive
/mnt/photos/Immich
        |
        | rsync
        v
Backup Drive
/mnt/immich-backup/Immich
```

Using `rsync` allows the process to efficiently copy changed data rather than recreating the entire backup every time.

## systemd Service

The backup script is controlled by a systemd service:

```text
immich-weekly-backup.service
```

The service runs the backup script when triggered.

## systemd Timer

A separate timer controls the schedule:

```text
immich-weekly-backup.timer
```

Current schedule:

```text
Sunday 03:00
```

This separates the backup logic from the scheduling logic.

Conceptually:

```text
systemd timer
      |
      v
immich-weekly-backup.service
      |
      v
immich-weekly-backup.sh
      |
      v
rsync + validation
```

## Retention

The current deletion retention period is:

```text
30 days
```

This provides a recovery window before deleted data is permanently removed from the backup workflow.

## Testing

Before depending on the automated schedule, I manually tested the backup process.

The manual test completed successfully.

The first scheduled systemd timer run also completed successfully.

This was important because a configured timer is not enough by itself.

The entire process needed to be verified end-to-end.

## Verification

The backup is checked at multiple layers.

### Storage Verification

```bash
lsblk
df -h
```

Used to confirm that both the source and backup drives are mounted correctly.

### Timer Verification

```bash
systemctl status immich-weekly-backup.timer
```

Used to confirm that the timer is enabled and scheduled.

### Service Verification

```bash
systemctl status immich-weekly-backup.service
```

Used to inspect the most recent service run.

### Logs

```bash
journalctl -u immich-weekly-backup.service
```

Used to inspect backup execution history and troubleshoot failures.

## Architecture

```text
                    systemd
                       |
                       v
        immich-weekly-backup.timer
                       |
                  Sunday 03:00
                       |
                       v
       immich-weekly-backup.service
                       |
                       v
       immich-weekly-backup.sh
                       |
          +------------+------------+
          |            |            |
          v            v            v
     Mount checks   Space checks   rsync
          |                         |
          +------------+------------+
                       |
                       v
             Backup verification
                       |
                       v
        /mnt/immich-backup/Immich
```

## Failure Scenarios Considered

The backup process was designed with several possible failure conditions in mind.

### Backup Drive Missing

If the backup drive is not mounted, the backup should not continue normally.

This reduces the risk of writing backup data to the wrong filesystem.

### Source Drive Missing

If the Immich photo drive is not mounted, the process should stop rather than treating an empty directory as valid source data.

### Low Free Space

Available storage should be checked before starting a large synchronization.

### Service Failure

If the backup service fails, systemd and `journalctl` provide a clear place to inspect the failure.

## What I Learned

### Automation Needs Safety Checks

Scheduling a command is easy.

Building reliable automation requires validating the environment before the command runs.

### A Directory Is Not the Same as a Mounted Disk

This project reinforced the difference between a filesystem path and the storage device that should exist behind that path.

### systemd Services and Timers Have Different Jobs

The service defines what should run.

The timer defines when it should run.

Separating those responsibilities makes automation easier to manage and troubleshoot.

### Backups Must Be Tested

A backup that has never been restored or at least verified is only an assumption.

I manually tested the workflow and then confirmed that the first scheduled execution also completed successfully.

### Logs Are Part of the Design

Using systemd means the automation integrates with Linux logging through `journalctl`, making failures easier to investigate.

## Result

The Immich photo library now has an automated weekly backup workflow using:

- Bash
- `rsync`
- systemd service
- systemd timer
- mount validation
- storage checks
- retention handling
- logging
- manual and scheduled verification

This project gave me practical experience building a repeatable Linux backup workflow instead of relying on manual file copies.
