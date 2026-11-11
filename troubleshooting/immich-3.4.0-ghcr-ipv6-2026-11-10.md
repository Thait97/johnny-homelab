# Immich v3.4.0 Upgrade — GHCR IPv6 Download Failure

**Date:** November 10, 2026  
**System:** Ubuntu Server / Docker Compose  
**Upgrade:** Immich **v3.3.5 → v3.4.0**  
**Status:** **Completed and verified**

## Incident

While upgrading Immich, Docker repeatedly failed to download container images from GitHub Container Registry (GHCR). Investigation indicated IPv6 connection resets while downloading image blobs. The failure occurred during image acquisition, not while migrating Immich data.

## Diagnosis and response

1. Checked GHCR connectivity over **IPv4 and IPv6** to isolate protocol-specific behavior.
2. Identified IPv6 connection resets affecting container blob downloads.
3. **Temporarily disabled IPv6 on the Ethernet interface** as a controlled workaround.
4. Successfully downloaded both required Immich images.
5. **Restored IPv6 afterward**, avoiding an undocumented permanent networking change.
6. Used Docker Compose to deploy the updated images.
7. Verified **all four Immich containers reported healthy**.
8. Confirmed the running application reported **v3.4.0** through the local Immich server API.

### Relevant command patterns

The investigation involved protocol-specific connectivity testing and the Docker Compose workflow. The following are **illustrative follow-up commands, not claimed verbatim execution logs**:

```bash
# Example tests of separate IP families
curl -4 -I https://ghcr.io/
curl -6 -I https://ghcr.io/

# From the directory with the validated Immich Compose configuration
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

Temporary IPv6 configuration commands, actual registry URLs, and API authentication values are omitted because the handoff did not record the exact commands or safe sanitized output. No permanent IPv6 disablement is claimed.

## Outcome

- **Immich v3.4.0 confirmed** through its local server API.
- **All four** Immich containers passed the recorded Docker health-status check.
- IPv6 was restored after the download workaround.
- The weekly backup and its validation were performed before the upgrade; see [November 10 backup verification](../automation/immich-backup-verification-2026-11-10.md).

## Lessons learned

- Registry image-download errors can originate in the **transport/network path**, not the application or Compose configuration.
- Comparing IPv4 and IPv6 behavior helps isolate dual-stack connectivity issues.
- A temporary workaround should be reversed and verified.
- Validate the **application's reported version** in addition to container health.

**Scope:** The successful upgrade and point-in-time checks are documented; this is not proof that all future GHCR transfers or every application feature are error-free.
