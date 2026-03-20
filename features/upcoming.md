# Upcoming Features

Open pull requests against `logos-blockchain/logos-execution-zone` that are under review but not yet merged.

## PR #400 — Validity Windows on Program Output

**Author:** Sergio Chouhy (@schouhy)
**Status:** Open
**Branch:** `schouhy/add-block-context`

### What it does

Adds validity windows to program outputs so that programs can specify a block ID range during which their results are valid. The validity window is a pair of optional bounds: `(Option<BlockId>, Option<BlockId>)`, where either or both bounds can be omitted for half-open or unbounded intervals.

When a program produces output with a validity window, the sequencer checks that the current block ID falls within the specified range before accepting the transaction. This applies to both public and privacy-preserving transactions.

For privacy-preserving transactions, the validity window flows through the privacy circuit. When multiple programs are composed via chained calls, the circuit computes the intersection of all validity windows (max of lower bounds, min of upper bounds).

### Why it matters

Without validity windows, a transaction remains valid indefinitely once signed. This creates problems for time-sensitive operations: expired governance votes could still be cast, stale oracle prices could be consumed, and transactions crafted for specific market conditions could execute long after those conditions changed. Validity windows give programs the ability to enforce temporal constraints at the protocol level.

### Key design decisions

- The validity window is part of `ProgramOutput`, meaning programs determine their own validity range (not users).
- Uses a builder pattern on `ProgramOutput` (`valid_from_id()` / `valid_until_id()`), making the feature opt-in. Existing programs default to `(None, None)` — always valid.
- Inclusive-inclusive range semantics (`from_id <= block_id && block_id <= until_id`).

## PR #403 — Block Context and Clock Program

**Author:** Sergio Chouhy (@schouhy)
**Status:** Open
**Branch:** `schouhy/add-block-context` (builds on PR #400)

### What it does

Introduces an on-chain block context account and a native Block Context Program (sometimes referred to as the "clock program"). The block context account stores the current block index and is automatically updated by the sequencer as the last transaction in every block.

This gives programs the ability to read the current block height during execution by including the block context account in their pre-states. Programs can use this for time-dependent logic: auction deadlines, vesting schedules, oracle freshness checks, or any computation that needs to know "what block are we in."

### Why it matters

PR #400 allows programs to say "this output is only valid during blocks 5-10." PR #403 goes further: it lets programs read the current block height and make decisions based on it. A voting program can check whether the voting period has ended. A payment stream program can calculate how much has vested. An oracle consumer can reject data that is too many blocks old.

Without on-chain block context, programs have no way to reason about time. The block height must be injected externally or not used at all. This PR makes time a first-class concept that programs can access directly.

### Key design decisions

- The block context account lives at a fixed, deterministic AccountId derived from `b"LEE/Context"`.
- The Block Context Program is a native program (not a user-deployed program). It has a fixed ProgramId and is available at genesis.
- Users cannot invoke the Block Context Program directly — the sequencer rejects unauthorized invocations.
- In privacy-preserving transactions, the block context account is read-only. The privacy circuit enforces that its pre-state and post-state are identical, preventing private transactions from mutating shared timing state.
- The block context account stores a `BlockContextData` struct with `block_index: u128`.

---

*Last updated: 2026-03-20*
