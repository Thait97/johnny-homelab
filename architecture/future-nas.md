# Future NAS Architecture

## Project Goal

I am designing a centralized NAS to separate persistent storage from application compute in my homelab.

The goal is to create a storage platform that can serve photos, documents, media, backups, and other long-lived data while allowing services such as Immich and Jellyfin to run on separate compute systems.

This page documents a **partially completed controller setup** and the **still-planned storage architecture**. TrueNAS SCALE is installed and was accessed locally over Ethernet, but external storage, data pools, shares, and production NAS service are **not deployed**.

## Hardware Status — November 2026

I purchased a **Lenovo ThinkCentre M720q Tiny** with:

- Intel Core i5-8400T
- 8 GB DDR4 RAM
- 256 GB SSD

**November 6 update:** TrueNAS SCALE 25.10.x was successfully installed and booted from a verified **SanDisk X400 256 GB SATA SSD**. The M720q's **Toshiba 256 GB NVMe** still contains Windows partitions and was not erased. Initial local web administration was verified over Ethernet. No data pool has been created. This does **not** mean the NAS storage system is deployed.

Current verified milestone:

```text
M720q purchased
→ TrueNAS SCALE installed on SanDisk SATA boot SSD
→ UEFI/drive inspection completed
→ first local Ethernet/web management verified
→ no ZFS data pool, HDD cage, shares, or production NAS deployment
```

The remaining physical expansion, HBA, drive-power, and cooling decisions still require engineering validation. [November 6 installation and Ubuntu-server verification report](../projects/lenovo-m720q-nas/progress-2026-11-06.md).

## Prior Six-Drive NAS Engineering Review — November 4, 2026 (Historical Research)

The earlier six-drive research is documented in the dedicated [Lenovo M720q Six-Drive NAS project](../projects/lenovo-m720q-nas/README.md), including a [decision log](../projects/lenovo-m720q-nas/decision-log.md) and [validation checklist](../projects/lenovo-m720q-nas/validation-checklist.md).

The **November 4 research candidate** was M720q PCIe x8 riser → **LSI 9207-8e** → external SAS cabling → separately powered/cooled **six-drive** shelf. The **LSI 9300-8e** remains an alternative. A Tiny5 riser `01AJ940` and dedicated 40 mm fan are also **research candidates only**.

**Critical blockers:** physical riser/card/bracket and lid clearance, controller firmware/TrueNAS compatibility, external SAS connectivity, power, and a directed cooling solution meeting the HBA's specified airflow requirements. These have **not** been tested on the purchased machine. The M.2/ASM1166 approach remains an earlier alternative under review, not the approved build.

**November 6 status: WAIT — TrueNAS controller installation completed; further component purchases remain on hold pending fitment, power, cooling, and architecture approval.** The operational Ubuntu homelab and its Immich backup disks remain unchanged; migration is a separate future project.

## Updated Preferred Storage Direction — November 6, 2026

The latest **proposed** configuration starts with **one RackChoice five-bay SATA/SAS hot-swap cage** mounted on a heavy-duty rack shelf, reserving room for a second cage later. The M720q remains the NAS controller; Immich and Jellyfin compute stay separate.

```text
M720q TrueNAS controller (installed / tested locally)
   |
PCIe riser + SAS HBA (not acquired)
   |
External SAS cable (not selected)
   |
RackChoice 5-bay cage on heavy-duty shelf (not acquired)
   + independent HDD power and cooling (not validated)
   + optional second cage in future
```

This **supersedes the six-drive enclosure concept as the preferred research direction**, not as a final approved build. All remaining purchases are on hold. The preceding six-drive HBA comparison is retained for historical engineering context.

## Architecture Validation Still Required

Two controller paths are currently being considered.

### Option A — M.2 SATA Controller

```text
Lenovo M720q
      |
      v
ASM1166 M.2 PCIe → 6× SATA
      |
      v
6 external 3.5-inch SATA HDDs
      |
      v
Dedicated external power + cooling
      |
      v
TrueNAS / ZFS storage
```

### Option B — PCIe HBA

```text
Lenovo M720q
      |
      v
M720q PCIe expansion
      |
      v
HBA in IT mode
      |
      v
External drive shelf
      |
      v
ZFS storage
```

Neither path is final yet.

The current research favors testing the PCIe HBA approach first, but the final controller path remains unapproved until fit, cooling, compatibility, and cost are validated.

## Components Not Yet Purchased

The following NAS components are still pending architecture validation:

- ASM1166 controller or HBA
- RAM upgrade
- NAS HDDs
- SATA or SAS cabling
- drive-power hardware
- final drive shelf/enclosure hardware
- final cooling hardware

This prevents the design from being locked in before the M720q is physically inspected.

## Why Validate Before Buying More Hardware?

The project is intentionally following this workflow:

```text
Architecture design
→ hardware selection
→ hardware acquisition
→ physical inspection
→ architecture validation
→ component selection
→ deployment
→ testing
→ documentation
```

Buying the controller, RAM, drive hardware, and disks before checking the actual machine could create compatibility problems or force the design around assumptions that turn out to be wrong.

The goal is to validate the available expansion paths first, then choose the cleanest storage architecture.

## Storage Software

The likely storage platform remains:

```text
TrueNAS SCALE
ZFS
```

**TrueNAS SCALE 25.10.x has been installed as the controller OS; a production ZFS data pool remains planned and has not been created.**

ZFS is attractive for the project because the NAS is intended to become the authoritative location for persistent data.

Concepts I am learning as part of this design include:

- pools and vdevs
- redundancy
- snapshots
- checksumming
- disk visibility
- resilvering
- capacity planning
- the difference between redundancy and backup

## NAS vs DAS

A major architecture decision was whether to attach media drives directly to the Jellyfin computer or centralize storage on the network.

### DAS

```text
Storage directly attached to one host
```

### NAS

```text
Storage provided over the network to multiple systems
```

I selected the centralized NAS direction because the same storage platform can serve multiple workloads rather than tying persistent data to one application server.

## Planned Data Roles

The future NAS is intended to hold persistent data such as:

- Immich photos
- documents
- movies
- TV media
- shared files
- backup data
- future homelab storage

Application-specific databases, caches, metadata, and transcode workloads can remain on local NVMe storage where appropriate.

## Immich and Jellyfin Architecture

Immich and Jellyfin are still planned to run on a separate combined compute node and access the NAS over the network.

Target compute characteristics include:

- Intel 12th-generation or newer Core i5
- Intel integrated graphics / Quick Sync
- 32 GB RAM
- 1 TB NVMe
- 2.5 GbE
- Linux
- Docker

Persistent media remains on the future NAS.

Local NVMe can hold:

- operating system
- application configuration
- databases
- metadata
- cache
- temporary transcodes

Conceptually:

```text
                     2.5 GbE Switch
                    /              \
                   /                \
                  v                  v
        Future NAS Controller    Compute Node
             M720q              Immich/Jellyfin
                |                     |
       Persistent storage        Local NVMe
                |                     |
       Photos / Media / Docs     DB / Cache / Config
```

## Network Design

The future rack is planned around a local 2.5 GbE switch.

```text
Router
   |
   v
2.5 GbE Switch
   |
   +-- NAS
   +-- Immich / Jellyfin compute
   +-- network infrastructure
   +-- monitoring / management
   +-- other rack systems
```

Approximate theoretical link rates:

```text
1 GbE   ≈ 125 MB/s
2.5 GbE ≈ 312.5 MB/s
```

Actual performance will be lower because of protocol overhead, disk performance, workload characteristics, and other system limits.

A key networking lesson is that transfers between the NAS and Jellyfin stay on the local network and do not consume internet bandwidth.

## Rack Considerations

The future rack is currently planned at approximately 18U.

Storage-related design considerations include:

- heavy equipment lower in the rack
- UPS near the bottom
- networking near the top
- front-to-rear airflow
- active cooling for storage controllers where required
- clean cable routing
- short Ethernet runs
- separation of storage and compute roles

Because the rack is likely to remain near the router, direct Ethernet is currently preferred over MoCA.

## Redundancy Is Not Backup

One of the most important storage concepts for this project is:

> RAID or ZFS redundancy is not the same thing as a backup.

Redundancy can help the system continue operating after certain disk failures.

It does not automatically protect against:

- accidental deletion
- filesystem-level mistakes
- malware
- catastrophic hardware loss
- theft
- fire
- application-level corruption

The final design still needs a separate backup strategy for important data.

## What I Am Learning

This project is helping me learn:

- hardware validation before deployment
- NAS vs DAS architecture
- M.2 PCIe-to-SATA controllers
- HBA vs RAID controllers
- IT mode
- SAS/SATA connectivity
- ZFS concepts
- storage redundancy
- backup strategy
- PCIe expansion
- 2.5 GbE networking
- rack airflow and cooling
- separation of compute and storage
- capacity planning

## Status

The controller has moved beyond hardware acquisition: TrueNAS SCALE was installed and local web access verified November 6. The storage system remains **pre-production** without HBA, cage, HDDs, pool, shares, or migration.

Current status:

```text
Lenovo M720q purchased
→ TrueNAS SCALE 25.10.x booted from SanDisk 256 GB SATA SSD
→ Toshiba NVMe Windows partitions preserved
→ local Ethernet management/web UI verified
→ HBA, riser, external five-bay cage not installed
→ ZFS data pool and shares not created
→ production NAS not deployed
```

The November 6 hands-on results, including the original Ubuntu-server verification, are in the [progress report](../projects/lenovo-m720q-nas/progress-2026-11-06.md). The next milestone is **physical HBA/riser/power/thermal validation**, not migration.
