# Immich Backup — Pre-Upgrade Verification (November 10, 2026)

**System:** Ubuntu Server / Immich / PostgreSQL / rsync / systemd  
**Status:** **Verification completed; full disaster-recovery restore NOT tested**

This is a dated verification record for the existing [Immich Backup Automation](immich-backup.md), not a new backup system.

## Existing policy

- **Source:** `/mnt/photos/Immich`
- **Separate backup drive:** `/mnt/immich-backup/Immich`
- **Schedule:** Sunday at **03:00**
- **Deleted-file retention:** **30 days** (unchanged)

## Verification performed

1. Inspected the weekly backup script and its mount/free-space safety checks.
2. Verified that the **November 8 scheduled job completed successfully**.
3. Verified the destination drive UUID rather than relying on directory existence alone. The actual UUID is intentionally omitted.
4. Ran a gzip integrity test against a recent PostgreSQL backup.
5. Compared SHA-256 hashes of the database backup and its copy on the backup drive; **hashes matched**.
6. Manually invoked the weekly backup systemd service; systemd reported **`0/SUCCESS`**.
7. Confirmed the four Immich containers restarted and reported healthy.
8. Investigated rsync dry-run differences and synchronized **266 missing files**.
9. Copied **three additional missing original PNG uploads**.
10. Final comparison found **no ordinary original-upload files pending transfer**. The remaining observed discrepancy was a timestamp difference on the `.immich` marker.

The 266-file synchronization and three PNG copies are recorded separately as described in the handoff; do not infer that they are mutually exclusive sets or deduce an exact total of unique files without the original manifest.

## Command families used or relevant to validation

**Examples for repeatable checks**, not a complete transcript of exactly what was typed:

```bash
lsblk -f
systemctl status immich-weekly-backup.timer
systemctl status immich-weekly-backup.service
journalctl -u immich-weekly-backup.service
gzip -t <database-backup.sql.gz>
sha256sum <source-backup.sql.gz> <copied-backup.sql.gz>
# Use a reviewed rsync --dry-run comparison before any actual synchronization
docker ps --format 'table {{.Names}}\t{{.Status}}'
```

Paths containing specific private backup filenames, drive UUIDs, and hashes are intentionally not published.

## What the evidence proves — and does not prove

**Verified:** scheduled execution, successful manual service run, sampled PostgreSQL backup integrity, matching backup-file hashes, source-to-destination synchronization checks, and container health at the recorded checks.

**Not verified:** complete disaster-recovery restoration, restoration of all database and media dependencies into a separate working environment, or recovery under drive-failure conditions. Gzip integrity and SHA-256 equality demonstrate file integrity/identity, **not successful PostgreSQL application restore**.

## Future work

- Perform a controlled test restore in an isolated environment.
- Review the `.immich` marker timestamp behavior without blindly overwriting it.
- Maintain current weekly schedule and 30-day retention until a separate approved policy change.
- Document full recovery objectives and acceptance criteria.

**Result:** Stronger evidence that the backup workflow completed and synchronized expected files before the Immich upgrade, without overstating recoverability.
