# MAS.S62 Fall 2025
# Cryptocurrency Design and Engineering

### NOTE:  This document is a draft and is subject to change.

## Information

Instructors:  [Tadge Dryja](tdryja@gmail.com) and [Neha Narula](narula@media.mit.edu) (MIT), [Edil Medeiros](edil@vinteum.org) (UnB)

Time: T/Th 11 AM - 12:30 PM

Place: E15-359

Discord: TBD

You are welcome to ask us questions via email.  However, if you think your
question would be useful for others to see, please file it as [an issue](https://github.com/mit-dci/cde-2025/issues)
in this repository!

Contact: [cryptocurrency-f25-staff@mit.edu](cryptocurrency-f25-staff@mit.edu)


Description:

Bitcoin and other cryptographic currencies have been rapidly gaining adoption. This course explores the design of Bitcoin and other blockchains and how they function in practice, focusing on cryptography, game theory, security, networking, and system architecture. In Fall 2025, this course will be co-taught with the University of Brasilia, connecting students through projects and discussions. Our goal is to get a more global view of the potential impact and use cases of the technology. Programming assignments in the course will give practical experience interacting with cryptocurrency software. Prior programming experience is required, and exposure to computer science systems or security courses is recommended.

Office hours: TBD

Office hours location: TBD

## Schedule

NOTE: The schedule is in flux and is subject to change.

Content covers essential technologies behind Bitcoin and other cryptocurrencies. Answering questions such as:
- **Foundations** (lectures 1-8): What problems do cryptocurrencies aim to solve, and what primitives enable decentralized digital money?
- **Protocol Design and Tradeoffs** (lectures 9-14): How do cryptocurrency architectures reflect their goals, and what are the key design choices and constraints?
- **Scaling, Privacy, and Future Directions** (lectures 15-23): What are the current limitations of cryptocurrency systems, and what emerging designs seek to address them?

| Date | Lecture | Main Concept | Guiding Questions | Topics Covered | Readings |
|------|---------|--------------|-------------------|----------------|----------|
| 2025-09-04 | 1 | Nature of money | What is money? Why might we want a new kind of money? | Money as a thing: stones, shells, salt, gold. Money as a set of functions: medium of exchange, store of value, unit of account. Money as a liability (IOU): fiat money and intermediaries.| [Aristotle](https://classics.mit.edu/Aristotle/politics.1.one.html) - Politics, Book I, Part IX |
| 2025-09-09 | 2 | Trust Minimization — Network Design  | How can a decentralized network implement money and payments without a central authority? | Requirements for decentralized money: scarcity, transferability, verifiability. Bitcoin architecture overview (ledger + consensus + incentives). Other paradigms: early e-cash attempts, other cryptocurrencies.| [Bitcoin: A Peer-to-Peer Electronic Cash System](https://bitcoin.org/en/bitcoin-paper), [Untraceable Electronic Cash](https://www.wisdom.weizmann.ac.il/~/naor/PAPERS/untrace.pdf) |
| 2025-09-11 | 3 | Cryptographic Commitments and Proofs | How can data be securely committed to without revealing it, and why is this critical for digital money? | Cryptographic hash functions. Properties: preimage resistance, collision resistance. Commitments and their use in contracts, blockchains. Merkle trees and proofs.| [Hash Functions](https://youtu.be/tLkHk__-M6Q?si=BC0bRDMqAF63PWEu) by Christof Paar |
| 2025-09-16 | 4 | Digital signatures and public-key cryptography | How can ownership and control of digital assets be proven without revealing private information? | Public-key cryptography basics. Digital signatures (ECDSA, Schnorr). Verification vs secrecy. Role of keys in cryptocurrency systems. Introduction to finite field DH | [New Directions in Cryptography](https://www.cs.jhu.edu/~rubin/courses/sp03/papers/diffie.hellman.pdf), [Simple Schnorr Multi-Signatures with Applications in Bitcoin](https://eprint.iacr.org/2018/068.pdf) |
| 2025-09-18 | 5 | State Models | What design choices exist for tracking digital assets?| UTXO model. Account model. Privacy, scalability, parallel validation tradeoffs | [Bitcoin Whitepaper](https://bitcoin.org/en/bitcoin-paper), [Transactions](https://en.bitcoin.it/wiki/Transaction), [Ethereum Whitepaper](https://ethereum.org/en/whitepaper/) |
| 2025-09-23 | 6 | Consensus | How can a decentralized system reach global consensus about ownership? | The double-spend problem. Proof-of-Work: mechanism and incentives. Proof-of-Stake alternatives: concepts and tradeoffs. | [Bitcoin Whitepaper](https://bitcoin.org/en/bitcoin-paper), [What Proof of Stake Is And Why it Matters](https://bitcoinmagazine.com/culture/what-proof-of-stake-is-and-why-it-matters-1377531463),[Permissionless Consensus](https://arxiv.org/pdf/2304.14701) |
| 2025-09-25 | 7 | Synchronization and Verification | How can new participants join and synchronize efficiently and securely? How can lightweight participants verify the system’s integrity without storing the full history? | Bootstrapping a node. Initial block download (IBD). P2P discovery and network topology. Design tradeoffs in node connectivity. Full node vs light client (SPV) security models. Bloom filters, compact block filters (Neutrino). Trust assumptions and attack surfaces. | [Bitcoin's Security Model: A Deep Dive](https://www.coindesk.com/markets/2016/11/13/bitcoins-security-model-a-deep-dive) |
| 2025-09-30 | 8 | Wallets and Malleability | What do we need to commit to? How do you know you've been paid vs how to construct a transaction & pay someone else? | Malleability. Pre-SegWit vs SegWit structure. Witness data and soft fork design. | [Inherent Malleability of ECDSA Signatures](https://www.derpturkey.com/inherent-malleability-of-ecdsa-signatures/) |
| 2025-10-02 | 9 | Money Programmability | How can a digital asset system implement programmable spending conditions? | Locking/unlocking scripts, standard opcodes. Example contracts (multisig, timelocks). | [Bitcoin Script: Past and Future](https://btctranscripts.com/misc/2020-04-08-john-newbery-contracts-in-bitcoin) |
| 2025-10-07 | 10 | Computability and Smart Contracts | What can decentralized ledgers compute, and what are the limits of their programmability? |Bitcoin Script vs EVM model. Stateless computation with UTXOs. Gas models and resource metering. Turing completeness vs bounded language design. | [Ethereum Whitepaper](https://ethereum.org/en/whitepaper/) |
| 2025-10-09 | 11 | Market Incentives | How are fees set and why do they matter? |Block size and transaction competition. Mempool mechanics and fee estimation. Fee incentives and markets in Bitcoin and Ethereum. Explore mining attacks, selfish mining, and undercutting | [Majority is Not Enough: Bitcoin Mining is Vunerable](https://arxiv.org/abs/1311.0243) |
| 2025-10-14 | 12 | Usability | How can wallets balance user privacy, key management, and usability? | Payment flow (acquiring addresses, online requirements, privacy impact, etc). Key derivation (BIP32, BIP39). Hierarchical deterministic wallets. Address types: legacy, SegWit, Taproot. Silent payments and receiver privacy.| [BIP32 Hierarchical Deterministic Wallets](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) |
| 2025-10-16 | 13 | Governance |What happens when participants in a decentralized network disagree, and how can protocol upgrades be managed? | Soft forks vs hard forks. Governance models: Bitcoin vs Ethereum vs others. The DAO hack, Bitcoin Cash, and Segwit as case studies. Risks of chain splits. | [The Ether Thief](https://www.bloomberg.com/features/2017-the-ether-thief/), Jonathan Bier, The Blocksize War: The battle over who controls Bitcoin’s protocol rules |
| 2025-10-21 | 14 | Client-side Validation | How can applications be built on top of a secure decentralized ledger without requiring global consensus for every action? | Client-side validation concept. Examples: Taproot Assets, RGB protocol, NFTs. Global vs local state debate. Off-chain assets and auditability.| [The State of Atomic Swaps](https://diyhpl.us/wiki/transcripts/scalingbitcoin/tokyo-2018/atomic-swaps/) |
| 2025-10-23 | 15 | Scalability | What options exist for scaling a decentralized currency system without sacrificing security or decentralization? | On-chain scaling (block size debates). Layer 2 approaches: Lightning Network, payment channels. Sidechains and federated models (Liquid, RSK). | [Bitcoin Lightning Network: Scalable off-chain instant payments](https://lightning.network/lightning-network-paper.pdf) |
| 2025-10-28 | 16 | Scalability continued| What options exist for scaling a decentralized currency system without sacrificing security or decentralization? |Explore Ark, channel factories, coinpools, statechains, multisignatures, aggregation. Understand quick syncing things like assumeutxo / swiftsync / utreexo / assumeutreexo | [Cross-Input Signature Aggregation for Bitcoin](https://cisaresearch.org/) |
| 2025-10-30 | 17 | Smart Contracts continued | How can contracts work off-chain? | Discreet Log Contracts (DLCs). Oracle models and trust assumptions. Settlement on-chain vs purely off-chain execution. Tie in with client side validation, cross chain swaps | [Discreet Log Contracts](https://adiabat.github.io/dlc.pdf) |
| 2025-11-04  |  | No Class | |  | |
| 2025-11-06  |  | No Class | |  | |
| 2025-11-11  |  | MIT Holiday | |  | |
| 2025-11-13 | 18 | Privacy | What are the design approaches for improving privacy in cryptocurrency transactions, and what are their tradeoffs? | De-anonymization heuristics. CoinJoin, PayJoin. Confidential Transactions. Stealth payments and scanning wallets. | [Confidential Transactions](https://web.archive.org/web/20150628230410/https://people.xiph.org/~greg/confidential_values.txt) |
| 2025-11-18 | 19 | Privacy — Zero Knowledge Proofs | What are zero-knowledge proofs and how are they used in privacy-preserving systems? | Understanding privacy-preserving designs that use zero-knowledge proofs, such as Bulletproofs, Mimblewimble, and Zcash | [The Ceremony](https://radiolab.org/podcast/ceremony), [Mimblewimble](https://scalingbitcoin.org/papers/mimblewimble.pdf), [Zerocash](https://ieeexplore.ieee.org/document/6956581) |
| 2025-11-20 | 20 | State Management | Can we design a secure system that reduces or eliminates the need for storing a full set of unspent outputs? | Accumulator proofs and Utreexo concept. Stateless client designs. Security and performance considerations. | [Utreexo](https://eprint.iacr.org/2019/611.pdf) |
| 2025-11-25 | 21 | Programmability — Expressivity | What are the tradeoffs between limiting and expanding the programmability of money in a decentralized system? | Spectrum of expressivity: Bitcoin Script → Taproot/MAST → Ethereum’s EVM. What are covenants? Use cases for expressive money: contracts, escrow, vaults. Risks of expressivity:  loss of fungibility, security. Stablecoins | [Bitcoin Covenants Wiki](https://covenants.info) |
| 2025-11-27 | | MIT Holiday | |   | |
| 2025-12-02 | 22  | Security — Post-Quantum  | How can a cryptocurrency remain secure against the potential future threat of quantum computers? |Understanding quantum risk to ECDSA/ECDH. Post-quantum signature proposals. Migration challenges and backward compatibility. Explore commit/reveal schemes. | [Quantum Computing for the Very Curious](https://quantum.country/qcvc) |
| 2025-12-04 | 23 | Unfair Behaviors  | How do design choices affect the potential for participants to extract unfair advantages? |Miner Extractable Value (MEV) concept. Flashbots and Ethereum MEV research. Bitcoin’s design defenses vs theoretical attacks. Ethical and systemic risks of MEV.  | [Flash Boys 2.0: Frontrunning, Transaction Reordering, and Consensus Instability in Decentralized Exchanges](https://arxiv.org/abs/1904.05234)|
| 2025-12-09 | 24 | Final Presentations  | |   | |


## Labs and Problem Sets

| # | Due Date | Assignment | 
|---|------|------------|
| 1 | 2025-09-25 | One way functions, digital signatures, and proof-of-work. |
| 2 | 2025-10-07 | Mining |
| 3 | 2025-10-16 | Schnorr signatures and privacy |
| 4 | 2025-10-28 | UTXOHunt |

All labs are due by 11:59 PM on the day specified.

## Final projects

You may form groups of 1-4 students and prepare a presentation and a ~4 page paper on one of the following: 

1. Create a new application using the concepts learned in class (see provided list for suggestions)
2. Update or add a new feature to an existing system like Bitcoin, Ethereum, or another cryptocurrency or shared ledger implementation
3. Propose a new formalization or framework
4. Pose and solve a new question or problem

Project proposals are due 2025-12-04 29:59 EDT.
