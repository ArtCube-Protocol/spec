# Inscription Envelopes

Complete envelope specifications for each ArtCube Protocol inscription type. Each section shows the metadata (tag 5), properties (tag 17), and body payload.

These examples correspond to the JSON files in this directory and form a complete provenance chain for the Silver Battle of the Centaurs.

---

## Entity Root

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <entity-root.json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "ENTITY_ROOT",
  "name": "ArtCube",
  "entity_type": "ORGANIZATION"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "ArtCube",
    "1": {
      "Protocol": "artcube",
      "Type": "Entity Root",
      "Entity": "ArtCube",
      "Jurisdiction": "Florida, USA",
      "Custody Model": "MULTISIG"
    }
  }
}
```

**Parent:** None (top-level inscription)

---

## Collection Parent

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <entity root inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <collection-parent.json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "COLLECTION_PARENT",
  "collection": "Eternal Creations — Michelangelo Posthumous Originals",
  "artist": "Michelangelo Buonarroti"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Eternal Creations — Michelangelo Posthumous Originals",
    "1": {
      "Protocol": "artcube",
      "Type": "Collection Parent",
      "Collection": "Eternal Creations — Michelangelo Posthumous Originals",
      "Collection Type": "EDITION_SERIES",
      "Artist": "Michelangelo Buonarroti",
      "Medium": "Precious metal castings (silver, gold, platinum)"
    }
  }
}
```

**Parent:** Entity Root inscription (tag 3)

---

## Genesis Title Anchor

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <collection parent inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <genesis.json>
OP_ENDIF
```

**Metadata (tag 5):**

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

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Battle of the Centaurs",
    "1": {
      "Protocol": "artcube",
      "Type": "Genesis",
      "Artist": "Michelangelo Buonarroti",
      "Title": "Battle of the Centaurs",
      "Medium": ".999 Fine Silver",
      "Period": "c.1490–1492",
      "Owner": "Silver Battle of the Centaurs, Inc.",
      "Jurisdiction": "Florida, USA",
      "Collection": "Eternal Creations"
    }
  }
}
```

**Parent:** Collection Parent inscription (tag 3)

---

## Title Event — Initial Ownership

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <genesis inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <title event json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "TITLE_EVENT",
  "event": "TITLE_EVENT",
  "status": "OWNED"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Title Event — OWNED",
    "1": {
      "Protocol": "artcube",
      "Type": "Title Event",
      "Status": "OWNED",
      "Owner": "Silver Battle of the Centaurs, Inc."
    }
  }
}
```

**Parent:** Genesis inscription (tag 3)

---

## Document Event — Assay Certificate

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <genesis inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <document event json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "DOCUMENT_EVENT",
  "event": "DOCUMENT_EVENT",
  "document_type": "ASSAY"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Document Event — ASSAY",
    "1": {
      "Protocol": "artcube",
      "Type": "Document Event",
      "Document": "ASSAY",
      "Issuer": "Independent Precious Metals Lab"
    }
  }
}
```

**Parent:** Genesis inscription (tag 3)

---

## Condition Event — Physical Inspection

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <genesis inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <condition event json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "CONDITION_EVENT",
  "event": "CONDITION_EVENT",
  "inspector": "Art Conservation Services LLC"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Condition Event — 2026-03-14",
    "1": {
      "Protocol": "artcube",
      "Type": "Condition Event",
      "Inspector": "Art Conservation Services LLC",
      "Condition": "Excellent"
    }
  }
}
```

**Parent:** Genesis inscription (tag 3)

---

## Custody Event — Storage

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <genesis inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <custody event json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "CUSTODY_EVENT",
  "event": "CUSTODY_EVENT",
  "custodian": "Miami Art Storage Inc."
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Custody Event — Miami Art Storage Inc.",
    "1": {
      "Protocol": "artcube",
      "Type": "Custody Event",
      "Custodian": "Miami Art Storage Inc.",
      "Location": "Miami, Florida, USA"
    }
  }
}
```

**Parent:** Genesis inscription (tag 3)

---

## IP Event — Rights Grant

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <genesis inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <ip event json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "IP_EVENT",
  "event": "IP_EVENT",
  "sub_type": "RIGHTS_GRANT"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "IP Event — RIGHTS_GRANT",
    "1": {
      "Protocol": "artcube",
      "Type": "IP Event",
      "Sub Type": "RIGHTS_GRANT",
      "Rights": "PUBLISH_IMAGES_HIGHRES, REPRODUCE_PRINTS"
    }
  }
}
```

**Parent:** Genesis inscription (tag 3)

---

## Title Event — Transfer

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <genesis inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <title event json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "TITLE_EVENT",
  "event": "TITLE_EVENT",
  "status": "TRANSFERRED"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Title Event — TRANSFERRED",
    "1": {
      "Protocol": "artcube",
      "Type": "Title Event",
      "Status": "TRANSFERRED",
      "Owner": "Collector Holdings LLC"
    }
  }
}
```

**Parent:** Genesis inscription (tag 3)

---

## Custody Event — New Location

**Envelope:**

```
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "application/json"
  OP_PUSH 3
  OP_PUSH <genesis inscription ID>
  OP_PUSH 5
  OP_PUSH <cbor of metadata below>
  OP_PUSH 7
  OP_PUSH "artcube"
  OP_PUSH 17
  OP_PUSH <cbor of properties below>
  OP_PUSH 19
  OP_PUSH "cbor"
  OP_PUSH 0
  OP_PUSH <custody event json>
OP_ENDIF
```

**Metadata (tag 5):**

```json
{
  "p": "artcube",
  "type": "CUSTODY_EVENT",
  "event": "CUSTODY_EVENT",
  "custodian": "Collector Holdings LLC"
}
```

**Properties (tag 17):**

```json
{
  "1": {
    "0": "Custody Event — Collector Holdings LLC",
    "1": {
      "Protocol": "artcube",
      "Type": "Custody Event",
      "Custodian": "Collector Holdings LLC",
      "Location": "New York, New York, USA"
    }
  }
}
```

**Parent:** Genesis inscription (tag 3)
