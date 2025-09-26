Chapter 4: Elliptic Curve Digital Signatures
Beyond Hash-Based Signatures
We've seen how Lamport signatures elegantly demonstrate that one-way functions alone suffice for digital signatures. But their one-time nature makes them impractical for real-world use. Let's explore the two main paths forward from here.
The cryptographic world splits signature schemes into two broad categories. First, there are hash-based signatures like Lamport's scheme and its modern descendants. These have a remarkable property: they're believed to be quantum-resistant. Why? Because quantum computers don't give us any significant speedup for inverting hash functions - Grover's algorithm only provides a square root improvement, reducing SHA-256's security from 256 bits to 128 bits, which is still utterly impractical to break.
The downside? Hash-based signatures tend to be large. Even with clever optimizations, we're talking about signatures and keys measured in kilobytes rather than the compact hundreds of bytes we'd prefer. They also often retain that one-time or few-time use limitation, though schemes like XMSS work around this with merkle trees.
The second category relies on number theory - mathematical facts about numbers and their relationships. This is where most practical signature schemes live today, and they split into two subcategories:

RSA-based systems: Built on the difficulty of factoring large composite numbers
Elliptic curve systems: Built on the discrete logarithm problem in elliptic curve groups

I'm not going to cover RSA here, despite its enormous importance - basically the entire internet runs on RSA. Why skip it? Because I don't know of any cryptocurrency that uses RSA for anything. The keys are just too large compared to elliptic curves, even though they're smaller than hash-based signatures. RSA needs keys of 2048-4096 bits to be secure, while elliptic curves achieve the same security with just 256 bits.
There's a dark cloud over these number-theoretic schemes though: quantum computers. We already know quantum algorithms (specifically Shor's algorithm) that can efficiently solve both factoring and the elliptic curve discrete logarithm problem. Not instantaneously, but exponentially faster than classical computers. When will quantum computers pose a real threat? I'm personally skeptical it'll happen soon - I think we have decades - but the crypto community is divided on this timeline.
The Elliptic Curve Magic
Let me show you how we can build incredibly compact, efficient signatures using elliptic curves. What we'll construct gives us the same 128-bit security level as our hash functions, but with keys of just 256-512 bits.
What Is an Elliptic Curve?
For our purposes, an elliptic curve is the set of points (x, y) that satisfy the equation:
y² = x³ + ax + b
where a and b are constants we choose when defining our system. This is called the "short Weierstrass form" - there are other forms, but this is what you'll encounter in practice.
For Bitcoin specifically, we use a = 0 and b = 7, giving us the beautifully simple:
y² = x³ + 7
Looks almost trivial, doesn't it? Don't let the simplicity fool you - the security comes from the group structure we'll build on these points, not the equation itself.
This specific curve wasn't chosen randomly. It's standardized as secp256k1 (Standards for Efficient Cryptography, prime 256-bit, Koblitz curve 1). The "k" indicates it's a Koblitz curve, which has special properties that allow certain optimizations. Interestingly, secp256k1 is almost exclusively used in Bitcoin and its derivatives - most other applications use secp256r1.
Building a Group from Points
When we plot this curve over real numbers (just for visualization), it has a distinctive shape - a smooth curve that extends to infinity in both directions. But that's not what we'll actually use. The real magic happens when we define a group operation on these points.
A group needs:

A set of elements (our curve points)
A binary operation (we'll call it "addition")
Closure: adding two points gives another point in the group
An identity element: a point O where P + O = P for any point P
Inverses: for every point P, there's a -P where P + (-P) = O
Associativity: (P + Q) + R = P + (Q + R)

Here's the geometric construction for point addition:
Adding two different points P and Q:

Draw a line through P and Q
This line intersects the curve at a third point
Reflect this point across the x-axis - that's P + Q

Doubling a point P (computing P + P):

Draw the tangent line at P
Find where it intersects the curve again
Reflect across the x-axis

The identity element:
We introduce a special "point at infinity" O that acts as our zero. It's not on the curve itself but serves as the identity: P + O = P for any point P.
Inverses:
The inverse of a point P = (x, y) is simply -P = (x, -y) - its reflection across the x-axis.
This might seem arbitrary, but it works! The operation is associative (though proving this requires some work with the algebra). The geometric intuition helps, but in practice, we compute everything algebraically using formulas derived from this construction.
From Real Numbers to Finite Fields
Here's where things get interesting for cryptography. Instead of using real numbers for our coordinates, we use a finite field - specifically, integers modulo a large prime p.
A finite field Fp consists of the integers {0, 1, 2, ..., p-1} with addition and multiplication defined modulo p:

a + b in Fp means (a + b) mod p
a × b in Fp means (a × b) mod p

For example, in F₇ = {0, 1, 2, 3, 4, 5, 6}:

5 + 4 = 9 mod 7 = 2
3 × 5 = 15 mod 7 = 1

Think of it like clock arithmetic - when you go past p-1, you wrap around to 0.
Every non-zero element in Fp has a multiplicative inverse. For instance, in F₇:

2 × 4 = 8 mod 7 = 1, so 2⁻¹ = 4
3 × 5 = 15 mod 7 = 1, so 3⁻¹ = 5

For Bitcoin's secp256k1, we use a specific 256-bit prime:
p = 2²⁵⁶ - 2³² - 2⁹ - 2⁸ - 2⁷ - 2⁶ - 2⁴ - 1
This looks complicated, but it's cleverly chosen. It's very close to 2²⁵⁶, which means:

We get almost the full range of 256-bit numbers
The small difference (those subtracted powers of 2) fits in 32 bits
This makes modular reduction very efficient - no expensive division needed!

When we use finite field coordinates, our smooth curve becomes a scattered set of points that looks random. But all our point addition formulas still work! The group structure remains intact, just in this discrete setting.
The Discrete Logarithm Problem
Now for the cryptographic magic. We choose a base point G on our curve (standardized as part of secp256k1). Then:
Key Generation:

Choose a random 256-bit number e (your private key)
Compute P = e·G (your public key)

Here, e·G means "add G to itself e times" - this is called scalar multiplication.
The One-Way Function:
Computing P = e·G is "easy" (polynomial time using the double-and-add algorithm). But given P and G, finding e is believed to be hard - this is the elliptic curve discrete logarithm problem (ECDLP).
Why is scalar multiplication fast? We don't actually add G to itself e times sequentially. Instead, we use the binary representation of e:
e = b₀·2⁰ + b₁·2¹ + b₂·2² + ... + b₂₅₅·2²⁵⁵

e·G = b₀·(2⁰·G) + b₁·(2¹·G) + b₂·(2²·G) + ...
We can compute 2ⁱ·G by repeatedly doubling: 2G = G+G, 4G = 2G+2G, etc. This takes only 256 doublings and at most 256 additions - polynomial in the bit length of e.
But to find e given P? The best known classical algorithm is Pollard's rho, which takes roughly 2¹²⁸ operations for our 256-bit curve. That's our 128 bits of security.
The visual intuition is compelling: as you compute G, 2G, 3G, 4G, ..., the points jump around the curve in what appears to be a random walk. There's no discernible pattern that would let you guess where 30G lands without computing it. Given P = 30G, you can't tell it's the 30th multiple without trying different values of e until you find it.
ECDSA: The Imperfect Standard
The Elliptic Curve Digital Signature Algorithm (ECDSA) is what Bitcoin uses, despite being inferior to Schnorr signatures in almost every way. Why? Patents.
Schnorr signatures were invented in the late 1970s and patented. The patent didn't expire until 2008, by which time ECDSA had become the standard. It's uglier mathematically, harder to prove secure, and doesn't support many useful tricks - but it was patent-free, so everyone used it.
Here's how ECDSA works:
Key Generation:

Private key: random e from the field
Public key: P = e·G

Signing a message m:

Choose a random nonce k (CRITICAL: must be unique for each signature!)
Compute R = k·G
Let r = x-coordinate of R
Compute s = k⁻¹(H(m) + r·e)
Signature is (r, s)

Verification (given message m, signature (r, s), public key P):

Compute u = s⁻¹·H(m)
Compute v = s⁻¹·r
Compute R' = u·G + v·P
Accept if x-coordinate of R' equals r

The verification works because of some algebraic manipulation I won't detail here - it's not particularly enlightening. The important point is that only someone knowing the private key e could have produced a valid signature.
The Critical Importance of the Nonce
That random k in the signing process? It's absolutely critical that it's:

Unique for every signature
Unpredictable
Never revealed

If you reuse a nonce k for two different messages, an attacker can recover your private key. If the nonce is predictable (like using a weak random number generator), same problem. This has caused real Bitcoin losses - the Android SecureRandom bug led to wallet thefts because the random number generator wasn't random enough.
In practice, most implementations now use RFC 6979, which generates k deterministically from the message and private key, ensuring it's unique for each message while removing the need for a good random number generator during signing.
Looking Ahead: Schnorr Signatures
In our next discussion, we'll explore Schnorr signatures, which Bitcoin finally adopted in the Taproot upgrade. They're simpler, more elegant, provably secure under standard assumptions, and enable remarkable tricks:

Signature aggregation (combining multiple signatures into one)
Threshold signatures (k-of-n multisig that looks like single-sig)
Adaptor signatures (conditional payments)
Blind signatures (privacy protocols)

The patent that held back progress for 30 years has expired, and the cryptocurrency world is now discovering what we've been missing. Schnorr signatures aren't just better - they enable entirely new categories of protocols.
The Bigger Picture
What we've built here is remarkable: from the simple equation y² = x³ + 7 and some modular arithmetic, we've created a signature scheme with 128-bit security using just 256-bit keys. The signatures themselves are just two 256-bit numbers - incredibly compact.
This compactness matters enormously for cryptocurrencies. Every transaction needs signatures, and blockchain space is precious. The difference between a 512-byte hash-based signature and a 64-byte ECDSA signature multiplied across millions of transactions is huge.
But remember the quantum threat looming over all of this. These beautiful elliptic curve constructions that give us such elegant, compact signatures will crumble when large-scale quantum computers arrive. That's why research into post-quantum signatures continues, trying to find schemes that are both quantum-resistant and practical.
For now though, elliptic curves rule the cryptocurrency world. Every Bitcoin transaction, every Ethereum smart contract call, every DeFi trade - they all rely on the hardness of the elliptic curve discrete logarithm problem. It's a remarkable foundation built on surprisingly simple mathematics: just points on a curve, addition rules, and the apparent difficulty of reversing scalar multiplication.
