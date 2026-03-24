# Writing LEZ Programs

A LEZ program is a RISC-V binary that runs inside the Risc0 zkVM. It receives a list of accounts and instruction data as input, performs arbitrary computation, and returns a `ProgramOutput` containing updated account states and optional chained calls to other programs.

Programs are identified by their image ID — a hash of the ELF binary. The same program ID is used in both public and private execution.

## The Program Model

Programs in LEZ are stateless black boxes. They receive accounts as input, modify them according to their logic, and return the result. They have no direct access to global state, the current block height, or any other external context unless it is passed as an input account.

The protocol enforces a set of invariants on every program output, regardless of what the program does internally:

- Account IDs are unique in the input set
- The number of pre-states and post-states must match
- Account nonces must remain unchanged
- Program ownership of accounts cannot change (except via the claim mechanism)
- Only the owning program can decrease an account's balance or modify its data
- Total balance must be conserved across all accounts in the call

If a program violates any of these rules, the transaction is rejected.

## Building with lez-framework

The `lez-framework` crate provides a `#[lez_program]` macro that handles the boilerplate of reading inputs and writing outputs, letting you focus on the program logic.

```rust
use lez_framework::lez_program;

#[lez_program]
fn my_program(accounts: Vec<Account>, instruction: MyInstruction) -> Vec<Account> {
    // your logic here
    accounts
}
```

The macro handles serialization, the Risc0 guest environment setup, and the `ProgramOutput` construction. Under the hood, programs use `read_nssa_inputs()` to consume inputs from the Risc0 environment and `write_nssa_outputs()` to commit results to the journal.

Repository: https://github.com/logos-co/lez-framework

## SPEL

SPEL (Smart Program Execution Language) is the higher-level language and framework for writing LEZ programs. It provides a more ergonomic interface than raw Rust, with built-in support for account management, cross-program calls, and testing.

Repository: https://github.com/logos-co/spel

Documentation and tutorials: https://github.com/KindaInsecureBot/spel-agent-resources

## Cross-Program Calls

Programs can call other programs atomically by emitting `ChainedCall` structs in their output. Each chained call specifies the target program ID, the accounts to pass, and the instruction data.

```rust
ChainedCall {
    program_id:       TOKEN_PROGRAM_ID,
    pre_states:       vec![user_account, vault_account],
    instruction_data: encode(&Transfer { amount: 100 }),
    pda_seeds:        vec![],
}
```

Calls execute depth-first. All calls in the chain are atomic — a failure at any step rolls back the entire transaction.

To authorize a program-derived account (PDA) in a chained call, include the PDA seed in `pda_seeds`. The execution layer derives the authorized account ID from the calling program's ID and the seed.

## Program-Derived Accounts (PDAs)

Programs can own accounts whose IDs are derived from the program ID and a seed value. This is how programs hold shared state — a liquidity pool, a vault, a registry — without requiring a user signature.

The account ID for a PDA is `SHA256("/NSSA/v0.2/AccountId/PDA/" || program_id || seed)` truncated to 32 bytes.

When a program emits a chained call with PDA seeds, the execution layer grants authorization for those derived accounts automatically, without requiring signatures from any user.

## Account Ownership and the Claim Mechanism

Every account has a `program_owner` field. Only the owning program can decrease the account's balance or modify its data. This is the core authorization primitive.

When a program creates a new account, it uses the `claim` flag in `AccountPostState`. A program can claim any account that currently has the default owner (`[0; 8]`). After claiming, the account's `program_owner` is set to the claiming program's ID.

This mechanism enables program-derived account creation: the first time a program touches a default-owned account in the correct way, it claims ownership.

## Built-In Programs

LEZ ships with several built-in programs:

**Token Program** — manages fungible tokens and NFTs. Supports: initialize (create token definition), mint, burn, transfer, print_nft. The token program is privacy-agnostic — the same program handles both public and private token accounts.

**AMM Program** — constant-product AMM with liquidity pools. Supports: new_definition (create pool), add_liquidity, remove_liquidity, swap. The AMM chains to the token program for all balance transfers.

**Authenticated Transfer Program** — a simple program for direct account-to-account transfers with ECDSA signature verification. Used in tutorials and tests.

## Learning Resources

- [CryptoLizards](https://github.com/KindaInsecureBot/cryptolizards) — interactive step-by-step tutorial, CryptoZombies style
- [spel-agent-resources](https://github.com/KindaInsecureBot/spel-agent-resources) — comprehensive SPEL documentation
- [lez-programs](https://github.com/logos-blockchain/lez-programs) — official example programs
- [lezard](https://github.com/gravityblast/lezard) — framework for building and testing LEZ programs
