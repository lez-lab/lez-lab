# Feature: Block Context

Block Context gives LEZ programs access to block timing information — enabling time-sensitive logic like deadlines, expiry windows, voting periods, and oracle freshness checks.

## Why it matters

Without block context, LEZ programs have no reliable notion of time. Block Context solves this with two complementary mechanisms:

### Mechanism A — `block_context_account`

A genesis-deployed system account at `b"LEE/Context"` that stores the current `block_index: u128`. The sequencer updates it as the last transaction in every block.

**Block validity rule:** A block is invalid unless its last transaction is exactly one invocation of the Block Context Program. User invocations are rejected.

Programs include this account as a read-only input and use `block_index` for:
- Payment deadlines
- Oracle price freshness checks
- Loan maturity
- TWAP accumulation

The account is committed inside the Risc0 ZKP — validators can verify temporal constraints without re-executing the program.

### Mechanism B — `ValidityWindow`

An optional field on any transaction:

```rust
ValidityWindow {
    valid_from: u64,
    valid_until: u64,
}
```

Sequencer rejects the tx if `current_height < valid_from` or `current_height >= valid_until`.

Use for:
- Voting periods (`[start_height, end_height)`)
- Governance delay windows
- Expiring offers or signatures

## Examples

- [`examples/voting/`](./examples/voting/) — voting program using `ValidityWindow` to enforce a voting period
- [`examples/payment-stream/`](./examples/payment-stream/) — payment stream using `block_context_account` for deadline enforcement

## Status

✅ Implemented — [`feature/block-context`](https://github.com/lee-buddy/logos-execution-zone/tree/feature/block-context)
