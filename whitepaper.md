# The ArtCube Protocol

**A Standardized Digital Provenance & Ownership Framework for Art**

Scott Spiegel, Antonio Vasaiely (BitBasel), Brian Laughlan (Trio)

Version 1.0 — Working Draft | February 2026

---

## Abstract

The ArtCube Protocol defines an open standard for anchoring artwork identity, provenance, ownership, and intellectual property rights to Bitcoin using Ordinals inscriptions. The ArtCube Protocol uses a hierarchical parent/child inscription structure — Entity Root, Collection Parent, Genesis Title Anchor, and append-only Events — to create tamper-proof, publicly verifiable provenance chains for physical and digital artworks. The protocol leverages Bitcoin's immutability and Ordinals parent/child provenance to enforce authorization at every level: only the holder of a parent inscription's UTXO can create children. This paper specifies the complete protocol, including JSON schemas, the hybrid title transfer model, an intellectual property framework, a three-tier document storage layer, and a verification algorithm.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Background](#2-background)
3. [Solution — High-Level Overview](#3-solution--high-level-overview)
4. [Technical Details](#4-technical-details)
5. [Verification Algorithm](#5-verification-algorithm)
6. [Reference Implementation](#6-reference-implementation)
7. [Considerations](#7-considerations)
8. [Alternatives](#8-alternatives)
9. [Conclusion](#9-conclusion)

---

## 1. Introduction

Provenance — the documented history of an artwork's origin, ownership, and condition — is the foundation of trust in the art market. Yet provenance today relies on paper certificates, gallery records, and centralized databases that can be forged, lost, or destroyed. There is no standardized, tamper-proof, publicly verifiable system for recording and retrieving artwork provenance.

The ArtCube Protocol solves this by inscribing structured provenance data directly on Bitcoin using the Ordinals protocol. Two core ideas underpin the design:

1. **Bitcoin as the canonical registrar.** Bitcoin is the most secure, permanent computing network in existence. Data inscribed on Bitcoin cannot be altered, deleted, or censored. This makes it the ideal registrar for artwork identity and provenance.

2. **Hierarchical parent/child inscriptions create a verifiable chain of trust.** Using Ordinals parent/child provenance, the ArtCube Protocol defines a four-level hierarchy: an organization's Entity Root contains Collection Parents, which contain Genesis Title Anchors for individual artworks, which contain append-only Events recording ownership changes, condition reports, documents, and IP rights. Each level is cryptographically linked to its parent — creating a child requires spending the parent's UTXO.

For collectors, galleries, and institutions, the ArtCube Protocol provides a permanent, publicly verifiable record that no single party controls. For the art market, it provides a shared standard that any application can implement.

> **Note:** This specification is a working draft. Field names, event schemas, and protocol rules may change before the first stable release.

---

## 2. Background

### 2.1 The Provenance Problem

The global art market exceeds $65 billion annually. Forgery and provenance fraud remain pervasive — estimates suggest that up to 50% of artworks in circulation may have incomplete or falsified provenance records. Traditional provenance documentation consists of paper certificates, gallery receipts, auction catalogues, and expert opinions. These records are easily forged, frequently lost in ownership transitions, and impossible to verify without contacting the issuing party.

No standardized, tamper-proof, publicly verifiable provenance system exists today. The art market lacks the equivalent of a land title registry — a shared, authoritative source of truth that any party can independently verify.

### 2.2 Bitcoin as a Registrar

The Ordinals protocol, introduced by Casey Rodarmor in 2023, enables arbitrary data to be inscribed directly on Bitcoin. Each inscription is immutable once confirmed in a block. The protocol assigns ordinal numbers to individual satoshis, allowing inscriptions to be tracked, transferred, and verified.

Critically for the ArtCube Protocol, Ordinals supports **parent/child inscriptions** via parent/child provenance. To create a child inscription, the parent inscription's UTXO must be spent as an input in the child's reveal transaction. This means:

- Only the current holder of a parent UTXO can create children
- The parent/child relationship is cryptographically enforced by Bitcoin consensus
- No external authentication system is required

Bitcoin provides 16+ years of continuous uptime, the highest hash rate of any blockchain, and proven resistance to censorship and modification. These properties make it uniquely suited as a permanent registrar.

### 2.3 Current Approaches and Their Limitations

**Certificate databases** (e.g., gallery and auction house records) are centralized. The issuing entity may cease operations, databases may be compromised, and records cannot be independently verified without the issuer's cooperation.

**NFT-based provenance on Ethereum and Solana** suffers from mutable metadata (token URIs can be changed), dependency on IPFS or centralized hosting for off-chain data, and uncertainty about the long-term viability of these chains.

**C2PA** (Coalition for Content Provenance and Authenticity) embeds provenance metadata in file headers. This metadata is easily stripped by social media platforms, image processors, and file converters. It has no blockchain anchor and no mechanism for ongoing provenance updates.

**Traditional paper provenance** is the weakest form — easily forged, lost, or destroyed. Authentication depends entirely on the reputation of the issuing expert.

None of these approaches combine immutability, hierarchical organization, a legal framework, and permanent document storage in a single open standard.

---

## 3. Solution — High-Level Overview

The ArtCube Protocol defines a hierarchical inscription structure with four levels:

```
Entity Root              → Organization identity (optional, strongly recommended)
  └─ Collection Parent   → Group of related artworks (optional, strongly recommended)
       └─ Genesis        → Individual artwork identity + legal anchor (required)
            └─ Events    → Append-only provenance chain (required)
```

Each level is a Bitcoin inscription containing structured JSON. The parent/child relationship at every level is enforced by Ordinals parent/child provenance — creating a child inscription requires spending the parent inscription's UTXO. This means only the current holder of a parent can extend the chain below it.

**Events** are always children of their Genesis inscription. There are eight event types, each forming its own linked list via `previous_event_pointer`: Title Events, Document Events, Condition Events, Custody Events, IP Events, Metadata Updates, Correction Events, and Supersession Events.

The top two levels (Entity Root and Collection Parent) are optional but strongly recommended. They provide organizational context that strengthens the provenance chain. A standalone Genesis inscription (with no parent hierarchy) is a valid ArtCube Protocol record.

**Burning** a parent permanently closes that level. Burning a Collection Parent means no new artworks can be added to the collection. Burning an Entity Root means no new collections can be created under that entity. Existing provenance chains below a burned parent remain fully valid and verifiable.

**Hybrid Title Transfer** requires two conditions for canonical ownership recognition: (1) the Genesis UTXO must be transferred to the new controlling address, and (2) a Title Event must be appended recording the transfer. Neither condition alone is sufficient. If they are misaligned, title state is considered indeterminate.

---

## 4. Technical Details

### 4.1 Protocol Identifiers

| Field | Value |
|-------|-------|
| Protocol identifier (JSON) | `"artcube"` |
| Metaprotocol (envelope tag 7) | `artcube` |
| Content-Type (envelope tag 1) | `application/json` |
| Metadata (envelope tag 5) | CBOR — human-readable summary |
| Properties (envelope tag 17) | CBOR — structured title and traits |
| Properties encoding (envelope tag 19) | `"cbor"` |
| Version (JSON) | `"1.0"` |

**Metaprotocol field (required).** Every ArtCube Protocol inscription MUST include the Ordinals metaprotocol field (tag `7`) with the value `artcube` in the inscription envelope. This is a protocol-level requirement defined by the [Ordinals specification](https://docs.ordinals.com/inscriptions.html#fields). The metaprotocol field allows indexers and explorers to identify ArtCube Protocol inscriptions without parsing the JSON body. It is set at the envelope level during inscription creation — it is not a JSON field.

**Example inscription envelope structure:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1        (content-type)
  OP_PUSH "application/json"
  OP_PUSH 5        (metadata — CBOR)
  OP_PUSH <cbor metadata>
  OP_PUSH 7        (metaprotocol)
  OP_PUSH "artcube"
  OP_PUSH 17       (properties — CBOR)
  OP_PUSH <cbor properties>
  OP_PUSH 19       (properties encoding)
  OP_PUSH "cbor"
  OP_PUSH 0        (body separator)
  OP_PUSH <json payload>
OP_ENDIF
```

The `"protocol": "artcube"` field inside the JSON body serves as a secondary identifier for applications that parse the inscription content directly. Both identifiers — the envelope metaprotocol tag and the JSON protocol field — MUST be present on every ArtCube Protocol inscription.

#### 4.1.1 Metadata (Tag 5)

Every ArtCube Protocol inscription SHOULD include a **metadata** field (envelope tag `5`) containing a CBOR-encoded map of human-readable summary fields. This allows generic Ordinals explorers to display meaningful information about the inscription without parsing the JSON body.

Metadata is a free-form key-value map with string keys. The ArtCube Protocol defines the following conventions:

| Key | Present on | Value |
|-----|-----------|-------|
| `p` | All | `"artcube"` |
| `type` | All | Inscription type (e.g. `"ENTITY_ROOT"`, `"GENESIS_TITLE_ANCHOR"`) |
| `name` | Entity Root | Entity name |
| `entity_type` | Entity Root | Entity type (e.g. `"ORGANIZATION"`) |
| `collection` | Collection Parent | Collection name |
| `artist` | Collection Parent, Genesis | Artist name |
| `title` | Genesis | Artwork title |
| `medium` | Genesis | Medium description |
| `owner` | Genesis | Legal owner at genesis |
| `event` | Events | Event type (e.g. `"TITLE_EVENT"`) |
| `status` | Title Event | Title status (e.g. `"OWNED"`) |
| `document_type` | Document Event | Document type (e.g. `"ASSAY"`) |
| `custodian` | Custody Event | Custodian name |
| `sub_type` | IP Event | IP event subtype |

**Example metadata for a Genesis inscription:**

```json
{
  "p": "artcube",
  "type": "GENESIS_TITLE_ANCHOR",
  "title": "Battle of the Centaurs",
  "artist": "Michelangelo Buonarroti",
  "medium": ".999 Fine Silver",
  "owner": "Silver Battle of the Centaurs, Inc."
}
```

#### 4.1.2 Properties (Tag 17)

Every ArtCube Protocol inscription SHOULD include **properties** (envelope tag `17`) containing CBOR-encoded structured attributes. Properties follow the Ordinals specification schema using integer keys, enabling marketplace filtering and trait-based discovery.

When properties are CBOR-encoded, the **properties encoding** tag (`19`) MUST be set to `"cbor"`.

The properties schema uses integer keys as defined by the Ordinals specification:

```
Properties = { 1: Attributes }
Attributes = { 0: <title>, 1: <traits> }
```

- **Title** (Attributes key `0`): A human-readable title for the inscription. See the table below for conventions per inscription type.
- **Traits** (Attributes key `1`): A map of string key-value pairs representing filterable attributes.

**Title conventions:**

| Inscription type | Title format | Example |
|-----------------|-------------|---------|
| Entity Root | Entity name | `"ArtCube"` |
| Collection Parent | Collection name | `"Eternal Creations — Michelangelo Posthumous Originals"` |
| Genesis | Artwork title | `"Battle of the Centaurs"` |
| Title Event | `"Title Event — {status}"` | `"Title Event — OWNED"` |
| Document Event | `"Document Event — {document_type}"` | `"Document Event — ASSAY"` |
| Condition Event | `"Condition Event — {date}"` | `"Condition Event — 2026-03-14"` |
| Custody Event | `"Custody Event — {custodian}"` | `"Custody Event — Miami Art Storage Inc."` |
| IP Event | `"IP Event — {sub_type}"` | `"IP Event — RIGHTS_GRANT"` |

**Trait conventions:**

All ArtCube Protocol inscriptions SHOULD include a `"Protocol"` trait with value `"artcube"` and a `"Type"` trait with the inscription type. Additional traits vary by inscription type:

| Trait key | Present on | Value |
|-----------|-----------|-------|
| `Protocol` | All | `"artcube"` |
| `Type` | All | Inscription type (e.g. `"Genesis"`, `"Entity Root"`) |
| `Entity` | Entity Root | Entity name |
| `Jurisdiction` | Entity Root, Genesis | Jurisdiction |
| `Custody Model` | Entity Root | `"MULTISIG"` or `"SINGLE_SIG"` |
| `Collection` | Collection Parent | Collection name |
| `Collection Type` | Collection Parent | e.g. `"EDITION_SERIES"` |
| `Artist` | Collection Parent, Genesis | Artist name |
| `Medium` | Collection Parent, Genesis | Medium |
| `Title` | Genesis | Artwork title |
| `Period` | Genesis | Date or period of original work |
| `Owner` | Genesis | Legal owner name |
| `Status` | Title Event | e.g. `"OWNED"`, `"TRANSFERRED"` |
| `Document` | Document Event | e.g. `"ASSAY"`, `"APPRAISAL"` |
| `Custodian` | Custody Event | Custodian name |
| `Location` | Custody Event | City/country |

**Example properties for a Genesis inscription (conceptual JSON representation of CBOR):**

```json
{
  "1": {
    "0": "Battle of the Centaurs",
    "1": {
      "Protocol": "artcube",
      "Type": "Genesis",
      "Artist": "Michelangelo Buonarroti",
      "Medium": ".999 Fine Silver",
      "Period": "c.1490–1492",
      "Owner": "Silver Battle of the Centaurs, Inc.",
      "Collection": "Eternal Creations"
    }
  }
}
```

#### 4.1.3 Envelope Layer Summary

The three envelope layers serve distinct purposes:

| Layer | Tag | Format | Purpose | Consumer |
|-------|-----|--------|---------|----------|
| Body | (after tag 0) | JSON | Canonical protocol data — source of truth | ArtCube Protocol validators |
| Metadata | 5 | CBOR | Human-readable summary | Generic explorers (ordiscan, etc.) |
| Properties | 17 | CBOR | Structured title and filterable traits | Marketplaces (Magic Eden, etc.) |

The JSON body is the canonical data. Metadata and properties are **derived summaries** — they make ArtCube inscriptions discoverable in the existing Ordinals ecosystem without requiring ArtCube-specific tooling. If there is any conflict between the body and envelope fields, the JSON body takes precedence.

---

### 4.2 Parent/Child Inscriptions

The ArtCube Protocol's authorization model is built on the Ordinals **parent/child provenance** mechanism. This section explains how it works at the Bitcoin transaction level, how the protocol uses it, and how it differs from the application-level `previous_event_pointer` linked list.

#### How parent/child works

Every Ordinals inscription is bound to a specific UTXO. To create a **child** inscription of a parent, the parent inscription's UTXO must be included as an input in the child's **reveal transaction**. The parent UTXO is not consumed — it is returned to the creator in the same transaction (typically to the same address). The child inscription is permanently and immutably linked to the parent by the Ordinals indexer.

This relationship is:
- **Immutable** — once confirmed, the parent/child link cannot be changed or revoked
- **Consensus-enforced** — Bitcoin validates that the parent UTXO was spent in the reveal transaction; no external system is involved
- **Permissioned** — only the party that controls the parent UTXO (holds the private key or satisfies the multisig threshold) can create children

#### How the ArtCube Protocol uses parent/child

The four-level hierarchy maps directly to Ordinals parent/child:

| Inscription | Parent | Created by |
|-------------|--------|------------|
| Entity Root | None (top-level) | Entity creator |
| Collection Parent | Entity Root | Entity Root UTXO holder |
| Genesis | Collection Parent | Collection Parent UTXO holder |
| Events | Genesis | Genesis UTXO holder |

When a verifier reads an inscription, they can query the Ordinals indexer for its parent. This returns the parent inscription ID, which can be followed up the chain to reconstruct the full hierarchy. No trust in the JSON content is required for hierarchy verification — it is independently verifiable from the Bitcoin transaction graph.

#### How to verify parent/child

Any Ordinals indexer (e.g., `ord`, Hiro, Bestinslot) exposes the parent of an inscription. Verification is straightforward:

1. Fetch the inscription metadata from the indexer
2. Read the `parent` field — this is the parent inscription ID
3. Fetch the parent inscription and verify its `inscription_type` matches the expected level
4. Repeat up the chain to the Entity Root (or until no parent exists)

The parent/child relationship is a fact of the Bitcoin transaction graph, not a claim in the JSON body. This is what makes the ArtCube Protocol's authorization trustless.

#### Parent/child vs. `previous_event_pointer`

These serve different purposes and should not be confused:

| | Parent/child (Ordinals) | `previous_event_pointer` (JSON field) |
|---|---|---|
| **Enforced by** | Bitcoin consensus | Application-level convention |
| **Purpose** | Authorization — proves who created the inscription | Ordering — links events into per-type chains |
| **Scope** | Vertical (hierarchy levels) | Horizontal (within a single event type) |
| **Trustless** | Yes — verifiable from transaction graph | Advisory — aids verification but not consensus-enforced |
| **Example** | Genesis is a child of Collection Parent | Title Event 2 points to Title Event 1 |

Parent/child answers: "Was this inscription authorized?" `previous_event_pointer` answers: "What is the order of events within this category?"

### 4.3 Inscription Hierarchy

#### Level 0: Entity Root (optional, strongly recommended)

An Entity Root inscription establishes an organization's identity on Bitcoin. It is a top-level inscription with no parent.

**Governance** declares how the Entity Root UTXO is held. This matters because whoever controls the Entity Root UTXO has exclusive write access to create new Collection Parents under that entity (via Ordinals parent/child). The `governance` field is a transparency declaration — it tells verifiers the custody model (single-key or multisig) and, for multisig wallets, the signer threshold (e.g., 2-of-3). The protocol does not enforce the custody model itself — Bitcoin consensus and the wallet enforce it. The governance field makes the security posture publicly auditable so that collectors, institutions, and verifiers can assess the trust level of the entity.

**Schema:**

```json
{
  "protocol": "artcube",
  "inscription_type": "ENTITY_ROOT",
  "version": "1.0",
  "entity": {
    "name": "string (required)",
    "type": "ORGANIZATION | INDIVIDUAL | ESTATE | TRUST | SPV | DAO",
    "founding_entities": "string[] (optional — organizations or individuals that created this entity)",
    "jurisdiction": "string (required)",
    "description": "string (optional)",
    "website": "string (optional)",
    "contact": "string (optional)"
  },
  "governance": {
    "utxo_custody": "MULTISIG | SINGLE_SIG",
    "authorized_signers": "number (optional)",
    "required_signers": "number (optional)"
  },
  "burn_rule": "Burning this Entity Root permanently closes the entity. No new Collection Parents may be created. Existing collections and provenance chains remain valid."
}
```

**Entity types:**

| Type | Description |
|------|-------------|
| `ORGANIZATION` | Company, gallery, museum, foundation |
| `INDIVIDUAL` | Private collector or artist |
| `ESTATE` | Artist or collector estate |
| `TRUST` | Legal trust structure |
| `SPV` | Special purpose vehicle — a legal entity created for a specific asset or transaction (e.g., holding title to a single high-value artwork) |
| `DAO` | Decentralized autonomous organization |

**Example — ArtCube:**

```json
{
  "protocol": "artcube",
  "inscription_type": "ENTITY_ROOT",
  "version": "1.0",
  "entity": {
    "name": "ArtCube",
    "type": "ORGANIZATION",
    "founding_entities": ["BitBasel", "Trio"],
    "jurisdiction": "Florida, USA",
    "description": "Art technology and exhibition platform specializing in Bitcoin-native provenance for physical artworks."
  },
  "governance": {
    "utxo_custody": "MULTISIG",
    "authorized_signers": 3,
    "required_signers": 2
  },
  "burn_rule": "Burning this Entity Root permanently closes the entity. No new Collection Parents may be created. Existing collections and provenance chains remain valid."
}
```

#### Level 1: Collection Parent (optional, strongly recommended)

A Collection Parent groups related artworks. It is a child of an Entity Root inscription (parent → Entity Root inscription ID).

**Schema:**

```json
{
  "protocol": "artcube",
  "inscription_type": "COLLECTION_PARENT",
  "version": "1.0",
  "collection": {
    "name": "string (required)",
    "type": "EDITION_SERIES | ONE_OF_ONE | OPEN_EDITION | CURATED_SET | EXHIBITION | ARCHIVE",
    "description": "string (optional)",
    "artist": {
      "name": "string (required)",
      "role": "string (optional)"
    },
    "medium": "string (optional)",
    "edition_info": {
      "total_editions": "number (optional)",
      "edition_notes": "string (optional)"
    }
  },
  "burn_rule": "Burning this Collection Parent permanently closes the collection. No new Genesis inscriptions may be created under it. Existing provenance chains remain valid."
}
```

**Collection types:**

| Type | Description |
|------|-------------|
| `EDITION_SERIES` | Numbered editions of the same work |
| `ONE_OF_ONE` | Single unique artwork |
| `OPEN_EDITION` | Unlimited edition |
| `CURATED_SET` | Curated group of distinct works |
| `EXHIBITION` | Works associated with a specific exhibition |
| `ARCHIVE` | Archival collection of documents or works |

**Example — Battle of the Centaurs Collection:**

```json
{
  "protocol": "artcube",
  "inscription_type": "COLLECTION_PARENT",
  "version": "1.0",
  "collection": {
    "name": "Battle of the Centaurs — Precious Metal Editions",
    "type": "EDITION_SERIES",
    "description": "Posthumous original castings from Michelangelo's Battle of the Centaurs marble relief (c.1490-1492), executed in precious metals by Treasure Investment Corporation",
    "artist": {
      "name": "Michelangelo Buonarroti",
      "role": "Original composition"
    },
    "medium": "Precious metal castings (silver, gold, platinum)",
    "edition_info": {
      "total_editions": 3,
      "edition_notes": "One edition each in .999 fine silver, .999 fine gold, and .999 fine platinum"
    }
  },
  "burn_rule": "Burning this Collection Parent permanently closes the collection. No new Genesis inscriptions may be created under it. Existing provenance chains remain valid."
}
```

#### Level 2: Genesis Title Anchor (required)

The Genesis inscription is the immutable root of trust for an individual artwork. One Genesis exists per artwork. It is a child of a Collection Parent (parent → Collection Parent inscription ID), or a standalone inscription if no hierarchy is used.

Genesis is **never modified** after inscription. All subsequent changes are recorded as child Events.

The `object_id_core` section is based on the [Object ID](https://www.getty.edu/publications/virtuallibrary/0892365722.html) standard developed by the J. Paul Getty Trust and adopted by ICOM, UNESCO, Interpol, and the FBI. Object ID defines the minimum fields needed to uniquely identify a physical artwork. The ArtCube Protocol inscribes these fields directly on Bitcoin, making them permanent and publicly verifiable.

**Schema:**

```json
{
  "protocol": "artcube",
  "inscription_type": "GENESIS_TITLE_ANCHOR",
  "version": "1.0",

  "record": {
    "asset_id": "string (required)",
    "object_id_record_date": "YYYY-MM-DD (required)"
  },

  "object_id_core": {
    "type_of_object": "string (required)",
    "title": "string (required)",
    "subject": "string (required)",
    "date_or_period": "object (required)",
    "maker": "object (required)",
    "materials_and_techniques": "object (required)",
    "measurements": "object (required)",
    "inscriptions_and_markings": "object (optional)",
    "distinguishing_features": "object (optional)"
  },

  "ownership_at_genesis": {
    "legal_owner_entity": {
      "name": "string (required)",
      "jurisdiction": "string (required)"
    },
    "physical_custodian": {
      "name": "string (required)",
      "location": "string (required)"
    }
  },

  "legal_title_anchor": {
    "title_anchor": true,
    "governing_law": "string (required)",
    "hybrid_transfer_rule": "Canonical title recognition requires both (i) transfer of the Genesis inscription UTXO to the new lawful authority and (ii) an appended Title Event recording the transfer.",
    "statement": "This Genesis inscription serves as the canonical Bitcoin provenance and legal title anchor for the physical artwork identified herein."
  },

  "intellectual_property": {
    "copyright": {
      "status": "OWNED | LICENSED | PUBLIC_DOMAIN | UNKNOWN | DISPUTED",
      "copyright_owner": { "name": "string", "type": "string", "jurisdiction": "string" },
      "copyright_year": "number or null",
      "registration": { "registered": "string", "registration_id": "string or null", "jurisdiction": "string or null" }
    },
    "moral_rights": {
      "waived": "YES | NO | UNKNOWN",
      "attribution_required": "YES | NO | UNKNOWN",
      "integrity_rights_notes": "string or null"
    },
    "rights_grants": {
      "baseline_policy": "BASELINE_DISPLAY_RIGHTS",
      "granted_to_holder": {
        "holder_scope": "ALIGNED_HOLDER",
        "rights": ["DISPLAY_NONCOMMERCIAL", "EXHIBIT", "PUBLISH_IMAGES_LOWRES"],
        "status": "GRANTED",
        "limitations": {
          "territory": "WORLDWIDE",
          "term": "PERPETUAL",
          "exclusivity": "NON_EXCLUSIVE",
          "revocable": "NO"
        }
      },
      "not_granted_by_default": [
        "PUBLISH_IMAGES_HIGHRES",
        "REPRODUCE_PRINTS",
        "MERCHANDISING",
        "COMMERCIALIZE",
        "CREATE_DERIVATIVES",
        "AI_TRAINING_ALLOWED",
        "AI_GENERATION_ALLOWED",
        "SUBLICENSE"
      ]
    },
    "digital_asset_rights": "object (optional)",
    "royalties": "object (optional)",
    "disclosures": "object (optional)"
  }
}
```

**All unknown fields must be explicitly marked `UNKNOWN`.** This prevents ambiguity — an absent field and an unknown field have different meanings.

**Example — Silver Battle of the Centaurs:**

```json
{
  "protocol": "artcube",
  "inscription_type": "GENESIS_TITLE_ANCHOR",
  "version": "1.0",
  "record": {
    "asset_id": "TIC-SBC-SLV-2022-001",
    "object_id_record_date": "2026-02-17"
  },
  "object_id_core": {
    "type_of_object": "Sculpture — High Relief Plaque (Posthumous Original Casting), One-of-One Silver Edition",
    "title": "Battle of the Centaurs",
    "subject": "Mythological battle scene (Lapiths vs. Centaurs) depicted in dense multi-figure combat relief.",
    "date_or_period": {
      "original_work": "c.1490–1492",
      "silver_casting": "2022"
    },
    "maker": {
      "original_artist": {
        "name": "Michelangelo Buonarroti",
        "role": "Original composition"
      },
      "wax_lineage_association": {
        "name": "Fonderia Artistica Ferdinando Marinelli",
        "location": "Florence, Italy",
        "role": "Wax stage / historic master mold lineage association"
      },
      "metal_casting_executor": {
        "name": "Treasure Investment Corporation (d/b/a Foundry Michelangelo)",
        "location": "Battle Ground, Washington, USA",
        "role": "Final .999 silver casting / metal pour",
        "year": 2022
      }
    },
    "materials_and_techniques": {
      "materials": [{ "name": ".999 Fine Silver", "confirmed": true }],
      "techniques": [{ "name": "Lost-wax casting (cire perdue)", "confirmed": true }]
    },
    "measurements": {
      "dimensions_cm": { "height": 80.0, "width": 88.9, "depth": 11.4 },
      "weight_kg": 72.57
    }
  },
  "ownership_at_genesis": {
    "legal_owner_entity": {
      "name": "Silver Battle of the Centaurs, Inc.",
      "jurisdiction": "Florida, USA"
    },
    "physical_custodian": {
      "name": "Treasure Investment Corporation",
      "location": "Battle Ground, Washington, USA"
    }
  },
  "legal_title_anchor": {
    "title_anchor": true,
    "governing_law": "Florida, USA",
    "hybrid_transfer_rule": "Canonical title recognition requires both (i) transfer of the Genesis inscription UTXO to the new lawful authority and (ii) an appended Title Event recording the transfer.",
    "statement": "This Genesis inscription serves as the canonical Bitcoin provenance and legal title anchor for the physical artwork identified herein."
  },
  "intellectual_property": {
    "copyright": { "status": "UNKNOWN" },
    "moral_rights": {
      "waived": "UNKNOWN",
      "attribution_required": "YES",
      "integrity_rights_notes": null
    },
    "rights_grants": {
      "baseline_policy": "BASELINE_DISPLAY_RIGHTS",
      "granted_to_holder": {
        "holder_scope": "ALIGNED_HOLDER",
        "rights": ["DISPLAY_NONCOMMERCIAL", "EXHIBIT", "PUBLISH_IMAGES_LOWRES"],
        "status": "GRANTED",
        "limitations": {
          "territory": "WORLDWIDE",
          "term": "PERPETUAL",
          "exclusivity": "NON_EXCLUSIVE",
          "revocable": "NO"
        }
      },
      "not_granted_by_default": [
        "PUBLISH_IMAGES_HIGHRES", "REPRODUCE_PRINTS", "MERCHANDISING",
        "COMMERCIALIZE", "CREATE_DERIVATIVES", "AI_TRAINING_ALLOWED",
        "AI_GENERATION_ALLOWED", "SUBLICENSE"
      ]
    }
  }
}
```

#### Level 3: Events (required, append-only children of Genesis)

Events are child inscriptions of the Genesis inscription (parent → Genesis inscription ID). Each event forms a per-category linked list via the `previous_event_pointer` field, which references the previous event **of the same type**. For the first event of a given type, this field is `null`.

**Chain ordering and fork resolution.** The `previous_event_pointer` linked list is advisory — it aids verification but is not consensus-enforced. If two events claim the same `previous_event_pointer` (a fork), the canonical event is the one confirmed **earliest by block height**. If both appear in the same block, the event with the **lower transaction index** within that block takes precedence. The later event is considered orphaned and should be ignored by verifiers.

**Common event fields** (required in every event):

```json
{
  "protocol": "artcube",
  "event_type": "EVENT_TYPE_HERE",
  "version": "1.0",
  "parent_genesis_inscription": "<genesis_inscription_id>",
  "effective_date": "YYYY-MM-DD",
  "previous_event_pointer": "<inscription_id or null>"
}
```

The ArtCube Protocol defines eight event types:

| Event Type | Purpose |
|-----------|---------|
| `TITLE_EVENT` | Records ownership state changes |
| `DOCUMENT_EVENT` | Records existence of off-chain documents |
| `CONDITION_EVENT` | Records physical condition of artwork |
| `CUSTODY_EVENT` | Records physical location changes |
| `IP_EVENT` | Records intellectual property rights evolution |
| `METADATA_UPDATE` | Records corrections or additions to non-critical metadata |
| `CORRECTION_EVENT` | Fixes minor errors in previous events |
| `SUPERSESSION_EVENT` | Replaces a Genesis inscription due to material identity error |

Each event type is specified in detail below.

---

### 4.4 Event Type Specifications

#### 4.4.1 Title Event

Records ownership state changes. See Section 4.5 for the Hybrid Title Transfer model.

**Allowed status values:** `OWNED` | `TRANSFERRED` | `PLEDGED` | `RELEASED` | `CORRECTED`

**Required fields:**
- `title_status` — one of the allowed status values
- `effective_date` — date the change took effect
- `current_legal_owner_entity` — object with `name` and `jurisdiction`
- `utxo_control_entity` — object with `name` and `address`
- `previous_event_pointer` — inscription ID of previous Title Event, or `null`

**Optional fields:**
- `agreement_hash` — SHA-256 hash of the transfer agreement document

**Example:**

```json
{
  "protocol": "artcube",
  "event_type": "TITLE_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-03-15",
  "previous_event_pointer": null,
  "title_status": "TRANSFERRED",
  "current_legal_owner_entity": {
    "name": "New Owner LLC",
    "jurisdiction": "Delaware, USA"
  },
  "utxo_control_entity": {
    "name": "New Owner LLC",
    "address": "bc1q..."
  },
  "agreement_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
}
```

#### 4.4.2 Document Event

Records the existence and integrity of off-chain documents. Documents are stored on Arweave (public or encrypted) or kept entirely off-platform. Bitcoin always stores the SHA-256 hash as the integrity proof.

**Required fields:**
- `document_type` — one of: `APPRAISAL` | `INSURANCE` | `ASSAY` | `LEGAL_AGREEMENT` | `LICENSE` | `CONDITION_REPORT` | `PROVENANCE_CERTIFICATE` | `TRANSFER_AGREEMENT` | `AUTHENTICATION`
- `document_hash_sha256` — SHA-256 hash of the **original plaintext** document
- `issuer` — entity that created the document
- `issued_date` — date the document was issued
- `storage_policy` — `PERMANENT_PUBLIC` | `PERMANENT_PRIVATE` | `HASH_ONLY`

**Optional fields (when using Arweave):**
- `arweave_tx_id` — Arweave transaction ID for document retrieval
- `encryption_method` — e.g. `AES-256-GCM` (required when `PERMANENT_PRIVATE`)

**Example:**

```json
{
  "protocol": "artcube",
  "event_type": "DOCUMENT_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-04-01",
  "previous_event_pointer": null,
  "document_type": "CONDITION_REPORT",
  "document_hash_sha256": "a1b2c3d4e5f6...",
  "issuer": "Art Conservation Services LLC",
  "issued_date": "2026-03-28",
  "storage_policy": "PERMANENT_PUBLIC",
  "arweave_tx_id": "xyz789..."
}
```

#### 4.4.3 Condition Event

Records physical condition of artwork at a point in time.

**Required fields:**
- `inspection_date` — date of physical inspection
- `inspector_entity` — entity that performed the inspection
- `condition_summary` — text summary of condition

**Optional fields:**
- `damage_or_repairs` — array of damage/repair records
- `media_hashes` — array of SHA-256 hashes of inspection photos (photos stored on Arweave)

**Example:**

```json
{
  "protocol": "artcube",
  "event_type": "CONDITION_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-05-10",
  "previous_event_pointer": null,
  "inspection_date": "2026-05-10",
  "inspector_entity": "Fine Art Conservation Group",
  "condition_summary": "Excellent condition. No visible damage, tarnishing, or structural compromise. Surface patina consistent with age.",
  "damage_or_repairs": [],
  "media_hashes": [
    "sha256:abc123...",
    "sha256:def456..."
  ]
}
```

#### 4.4.4 Custody Event

Records physical location changes. **Custody does not imply title transfer.** An artwork may be loaned, exhibited, or stored with a third party without any change of ownership.

**Required fields:**
- `custodian` — entity holding the artwork
- `location` — physical location
- `effective_date` — date custody began

**Optional fields:**
- `authorization_reference` — reference to loan agreement or custody authorization

**Example:**

```json
{
  "protocol": "artcube",
  "event_type": "CUSTODY_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-06-01",
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
  },
  "authorization_reference": "Loan agreement hash: sha256:789abc..."
}
```

#### 4.4.5 IP Event

Records intellectual property rights evolution. IP Events use a `sub_type` field to distinguish between six categories. **IP rights changes do not imply title transfer.** They are separate concerns recorded in separate event streams.

**Sub-types:**

| Sub-type | Purpose |
|----------|---------|
| `IP_ATTESTATION` | Declares or confirms IP status |
| `RIGHTS_GRANT` | Grants specific commercial/usage rights |
| `LICENSE` | Records a licensing agreement |
| `RIGHTS_CHANGE` | Modifies existing rights |
| `REVOCATION` | Revokes previously granted rights |
| `DISPUTE` | Records an IP dispute |

**Required fields (all sub-types):**
- `sub_type` — one of the six sub-types above
- `effective_date`
- `parties` — entities involved
- `description` — human-readable description of the IP action

**Additional fields vary by sub-type:**

**RIGHTS_GRANT additional fields:**
- `rights_granted` — array of rights (e.g. `REPRODUCE_PRINTS`, `MERCHANDISING`)
- `limitations` — territory, term, exclusivity, revocable

**LICENSE additional fields:**
- `license_type` — e.g. `EXCLUSIVE`, `NON_EXCLUSIVE`
- `license_hash` — SHA-256 hash of the license agreement
- `licensed_rights` — array of rights covered

**REVOCATION additional fields:**
- `revoked_rights` — array of rights being revoked
- `revocation_reason` — text explanation
- `original_grant_pointer` — inscription ID of the original grant being revoked

**Example — Rights Grant:**

```json
{
  "protocol": "artcube",
  "event_type": "IP_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-07-01",
  "previous_event_pointer": null,
  "sub_type": "RIGHTS_GRANT",
  "parties": {
    "grantor": { "name": "Silver Battle of the Centaurs, Inc." },
    "grantee": { "name": "Art Publishing House LLC" }
  },
  "description": "Grant of high-resolution reproduction rights for catalogue publication.",
  "rights_granted": ["PUBLISH_IMAGES_HIGHRES", "REPRODUCE_PRINTS"],
  "limitations": {
    "territory": "North America",
    "term": "5 years",
    "exclusivity": "NON_EXCLUSIVE",
    "revocable": "YES"
  }
}
```

#### 4.4.6 Metadata Update

Records corrections or additions to non-critical metadata. Used for updated measurements, new photographs, or other factual additions that do not affect the artwork's identity.

**Required fields:**
- `updated_fields` — object containing the fields being updated with their new values
- `reason` — text explanation of why the update is needed

**Example:**

```json
{
  "protocol": "artcube",
  "event_type": "METADATA_UPDATE",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-08-15",
  "previous_event_pointer": null,
  "updated_fields": {
    "measurements.weight_kg": 72.63
  },
  "reason": "Updated weight after precision re-measurement during condition assessment."
}
```

#### 4.4.7 Correction Event

Fixes minor errors in previous events. The original event remains on-chain (immutable). The correction references it and states the corrected values.

**Required fields:**
- `corrected_event_pointer` — inscription ID of the event being corrected
- `corrected_fields` — object containing field names and their corrected values
- `reason` — text explanation

**Example:**

```json
{
  "protocol": "artcube",
  "event_type": "CORRECTION_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-09-01",
  "previous_event_pointer": null,
  "corrected_event_pointer": "def456i0",
  "corrected_fields": {
    "inspector_entity": "Fine Art Conservation Group LLC"
  },
  "reason": "Corrected inspector entity legal name (was missing 'LLC' suffix)."
}
```

#### 4.4.8 Supersession Event

Used when a **material identity error** requires a new Genesis inscription. The old Genesis remains on-chain (immutable) but is marked as superseded. The new Genesis becomes the canonical root.

**Trust assumption:** The Supersession Event is created by the Genesis UTXO holder — the same party with write access to the provenance chain. The protocol assumes this party has authority to declare supersession. Verifiers should treat supersession as an assertion by the controlling party, subject to the same trust framework as all other events. Off-chain disputes about supersession legitimacy are resolved outside the protocol.

**Required fields:**
- `new_genesis_inscription` — inscription ID of the replacement Genesis
- `reason` — text explanation of why supersession is necessary

**Example:**

```json
{
  "protocol": "artcube",
  "event_type": "SUPERSESSION_EVENT",
  "version": "1.0",
  "parent_genesis_inscription": "abc123i0",
  "effective_date": "2026-10-01",
  "previous_event_pointer": null,
  "new_genesis_inscription": "ghi789i0",
  "reason": "Original Genesis contained material attribution error. New Genesis created with corrected maker information."
}
```

---

### 4.5 Authorization Chain

Authorization at every level is enforced by the Ordinals parent/child mechanism. No external authentication system is required.

| Level | Who Creates Children? | Proof |
|-------|----------------------|-------|
| Entity Root | Entity Root UTXO holder | Must spend Entity Root UTXO in child reveal transaction |
| Collection Parent | Collection Parent UTXO holder | Must spend Collection Parent UTXO in child reveal transaction |
| Genesis | Genesis UTXO holder | Must spend Genesis UTXO in child reveal transaction |

This means:
- Only the Entity Root holder can create new collections
- Only the Collection Parent holder can add new artworks to a collection
- Only the Genesis holder can append events to an artwork's provenance chain
- UTXO custody is a dual concern: title control **and** record-keeping authority

### 4.6 Hybrid Title Transfer Model

The ArtCube Protocol adopts a dual-condition model for canonical title recognition. A transfer is valid only when **both** conditions are satisfied:

1. **UTXO Transfer** — The Genesis inscription UTXO is transferred to the new controlling address.
2. **Title Event** — A Title Event is appended with `title_status: "TRANSFERRED"`, recording the new legal owner entity and UTXO control entity.

**Canonical title state** is derived from alignment between Genesis UTXO control and the most recent valid Title Event.

| UTXO Holder | Title Event Says | State |
|-------------|-----------------|-------|
| Alice | Alice owns | **VALID** — aligned |
| Bob | Alice owns | **INDETERMINATE** — UTXO moved without Title Event |
| Alice | Bob owns | **INDETERMINATE** — Title Event without UTXO transfer |
| Bob | Bob owns | **VALID** — aligned |

If UTXO control and the latest Title Event are misaligned, the title state is **INDETERMINATE** until a corrective action aligns them.

**Write access during indeterminacy.** Regardless of what the most recent Title Event declares, only the UTXO holder has write access — the ability to append new events (including corrective Title Events). This means the UTXO holder can always self-correct misalignment by inscribing a new Title Event. The Title Event claimant who does not hold the UTXO has no on-chain recourse; their claim is recorded but cannot be enforced at the protocol level.

Bitcoin enforces technical control through UTXO ownership. Legal ownership remains governed by applicable law and contractual agreement. The protocol records declared states — it does not independently establish or transfer legal rights.

### 4.7 Intellectual Property Framework

The Genesis inscription includes a structured IP baseline that establishes default rights and a framework for rights evolution.

**Copyright status** (one of): `OWNED` | `LICENSED` | `PUBLIC_DOMAIN` | `UNKNOWN` | `DISPUTED`

**Baseline rights** (granted to the aligned holder by default):
- `DISPLAY_NONCOMMERCIAL` — Non-commercial display of the physical artwork
- `EXHIBIT` — Exhibition in galleries, museums, and public spaces
- `PUBLISH_IMAGES_LOWRES` — Low-resolution image publication for editorial purposes

**Holder scope:** `ALIGNED_HOLDER` — the entity recognized as owner under the hybrid model (where UTXO control and latest Title Event are aligned).

**Scope:** Worldwide, perpetual, non-exclusive, non-revocable.

**Rights NOT granted by default** (require explicit IP Events):
- `PUBLISH_IMAGES_HIGHRES`
- `REPRODUCE_PRINTS`
- `MERCHANDISING`
- `COMMERCIALIZE`
- `CREATE_DERIVATIVES`
- `AI_TRAINING_ALLOWED`
- `AI_GENERATION_ALLOWED`
- `SUBLICENSE`

**Moral rights:**
- Attribution required: `YES` | `NO` | `UNKNOWN`
- Waived: `YES` | `NO` | `UNKNOWN`
- Integrity rights notes: free text or null

Expanded rights are managed through append-only IP Events (Section 4.3.5). Each IP Event creates an immutable on-chain record of the rights action.

### 4.8 Document Storage Layer

The ArtCube Protocol uses a three-tier storage model:

| Tier | Role | Stores |
|------|------|--------|
| **Bitcoin** | Truth layer | SHA-256 hashes + structured metadata |
| **Arweave** | Retrieval layer | Permanent document storage (plaintext or encrypted) |
| **Off-chain** | Operational layer | Fast access copies (optional) |

**The Bitcoin hash is always of the original plaintext document.** This is the integrity proof regardless of whether the document is stored publicly, privately, or not at all.

#### Storage Policies

| Policy | Bitcoin | Arweave | Who Can Verify |
|--------|---------|---------|----------------|
| `PERMANENT_PUBLIC` | SHA-256 hash | Plaintext document | Anyone |
| `PERMANENT_PRIVATE` | SHA-256 hash | Encrypted document | Key holders only |
| `HASH_ONLY` | SHA-256 hash | Nothing | Document holder only |

#### Encryption (PERMANENT_PRIVATE)

Arweave is public by default. For private documents, encryption is handled client-side before upload:

- **Method:** AES-256-GCM
- **Key derivation:** The document creator signs a fixed message with their wallet. The signature is combined with the document's SHA-256 hash via HKDF to derive a unique AES-256 key per wallet + document pair.
- **No server-side key storage required.** The key can be re-derived from the wallet at any time.

#### Recommended Storage Policies

| Document Type | Recommended Policy | Rationale |
|---------------|-------------------|-----------|
| Condition report | `PERMANENT_PUBLIC` | Public provenance value |
| Provenance certificate | `PERMANENT_PUBLIC` | Core mission — verifiable history |
| Assay / authentication | `PERMANENT_PUBLIC` | Third-party verification value |
| Appraisal | `PERMANENT_PRIVATE` | Contains sensitive valuations |
| Insurance policy | `PERMANENT_PRIVATE` | Confidential terms and coverage |
| Legal agreement | `PERMANENT_PRIVATE` | Confidential contractual terms |
| License contract | `PERMANENT_PRIVATE` | May contain sensitive commercial terms |
| Transfer agreement | `PERMANENT_PRIVATE` | Parties and terms are confidential |

These are recommendations. The user chooses the policy per document.

### 4.9 Correction and Supersession

Genesis is **never edited.** Errors are handled via two mechanisms:

**Minor errors** — A `CORRECTION_EVENT` is inscribed as a child of Genesis. It references the event being corrected, states the corrected fields, and includes an explanation. The original event remains on-chain.

**Material identity errors** — When the identity of the artwork itself is wrong (e.g., misattribution, wrong object):
1. Create a **new Genesis inscription** with correct information
2. Inscribe a `SUPERSESSION_EVENT` as a child of the old Genesis, pointing to the new Genesis

The old Genesis remains on-chain. The new Genesis becomes the canonical root. Superseded provenance chains can still be audited — full history is always preserved.

### 4.10 Burning Semantics

Burning an inscription means sending it to a provably unspendable output via `OP_RETURN`. This is the only mechanism the ArtCube Protocol recognizes as a valid burn — sending to a known-unspendable address (e.g., a zero address) is not sufficient, as such addresses are technically spendable if the private key were discovered. An `OP_RETURN` output is cryptographically unspendable by Bitcoin consensus rules.

Burning has specific meaning at each level:

| Level | Effect |
|-------|--------|
| Entity Root | Entity permanently closed. No new Collection Parents may be created. |
| Collection Parent | Collection permanently closed. No new Genesis inscriptions may be created under it. |
| Genesis | Provenance chain permanently frozen. No new Events can be appended. Title state becomes indeterminate. |

**Existing provenance chains below a burned parent remain fully valid and verifiable.** Burning is a permanent action — it cannot be reversed.

---

## 5. Verification Algorithm

Any party can independently verify an artwork's full provenance chain using the following procedure:

```
VERIFY_PROVENANCE(genesis_id):

  1. FETCH Genesis inscription by genesis_id
     → Verify content_type = "application/json"
     → Verify protocol = "artcube"
     → Verify inscription_type = "GENESIS_TITLE_ANCHOR"
     → Parse JSON content

  2. CHECK HIERARCHY (optional but recommended):
     a. Read parent pointer of Genesis → collection_parent_id
     b. If collection_parent_id exists:
        → Fetch Collection Parent inscription
        → Verify inscription_type = "COLLECTION_PARENT"
        → Verify Collection Parent UTXO is NOT burned
        → Read parent pointer of Collection Parent → entity_root_id
     c. If entity_root_id exists:
        → Fetch Entity Root inscription
        → Verify inscription_type = "ENTITY_ROOT"
        → Verify Entity Root UTXO is NOT burned

  3. VERIFY GENESIS UTXO STATUS:
     → Determine current holder of Genesis UTXO
     → If UTXO is burned → title state = FROZEN (no further events possible)
     → Record current_utxo_holder

  4. COLLECT ALL EVENTS:
     → Find all child inscriptions of Genesis
     → Filter to valid ArtCube Protocol events (protocol = "artcube")
     → Group by event_type

  5. RECONSTRUCT EVENT CHAINS:
     For each event_type group:
       → Sort by previous_event_pointer
       → Build linked list: null → event_1 → event_2 → ... → latest
       → Verify chain is contiguous (no gaps, no forks)

  6. DETERMINE TITLE STATE:
     → Find latest TITLE_EVENT in chain
     → Compare title_event.utxo_control_entity with current_utxo_holder
     → If aligned → title state = VALID
     → If misaligned → title state = INDETERMINATE

  7. VERIFY DOCUMENTS (optional):
     For each DOCUMENT_EVENT:
       → If storage_policy = PERMANENT_PUBLIC:
            Fetch from Arweave (arweave_tx_id)
            → SHA-256 hash the content
            → Compare to document_hash_sha256 on Bitcoin
            → Match = integrity confirmed
       → If storage_policy = PERMANENT_PRIVATE:
            Request decryption key from owner
            → Decrypt Arweave content
            → SHA-256 hash the plaintext
            → Compare to document_hash_sha256 on Bitcoin
       → If storage_policy = HASH_ONLY:
            Request original document from owner
            → SHA-256 hash the document
            → Compare to document_hash_sha256 on Bitcoin

  8. CHECK FOR SUPERSESSION:
     → If any SUPERSESSION_EVENT exists in event chain:
        → Follow new_genesis_inscription pointer
        → Restart verification from step 1 with new Genesis

  9. RETURN verification result:
     {
       genesis_id,
       hierarchy: { entity_root, collection_parent },
       title_state: VALID | INDETERMINATE | FROZEN,
       current_owner: { from latest aligned TITLE_EVENT },
       event_count: { by type },
       document_integrity: { verified / failed / pending },
       superseded: true/false
     }
```

---

## 6. Reference Implementation

**ArtCube** is the first application built on the ArtCube Protocol. Developed by Trio and BitBasel, ArtCube is an open-source Next.js application that provides a complete interface for creating, managing, and verifying ArtCube Protocol provenance chains.

The reference implementation includes:
- Genesis Title Anchor creation with structured Object ID forms
- Append-only event inscription (all eight event types)
- Hybrid title transfer workflow via PSBTs
- Document upload with Arweave integration and three-tier storage
- Provenance chain verification and public explorer
- Wallet-based authentication using BIP-322 signatures

---

## 7. Considerations

**Lost UTXO.** If the Genesis UTXO is lost (private key compromise, wallet failure), no new events can be appended to the provenance chain. The title state becomes indeterminate. Institutional holders should use multisig wallets for all Genesis, Collection Parent, and Entity Root UTXOs.

**Multisig custody.** For institutional use, the ArtCube Protocol strongly recommends multisig custody (e.g., 2-of-3) for all UTXOs in the hierarchy. This prevents single points of failure and provides organizational key management.

**Fee economics.** Each event requires a Bitcoin transaction. During periods of high fees, batch operations and fee estimation should be used. JSON inscriptions are small (typically 1-5 KB), resulting in minimal additional fee impact beyond the base transaction cost.

**Privacy.** The Entity Root, Collection Parent, and Genesis structure is public by design. Art provenance is inherently transparent — the value of the record comes from its public verifiability. Sensitive documents can be stored encrypted via the `PERMANENT_PRIVATE` policy.

**Metadata size.** ArtCube Protocol inscriptions use structured JSON, typically 1-5 KB per inscription. This is minimal compared to image or media inscriptions and has negligible impact on Bitcoin block space.

**Backwards compatibility.** A standalone Genesis inscription (without Entity Root or Collection Parent hierarchy) is a valid ArtCube Protocol record. The hierarchy is optional and additive — it strengthens the provenance chain but is not required for basic functionality.

**Ordering guarantees.** Bitcoin provides a total ordering of transactions within blocks (by transaction index) and between blocks (by block height). ArtCube Protocol events inscribed in the same block are ordered by their transaction index within the block. The `previous_event_pointer` linked list provides an additional application-level ordering signal within each event category. In the event of a chain fork (two events claiming the same predecessor), block height and transaction index determine the canonical event (see Section 4.3, "Chain ordering and fork resolution").

---

## 8. Alternatives

### C2PA (Coalition for Content Provenance and Authenticity)

C2PA embeds provenance metadata directly in file headers. While useful for digital media attribution, it is unsuitable for artwork provenance:
- Metadata is easily stripped by image processors, social media platforms, and file converters
- No blockchain anchor — provenance data lives entirely in the file
- No mechanism for ongoing provenance updates (ownership changes, condition reports)
- Designed for digital media, not physical artwork lifecycle management
- No legal framework or title transfer mechanism

### NFT-Based Provenance (Ethereum, Solana)

NFT platforms on smart contract chains provide some provenance functionality but have fundamental limitations:
- **Mutable metadata.** Token URIs can be changed by the contract owner, altering the provenance record
- **IPFS dependency.** Off-chain data stored on IPFS may become unavailable if no node pins it
- **Chain longevity risk.** Ethereum and Solana have operated for far less time than Bitcoin and have undergone breaking protocol changes
- **No structured provenance standard.** NFT metadata follows informal conventions, not a rigorous provenance protocol
- **No hierarchical organization.** No native parent/child mechanism for entity → collection → artwork relationships

### Certificate Databases

Traditional certificate databases (gallery records, auction house databases, authentication services):
- **Centralized.** The issuing entity controls the database and can modify, delete, or lose records
- **Not publicly verifiable.** Third parties cannot independently verify records without the issuer's cooperation
- **Single point of failure.** If the issuing entity ceases operations, the records may be lost
- **No interoperability.** Each database is proprietary with no shared standard

### Traditional Paper Provenance

Paper certificates, gallery receipts, and expert opinions:
- Easily forged
- Frequently lost during ownership transitions
- Degrade over time
- No public verification mechanism
- Authentication depends entirely on the reputation of the issuing expert

The ArtCube Protocol addresses the limitations of all these approaches by anchoring structured, hierarchical provenance data directly on Bitcoin — immutable, publicly verifiable, and permanent.

---

## 9. Conclusion

The ArtCube Protocol provides a comprehensive, open standard for permanent artwork provenance built on the most secure computing network in existence.

The protocol's key contributions are:

1. **Hierarchical inscription structure** — Entity Root → Collection Parent → Genesis → Events creates an organizational chain of trust enforced by Bitcoin consensus.

2. **Hybrid title transfer model** — Requiring both UTXO transfer and a Title Event prevents unilateral ownership claims and provides a clear mechanism for determining canonical title state.

3. **Structured intellectual property framework** — Baseline display rights are granted by default. All expanded rights require explicit, immutable IP Events.

4. **Three-tier document storage** — Bitcoin anchors truth (hashes), Arweave provides permanent retrieval (documents), and off-chain systems handle operational access.

5. **Append-only, publicly verifiable** — Nothing is overwritten. The complete history of every artwork is permanently available for independent verification.

The ArtCube Protocol is an open standard. Any application can implement it. The protocol is not controlled by any single entity — it is defined by this specification and enforced by Bitcoin consensus.

The first implementation is ArtCube, built by Trio and BitBasel.

---

## Appendix A: Full Hierarchy Example

The following illustrates a complete ArtCube Protocol hierarchy for the Battle of the Centaurs silver edition:

```
Entity Root: ArtCube
  inscription_type: ENTITY_ROOT
  entity.type: ORGANIZATION
  entity.founding_entities: [BitBasel, Trio]
  entity.jurisdiction: Florida, USA
  │
  └─ Collection Parent: Battle of the Centaurs — Precious Metal Editions
       inscription_type: COLLECTION_PARENT
       collection.type: EDITION_SERIES
       collection.artist: Michelangelo Buonarroti
       │
       └─ Genesis: Silver Battle of the Centaurs
            inscription_type: GENESIS_TITLE_ANCHOR
            asset_id: TIC-SBC-SLV-2022-001
            owner: Silver Battle of the Centaurs, Inc.
            │
            ├─ Title Event 1 (OWNED, aligned with UTXO holder)
            ├─ Document Event 1 (condition report, PERMANENT_PUBLIC)
            ├─ Document Event 2 (assay certificate, PERMANENT_PUBLIC)
            ├─ Condition Event 1 (initial inspection)
            ├─ Custody Event 1 (stored at Miami Art Storage)
            ├─ IP Event 1 (RIGHTS_GRANT: high-res publication rights)
            ├─ Title Event 2 (TRANSFERRED → new owner, previous → Title Event 1)
            └─ Custody Event 2 (transferred to new location, previous → Custody Event 1)
```

Each event is a separate Bitcoin inscription, permanently recorded and independently verifiable.

---

*The ArtCube Protocol is released under the Creative Commons Attribution 4.0 International License (CC-BY-4.0).*
