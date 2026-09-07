# attestbook — public witness

This repository is the external witness for the attestbook transparency log at
<https://attestbook.iguchi-apps.workers.dev>.

## What is here

`sth/<tree size, zero-padded>.json` — one file per **signed tree head**, written once and never
changed:

```json
{
 "key_id": "vk_...",
 "root_hash": "<64 hex characters>",
 "sig": "<Ed25519 signature, base64url>",
 "timestamp": "2026-09-07T14:00:00Z",
 "tree_size": 42
}
```

## What is NOT here

No entries, no playbooks, no runs, no identifiers, no personal data. A tree head is a hash and a
signature; it says how many entries the log held and what its root was, and nothing about what any
of them were.

## Why it exists

A signed tree head proves the venue committed to a set of entries. It does not by itself prove the
venue showed the **same** head to everyone — a venue keeping two logs could sign both. Publishing
every head here, in a history the venue cannot rewrite and anyone can read, closes that gap: if the
venue ever serves a head that is not in this history, or reorders one that is, it is visible.

## How to check a head

1. Fetch the venue's public keys: `GET https://attestbook.iguchi-apps.workers.dev/v1/venue/keys`
   (no key needed). The response also carries the rotation rule.
2. The signed bytes are `JCS({"root_hash": ..., "timestamp": ..., "tree_size": ...})` — RFC 8785
   canonical JSON, which for these three keys is simply the object with the keys in that order and
   no spaces.
3. Verify `sig` (Ed25519, base64url unpadded) against the key whose validity window contains
   `timestamp`.

To check that a specific entry is in the log, ask the venue for an inclusion proof
(`GET /v1/log/proof?seq=`) and fold it against the `root_hash` of a head from this repository —
RFC 6962 §2.1.1.

## Commit cadence

Hourly, from the venue, when the tree has grown. A gap in the history means the venue did not
publish during that period; from Stage 1 a witness copy older than 26 hours withholds the highest
verification level (`witness_stale`).
