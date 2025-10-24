# Chapter 12: Protocol Evolution - SegWit and Taproot

## Introduction: Building on Programmability

We've seen how Bitcoin's script language enables programmable money through smart contracts. Now we'll explore how the protocol itself has evolved to solve critical problems while maintaining backward compatibility with older nodes. This chapter examines two major protocol upgrades: Segregated Witness (SegWit) from 2017 and Taproot from 2021, which represent significant advances in Bitcoin's capabilities.

Before diving into these upgrades, let's review the fundamental structure we're building upon. A Bitcoin transaction consists of inputs (money being spent) and outputs (money being received). Each output contains a **lock script** (the contract), which establishes conditions for authorizing future spending. When a subsequent transaction wants to spend that output, it must provide an **unlock script** that produces data validated by the lock script. These scripts are executed when the spending transaction is validated on the network, not when the output is created.

## Unspendable Contracts: OP_RETURN

Beyond spendable contracts, Bitcoin allows for deliberately unspendable outputs. The simplest example is `OP_RETURN`, which creates what we call "no one can spend" contracts.

### How OP_RETURN Works

Regardless of what data you place in the unlock script, when execution reaches `OP_RETURN` in the lock script, the script fails immediately. There's no possible unlock script that can satisfy this contract. While this might seem pointless, it serves an important purpose: **arbitrary data storage**.

Since `OP_RETURN` appears in the output's lock script, we can include data after the opcode. This data becomes permanently recorded in the blockchain. This represents the most primitive form of arbitrary data registration in Bitcoin.

### Standardness Rules

Here we encounter an important distinction: **standard** versus **non-standard** transactions. Standard transactions follow specific templates; non-standard transactions do not. This distinction matters for policy, not validity. A non-standard transaction can still be valid according to consensus rules.

Why does this matter? Because of mempool policies. When nodes receive valid transactions or blocks, they must decide what to propagate. The default network policy is to announce and relay standard transactions while refusing to propagate non-standard ones. This makes it harder to broadcast unusual transactions, though it doesn't prevent their inclusion in blocks if a miner chooses to include them.

For `OP_RETURN`, the standardness rule is:
- Transactions may include exactly one output with zero amount
- The script must be `OP_RETURN` followed by data
- Maximum data size: 83 bytes

This limitation exists because if everyone could freely register unlimited arbitrary data on-chain, the system would become bloated and difficult to maintain.

## The Transaction Malleability Problem

Now we arrive at one of Bitcoin's most significant historical problems, which SegWit was designed to solve: **transaction malleability**.

### Understanding the Problem

Transaction malleability means that multiple valid versions of the same transaction can exist with different transaction IDs (TXIDs). This happens because TXIDs are calculated from the entire transaction, including the signature data. But here's the catch: you can create different valid signatures for the same spending authorization.

Let's trace through how this creates problems:

1. Alice creates a transaction spending output X to pay Bob
2. The transaction includes Alice's signature in the unlock script
3. The TXID is calculated from the entire transaction, including the signature
4. Alice could create a different valid signature for the same transaction
5. This produces a different TXID, even though both transactions are semantically identical
6. Both versions spend the same coins and create the same outputs, but they have different identifiers

Why is this a problem? Consider a more complex scenario:

1. Transaction TX0 creates an output that Alice owns
2. Transaction TX1 spends TX0 and pays Bob
3. Transaction TX2 spends an output from TX1
4. If TX1's signature changes, its TXID changes
5. Now TX2 is referencing an invalid TXID
6. TX2 becomes invalid, even though nothing semantically changed

This breaks any protocol that needs to reference unconfirmed transactions in a chain of dependencies. You can't reliably build transaction chains because intermediate TXIDs might change before confirmation.

### Real-World Impact

This problem particularly affected early Lightning Network development and other layer-two protocols that required creating chains of unconfirmed transactions. These protocols needed to construct complex transaction trees where later transactions spent outputs from earlier ones, all before any confirmations. Transaction malleability made this fundamentally unreliable.

## Segregated Witness (SegWit): The Solution

SegWit, activated in August 2017, solved transaction malleability while simultaneously addressing several other protocol limitations. The solution is conceptually elegant, though technically intricate.

### The Core Idea

The fundamental insight: separate the signature data (the **witness**) from the transaction proper. Here's how it works:

**Traditional Transaction Structure:**
```
[Version] [Input Count] [Inputs with Unlock Scripts] [Output Count] [Outputs] [Locktime]
```

**SegWit Transaction Structure:**
```
[Version] [Marker] [Flag] [Input Count] [Inputs - No Scripts!] 
[Output Count] [Outputs] [Witness Data] [Locktime]
```

The key changes:
1. **Marker** (0x00) and **Flag** (0x01): Signal that this is a SegWit transaction
2. **Empty unlock scripts**: Inputs contain no script data
3. **Separate witness section**: Signature data moves to a new structure at the end

### How Witness Data Works

The witness section contains a stack of data items for each input. When a SegWit script executes:

1. Start with a pre-populated stack from the witness data
2. Execute the lock script with this initial stack
3. The lock script validates the witness data

Effectively, we moved the data that was at the beginning of the unlock script into the separate witness section. It's the same data, just relocated.

### Solving Malleability

Now here's the crucial part: **how does this solve malleability?**

The TXID calculation had to remain unchanged to avoid breaking pre-SegWit nodes. Those nodes don't see the witness data (shown in blue in diagrams), only the black parts. Therefore:

**The TXID is calculated from the transaction WITHOUT the witness data.**

Since signatures now live in the witness section and don't enter the TXID calculation, you can't modify the TXID by changing signatures. All other transaction fields are determined by the spending logic and can't be freely modified. Problem solved!

### The Witness Transaction ID (WTXID)

But removing signatures from the TXID creates a new problem: we use TXIDs to calculate the Merkle root for the proof-of-work calculation. If we exclude signatures, someone could modify a transaction's witness data after block confirmation without invalidating the proof of work. This would allow post-confirmation transaction modification.

The solution: introduce a second identifier called the **Witness Transaction ID (WTXID)**:

- **TXID**: Hash of the transaction without witness data
- **WTXID**: Hash of the complete transaction including witness data

The protocol then requires:
1. Build a second Merkle tree from WTXIDs
2. Commit the WTXID Merkle root in the coinbase transaction
3. This covers all witness data under the proof of work

This ensures that once a block is mined, neither the transaction data nor the witness data can be modified without invalidating the proof of work.

### Backward Compatibility Through Network Segregation

SegWit effectively segregates the network into two groups:

**Pre-SegWit (Legacy) Nodes:**
- Never see witness data (the blue parts)
- Cannot validate SegWit transactions fully
- Still enforce basic transaction structure rules
- Treat SegWit transactions as anyone-can-spend (we'll see why shortly)

**SegWit Nodes:**
- See and validate complete transactions including witness data
- Enforce additional validation rules for SegWit transactions
- Can communicate with both legacy and SegWit nodes

The network protocol includes mechanisms for nodes to identify each other's capabilities. SegWit nodes don't send witness data to legacy nodes, knowing they can't process it.

### Future Upgrade Mechanism

Notice the **flag** field in SegWit transactions? Currently it must be 0x01, but this provides an upgrade mechanism for the future. If the flag changes to 0x02 or another value:

- SegWit version 0 nodes validate the transaction as valid regardless of content
- Future protocol versions can define new validation rules
- This allows protocol evolution without breaking existing SegWit nodes

This pattern of leaving "upgrade hooks" appears throughout SegWit's design.

## SegWit Script Types

SegWit introduced two new standard script types, both maintaining backward compatibility:

### Pay-to-Witness-Public-Key-Hash (P2WPKH)

This is SegWit's version of P2PKH (Pay to Public Key Hash).

**Output Script (Witness Program):**
```
OP_0 <20-byte-pubkey-hash>
```

**Witness Data:**
```
<signature> <public-key>
```

When a SegWit node validates this:
1. Recognizes OP_0 as witness version 0
2. Next 20 bytes specify the public key hash
3. Retrieves witness data from the witness section
4. Validates the signature using the public key
5. Verifies the public key hashes to the specified value

### Pay-to-Witness-Script-Hash (P2WSH)

This is SegWit's version of P2SH (Pay to Script Hash).

**Output Script:**
```
OP_0 <32-byte-script-hash>
```

**Witness Data:**
```
<items> <items> ... <serialized-script>
```

The validation process:
1. Recognizes witness version 0
2. Next 32 bytes specify the script hash
3. Last item in witness is the actual script
4. Verifies the script hashes to the specified value
5. Executes the script with remaining witness items as initial stack

### Why Legacy Nodes See Anyone-Can-Spend

Here's something subtle but important. When legacy nodes see these scripts, they interpret them as:

```
OP_0 <data>
```

Since OP_0 pushes zero (false) to the stack, and the next element pushes some data, the script ends with data on the stack—which legacy nodes interpret as true (any non-empty data). To legacy nodes, these look like anyone-can-spend outputs!

This seems dangerous, but it works because:
1. Miners running SegWit nodes won't mine invalid transactions
2. Economic majority runs SegWit nodes, making invalid blocks unprofitable
3. Legacy nodes that accept invalid spends will diverge from the real chain

This is why SegWit required careful deployment with strong community consensus.

## Block Size and Weight Units

SegWit also addressed the contentious block size debate that dominated Bitcoin discussions from 2014-2017. The debate centered on whether to increase the 1MB block size limit through a hard fork.

SegWit provided an alternative: increase capacity without a hard fork by changing how block size is calculated.

### The New Calculation

SegWit introduced **weight units** rather than bytes:

**Block Weight = (Base Size × 4) + Witness Size**

Where:
- **Base Size**: Transaction data excluding witness (the black parts)
- **Witness Size**: Witness data only (the blue parts)
- **Maximum Block Weight**: 4,000,000 weight units

This effectively allows blocks larger than 1MB while maintaining backward compatibility. Legacy nodes still see blocks that appear to be under 1MB (they don't see witness data), while SegWit nodes can process blocks up to approximately 4MB depending on the ratio of witness to non-witness data.

This represented a compromise: capacity increase through a soft fork rather than a hard fork, preserving network unity at the cost of added complexity.

## Taproot: The Next Evolution

In November 2021, Bitcoin activated Taproot (technically SegWit version 1), bringing two major improvements: Schnorr signatures and Merkle-ized scripts.

### Schnorr Signatures

Taproot replaces ECDSA with Schnorr signatures. While both are elliptic curve signature schemes, Schnorr offers several advantages:

**1. Linearity**: Schnorr signatures have mathematical properties that enable advanced protocols:
   - Multiple signatures can be aggregated into a single signature
   - Key aggregation protocols (MuSig, FROST)
   - Adapter signatures for atomic swaps and Lightning payments
   - Threshold signatures with better privacy and efficiency

**2. Provable Security**: Schnorr has stronger security proofs than ECDSA

**3. Standardization**: Enables protocol improvements not possible with ECDSA

These cryptographic tricks enable numerous interesting protocols we'll explore in later chapters, particularly for multi-signature arrangements and layer-two protocols.

### Merkle-ized Alternative Script Trees (MAST)

Taproot's second major innovation solves a fundamental limitation: script size restrictions. Previously, scripts needed to fit entirely within block size limits. Taproot allows effectively unlimited script complexity through a clever structure.

**The Basic Idea:**

Instead of revealing the entire script, construct a Merkle tree of possible spending conditions. When spending, reveal only:
1. The specific condition you're using
2. A Merkle proof that this condition is part of the committed tree

This provides several benefits:

**1. Arbitrarily Large Scripts**: The full script tree can be enormous—megabytes, gigabytes, doesn't matter. Only the executed branch must fit in a block.

**2. Dramatic Privacy Improvement**: Different parties might have different spending conditions, but only the condition actually used gets revealed. Other possible conditions remain hidden.

**3. Fungibility**: When using the key-path spend (the simplest case), all Taproot outputs look identical on-chain.

### Taproot Output Structure

A Taproot output looks like:

```
OP_1 <32-byte-public-key>
```

Note OP_1 instead of OP_0—this signals Taproot (witness version 1).

The 32-byte public key is actually a **tweaked** key that commits to a Merkle root of possible scripts. This enables two spending paths:

**Key Path Spend:**
- Provide a signature for the tweaked public key
- Looks like a simple single-signature spend
- Provides maximum privacy
- Most efficient (lowest fees)

**Script Path Spend:**
- Reveal one script from the tree
- Provide a Merkle proof (the **control block**)
- Provide witness data satisfying that script
- Only this script is visible on-chain

### The Control Block

When doing a script path spend, the control block contains:
- The internal public key (before tweaking)
- The Merkle proof for the executed script
- Leaf version information

This allows validators to:
1. Verify the script was committed to the output
2. Verify the internal key was correctly tweaked
3. Validate the script execution

### Privacy Benefits

Consider a multi-party contract where:
- Alice has spending condition A
- Bob has spending condition B
- Carol has spending condition C
- All three together can spend with condition D

With Taproot:
- If Alice spends, only condition A is revealed
- Bob never sees conditions B, C, or D
- Carol never sees A, B, or D
- Each party only knows their own condition and what was revealed on-chain

This provides unprecedented privacy for complex contracts. Parties can share coins under different conditions without revealing all conditions to each other or to the public.

## Layered Protocols: Building on Bitcoin

With these primitives established, we can understand how layer-two protocols work. This mirrors how internet protocols layer on top of each other.

### The Protocol Stack Analogy

Consider the Internet protocol stack:

**Ethernet** → **IP** → **TCP** → **Application (HTTP, etc.)**

Each layer:
- Solves a specific problem
- Provides services to the layer above
- Operates independently from layers above and below

For example:
- **Ethernet**: Physical transmission of bits
- **IP**: Addressing and routing
- **TCP**: Reliable, ordered delivery
- **Application**: Specific service logic

Bitcoin as a protocol occupies the application layer of the Internet stack. But Bitcoin itself can serve as a base layer for higher-level protocols.

### Bitcoin as a Base Layer

Bitcoin transactions are the "packets" of the Bitcoin protocol. To build layer-two protocols:

1. Embed additional protocol data in transaction scripts
2. Other clients understand and process this embedded data
3. These clients enforce additional rules beyond base Bitcoin rules

The base layer (Bitcoin) continues transmitting transactions while layer-two protocols run "on top," using Bitcoin's security and settlement guarantees.

**Key Insight**: Layer-two protocols don't change Bitcoin's base protocol. They add interpretations and conventions on top of standard Bitcoin transactions.

### Data Embedding Locations

Where can layer-two protocols embed data?
- In lock scripts (the contracts)
- In unlock scripts (the spending data)
- In witness data (for SegWit/Taproot)
- In special fields like Taproot's annex

### Coordination Requirement

For these protocols to work, there must be coordination between the payer and payee. When someone pays you, you must communicate the proper script structure. They need to know what contract to create so you can later spend it.

This coordination happens off-chain through protocol-specific mechanisms, but the final settlement occurs through standard Bitcoin transactions that anyone can verify.

## The State of Bitcoin Smart Contracts

There's sometimes a claim that "Bitcoin doesn't have smart contracts" or that Bitcoin's contracts aren't "smart contracts." This is, at best, misleading.

Bitcoin's contracts have a more restricted execution context than other blockchains like Ethereum, Solana, or Polygon. Bitcoin scripts don't have access to:
- The complete transaction context
- The UTXO set
- Other transactions in the blockchain
- The full blockchain state

Other blockchain platforms provide scripts with much richer execution contexts, enabling different types of applications that are difficult or impossible in Bitcoin's model.

However, Bitcoin's scripting capabilities—especially with SegWit and Taproot—are remarkably powerful for specific use cases:
- Multi-signature arrangements
- Time-locked contracts
- Hash-locked contracts
- Complex conditional spending
- Privacy-preserving contracts
- Layer-two protocol foundations

The restrictions in Bitcoin's model aren't merely limitations—they're features that provide:
- Greater security through reduced attack surface
- Better scalability through simpler validation
- Stronger guarantees through limited side effects
- Foundation for layer-two protocols that can achieve computational completeness

## Conclusion: Continuous Evolution

The progression from basic Bitcoin to SegWit to Taproot demonstrates how Bitcoin evolves while maintaining backward compatibility. Each upgrade:

1. Solves specific problems (malleability, capacity, privacy)
2. Enables new capabilities (Schnorr tricks, large scripts)
3. Includes mechanisms for future upgrades (version flags, witness versions)
4. Maintains compatibility with older nodes

This conservative, methodical approach to protocol evolution—where each change must maintain network unity and avoid forcing upgrades—creates challenges but also ensures Bitcoin's stability and security.

The system becomes more capable without losing its fundamental properties: decentralization, permissionlessness, and security. As we'll see in subsequent chapters, these base-layer improvements enable increasingly sophisticated layer-two protocols and privacy-preserving techniques that expand Bitcoin's capabilities far beyond simple payments.

The journey from OP_RETURN to Taproot shows us that Bitcoin's programmability is not fixed—it's an evolving canvas where each improvement opens new possibilities while preserving the core principles that make Bitcoin unique.