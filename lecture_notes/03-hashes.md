Chapter 3: Hash Functions and Digital Commitments
The Magic of One-Way Functions
Let's start with something that seems almost magical when you first encounter it: the cryptographic hash function. Picture a mathematical meat grinder that takes any input - whether it's a single letter, the complete works of Shakespeare, or every movie ever made - and produces an output that's always exactly the same size. But here's where it gets interesting: even though the process is completely deterministic (the same input always produces the same output), it's computationally impossible to reverse. You can't look at the ground meat and figure out which cuts went into the grinder.
A hash function H takes an input of arbitrary size - what we call the preimage - and produces an output of fixed size called the hash or digest. The term "digest" is particularly apt; just as your digestive system breaks down complex foods into simpler components, a hash function "digests" data into a fixed-size summary. For our discussion, we'll primarily work with SHA-256 (Secure Hash Algorithm, 256-bit version), which produces outputs of 256 bits.
To grasp how large this space is, consider that 2^256 is approximately 10^77. The physicist's best estimate for the number of atoms in the observable universe is around 10^80. So we're talking about a number space so vast that if you assigned a unique hash to every atom in the universe, you'd barely make a dent in the available possibilities. This astronomical size is not an accident - it's carefully chosen to make certain attacks computationally infeasible even with quantum computers on the horizon.
A Tale of Two Types of Cryptography
Before diving deeper, it's worth understanding that cryptographic hash functions occupy an interesting position in the cryptographic landscape. Cryptography broadly divides into two camps. The first camp is based on hard mathematical problems - things like factoring large numbers or computing discrete logarithms. We have mathematical proofs about why these problems are difficult, even if we can't prove they're impossible to solve efficiently.
The second camp, where hash functions live, is more heuristic. We don't have elegant mathematical proofs that hash functions are secure. Instead, we have functions that have survived decades of attacks by the world's best cryptanalysts. SHA-256, for instance, was designed by the NSA and published in 2001. Despite enormous incentives to break it (imagine being able to forge Bitcoin transactions!), it remains secure. This is engineering more than pure mathematics - we know it works because we've tested it extensively, not because we have a mathematical proof.
The Three Pillars of Cryptographic Hash Functions
Not every hash function is suitable for cryptographic use. Your programming language's built-in hash table function, for instance, is designed for speed, not security. To earn the "cryptographic" designation, a hash function needs three critical properties, each protecting against different attack scenarios.
1. Preimage Resistance: The One-Way Street
Given a hash value y, it should be computationally infeasible to find any input x such that H(x) = y.
This is what makes the function "one-way." Think of it like dropping a glass - easy to do, impossible to undo. The only known way to find a preimage is brute force: try inputs until you find one that works. With SHA-256, you'd need to try, on average, 2^255 inputs. Even if you could check a trillion candidates per second on every computer on Earth, the sun would burn out long before you succeeded.
This property enables hiding. You can show the hash of a secret without revealing the secret itself. It's like showing someone a locked safe - they can see the safe exists, but not what's inside.
2. Second Preimage Resistance: No Duplicates Allowed
Given an input-output pair (x₀, y₀) where H(x₀) = y₀, you cannot find a different input x' that produces the same hash.
This is subtly but importantly different from preimage resistance. Here, we're not just showing the hash - we're showing one valid input as well, and challenging someone to find another. You might think this would make the problem easier, but it doesn't help much. An attacker still needs to search through approximately 2^256 candidates on average.
This property is crucial for using hashes as identifiers. If you could find two different messages with the same hash, you could potentially substitute one for the other in a system that identifies messages by their hashes.
3. Collision Resistance: The Birthday Paradox Protection
You cannot efficiently find any two different inputs x and z such that H(x) = H(z).
Now we're giving the attacker maximum freedom: choose any two inputs you want, just make them hash to the same value. This is actually easier than the previous attacks, but not by as much as you might think. Thanks to what cryptographers call the "birthday attack" (named after the birthday paradox - in a room of just 23 people, there's a 50% chance two share a birthday), you "only" need to try about 2^128 different inputs on average.
Here's why: As you generate hashes for random inputs, you're not looking for a specific target - you're looking for any collision. Each new hash you generate could potentially collide with any of the previous ones you've computed. So if you've already computed n hashes, your next attempt has n chances to create a collision. This accumulation of opportunities is what reduces the search space from 2^256 to 2^128.
Even so, 2^128 operations is still completely impractical. This gives us what we call "128 bits of security" - meaning the best known attack requires 2^128 operations. This has become the gold standard for cryptographic security, expected to remain unbreakable for decades even accounting for advances in computing power.
The Avalanche Effect: Chaos Theory in Action
Let me demonstrate one of the most important properties of cryptographic hash functions: the avalanche effect. When we change "Edil" to "Edin" - flipping just a single bit in the ASCII representation - the resulting SHA-256 hash changes in 124 out of 256 bit positions. This isn't coincidence; it's by design.
bash$ echo -n "Edil" | sha256sum | xxd -r -p | xxd -b
# [binary output with specific pattern]

$ echo -n "Edin" | sha256sum | xxd -r -p | xxd -b  
# [binary output with ~50% of bits different]

# Comparing the two: 124 bits different out of 256
The avalanche effect ensures that similar inputs produce completely unrelated outputs. Without it, you could potentially learn something about the input by looking at the output. Imagine if changing one letter in a document only changed a few bits in the hash - you could potentially figure out what changed by comparing hashes. The avalanche effect prevents this kind of analysis.
This property comes from the internal structure of hash functions like SHA-256, which use operations like XOR, bit rotation, and modular addition in complex feedback loops. Each bit of input affects multiple bits in the first round of processing, which affect even more bits in the second round, and so on, until the effect has cascaded (like an avalanche) through the entire output.
Three Ways to Use Hashes
1. As Names: The Universal Identifier
In Bitcoin, every transaction and every block has a unique identifier - its hash. Looking at a block explorer like mempool.space, those long hexadecimal strings identifying each transaction are SHA-256 hashes of the transaction data. But why use hashes instead of simple counters (transaction #1, #2, etc.)?
Several reasons make hashes superior:

Decentralization: No need for a central authority to assign ID numbers
Verification: The ID itself proves the content hasn't been tampered with
Universality: Anyone can independently compute the same ID for the same data
Uniqueness: Collision resistance ensures different transactions won't share IDs

Bitcoin actually had two transactions with the same ID early in its history. This wasn't due to a hash collision (which would break SHA-256), but rather a bug where the exact same transaction data was included twice. This edge case has since been fixed.
This pattern extends beyond blockchain. The Nix package manager I use on my Mac identifies every version of every package with a hash. Look at the Nix store:
bash$ ls /nix/store | grep coreutils
3v4r5msnvdk-coreutils-9.5
k9h15n2xaax-coreutils-9.6  
m2q3p44xmkz-coreutils-9.7
Each hash uniquely identifies not just the version but the exact compilation flags, dependencies, and configuration. Change anything, and you get a different hash - and thus a different package identity.
2. As Pointers: Building the Chain in Blockchain
Each Bitcoin block contains the hash of the previous block in its header. This creates what computer scientists call a linked list, but with a crucial property: the links can't be forged or changed. If you try to modify a historical block, its hash changes. This breaks the link from the next block, whose hash would also need to change, and so on. You'd need to rebuild the entire chain from that point forward - which, in Bitcoin's case, requires massive computational work due to proof-of-work.
This is why we call it a "blockchain" - it's a chain of blocks linked by cryptographic hashes. The hash pointers create an immutable historical record. Even if an attacker controlled significant computing power, they couldn't silently rewrite history; any changes would be immediately apparent to anyone verifying the chain.
3. As Commitments: Proving Knowledge Without Revealing It
This is perhaps the most subtle and powerful use of hash functions. A hash commitment lets you prove you knew something at a specific time without revealing what that something was until later.
Let me give a practical example. Say I want to bet on tonight's Flamengo vs. Fluminense match. I write down the score I predict, compute its hash, and give you the hash before the match starts. After the match, I reveal my prediction, and you can verify it matches the hash I gave you earlier. The hash proves I knew my prediction before the match, but didn't reveal what that prediction was until after.
Hash commitments give us two critical properties:
Hiding: The commitment reveals nothing about the value. Thanks to preimage resistance, seeing H(x) tells you nothing about x.
Binding: Once committed, you can't change your mind. Thanks to second preimage resistance, you can't find a different x' that produces the same hash.
In 1996, IBM researchers used this technique to prove they hadn't cheated in designing the S-boxes for the DES encryption algorithm. They published hashes of their design criteria years before revealing the criteria themselves, proving they hadn't retrofitted their explanations.
The Dictionary Attack Problem
The betting game reveals a crucial limitation of simple hash commitments. If you're betting on a die roll (1-6), there are only six possible values you might have committed to. An adversary can pre-compute all six hashes:
H("1") = 6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b
H("2") = d4735e3a265e16eee03f59718b9b5d03019c07d8b6c51f90da3a666eec13ab35
H("3") = 4e07408562bedb8b60ce05c1decfe3ad16b72230967de01f640b7e4729b49fce
H("4") = 4b227777d4dd1fc61c6f884f48641d02b4d121d3fd328cb08b5531fcacdabf8a
H("5") = ef2d127de37b942baad06145e54b0c619a1f22327b2ebbcfbec78f5564afe39d
H("6") = e7f6c011776e8db7cd330b54174fd76f7d0216b612387a5ffcfb81e6f0919683
When you send your commitment, they immediately know your bet. This is called a dictionary attack, and it's devastatingly effective when the input space is small.
The solution is to add randomness - a salt or nonce (number used once):
H(bet || nonce)
For example:
H("3" || "a7f3d2b9c8e5f1a4d6b8e2c9f5a1d7b3e9c5f2a8d4b1e7c3f9a5d2b8e4c1f6")
Now, even though there are only six possible bets, there are 2^256 possible nonces. An attacker can't pre-compute a dictionary for all possible combinations. When you reveal both your bet and your nonce after the roll, anyone can verify your commitment, but they couldn't have predicted your bet beforehand.
This same principle protects password databases. Websites don't (or shouldn't) store your password directly - they store its salted hash. Even if the database leaks, attackers can't easily reverse the hashes to get passwords, especially if each password has a unique salt.
The Challenge of Generating Good Randomness
The question "how is the nonce generated?" is worth a billion dollars - literally. Generating truly random numbers is one of the hardest problems in practical cryptography. Your computer is a deterministic machine; it doesn't naturally produce randomness.
Modern systems gather entropy from various sources:

Hardware randomness: Thermal noise in resistors, timing variations in oscillators
User input: Mouse movements, keyboard timings
System events: Disk access patterns, network packet arrivals

This raw entropy feeds into a cryptographically secure pseudorandom number generator (CSPRNG). But bugs in random number generation have caused catastrophic failures. Poor randomness in Android's Java SecureRandom led to Bitcoin wallet thefts. Debian's OpenSSL once had a bug that reduced the entropy to just 15 bits, making all keys generated during that period easily crackable.
For our hash commitments, the nonce MUST be unpredictable. If an attacker can guess your nonce, they can compute your commitment themselves and defeat the entire scheme.
Digital Signatures: The Cryptographic Identity
Digital signatures are the cryptographic equivalent of handwritten signatures, but far more powerful. Unlike a physical signature that's the same on every document (making forgery by copying possible), a digital signature is unique to both the signer AND the specific message being signed.
The three core algorithms are:
1. GenerateKeys() → (secret_key, public_key)
The notation varies across the literature. You'll see:

e for the secret key (historically "encryption" key, though we're signing not encrypting)
P for the public key (often capital to distinguish from the prime p we'll use later)
Or more clearly: sk and pk for secret/public key

2. Sign(secret_key, message) → signature
The signature depends on both inputs. Change the message by even one bit, and you need a completely different signature. This is fundamentally different from the physical world, where you can't sign a blank check that someone fills in later.
3. Verify(message, signature, public_key) → {true, false}
Verification confirms three things:

The signature was created using the private key corresponding to this public key
The signature is for this exact message
The message hasn't been altered since signing

This gives us two crucial security properties:
Non-repudiation: If a signature verifies with your public key, you can't deny making it (assuming your private key hasn't been compromised). This is actually stronger than physical signatures - it's mathematically provable that only the private key holder could have created this signature.
Authenticity: Any change to the message, even a single bit, invalidates the signature. This protects against tampering in a way physical signatures on paper documents cannot.
Lamport Signatures: Building Signatures from Hashes Alone
Leslie Lamport invented this scheme in 1979, demonstrating that one-way functions alone are sufficient for digital signatures. While impractical for real-world use, Lamport signatures beautifully illustrate how complex cryptographic protocols can emerge from simple primitives.
The Construction
Key Generation:
We need to create two sets of 256 random numbers, each number being 256 bits:

One set represents 0-bits (let's visualize them in black)
One set represents 1-bits (let's visualize them in red)

Private Key:
  For bit 0: [x₀,₀, x₁,₀, x₂,₀, ..., x₂₅₅,₀]  (256 numbers)
  For bit 1: [x₀,₁, x₁,₁, x₂,₁, ..., x₂₅₅,₁]  (256 numbers)
  Total: 512 × 256 bits = 16 KB
The public key consists of hashes of all these numbers:
Public Key:
  For bit 0: [H(x₀,₀), H(x₁,₀), ..., H(x₂₅₅,₀)]
  For bit 1: [H(x₀,₁), H(x₁,₁), ..., H(x₂₅₅,₁)]
  Total: Also 16 KB
Signing:
To sign a message M:

Compute h = H(M), giving us 256 bits
For each bit position i in h:

If bit i = 0, include x_{i,0} in the signature
If bit i = 1, include x_{i,1} in the signature



The signature reveals exactly one number from each position, chosen based on the message hash.
Verification:
Given message M, signature S, and public key P:

Compute h = H(M)
For each bit position i:

Extract the i-th number from the signature
Hash it
If bit i of h is 0, check it matches H(x_{i,0}) in P
If bit i of h is 1, check it matches H(x_{i,1}) in P


Accept only if all 256 checks pass

The Fatal Flaw: Information Leakage
Each signature reveals exactly half of your private key - one number for each of the 256 positions. But which half depends on the message hash, which appears random. Let's trace through what an attacker learns:
After one signature:

Attacker knows 256 of your 512 secret numbers
Coverage: 50% of private key

After two signatures:

The second message likely has different hash bits in ~128 positions
Attacker learns ~128 new secret numbers
Coverage: ~75% of private key

After three signatures:

Coverage: ~87.5% of private key

The real attack doesn't require knowing the entire private key. Once an attacker knows enough pieces, they can forge signatures by finding messages whose hashes only use the known positions.
For example, if an attacker knows 400 of your 512 numbers, they can search for a message whose hash happens to only require those 400 positions. This is a partial collision - they don't need the full hash to match something specific, just certain bits to fall in certain positions. With modern computing power, finding such partial collisions becomes feasible once enough of the private key is exposed.
Historical Context and Modern Relevance
Lamport signatures might seem like a historical curiosity, but they're experiencing a renaissance due to the quantum threat. Most practical signature schemes rely on problems that quantum computers can solve efficiently:

RSA relies on factoring large numbers
ECDSA (used in Bitcoin) relies on the elliptic curve discrete logarithm problem
Both fall to Shor's algorithm on a quantum computer

But hash functions? Grover's algorithm only gives quantum computers a square root speedup against hash functions. SHA-256's security degrades from 256 bits to 128 bits - still utterly impractical to break.
This has led to modern constructions:

XMSS (eXtended Merkle Signature Scheme): Uses hash trees to get many signatures from one key
SPHINCS+: A stateless hash-based signature scheme selected by NIST for post-quantum standardization
LMS (Leighton-Micali Signatures): Another hash-based scheme in the NIST standards

These build on Lamport's ideas with clever optimizations to make hash-based signatures practical for real-world use.
The Broader Pattern
Throughout these constructions, we see how simple primitives compose into complex systems. A hash function - just a deterministic scrambler - enables:

Unique identifiers that can't be forged
Immutable chains of data
Binding commitments that hide information
Digital signatures (with some limitations)

Each primitive provides specific guarantees. By combining them carefully, we build systems with emergent properties. The blockchain itself exemplifies this: hash functions, digital signatures, and consensus rules combine to create a distributed ledger that no single party controls.
This composability is why cryptography is so powerful. We don't need to solve every problem from scratch. We can build on proven primitives, combining them in new ways to solve new problems. Understanding these building blocks - really understanding them, not just memorizing their properties - lets you see why certain constructions work and others fail.
In our next chapter, we'll explore practical signature schemes based on elliptic curve cryptography. We'll see how different mathematical foundations lead to different tradeoffs, why Bitcoin chose ECDSA, and what happens when supposedly random nonces aren't random at all.
