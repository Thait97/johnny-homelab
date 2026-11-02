# Future NAS Architecture

## Project Goal

I am designing a centralized NAS to separate persistent storage from application compute in my homelab.

The goal is to create a storage platform that can serve photos, documents, media, backups, and other long-lived data while allowing services such as Immich and Jellyfin to run on separate compute systems.

This page documents the current design direction. It is a planned architecture, not a completed deployment.

## Design Direction

The selected approach is:

```text
Mini/Tiny PC
    |
    | PCIe
    v
HBA in IT mode
    |
    | External Mini-SAS HD
    v
8-bay drive shelf
    |
    v
ZFS storage pool
    |
    v
2.5 GbE network
```

## NAS Controller Requirements

The NAS controller should provide:

- real PCIe expansion
- PCIe 3.0 x8-class connectivity for the HBA
- 16 GB RAM minimum
- 32 GB RAM preferred
- NVMe boot storage
- 2.5 GbE networking
- active cooling for the HBA
- preferably two M.2 slots

A business-class Mini/Tiny system is preferred where it can provide the required PCIe expansion.

## HBA and Drive Connectivity

The storage design uses a Host Bus Adapter rather than a traditional hardware RAID controller.

Target controller class:

```text
LSI / Broadcom 9300-8e
```

The HBA should operate in:

```text
IT mode
```

IT mode exposes individual disks directly to the operating system instead of hiding them behind a hardware RAID virtual disk.

That is important for a storage stack such as ZFS, which is designed to manage the disks directly.

Conceptually:

```text
NAS controller
     |
     v
PCIe HBA (IT mode)
     |
     v
External Mini-SAS HD
     |
     v
8 individual SATA drives
     |
     v
ZFS
```

## Why Not USB Storage?

I am intentionally avoiding USB as the primary connection method for the long-term multi-drive NAS.

The preferred design uses PCIe and an HBA because it provides a more direct and predictable storage path for multiple disks.

I am also avoiding an eight-drive design based on loose USB adapters or many M.2-to-SATA adapters unless an HBA-based design becomes impossible.

## Storage Software

The likely storage platform is:

```text
TrueNAS SCALE
ZFS
```

ZFS is attractive for this project because the NAS is intended to become the authoritative location for persistent data.

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

The NAS is intended to hold persistent data such as:

- Immich photos
- documents
- movies
- TV media
- shared files
- backup data
- future homelab storage

Application-specific databases, caches, metadata, and transcode workloads can remain on local NVMe storage where appropriate.

## Immich and Jellyfin Architecture

The architecture is evolving toward a separate compute node for Immich and Jellyfin.

Target compute characteristics include:

- Intel 12th-generation or newer Core i5
- Intel integrated graphics
- Intel Quick Sync support
- 32 GB RAM
- 1 TB NVMe
- 2.5 GbE
- Linux
- Docker

Persistent media remains on the NAS.

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
               NAS              Compute Node
                |                Immich/Jellyfin
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
- active HBA cooling
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

## Why This Architecture Was Selected

The centralized NAS design gives me a clearer separation of responsibilities:

```text
NAS
= persistent storage

Immich / Jellyfin node
= application compute

Network infrastructure
= DNS, remote access, monitoring, routing support
```

This makes individual systems easier to replace or upgrade without moving the authoritative data with every compute change.

## What I Am Learning

This design project is helping me learn:

- NAS vs DAS architecture
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

This architecture is still in the planning and purchasing stage.

I will update this document as hardware is selected, installed, tested, and moved from planned design into the production homelab.
