# Lenovo M720q NAS — Controller and Five-Bay Expansion Validation Checklist

**Gate:** Research and physical fitment come before additional purchases.  
**Current status (November 6):** TrueNAS boot, storage identities and initial Ethernet/web access verified; expansion and production storage remain unverified.

## Acquired

- [x] Purchased Lenovo ThinkCentre M720q Tiny (listing: i5-8400T, 8 GB DDR4, 256 GB SSD)
- [x] Identified M720q during hands-on TrueNAS installation; full PCIe expansion/thermal fit inspection still outstanding

## Machine Inspection — Read-Only / Non-Destructive

- [x] Identified SanDisk X400 256 GB **SATA SSD** as TrueNAS boot drive
- [ ] Record occupied and available M.2 slots
- [ ] Record installed memory arrangement (1×8 GB vs 2×4 GB, or other)
- [ ] Inspect physical PCIe expansion connection and riser provisions
- [x] Verified successful TrueNAS boot and examined UEFI boot entries using `efibootmgr -v` (full hardware-health validation pending)
- [x] Verified two internal disks via `lsblk -f`: SanDisk SATA boot drive and Toshiba 256 GB NVMe with existing Windows partitions (RAM module layout pending)
- [ ] Document current case interior and rear-panel photos (sanitize serial number)
- [x] Confirmed SanDisk SATA drive holds the TrueNAS boot pool; long-term boot-drive reliability not yet tested

## Riser & HBA Fitment — No Purchasing Until Reviewed

- [ ] Confirm `01AJ940` electrical and mechanical compatibility
- [ ] Ensure `01AJ929` is not substituted without independent verification
- [ ] Obtain measurements for 6.6-inch-long HBA and heatsink clearance
- [ ] Confirm low-profile bracket fit and external SAS connector clearance
- [ ] Check CPU thermal assembly interference
- [ ] Check original lid clearance and whether an unmodified cover can close
- [ ] Check serviceability and secure card mounting

## HBA / OS Compatibility

- [ ] Confirm 9207-8e firmware/IT mode and chosen TrueNAS SCALE release support
- [ ] Confirm 9300-8e firmware/IT mode and chosen TrueNAS SCALE release support
- [ ] Inspect controller authenticity/condition before eventual deployment
- [ ] Confirm direct disk and SMART visibility in a future controlled test
- [ ] Choose a controller only after balancing support, power, fitment, and cooling

## Cooling & Power — Primary Safety Gate

- [ ] Define the directed air path over the HBA heatsink
- [ ] Verify airflow design against HBA manufacturer requirement (200 LFM)
- [ ] Test candidate 40 mm fan electrical compatibility and startup reliability
- [ ] Validate power-on, reboot, and fan-failure behavior
- [ ] Verify fan mounting, intake guarding, and cable routing
- [ ] Confirm air path and cover do not compromise CPU cooling
- [ ] Plan HDD PSU capacity including startup loads and safe wiring
- [ ] Plan enclosure drive airflow and cooling
- [ ] Obtain suitable thermal and stability evidence before approving operational use

## External Storage System — Not Yet Designed/Approved

- [ ] Validate preferred RackChoice **five-bay** hot-swap cage and heavy-duty shelf mounting (second cage reserved as a future option)
- [ ] Select SFF-8088 or SFF-8644 cable solution **after** HBA selection
- [ ] Verify correct SAS-to-SATA routing/connectivity and connector direction
- [ ] Confirm drive enumeration, SMART data, and stable reconnects
- [x] Boot drive identified and TrueNAS OS installed; [ ] ZFS data pool/layout remains unconfigured
- [ ] Evaluate total additional budget ($270–610 / $280–625 excluding HDDs and M720q)
- [ ] Compare modified Tiny vs larger chassis for cost, safety, and maintenance

## Production Protection

- [x] Existing Ubuntu server and Immich main/backup storage designated **out of scope** for early NAS experiments
- [x] November 6 session preserved existing Ubuntu photo/backup mounts and did not change backup scheduling or retention; future migration validation still required
- [ ] Create migration plan only after the NAS passes inspection and controlled tests
- [ ] Demonstrate an appropriate restore procedure before moving authoritative data

## Evidence to Add Later (Do Not Invent)

- [ ] Sanitized M720q exterior/interior photos
- [x] Identified boot SATA SSD and preserved Windows-containing NVMe; [ ] detailed RAM/BIOS health and PCIe inspection records
- [ ] Measured HBA bracket, lid and heatsink clearance
- [ ] Fan model, mounting, air path and thermal validation
- [ ] HBA firmware and Linux/TrueNAS detection results
- [ ] Five-drive enumeration and SMART results (external disks not yet connected)
- [ ] Final bill of materials and actual cost
- [ ] Deployment and migration verification

## November 6 Point-in-Time Checks

- [x] TrueNAS SCALE 25.10.x installed and booted from SanDisk SATA SSD
- [x] Existing Windows partitions on Toshiba NVMe preserved
- [x] Web interface reached from MacBook on wired LAN (address redacted)
- [x] Existing Ubuntu server reconnected via Ethernet and reachable over SSH
- [x] Ubuntu photo and backup disks mounted at expected paths
- [x] Core Immich, Pi-hole, Uptime Kuma, and Portainer containers reported healthy
- [ ] OpenGym API and web containers fully healthy at final check (both still `health: starting`)
- [ ] External HBA, five-bay drive detection, ZFS pool, and shares (not installed)

See [November 6 installation progress report](progress-2026-11-06.md).

**Do not mark a checklist item complete merely because a product specification claims a feature. Physical inspection and relevant testing are required.**
