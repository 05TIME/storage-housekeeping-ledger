# Re-check procedure

Use this when the user asks to verify an earlier housekeeping or backup ledger.

1. Ask the user for the ledger note title if it was not given.
2. Ask `file-agent` to find the saved ledger note and read it:

   delegate --target file-agent --instruction 'Find the saved text note file whose name matches the ledger title the user gave, inside the user directory, and return its location and full text unchanged. Read only.'

   If no note is found, report that no matching ledger exists and stop. Do not pick a different ledger.
3. From the returned text, collect every entry that has both a path and a fingerprint, keeping entry numbers.
4. Ask `file-agent` for the current fingerprints:

   delegate --target file-agent --instruction 'For each file path listed in this request, return whether it exists and, if it does, its SHA-256 value. Read only.'

5. Compare each path with its recorded fingerprint:
   - same value: matches
   - different value: differs
   - file not found: missing
6. Report counts and the list of differing or missing files, citing the original entry numbers.
7. Save the re-check report as a new note with `knowledge-agent`, titled "Re-check of" plus the original ledger title. The original ledger is never edited.

A recheck never moves, repairs or deletes files. If the user wants to act on the findings, start a new plan run.
