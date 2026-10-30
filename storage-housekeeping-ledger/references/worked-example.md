# Worked example

Illustrative data only; real runs use the results returned by the NAS experts.

User: "Clean up duplicates in my Photos folder and keep a record."

1. `file-agent` resolves the Photos folder.
2. Duplicate scan returns one group of three files with the same SHA-256:
   - Photos/2024/beach.jpg, modified 2024-07-02
   - Photos/Imports/beach.jpg, modified 2024-08-15
   - Photos/Imports/beach (1).jpg, modified 2024-08-15
3. Plan with keeper-rule-1:
   - Entry 2: keep Photos/2024/beach.jpg, earliest modified time.
   - Entry 3: propose move Photos/Imports/beach.jpg, same SHA-256 as entry 2, newer.
   - Entry 4: propose move Photos/Imports/beach (1).jpg, same SHA-256 as entry 2, newer.
4. The user approves entries 3 and 4.
5. `file-agent` creates housekeeping-review and moves both files. Entries 5 and 6 record "performed" with old and new paths.
6. Fingerprints at the new paths match the scan; entries 7 and 8 record "matches". `knowledge-agent` saves the ledger as "Photos housekeeping 2026-11-02".

A month later the user asks to re-check that ledger. All three fingerprints still match, so the report says 3 matches, 0 differs, 0 missing.
