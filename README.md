# storage-housekeeping-ledger

A UGREEN AI NAS Skill (Uliya Skill Development Standard) for replayable, auditable home-storage housekeeping:

- **plan**: read-only duplicate scan, deterministic keeper rule, numbered plan, approved moves to a review folder (never deletes)
- **backup-check**: SHA-256 comparison of a source folder against a backup folder
- **recheck**: verify an earlier saved ledger against current fingerprints

Every step is written to a ledger with its reason, the real expert result it used, and SHA-256 evidence.

## Layout

- `storage-housekeeping-ledger/`: the Skill package (`SKILL.md` + `references/*.md`, Markdown only, no scripts)
- `docs/`: Round 1 proposal PDF and its source answers

## Validation

Passes the official `uliya-skill-kit` 0.7.3 static checker (`static-pass`, zero diagnostics). Not yet run on a device.

## License

MIT
