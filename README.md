# The ArtCube Protocol

**A Standardized Digital Provenance & Ownership Framework for Art**

Version 1.0 — Working Draft | February 2026

> **Status:** This protocol is under active development. Field names, event schemas, and protocol rules may change before the first stable release. Feedback and contributions are welcome.

---

## What is the ArtCube Protocol?

The ArtCube Protocol defines an open standard for anchoring artwork identity, provenance, ownership, and intellectual property rights to Bitcoin using Ordinals inscriptions.

The protocol uses a four-level hierarchical parent/child inscription structure:

```
Entity Root              → Organization identity
  └─ Collection Parent   → Group of related artworks
       └─ Genesis        → Individual artwork identity + legal anchor
            └─ Events    → Append-only provenance chain
```

Each level is a Bitcoin inscription containing structured JSON. Parent/child relationships are enforced by Ordinals parent/child provenance — creating a child requires spending the parent's UTXO. No external authentication is required.

## Key Features

- **Immutable** — Inscribed directly on Bitcoin. Cannot be altered, deleted, or censored.
- **Hierarchical** — Entity → Collection → Artwork → Events provides organizational chain of trust.
- **Hybrid Title Transfer** — Ownership requires both UTXO transfer and an on-chain Title Event.
- **Structured IP Framework** — Baseline display rights by default. Expanded rights require explicit IP Events.
- **Three-Tier Document Storage** — Bitcoin (hashes) + Arweave (documents) + off-chain (operational).
- **Open Standard** — Any application can implement the ArtCube Protocol.

## Whitepaper

Read the full protocol specification: **[whitepaper.md](./whitepaper.md)**

## JSON Schemas

Machine-readable schemas for all inscription types:

- [`schemas/entity-root.json`](./schemas/entity-root.json)
- [`schemas/collection-parent.json`](./schemas/collection-parent.json)
- [`schemas/genesis.json`](./schemas/genesis.json)
- **Events:**
  - [`schemas/events/title-event.json`](./schemas/events/title-event.json)
  - [`schemas/events/document-event.json`](./schemas/events/document-event.json)
  - [`schemas/events/condition-event.json`](./schemas/events/condition-event.json)
  - [`schemas/events/custody-event.json`](./schemas/events/custody-event.json)
  - [`schemas/events/ip-event.json`](./schemas/events/ip-event.json)
  - [`schemas/events/metadata-update.json`](./schemas/events/metadata-update.json)
  - [`schemas/events/correction-event.json`](./schemas/events/correction-event.json)
  - [`schemas/events/supersession-event.json`](./schemas/events/supersession-event.json)

## Examples

Real-world examples using the Battle of the Centaurs silver sculpture:

- [`examples/entity-root.json`](./examples/entity-root.json) — Trio/BitBasel
- [`examples/collection-parent.json`](./examples/collection-parent.json) — Precious Metal Editions
- [`examples/genesis.json`](./examples/genesis.json) — Silver Battle of the Centaurs
- [`examples/full-chain.md`](./examples/full-chain.md) — Complete provenance chain walkthrough

## Reference Implementation

ArtCube by Trio/BitBasel is the first application built on the ArtCube Protocol.

## License

This specification is released under the [Creative Commons Attribution 4.0 International License (CC-BY-4.0)](./LICENSE).
