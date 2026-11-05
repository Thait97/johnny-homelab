# Lenovo M720q NAS — Engineering Decision Log

**Status:** Research-only decisions as of November 4, 2026. No unlisted parts have been approved for purchase.

| Decision / question | Current direction | Reasoning | What would change it | Validation |
| --- | --- | --- | --- | --- |
| Dedicated storage vs attaching HDDs to Jellyfin PC | Dedicated NAS | Separates persistent data from application compute | Total cost/complexity proves excessive | Architecture decision only |
| Controller computer | M720q Tiny purchased | Small business-class platform with potential PCIe expansion | Actual unit lacks a usable riser/physical clearance | **Pending physical inspection** |
| Storage host interface | PCIe SAS HBA preferred for six-drive shelf | Direct per-disk visibility; avoids primary USB DAS | HBA cannot safely fit/cool; reassess chassis or options | **Unverified** |
| HBA candidate | 9207-8e leads evaluation | Lower specified nominal power (9.8 W vs 14.5 W) | Firmware/support/thermal/connectivity/fitment disadvantage | **Not selected** |
| Alternative HBA | 9300-8e retained | Different external connector generation and controller option | Better verified support or integration | **Not selected** |
| PCIe riser | Tiny5 `01AJ940` candidate | Investigate actual mechanical/electrical fit | Inspection indicates incompatible or inaccessible interface | **Not purchased or fitted** |
| Riser `01AJ929` | Do not substitute without verifying | Reportedly electrically different | Part-number-specific evidence says otherwise | **Unverified** |
| HBA cooling | Dedicated directed 40 mm airflow concept | Both HBA user guides call for 200 LFM | Evidence supports another controlled air path | **Unvalidated** |
| Fan power | 5 V USB fan under evaluation | Potentially reversible power path | Cannot reliably start or maintain adequate cooling | **No fan approved** |
| Cover | Prefer original if validated; consider reversible alternative | Avoid uncontrolled modifications and protect intake | Physical measurements show lid cannot close/ventilate | **Unverified** |
| External SAS cables | Not selected | Connector standards differ: SFF-8088 vs SFF-8644 | HBA and enclosure/backplane selected | **Hold** |
| HDD power and enclosure | Dedicated external power/cooling | HDDs must not depend on Tiny's internal power budget | Engineering review chooses another safe topology | **Hold** |
| Storage OS | TrueNAS SCALE/ZFS being evaluated | Interested in ZFS disk visibility and management | Support, resources, or operational needs suggest another platform | **Not deployed** |
| Production migration | Separate future project | Protect functioning Immich and backups | Verified backup/restore and tested NAS readiness | **Not started** |

## Thermal and Fitment Go/No-Go

The HBA candidate is **not approved** simply because its PCIe specification matches the slot. Validation needs physical clearance, correct riser orientation, safe external connector access, sufficient and reliable air movement, power compatibility, and OS/firmware support. A 40 mm fan by itself is not proof of sufficient heatsink airflow.

## Source Notes

Manufacturer numbers in the [project overview](README.md) are published HBA figures; they have **not** been measured inside this Tiny chassis.

Reference documents:

- [Lenovo M720 Tiny PSREF](https://psref.lenovo.com/product/ThinkCentre/ThinkCentre_M720_Tiny?tab=spec)
- [Broadcom 9207-8e HBA User Guide](https://docs.broadcom.com/doc/12353330)
- [Broadcom 9300-8e HBA User Guide](https://docs.broadcom.com/doc/12354878)

## Change History

- **2026-11-04:** Recorded M720q as purchased only; narrowed the current PCIe/HBA research to 9207-8e vs 9300-8e; identified thermal and bracket/cover fit as major unresolved constraints; paused additional purchases.
- **Future entries:** Record inspection findings, screenshots/measurements, rejected options, the final approved bill of materials, and actual verification results *when available*.
