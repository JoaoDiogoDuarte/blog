---
title: "Understanding collision resistance, quantifiers and (non-)uniformity"
date: 2026-10-22
slug: collision-resistance-quantifiers
---

One of the things I have always loved in cryptography is black-box separation results [RTV04].
Two classic examples are the separation of Public Key Encryption (PKE) from One-way Functions (OWF) by Impagliazzo and Rudich [IR89], and the separation of Collision Resistant Hash Functions (CRHF) from One-way Permutations (OWP) by Simon [Sim98].
After Simon's seminal paper, I started reading Hsiao and Reyzin's paper on separating public-coin CRHFs from secret-coin CRHFs [HR04], which claims on the first page that

> It is easy to see there is no meaningful way to formalize the notion of collision-resistance for a single fixed-output-length hash function.

<!--more-->

As someone who works primarily in EasyCrypt [BGHZ11], I found this strange, as I thought we had a collision resistance proof for SHA3 with output length $x$ (more on this later!).
Theory papers are almost always asymptotic, but they split on uniformity.
For example, Goldreich [Gol01] and Goldwasser–Bellare [GB08] default to non-uniform adversaries, while Katz–Lindell [KL20] and much of the separation literature use uniform ones.
Concrete security is mostly used to analyse specific constructions [BKR00, BDJR97].
EasyCrypt also works in the concrete setting, since its program logics produce explicit probability bounds rather than asymptotic statements.

Because of this and some free time on a flight, I thought this would be an interesting first blog post.
It is about how quantifiers always seem to introduce irritating, nuanced pain points that, at least for me, are invisible at first sight.

## Security models

We have three main flavours of security proofs, those done in the asymptotic uniform setting [GM84, KL20], those done in the asymptotic non-uniform setting [Gol01], and those done in the concrete setting [BKR00, BDJR97].
In my opinion, the difference between these notions lies almost entirely in the order of the quantifiers.

$$
\begin{array}{lll}
\textbf{Asymptotic Uniform:} & \forall \mathcal{A}\ \forall \ell &
  \Pr\big[\,G^{\mathcal{A}(1^\ell)}(1^\ell) \Rightarrow 1\,\big] = \mathsf{negl}(\ell) \\[6pt]
\textbf{Asymptotic Non-uniform:} & \forall \ell\ \forall \mathcal{A}_\ell,\ |\mathcal{A}_\ell| \le p(\ell) &
  \Pr\big[\,G^{\mathcal{A}_\ell(1^\ell)}(1^\ell) \Rightarrow 1\,\big] = \mathsf{negl}(\ell) \\[6pt]
\textbf{Concrete:} & \text{fix } \ell,\ \forall \mathcal{A},\ |\mathcal{A}| \le t &
  \Pr\big[\,G^{\mathcal{A}} \Rightarrow 1\,\big] \le \varepsilon(t)
\end{array}
$$

Here $|\mathcal{A}|$ measures the adversary's size and running time together, including anything hard-coded into its code.
$G^{\mathcal{A}(\cdot)}$ denotes the game run with $\mathcal{A}$ started on the given inputs, plus whatever the game itself hands it (keys, challenges, oracle answers).

In the non-uniform setting we quantify first over the security parameter and then over the adversary.
One can think of $\mathcal{A}_\ell$ as a fixed machine together with an advice string $a_\ell$ that depends on $\ell$.
In the uniform setting the order is reversed, as we fix the adversary's code once, and it must then work for every security parameter.
A uniform adversary is therefore a non-uniform one whose advice does not depend on the security parameter.
It still has a constant amount of advice, namely whatever is written into its code.

Notice that the definition says nothing about where the advice comes from!
It could be the result of some arbitrarily long offline preprocessing phase that compresses something very useful into polynomially many bits, such as a hard-coded collision.
In the concrete setting there is only one $\ell$, so whatever advice helps at that $\ell$ can simply be written into $\mathcal{A}$.
This is why concrete security is, in effect, non-uniform [BL13].

### Side note on modelling non-uniform adversaries
Note, I prefer to think of non-uniform adversaries as Turing machines with an advice tape rather than as circuits.
The two are equivalent [KL80, AB09], but the advice tape makes the possible preprocessing step much more visible, and this will matter further along.

## Why fixing $\ell$ breaks a single hash function

Take a single hash function such as $H \colon \{0,1\}^{*} \to \{0,1\}^n$.
Assume, intuitively, that it is not feasible for an adversary to find a collision in $H$.
Let us try to formalize this in the following game, which is the usual collision game [RoSh04] with the key removed.

$$
\begin{array}{l}
\underline{\mathsf{Fixed}_H(\mathcal{A})} \\
(x_1, x_2) \leftarrow \mathcal{A} \\
\text{return } x_1 \ne x_2 \wedge H(x_1) = H(x_2)
\end{array}
$$

In the concrete setting we would want the following.

$$
\forall \mathcal{A},\ |\mathcal{A}| \le t,\quad
\Pr\big[\mathsf{Fixed}_H(\mathcal{A}) \Rightarrow 1\big] \le \text{small}
$$

This statement is **false**!
Since $H$ maps infinitely many inputs to finitely many outputs, a collision $(c_1, c_2)$ must exist, and in fact all but at most $2^n$ inputs collide with some other input.
Note that $H$ is fixed before we quantify over $\mathcal{A}$.
So there is an $\mathcal{A}$ with that collision written into it, namely the one that returns $(c_1, c_2)$.
This single and annoying $\mathcal{A}$ wins with probability $1$.
We don't need to *know* $(c_1, c_2)$, it is enough that it exists.

Adding a security parameter doesn't help on its own.
Let $H$ take $1^\ell$ as an extra input, with output length growing in $\ell$.

$$
\begin{array}{l}
\underline{\mathsf{Var}_H^\ell(\mathcal{A})} \\
(x_1, x_2) \leftarrow \mathcal{A}(1^\ell) \\
\text{return } x_1 \ne x_2 \wedge H(x_1, 1^\ell) = H(x_2, 1^\ell)
\end{array}
$$

In the asymptotic *non-uniform* setting, we have the following.

$$
\forall \ell\ \forall \mathcal{A}_\ell,\ |\mathcal{A}_\ell| \le p(\ell),\quad
\Pr\big[\mathsf{Var}_H^\ell(\mathcal{A}_\ell) \Rightarrow 1\big] = \mathsf{negl}(\ell)
$$

Again, this is **false**.
$\mathcal{A}_\ell$ is chosen after $\ell$, so it can have a collision for $H(\cdot, 1^\ell)$ written into it.
That collision has size $O(\ell)$, well within the polynomial bound.
This is annoying!

Finally, let us consider the asymptotic *uniform* case.

$$
\forall \mathcal{A}\ \forall \ell,\quad
\Pr\big[\mathsf{Var}_H^\ell(\mathcal{A}) \Rightarrow 1\big] = \mathsf{negl}(\ell)
$$

This statement is potentially true!
$\mathcal{A}$ is fixed before $\ell$.
It would need a collision for every $\ell$ written into finitely many bits of code.
It can hold a few, but for large $\ell$ it has to actually find them.
Hsiao and Reyzin make exactly this point in a footnote [HR04], noting that such a function can only be collision resistant against uniform adversaries.

For a fixed-output-length function like SHA-256 there is no $\ell$ to grow.
So the uniform option doesn't exist, and we are back in the first case.
Every failure above has the same shape, namely that $H$ is fixed, so a collision exists, and the adversary is chosen after it.

## Putting something after the adversary

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
\Pr\big[\mathsf{Keyed}_H(\mathcal{A}) \Rightarrow 1\big] \le \varepsilon(t)
$$

The adversary can still have collisions written into it, but only for about $t / 2n$ keys.
The random key turns "a collision exists" into "you need a collision for *this* key".

This is also why the EasyCrypt SHA-3 result [ABB+19] is meaningful.
The theorem says that for every adversary with oracle access to a truly random permutation $P$ and to SHA3-$x$ instantiated with $P$, making queries of total cost at most $\sigma$, the following holds.

$$
\Pr\big[\mathsf{CR}^{\mathcal{A}}_{\text{SHA3-}x} = 1\big] \le
\frac{\sigma^2 - \sigma}{2^{1600+1}} + \frac{\sigma^2}{2^{2x-2}} + \frac{\sigma^2 - \sigma + 2}{2^{x-1}}
$$

The permutation $P$ is sampled inside the game, after the adversary is fixed, so it plays the role of the key.

## Why do we even care about these adversaries?

I argue that the hard-coded collision is a trivial attack.
The adversary never finds anything, it just has the answer written in, which feels like it's "cheating".
And the fact that a collision exists does not mean anyone knows it.
It's "collision resistance" and not "collision freeness"!
For a hash function like SHA-256, what we actually care about is whether someone can *find* a collision, not whether one exists (in fact, one will exist for ANY compressing function).
Seen this way, keys feel like a quirk of the theory that we just side-step around.

If the hard-coded adversary is the problem, why not just exclude it?
The obvious attempt is to quantify only over adversaries that do not have a collision written into them.
This does not work, because "has a collision written into it" is not something we can define.
What if the advice is "the collision can be found between range X and range Y"?
A collision can be stored encoded, split up, or reconstructed after a few steps of computation.
Are those cryptographically "interesting" advice strings, or are they "cheating"?
Any adversary that outputs a collision in time $t$ has, in some sense, a collision written into it.
So excluding all of them means excluding every adversary that wins, and the definition becomes empty.
More generally, excluding adversaries is a slippery slope that cryptography tries to avoid.
It is easy to over-reach and exclude adversaries we actually care about.

## IND-CCA and guards

This reminds me of how we deal with trivial wins in IND-CCA [RaSi91, BDPR98].
There, we handle them directly in the code of the game.
An adversary could come preloaded with some decryption, but this is not a problem.
The key, the challenge bit and the encryption randomness are all sampled after the adversary is fixed.
So nothing it carries in its code can tell it which message was encrypted.
The only trivial win left is to ask the oracle to decrypt the challenge itself, which the guard in the decryption oracle refuses to answer.
So between the randomness and the guard, the game removes every trivial win.

The collision game for a fixed hash function has neither of these.
Nothing is sampled after the adversary, so a preloaded collision is still valid when the game runs.
And there is no guard we could write, because the game cannot tell a stored collision from a found one.
Adding a key is exactly the move that gives the collision game what IND-CCA already has.

## What actually works

The fixes that do work avoid excluding anyone.
The first is to put something random after the adversary, such as a key or an ideal permutation, so that a stored collision is useless.

The second is to weaken what counts as a win, as in multi-collision resistance [BKP18].
In that setting, an adversary only wins by finding more colliding inputs than its own code could store.

Rogaway, in [Rog06], observes that common hash functions are keyless, while our formal definitions of collision resistance are keyed.
His fix is to stop trying to directly calculate a proper collision resistance bound for a fixed $H$.
Instead, when a cryptosystem $G$ is built from $H$, we prove a relative bound

$$
\forall \mathcal{A},\quad \mathsf{Adv}_G(\mathcal{A}) \le \mathsf{Adv}_H\big(\mathcal{B}(\mathcal{A})\big)
$$

for an explicit $\mathcal{B}$ that runs $\mathcal{A}$.
Crucially, $\mathcal{B}$ must be written down explicitly, since if the theorem only said "there exists a $\mathcal{B}$", that $\mathcal{B}$ could simply print a collision for $H$.
Any attack on $G$ becomes an attack on $H$, and whether anyone can attack $H$ is left as an assumption, not a theorem.

The auxiliary-input line of work [Unr07, DGK17, CDG18] is another answer, as it keeps precomputation in the model but bounds the advice and counts it explicitly.
Admittedly, I still need to give this work a proper read, and maybe that'll become another blog post in the future.

## Closing thoughts

I think foundations are essential and should be studied by all aspiring cryptographers.
However, I also, somewhat naively, think we should ask whether they need some altering when we cannot express something as basic as "it is hard to find a collision" without a list of caveats.
Keys, uniformity, ideal models and explicit reductions each fix part of the problem, but none of them says exactly what we mean by "it is hard to find a collision".
I have no good answer to this.

In my interpretation, Bernstein and Lange make the same point for concrete security as a whole in "Non-uniform cracks in the concrete" [BL13].
Concrete security usually defines $2^b$ security as the nonexistence of a good attack running in time at most $2^b$.
They show that, under this definition, attacks well below $2^{128}$ exist against, e.g., AES-128, but these attacks rely on a huge precomputation that the definition does not account for.
They are not realistic and pose no threat to anyone, since actually running them would be far more expensive than the usual attacks.
But the definitions cannot separate them from real attacks, which Bernstein and Lange see as a flaw in the definitions, not in the primitives.

Decades of work have been built on the current foundations, and changing them may not be worth it, or even possible (both in a social and a mathematical sense).
What I take away is that these quirks are worth understanding.
They decide what a security theorem actually says, and that is easy to lose sight of when the proof checks.

## AI Usage

I used Claude Opus 5.5 on high effort to format the text and references and to proofread this.
Since I am someone who thinks better by talking, I also used the chat function as a bit of a back and forth to realize my idea.

## References

- [AB09] S. Arora, B. Barak. Computational Complexity: A Modern Approach. Cambridge University Press, 2009.
- [ABB+19] J. B. Almeida, C. Baritel-Ruet, M. Barbosa, G. Barthe, F. Dupressoir, B. Grégoire, V. Laporte, T. Oliveira, A. Stoughton, P.-Y. Strub. Machine-Checked Proofs for Cryptographic Standards: Indifferentiability of Sponge and Secure High-Assurance Implementations of SHA-3. CCS 2019.
- [BDJR97] M. Bellare, A. Desai, E. Jokipii, P. Rogaway. A Concrete Security Treatment of Symmetric Encryption. FOCS 1997.
- [BDPR98] M. Bellare, A. Desai, D. Pointcheval, P. Rogaway. Relations Among Notions of Security for Public-Key Encryption Schemes. CRYPTO 1998.
- [BGHZ11] G. Barthe, B. Grégoire, S. Heraud, S. Zanella-Béguelin. Computer-Aided Security Proofs for the Working Cryptographer. CRYPTO 2011.
- [BKP18] N. Bitansky, Y. T. Kalai, O. Paneth. Multi-Collision Resistance: A Paradigm for Keyless Hash Functions. STOC 2018.
- [BKR00] M. Bellare, J. Kilian, P. Rogaway. The Security of the Cipher Block Chaining Message Authentication Code. Journal of Computer and System Sciences 61(3), 2000. (Preliminary version at CRYPTO 1994.)
- [BL13] D. J. Bernstein, T. Lange. Non-uniform Cracks in the Concrete: The Power of Free Precomputation. ASIACRYPT 2013.
- [CDG18] S. Coretti, Y. Dodis, S. Guo. Non-Uniform Bounds in the Random-Permutation, Ideal-Cipher, and Generic-Group Models. CRYPTO 2018.
- [Dam87] I. Damgård. Collision Free Hash Functions and Public Key Signature Schemes. EUROCRYPT 1987.
- [DGK17] Y. Dodis, S. Guo, J. Katz. Fixing Cracks in the Concrete: Random Oracles with Auxiliary Input, Revisited. EUROCRYPT 2017.
- [GB08] S. Goldwasser, M. Bellare. Lecture Notes on Cryptography. 2008.
- [GM84] S. Goldwasser, S. Micali. Probabilistic Encryption. Journal of Computer and System Sciences 28(2), 1984.
- [Gol01] O. Goldreich. Foundations of Cryptography, Volume 1: Basic Tools. Cambridge University Press, 2001.
- [HR04] C.-Y. Hsiao, L. Reyzin. Finding Collisions on a Public Road, or Do Secure Hash Functions Need Secret Coins? CRYPTO 2004.
- [IR89] R. Impagliazzo, S. Rudich. Limits on the Provable Consequences of One-Way Permutations. STOC 1989.
- [KL80] R. M. Karp, R. J. Lipton. Some Connections between Nonuniform and Uniform Complexity Classes. STOC 1980.
- [KL20] J. Katz, Y. Lindell. Introduction to Modern Cryptography, 3rd edition. CRC Press, 2020.
- [RaSi91] C. Rackoff, D. R. Simon. Non-Interactive Zero-Knowledge Proof of Knowledge and Chosen Ciphertext Attack. CRYPTO 1991.
- [Rog06] P. Rogaway. Formalizing Human Ignorance: Collision-Resistant Hashing without the Keys. VIETCRYPT 2006.
- [RoSh04] P. Rogaway, T. Shrimpton. Cryptographic Hash-Function Basics: Definitions, Implications, and Separations for Preimage Resistance, Second-Preimage Resistance, and Collision Resistance. FSE 2004.
- [RTV04] O. Reingold, L. Trevisan, S. Vadhan. Notions of Reducibility between Cryptographic Primitives. TCC 2004.
- [Sim98] D. R. Simon. Finding Collisions on a One-Way Street: Can Secure Hash Functions Be Based on General Assumptions? EUROCRYPT 1998.
- [Unr07] D. Unruh. Random Oracles and Auxiliary Input. CRYPTO 2007.
