# Controlled Ubuntu Maintenance and Reboot — November 13, 2026

**Status:** Updates installed, controlled reboot completed, connectivity and services verified.

## Package review and upgrades

Discovered **eight** pending package upgrades and reviewed changes first with simulated APT operations:

```bash
sudo apt-get -s upgrade
sudo apt-get -s install --only-upgrade PACKAGE
```

Package installations were staged to reduce risk: Docker Buildx and Compose first, then Tailscale, with containerd held until backup, service, timer, and restart-policy checks passed.

| Component | Previous | Updated |
| --- | --- | --- |
| Docker Buildx | 0.38.1 | 0.39.0 |
| Docker Compose | 5.7.0 | 5.7.1 |
| containerd | 2.4.2 | 2.4.4 |
| Tailscale | 1.104.6 | 1.104.7 |

Also updated four GTK-related packages: `gtk-update-icon-cache`, `libgtk-4-1`, `libgtk-4-bin`, and `libgtk-4-common`. No packages were slated for removal in the reviewed simulation.

Confirmed the active SSH session was over local LAN before updating Tailscale. Prior to updating containerd, verified backups, Docker container restart policies, active maintenance jobs and systemd state. `Docker Live Restore Enabled: false` was observed, and all nine containers had `always` or `unless-stopped` restart policies. The containerd package update restarted `containerd.service`; Docker and containerd remained active, with nine containers running afterward.

Final `apt list --upgradable` showed **no pending packages**.

## libc reboot requirement

An unattended `libc6` security update left a reboot-required marker.

```text
REBOOT REQUIRED
libc6
```

Automatic reboot was disabled. A controlled manual reboot was chosen after checking:
- Recent database and full-photo-library backups.
- No active backup or patch-updater jobs.
- Docker/containerd and Tailscale enabled and running.
- Local SSH access and container restart policies.
- All nine containers running.
- All five scheduled automation timers enabled.

### SSH systemd socket activation

Investigated the apparently contradictory state:

```text
ssh.service = disabled / active
ssh.socket  = enabled / active
```

Socket activation explained how SSH remained available despite the service unit being disabled.

## Post-reboot checks

Successfully reconnected via SSH and observed:
- Docker and containerd active.
- Tailscale active; `ssh.socket` active.
- All nine containers recovered and running.
- **Eight containers reported healthy; Portainer reported running without a configured health check.** Do not describe Portainer as health-check-healthy.
- All five automation timers remained enabled and active.
- No pending APT upgrades.
- `PASS: NO REBOOT REQUIRED`.

Immich was still at **v3.4.1**. The patch-image preparation automation did **not** automatically upgrade Immich or its database.

## Final timer inventory

| Timer | Schedule / status |
| --- | --- |
| Immich DB replication | Daily 03:30 — enabled/active |
| Immich full backup | Sunday 03:00 — enabled/active |
| Immich patch-image preparation | Wednesday 04:00 — enabled/active |
| APT package-list refresh | Daily — enabled/active |
| APT unattended security upgrades | Daily — enabled/active |

All times are server-local Pacific time; time-zone adjustment between PST and PDT is expected.

## Remaining work

Observe the next daily replication, Sunday full backup, and Wednesday image preparation in actual scheduled execution. Add meaningful failure alerts. Test a complete Immich database/media restore in an isolated environment. Continue to keep server reboots, PostgreSQL/Valkey updates, and Immich application deployment under manual change control.

**Skills:** APT simulation and staged upgrades, runtime risk control, Tailscale/SSH verification, systemd socket activation, reboot planning, container recovery and timer verification.
