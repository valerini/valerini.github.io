---
layout: post_container
title: The State Of Post Quantum Cryptography
description: "Review of the state of post quantum cryptography."
image: /images/Classical_vs_Quantum.png
---

# The State Of Post Quantum Cryptography
<span style="color:lightgray">{{ page.date | date: "%B %-d, %Y" }}</span>

Peter Shor's quantum algorithm (1994) efficiently (in polynomial time) breaks RSA- and DH-based signatures and public-key encryption.
In a [harvest-now and decrypt-later](https://en.wikipedia.org/wiki/Harvest_now,_decrypt_later) attack,
encrypted data can be recorded today and decrypted once a sufficiently powerful quantum computer becomes available, so quantum-resistant encryption is needed today. A weak ciphertext once intercepted can be stored by an adversary forever. For signatures, it is only important that, when the signatures are being checked, a quantum computer is not available to forge them. Therefore, transitioning to quantum-resistant signatures is less urgent than transitioning to quantum-resistant encryption, since cryptographically relevant quantum computers still appear to be several years away. Blockchains present a more nuanced case, which I discuss below. Actually, the multi-billion-dollar bounty secured by blockchains is a testament to the fact that a cryptographically relevant quantum computer has not been built yet.

<div>
<img src="/images/Classical_vs_Quantum.png" style="width:600px"/>
</div>

## How far away are quantum computers?

To quote <a href="https://github.com/CardsAgainstCryptography/CAC">cards against cryptography</a>: "<i>quantum computers have been 10 years away for 3 decades</i>". That's ironically quite true.
The progress however is steady (see the [6th annual report](https://globalriskinstitute.org/publication/2024-quantum-threat-timeline-report/) of the Global Risk Institute). No roadblocks have been found yet. Hundreds of companies (400+) by now are focusing on quantum technologies, some have even become publicly tradeable. Tech giants are in the game: most notably, IBM with a [Nighthawk chip](https://newsroom.ibm.com/2025-11-12-ibm-delivers-new-quantum-processors,-software,-and-algorithm-breakthroughs-on-path-to-advantage-and-fault-tolerance) (from Nov 2025), Google with a [Willow chip](https://blog.google/technology/research/google-willow-quantum-chip/) (from Dec 2024). Both have qubits built from superconducting circuits. John M. Martinis who leads Google's quantum computing team received a Nobel Prize in Physics in 2025. There are other promising approaches for qubits: ion traps, cold atoms, photons, etc. Despite all the advances, it is hard to project the progress to any non-trivial timeline.

News announcements regularly highlight chips with thousands of qubits, yet those are very noisy physical qubits. A logical qubit is an abstraction obtained by applying error-correction to physical qubits. Google's most recent chip gives 1 logical qubit from 105 physical qubits.

The most [recent result](https://arxiv.org/abs/2602.11457) (Feb 2026, not peer-reviewed yet) requires on the order of 100,000 high-quality physical qubits to break RSA-2048. Whether such devices can be built at scale remains uncertain. In contrast, [work by Craig Gidney](https://arxiv.org/pdf/2505.15917) (June 2025), under less restrictive hardware assumptions, places the requirement at under one million physical qubits. Although existing quantum computers are not yet “cryptographically relevant,” the gap appears to be narrowing.

In 2001 a quantum computer factored 15, yet have not factored 21. Looks like that would require  [one hundred times larger](https://algassert.com/post/2500) circuit and that is a big step up. There is a good [talk from Seyoon Ragavan](https://www.youtube.com/watch?v=TVev-BYtPX8) the state-of-the-art in quantum factoring algorithms.

Quantum chips are bulky objects and won't be in personal devices anytime soon, but will be accessibly remotely.

## Quantum Model of Computation

<b>Classical computers</b> that take as input n bits, mathematically operates on an array of length n of 0s and 1s with 2-bit-input NAND gates. If the number of input bits is n, then the number of gates as a function of n determines the complexity of the algorithm. A different set of gates (e.g. {OR, AND} instead of {NAND}) will change the complexity by at most a constant.

<b>Quantum computers</b> that operate on n qubits, given n-bits of classical input, mathematically operate on unit-length complex vectors of length 2<sup>n</sup>. The initial value is  (0, 0, .., 0, 1, 0, ..., 0, 0), where 1 is at one of 2<sup>n</sup> positions given by the n-bits classical input to the algorithm. A single gate is a multiplication of our 2<sup>n</sup>-length vector by a unitary 2<sup>n</sup> x 2<sup>n</sup> complex matrix (unitary means that it preserves the unit-length of the vector). The matrix should come from a certain "universal" set of matrices. Here is an example of a "practical" set of universal gates: take one of these four matrices H (Hadamard, 2x2), S (Phase, 2x2), CNOT (4x4) or T (π/8 gate, 2x2, non-Clifford gate, hardest to realize) and lift it to a 2<sup>n</sup> x 2<sup>n</sup> matrix using tensor product with identity matrices on the right and left. Practically, it means that each gate physically operates on 1 or 2 qubits (for 2x2 and 4x4 matrices respectively) at a time. Finally, the resulting output is a "measurement" - it outputs a number from 1 to 2<sup>n</sup> (that's an n-bits number) with probability equal to the absolute value in that position of the vector, squared. The number of gates as a function of n determines the complexity of the algorithm. Measurement can always be deferred to the end. A different set of gates might have a polylogarithmic effect on complexity.

<b>Source of speed-up</b>: quantum algorithms operating on n physical qubits mathematically operate on exponentially long vectors (2<sup>n</sup>-length), although the output is still only n-bits, hence the tricky part is to translate from the vastness of quantum information to succinct classical in a useful way. It is not always possible (e.g. Grover's algorithm for brute-force search is optimal - but still has exponential complexity).

<b>For Peter Shor's algorithm</b>, there are two parts to it: first part is completely classical, it reduces the problem (of factoring or finding discrete log) to period finding; and then the second part is quantum, it finds the period. Quantum Fourier Transform can be done efficiently (in O(n<sup>2</sup>)), and gives period after measurement (<a href="https://lucatrevisan.github.io/teaching/cs259q-12/lecture08.pdf">it's a bit more subtle</a>).

<b>Finding short vectors in lattices</b> also feels like period finding, albeit you don't know which direction to look at. A lattice is periodic in any direction, however in most of them the period is large. Cryptosystems built from this problem are currently considered quantum-safe.

## Quantum Threat to Blockchains

Quantum computers slightly weaken hashes (although that is [debatable](https://www.youtube.com/watch?v=eB4po9Br1YY)) and completely break EC-signatures with the following implications and mitigations:

* <b>For consensus with hash-based proof-of-work</b>: quantum computers have quadratic speedup over bruteforce search (due to [Grover's search algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm)), so theoretically they will be searching for nonces faster than classical miners taking over block production which might not be a problem depending on how democratized quantum computers become (if at all).
* <b>For consensus with proof-of-stake</b>: validators signing blocks could be impersonated by a quantum adversary. Any deep forks could be mitigated with checkpointing and a quantum-secure checkpointing chain could run in parallel to the faster EC-based consensus chain.
* <b>For account signatures</b>: a hash of a secret seed to generate a secret signing key will put a quantum shield on the account. This way in case a quantum attacker suddenly pops up and blockchains stop accepting EC-signatures, one can still use hash-based ZK-proof of knowledge of the seed (e.g. a STARK-proof) to prove the ownership of the account.

## Post-Quantum Cryptography

NIST is faciliating the standartization of quantum-safe cryptography with a public competition. It took 9 years so far, but it is still ongoing. Selecting new crypto is not an easy task. By now 3 new standards were published: 2 for signatures and 1 for public-key encryption<span style="color:blue">\*</span>. One more public-key encryption and one signature standard is underway. This brings the total to 5 new algorithms, but this is not the finish line: a new smaller-scope competition has been opened for additional signature algorithms.

<span style="color:blue">\*</span><span style="color:lightgray"> I am using the terms key-encapsulation mechanism (KEM), public-key encryption and key-exchange interchangeably. There are slight differences in those that won't matter for this post. Generally if you have any one of those, you can typically get the others.</span>

<b>NIST Competition Timeline:</b>

* 2016 (Dec) - NIST announced the call for proposals for post-quantum schemes ([full timeline](https://csrc.nist.gov/Projects/post-quantum-cryptography/workshops-and-timeline))
* 2020 (Oct) - _stateful_ [post-quantum signature standards](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-208.pdf): XMSS and LMS. Those are advised to be used only in hardware to enforce state advance with cold reboots.
* 2024 (Aug) - from 82 initial submissions, three FIPS standards were published:
  * **[ML-KEM](https://csrc.nist.gov/pubs/fips/203/final)** (Module-Lattice-Based) Key Encapsulation Mechanism or **Encryption**
    * Based on structured lattices, [CRYSTALS-KYBER](https://pq-crystals.org/). Has a Diffie-Hellman style of key-exchange, although as a key exchange it will have 3 rounds instead of 2 as for Diffie-Hellman.
<!--    * Patents: [lifted](https://csrc.nist.gov/csrc/media/Projects/post-quantum-cryptography/documents/selected-algos-2022/nist-pqc-license-summary-and-excerpts.pdf) -->
    * Performance: very fast (faster than x25519), but keys and ciphertexts are big, approximately a kilobyte each - pk:1KB, ct:1KB. The secret key can be stored as a short seed at the expense of the expansion-time, the expanded secret key is about 2 KB, while the seed is only 32 B.
  * **[ML-DSA](https://csrc.nist.gov/pubs/fips/204/final)** (Module-Lattice-Based) **Signature**
    * Based on structured lattices, [CRYSTAL-DILITHIUM](https://pq-crystals.org/dilithium/index.shtml).
    * Performance: sizes are large - signature: 2-4KB, pk: 1-2KB, the speed is very good.
  * **[SLH-DSA](https://csrc.nist.gov/pubs/fips/205/final)** (Stateless Hash-Based Digital) **Signature**
    * Based on hash signatures, [SPHINCS+](https://sphincs.org/data/sphincs+-r3.1-specification.pdf)
    * Each key pair can be used to only sign 2^64 messages which is reasonable since even if &ldquo;*a key pair is signing 10 billions messages a second, it would take over 58 years to sign 2^64 messages*&rdquo;.
    * Performance: keys are small, but signatures are huge: 10-20 KB, the signing and verification are slow.
* 2025 (Mar) - **HQC (Encryption)** was selected for standartization as a code-based key-encapsulation mechanism, a draft is expected early 2026.
  * Performance: several times worse than ML-KEM in all metrics, but based on a different underlying assumption so diversifies the toolkit.
* 2025 (Sep) - **FN-DSA (Signature)** based on Falcon signature scheme from structured lattices is [written as a draft standard](https://csrc.nist.gov/csrc/media/presentations/2025/fips-206-fn-dsa-(falcon)/images-media/fips_206-perlner_2.1.pdf), but not made public yet. While ML-KEM is a Diffie-Hellman-style key exchange, Falcon is an RSA-style key exchange - it uses a trapdoor function. It has small signatures and public keys making it "best among the worst", but tricky to implement in constant-time (see [Cloudflare blogpost](https://blog.cloudflare.com/nist-post-quantum-surprise/#floating-points-falcons-achilles) and [x-post](https://x.com/bwesterb/status/1509583201848672258)).

By now we have 3 standardized schemes for post-quantum: ML-KEM - for public key encryption, ML-DSA, SLH-DSA - for signatures (not including stateful LMS and XMSS here), with HQC and Falcon to be added soon for each category respectively, bringing us to the total of 5 schemes.

The competition had of course its loud failures of schemes that survived to the last rounds: Rainbow, GemSS and SIKE.

The source of most up-to-date information and discussions on the course of the competition is [the public pqc-forum](https://groups.google.com/a/list.nist.gov/g/pqc-forum/).

There are other cryptographic standards, such as ISO (international), Germany’s BSI, and France’s ANSSI. Over time, many of these organizations have been moving toward alignment with NIST recommendations, so the results of the NIST competition are widely influential.

Cloudflare, which proxies approximately 20% of the web, [reports](https://blog.cloudflare.com/pq-2025/) (Oct 2025)  that the majority of human-initiated traffic with Cloudflare is using post-quantum encryption already! Around 15 million TLS connections are established with Cloudflare per second, 2 public keys and 5 signatures are being sent to initiate a connection (only one of the signatures is produced on the fly); the median cert-chain (w. compression, pre-postquantum) is 3.2 KB; and half of the data sent over more than half of QUIC connections is just for the certs. Certificates aren't post-quantum yet, IETF standartization is in progress for [hybrid certs](https://datatracker.ietf.org/doc/draft-ietf-lamps-pq-composite-sigs/).

[Let’s Encrypt issues](https://letsencrypt.org/stats/) more than 6 million certificates a day.

## Additional competition for signature schemes

NIST initiated an additional competition for signature schemes, are the selected ones have performance that is much worse than our current schemes.
Started in 2022 with 14 schemes surviving by now (see [2024 (Oct) announcement](https://csrc.nist.gov/Projects/pqc-dig-sig/round-2-additional-signatures)).
The three most subjectively notable are:
* FAEST - hash-based - similar PICNIC but based on AES (PICNIC is based on a secure multiparty computation of a block cipher LowMC). Generally it would be nice to improve hash-based signatures in terms of performance.
* HAWK -  kind of like FALCON, but without the Floating-Point hassle. And it signs [5 to 30 times faster](https://eprint.iacr.org/2022/1155).
* UOV - unbalanced Oil and Vinegar - large public keys (70 KB), small sigs (100 B) and excellent signing and verification time. Really good for when the public key need not be sent. Based on a 26-year old problem that withstood cryptanalysis so far (for appropriately chosen params).

## Concluding Remarks

There are sadly scarce applications for quantum computers except for breaking RSA and Diffie-Hellman systems today (possibly quantum chemistry or "simulating nature" per Richard Feynman's vision - but that's	far-fetched). There is clearly a lack of research into quantum algorithms in general. I am sure though there will eventually be many. Still it would be quite extraordinary if Diffie, Hellman, Merkle and others are remembered in centuries not so much for invention of public key cryptography (that might get replaced with quantum channels) but rather for facilitating the invention of a quantum computer!

**Further references**:
* Cloudflare: blog-post ["State of the post-quantum Internet in 2025"](https://blog.cloudflare.com/pq-2025/)
* Craig Gidney: [blog](https://algassert.com/)
* Scott Aaronson: [blog](https://scottaaronson.blog/)
* Sam Jaques: [talk](https://www.youtube.com/watch?v=nJxENYdsB6c) "Expected and Unexpected Developments in Quantum Computing", PQCrypto2025
* [PQCrypto](https://pqcrypto2026.irisa.fr/) - a conference on quantum cryptography (annual, started in 2006)
* [Q-Day Clock](https://www.projecteleven.com/q-day-clock) - somewhat similar to doomsday clock, but for quantum computing.
* Mike and Ike is still the best book on quantum computing (except for modern quantum error correction codes, e.g. surface code): "[Quantum Computation and Quantum Information](https://www.amazon.com/Quantum-Computation-Information-10th-Anniversary/dp/1107002176)" by Michael A. Nielsen and Isaac L. Chuang.
* Umesh Vazirani gave a Coursera course in 2012: "Quantum Mechanics & Quantum Computation" though no longer available online, the recordings can still be found on Youtube. It is an excellent intro.
* Lecture notes from courses [by Umesh Vazirano (2007)](https://people.eecs.berkeley.edu/~vazirani/quantum.html), [by David Mermin (2007)](http://mermin.lassp.cornell.edu/qcomp/CS483.html), [Luca Trevisan (2012)](https://lucatrevisan.github.io/teaching/cs259q-12/), [by Scott Aaronson (2017)](https://www.scottaaronson.com/blog/?p=3943), [by Ronald de Wolf (2019)](https://arxiv.org/abs/1907.09415), [by John Preskill (2024)](https://www.preskill.caltech.edu/ph229/).
* Other books: [From Classical to Quantum Shannon Theory](https://arxiv.org/abs/1106.1445) by Mark M. Wilde (2011); Classical and Quantum Computation by Kitaev, Shen, and Vyalyi (2002), Quantum Computing Since Democritus by Aaronson (2013), [The Theory of Quantum Information](https://cs.uwaterloo.ca/~watrous/TQI/) by Watrous (2018).

<!-- As Ronald de Wolf noted "to all those currently working on yet another book about quantum computing: what this field needs most is more algorithms, not more books". -->

<!-- In the standard circuit model, a quantum computation on n qubits consists of applying a sequence of unitary transformations (each acting on one or a few qubits) to a state vector of 2^n complex amlitudes, followed by measurements that yield classical outcomes according to the squared magnitudes of the amplitudes. -->