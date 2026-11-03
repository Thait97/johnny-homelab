# Troubleshooting Immich External Library Re-Indexing

**Environment:** Ubuntu Server 24.04 LTS, Docker, Immich v3.3.5, PostgreSQL, Python, systemd  
**Investigation:** November 2–3, 2026  
**Status:** Probable root cause identified; overnight validation pending

**Skills demonstrated:** Linux | Docker inspection | PostgreSQL | Python | systemd | Root-cause analysis | Data integrity | Safe troubleshooting

## Problem

Photos moved to Immich Trash and permanently deleted later appeared in automated photo-review albums. My **Burst Review** workflow groups rapid photo sequences; my **Screenshot Review** Python script and systemd timer collect screenshots into a review album.

At first, this looked like a possible backup restoration or review-automation error. I investigated the source of the returning photos while preserving the original library and backup data.

## Environment

- Ubuntu Server and Docker / Docker Compose
- Immich and PostgreSQL
- Python review automation and systemd timers
- Dedicated local photo backup
- Tailscale remote administration
- Read-only iPhone photo archive mounted as an Immich external library

## Investigation

### 1. Record deleted assets

Before emptying Trash, I queried PostgreSQL for deleted asset IDs and timestamps:

```sql
SELECT
    id,
    "originalFileName",
    "fileCreatedAt",
    "createdAt",
    "deletedAt"
FROM asset
WHERE "deletedAt" IS NOT NULL
ORDER BY "deletedAt" DESC;
```

I exported the records to a local text file for comparison. Asset IDs are more reliable identifiers than filenames, which can be duplicated.

### 2. Verify permanent deletion

A controlled initial test found that **13 selected asset records** were removed. A larger test recorded **1,592 assets** before deletion. After emptying Trash, I checked the original IDs again:

```text
remaining_old_assets
--------------------
0
```

This confirmed the **original database records** were gone, not that the original external-library files had been removed.

### 3. Compare returning records

After a later nightly cycle, a previously deleted photograph appeared again with the same filename and capture timestamp but **a new Immich asset ID** and a later record-creation timestamp. That strongly suggested **re-indexing** rather than persistence of the old database record.

### 4. Distinguish external and managed assets

PostgreSQL inspection showed the replacement asset belonged to an external library:

```text
isExternal = true
```

I also found an older managed-library record with the same filename. This reinforced why filename-only matching is unreliable.

### 5. Inspect container mounts

I inspected Immich's Docker mounts:

```bash
sudo docker inspect immich_server \
  --format '{{range .Mounts}}{{println .Source .Destination .RW}}{{end}}'
```

The photo archive mount reported:

```text
RW: false
```

The container had read-only access to those original files. Removing an asset from Immich's database therefore did not necessarily delete the original file in the external archive.

### 6. Identify the scheduled scan

Immich administration settings showed periodic external-library scanning was enabled:

```cron
0 0 * * *
```

The scheduled midnight scan aligned with the creation time of the replacement asset. The evidence strongly indicated that the scan rediscovered files still present in the external archive.

### 7. Inspect review automations

I inspected the Screenshot Review systemd service and Python script. It queries active screenshot assets in PostgreSQL, checks whether they are in the review album, and adds missing assets using the Immich API. It **does not upload or restore photos**.

The Burst Review workflow also adds **existing qualifying Immich asset IDs** to an album. Thus, **creating an asset record** and **adding an existing asset to an album** are separate operations.

## Actions Taken

- Disabled periodic external-library scanning as a temporary mitigation
- Verified experimental library watching was already disabled
- Preserved the read-only external photo archive
- Recorded 1,592 deleted asset IDs for follow-up
- Verified all 1,592 original IDs were absent after emptying Trash
- Left existing photo backups untouched
- Retained review automation for targeted follow-up testing

No original backup files were manually deleted.

## Probable Mechanism

```text
Original file remains in read-only external archive
                       |
                       v
              Scheduled Immich scan
                       |
                       v
             New Immich asset record
                       |
                       v
              Photo review detection
                       |
                       v
              Asset added to album
```

**Working conclusion:** External-library re-indexing is the most likely explanation, rather than the weekly backup restoring deleted photos. Disabling scanning is a **mitigation under test**, not a verified permanent resolution.

## Lessons Learned

- **Database deletion is not always file deletion.** External libraries can retain source files independently of Immich's asset database.
- **Asset IDs matter.** Distinct assets may share a filename.
- **Timestamps help reconstruct failures.** Deletion time, creation time, and scheduled jobs narrowed the probable mechanism.
- **Separate the layers.** External files, database records, scheduled scans, and review scripts each have different responsibilities.
- **Protect data during troubleshooting.** Inspection and non-destructive mitigation reduced risk; backups remained untouched.

## Follow-Up Validation — Planned for November 4, 2026

- Compare recorded deleted assets with current database records
- Check whether replacement asset IDs have appeared for previously deleted photos
- Confirm scheduled external-library scanning remains disabled
- Test Burst Review and Screenshot Review separately
- Review logic for intentionally kept photographs
- Investigate thumbnail-generation errors observed in Immich logs

**Result as of November 3:** A probable external-library re-indexing mechanism was isolated, a non-destructive mitigation applied, and a repeatable validation process established. **Final resolution is pending validation.**
