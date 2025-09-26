# Chapter 8: Consensus Rules and Bitcoin's Monetary Policy

## Introduction to Consensus

In the previous chapter, we explored Bitcoin transactions in detail. Now we turn our attention to **consensus** - the set of rules that determine what is valid and what is invalid in the Bitcoin system, both for transactions and for blocks.

Consensus rules are the foundation that allows a distributed network of nodes to agree on the state of the system without a central authority. These rules must be followed by all participants, and any deviation results in automatic rejection by the network.

While there's ongoing debate about which component of Bitcoin is most important, transactions serve as the fundamental building blocks that all other rules exist to support. Every consensus rule we'll examine exists to ensure that Bitcoin transactions can function properly and securely.

## Block Structure

A Bitcoin block consists of exactly two components:

1. **Header**: Exactly 80 bytes containing metadata
2. **Transaction List**: A variable-length list of transactions

The transaction list is simply a concatenation of transactions, one after another. Each transaction has a well-defined structure that allows parsers to determine where one transaction ends and the next begins. A counter at the beginning of the list indicates how many transactions are included.

Every transaction in a block must be individually valid according to the transaction validation rules we discussed previously. However, the first transaction in every block is special.

## The Coinbase Transaction

The first transaction in every block is called the **coinbase transaction**. This transaction follows different rules from all other transactions in the system:

### Key Properties of Coinbase Transactions

1. **No Spending**: The coinbase transaction doesn't spend any existing money. Instead of referencing a previous UTXO, its input contains:
   - Transaction ID: 32 bytes of zeros (0x00000...)
   - Index: 0xFFFFFFFF (all F's in hexadecimal)
   - Script signature: Arbitrary data (with some restrictions)

2. **Money Creation**: The coinbase transaction creates new bitcoins from nothing. This is the only mechanism in Bitcoin that can increase the total money supply.

3. **Maturation Period**: Outputs from coinbase transactions must wait 100 blocks before they can be spent. This prevents complications that could arise if a block is later invalidated through reorganization.

### The Script Signature Field

In the coinbase transaction's input, the script signature field can contain arbitrary data. This is where Satoshi placed the famous message in the genesis block: "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks."

Today, this field has some required elements, but miners still have space to include additional data of their choice.

## Bitcoin's Monetary Policy

### Subsidy Mechanism

The coinbase transaction can create new money according to a predetermined schedule called the **subsidy**. The total amount that can be created in each block equals:

**Total Reward = Subsidy + Transaction Fees**

The subsidy follows a specific mathematical rule:
- Initial subsidy: 50 bitcoins per block
- **Halving**: Every 210,000 blocks (approximately 4 years), the subsidy is cut in half
- This continues until the subsidy becomes negligible

### The Halving Schedule

Since Bitcoin's launch in 2009, the halving events have occurred as follows:

- **2009-2012**: 50 BTC per block
- **2012-2016**: 25 BTC per block (first halving)
- **2016-2020**: 12.5 BTC per block (second halving)
- **2020-2024**: 6.25 BTC per block (third halving)
- **2024-present**: 3.125 BTC per block (fourth halving)

The process continues with 32 total halvings until approximately the year 2140, when new bitcoin creation will cease entirely.

### Transaction Fees

Miners can also collect transaction fees from all transactions included in their block. These fees represent the difference between the total value of inputs and outputs in each transaction. 

Importantly, miners are not required to collect these fees - any uncollected fees are simply lost forever. However, economic incentives ensure that miners collect all available fees to maximize their revenue.

### Total Supply

This halving mechanism creates a predictable monetary policy with several key characteristics:

- **Total cap**: Approximately 21 million bitcoins will ever be created
- **Distribution curve**: The emission rate decreases exponentially over time
- **Current status**: As of 2024, over 19.9 million bitcoins have been mined (about 95% of the total supply)
- **Remaining supply**: Less than 5% of bitcoins remain to be mined over the next 100+ years

## Consensus Enforcement

### Network Validation

Every node in the Bitcoin network independently validates every transaction and block. When a node receives new data, it checks that data against all consensus rules. If the data violates any rule, the node rejects it entirely.

This creates a powerful enforcement mechanism: if you violate consensus rules, you automatically disconnect yourself from the network. Your data will be rejected by other nodes, and your version of the ledger will diverge from the rest of the network.

### Example: Invalid Subsidy

Consider what happens if a miner tries to claim more than the allowed subsidy:

1. Miner creates a block claiming 50 BTC when only 3.125 BTC is allowed
2. Miner propagates this block to other nodes
3. Each receiving node validates the block and detects the invalid subsidy
4. All nodes reject the block as invalid
5. The miner has spent computational resources but receives no reward
6. The miner's version of the blockchain diverges from the network

This creates strong economic incentives to follow the rules correctly.

## Network Synchronization

### Node Communication

When nodes connect to the Bitcoin network, they engage in a handshaking process where they exchange information about their current state. If one node discovers that another has seen more blocks, it will request the missing blocks and validate them one by one.

### Initial Block Download (IBD)

A new node joining the network must download and validate the entire blockchain history:

1. **Download**: Retrieve all blocks from January 2009 to present (currently 900,000+ blocks)
2. **Validation**: Verify every transaction in every block
3. **State Construction**: Build the current UTXO set from this history

This process can take several days on typical hardware, with most time spent verifying cryptographic signatures rather than downloading data.

### Handling Network Partitions

Bitcoin's design gracefully handles temporary network disconnections:

1. Node goes offline and misses several blocks
2. Upon reconnection, node requests missing blocks from peers
3. Node validates each missed block in sequence
4. Node updates its state to match the rest of the network

This mechanism ensures that all honest nodes eventually converge on the same view of the blockchain, regardless of temporary connectivity issues.

## Immutability Through Incentives

The combination of computational cost and economic incentives creates Bitcoin's security model. Miners must invest real-world resources (electricity, hardware, facilities) to participate in the system. When they violate consensus rules, they forfeit these investments while gaining nothing.

This economic reality makes the consensus rules remarkably stable. Even though anyone can modify their local copy of the Bitcoin software, doing so in a way that violates consensus rules simply disconnects them from the valuable network.

The beauty of this system lies in its voluntary nature: participation is optional, but following the rules is necessary for participation to have any value. This creates a self-reinforcing system where economic incentives align individual behavior with collective security.

---

*This chapter introduced the fundamental concepts of consensus in Bitcoin, focusing on how new money is created and how the network maintains agreement on valid transactions and blocks. In the next chapter, we'll explore the proof-of-work mechanism that determines which miner gets to create each block and collect these rewards.*
