# Accounts in LEZ

## Account Structure

Every account in LEZ, whether public or private, shares the same schema:

```rust
struct Account {
    program_owner: ProgramId,   // [u32; 8] — the program that controls this account's data
    balance: u128,              // native token balance
    data: Data,                 // arbitrary byte string (max 100KB)
    nonce: u128,                // serves different purposes for public vs private accounts
}
```

An account that has never been touched has default values: zero balance, empty data, default program owner (`[0; 8]`), and zero nonce. The protocol treats the entire AccountId space as occupied — uninitialized accounts simply have default values and are not explicitly stored.

## Public Accounts

### AccountId Derivation

A public account's identity is derived from its owner's secp256k1 public key:

```
AccountId = SHA256("/LSSA/v0.3/AccountId/Public/" || public_key)
```

The public key is used for BIP-340 Schnorr signatures that authorize transactions involving the account.

### Nonce

For public accounts, the nonce is a sequential counter that tracks the number of transactions the account has signed. Each time the account's owner signs a transaction, the nonce increments by one after the transaction is accepted. This prevents replay attacks: a transaction signed with nonce N is only valid when the account's current nonce is exactly N.

### Authorization

Public account authorization works through digital signatures. When a transaction references a public account that requires authorization (typically the sender in a transfer), the account owner signs the transaction message with their private key. The sequencer verifies the BIP-340 Schnorr signature against the account's public key.

The `is_authorized` flag in `AccountWithMetadata` is set to `true` for any account whose owner provided a valid signature in the transaction's witness set.

### Program Ownership

The `program_owner` field determines which program can modify the account's `data` and decrease its `balance`. An account with the default program owner (`[0; 8]`) is uninitialized and can be claimed by any program through the claiming mechanism. Once claimed, only the owning program can modify its data.

Any program can increase an account's balance (credit tokens to it), but only the owning program can decrease the balance (debit tokens from it).

## Private Accounts

Private accounts use the same `Account` struct but are represented on-chain differently. Instead of storing the account data in the clear, the chain stores a cryptographic commitment — a hash that binds the data to its owner without revealing either.

### Key Material

A private account is controlled by three key components:

**Nullifier Secret Key (nsk)**: A 32-byte secret that represents spending authority. The nsk is used to derive nullifiers when spending or updating the account. It never leaves the owner's machine. Possession of the nsk is what it means to "own" a private account.

**Nullifier Public Key (npk)**: Derived from the nsk. Used as the account's identity and included in commitment computations.

**Incoming Viewing Key Pair (isk / ipk)**: A secp256k1 scalar/point pair used for Diffie-Hellman key exchange. The incoming viewing public key (ipk) is shared with senders so they can encrypt account data that only the recipient can decrypt. The incoming viewing secret key (isk) is used to derive the shared secret needed for decryption.

### AccountId Derivation

A private account's identity is derived from its nullifier public key:

```
AccountId = SHA256("/LSSA/v0.3/AccountId/Private/" || npk)
```

### Nonce

For private accounts, the nonce serves a completely different purpose than for public accounts. It is a pseudorandom blinding factor that ensures each commitment is unique, even if the account data has not changed. The nonce is derived iteratively:

- Initial nonce: `SHA256(npk)` (technically `private_account_nonce_init`)
- Subsequent nonces: `SHA256(nsk || previous_nonce)`

This derivation is deterministic given the keys, which means the account owner can always reconstruct their nonce sequence.

### Authorization

Private accounts do not use digital signatures for authorization. Instead, authorization is proven by knowledge of the nullifier secret key. When spending a private account, the user computes a nullifier that can only be produced with the correct nsk. The zero-knowledge proof attests that the nullifier was correctly derived without revealing the nsk itself.

## CommitmentSet

The CommitmentSet is the on-chain data structure that stores all active private account commitments. It is implemented as a Merkle tree.

### Commitment Computation

A commitment binds the account data to its owner:

```
Commitment = SHA256(
    "/LSSA/v0.3/Commitment/" ||
    npk ||
    program_owner ||
    balance_le ||
    nonce_le ||
    SHA256(data)
)
```

The commitment is unconditionally hiding because the nonce acts as a random blinding factor. Even if an observer knows the program owner and suspects a particular balance, they cannot verify their guess because they do not know the nonce.

### Membership Proofs

When spending a private account, the user must prove that the account's commitment exists in the CommitmentSet. This is done via a Merkle membership proof — a path from the commitment's leaf to the tree root, consisting of sibling hashes at each level.

The CommitmentSet maintains a history of root hashes. This allows proofs generated against a slightly stale root to remain valid even if new commitments have been inserted since the proof was computed. Without root history, concurrent private transactions would frequently invalidate each other.

## NullifierSet

The NullifierSet is a set of all nullifiers that have been revealed on-chain. Its purpose is to prevent double-spending: each commitment can only be consumed once.

### Nullifier Computation

There are two nullifier types:

**Initialization nullifier** — used when a private account is created for the first time:

```
Nullifier = SHA256("/LSSA/v0.3/Nullifier/Initialize/" || npk)
```

**Update nullifier** — used when an existing private account's state is modified:

```
Nullifier = SHA256("/LSSA/v0.3/Nullifier/Update/" || commitment || nsk)
```

The update nullifier requires the nsk, which means only the account owner can produce it. The commitment links the nullifier to a specific state, and the nsk proves ownership. An observer who sees a nullifier on-chain cannot determine which commitment it corresponds to without knowing the nsk.

### Double-Spend Prevention

When a transaction reveals a nullifier, the sequencer checks that it does not already exist in the NullifierSet. If it does, the transaction is rejected. After acceptance, the nullifier is added to the set permanently. This ensures each commitment can be spent exactly once.

## Account Scanning with View Tags

When someone sends a private transfer to your account, the encrypted output is published on-chain along with a view tag. The view tag is a single byte computed as:

```
view_tag = SHA256("/NSSA/v0.2/ViewTag/" || npk || vpk)[0]
```

Wallet software scans new transactions by comparing each output's view tag against the expected value for the user's keys. Since the view tag is one byte, roughly 255 out of every 256 outputs are filtered out immediately. Only outputs with a matching view tag undergo the more expensive Diffie-Hellman and decryption steps.

The encryption itself uses ECIES-style key agreement:

1. The sender generates an ephemeral secret key (esk) and corresponding ephemeral public key (epk)
2. A shared secret is computed via ECDH: `ss = ipk_recipient * esk_sender`
3. A symmetric key is derived via KDF: `key = SHA256("NSSA/v0.2/KDF-SHA256/" || ss || commitment || output_index)`
4. The account data is encrypted with ChaCha20 using the derived key

The recipient computes the same shared secret using their isk and the sender's epk: `ss = epk_sender * isk_recipient`. This yields the same symmetric key, allowing decryption.

## Owning vs Viewing

LEZ's key structure creates a natural separation between ownership and observation:

**Owning an account** requires the nullifier secret key (nsk). With the nsk, you can produce nullifiers to spend or update the account's state. You can also derive the npk and compute commitments.

**Viewing an account** requires the incoming viewing secret key (isk). With the isk (and the npk/vpk), you can detect incoming transfers via view tag matching and decrypt the encrypted account data. You can see the account's current state (balance, data, program owner) but you cannot spend it.

This means a watch-only wallet can be constructed by sharing the isk without exposing the nsk. The watch-only wallet can track balances and transaction history but cannot authorize any state changes.

## Program Derived Accounts (PDAs)

Some accounts are not owned by users but by programs. These use Program Derived Addresses:

```
PDA_AccountId = SHA256("/LSSA/v0.3/AccountId/PDA/" || program_id || seed)
```

PDAs are used for protocol-level accounts like AMM vaults and liquidity pool token definitions. When a program makes a chained call and provides PDA seeds, the called program automatically treats those PDA accounts as authorized. This enables programs to manage their own accounts without user signatures.
