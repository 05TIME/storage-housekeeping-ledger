---
name: storage-housekeeping-ledger
description: Replayable, auditable housekeeping for home NAS storage. Produces a duplicate-file plan, an optional backup-copy integrity check, and a saved housekeeping ledger in which every proposed or performed step carries its reason, its source result, and SHA-256 evidence that can be re-checked later. Moves duplicates into a review folder only after the user approves the exact list; never deletes. Use when the user says things like "clean up duplicates in my Photos folder", "tidy my Downloads and keep a record", "check that my backup folder still matches the originals", or "re-check last month's housekeeping ledger".
---

# Goal

Give a household one trustworthy housekeeping routine for a NAS folder the user chooses. The user-visible results are:

1. A **housekeeping plan**: duplicate groups found in the chosen folder, with one proposed keeper per group and a written reason for every proposal.
2. An optional **backup check**: whether files in a chosen backup folder still match their originals by SHA-256.
3. A **housekeeping ledger** saved as a text note on the NAS: an ordered list of entries, each with the step, the delegated expert, the real result it was based on, the reason, the decision (proposed, approved, declined, performed, failed) and fingerprints, so the run can be re-checked later.
4. A **re-check report** when the user asks to verify an earlier ledger: which recorded fingerprints still match, which changed and which files are missing.

Scope: one source folder per run (plus at most one backup folder). The Skill never deletes files. The only file change is moving approved duplicates into a review folder, which the user can undo by moving them back.

Reference material, read when the matching step starts:

- [Ledger format](references/ledger-format.md) defines entry fields, ordering and the deterministic keeper rule.
- [Approval and safety rules](references/approval-and-safety.md) defines what needs confirmation and what is never done.
- [Re-check procedure](references/recheck-procedure.md) explains how a saved ledger is verified later.
- [Worked example](references/worked-example.md) shows a full run on illustrative data.

# Required input

- **Mode**: `plan` (scan and propose, the default), `backup-check`, or `recheck`.
- **Source folder**: the folder the user names, for example their Photos or Downloads share. Ask the user if it is not given. Do not guess a path; let `file-agent` resolve the user's own directory.
- **Backup folder** (backup-check only): the folder that should hold copies.
- **Ledger note** (recheck only): the title of a ledger previously saved by this Skill.
- **Review folder name** (only if the user approves moves): default `housekeeping-review` inside the source folder, confirmed with the user.

Do not collect credentials or any parameter owned by a child expert.

# Generation-time capability evidence

- `ug-filemgr` → `file-agent`
- `ug-filesystem` → `file-agent`
- `file-workflow` → `file-agent`
- `ug-knowledge` → `knowledge-agent`

Actions relied on: `ug-filemgr` scan-dedupe (read); `ug-filesystem` find, ls, stat, cat, sha256sum (read) and mkdir (state-change, gateway approval); `file-workflow` get-user-dir (read) and batch-move (state-change, workflow approval); `ug-knowledge` save-text (state-change, workflow approval). No other capability is used.

# Orchestration

Run steps in order. Each step uses only the real result returned by the previous step. Record a ledger entry after every step, following [Ledger format](references/ledger-format.md).

## Plan mode

**Step 1. Resolve the folder.**

delegate --target file-agent --instruction 'Resolve the folder the user named for housekeeping inside their own user directory. Return its full path and confirm it exists. Do not create, move or change anything.'

If the folder cannot be resolved, report that and stop.

**Step 2. Scan for duplicates (read-only).**

delegate --target file-agent --instruction 'Run a duplicate scan on the folder path resolved in step 1, including subfolders. For every duplicate group return each file path, size in bytes, modified time and SHA-256 value. Read only: do not move, rename, tag or delete anything.'

If the scan returns no results (zero matches), tell the user no duplicates were found, save a ledger with a single "no duplicates" entry, and stop.

**Step 3. Build the plan.** For each group, apply the deterministic keeper rule from [Ledger format](references/ledger-format.md) and write one ledger entry per file with its reason. Show the plan as a numbered list with group count, file count and reclaimable bytes.

**Step 4. Ask for approval.** Follow [Approval and safety rules](references/approval-and-safety.md). Show the exact files that would move and the review folder name. The user may approve all, approve a subset by number, or decline. A decline is recorded and the run ends after step 6.

**Step 5. Move approved duplicates (only after approval).**

delegate --target file-agent --instruction 'Create the review folder named housekeeping-review inside the folder resolved in step 1 if it does not exist, then move exactly the files the user approved in step 4, and only those, into it. Keep each file name; if a name already exists in the review folder, stop and report the conflict instead of overwriting. Do not delete anything. Return the original and new path of every moved file and any file that failed.'

**Step 6. Verify and save the ledger.** For moved files, ask `file-agent` for the SHA-256 at the new path and compare it with the fingerprint from step 2; record match or mismatch.

delegate --target file-agent --instruction 'Return the SHA-256 value of each file at the new paths reported by the move in step 5. Read only.'

delegate --target knowledge-agent --instruction 'Save the housekeeping ledger text supplied in this request as a new text note titled with the source folder name and the run date. Do not change the text. Return the saved note title and location.'

## Backup-check mode

Resolve both folders as in step 1, then:

delegate --target file-agent --instruction 'For the source folder and backup folder resolved earlier, list files by relative path and return the SHA-256 value of each file in both folders. Read only.'

Compare by relative path: same fingerprint is "matches", different is "differs", absent in the backup is "missing from backup". Report counts and the list, record each comparison in the ledger, and save it with `knowledge-agent` as in step 6. Nothing is copied or repaired automatically.

## Recheck mode

Follow [Re-check procedure](references/recheck-procedure.md): ask `file-agent` to find and read the saved ledger text the user names, ask `file-agent` for the current SHA-256 of each recorded path, and report matches, changes and missing files. Recheck is read-only except for saving the new re-check report.

# Completion and failure

Handle every delegation state explicitly:

- `completed`: use only the returned result in the next step and record it in the ledger.
- `failed`: record the failure and its message in the ledger, stop every dependent step, and tell the user which steps finished. Never claim a move or save that did not complete.
- `cancelled`: record that the user cancelled, stop, and report what was already done (for example, files moved before the cancel).
- `waiting_user`: relay the expert's question to the user and wait; do not answer on the user's behalf.
- `waiting_approval`: show the approval request to the user unchanged and wait for their answer before continuing.

Empty results: if a scan, listing or ledger lookup returns an empty result, say so plainly and stop that branch. Do not substitute a different folder or similar files.

Partial results: if some moves or fingerprint reads fail, report the succeeded and failed files separately, keep the ledger accurate, and suggest a recheck run.

The run is complete only when the ledger note is saved and the user has seen the summary: duplicates found, files moved, fingerprint matches, and the ledger note title.
