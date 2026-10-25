---
title: "Understanding collision resistance, quantifiers and (non-)uniformity"
date: 2026-10-22
slug: collision-resistance-quantifiers
---

One of the things I have always loved in cryptography is black-box separation results (informally, they say that you cannot build X from Y in a black-box way).
The two classic examples that I started with are the separation of key agreement from one-way permutations (OWPs) by Impagliazzo and Rudich [IR89], and the separation of collision-resistant hash functions (CRHFs) from OWPs by Simon [Sim98].
After reading Simon's seminal paper, I moved on to Hsiao and Reyzin's paper on separating public-coin CRHFs from secret-coin CRHFs [HR04].
They claim on the first page that

> It is easy to see there is no meaningful way to formalize the notion of collision-resistance for a single fixed-output-length hash function.

<!--more-->

As someone who frequently works with EasyCrypt, I found this strange, as I thought we had a concrete collision resistance proof for SHA-3 with a fixed output length $x$ [ABB+19, Thm 2.3].
Hsiao and Reyzin also note in a footnote on the second page that

> It is possible to define a single hash function (with variable output-length; cf. previous paragraph) instead of a collection of them. In this case, it can be collision-resistant only against a uniform adversary.

The mention of uniformity reminded me of the pain I went through to grasp the nuanced differences between uniform and non-uniform models, and why we should even care.
For example, a variable-output-length hash function can be secure in one model and insecure in the other, even though at face value both definitions look almost identical.
This seems weird.
We consider SHA3-256 to be collision resistant, yet according to Hsiao and Reyzin there is no meaningful way to even say this, and even a variable-output-length version would only be collision resistant against uniform adversaries.

## Security models

We have two main flavours of security proofs, those done in the asymptotic uniform setting [KL20] and those done in the concrete setting [BKR00, BDJR97].
There is also an asymptotic non-uniform setting [GM84, Gol01], which I rarely come across, so I will not discuss it further.
Theory papers are almost always asymptotic, while concrete security is mostly used to analyse specific constructions.
Let us discuss their differences.

### Quantifiers

In my opinion, the difference between these notions lies almost entirely in the **quantifiers**.
In the asymptotic uniform notion, the adversary is one fixed program, and we ask how its success behaves as the security parameter grows.
In the concrete notion, we first fix the security parameter to a particular value and assign a budget $t$, and then ask for one bound that covers every adversary within that budget.
In a relatively informal way of writing this, we can express the two security notions as

$$
\begin{array}{lll}
\textbf{Asymptotic uniform:} & \forall\, \text{PPT}\,\, \mathcal{A}\,\, \forall \ell &
  \Pr\big[\,G^{\mathcal{A}(1^\ell)}(1^\ell) \Rightarrow 1\,\big] \le \text{neg}_{\mathcal{A}}(\ell) \\[6pt]
\textbf{Concrete (non-uniform):} & \text{fix } \ell,\ \forall a_\ell\,  \forall \mathcal{A},\ |\mathcal{A}(a_\ell)| \le t &
  \Pr\big[\,G^{\mathcal{A}(a_\ell)}(1^\ell) \Rightarrow 1\,\big] \le \text{small}(t)
\end{array}
$$

Here $\ell$ is the security parameter, which in our examples will be the output length of a hash function, and $|\mathcal{A}(a_\ell)|$ measures the adversary's size and running time together, including its advice and anything else hard-coded into its code.
The advice $a_\ell$ is extra information that may depend on $\ell$, and while its size counts towards the budget $t$, the work needed to produce it is free.
$G^{\mathcal{A}(\cdot)}$ denotes the game run with $\mathcal{A}$ started on the given inputs, plus whatever the game itself hands it (keys, challenges, oracle answers).
The negligible function $\text{neg}_{\mathcal{A}}$ is chosen separately for each adversary, so different adversaries may get different bounds.
This matters, since a single $\text{neg}(\ell)$ for all adversaries would result in a false statement.
Notice that in the concrete case the bound $\text{small}(t)$ cannot depend on the adversary, so it has to hold for **all** adversaries of size at most $t$.

### Non-uniform

One can think of a non-uniform adversary as a fixed machine whose code is a finite string, and accepts an advice string $a_\ell$ that depends on $\ell$ as input.
In the concrete definition, $\ell$ is fixed first and we then quantify over every $a_\ell$, so the advice, and with it the adversary, may depend on $\ell$.
Notice that the definition says nothing about where the advice comes from!
It could be the result of some arbitrarily long offline preprocessing phase that compresses something very useful into at most $t$ bits, such as a hard-coded collision.

Side note, I prefer to think of non-uniform adversaries as Turing machines with an advice tape rather than as circuits.
The advice tape makes the possible preprocessing step much more visible.

### Uniform

The adversary's code is one finite string that does not depend at all on $\ell$, and is only given information on $\ell$ inside the game, after its code is fixed.
It still could come preloaded with collisions for some fixed lengths, irrespective of $\ell$.
But a finite string can't contain a collision for each of the infinitely many lengths, so even if the adversary's code comes preloaded with 1000 collisions, it only *trivially* wins at those 1000 lengths.
Beyond them it has to actually compute a collision, and since it runs in polynomial time, it cannot simply brute-force one.

This is actually fine, because a negligible function only has to be small for large enough $\ell$, so the probability of the adversary winning at those 1000 lengths is irrelevant.

#### What is large enough?

Of course, nothing stops the code from holding even more collisions, such as a billion trillion of them.
Any finite number of collisions is allowed, and it pushes the point where the adversary has to start computing further out.
So what is "large enough"?
Unfolding the definition of negligible makes this clearer.

$$
\forall\, \text{PPT } \mathcal{A}\ \ \forall c\ \ \exists N\ \ \forall \ell \ge N \quad \Pr\big[\,G^{\mathcal{A}(1^\ell)}(1^\ell) \Rightarrow 1\,\big] \le \ell^{-c}
$$

The threshold $N$ is chosen *after* the adversary, so this number may depend on anything preceding it in the quantifier order, including how many collisions are hard-coded into its code.
An adversary with a billion trillion collisions simply gets a larger $N$.

This also means that the asymptotic definition is **not** a collection of statements, one for each $\ell$.
The definition is a single statement about what happens as $\ell$ grows, and it says nothing about any concrete parameter, such as the first 1000 lengths, or the one you actually deploy.

## Why fixing $\ell$ breaks a single hash function

### Fixed length

Take a single hash function such as $H \colon \{0,1\}^{2n} \to \{0,1\}^n$.
Assume, intuitively, that it is not feasible for an adversary to find a collision in $H$.
Let us try to formalise this in the following game, which is the usual collision game [RoSh04] with the "key" removed (yes, there are keys, we will get to that).

$$
\begin{array}{l}
\underline{\mathsf{Fixed}_H(\mathcal{A})} \\
(x_1, x_2) \leftarrow \mathcal{A} \\
\text{return } x_1 \ne x_2 \wedge H(x_1) = H(x_2)
\end{array}
$$

In the *concrete* setting we would want the following.

$$
\forall \mathcal{A},\ |\mathcal{A}| \le t,\quad
\Pr\big[\mathsf{Fixed}_H(\mathcal{A}) \Rightarrow 1\big] \le \text{small}(t)
$$

Even if $H$ is intuitively collision resistant, surprisingly, this statement is **false**!
Since $H$ maps inputs to shorter outputs, a collision must exist.
Because $H$ is fixed before we quantify over $\mathcal{A}$, there is an $\mathcal{A}$ that just prints this collision and wins with probability $1$.
As we quantify over ALL adversaries, this is enough to break the security statement.
This stupid and annoying $\mathcal{A}$ shows that, to break this definition, nobody needs to *know* the collision, it is enough that it exists.
Hence, for any real-world hash function, this basically gives us nothing, it's completely vacuous.

Note how modelling the adversary as uniform makes no sense here, since there is no security parameter to tend to infinity.
Neither definition works for a fixed $H$, and this is exactly what Hsiao and Reyzin mean by "no meaningful way to formalize".
Koblitz and Menezes make the same observation for non-uniform adversaries, calling a theorem that assumes collision resistance of a fixed hash function "a useless and misleading triviality" [KM13a].

### Can variable output length fix this?

Adding the security parameter back doesn't help in the concrete setting.
Let $H$ take $1^\ell$ as an extra input and output $\ell$ bits (assume that its input length is greater than its output).

$$
\begin{array}{l}
\underline{\mathsf{Var}_H^\ell(\mathcal{A})} \\
(x_1, x_2) \leftarrow \mathcal{A}(1^\ell) \\
\text{return } x_1 \ne x_2 \wedge H(x_1, 1^\ell) = H(x_2, 1^\ell)
\end{array}
$$

In the *concrete* setting, where $\ell$ is fixed, this just degenerates to the fixed-length situation.

Let us understand what happens when we consider the asymptotic *uniform* case.

$$
\forall\, \text{PPT } \mathcal{A}\ \forall \ell,\quad
\Pr\big[\mathsf{Var}_H^\ell(\mathcal{A}) \Rightarrow 1\big] \le \text{neg}_{\mathcal{A}}(\ell)
$$

This statement is potentially true!
Unlike in the concrete setting, $\mathcal{A}$ does not depend on $\ell$, and as we saw above, its code can only hold collisions for finitely many lengths, so for large enough $\ell$ it has to actually find one in polynomial time.
Remember Hsiao and Reyzin's remark that such a function can only be collision resistant against uniform adversaries?
Well, there you go!

Both asymptotic definitions let an adversary break finitely many lengths, but a non-uniform adversary can carry a different answer for every length, since its advice string may depend on $\ell$.
This breaks the scheme outright when each length has a single fixed instance, such as an unkeyed hash, while with keys, the advice can only cover a few of them.

## Putting something after the adversary

How do we fix the concrete setting?
The standard fix, going back to Damgård [Dam87], is to make the hash depend on something chosen *after* the adversary is fixed, such as a key [RoSh04].

$$
\begin{array}{l}
\underline{\mathsf{Keyed}_H(\mathcal{A})} \\
k \leftarrow \mathcal{K} \\
(x_1, x_2) \leftarrow \mathcal{A}(k) \\
\text{return } x_1 \ne x_2 \wedge H_k(x_1) = H_k(x_2)
\end{array}
$$

$$
\forall \mathcal{A},\ |\mathcal{A}| \le t,\quad
\Pr\big[\mathsf{Keyed}_H(\mathcal{A}) \Rightarrow 1\big] \le \text{small}(t)
$$

The key is public, and the adversary even gets it as input, but the adversary's code and advice are fixed before the key is sampled.
In quantifier terms, the hash function is now chosen after $\forall \mathcal{A}$ instead of before it.
The adversary can still have collisions written into it, but only for the few keys that fit into its code.
So it is guaranteed to win if the sampled key happens to be one of them, and $\text{small}(t)$ can never be smaller than that probability.
The random key turns "a collision exists" into "you need a collision for *this* key, but I won't tell you which until you set your code".
Naturally, you need a large key space, otherwise this isn't very interesting.

Keys are therefore a second escape route besides uniformity, and they rule out the hard-coded collision even when non-uniform advice is allowed.
This is also why the EasyCrypt SHA-3 result [ABB+19, Thm 2.3] is meaningful.
Instead of the fixed permutation inside SHA-3, it uses a truly random permutation $P$, sampled inside the game after the adversary is fixed, so $P$ plays the role of the key.
The theorem says that for every adversary with oracle access to $P$ and to SHA3-$x$ instantiated with $P$, making queries of total cost at most $\sigma$, the following holds.

$$
\Pr\big[\mathsf{CR}_{\text{SHA3-}x}(\mathcal{A}) \Rightarrow 1\big] \le
\frac{\sigma^2 - \sigma}{2^{1600+1}} + \frac{\sigma^2}{2^{2x-2}} + \frac{\sigma^2 - \sigma + 2}{2^{x-1}}
$$

Of course, the real SHA3-$x$ uses one fixed permutation (Keccak-$f$[1600]), so for the actual hash function the hard-coded adversary is back and Hsiao and Reyzin's sentence still applies.

## Why do we even care about these adversaries?

I argue that the hard-coded collision is a trivial attack.
The adversary never finds anything, it just has the answer written in, which feels like it's "cheating".
And the fact that a collision exists does not mean anyone knows it.
It's "collision resistance" and not "collision nonexistence".
For a hash function like SHA3-256, what we actually care about is whether someone can *find* a collision, not whether one exists (in fact, one will exist for ANY compressing function).
Seen this way, keys feel like a quirk of the theory that we use to side-step the issue.

The flip side is that once someone does find one, the hard-coded adversary stops being hypothetical.
This happened to SHA-1 [SBK+17], a single published collision was enough to end its collision resistance.
Just like the SHA-3 theorem above, the ideal-cipher collision bounds for the Merkle–Damgård and Davies–Meyer design that SHA-1 follows [BRS02] are still correct theorems, even though nobody relies on SHA-1 for collision resistance any more.
Nothing in the mathematics changed when that collision was found, only what people knew, which is exactly what makes "we know a collision" so hard to capture formally.

If the hard-coded adversary is the problem, why not just exclude it?
Restricting adversaries is nothing unusual in itself, we do it all the time, but usually in one of two ways.
Either we bound their resources (running time, size, number of queries), or we restrict how they interact with the game, for instance by requiring them to never reuse a nonce, or to never ask for the challenge ciphertext to be decrypted.
Restrictions of the second kind are visible in the transcript, so the game can enforce them itself with a guard.

### How do we even classify these adversaries?

"Has a collision written into it" is neither one of those.
A resource bound doesn't catch it, since a collision is short to write down.
It also leaves no trace in the transcript, since a stored collision and a found one look the same once they are output.
It is not even clear what such a restriction would say.
What if the advice is a short interval of inputs that is guaranteed to contain a collision, or a seed from which a cheap search finds one?
A collision can be stored encoded, split up, or reconstructed after a few steps of computation.
Are those cryptographically "interesting" advice strings, or are they "cheating"?
In short, it is not clear how we would restrict these adversaries at all, and any attempt risks excluding adversaries we actually care about.

## What actually works

The fixes that do work never have to say which adversaries "have" a collision.
The first is to put something random after the adversary, such as a key or an ideal permutation, so that a stored collision only helps for the few keys it was stored for.

The second is to ask more of the adversary before it counts as a win, as in multi-collision resistance [BKP18].
In that setting, the hash can be keyless and the adversary can have advice, but it only wins by finding more inputs with the same hash than its advice could store.
This is a weaker notion, but one that makes sense for a keyless function.

The third is Rogaway's [Rog06], who accepts that a collision finder for a fixed $H$ always exists, and builds on the fact that nobody knows it.
When a cryptosystem $G$ is built from an unkeyed $H$, he never states a collision resistance bound for $H$, and instead gives an explicit reduction that turns any attack on $G$ into a collision finder for $H$.
So whether $H$ is collision resistant becomes an informal assumption about what people know, which can change over time, as it did for SHA-1, while we still trust SHA3-256.

## Advice can still help adversaries

Keys are not the end of the story either.
Bernstein and Lange show that advice which merely *helps*, such as huge precomputed tables, breaks the usual concrete security conjectures even for keyed primitives like AES-128 [BL13].
Koblitz and Menezes argue that the non-uniform model is a poor fit for practice-oriented reductions in general [KM13a].
Both sides of this debate, together with the auxiliary-input line of work [Unr07, DGK17, CDG18] that allows bounded advice about the sampled oracle itself, deserve their own post, so I will leave it here for now.

## Closing thoughts

What is my opinion?
Honestly, I don't know, and I feel like I need decades of experience to begin to form one.
On one hand, concrete security gives explicit numerical bounds, but how useful they are depends on what advice the model allows.
On the other hand, with asymptotic uniform security, the adversary cannot carry advice that grows with the security parameter, but you lose the coefficients, and there is no guarantee that the scheme is secure at any particular parameter.

Regardless, I think foundations are essential and should be studied by all aspiring cryptographers.
However, I also, somewhat naively, think we should ask whether they need some altering when we cannot express something as basic as "it is hard to find a collision" without a list of caveats.
Keys, uniformity, ideal models and explicit reductions each fix part of the problem, but none of them says exactly what we mean.
Maybe we don't even know what we mean by collision resistance!

Decades of work have been built on the current foundations, and changing them may not be worth it, or even possible (both in a social and a mathematical sense).
What I take away is that these quirks are worth understanding.
They decide what a security theorem actually says, and that is easy to lose sight of.

## AI Usage

I used Claude Opus 5.5 on high effort to format the text and references and to proofread this.
Since I am someone who thinks better by talking, I also used the chat function as a bit of a back and forth to develop my ideas.

## References

- [ABB+19] J. B. Almeida, C. Baritel-Ruet, M. Barbosa, G. Barthe, F. Dupressoir, B. Grégoire, V. Laporte, T. Oliveira, A. Stoughton, P.-Y. Strub. Machine-Checked Proofs for Cryptographic Standards: Indifferentiability of Sponge and Secure High-Assurance Implementations of SHA-3. ACM CCS 2019, pp. 1607–1622. Available at <https://doi.org/10.1145/3319535.3363211>.
- [BDJR97] M. Bellare, A. Desai, E. Jokipii, P. Rogaway. A Concrete Security Treatment of Symmetric Encryption. FOCS 1997, pp. 394–403. Available at <https://doi.org/10.1109/SFCS.1997.646128>.
- [BKP18] N. Bitansky, Y. T. Kalai, O. Paneth. Multi-Collision Resistance: A Paradigm for Keyless Hash Functions. STOC 2018, pp. 671–684. Available at <https://doi.org/10.1145/3188745.3188870>.
- [BKR00] M. Bellare, J. Kilian, P. Rogaway. The Security of the Cipher Block Chaining Message Authentication Code. Journal of Computer and System Sciences 61(3), pp. 362–399, 2000. (Preliminary version at CRYPTO 1994.) Available at <https://doi.org/10.1006/jcss.1999.1694>.
- [BL13] D. J. Bernstein, T. Lange. Non-uniform Cracks in the Concrete: The Power of Free Precomputation. ASIACRYPT 2013, LNCS 8270, pp. 321–340. Available at <https://doi.org/10.1007/978-3-642-42045-0_17>.
- [BRS02] J. Black, P. Rogaway, T. Shrimpton. Black-Box Analysis of the Block-Cipher-Based Hash-Function Constructions from PGV. CRYPTO 2002, LNCS 2442, pp. 320–335. Available at <https://doi.org/10.1007/3-540-45708-9_21>.
- [CDG18] S. Coretti, Y. Dodis, S. Guo. Non-Uniform Bounds in the Random-Permutation, Ideal-Cipher, and Generic-Group Models. CRYPTO 2018, LNCS 10991, pp. 693–721. Available at <https://doi.org/10.1007/978-3-319-96884-1_23>.
- [Dam87] I. Damgård. Collision Free Hash Functions and Public Key Signature Schemes. EUROCRYPT 1987, LNCS 304, pp. 203–216. Springer, 1988.
- [DGK17] Y. Dodis, S. Guo, J. Katz. Fixing Cracks in the Concrete: Random Oracles with Auxiliary Input, Revisited. EUROCRYPT 2017, LNCS 10211, pp. 473–495. Available at <https://doi.org/10.1007/978-3-319-56614-6_16>.
- [GM84] S. Goldwasser, S. Micali. Probabilistic Encryption. Journal of Computer and System Sciences 28(2), pp. 270–299, 1984. Available at <https://doi.org/10.1016/0022-0000(84)90070-9>.
- [Gol01] O. Goldreich. Foundations of Cryptography, Volume 1: Basic Tools. Cambridge University Press, 2001. Available at <https://doi.org/10.1017/CBO9780511546891>.
- [HR04] C.-Y. Hsiao, L. Reyzin. Finding Collisions on a Public Road, or Do Secure Hash Functions Need Secret Coins? CRYPTO 2004, LNCS 3152, pp. 92–105. Available at <https://doi.org/10.1007/978-3-540-28628-8_6>.
- [IR89] R. Impagliazzo, S. Rudich. Limits on the Provable Consequences of One-Way Permutations. STOC 1989, pp. 44–61. Available at <https://doi.org/10.1145/73007.73012>.
- [KL20] J. Katz, Y. Lindell. Introduction to Modern Cryptography, 3rd edition. Chapman & Hall/CRC, 2020. Available at <https://www.cs.umd.edu/~jkatz/imc.html>.
- [KM13a] N. Koblitz, A. Menezes. Another Look at Non-Uniformity. Groups Complexity Cryptology 5, pp. 117–140, 2013. Available at <https://eprint.iacr.org/2012/359>.
- [Rog06] P. Rogaway. Formalizing Human Ignorance: Collision-Resistant Hashing without the Keys. VIETCRYPT 2006, LNCS 4341, pp. 211–228. Available at <https://doi.org/10.1007/11958239_14>.
- [RoSh04] P. Rogaway, T. Shrimpton. Cryptographic Hash-Function Basics: Definitions, Implications, and Separations for Preimage Resistance, Second-Preimage Resistance, and Collision Resistance. FSE 2004, LNCS 3017, pp. 371–388. Available at <https://doi.org/10.1007/978-3-540-25937-4_24>.
- [SBK+17] M. Stevens, E. Bursztein, P. Karpman, A. Albertini, Y. Markov. The First Collision for Full SHA-1. CRYPTO 2017, LNCS 10401, pp. 570–596. Available at <https://doi.org/10.1007/978-3-319-63688-7_19>.
- [Sim98] D. R. Simon. Finding Collisions on a One-Way Street: Can Secure Hash Functions Be Based on General Assumptions? EUROCRYPT 1998, LNCS 1403, pp. 334–345. Available at <https://doi.org/10.1007/BFb0054137>.
- [Unr07] D. Unruh. Random Oracles and Auxiliary Input. CRYPTO 2007, LNCS 4622, pp. 205–223. Available at <https://doi.org/10.1007/978-3-540-74143-5_12>.
