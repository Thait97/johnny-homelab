# Immich Patch-Only Image Preparation — November 11–13, 2026

**Status:** Implemented and dry-run/systemd tested; timer enabled. **First actual scheduled image preparation not yet observed. Automatic Immich deployment intentionally disabled.**

## Purpose and policy

Detect official, stable Immich patch versions in the currently pinned major/minor series, validate backup/storage prerequisites, and prepare container images for a later **manual deployment**.

| Candidate relative to v3.4.1 | Policy |
| --- | --- |
| v3.4.2 or v3.4.10 | Eligible patch |
| v3.5.0 | Blocked minor upgrade |
| v4.0.0 | Blocked major upgrade |
| v3.4.0 | Blocked downgrade |
| v3.4.1 or prerelease | Blocked |

The system also checks for official published stable releases and refuses to proceed if safety conditions fail. PostgreSQL/Valkey are **never automatically upgraded**.

## Components

```text
/usr/local/lib/immich-updater/
  updater.sh
  check_patch.py
  check_backup.py
  download_images.sh
  verify_images.sh
```

- `check_patch.py`: stable patch release eligibility.
- `check_backup.py`: source/backup mount, backup freshness and storage checks.
- `download_images.sh`: prepares Immich server and machine-learning images using a localhost Tinyproxy workflow.
- `verify_images.sh`: verifies prepared image presence and compatibility.
- `updater.sh`: orchestration and safeguards.

A shared maintenance lock `/run/immich-weekly-backup.lock` is used to coordinate this with full-library backup and daily database replication.

## Testing evidence

- Bash/Python syntax validation.
- Positive and negative simulated patch-version cases: newer patches, downgrade, same version, major/minor changes and prereleases.
- Simulated download success followed by simulated image-verification failure; orchestration stopped safely.
- A controlled simulation was adjusted after an initial test depended on an image that was not present locally.
- Dry-run testing did not deploy or restart containers.
- A systemd test with image preparation **disabled** confirmed lock acquisition, correct drives, recent database backup, valid full backup, free space and patch detection.

These checks establish the safety logic and service invocation under test conditions; they **do not prove a real scheduled image pull has succeeded**.

## systemd scheduling

```text
/etc/systemd/system/immich-patch-updater.service
/etc/systemd/system/immich-patch-updater.timer
```

- Every **Wednesday at 04:00** in the server's local Pacific timezone (PST/PDT).
- `Type=oneshot`, root execution, `IMMICH_PREPARE_IMAGES=1`.
- Depends on Docker and Tinyproxy; 45-minute timeout; journal logging.
- Enabled and active even after the November 13 reboot.

**Safeguard:** The timer prepares images only; the decision to deploy updates and migrate application data remains manual.

## Future validation

Observe the first real Wednesday execution, verify that the expected patch was selected, verify actual image download and checksum/version outcomes, inspect journal logs, and add failure/patch notifications. Review any skipped runs and image cache behavior.

**Skills:** Bash/Python orchestration, semantic-version policy, official release checks, localhost proxy, GHCR troubleshooting, fail-closed handling, concurrency locking, safe automation.
