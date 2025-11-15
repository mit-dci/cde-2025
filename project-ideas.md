# Project ideas for MAS.S62

# High-level:

* Design and implement an application or system
* Add a new feature to an existing system like Bitcoin, Ethereum, or another cryptocurrency 
* Propose a formalization in this space for a topic that has not been formalized yet
* Analyze or systematize a new area
* Pose and solve an interesting problem

# More specific ideas, collected from many sources:

* Applications:
  * Build an application that receives ecash (https://cashu.space/) as payment (e.g. for consuming data in a web API).
  * Implement a game receiving Lightning payments (e.g. https://www.youtube.com/watch?v=gpzCvff3kPg)
  * Create a wallet that does something new or interesting
* Analysis
  * Analyze Bitcoin covenant proposals (refs: https://covenants.info/, https://utxos.org/uses/, https://bitcoin.softforks.org/). Explain the pros and cons of different proposals
  * Analyze and/or implement a coin selection algorithm (ref: https://arxiv.org/pdf/2311.01113)
  * Analyze and/or implement a fee estimation algorithm
  * Analyze a real CVE of Bitcoin Core (https://bitcoincore.org/en/security-advisories/). Even better if you build an exploit (can use Warnet for simulation. Should be something other than what we did in the last lab)
  * Analyze what’s CISA and its implications (reference: https://cisaresearch.org/)
  * Analyze the BitVM bridges and what they are useful for
  * Comparison and analysis of ETH L2s
  * Comparison and analysis of zk-rollups
* Monitoring and Simulation
  * Create monitoring tools for electrum indexers. E.g https://b10c.me/projects/024-peer-observer/, for electrum. A more concrete output could be an analysis of the electrum p2p network. Is the electrum p2p network just a honeypot for chainanalysis? https://github.com/0xB10C/project-ideas/issues/11
  * Simulate how easy it is to propagate non-standard transactions with filtering nodes.
* Implement a new opcode (OP_MUL?) in Bitcoin Core, btcd, or https://github.com/BitVM/rust-bitcoin-scriptexec 
* (Challenging) Add a fuzzing target (reference: https://github.com/bitcoinfuzz/bitcoinfuzz). Explain fuzzing tests and their impact on the security of the network
* TEE secured silent payments scanning (https://silentpayments.xyz/docs/)
