---
name: lez-lab
description: Knowledge base and navigation guide for the LEZ Lab repository and the Logos Execution Zone ecosystem. Use when working with LEZ programs, understanding the LEZ protocol, exploring ecosystem tools, or contributing to lez-lab. Covers repo structure, program inventory, learning resources, and how to add new content.
---

# LEZ Lab — Agent Skill

## What is LEZ Lab

lez-lab/lez-lab is a public learning hub and tracker for the Logos Execution Zone (LEZ) — a ZK-based privacy-first L2 with hybrid public/private state.

## Repo structure

  learn/      Protocol documentation — architecture, accounts, execution, keys, programs
  features/   One folder per protocol feature, added when merged upstream
  programs/   Ecosystem programs with descriptions and examples
  tools/      Dev tools for exploring and interacting with LEZ
  ecosystem/  Integration layer — module, wallet UI
  .claude/    Agent skills and resources

## How to add content

When a new feature merges to logos-blockchain/logos-execution-zone:
  1. Create features/<feature-name>/README.md
  2. Add runnable examples under features/<feature-name>/examples/
  3. Update the root README table

When a new program appears in the ecosystem:
  1. Create programs/<program-name>/README.md
  2. Include: what it does, repo link, how to run, example usage

## Key ecosystem repos

  logos-blockchain/logos-execution-zone          Core protocol
  logos-blockchain/lez-programs                  Official programs
  logos-co/lez-framework                         #[lez_program] macro
  logos-co/spel                                  SPEL language
  logos-co/lez-multisig                          Multisig program
  logos-co/eth-lez-atomic-swaps                  ETH/LEZ swaps
  logos-co/logos-lez-payment-streams             Payment streams
  jimmy-claw/spelbook                            Program registry
  gravityblast/lezard                            LEZ dev framework
  KindaInsecureBot/cryptolizards                 Interactive LEZ tutorial
  KindaInsecureBot/spel-agent-resources          SPEL docs and agent skill

## Learning resources

  learn/                                         Protocol deep dives in this repo
  KindaInsecureBot/spel-agent-resources          SPEL documentation
  jimmy-claw/blog                                Field notes from building on LEZ
  KindaInsecureBot/cryptolizards                 CryptoZombies-style LEZ tutorial
