# LEZ Lab 🧪

A living learning lab for the [Logos Execution Zone](https://github.com/logos-blockchain/logos-execution-zone) — a privacy-first programmable L2 with hybrid public/private state.

## What is this?

This repo tracks everything being built on and around LEZ. Every time a new feature ships or a new program is deployed, LEZ Lab tests it, documents it, and builds tools to help you understand it.

**Who is this for?**
- Developers who want to build on LEZ and need working examples
- Anyone curious about how ZK-based private execution works in practice
- Contributors wanting to understand the LEZ protocol before diving into the codebase

## What's inside

| Folder | What's here |
|--------|-------------|
| `features/` | One folder per LEZ protocol feature — explained and runnable |
| `programs/` | Ecosystem programs built on LEZ (AMM, multisig, registry, etc.) |
| `tools/` | Dev tools to explore, debug, and interact with LEZ |

## Getting started

→ Start with [`features/block-context`](./features/block-context/) — the first feature tracked here.

## About LEZ

LEZ is a programmable L2 with protocol-level privacy:
- Hybrid public/private state — programs handle both in the same execution environment
- Privacy is enforced at the protocol level via ZKPs (Risc0 / RISC-V), not the application layer
- Same bytecode executes in both public and private contexts

→ [LEZ GitHub](https://github.com/logos-blockchain/logos-execution-zone)
