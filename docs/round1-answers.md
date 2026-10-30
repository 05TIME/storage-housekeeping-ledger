# UGREEN Hackathon 2026 Round 1 answers

Source text for docs/UGREEN-R1-storage-housekeeping-ledger.pdf. Word count of the ten answers: 472 (limit 500).

## 1. One sentence: What does the Skill do?

Storage Housekeeping Ledger finds duplicate files and checks backup folders on a UGREEN NAS, moves duplicates aside only after approval, and saves a fingerprinted ledger that can be re-checked later.

## 2. Who is the target user and problem?

Families and home users whose photo and download folders fill with copies. Cleanup tools either delete silently or leave no record, so people avoid cleaning, or later cannot tell what changed, why, or whether backups still match.

## 3. What is your proposed solution?

A Uliya Skill that turns housekeeping into an auditable routine. Every proposal, approval and action becomes a numbered ledger entry with its reason, source result and SHA-256 fingerprint. Nothing is deleted; approved duplicates go to a review folder, so every change is reversible.

## 4. What is the Core User Flow?

The user says "clean up duplicates in my Photos folder". The Skill resolves the folder, scans read-only, applies a deterministic keeper rule, and shows a numbered plan. The user approves all, some, or none. Approved files move to a review folder, fingerprints are re-verified, and the ledger is saved. Later, "re-check last month's ledger" reports matches, changes and missing files.

## 5. What substantive difference does your idea offer compared with existing tools or conventional AI solutions?

Conventional dedupe tools act once and forget; AI assistants explain decisions in chat that disappear. This Skill makes housekeeping replayable: the same scan always yields the same plan, every step cites the real result it used, and a saved ledger lets anyone verify outcomes weeks later without trusting the assistant.

## 6. Which UGREEN AI NAS capabilities will be used, and at which step in the workflow?

file-workflow get-user-dir resolves folders (step 1). ug-filemgr scan-dedupe finds duplicates (step 2). ug-filesystem sha256sum, stat, ls and find provide fingerprints and backup comparison (steps 2, 6, recheck). ug-filesystem mkdir and file-workflow batch-move run approved moves (step 5). ug-knowledge save-text saves ledgers (step 6); ug-filesystem cat reads them back for recheck.

## 7. What was your technical approach? Describe the planned Skill workflow, supported NAS CLI capabilities, expert agent orchestration, required inputs, permissions, AI model choices, and data handling, as applicable;

The package is pure Markdown following the Uliya standard: SKILL.md plus four linked references. The parent Skill only issues delegate commands to file-agent and knowledge-agent, matching catalog mappings. Inputs are mode, source folder, optional backup folder, ledger title. State changes go through gateway and workflow approval. Any NAS model works; the local model is preferred. Data never leaves the NAS.

## 8. What is the current development status? Identify what was implemented already, not yet implemented, or is pending NAS validation if selected to be a Finalist.

Implemented: complete Skill package with plan, backup-check and recheck modes, ledger format, keeper rule, approval rules and worked example. It passes the official uliya-skill-kit static checker with zero diagnostics. Pending NAS validation: real expert outputs, approval prompts, and ledger note saving and reading on device.

## 9. What is your final validation plan? Name two core test cases and one edge case to validate if selected as a Finalist.

Core 1: seeded Photos folder with three duplicate groups; approve a subset, confirm only those files move and fingerprints match. Core 2: backup folder with one altered and one missing file; confirm both are reported correctly. Edge: folder with no duplicates must report empty results, save a single-entry ledger, and change nothing.

## 10. Are there any known limitations of your Project? Describe current known limitations, technical risks, and any external dependencies.

One source folder per run; very large folders may scan slowly. Recheck depends on finding the saved note by title. Behaviour relies on expert output formats not yet observed on a device. Fingerprints detect change but cannot repair it. Dependencies: UGOS experts and capabilities only.
