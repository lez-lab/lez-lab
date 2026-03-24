# LEZ Tooling

An overview of the tools available for building, testing, and exploring the Logos Execution Zone.

## Development Frameworks

**lez-framework** — the `#[lez_program]` macro and supporting crates for writing LEZ programs in Rust. Handles Risc0 guest setup, input deserialization, output serialization, and PDA derivation. The standard starting point for writing any LEZ program.

https://github.com/logos-co/lez-framework

**SPEL** — Smart Program Execution Language, a higher-level framework on top of lez-framework. Provides a more ergonomic developer experience with better tooling support for account management, testing, and deployment.

https://github.com/logos-co/spel

**lezard** — an alternative framework by Andrea Franz (gravityblast) for building, testing, and deploying LEZ programs. Worth exploring if lez-framework feels too low-level.

https://github.com/gravityblast/lezard

## Scaffolding and Project Setup

**logos-scaffold** — CLI tool that bootstraps a complete Logos application environment. Sets up the full stack: LEZ node, wallet, and application layer in one command. The fastest way to get a local development environment running.

https://github.com/logos-co/logos-scaffold

## Exploration and Learning

**CryptoLizards** — interactive LEZ programming tutorial modeled after CryptoZombies. Teaches LEZ program development step-by-step through hands-on exercises. Best starting point if you are new to LEZ.

https://github.com/KindaInsecureBot/cryptolizards

**lez-code-explorer** — interactive code explorer for LEZ blockchain programs. Browse and understand LEZ program source code with context.

https://github.com/KindaInsecureBot/lez-code-explorer

**spel-agent-resources** — comprehensive documentation for the SPEL framework, including tutorials, reference documentation, and an AI agent skill for working with LEZ programs.

https://github.com/KindaInsecureBot/spel-agent-resources

## Program Registry

**SPELbook** — on-chain program registry for LEZ. Enables program discovery by storing IDL (Interface Description Language) metadata on-chain. Analogous to an npm registry for LEZ programs.

https://github.com/jimmy-claw/spelbook

## Wallet

**logos-execution-zone-wallet-ui** — the official wallet UI for LEZ. Supports creating accounts, token transfers, private transactions, and interacting with programs. Required for end-to-end testing with real transactions.

https://github.com/logos-blockchain/logos-execution-zone-wallet-ui

## Indexer and Explorer

**logos-execution-zone-module** — the Logos Core module wrapping the LEZ indexer. Provides block, account, and transaction queries via a Qt plugin interface. Used by the Logos desktop application.

https://github.com/logos-blockchain/logos-execution-zone-module

## Testing

**logos-blockchain-testing** — testing framework and utilities for the Logos blockchain, including LEZ. Provides helpers for writing integration tests against a running LEZ node.

https://github.com/logos-blockchain/logos-blockchain-testing

## Running a Node

To run a local LEZ node for development:

```bash
# Using logos-scaffold (recommended)
logos-scaffold init my-project
cd my-project
logos-scaffold start

# Or directly with Docker
git clone https://github.com/logos-blockchain/logos-execution-zone
cd logos-execution-zone
docker compose up
```

The sequencer API is available at `localhost:8080` by default. The indexer JSON-RPC API is at `localhost:8081`.
