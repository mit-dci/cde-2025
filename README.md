# MAS.S62 Fall 2025
# Cryptocurrency Design and Engineering

### NOTE:  This document is a draft and is subject to change.

## Information

Instructors: Neha Narula <narula@media.mit.edu> (MIT), Edil Medeiros <edil@vinteum.org> (UnB)

Guest lecturers: [Ethan Heilman](https://www.ethanheilman.com/), [Joachim Neu](https://www.jneu.net/),  [Trey Del Bonis](https://tr3y.io/), [James Prestwich](https://x.com/_prestwich), [Madars Virza](https://madars.org/), [Pieter Wuille](http://pieterwuillefacts.com/), [Jason Milionis](https://jasonmili.github.io/), [Michael Mosier](https://www.arktouros.co/team/michael-mosier), and [Amanda Tuminelli](https://www.defieducationfund.org/about). 

Time: Tuesdays 1-4 PM

Place: E15-359

Discord: Please email us with your name and affiliation for the link

You are welcome to ask us questions via email.  However, if you think your
question would be useful for others to see, please file it as [an issue](https://github.com/mit-dci/cde-2025/issues)
in this repository!

Contact: <cryptocurrency-f25-staff@mit.edu>


Description:

Bitcoin and other cryptographic currencies have been rapidly gaining adoption. This course explores the design of Bitcoin and other blockchains and how they function in practice, focusing on cryptography, game theory, security, networking, and system architecture. In Fall 2025, this course will be co-taught with the University of Brasilia, connecting students through projects and discussions. Our goal is to get a more global view of the potential impact and use cases of the technology. Programming assignments in the course will give practical experience interacting with cryptocurrency software. Prior programming experience is required, and exposure to computer science systems or security courses is recommended.

Office hours: 2-3 PM ET Thursdays

Office hours location: Discord

If you'd like help in-person, you can join us at DCI (E15-350) Friday afternoons. Email the staff mailing list or ask on Discord to learn more.

## Schedule

NOTE: The schedule is in flux and is subject to change.

Content covers essential technologies behind Bitcoin and other cryptocurrencies. Answering questions such as:
- **Foundations**: What problems do cryptocurrencies aim to solve, and what primitives enable decentralized digital money?
- **Protocol Design and Tradeoffs**: How do cryptocurrency architectures reflect their goals, and what are the key design choices and constraints?
- **Scaling, Privacy, and Future Directions**: What are the current limitations of cryptocurrency systems, and what emerging designs seek to address them?

| Date | Topic | Main Concept | Guiding Questions | Topics Covered | Readings |
|------|---------|--------------|-------------------|----------------|----------|
| 2025-09-09 | 1 | Nature of money | What is money? Why might we want a new kind of money? | Money as a thing: stones, shells, salt, gold; set of functions: medium of exchange, store of value, unit of account; liability (IOU): fiat money and intermediaries. Digital payments. Ecash. | Optional: [What is Money?](https://cooperative-individualism.org/innes-a-mitchell_what-is-money-1913-may.pdf), [On the Origins of Money](https://cdn.mises.org/On%20the%20Origins%20of%20Money_5.pdf), [Aristotle](https://classics.mit.edu/Aristotle/politics.1.one.html) - Politics, Book I, Part IX, [Senate testimony: Investigating the Real Impacts of Debanking in America](https://www.banking.senate.gov/imo/media/doc/klein_testimony_2-5-25.pdf)  |
| 2025-09-09 | 2 | Trust Minimization — Network Design  | Guest Lecture: Ethan Heilman.  How can a decentralized network implement money and payments without a central authority? | Requirements for decentralized money: scarcity, transferability, verifiability. Bitcoin architecture overview (ledger + consensus + incentives). | [Untraceable Electronic Cash (1988)](https://web.archive.org/web/20110903023027/http://blog.koehntopp.de/uploads/chaum_fiat_naor_ecash.pdf), [Bitcoin: A Peer-to-Peer Electronic Cash System (2008)](https://bitcoin.org/en/bitcoin-paper)  |
| 2025-09-16 | 3 | Hash functions | Guest lecture: Ethan Heilman. How can data be securely committed to without revealing it, and why is this critical for digital money? | Cryptographic hash functions. Properties: preimage resistance, collision resistance. Commitments and their use in contracts, blockchains. | [Hash Functions](https://youtu.be/tLkHk__-M6Q?si=BC0bRDMqAF63PWEu) by Christof Paar, [The Library of Babel](https://sites.evergreen.edu/politicalshakespeares/wp-content/uploads/sites/226/2015/12/Borges-The-Library-of-Babel.pdf) by Jorge Luis Borges |
| 2025-09-16 | 4 | Cryptographic commitments and digital signatures  | Guest lecture: Ethan Heilman. How can ownership and control of digital assets be proven without revealing private information? | Merkle trees and proofs. Public-key cryptography basics. Digital signatures (ECDSA, Schnorr). Verification vs secrecy. Role of keys in cryptocurrency systems. Introduction to finite field DH | [New Directions in Cryptography](https://www.cs.jhu.edu/~rubin/courses/sp03/papers/diffie.hellman.pdf), [Simple Schnorr Multi-Signatures with Applications in Bitcoin](https://eprint.iacr.org/2018/068.pdf) |
| 2025-09-23 | 5 | Consensus | Guest lecture: Joachim Neu. How can a decentralized system reach global consensus about ownership? | The double-spend problem. Proof-of-Work and Proof-of-Stake: mechanism and incentives. History of the distributed consensus problem, what's new in blockchains, concepts and tradeoffs. | [Bitcoin Whitepaper](https://bitcoin.org/en/bitcoin-paper), [Security proof for Nakamoto consensus](https://decentralizedthoughts.github.io/2019-11-29-Analysis-Nakamoto/),[Permissionless Consensus](https://arxiv.org/pdf/2304.14701), [Simplex consensus protocol](https://simplex.blog/protocol/) |
| 2025-09-30 | 6 | State Models | What design choices exist for tracking digital assets? | UTXO model. Account model. Privacy, scalability, parallel validation tradeoffs | [Bitcoin Whitepaper](https://bitcoin.org/en/bitcoin-paper), [Transactions](https://en.bitcoin.it/wiki/Transaction), [Ethereum Whitepaper](https://ethereum.org/en/whitepaper/) |
| 2025-09-30 | 7&8 | Mining, forks, and governance | How does mining work in practice, and how do you upgrade the software? What happens when people disagree? | Mechanics of how forks work. Soft forks vs hard forks in practice. Governance models: Bitcoin vs Ethereum vs others. The DAO hack, Bitcoin Cash, and Segwit as case studies. Risks of chain splits.  | [The Ether Thief](https://www.bloomberg.com/features/2017-the-ether-thief/), Optional: Jonathan Bier, The Blocksize War: The battle over who controls Bitcoin’s protocol rules, [Majority is Not Enough: Bitcoin Mining is Vunerable](https://arxiv.org/abs/1311.0243) |
| 2025-10-07 | 9 | Synchronization and Verification | How can new participants join and synchronize efficiently and securely? How can lightweight participants verify the system’s integrity without storing the full history? | Bootstrapping a node. Initial block download (IBD). P2P discovery and network topology. Design tradeoffs in node connectivity. Full node vs light client (SPV) security models. Trust assumptions and attack surfaces. | [Bitcoin's Security Model: A Deep Dive](https://www.coindesk.com/markets/2016/11/13/bitcoins-security-model-a-deep-dive) |
| 2025-10-07 | 10 | Money Programmability | How can a digital asset system implement programmable spending conditions? | Bitcoin script and opcodes. Use cases for expressive money, specifically: contracts, escrow, vaults. Example contracts. | [Bitcoin Script: Past and Future](https://btctranscripts.com/misc/2020-04-08-john-newbery-contracts-in-bitcoin) |
| 2025-10-14 | 11 | Computability and Smart Contracts | What can Bitcoin compute, and what are the limits of its programmability? | Client-side validation. Off-chain vs. on-chain, oracle models and trust assumptions. Taproot, covenants. | [Bitcoin Covenants Wiki](https://covenants.info)  |
| 2025-10-21 | 12 | Scalability | How do we balance the tradeoffs for scaling a decentralized currency system with impacts to security, interactivity, and decentralization? | What makes scaling hard. Goals: reducing verification time, tradeoffs. On-chain scaling (block size debates). Layer 2 approaches: Lightning Network, payment channels. Sidechains and federated models. | [Bitcoin Lightning Network: Scalable off-chain instant payments](https://lightning.network/lightning-network-paper.pdf) |
| 2025-10-21 | 13 | Scalability continued| How do we balance the tradeoffs for scaling a decentralized currency system with impacts to security, interactivity, and decentralization?  | Explore Ark, channel factories, coinpools, statechains, aggregation, BitVM, and rollups.  | [Cross-Input Signature Aggregation for Bitcoin](https://cisaresearch.org/) (more to be added) |
| 2025-10-28 | 14&15 | Programmability in Ethereum | Guest lecture: James Prestwich. Details of the EVM and Ethereum's programming model | EVM theory, smart contract development, security and common pitfalls, decentralized applications, other programming models. | [Ethereum Whitepaper](https://ethereum.org/en/whitepaper/), [evm.codes](evm.codes), [Mastering Ethereum](https://github.com/ethereumbook/ethereumbook), [Solidity language documentation](https://docs.soliditylang.org) |
| 2025-11-04 | 16 | Privacy | Guest lecture: Madars Virza. What are the design approaches for improving privacy in cryptocurrency transactions, and what are their tradeoffs? | De-anonymization heuristics. CoinJoin, PayJoin. Confidential Transactions. Stealth payments and scanning wallets. | [Confidential Transactions](https://web.archive.org/web/20150628230410/https://people.xiph.org/~greg/confidential_values.txt) (more to be added) |
| 2025-11-04 | 17 | Privacy — Zero Knowledge Proofs | Guest lecture: Madars Virza. What are zero-knowledge proofs and how are they used in privacy-preserving systems? | Understanding privacy-preserving designs that use zero-knowledge proofs, such as Bulletproofs, Mimblewimble, Zcash, and Tornado Cash | [The Ceremony](https://radiolab.org/podcast/ceremony), [Mimblewimble](https://scalingbitcoin.org/papers/mimblewimble.pdf), [Zerocash](https://ieeexplore.ieee.org/document/6956581) |
| 2025-11-11 | | MIT Holiday | |   | |
| 2025-11-18 | 18  | Security — What can go wrong  | Guest lecture: Ethan Heilman. How can a cryptocurrency remain secure against breaks in the underlying cryptography? |Understanding quantum risk to ECDSA/ECDH. Post-quantum signature proposals. Migration challenges and backward compatibility. Explore commit/reveal schemes. | [Quantum Computing for the Very Curious](https://quantum.country/qcvc) (more to be added) |
| 2025-11-18 | 19 | Market Incentives | Guest lecture: Pieter Wuille. How are fees set and why do they matter? | Block size and transaction competition. Mempool mechanics, fee estimation, and how this connects to mining and validating. Securely operating an open access P2P network |  |
| 2025-11-25 | | No class | |   | |
| 2025-12-02 | 20 | Unfair Behaviors  | Guest lecture: Jason Milionis. How do design choices affect the potential for participants to extract unfair advantages? | Miner Extractable Value (MEV) concept. Flashbots and Ethereum MEV research. Bitcoin’s design defenses vs theoretical attacks. Ethical and systemic risks of MEV. | [Flash Boys 2.0: Frontrunning, Transaction Reordering, and Consensus Instability in Decentralized Exchanges](https://arxiv.org/abs/1904.05234)|
| 2025-12-02 | 21 | Laws and Regulation | What are the risks to developing open source software that might facilitate the movement of funds? What does it mean to be custodial vs. non-custodial? The importance of privacy. The BSA and its costs. | Lawyers and regulatory experts. Relevant cases.  | [Bernstein vs. US Department of Justice](https://www.eff.org/cases/bernstein-v-us-dept-justice), [US vs. Storm and Semenov](https://www.justice.gov/usao-sdny/file/1311391/dl?inline), [Peanut Butter & Watermelon: Financial Privacy in the Digital Age](https://www.sec.gov/newsroom/speeches-statements/peirce-remarks-blockchain-conference-080425), [A Cypherpunk's Manifesto](https://www.activism.net/cypherpunk/manifesto.html), [A Reckoning Looms for America’s 50-Year Financial Surveillance System](https://www.cato.org/cato-journal/spring/summer-2021/reckoning-looms-americas-50-year-financial-surveillance-system), [Anti-money laundering: The world's least effective policy experiment? Together, we can fix it](https://www.tandfonline.com/doi/full/10.1080/25741292.2020.1725366) |
| 2025-12-09 | 22 | Final Presentations  | |   | |


## Labs and Problem Sets

| # | Due Date | Assignment | 
|---|------|------------|
| 1 | 2025-09-30 | Hash functions |
| 2 | 2025-10-14 | Digital signatures |
| 3 | 2025-10-31 | Proof of work |
| 4 | 2025-11-14 | Warnet |

All labs are due by 11:59 PM on the day specified.

## Final projects

You may form groups of 1-4 students and prepare a presentation and a ~5 page paper on one of the following: 

1. Create a new application using the concepts learned in class
2. Update or add a new feature to an existing system like Bitcoin, Ethereum, or another cryptocurrency or decentralized system
3. Propose a new formalization or framework
4. Analyze or systematize a new area
5. Pose and solve a new question or problem

See project ideas [here](https://github.com/mit-dci/cde-2025/blob/main/project-ideas.md).

Due dates:
* Proposals (2 pages) are due 2025-11-21 23:59 EDT
* Presentations are in class 2025-12-09
* Papers are due 2025-12-09 15:59 EDT


[![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

The lecture slides are licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
