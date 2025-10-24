# Chapter: Bitcoin Programmability and Script Execution

## Introduction: The Power of Programmable Money

Bitcoin represents far more than simple value transfer between addresses. At its core, Bitcoin implements a sophisticated system of programmable conditions that determine who can spend what, when, and under what circumstances. This programmability transforms Bitcoin from a simple payment system into a platform for executing arbitrary financial contracts without trusted intermediaries.

Understanding Bitcoin's programmability requires grasping a fundamental concept: every bitcoin is locked by a small computer program, and spending that bitcoin requires executing another program that successfully satisfies the conditions set by the first. This chapter explores how these programs work, when they execute, what they can express, and the standard patterns that have emerged for common use cases.

## Transaction Structure Revisited

Before diving into programmability, let's firmly establish the transaction structure that underpins everything in Bitcoin. This foundation must become second nature, as it forms the basis for all cryptocurrencies, not just Bitcoin.

### The Basic Building Blocks

Every Bitcoin transaction consists of **inputs** and **outputs**:

- **Inputs**: Represent money someone is spending
- **Outputs**: Represent money someone is receiving

Each output in a transaction contains exactly two fields:

1. **Amount**: The quantity of satoshis being sent
2. **Locking Script** (script pubkey): A program that establishes conditions for spending this output

This locking script is a contract. It specifies the requirements that must be met for someone to spend that amount in a future transaction.

### What Does "Spending" Mean?

To spend an output means to create a new transaction that references that output in one of its inputs. Let's call the first transaction TX₀ and the spending transaction TX₁.

When TX₁ spends output 0 from TX₀, this creates a permanent record in the blockchain. Once TX₁ is confirmed, output 0 of TX₀ can never be used again in any other transaction. This is the fundamental rule that prevents double-spending in Bitcoin.

### Input Structure

Each input in a transaction contains four fields:

1. **Transaction ID (TXID)**: Identifies which previous transaction contains the output being spent
2. **Index**: Specifies which output from that transaction (outputs are numbered 0, 1, 2, etc.)
3. **Unlocking Script** (script sig): A program that provides the data needed to satisfy the locking script
4. **Sequence**: A field used for certain timing semantics (which we'll explore later)

Together, the TXID and index form a pointer: "I want to spend output `index` from transaction `TXID`."

The unlocking script typically contains data like digital signatures and public keys, but we must be careful not to assume this is always the case. As we'll see, these scripts can contain arbitrary programs and data.

### The Crucial Combination

Here's the key insight: the unlocking script from the input and the locking script from the output being spent are combined to form a single program. This combined program must evaluate to true for the transaction to be valid. If it evaluates to false, the transaction is rejected—the spender has failed to satisfy the contract's conditions.

This is how Bitcoin enforces ownership and spending conditions without any central authority. The protocol itself validates that you've met the requirements to spend money.

## Script Execution Timing

Understanding **when** scripts execute is critical for building applications on Bitcoin. There are two distinct moments in time to consider:

### Time T₀: When TX₀ is Received

When a node first sees transaction TX₀, it must validate it. Part of this validation process involves executing scripts—but which scripts?

The node executes the unlocking scripts from TX₀'s inputs combined with the locking scripts from the outputs those inputs are spending. Each input is validated independently by running its unlocking script concatenated with the corresponding locking script from a previous transaction.

What the node does **not** execute at time T₀ is the locking script that appears in TX₀'s outputs. Why? Because those outputs haven't been spent yet. They're being created, not consumed.

### Time T₁: When TX₁ is Received

Later, at time T₁, when a node receives transaction TX₁ that attempts to spend one of TX₀'s outputs, now the node executes:

- The unlocking script from TX₁'s input
- Combined with the locking script from TX₀'s output

This temporal separation is crucial. The locking script establishes future conditions (a contract), while the unlocking script provides the proof that those conditions are met (fulfilling the contract).

Throughout this chapter, we'll examine various contracts, showing both the locking script (the contract itself) and the unlocking script (the spending conditions). Always remember: these execute at different times, in different transactions.

## Two Models of Programmability

Bitcoin introduced a particular model of computation that differs fundamentally from what came later with platforms like Ethereum. Understanding this distinction clarifies both Bitcoin's design philosophy and its limitations.

### The Bitcoin Model: Validation

Bitcoin's scripting system follows what we can call the **validation model**. In this paradigm:

- Scripts are extremely limited in their ability to **produce** new information
- Scripts excel at **validating** existing information
- Most computation happens outside the script, with results hard-coded into the transaction
- Scripts verify that these pre-computed results are correct

For example, Bitcoin scripts can:
- Validate digital signatures (checking if you are who you claim to be)
- Validate temporal conditions (checking if the current time meets requirements)
- Validate knowledge of secrets (checking if you know a preimage of a hash)
- Choose between different spending conditions (conditional logic: "this OR that")

What Bitcoin scripts cannot do efficiently:
- Perform arbitrary mathematical computations
- Iterate with loops
- Access external state or information
- Create complex data structures

The fundamental mechanism in Bitcoin scripts is conditional choice: "I can spend this if condition A is met, OR if condition B is met." This seemingly simple primitive enables surprisingly sophisticated contracts, as we'll see.

### The Ethereum Model: Computation

The alternative paradigm, exemplified by Ethereum, we can call the **computation model**:

- Scripts are Turing-complete (can express any computation)
- Smart contracts can produce new information, not just validate
- Contracts can maintain internal state across executions
- Arbitrary loops and complex data structures are supported

Ethereum's model is more expressive but introduces significant complexity in terms of security, cost management (gas), and potential for bugs. Bitcoin's simpler model avoids these issues at the cost of reduced expressiveness.

Neither model is inherently "better"—they represent different trade-offs in the design space. Bitcoin prioritizes security, predictability, and auditability over computational flexibility.

## Bitcoin Script: A Stack-Based Language

Bitcoin scripts are written in a simple, stack-based programming language called Bitcoin Script. Understanding this language is essential for grasping how Bitcoin's programmability works.

### Stack Basics

A stack is a Last-In-First-Out (LIFO) data structure. Think of it like a stack of plates:
- You can **push** a new plate onto the top
- You can **pop** the top plate off
- You can only access the top plate directly

In Bitcoin Script, all operations work with this stack. Data gets pushed onto the stack, operations manipulate the top elements, and the final result (at the top of the stack after all operations complete) determines if the script succeeds or fails.

### Execution Model

Script execution follows this process:

1. Start with an empty stack
2. Process the unlocking script from left to right:
   - Data values get pushed onto the stack
   - Opcodes (operations) manipulate the stack
3. Then process the locking script from left to right:
   - Continue pushing data and executing opcodes
4. After all operations complete, examine the top of the stack:
   - If the top element is true (non-zero), the script succeeds
   - If the top element is false (zero) or the stack is empty, the script fails

### A Simple Example

Let's trace through a trivial script to see how this works:

**Locking Script**: `8`
**Unlocking Script**: `8 OP_EQUAL`

Wait—these scripts seem backwards from our earlier description. Actually, for this example, we're showing both pieces, and they'll be executed in order: unlocking script first, then locking script.

Let's execute this combined script: `8 OP_EQUAL 8`

1. Start with an empty stack: `[]`
2. Push `8` onto the stack: `[8]`
3. Execute `OP_EQUAL`: This opcode pops two items from the stack and pushes `true` if they're equal, `false` otherwise. But we only have one item! This would actually fail.

Let me correct this example:

**Locking Script**: `OP_EQUAL`
**Unlocking Script**: `8 8`

Combined execution: `8 8 OP_EQUAL`

1. Start: `[]`
2. Push `8`: `[8]`
3. Push `8`: `[8, 8]`
4. Execute `OP_EQUAL`: Pops `8` and `8`, compares them, pushes `true`: `[true]`
5. Script succeeds because the top of the stack is `true`

### A More Realistic Example

**Locking Script**: `8 OP_EQUAL`
**Unlocking Script**: `8`

Combined: `8 8 OP_EQUAL`

This is identical to our previous example. The locking script establishes a contract: "To spend this, you must provide the number 8."

The unlocking script provides: "Here is 8."

When combined and executed, they produce `true`, and the transaction is valid.

### Important Properties

Several critical properties of this execution model:

**Trivially Breakable**: The contract `8 OP_EQUAL` is trivial to satisfy—anyone can see that they need to provide `8`. This isn't a security flaw; it's an intentionally simple example. Real contracts use cryptographic primitives to ensure only authorized parties can spend funds.

**Semantic Equivalence**: Multiple programs can achieve the same result. For example, these are semantically equivalent to our previous example:

- `4 OP_DUP OP_ADD 8 OP_EQUAL`
- `2 2 OP_ADD OP_DUP OP_ADD 8 OP_EQUAL`

Both produce `8` through different computations, then compare it to `8`. This flexibility becomes important for a subtle purpose: encoding arbitrary data in transactions.

## Writing Data to the Blockchain

A seemingly tangential question reveals important insights into Bitcoin's structure: How can we write arbitrary data to the blockchain?

### Where Can Data Live?

Consider what actually gets stored in Bitcoin blocks:

1. **Block Header** (80 bytes): Contains specific fields with defined purposes—no room for arbitrary data
2. **Transactions**: The only place with any flexibility

Within transactions, what can we control?

- **Version** (4 bytes): Fixed value
- **Lock time** (4 bytes): Has semantic meaning, can't be arbitrary
- **Inputs**:
  - TXID: Must reference a real, valid transaction
  - Index: Must reference a valid output
  - Sequence: Has semantic meaning
  - **Unlocking Script**: Flexible content! ✓
- **Outputs**:
  - Amount: Represents value, not arbitrary data
  - **Locking Script**: Flexible content! ✓

The unlocking and locking scripts are the **only** places we can write arbitrary data in Bitcoin.

### The Constraint: Scripts Must Be Valid

However, we can't just write random bytes. These scripts are executed as programs—they must produce valid results. If we want to write data to the blockchain, we must embed that data within scripts that still evaluate correctly.

This creates an interesting challenge: How do we write arbitrary data while maintaining script validity?

Several techniques have emerged:

**OP_RETURN** (which we'll explore in detail shortly): Explicitly allows embedding data by making the output unspendable

**False Branches**: Using conditional opcodes to include data that never gets executed

**Stack Manipulation**: Pushing data onto the stack then immediately discarding it

**Encoding in Valid Programs**: Creating unnecessarily complex programs that happen to contain your data

### Historical Data on the Blockchain

The Bitcoin blockchain contains numerous examples of embedded data:

- The **Genesis Block** coinbase includes the famous Times newspaper headline: "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks"
- The **Bitcoin whitepaper** PDF, encoded across multiple transactions
- Various images, text files, and even controversial content
- Recent innovations like Ordinals have pushed this concept further

The coinbase transaction (the first transaction in every block) follows special rules that allow more arbitrary data in its unlocking script, since it doesn't actually spend any previous output.

## Second-Layer Interpretations

Understanding arbitrary data in Bitcoin leads to an important concept: **second-layer systems**.

A second-layer system consists of:

1. **Bitcoin nodes**: Running the normal Bitcoin protocol, validating transactions and blocks
2. **Specialized clients**: Reading the blockchain but interpreting certain data differently

These specialized clients treat the blockchain as a database. They scan through transactions looking for specific patterns or data structures that have meaning in their protocol, while the underlying Bitcoin nodes remain unaware of this additional layer of meaning.

Examples of second-layer concepts include:

- **Colored coins**: Marking specific satoshis with additional meaning
- **Timestamps**: Proof-of-existence protocols
- **Token protocols**: Creating assets beyond bitcoin itself
- **Ordinals and inscriptions**: Recent innovations in NFT-like functionality

The Bitcoin blockchain simply records the data. The second-layer clients give that data meaning.

This is a powerful pattern: Bitcoin provides a secure, timestamped, immutable database; applications build whatever logic they need on top.

---

## Standard Contracts

With the theoretical foundation established, let's examine the practical contracts that Bitcoin actually uses. These contracts have emerged as standardized patterns for common use cases.

But first, we need to understand an important distinction in Bitcoin: **standard** versus **non-standard** transactions.

### Standard vs. Non-Standard

Bitcoin's consensus rules define what makes a transaction **valid**. If a transaction follows all consensus rules, it can be included in a valid block, and all nodes will accept it.

However, there's a separate concept: **standardness**. Standard transactions are those that follow specific templates or patterns. This distinction matters because of **policy rules** versus **consensus rules**.

**Consensus Rules**: Enforced by all nodes, define validity, cannot be violated without creating an invalid block

**Policy Rules**: Local preferences about which valid transactions to relay and mine. Nodes can have different policies.

The default policy in Bitcoin is:
- **Relay and propagate** transactions that are standard
- **Do not relay** transactions that are non-standard (even if valid)
- **Mine either**, but most miners prefer standard transactions

This means non-standard transactions are valid and can be included in blocks, but they face difficulty propagating through the network. Miners can still include them directly if they receive them through other channels.

Why this distinction? Standard templates have been vetted for safety and efficiency. Non-standard patterns might be valid but unusual, potentially indicating bugs or attacks. This policy provides a conservative default while maintaining the flexibility to include any valid transaction.

### Contract Pattern 1: Anyone Can Spend

The simplest possible Bitcoin contract is one that anyone can spend. This might seem useless—why create money that anyone can take? But it serves important pedagogical and testing purposes.

**Locking Script**: `OP_TRUE`

`OP_TRUE` is an opcode that simply pushes the value `true` onto the stack. That's it. The script immediately succeeds.

**Unlocking Script**: (empty)

You don't need to provide anything to spend this output. The locking script succeeds on its own.

**Combined Execution**: `OP_TRUE`

1. Start: `[]`
2. Execute `OP_TRUE`: `[true]`
3. Top of stack is `true` → success

Anyone who sees this output can spend it by creating a transaction with an empty unlocking script. This is a genuinely open contract—first come, first served.

**Use Cases:**
- Testing and development
- Demonstration purposes
- Intentional giveaways or faucets (though better mechanisms exist)
- Illustrating that Bitcoin's security comes from cryptographic conditions, not obscurity

### Contract Pattern 2: No One Can Spend

The opposite extreme: creating outputs that are provably unspendable.

**Locking Script**: `OP_RETURN <data>`

`OP_RETURN` is a special opcode that immediately terminates script execution with a failure. No matter what the unlocking script contains, once execution reaches `OP_RETURN`, the script fails.

**Unlocking Script**: (anything—it doesn't matter)

**Combined Execution**: `<anything> OP_RETURN <data>`

1. Process the unlocking script operations
2. Hit `OP_RETURN`
3. Script immediately fails

No unlocking script can satisfy this contract. The funds are permanently unspendable.

**Why Would Anyone Do This?**

The key insight: we can include arbitrary data after `OP_RETURN`. Since the script fails immediately upon reaching `OP_RETURN`, it doesn't matter what comes after—those bytes are never interpreted as opcodes. They're just data.

This provides a clean, explicit way to embed data in transactions:

**Locking Script**: `OP_RETURN 0x48656c6c6f20576f726c64`

The hex data decodes to "Hello World". This data is now permanently part of the blockchain, but the output is provably unspendable.

### OP_RETURN Standardness Rules

Because `OP_RETURN` outputs can be used to bloat the blockchain with arbitrary data, Bitcoin has strict standardness rules:

1. A transaction can contain **at most one** output with `OP_RETURN`
2. That output must have an **amount of zero satoshis**
3. The data after `OP_RETURN` can be **at most 83 bytes**

Transactions violating these rules are non-standard (but still valid if miners include them).

**Modern Uses:**
- Timestamping documents (storing hashes)
- Merkle roots of off-chain data
- Colored coin protocols
- Various token and NFT schemes
- Anchoring side-chains

The beauty of `OP_RETURN`: it creates an output that nodes don't need to track in their UTXO set. Since it's provably unspendable, it can be immediately discarded, reducing the burden on node operators.

### Contract Pattern 3: Pay to Public Key (P2PK)

Now we move to cryptographic contracts—ones that only specific parties can satisfy. The first such pattern is Pay to Public Key.

**Locking Script**: `<pubkey> OP_CHECKSIG`

This script contains:
- A public key (33 or 65 bytes, depending on compressed vs. uncompressed format)
- The `OP_CHECKSIG` opcode

`OP_CHECKSIG` is one of Bitcoin's most important opcodes. It verifies an ECDSA digital signature.

**Unlocking Script**: `<signature>`

To spend this output, you must provide a valid signature that corresponds to the public key specified in the locking script.

**Combined Execution**: `<signature> <pubkey> OP_CHECKSIG`

1. Push signature onto stack: `[<sig>]`
2. Push public key onto stack: `[<sig>, <pubkey>]`
3. Execute `OP_CHECKSIG`:
   - Pops public key and signature
   - Verifies the signature is valid for this public key and this transaction
   - Pushes `true` if valid, `false` if invalid
4. Result: `[true]` or `[false]`

If the signature is valid, the script succeeds and the transaction is accepted.

**What Is Being Signed?**

This is crucial: the signature doesn't just prove "I have the private key." It signs **specific details of the spending transaction**:

- Which outputs are being spent (inputs)
- Where the money is going (outputs)  
- Other transaction metadata

This prevents someone from taking your signature for one transaction and reusing it in a different transaction. The signature cryptographically commits to the transaction details.

**Historical Note:**

P2PK was the original payment pattern in early Bitcoin. Satoshi's coins were sent to P2PK outputs. However, it has largely been replaced by the more flexible P2PKH pattern.

**Limitations:**
- The public key is exposed before spending, which may have future quantum computing implications
- The public key takes up significant space in the blockchain
- Less privacy—the public key is visible in the locking script

### Contract Pattern 4: Pay to Public Key Hash (P2PKH)

This is Bitcoin's most common classical payment pattern—what most people think of as "sending bitcoin to an address."

**Locking Script**: `OP_DUP OP_HASH160 <pubkey_hash> OP_EQUALVERIFY OP_CHECKSIG`

This script is more complex. Let's break down the opcodes:

- `OP_DUP`: Duplicates the top item on the stack
- `OP_HASH160`: Pops an item, applies SHA-256 then RIPEMD-160 hashing, pushes result
- `<pubkey_hash>`: The hash of the recipient's public key (20 bytes)
- `OP_EQUALVERIFY`: Pops two items, verifies they're equal, fails if not
- `OP_CHECKSIG`: Verifies a signature as before

**Unlocking Script**: `<signature> <pubkey>`

To spend, you provide both your signature and your public key.

**Combined Execution**: `<signature> <pubkey> OP_DUP OP_HASH160 <pubkey_hash> OP_EQUALVERIFY OP_CHECKSIG`

Let's trace through this carefully:

1. Push signature: `[<sig>]`
2. Push public key: `[<sig>, <pubkey>]`
3. `OP_DUP`: Duplicate top item: `[<sig>, <pubkey>, <pubkey>]`
4. `OP_HASH160`: Hash the top item: `[<sig>, <pubkey>, <hash_of_pubkey>]`
5. Push the required hash: `[<sig>, <pubkey>, <hash_of_pubkey>, <required_hash>]`
6. `OP_EQUALVERIFY`: Pop two items, verify they're equal, fail if not: `[<sig>, <pubkey>]`
7. `OP_CHECKSIG`: Verify signature with public key: `[true]` or `[false]`

**Why This Complexity?**

P2PKH offers several advantages over P2PK:

**Privacy**: The public key isn't revealed until spending time. The locking script contains only a hash. This provides protection against future cryptographic attacks (like quantum computers) that haven't happened yet.

**Address Format**: The public key hash can be encoded as a Bitcoin address (starting with `1` in legacy format). This is shorter and more user-friendly than a raw public key.

**Validation**: The script ensures that the public key provided in the unlocking script actually hashes to the expected value, then verifies the signature. This prevents someone from substituting a different public key.

**Standardness**: P2PKH is the standard for regular Bitcoin transactions. Nearly all wallet software supports creating and spending P2PKH outputs.

### Understanding Bitcoin Addresses

Bitcoin addresses are simply encoded public key hashes with additional metadata:

1. Start with a public key
2. Hash it with SHA-256, then RIPEMD-160 (producing 20 bytes)
3. Add a version byte (0x00 for mainnet P2PKH)
4. Calculate a checksum (first 4 bytes of SHA-256(SHA-256(version+hash)))
5. Encode in Base58

The result is a string like: `1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa`

This encoding makes addresses:
- Shorter than raw public keys
- More readable (Base58 excludes confusing characters like 0, O, I, l)
- Protected by a checksum (typos are detected)

When you "send to an address," you're creating a P2PKH output with the decoded hash.

### Contract Pattern 5: Pay to Script Hash (P2SH)

P2SH represents a major innovation in Bitcoin, enabling much more complex contracts while maintaining simple addresses.

**The Problem**: Suppose we want a multi-signature contract requiring 2 of 3 parties to sign. The locking script would be quite long. The sender would need to know all the details of this complex contract. The addresses would be unwieldy.

**The Solution**: Let the **recipient** specify the full contract details, and let the **sender** just commit to a hash of that contract.

**Locking Script**: `OP_HASH160 <script_hash> OP_EQUAL`

The locking script contains only:
- `OP_HASH160`: The hash operation
- A 20-byte hash of the actual redeem script
- `OP_EQUAL`: Verification

**Unlocking Script**: `<data>` `<serialized_redeem_script>`

The unlocking script must provide:
- Whatever data the redeem script requires (signatures, etc.)
- The actual redeem script whose hash matches the locking script

**Two-Phase Execution:**

P2SH is unique—it executes in two phases:

**Phase 1**: Verify the script hash

1. Push unlocking script data onto stack
2. Push serialized redeem script: `[<data>, <redeem_script>]`
3. `OP_HASH160`: Hash the redeem script: `[<data>, <hash_of_redeem_script>]`
4. Push required hash: `[<data>, <hash_of_redeem_script>, <required_hash>]`
5. `OP_EQUAL`: Verify hashes match: `[<data>, true]`

If this fails, the transaction is invalid.

**Phase 2**: Execute the redeem script

If phase 1 succeeds, the redeem script is deserialized and executed with the remaining stack data:

6. Deserialize redeem script into opcodes
7. Execute it with `<data>` already on the stack

**Example: 2-of-3 Multisig**

Redeem script: `2 <pubkey1> <pubkey2> <pubkey3> 3 OP_CHECKMULTISIG`

This requires 2 signatures from among 3 public keys.

Locking script (P2SH): `OP_HASH160 <hash_of_redeem_script> OP_EQUAL`

Unlocking script: `0 <sig1> <sig2> 2 <pubkey1> <pubkey2> <pubkey3> 3 OP_CHECKMULTISIG`

(The `0` is due to a bug in `OP_CHECKMULTISIG` that we must work around—it pops an extra item)

**Advantages:**

**Sender Simplicity**: The sender only needs a hash, encoded as an address (starting with `3` in legacy format)

**Recipient Control**: Recipients define arbitrarily complex spending conditions

**Privacy**: The full contract details aren't revealed until spending time

**Flexibility**: Enables multi-signature wallets, time-locked contracts, and many advanced patterns

**Upgradability**: New contract types can be deployed without changing how senders create transactions

P2SH addresses start with `3` (mainnet) and are created by hashing the redeem script:

1. Create the redeem script
2. Hash with SHA-256, then RIPEMD-160
3. Add version byte (0x05 for mainnet P2SH)
4. Checksum and Base58 encode

### Contract Pattern 6: Multi-Signature Scripts

Multi-signature contracts require multiple parties to authorize spending. While often used through P2SH, the underlying opcodes deserve explanation.

**Locking Script**: `<m> <pubkey1> <pubkey2> ... <pubkeyn> <n> OP_CHECKMULTISIG`

Where:
- `m`: Minimum number of signatures required
- `n`: Total number of public keys
- Public keys listed explicitly

Common patterns:
- 2-of-2: Both parties must sign
- 2-of-3: Any 2 of 3 parties must sign  
- 3-of-5: Any 3 of 5 parties must sign

**Unlocking Script**: `0 <sig1> <sig2> ... <sigm>`

Provides m valid signatures (and a dummy 0 due to an off-by-one bug in `OP_CHECKMULTISIG`).

**Execution:**

`OP_CHECKMULTISIG` is complex:
1. Pops n (number of public keys)
2. Pops n public keys
3. Pops m (number of signatures required)
4. Pops m signatures
5. Pops one additional item (the bug)
6. Verifies that at least m of the signatures are valid for public keys in the list
7. Pushes true if valid, false otherwise

**Use Cases:**

**Escrow**: 2-of-3 where buyer, seller, and arbitrator each hold a key. Normal transactions need buyer + seller. Disputes need two of the three parties.

**Corporate Wallets**: Requiring multiple executives to approve large transactions

**Personal Security**: 2-of-3 where you hold two keys in different locations and a trusted party holds a third as backup

**Threshold Security**: Distributing trust among multiple parties, increasing security even if some keys are compromised

**Limitations:**

**Size**: Multi-signature scripts are large. Each public key is 33 bytes, and they're all included in the locking script (or redeem script for P2SH).

**Privacy**: When spending, all public keys are revealed, even those that didn't sign.

**Fixed Structure**: The m-of-n structure is rigid. More complex policies (like "Alice OR Bob OR (Carol AND Dave)") require different constructions.

Later innovations like Taproot introduce more efficient multi-signature schemes using Schnorr signatures.

### Contract Pattern 7: Time-Locked Contracts

Bitcoin supports time-based conditions: "This money can't be spent until time T" or "This money can't be spent until block height N."

There are two mechanisms for time locks, operating at different levels:

**Transaction-Level Time Lock**: `nLockTime` field
**Output-Level Time Lock**: `OP_CHECKLOCKTIMEVERIFY` (CLTV) opcode

#### Transaction-Level: nLockTime

Every transaction has a 4-byte `nLockTime` field that specifies the earliest time the transaction can be included in a block:

- If `nLockTime` < 500,000,000: Interpreted as block height
- If `nLockTime` ≥ 500,000,000: Interpreted as Unix timestamp

**Rules:**
- If `nLockTime = 0`: No time lock (can be included immediately)
- If `nLockTime ≠ 0`: Transaction cannot be included before that height/time
- All inputs must have `sequence < 0xFFFFFFFF` for nLockTime to be enforced

This allows creating transactions that become valid at a future time.

**Use Case**: Alice wants to give Bob money on his birthday next year. She creates and signs a transaction now with nLockTime set to his birthday, and gives him the signed transaction. He can't spend it until then, but Alice has already authorized it.

#### Output-Level: OP_CHECKLOCKTIMEVERIFY

`OP_CHECKLOCKTIMEVERIFY` (CLTV) creates outputs that can only be spent after a certain time.

**Locking Script**: `<time> OP_CHECKLOCKTIMEVERIFY OP_DROP <pubkey> OP_CHECKSIG`

**Unlocking Script**: `<signature>`

**Execution:**

1. Push time value: `[<time>]`
2. `OP_CHECKLOCKTIMEVERIFY`: Verifies that:
   - The spending transaction's nLockTime is ≥ this time value
   - The input's sequence is < 0xFFFFFFFF
   - If valid, leaves stack unchanged; if invalid, fails immediately
3. Stack still has: `[<time>]`
4. `OP_DROP`: Removes the time value: `[]`
5. Continue with normal signature checking...

**Key Points:**

**Double Lock**: CLTV doesn't enforce the time lock itself—it requires the spending transaction to have an appropriate nLockTime. This creates a two-layer verification.

**Output-Level**: Unlike nLockTime which applies to entire transactions, CLTV applies to individual outputs. Different outputs in the same transaction can have different time locks.

**Auditability**: The time lock is visible in the locking script, making it verifiable without seeing the spending transaction.

**Use Cases:**

**Vesting**: Salary payments that unlock gradually over time

**Inheritance**: Funds that can only be claimed after a certain date

**Bonds**: Time-locked deposits in financial contracts

**Refunds**: Money that returns to the sender if not claimed by a deadline (combined with other conditions)

#### Sequence-Based Time Locks: OP_CHECKSEQUENCEVERIFY

A related mechanism uses relative time locks based on the age of the UTXO being spent.

**The Sequence Field Revisited:**

Remember the 4-byte sequence field in each input? When used for relative time locks:

- Bit 31 (highest bit): If set, disables relative time lock
- Bit 22: Determines interpretation
  - If set: Time lock is in block time (512 seconds per unit)
  - If clear: Time lock is in block height (blocks)
- Lower bits: The time/height value

**OP_CHECKSEQUENCEVERIFY** (CSV) verifies that the UTXO being spent is old enough:

**Locking Script**: `<relative_time> OP_CHECKSEQUENCEVERIFY OP_DROP <pubkey> OP_CHECKSIG`

**Execution:**

CSV verifies that the input's sequence value is compatible with the required relative time lock. The UTXO must have been confirmed in a block at least `<relative_time>` blocks (or time units) ago.

**Use Case: Payment Channels**

Alice and Bob open a payment channel. They create a transaction that allows either party to close the channel, but with a time delay. This gives the other party time to dispute an invalid closure.

Locking script: "After 1 day, Alice can take all the funds, OR Bob can take them with proof of a newer state"

This pattern is fundamental to the Lightning Network.

### Contract Pattern 8: Hash Locks (Atomic Swaps)

Hash locks enable trustless exchanges across different blockchains—atomic swaps.

**Concept**: Lock money with a hash puzzle. Anyone who can reveal the preimage of a specific hash can claim the money.

**Locking Script**: `OP_HASH160 <hash_of_secret> OP_EQUALVERIFY <pubkey> OP_CHECKSIG`

This requires:
1. Knowledge of the secret that hashes to the specified value
2. A valid signature from the recipient

**Unlocking Script**: `<signature> <secret>`

**Execution:**

1. Push signature: `[<sig>]`
2. Push secret: `[<sig>, <secret>]`
3. `OP_HASH160`: Hash the secret: `[<sig>, <hash_of_secret>]`
4. Push required hash: `[<sig>, <hash_of_secret>, <required_hash>]`
5. `OP_EQUALVERIFY`: Verify hashes match or fail: `[<sig>]`
6. Push public key: `[<sig>, <pubkey>]`
7. `OP_CHECKSIG`: Verify signature: `[true]`

**Atomic Swap Protocol:**

Alice has bitcoin. Bob has litecoin. They want to exchange without trust.

**Setup:**
1. Alice generates a secret S
2. Alice computes H = hash(S)

**Bitcoin Transaction** (Alice → Bob):
- Locking script: "Bob can claim with H preimage and his signature, OR Alice can reclaim after 24 hours"

**Litecoin Transaction** (Bob → Alice):
- Locking script: "Alice can claim with H preimage and her signature, OR Bob can reclaim after 12 hours"

**Execution:**
1. Both transactions are broadcast
2. Alice claims the litecoin by revealing S
3. Now Bob knows S (it's public in the blockchain)
4. Bob uses S to claim the bitcoin
5. Both parties have successfully exchanged, trustlessly

**Atomicity**: Either both transactions complete, or neither does. If Alice never reveals S, both parties can reclaim their funds after the time locks expire.

This pattern extends to the Lightning Network, where hash locks enable secure multi-hop payments across payment channels.

## Script Limitations and Safety Features

Bitcoin Script is intentionally limited to prevent various attacks and ensure script execution is predictable and safe.

### Disabled Opcodes

Many opcodes are disabled and will cause immediate script failure if encountered:

- String manipulation opcodes
- Bitwise operations beyond basic boolean logic  
- Several arithmetic opcodes
- `OP_CAT` (concatenation)
- `OP_SUBSTR` (substring extraction)

These were disabled due to security concerns or potential for creating excessively large scripts.

### Resource Limits

Scripts face strict resource limits:

**Maximum Script Size**: 10,000 bytes
**Maximum Stack Size**: 1,000 items
**Maximum Item Size**: 520 bytes
**Maximum Operation Count**: 201 opcodes executed

These limits prevent denial-of-service attacks where excessively complex scripts consume excessive CPU or memory.

### No Loops

Bitcoin Script is not Turing-complete. There are no looping constructs. You cannot write:

```
while (condition) {
    do_something
}
```

This ensures all scripts terminate in finite, predictable time. Miners can calculate exactly how long validation will take.

### No State or Persistence

Scripts cannot:
- Read or write blockchain state
- Access information about other transactions
- Maintain variables across executions
- Call other contracts

Each script execution is isolated, deterministic, and purely functional—given the same inputs, it always produces the same outputs.

These limitations might seem restrictive, but they serve crucial purposes:
- **Security**: Simple scripts are easier to audit and verify
- **Predictability**: Validation time is bounded and calculable  
- **Consensus**: All nodes can independently verify scripts with identical results
- **Scalability**: Simple scripts process quickly

## The Standardness Policy Framework

Let's revisit the distinction between valid and standard transactions with more precision.

### Consensus Rules (Validity)

Consensus rules are enforced by the protocol. Violating them creates an invalid transaction or block that all nodes reject:

- Transaction inputs must reference unspent outputs
- Scripts must execute successfully
- Signatures must be valid
- Amounts must balance correctly (inputs ≥ outputs + fees)
- No double-spending
- Block proof-of-work must meet difficulty target

These rules are unbreakable. They define what Bitcoin is.

### Policy Rules (Standardness)

Policy rules are local preferences about which valid transactions to relay:

- Minimum fee rate (usually 1 satoshi per byte)
- Maximum transaction size
- Allowed script templates
- Rules about OP_RETURN usage
- Limits on signature operations

These rules can differ between nodes. They're conventions, not laws.

### Standard Script Templates

The standard script templates accepted by default Bitcoin Core nodes:

1. **P2PKH**: Pay to Public Key Hash
2. **P2PK**: Pay to Public Key (becoming rare)
3. **P2SH**: Pay to Script Hash
4. **Multisig**: Direct multisig (limited to 3 keys for standardness)
5. **OP_RETURN**: Data embedding (with restrictions)
6. **Null Data**: Provably unspendable outputs
7. **Witness Programs** (Segwit types, which we'll cover next lecture)

Transactions using other patterns are non-standard but can still be valid.

### Why Non-Standard Transactions Exist

Non-standard transactions serve several purposes:

**Experimentation**: Testing new contract types before they become standard

**Special Use Cases**: Unusual but legitimate patterns that don't fit templates

**Protocol Upgrades**: New features may be valid but not yet standard

**Direct Mining**: Non-standard transactions can be submitted directly to miners who choose to include them

The standardness framework provides a conservative default while maintaining Bitcoin's permissionless nature—you can always create any valid transaction, even if most nodes won't relay it.

## Practical Implications for Developers

Understanding Bitcoin's programmability has practical implications:

### Transaction Construction

When building applications:

1. **Choose the Right Pattern**: Use standard templates when possible (P2PKH, P2SH)
2. **Consider Privacy**: P2PKH hides public keys until spending; P2PK exposes them
3. **Plan for Fees**: Complex scripts cost more to validate, requiring higher fees
4. **Test Thoroughly**: Non-standard scripts may not propagate as expected
5. **Think About Upgrades**: Consider using P2SH for flexibility in spending conditions

### Security Considerations

**Script Correctness**: A mistake in a locking script might make funds unspendable or insecurely spendable

**Signature Coverage**: Understand what parts of transactions signatures commit to

**Time Lock Risks**: Ensure time locks align with your security model

**Key Management**: Complex scripts often require managing multiple keys securely

**Testing**: Always test scripts on testnet before using on mainnet

### Privacy Patterns

Script design affects privacy:

- **P2PKH**: Hides public keys until spending
- **P2SH**: Hides full contract until spending
- **Script Reuse**: Using identical scripts across transactions links them
- **Amount Patterns**: Similar amounts across related scripts can reveal connections

Later innovations like Taproot dramatically improve script privacy.

## Conclusion: The Foundation of Programmable Money

Bitcoin's script system, while limited compared to Turing-complete smart contract platforms, provides a powerful foundation for programmable money. By focusing on validation rather than computation, Bitcoin achieves:

**Security**: Simple, auditable scripts that are easier to verify and harder to exploit

**Predictability**: Bounded execution time and resource consumption

**Flexibility**: Despite limitations, creative use of opcodes enables sophisticated contracts

**Upgradeability**: New patterns can be deployed within the existing framework

The standard contracts we've examined—P2PKH, P2SH, multi-signature, and time locks—cover the vast majority of use cases. More exotic applications build on these primitives, either directly or through second-layer protocols.

Understanding script execution timing, the validation model, and standard patterns provides the foundation for everything built on Bitcoin. From simple payments to complex protocols like the Lightning Network, it all comes down to carefully crafted combinations of these basic building blocks.

In the next chapter, we'll explore how Bitcoin evolved its programmability through Segregated Witness (SegWit), addressing critical malleability issues and enabling new capabilities. Following that, we'll examine the Taproot upgrade, which brings Schnorr signatures and dramatically improved privacy and efficiency to Bitcoin's smart contracts.

The journey from basic "anyone can spend" contracts to sophisticated multi-party, time-locked, hash-locked protocols demonstrates Bitcoin's elegant design: powerful enough for complex applications, simple enough to verify and trust.

---

*This chapter provided a comprehensive exploration of Bitcoin's programmability model. In the next lecture, we'll examine the SegWit upgrade, which revolutionized how Bitcoin handles transaction signatures and paved the way for second-layer scaling solutions.*
