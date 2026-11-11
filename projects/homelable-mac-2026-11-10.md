# Homelable Standalone — Local Mac Deployment & Rack Design Evaluation

**Date:** November 10, 2026  
**Status:** **Docker Desktop and frontend-only Homelable deployed locally on Mac; first UI load verified. Rack layout not created yet.**

**Source:** [Pouzor/homelable](https://github.com/Pouzor/homelable)

## Objective

Evaluate a visually clear, portfolio-friendly way to model my planned **18U, 19-inch, all-black rack** and network connections, while keeping physical fitment and engineering measurements in **Onshape**.

## Mac preparation

- Investigated limited MacBook Air storage.
- Available space increased from approximately **1 GiB to 11 GiB** during cleanup.
- Selected unused apps were moved toward removal; **Trash was not confirmed emptied**, and permanent deletion is **not claimed**.
- Installed Docker Desktop for **Apple Silicon**.
- Verified Docker Engine version **29.9.3**.

## Homelable — Completed

1. Reviewed Homelable's network diagrams and Rack Canvas capabilities.
2. Chose the **frontend-only standalone image**, avoiding backend setup.
3. Pulled `ghcr.io/pouzor/homelable-frontend-standalone:latest`.
4. Created and started the `homelable-standalone` container.
5. Published the service only on **`127.0.0.1:3000`** on the Mac.
6. Opened the UI in Chrome and verified the **default demo diagram loaded**.

### Relevant verification examples

These commands demonstrate the installed container and engine without claiming they were the exact commands typed:

```bash
docker version
docker ps --filter name=homelable-standalone
docker inspect homelable-standalone --format '{{json .HostConfig.PortBindings}}'
```

The verified local address is **http://localhost:3000** on the Mac itself.

**Important boundaries:** This is **not** installed on the Ubuntu server. No Homelable backend, scanner, inventory sync, MCP integration, or new production monitoring service was deployed. Frontend-only mode keeps canvas data in browser local storage; diagrams should be exported and backed up before clearing browser data.

## Next steps — Not completed

- Create an **18U 19-inch all-black Rack Canvas**.
- Model existing equipment separately from future equipment.
- Add shelves, faceplates, ports, switch connections and patch cables.
- Export useful rack/network diagrams.
- Compare placement sketches to actual Onshape dimensions, cable bends, clearance, power and airflow.

Homelable is a **visual arrangement tool**, not a CAD, thermal, or electrical simulator. Use Onshape for true physical clearances. Do not claim that a complete rack design, export or hardware fit verification exists yet.

## Portfolio skills

macOS maintenance | Apple Silicon Docker installation | container/image management | loopback-only service publication | self-hosted UI evaluation | documented deployment boundaries | infrastructure visualization

**Result:** Verified a working standalone visualization application on the Mac with no change to the production Ubuntu Docker stack.
