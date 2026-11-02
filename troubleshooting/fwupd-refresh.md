# fwupd Refresh Troubleshooting Case Study

## Problem

During a routine Ansible health check, the server reported:

```text
Systemd: ALERT - 1 failed
```

The important part was not to assume the entire server was unhealthy.

Instead, I narrowed the issue down to the specific failed systemd unit.

## Failed Unit

The failed service was:

```text
fwupd-refresh.service
```

This service is related to refreshing firmware metadata.

## Investigation

I first listed failed systemd units:

```bash
sudo systemctl --failed --no-pager
```

This confirmed that `fwupd-refresh.service` was the failed unit.

Next, I inspected the service status in more detail:

```bash
systemctl status fwupd-refresh.service --no-pager -l
```

Then I reviewed the service logs for the current boot:

```bash
journalctl -u fwupd-refresh.service -b --no-pager -n 50
```

The logs showed that the service had succeeded previously but had encountered temporary refresh failures later.

## Troubleshooting Approach

The key was to isolate the problem instead of treating the alert as a full-server failure.

Conceptually:

```text
Ansible health check
        |
        v
1 failed systemd unit
        |
        v
Identify failed unit
        |
        v
fwupd-refresh.service
        |
        v
Inspect status and logs
        |
        v
Test the refresh manually
```

## Manual Test

I manually tested the firmware metadata refresh:

```bash
fwupdmgr refresh --force
```

The refresh completed successfully.

That indicated the service itself was capable of working and that the earlier failure was likely temporary rather than a persistent configuration problem.

## Restarting the Service

After the manual refresh succeeded, I started the service again:

```bash
sudo systemctl start fwupd-refresh.service
```

The service completed successfully.

## Final Verification

I checked failed systemd units again:

```bash
sudo systemctl --failed --no-pager
```

The result showed:

```text
0 loaded units listed.
```

This confirmed that the failed state had cleared.

## Root Cause

The evidence pointed to a temporary failure during firmware metadata refresh rather than a broken server or permanently misconfigured service.

The server itself remained operational.

## What I Learned

### One Failed Unit Does Not Mean the Whole Server Is Down

A systemd alert should be investigated at the unit level.

The server can still be healthy even when one background service has failed.

### Start With `systemctl --failed`

When a monitoring or automation tool reports failed systemd units, this command quickly identifies which service needs attention:

```bash
sudo systemctl --failed --no-pager
```

### Logs Provide the Context

`systemctl status` shows the current state, while `journalctl` provides the historical context needed to understand why the failure happened.

### Test the Underlying Operation Directly

Running:

```bash
fwupdmgr refresh --force
```

helped determine whether the underlying firmware refresh process still worked.

This narrowed the issue down to a temporary service failure.

### Always Re-Verify

The troubleshooting process was not complete until:

```bash
sudo systemctl --failed --no-pager
```

returned no failed units.

## Result

The `fwupd-refresh.service` alert was resolved without rebooting or making unnecessary system changes.

The troubleshooting process showed the value of:

- isolating the specific failed unit
- checking service status
- reading logs
- testing the underlying command directly
- restarting the affected service
- verifying the final system state

The main lesson was simple:

> One failed systemd unit does not mean the entire server is unhealthy.
