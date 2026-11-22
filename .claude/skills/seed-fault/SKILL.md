---
name: seed-fault
description: Exercise 2. Temporarily seed one code fault so the PostToolUse hook and the tests can be observed catching it, then repair it. Use only when asked to seed, inject, or demonstrate a fault.
allowed-tools: Read, Edit, Bash(python -m pytest*), Bash(make test), Bash(diff*), Bash(cp*)
disable-model-invocation: true
---

# Seeding a fault: $ARGUMENTS

This is a three-phase, user-paced exercise. Do one phase per turn and stop
at the end of each. Never run the next phase until the user asks for it.

Rules for every phase:
- Seed exactly the one fault described in `$ARGUMENTS`. No other edits.
- Never touch `data/raw/`, `.claude/hooks/`, or `.claude/settings.json`.
- Never commit. Never work around or silence the hook.
- Show real output verbatim. Do not summarise, paraphrase, or invent it.

## Phase 1: seed the fault (then STOP)

1. Read the target file. Save a pristine copy outside the repo so the
   repair is exact: `cp <file> /tmp/seed-fault-<basename>.orig`
2. Apply the fault with the **Edit tool** (not sed, not a Bash write), so the
   PostToolUse hook fires.
3. Show the exact diff: `diff -u /tmp/seed-fault-<basename>.orig <file>`.
4. Show the hook's actual stderr exactly as the harness reported it. If the
   hook printed nothing, say "the hook reported no output"; do not fabricate
   any.
5. Reply "Fault seeded. Capture the output, then tell me to continue." and
   **stop**. Run no tests in this phase.

## Phase 2: show the failure (when told to continue, then STOP)

1. Leave the fault in place. Do not edit anything.
2. Run the relevant tests: the narrowest `python -m pytest` selection that
   covers the changed code (for `src/features.py` or `src/config.py`,
   `tests/test_leakage.py` and `tests/test_contract.py`). Use `make test`
   only if unsure which file applies.
3. Show the actual output, failures included. If nothing fails, say so plainly:
   that means the fault slipped past the tests, which is itself the finding.
4. Reply "Fault still in place. Tell me when to repair it." and **stop**.
   Do not propose or start a fix.

## Phase 3: repair (only when the user asks)

1. Restore the affected code by Edit, reversing the seeded change, or by
   copying back `/tmp/seed-fault-<basename>.orig`. Confirm with
   `diff -u /tmp/seed-fault-<basename>.orig <file>` that nothing differs.
2. Run `make test` and show the passing result verbatim, including the count.
3. Report that the repair is done, so the user can record how many turns it
   took. Leave the changes uncommitted.
