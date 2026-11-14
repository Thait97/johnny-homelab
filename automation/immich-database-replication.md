# Immich PostgreSQL Backup Replication — November 11–13, 2026

**Status:** Implemented, manually tested through systemd, and scheduled. **First unattended scheduled run after November 13 still to be observed.**

## Goal

Replicate Immich's daily PostgreSQL backup files from the primary photo drive to an independent physical backup drive without changing Immich's database or replacing the weekly full-library backup.

Source: `/mnt/photos/Immich`  
Destination: `/mnt/immich-backup/Immich`

Immich normally creates its daily database backup around **02:00**. Replication runs at **03:30** local Pacific time.

## Implementation

Script: `/usr/local/sbin/immich-db-replicate.sh`

1. Acquire shared lock `/run/immich-weekly-backup.lock`.
2. Verify both drives by UUID and ensure required directories exist.
3. Consider database-backup files older than 30 minutes.
4. Test compressed-file integrity with `gzip -t`.
5. Verify any existing backup copies.
6. Transfer new backups to a **temporary destination filename**.
7. Compare destination contents with the source.
8. Test the copied gzip archive.
9. Rename the verified temporary file to its final name on the destination filesystem.
10. Log success or failure.

The script does not intentionally delete historical backup files, restart Immich, or edit the PostgreSQL database.

### Why temporary files matter

An early version wrote directly to the final filename. An interrupted transfer could therefore leave a partial copy that looked complete on subsequent runs. The revised design transfers, checks, then renames locally. This improves safe retries and prevents partially copied files from appearing under their final names.

## Verified testing

Replicated the November 13 compressed database backup:

```text
immich-db-backup-20261113T020000-v3.4.1-pg14.19.sql.gz
```

Existing backups were verified on both disks. The backup checker confirmed the November 13 backup present and valid on both. The systemd service run returned:

```text
Result=success
ExecMainStatus=0
PASS: Database replication completed
```

## Timer

```text
/etc/systemd/system/immich-db-replicate.service
/etc/systemd/system/immich-db-replicate.timer
```

- Daily at **03:30** Pacific local time.
- `Type=oneshot`, root, required mount paths, 30-minute timeout, journal logging and persistent timer behavior.
- Timer enabled and active after the controlled November 13 reboot.

## Limitations

Thirty-minute file age and `gzip -t` do not absolutely establish that a source file is finished being written. More robust completion detection is a future hardening task. Matching copies and valid compressed archives do **not** constitute a full PostgreSQL restore test.

## Relationship to other maintenance

The shared backup lock coordinates daily DB replication, Sunday full-library backup, and Wednesday patch-image preparation. This is separate from the existing Sunday 03:00 weekly backup with **30-day deletion retention**.

**Skills:** systemd, shell scripting, file locks, UUID checks, integrity testing, safe retries, temporary-file/atomic-rename patterns, backup design.
