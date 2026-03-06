# Full Provenance Chain Walkthrough

This document walks through a complete ArtCube Protocol provenance chain for the **Silver Battle of the Centaurs** — a .999 fine silver sculpture posthumously cast from Michelangelo's original marble relief.

---

## Hierarchy

```
Entity Root: Trio / BitBasel (ORGANIZATION, Florida, USA)
  │
  └─ Collection Parent: Battle of the Centaurs — Precious Metal Editions (EDITION_SERIES)
       │
       └─ Genesis: TIC-SBC-SLV-2022-001 — Silver Battle of the Centaurs
            │
            ├─ Title Event 1: OWNED (initial state)
            ├─ Document Event 1: Assay certificate (PERMANENT_PUBLIC)
            ├─ Document Event 2: Condition report (PERMANENT_PUBLIC)
            ├─ Condition Event 1: Initial inspection
            ├─ Custody Event 1: Stored at Miami Art Storage
            ├─ IP Event 1: High-res publication rights granted
            ├─ Title Event 2: TRANSFERRED to new owner
            └─ Custody Event 2: Transferred to New York vault
```

---

## Step 1: Entity Root

Trio/BitBasel inscribes an Entity Root on Bitcoin establishing their organizational identity.

- **Inscription type:** `ENTITY_ROOT`
- **Entity type:** `ORGANIZATION`
- **Jurisdiction:** Florida, USA
- **Custody:** 2-of-3 multisig
- **No parent** — this is a top-level inscription

See: [`entity-root.json`](./entity-root.json)

---

## Step 2: Collection Parent

Using the Entity Root UTXO as input (parent/child), Trio/BitBasel inscribes a Collection Parent grouping all precious metal editions of the Battle of the Centaurs.

- **Inscription type:** `COLLECTION_PARENT`
- **Collection type:** `EDITION_SERIES`
- **Total editions:** 3 (silver, gold, platinum)
- **Tag 3:** Entity Root inscription ID

See: [`collection-parent.json`](./collection-parent.json)

---

## Step 3: Genesis Title Anchor

Using the Collection Parent UTXO as input (parent/child), the Genesis inscription is created for the silver edition specifically.

- **Inscription type:** `GENESIS_TITLE_ANCHOR`
- **Asset ID:** `TIC-SBC-SLV-2022-001`
- **Tag 3:** Collection Parent inscription ID
- **Owner at genesis:** Silver Battle of the Centaurs, Inc.
- **IP baseline:** DISPLAY_NONCOMMERCIAL, EXHIBIT, PUBLISH_IMAGES_LOWRES

This is the permanent, immutable root of trust for this artwork. It is never modified.

See: [`genesis.json`](./genesis.json)

---

## Step 4: Initial Title Event

The Genesis holder inscribes the first Title Event confirming ownership.

```json
{
  "protocol": "ACP",
  "event_type": "TITLE_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-02-17",
  "previous_event_pointer": null,
  "title_status": "OWNED",
  "current_legal_owner_entity": {
    "name": "Silver Battle of the Centaurs, Inc.",
    "jurisdiction": "Florida, USA"
  },
  "utxo_control_entity": {
    "name": "Silver Battle of the Centaurs, Inc.",
    "address": "bc1q..."
  }
}
```

**Verification:** UTXO holder matches Title Event → title state is **VALID**.

---

## Step 5: Assay Certificate (Document Event)

An assay certificate confirming the silver purity is uploaded to Arweave and anchored on Bitcoin.

```json
{
  "protocol": "ACP",
  "event_type": "DOCUMENT_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-03-01",
  "previous_event_pointer": null,
  "document_type": "ASSAY",
  "document_hash_sha256": "a1b2c3d4...",
  "issuer": "Independent Precious Metals Lab",
  "issued_date": "2026-02-28",
  "storage_policy": "PERMANENT_PUBLIC",
  "arweave_tx_id": "ar_tx_001..."
}
```

**Verification:** Fetch from Arweave → SHA-256 → compare to Bitcoin hash → integrity confirmed.

---

## Step 6: Condition Report (Document Event)

```json
{
  "protocol": "ACP",
  "event_type": "DOCUMENT_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-03-15",
  "previous_event_pointer": "<assay_document_event_id>",
  "document_type": "CONDITION_REPORT",
  "document_hash_sha256": "e5f6a7b8...",
  "issuer": "Art Conservation Services LLC",
  "issued_date": "2026-03-14",
  "storage_policy": "PERMANENT_PUBLIC",
  "arweave_tx_id": "ar_tx_002..."
}
```

Note: `previous_event_pointer` references the assay Document Event — building the per-type linked list.

---

## Step 7: Physical Inspection (Condition Event)

```json
{
  "protocol": "ACP",
  "event_type": "CONDITION_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-03-14",
  "previous_event_pointer": null,
  "inspection_date": "2026-03-14",
  "inspector_entity": "Art Conservation Services LLC",
  "condition_summary": "Excellent condition. No visible damage, tarnishing, or structural compromise.",
  "media_hashes": ["sha256:img001...", "sha256:img002..."]
}
```

---

## Step 8: Storage (Custody Event)

```json
{
  "protocol": "ACP",
  "event_type": "CUSTODY_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-04-01",
  "previous_event_pointer": null,
  "custodian": {
    "name": "Miami Art Storage Inc.",
    "type": "FACILITY"
  },
  "location": {
    "facility": "Climate-Controlled Fine Art Vault",
    "city": "Miami",
    "state": "Florida",
    "country": "USA"
  }
}
```

---

## Step 9: IP Rights Grant

High-resolution reproduction rights granted for a catalogue publication.

```json
{
  "protocol": "ACP",
  "event_type": "IP_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-05-01",
  "previous_event_pointer": null,
  "sub_type": "RIGHTS_GRANT",
  "parties": {
    "grantor": { "name": "Silver Battle of the Centaurs, Inc." },
    "grantee": { "name": "Art Publishing House LLC" }
  },
  "description": "Grant of high-resolution reproduction rights for exhibition catalogue.",
  "rights_granted": ["PUBLISH_IMAGES_HIGHRES", "REPRODUCE_PRINTS"],
  "limitations": {
    "territory": "North America",
    "term": "5 years",
    "exclusivity": "NON_EXCLUSIVE",
    "revocable": "YES"
  }
}
```

---

## Step 10: Title Transfer

The artwork is sold. Both the UTXO and a Title Event record the transfer.

```json
{
  "protocol": "ACP",
  "event_type": "TITLE_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-06-15",
  "previous_event_pointer": "<title_event_1_id>",
  "title_status": "TRANSFERRED",
  "current_legal_owner_entity": {
    "name": "Collector Holdings LLC",
    "jurisdiction": "New York, USA"
  },
  "utxo_control_entity": {
    "name": "Collector Holdings LLC",
    "address": "bc1q_new..."
  },
  "agreement_hash": "c9d0e1f2..."
}
```

The Genesis UTXO is simultaneously transferred to the new owner's address. Both conditions of the hybrid transfer model are satisfied → title state is **VALID**.

---

## Step 11: New Custody Location

The new owner moves the artwork to their vault.

```json
{
  "protocol": "ACP",
  "event_type": "CUSTODY_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_id>",
  "effective_date": "2026-06-20",
  "previous_event_pointer": "<custody_event_1_id>",
  "custodian": {
    "name": "Collector Holdings LLC",
    "type": "OWNER"
  },
  "location": {
    "facility": "Private Collection Vault",
    "city": "New York",
    "state": "New York",
    "country": "USA"
  }
}
```

---

## Verification

Any third party can verify this entire chain:

1. Look up the Genesis inscription on Bitcoin
2. Follow parent pointer up to Collection Parent → Entity Root
3. Confirm hierarchy is intact and not burned
4. Fetch all child inscriptions of Genesis
5. Reconstruct each event-type linked list
6. Confirm latest Title Event aligns with current UTXO holder
7. For document events: fetch from Arweave, hash, compare to Bitcoin hash

The entire history is permanent, publicly verifiable, and cannot be altered by any party.
