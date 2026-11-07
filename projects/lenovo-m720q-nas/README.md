# Lenovo ThinkCentre M720q — TrueNAS Controller & Future Five-Bay NAS

**Status: TrueNAS SCALE installed and locally accessible; storage/DAS/production NAS not deployed. Further hardware purchases: HOLD.**  
**Research baseline:** November 4, 2026  
**Hands-on milestone:** November 6, 2026 — [Installation & verification report](progress-2026-11-06.md)  
**Skills practiced:** hardware compatibility research | storage architecture | PCIe/SAS | thermal planning | risk management | technical documentation

## Objective

I installed **TrueNAS SCALE 25.10.x** on a Lenovo ThinkCentre M720q Tiny to prepare it as a **storage-only NAS controller**. The current preferred *unapproved* enclosure plan is a separately powered/cooled **five-bay hot-swap cage** on a heavy-duty rack shelf (with room for a second cage later). My goal is centralized storage for Immich, Nextcloud, Jellyfin, and other workloads while keeping application compute on separate nodes.

The **controller operating-system installation, initial boot, disk identification, and wired LAN web access have been verified**. The expansion HBA, external drive cage, HDD detection, ZFS data pools, shares, cooling, performance, and reliability **have not been tested or deployed**.

## Acquired vs. Proposed Hardware

| Component | Current status |
| --- | --- |
| Lenovo ThinkCentre M720q Tiny | **Purchased; TrueNAS installed and booted**; expansion fitment pending |
| Intel Core i5-8400T | Controller processor, documented in November 6 setup |
| 8 GB DDR4 | Installed capacity reported; DIMM arrangement not yet documented |
| SanDisk X400 256 GB SATA SSD | **Verified TrueNAS boot drive** |
| Tiny5 PCIe x8 riser `01AJ940` | Research candidate; **not purchased or fit-tested** |
| LSI 9207-8e | **Preferred research candidate**, not purchased |
| LSI 9300-8e | Alternative candidate, not purchased |
| Directed 40 mm fan / duct / replacement lid | Conceptual; not purchased or validated |
| Toshiba 256 GB NVMe | Existing Windows partitions retained; planned physical removal, not erased |
| External SAS cabling, RackChoice five-bay cage, drive PSU, HDDs | Not finalized or purchased |
| TrueNAS SCALE 25.10.x | **Installed; boot and local web UI verified**; no data pool or shares |

### Proposed storage expansion (not yet built)

```text
Router --- wired LAN --- M720q TrueNAS controller (OS installed)
                             |
                      PCIe riser (pending)
                             |
                      SAS HBA (pending)
                             |
                  External SAS cabling (TBD)
                             |
               RackChoice 5-bay hot-swap cage
                    (not purchased)
                       /            \
               Drive power      Drive cooling
                 (TBD)             (TBD)
```

The five-bay shelf-mount concept supersedes the earlier six-drive enclosure sizing plan. A second cage may be added later, subject to HBA lane capacity and separate power/cooling design. No data disks or pools have been installed.

## November 6 — Confirmed Installation & Verification

The full [progress report](progress-2026-11-06.md) records the following:

- Installed TrueNAS SCALE on the **SanDisk X400 256 GB SATA SSD** and verified it boots.
- Checked `lsblk -f` and `efibootmgr -v`: TrueNAS boot disk identified, both internal drives seen, and existing Windows boot entries/partitions preserved on the Toshiba NVMe.
- Obtained a **private LAN address over Ethernet** and accessed the TrueNAS web interface from a MacBook.
- **No data pool or NAS shares created.** The Toshiba NVMe has **not** been erased.
- Reconnected the original Ubuntu server and verified its photo and backup mounts with `lsblk -o NAME,SIZE,MOUNTPOINTS`, plus container status via `docker ps`.
- Immich components, Pi-hole, Uptime Kuma, and Portainer reported healthy; `opengym-api-1` and `opengym-web-1` still reported **health: starting** at the final inspection.

These are *setup and point-in-time validation* results, not production storage or long-term stability tests.

## Research: HBA Candidates

Figures below are **manufacturer-specification research**, *not measured operating results inside the M720q*. The two HBAs were originally researched for six drives; this earlier comparison remains relevant to evaluating the newer five-bay concept. The full installation cannot yet be called compatible.

| Specification | LSI 9207-8e | LSI 9300-8e |
| --- | --- | --- |
| PCIe host | 3.0 x8 | 3.0 x8 |
| External SAS connectors | 2 × SFF-8088 | 2 × SFF-8644 |
| Nominal board power | 9.8 W | 14.5 W |
| Documented worst-case board power | 16 W | 22.5 W |
| Board dimensions (approx.) | 6.6 × 2.7 in | 6.6 × 2.7 in |
| Specified minimum airflow | 200 LFM | 200 LFM |
| Status | **Preferred for evaluation** | Alternative |

The 9207-8e currently leads on lower nominal power, but this does **not** prove it will run cool enough or fit with the M720q lid, riser, rear connectors, and CPU thermal assembly. Compatibility with the eventual TrueNAS SCALE release and correct IT-mode firmware are also **unverified**.

Manufacturer technical references: [Broadcom 9207-8e User Guide](https://docs.broadcom.com/doc/12353330) and [Broadcom 9300-8e User Guide](https://docs.broadcom.com/doc/12354878).

## Riser and Clearance Research

The Tiny5 riser **`01AJ940`** is the leading research candidate. **`01AJ929` must not be assumed interchangeable.** Riser part number, electrical lane mapping, mounting, bracket geometry, lid closure, and connector exits require verification against the actual machine.

Lenovo's general [M720 Tiny platform specifications](https://psref.lenovo.com/product/ThinkCentre/ThinkCentre_M720_Tiny?tab=spec) describe a PCIe 3.0 x8 low-profile expansion option, but a platform-level specification **does not prove that this specific renewed unit has the required riser or usable physical clearance**.

## Cooling: Primary Open Risk

Both evaluated HBA guides specify **200 linear feet per minute** minimum airflow under their documented conditions. This is an **air-velocity requirement**, not simply a fan diameter or CFM rating.

Concepts to evaluate, none approved yet:

1. Original cover with measured/verified directed internal airflow
2. Reversible ventilated replacement cover with a secured fan and air path
3. Secure external fan with guarded intake and reliable mounting

A USB-powered 5 V Noctua 40 mm fan is a **candidate only**. Its electrical power, mounting, airflow delivery through the actual restriction, power-on behavior, noise, and ability to cool the HBA still need validation. A fan pointed vaguely at the case does not prove 200 LFM reaches the heatsink.

**No HBA purchase or operation is approved until fitment and a credible cooling design are established.**

## Cost Planning

Preliminary **additional cost estimates**, excluding purchased M720q and HDDs:

| Candidate route | Estimate |
| --- | --- |
| 9207-8e-based configuration | ~$270–610 |
| 9300-8e-based configuration | ~$280–625 |

These are research ranges **not quotes**, confirmed totals, or authorized purchases. Compare them with the cost, serviceability, safety, and thermal flexibility of a larger chassis before committing.

## Decision and Evidence Tracking

- [Decision log](decision-log.md) — why options are shortlisted or deferred, including what could change the preferred controller
- [Validation checklist](validation-checklist.md) — explicit unverified criteria and evidence required for approval
- [November 6 progress report](progress-2026-11-06.md) — TrueNAS installation, network verification, and safe return to the Ubuntu server
- [General Future NAS Architecture](../../architecture/future-nas.md) — wider NAS, storage, compute, and networking direction

## Protect the Working Homelab

The current Ubuntu server, Immich library, separate backup disk, Sunday 03:00 rsync-based backup, and 30-day deletion retention **remain unchanged** during research and fitment. No production drive will be repurposed for early NAS tests. Migration requires a separate, verified plan.

## Next Milestone

**Next milestone: PCIe/HBA and shelf engineering validation.** Boot drive identity, TrueNAS boot, and initial LAN access are verified. Remaining checks include RAM layout, PCIe riser dimensions, HBA/bracket/cover clearance, cooling route, external connectors, drive-power safety, and five-bay cage mounting. Decide HBA, cabling, enclosure, and cooling **only after** evidence is recorded.

Later, add *actual* sanitized photographs, test results, SMART/device detection, thermal measurements, and storage-pool verification. None are claimed here.

**Project verdict: Controller OS successfully installed; WAIT on additional NAS parts until engineering validation.**
