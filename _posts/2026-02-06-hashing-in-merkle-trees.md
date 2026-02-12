---
layout: post
title: Hashing in Merkle Trees
description: "Review of different hash functions: history, properties, constructions, performance."
image: /images/Rust_Hash_Benches.png
---

# Hashing in Merkle Trees
<span style="color:lightgray">{{ page.date | date: "%B, %Y" }}</span>

A hash function compresses the input into a fixed-length string. It might be used for compression itself, but might also be used to remove structure from the user's input for security purposes (e.g. when signing a message).

A typical blockchain would use a hash-function in the following common cases:
* for constructing Merkle trees and Merkle proofs,
* to hash a transaction being signed,
* for proof-of-work,
* for public keys to generate shorter addresses.

Because hash function compress data, collisions are inevitable, but are hard to find.
In particular, based on the application we require different security properties of a hash-function:
* **Pre-image resistance**: given h it is hard to find x, s.t. H(x) = h.
* **Second preimage resistance**: given x, it is hard to find x', s.t. H(x) = H(x').
* **Collision resistance**<span style="color:blue">\*</span>: hard to find x1, x2, s.t. H(x1) = H(x2).

<span style="color:blue">\*</span><span style="color:lightgray">Due to birthday paradox a hash function that outputs t bits is at most t/2-bits collision-resistant.</span>

Note that collision resistanace is the strongest property of all.
Collision resistance implies second pre-image resistance implies pre-image resistance (for sufficiently long inputs which we will consider here).

And ocasionally additionally required (esp. when hashing secrets):
* **Length-extension resistance**: given H(x) it is difficult to find a pair: ext and H(x &#124;&#124; ext), i.e. it is difficult to extend the input of the hash knowing only the output. This usually comes up as a problem when a secret is hashed, in which case an [HMAC](https://datatracker.ietf.org/doc/html/rfc2104) gives this property for the otherwise length-extension susceptible function.

Although for completeness, it should be noted that some exotic properties are sometimes considered for keyed hash functions, e.g.
target collision resistance ([TCR](https://eprint.iacr.org/1997/009)), enhanced target collision resistance ([eTCR](https://link.springer.com/chapter/10.1007/11818175_3)), etc.

**History of NIST standards for hash functions:** The Secure Hash Algorithm (SHA) was developed by NSA and published by NIST in 1993 as FIPS-180, it was called SHA-0. SHA-0 outputs 160-bits digests, hence was only 80-bits collision resistant at best. In 1995, NSA added an extra instruction to the SHA-0's compression function, NIST updated the standard accordingly without giving any rational behind the tweak. The resulting function was called SHA-1. Ten years later it was found that this extra instruction was crucial for collision resistance. SHA-1 became the de-facto standard for collision resistant hashing. As SHA-1 started seeing more and more attacks, in 2001 SHA-2 developed by NSA was added to the standard, now becoming FIPS 180-2, it is three to four times slower than SHA-1. In 2011 NIST recommended to move from SHA-1 to SHA-2 as attacks discovered in 2005 in SHA-1 were becoming more practical. In February 2017 a collision was found as two pdf documents, so for crypto-applications SHA-1 is mostly abandoned, although it is still believed to be a one-way function (i.e. to satisfy the pre-image resistance property). SHA-1 and SHA-2 were not created by publicly sourced contests. They were created by the NSA and then released as public-use patents. But through a public contest announced in 2017, SHA-3 was picked as a finalist and standardised in 2012 as FIPS-202.

Current NIST standards for hashing (see [NIST-FIPS-202](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf)):

![Security Strength of SHA-1, SHA-2 and SHA-3](/images/SHA_strengths_FIPS_202.png){: style="max-width:500px; width:100%; height:auto;" }

## SHA-2

SHA-2 ([FIPS 180-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf)) is often called just "SHA": standardised in 2001, mostly used SHA2-256, SHA2-512. SHA2-512/256 which is roughly the same as SHA2-512 truncated to 256 bits is sometimes used in place of SHA2-256 as it is more [speedy](eprint.iacr.org/2010/548) on 64-bits architectures and also resistant against length-extension attacks. Historically SHA-256 and SHA-512 stand for SHA2-256 and SHA2-512 respectively.

SHA-2 uses the Merkle-Damgard paradigm with Davies-Meyer's compression function, for a block-cipher E it works as follows:

![Merkle Damgard Construction](/images/Merkle_Damgard_construction.png){: style="max-width:650px; width:100%; height:auto;" }

The block cipher E used for SHA-2 is NIST-designed SHACAL-2 cipher running the round function on a 64B block with different pre-specified round keys: 64 rounds for SHA2-256 and 80 rounds for SHA2-512.
[Attack on SHA2-512/256 (2016)](https://eprint.iacr.org/2016/374.pdf) found collision for 43 rounds (out of 80). [Attack on SHA2-256 (2011)](https://eprint.iacr.org/2011/037.pdf) found collision for 46 rounds (out of 64). No attack are known on full SHA2 other than the trivial birthday-bound.

**SHA2 Native Instruction**: SHA2-256 has a native instruction in many modern processors making it about 10x faster! Even SHA2-512 is harware-supported on modern ARM processors.

Note that AES can't easily be used out-of-the-box for E, since AES has 128-size blocks thus giving only 64-bits security for collision resistance. AES is also too slow for short-messages due to key-switching overhead.

**Block size and padding.** SHA2-256 has 64B-size blocks. Padding for SHA2 is 10\*L, where L is a 64 bits long length (in bits) of the original message, so SHA2-256 the padding always adds at least 65 bits (~9B) to the message.
When hashing a 64B-input, the conventional padding will overflow the input making it two blocks instead of one, which for example is a 2x overhead for internal Merkle tree nodes.

Bitcoin is using SHA2-256 for the proof-of-work challenges. All of Bitcoin miners are currently computing **2^95** hashes per year. Bitcoin also constructs a Merkle tree for a block with transactions as leaves, using double SHA2-256, the root of the tree is put into a block's header.

## SHA-3

SHA-3 ([FIPS 202](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf)) was standardized in 2015. Most popular are SHA3-256, SHA3-512 modes.

While there is nothing wrong with SHA-2 (as of today), SHA-3 got standardised to diversify the crypto-toolkit as new attacks were being discovered for SHA-1 which shares similar designs with SHA-2. So, in the event one of the functions get broken, we have another one to substitute with (though it has [its own controversies](https://www.imperialviolet.org/2017/05/31/skipsha3.html)).

![SHA3 Sponge Construction](/images/SHA3_SpongeConstruction.png){: style="max-width:650px; width:100%; height:auto;" }
<br/><span style="color:lightgray;margin:0;padding:0;"><small>From "A Graduate Course in Applied Cryptography" [version 0.6](https://toc.cryptobook.us/book.pdf) by Dan Boneh and Victor Shoup</small></span>

SHA-3 is a sponge construction. Where the function f is a public premutation called Keccak that maps 1600-bits to 1600-bits. For SHA3-256: c = 512 bits, for SHA3-512: c = 1024 bits. For SHAKE128: c = 256 bits, for SHAKE256: c = 512 bits. The output is r first bits of the last execution of f. The c last bits are not part of the output, giving length-extension resistance in contrast to SHA-2. f includes 24 rounds of iterations: [an attack on SHA-3 (2019)](https://eprint.iacr.org/2019/147.pdf) found a collision on 5 rounds (out of 24); no attacks are known on full SHA3 other than the trivial birthday bound.

**Padding and block-size.** Padding for SHA3 is 01 &#124;&#124; 10\*1, for SHAKE is 1111 &#124;&#124; 10\*1. SHA3 blocks are larger than SHA2 (for SHA3-256 blocks are 136B).

[Ethereum is using](https://i.sstatic.net/afWDt.jpg) Keccak-256 (which is not SHA3, though quite the same): Ethereum launched just before FIPS 202 was published (August 2015) but after the Keccak was announced as a winner, so Ethereum's hash function has some minor differences from SHA3. It's used in different Merkle trees, including for accounts' states. It had been used for proof-of-work mechanism, which since retired in favor of proof-of-stake.

## Other hash functions

* **[BLAKE](https://www.aumasson.jp/blake/)** - designed in 2008 was one of five NIST finalists for SHA-3, designed by J.P.Aumasson, L.Henzen, W.Meier, R.C.W.Phan. It relies on a core permutation built from [ChaCha stream cipher](https://cr.yp.to/chacha/chacha-20080128.pdf).
The initial BLAKE submission (2008) had 14 and 10 rounds for two variants, the later increase to 16 and 14 (2010) was apparently motivated by already high speed of BLAKE.

* **[BLAKE2](https://www.blake2.net/blake2.pdf)**  ([RFC7693](https://datatracker.ietf.org/doc/html/rfc7693#ref-BLAKE2)) - designed in 2013 by J.P.Aumasson, S.Neves, Zooko Wilcox-O’Hearn, C.Winnerlein - is in an improved-performance version of BLAKE with a reduced number of rounds (12 and 10), the team says: "based on the security analysis performed so far, and on reasonable assumptions on future progress, it is unlikely that 16 and 14 rounds are meaningfully more secure than 12 and 10 rounds." BLAKE2 is designed to be efficient on a single CPU core (BLAKE2b with 12 rounds is more efficient on 64-bit CPUs, produces digests of any size between 1 and 64 bytes; and BLAKE2s with 10 rounds is more efficient on 32-bit CPUs, produces digests of any size between 1 and 32 bytes). BLAKE2bp and BLAKE2sp are designed to be efficient on multicore or SIMD chips. BLAKE2 is faster than SHA-3 and SHA-2.

* **[BLAKE3](https://github.com/BLAKE3-team/BLAKE3-specs/blob/ea51a3ac997288bf690ee82ac9cfc8b3e0e60f2a/blake3.pdf)** - designed in 2019 by J.O’Connor, J.P.Aumasson, S.Neves, Zooko Wilcox-O’Hearn is an attempt to unify all the different variants of BLAKE2 into a single hash-function that performs well on all architectures. It has even smaller number of rounds (7 instead of 10 in BLAKE2s). BLAKE3 splits its input into 1024 bytes chunks and arranges them as the leaves of a binary tree, each chunk is compressed in parallel. Has 128-bit security level and 256-bit default output size.

* **[ParallelHash](https://csrc.nist.gov/pubs/sp/800/185/final)** - published by NIST in 2016 as designed for hashing long messages by taking advantage of the parallelism in the processors. First the input is split into blocks, each block is hashed in parallel, then a single pass of cSHAKE is done over the concatenated results.

* **[KangarooTwelve](https://eprint.iacr.org/2016/770.pdf)** ([RFC9861](https://www.rfc-editor.org/rfc/rfc9861.html)) - published in 2016, shares similar design to SHA-3 (with same round function), but a reduced number of rounds, from 24 to 12, thus it is twice as fast.

* **[Haraka v2](https://eprint.iacr.org/2016/098.pdf)** - very fast hash-function, relatively new (2016), based on AES-NI. One of the possible hash instantiations used optionally in [SPHINCS+ Round 3](https://sphincs.org/data/sphincs+-r3.1-specification.pdf). It drops collision-resistance in favor or pre-image resistance and second- preimage-resistance as is required for hash-based signatures and HMACs. However, collision resistance is required for accumulators (e.g. Merkle trees).

* **[ChaCha](https://cr.yp.to/chacha/chacha-20080120.pdf)** ([RFC8439](https://www.rfc-editor.org/rfc/rfc8439)) - as suggested in [SPHINCS original paper (2014)](https://eprint.iacr.org/2014/795.pdf) can be used in place of a public permutation in sponge, giving the following 512-to-256 hash function suitable for Merkle trees. Assuming
  * E is a ChaCha permutation taking 512-bits inputs to 512-bits outputs
  * C is a public constant string,
  * TRUNC truncates to the first 256 bits, then 
  * <code>H(M1 &#124;&#124; M2) = TRUNC<sub>256</sub>(E(M1 &#124;&#124; C) &oplus; (M2 || 0<sup>256</sup>))</code> is a 512-to-256 compression function.

  It is recommended to do 12 rounds of ChaCha, 20 rounds is also a common instantiation; [the best attack](https://eprint.iacr.org/2025/437.pdf) is now for 7 rounds of ChaCha. This construction needs more vetting.

## Hashing in Merkle Trees
Hashing in a tree-fasion is done to either parallelize hashing, or to make it easier to do small changes to the large data being hashed, or for Merkle-tree accumulators which serves as cryptographic commitments with efficiently computable proofs of inclusion.

Under some plausible assumptions, [it was shown](https://eprint.iacr.org/2024/1095.pdf) that to get (n*256)-to-256 collision-resistant function from 512-to-256 collision-resistant function, n-1 calls are required, so Merkle trees and Merkle-Damgard construction (e.g. SHA-2) are in some sense optimal.

![Sequentian vs. Parallel Hashing](/images/Parallel_vs_Sequential_Hashing.png){: style="max-width:650px; width:100%; height:auto;" }

The trees in accumulators can get rather big, for example:

* **Certificate Trasparency Logs** ([RFC9162](https://www.rfc-editor.org/rfc/rfc9162.html)) are "append-only" and publicly-auditable ledgers of certificates being created, updated, and expired. There are a number of them run by Google, Cloudflare, Sectigo, DigiCert and others (see [stats](https://radar.cloudflare.com/certificate-transparency#certificate-transparency-logs)). Some of the longest for example have 93M, 77M, 76M entries. That gives binary trees of about <u>27 levels high</u>.

* **Ethereum** currently has [around 363M addresses](https://etherscan.io/chart/address), but the account trees are not binary, they are 16-ary Merkle Patricia Tries - the place of the account in a trie is determined by the 256-bits hash of 160-bits account's address, and tree branches are not instantiated all the way to the bottom of the trie, but kept compressed. Because of the large ary-ness of the tree, the Merkle proofs are large than for binary trees, but since the height of the tree is low, there are less nodes to update on leaf changes saving disk I/Os and CPU-time.

* **BitTorrent** is using [binary Merkle trees](https://bittorrent.org/beps/bep_0052_torrent_creator.py): the file by default is split into 16 KB blocks that are first hashed with SHA2-256 and put in a Merkle tree. For a 10GB file, the tree would be about <u>20 levels high</u>.

## Performance

We measure performance of various hash-functions with 256-bits outputs with short-message inputs.
Not suprisingly, the graphs jump in steps with the block-length.
The red line shows the performance for 512-to-256 compression for constructions suitable for Merkle hashing. Those functions that drop the padding/domain-separation to have fewer number of blocks to hash would require careful in-protocol handling at the benefit of better performance.

Here are my benchmarks (or rather attempts) for Rust crates on Apple M3 Pro (2023). Not sure why SHA2 in software is slower than SHA3 in software, possibly because Rust compiler favors safety over performance.

![Rust hashing benchmarks](/images/Rust_Hash_Benches.png){: style="max-width:600px; width:100%; height:auto;" }

For performance in Merkle trees, we look at a specific point of these graphs for hashing 64 bytes input. Note the drastic 6x differenece depending on the choice of the function:

<table class="bordered-table">
    <tr style="border: 1px solid #cccccc; border-collapse: collapse;">
      <td markdown="span" style="color:#1E90FF">**258 ns**</td>
      <td markdown="span"><span style="color:#1E90FF">**SHA2**</span></td>
    </tr>
    <tr>
      <td markdown="span" style="color:#FFA500">**294 ns**</td>
      <td markdown="span"><span style="color:#FFA500">**SHA3**</span></td>
    </tr>
    <tr>
      <td markdown="span" style="color:#FF0000">**289 ns**</td>
      <td markdown="span"><span style="color:#FF0000">**SHA3-ASM**</span></td>
    </tr>
    <tr>
      <td markdown="span" style="color:#1E90FF">**137 ns**</td>
      <td markdown="span"><span style="color:#1E90FF">**SHA2 no padding**</span></td>
    </tr>
    <tr>
      <td markdown="span" style="color:#FFD700">**122 ns**</td>
      <td markdown="span"><span style="color:#FFD700">**BLAKE2**</span></td>
    </tr>
    <tr>
      <td markdown="span" style="color:#9ACD32">**90 ns**</td>
      <td markdown="span"><span style="color:#9ACD32">**BLAKE3**</span></td>
    </tr>
    <tr>
      <td markdown="span" style="color:#0000CD">**61 ns**</td>
      <td markdown="span"><span style="color:#0000CD">**SHA2-NI**</span></td>
    </tr>
    <tr>
      <td markdown="span" style="color:#0000CD">**42 ns**</td>
      <td markdown="span"><span style="color:#0000CD">**SHA2-NI no padding**</span></td>
    </tr>
</table>

<!-- SHA2:  257.6760835
SHA3:  294.0
SHA3-ASM:  289.375
SHA2 no padding:  136.607708
BLAKE2:  122.264854
BLAKE3:  89.5889795
SHA2-NI:  60.997104
SHA2-NI no padding:  42.165167 -->

Some applications might want to introduce security boundary between storage and execution; and check merkle proofs on every read from the database. Performance of the hash-function will then become even more important.