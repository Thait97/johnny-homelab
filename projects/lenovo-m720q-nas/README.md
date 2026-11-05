# Lenovo ThinkCentre M720q — Six-Drive NAS (Planning & Validation)

**Status: WAIT — engineering validation required. Not deployed.**  
**Research snapshot:** November 4, 2026  
**Skills practiced:** hardware compatibility research | storage architecture | PCIe/SAS | thermal planning | risk management | technical documentation

## Objective

I am planning to use a Lenovo ThinkCentre M720q Tiny as a **storage-only NAS controller** connected to a separately powered, cooled six-drive HDD enclosure. My goal is centralized storage for Immich, Nextcloud, Jellyfin, and other workloads while keeping application compute on separate nodes.

This is an **engineering research and purchasing project**, not a completed NAS build. Actual installation, cooling, drive detection, ZFS configuration, performance, and operational reliability have **not** been tested.

## Acquired vs. Proposed Hardware

| Component | Current status |
| --- | --- |
| Lenovo ThinkCentre M720q Tiny | **Purchased**; physical inspection/validation pending |
| Intel Core i5-8400T | Listed specification of purchased system; check during inspection |
| 8 GB DDR4 | Listed specification; DIMM arrangement not inspected |
| 256 GB SSD | Listed specification; M.2 vs 2.5-inch interface not verified |
| Tiny5 PCIe x8 riser `01AJ940` | Research candidate; **not purchased or fit-tested** |
| LSI 9207-8e | **Preferred research candidate**, not purchased |
| LSI 9300-8e | Alternative candidate, not purchased |
| Directed 40 mm fan / duct / replacement lid | Conceptual; not purchased or validated |
| External SAS cabling, six-drive shelf, drive PSU, HDDs | Not finalized or purchased |
| TrueNAS SCALE / ZFS | Being evaluated; **not installed/deployed** |

### Proposed topology (not yet built)

```text
Lenovo M720q Tiny (purchased; not deployed)
        |
        | candidate Tiny5 PCIe x8 riser
        v
External-port SAS HBA (9207-8e / 9300-8e candidate)
        |
        | appropriate SAS breakout/connectivity (TBD)
        v
Six-drive external 3.5-inch HDD enclosure (not acquired)
        +-- independent, correctly sized HDD power
        +-- independent drive and controller cooling
        |
        v
Possible TrueNAS SCALE / ZFS pool (not created)
        |
        v
LAN connection to separate application-compute node(s)
```

No cable type or drive topology is approved until controller selection, enclosure/backplane design, and power validation.

## Research: HBA Candidates

Figures below are **manufacturer-specification research**, *not measured operating results inside the M720q*. The two HBAs can support six drives in principle; the full installation cannot yet be called compatible.

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
- [General Future NAS Architecture](../../architecture/future-nas.md) — wider NAS, storage, compute, and networking direction

## Protect the Working Homelab

The current Ubuntu server, Immich library, separate backup disk, Sunday 03:00 rsync-based backup, and 30-day deletion retention **remain unchanged** during research and fitment. No production drive will be repurposed for early NAS tests. Migration requires a separate, verified plan.

## Next Milestone

**Physical M720q inspection**: document SSD type, RAM module layout, BIOS/boot health, physical PCIe path, riser dimensions, HBA and bracket clearances, lid fit, cooling route, external connector space, and power options. Decide HBA, cabling, enclosure, and cooling **only after** evidence is recorded.

Later, add *actual* sanitized photographs, test results, SMART/device detection, thermal measurements, and storage-pool verification. None are claimed here.

**Project verdict: WAIT — purchases on hold pending engineering validation.**
