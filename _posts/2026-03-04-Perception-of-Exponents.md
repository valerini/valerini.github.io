---
layout: post_container
title: Perception of Exponents
description: "Putting different exponents into perspective"
image: /images/Exponents.png
---

# Perception of Exponents
<span style="color:lightgray">{{ page.date | date: "%B, %Y" }}</span>

![Exponents](/images/Exponents.png){: style="max-width:600px; width:100%; height:auto;" }

Exponents are very counterintuitive, they get very big very fast.
I find it helpful to always keep this number in mind: the number of atoms in the observable universe is about  2<sup>300</sup>, actually about 2<sup>266</sup> more precisely. So with 256 bits one can count 1/1000-th of the atoms in the Universe. Therefore a brute-force search for a 256-bits key would have to "look under" each 1000-th atom in the universe... that "actually feels" like a safe margin!

I also find it very helpful to know how much hashing the Bitcoin network is currently doing per second, that's about 2<sup>70</sup> (and thus 2<sup>95</sup> per year). Then it makes sense, why 80-bits keys are no longer considered adequate.

Here are some more helpful numbers along these lines:

* **2<sup>25</sup>** seconds in a year;
* **2<sup>34</sup>** hashes/second modern GPU (GeForce RTX 4090 - $2,000)
* **2<sup>50</sup>** hashes/second industrial-grade miner (Bitmain Antminer S23 Hyd 3U - $30,000 + $5,000/year for electricity)
* **2<sup>59</sup>** the age of the universe in seconds
* **2<sup>70</sup>** hashes/second Bitcoin network
* **2<sup>73</sup>** [number of stars](https://www.esa.int/Science_Exploration/Space_Science/How_many_stars_are_there_in_the_Universe) in the Universe
* **2<sup>80</sup>** steps to attack considered feasible
* **2<sup>95</sup>** hashes/year Bitcoin network
* **2<sup>128</sup>** steps to attack is about the lifetime of the universe for bitcoin hashpower if one step is one hash
* **2<sup>256</sup>** steps an attacker would not be able to do ever
* **2<sup>266</sup>** atoms in the observable universe
* **2<sup>403</sup>** the number of ops the universe has made in its lifetime, if one atom is one compute device doing one op at a speed the light traverses its diameter (1 fm) (see the paper of [Lloyd from 2001](https://arxiv.org/pdf/quant-ph/0110141) for a more accurate assessment).

It is only meaninful to think about an exponential-time algorithm, O(2<sup>n</sup>), for inputs of small length n < 300. For larger n it becomes kind of meaninless. Same is true for a polynomial-time algorithm, if the runtime is O(n<sup>300</sup>) it samewise infeasible.

I was once in a group of students helping Don Knuth to clean-up a small local library in the Stanford's CS department. At the time he was working on SAT solvers for Volume 4 of his The Art of Computer Programming book series. When we were done, I asked him: "What's your intuition now, do you think P = NP or not?" And his answer was most remarkable. He said he feels P = NP, but the degree of the polynomial would be so large, that it is not tractable in any practical sense. Indeed, if it takes O(n<sup>300</sup>) time to solve an NP-complete problem, it is well above feasible. So cryptographers would still stay in business ;)
