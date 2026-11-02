# Future NAS Architecture

## Project Goal

I am designing a centralized NAS to separate persistent storage from application compute in my homelab.

The goal is to create a storage platform that can serve photos, documents, media, backups, and other long-lived data while allowing services such as Immich and Jellyfin to run on separate compute systems.

This page documents the current design direction. It is a planned architecture, not a completed deployment.

## Hardware Status — November 2026

I purchased a **Lenovo ThinkCentre M720q Tiny** with:

- Intel Core i5-8400T
- 8 GB DDR4 RAM
- 256 GB SSD

The system is intended to become the dedicated NAS controller, but it is **not yet deployed**.

Current status:

```text
Hardware acquired
→ pending physical inspection
→ pending architecture validation
→ not yet deployed
```

Before purchasing the remaining NAS hardware, I will inspect the machine to verify:

- whether the included 256 GB SSD is M.2 NVMe or 2.5-inch SATA
- the actual RAM configuration
- available M.2 and PCIe expansion paths
- whether the 256 GB SSD can remain the TrueNAS boot drive
- BIOS condition and general hardware health
- whether the M720q has usable PCIe expansion/riser capability

The final storage-controller design will be selected only after the physical system is validated.

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

The HBA option may become preferable if the physical machine supports it cleanly.

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

This is still a planned deployment.

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

The NAS has moved beyond planning-only because the controller computer has now been purchased, but the system is still not part of the production homelab.

Current status:

```text
Lenovo M720q purchased
→ awaiting inspection
→ controller architecture not finalized
→ remaining components not purchased
→ TrueNAS not deployed
→ storage pool not deployed
```

After the M720q is physically inspected, this page will be updated with the verified SSD type, RAM configuration, free expansion interfaces, BIOS/hardware-health results, selected controller path, RAM target, boot-drive arrangement, storage topology, and thermal requirements.
