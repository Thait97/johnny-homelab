# Storage Mount Troubleshooting

## Problem

One of the most important storage lessons in my homelab was learning that a mount-point directory existing does not prove the underlying disk is actually mounted.

For example, a path such as:

```text
/mnt/photos
```

can still exist even if the intended drive is missing or failed to mount.

That creates a risk: applications or backup scripts may write into the directory on the root filesystem instead of the intended storage disk.

## Environment

Current storage includes:

```text
120 GB SSD — Ubuntu OS
1 TB HDD — Immich photos
256 GB SSD — application/storage use
1 TB HDD — Immich backup
```

Important mount points include:

```text
/mnt/photos
/mnt/storage
/mnt/immich-backup
```

## Troubleshooting Method

I compare the block-device view with the mounted-filesystem view.

### Block Devices

```bash
lsblk
```

This helps show disks, partitions, and mount relationships.

### Mounted Filesystems

```bash
df -h
```

This confirms which filesystems are actually mounted and where.

## Key Lesson

This is not enough:

```bash
ls /mnt/photos
```

because the directory may exist on the root filesystem even when the dedicated photo drive is not mounted.

A better check is to verify the device and filesystem relationship using both `lsblk` and `df -h`.

## Failure Model

```text
Expected:
Photo HDD
   |
   v
/mnt/photos
   |
   v
Immich data

Failure case:
Photo HDD not mounted
   X
/mnt/photos directory still exists
   |
   v
Writes may land on OS filesystem
```

## Why This Matters for Backups

The same issue applies to the backup destination.

If:

```text
/mnt/immich-backup
```

exists but the backup disk is not mounted, a backup script could write large amounts of data to the wrong filesystem.

That is why my Immich backup automation includes mount validation before synchronization begins.

## Troubleshooting Workflow

```text
Application cannot see storage
        |
        v
Check lsblk
        |
        v
Check df -h
        |
        v
Compare expected mount point
        |
        +-- mounted correctly --> continue application troubleshooting
        |
        +-- not mounted --> fix storage/mount issue first
```

## What I Learned

This incident reinforced:

- directories and mounted filesystems are different concepts
- storage should be verified before troubleshooting the application layer
- backup automation needs mount checks
- `lsblk` and `df -h` answer related but different questions
- infrastructure troubleshooting should start with dependencies such as storage before changing application configuration

## Result

Storage checks are now part of my normal troubleshooting workflow and backup design, reducing the risk of applications or scripts silently writing to the wrong filesystem.
