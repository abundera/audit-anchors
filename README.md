# Abundera Audit Anchors

Immutable, off-platform audit anchor hashes for [Abundera](https://abundera.ai).

## What This Is

Every hour, Abundera's infrastructure generates a cryptographic checkpoint (anchor) of its tamper-evident audit log chain. Each anchor is a SHA-256 hash covering a segment of consecutive audit entries. Anchors are published here **in addition to** Cloudflare R2 object storage (with WORM/Bucket Lock protection), creating a dual-platform integrity guarantee.

## Why It Exists

Storing audit anchors solely within Cloudflare's infrastructure means a compromise of Cloudflare (or a privileged insider) could theoretically tamper with both the primary logs and the anchors simultaneously. By publishing anchor hashes to GitHub — a completely separate trust boundary — we ensure that:

- **Tampering is detectable** even if Cloudflare is fully compromised
- **Git commits provide independent timestamps** (SHA-1 commit hashes with GitHub's server-side timestamps)
- **Anyone can verify** the anchor chain without Cloudflare access
- **History is immutable** — git's append-only commit model prevents retroactive modification

## Anchor Format

Each file is a JSON object:

```json
{
  "anchor_sequence": 1234,
  "start_id": 50001,
  "end_id": 50150,
  "entry_count": 150,
  "segment_hash": "sha256:abcdef...",
  "previous_anchor_hash": "sha256:123456...",
  "anchor_hash": "sha256:789abc...",
  "timestamp": "2026-02-18T23:00:00.000Z",
  "schema_version": 1
}
```

| Field | Description |
|-------|-------------|
| `anchor_sequence` | Monotonically incrementing anchor number |
| `start_id` / `end_id` | Range of audit entry IDs covered by this anchor |
| `entry_count` | Number of entries in the segment |
| `segment_hash` | SHA-256 hash of all entries in this segment |
| `previous_anchor_hash` | Hash of the preceding anchor (chain link) |
| `anchor_hash` | SHA-256 hash of this anchor's contents |
| `timestamp` | When the anchor was generated (UTC) |

## Directory Structure

```
audit-anchors/
  YYYY/
    MM/
      DD/
        HH00-anchor.json
```

## Monthly Verification

On the 1st of each month, an automated verification report cross-references:

1. **D1 database chain** — walks all audit entries and verifies hash links
2. **R2 anchor chain** — verifies segment hashes and anchor-to-anchor links
3. **GitHub anchors** — verifies each R2 anchor matches its GitHub counterpart
4. **Gap detection** — flags any missing hourly anchors

Reports are stored in R2 at `verification-reports/YYYY/MM-report.json`.

## Verification

To verify the anchor chain independently:

1. Clone this repository
2. Walk the anchor files chronologically
3. Verify each `previous_anchor_hash` matches the preceding anchor's `anchor_hash`
4. Any break in the chain indicates tampering or data loss

## More Information

- [Security Portal](https://abundera.ai/security/portal)
- [Security Certificate](https://abundera.ai/security/certificate)
- [Security Posture Overview](https://abundera.ai/security)

## Contact

security@abundera.ai
