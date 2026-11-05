# Lenovo M720q Six-Drive NAS — Validation Checklist

**Gate:** Research and physical fitment come before additional purchases.  
**Current status:** All checks below are **unverified** unless explicitly described as documented purchase information.

## Acquired

- [x] Purchased Lenovo ThinkCentre M720q Tiny (listing: i5-8400T, 8 GB DDR4, 256 GB SSD)
- [ ] Confirm received model/specification and physically inspect condition

## Machine Inspection — Read-Only / Non-Destructive

- [ ] Confirm SSD form factor and interface: M.2 NVMe or 2.5-inch SATA
- [ ] Record occupied and available M.2 slots
- [ ] Record installed memory arrangement (1×8 GB vs 2×4 GB, or other)
- [ ] Inspect physical PCIe expansion connection and riser provisions
- [ ] Check BIOS access, health, and boot behavior
- [ ] Verify CPU/RAM/storage specifications as shipped
- [ ] Document current case interior and rear-panel photos (sanitize serial number)
- [ ] Determine whether the SSD can remain as a dedicated TrueNAS boot device

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

- [ ] Finalize six-drive enclosure/backplane design
- [ ] Select SFF-8088 or SFF-8644 cable solution **after** HBA selection
- [ ] Verify correct SAS-to-SATA routing/connectivity and connector direction
- [ ] Confirm drive enumeration, SMART data, and stable reconnects
- [ ] Finalize boot-drive and ZFS layout
- [ ] Evaluate total additional budget ($270–610 / $280–625 excluding HDDs and M720q)
- [ ] Compare modified Tiny vs larger chassis for cost, safety, and maintenance

## Production Protection

- [x] Existing Ubuntu server and Immich main/backup storage designated **out of scope** for early NAS experiments
- [ ] Keep current Sunday 03:00 backup and 30-day retention unchanged
- [ ] Create migration plan only after the NAS passes inspection and controlled tests
- [ ] Demonstrate an appropriate restore procedure before moving authoritative data

## Evidence to Add Later (Do Not Invent)

- [ ] Sanitized M720q exterior/interior photos
- [ ] SSD identification, RAM inspection, BIOS observations
- [ ] Measured HBA bracket, lid and heatsink clearance
- [ ] Fan model, mounting, air path and thermal validation
- [ ] HBA firmware and Linux/TrueNAS detection results
- [ ] Six-drive enumeration and SMART results
- [ ] Final bill of materials and actual cost
- [ ] Deployment and migration verification

**Do not mark a checklist item complete merely because a product specification claims a feature. Physical inspection and relevant testing are required.**
