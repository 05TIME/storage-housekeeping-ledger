# Ledger format

The ledger is plain text so it can be saved with `knowledge-agent` and read by a person without any tool.

## Header

- Ledger title: source folder name and run date.
- Mode: plan, backup-check or recheck.
- Source folder and, if used, backup folder and review folder, as returned by `file-agent`.
- Keeper rule version: `keeper-rule-1`.

## Entries

Entries are numbered from 1 in the order the steps ran. Each entry has these fields:

| Field | Meaning |
| --- | --- |
| Entry | Sequence number |
| Step | Step name from SKILL.md, for example "scan" or "move" |
| Expert | `file-agent` or `knowledge-agent` |
| Based on | The real result used, for example "duplicate group 3 from step 2" |
| Path | File path the entry concerns, if any |
| Fingerprint | SHA-256 value returned by `file-agent`, if any |
| Decision | proposed-keep, proposed-move, approved, declined, performed, failed, matches, differs, missing |
| Reason | One sentence in plain language |

## Deterministic keeper rule (keeper-rule-1)

Within one duplicate group, all files have the same SHA-256. Keep exactly one file, chosen by these tie-breakers in order:

1. The file with the earliest modified time.
2. If equal, the shortest full path.
3. If still equal, the path that sorts first alphabetically.

Every other file in the group is proposed for moving, with the reason "same SHA-256 as keeper <entry number>; newer or longer path". Because the rule depends only on returned paths, times and fingerprints, the same scan result always produces the same plan.

## Summary block

End the ledger with totals: groups, files proposed, files approved, files moved, fingerprint matches, mismatches and failures.
