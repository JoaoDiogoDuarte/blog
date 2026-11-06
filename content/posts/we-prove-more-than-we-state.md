---
title: "We (sometimes) prove more than we state, concretely"
date: 2026-10-26
slug: we-prove-more-than-we-state
---

This post is a sort of an appendix to my post on [collision resistance and quantifiers](https://diogoduarte.pt/blog/collision-resistance-quantifiers/), where Rogaway's explicit reductions came up as one of the ways around unkeyed hash functions.

I was reading Rogaway's Formalizing Human Ignorance [Rog06], and on the second page, in the contributions paragraph, they say

> [T]he theorem statement will say that there is an explicitly given reduction: given an adversary $\mathcal{A}$ against $\Pi$ there is a corresponding, explicitly-specified adversary $\mathcal{C}$, as efficient as $\mathcal{A}$, for finding collisions in $H$.

<!--more-->

Wait a second, isn't this what we always do?
Every reduction I have written builds the adversary against the primitive step by step, out of the adversary against the scheme.
If we back up a little, they state that

> Conventionally, a theorem would be given to capture the idea that the existence of an effective adversary $\mathcal{A}$ against $\Pi$ implies the existence of an effective adversary $\mathcal{C}$ against $H$ …

Read quickly, these two sentences seem to say the same thing, and for a long time I would have read them as equivalent (sometimes I still do if I am not paying attention).
They are not, in fact, the same.
The "conventional" theorem **statement** only says that $\mathcal{C}$ exists, while Rogaway's specifies the code of this $\mathcal{C}$.
Naturally, this then becomes a witness to the existential in the conventional statement.

I think I kept conflating both statements because almost every proof I have read builds its reduction explicitly, so I got used to reading every theorem as if it did too.
The proof builds $\mathcal{C}$ step by step, the theorem only says one exists, and in my head the two collapse into one.
As we will see, the conventional statement actually claims much less than what I expected, which is exactly why Rogaway bothers to make this distinction.

## Theorem statements

Rogaway distinguishes three ways of stating a security result for a protocol $\Pi$ built from any hash function $H$ [Rog06, Sec. 4].
Throughout, I fix the construction $\Pi$, write $\Pi^H$ for $\Pi$ built from $H$, and assume that $\Pi^H$ and $H$ are efficiently computable.
I write $\mathsf{G}$ for the security game of $\Pi$ and $\mathsf{Coll}$ for the collision game of $H$.

All three statements quantify over the hash function, and I will write this $\forall H$ explicitly.
We can read $\forall H$ as "for any hash function $H$", so once a statement is proved, we can instantiate it with the hash function we actually use, say SHA3-256.
If we only proved bounds for one specific $H$, then wherever the statement says that some collision finder exists, the one that prints a hard-coded collision for that $H$ would do, and the statement would say nothing.

Note that we assume that the reduction always has black-box access to $H$.

### Existential (C0)

Rogaway's prose version is the following.

> **C0.** If there's an effective algorithm $\mathcal{A}$ for attacking protocol $\Pi$ then there's an effective algorithm $\mathcal{C}$ for finding collisions in $H$.

Mathematically, I interpret it like this.

$$
\forall  \text{ efficient } H,\quad
\big(\exists \text{ efficient } \mathcal{A},\ \Pr[\mathsf{G}^{\mathcal{A}} \Rightarrow 1] \text{ large}\big)
\implies
\big(\exists \text{ efficient } \mathcal{C},\ \Pr[\mathsf{Coll}^{\mathcal{C}} \Rightarrow 1] \text{ large}\big)
$$

As alluded to in my previous post, this says nothing for an unkeyed $H$, since $\mathcal{C}$ is chosen after $H$, and the collision finder that just prints a hard-coded collision for that $H$ always exists and wins with probability $1$, so the conclusion is always true.
Putting $\forall H$ in front does not help, since the printer is allowed to depend on $H$.
Rogaway notes that this is how results have traditionally been stated, and that the proof is usually constructive even when the statement is not.

They note that a neat example of this occurrence is via Stinson [Sti06].

> **Theorem 4.1.** Let $f \colon X \to Y$ be any hash function. If there exists an $(\varepsilon, q)$ algorithm that solves Second preimage for the hash function $f$, there exists an $(\varepsilon, q)$ algorithm that solves Collision for the hash function $f$.

The proof is the sentence just before the theorem, which says to choose $x \in X$ at random and run the second-preimage finder on it.
So the proof is completely explicit, but the statement only says that some collision finder exists.

### Code-constructive (C1)

> **C1.** If you know an effective algorithm $\mathcal{A}$ for attacking protocol $\Pi$ then you know an effective algorithm $\mathcal{C}$ for finding collisions in $H$.

The change of wording from "exists" to "knowing" reflects the fact that we need to know the constructions of both adversaries.
For example, we define an **explicit** collision finder $\mathcal{C}$ (i.e. we actually need to write down its code) that gets the code $\langle \mathcal{A} \rangle$ of the adversary.
I also require the resulting collision finder to be efficient whenever $\mathcal{A}$ is, with the overhead of the reduction accounted for.
I therefore interpret Rogaway's statement as follows.

$$
\exists \text{ efficient } \mathcal{C},\,\,\forall  \text{ efficient } H,\,\,\forall \text{ efficient } \mathcal{A},\quad
\Pr[\mathsf{G}^{\mathcal{A}} \Rightarrow 1] \text{ large}
\implies
\Pr[\mathsf{Coll}^{\mathcal{C}(\langle \mathcal{A} \rangle)^{H}} \Rightarrow 1] \text{ large}
$$

We provide the $\mathcal{C}$ that we explicitly construct to prove this statement.
Note that we fix the reduction before the hash function and the adversary, so it can only describe one collision finder that works for every $H$ and every efficient $\mathcal{A}$, and in particular it cannot carry a collision for any particular $H$.
If we swapped quantifiers, or moved it after the implication, then there could be one reduction per hash function or per adversary, such as the printer for that $H$, which we do not want.

Note that this is a **non black-box** reduction in the adversary since we have access to its code.

### Black-box-constructive (C2)

> **C2.** If you possess effective means $\mathcal{A}$ for attacking protocol $\Pi$ then you possess effective means $\mathcal{C}$ for finding collisions in $H$.

$$
\exists \text{ efficient } \mathcal{C},\,\,\forall  \text{ efficient }  H,\,\,\forall \mathcal{A},\quad
\Pr[\mathsf{G}^{\mathcal{A}} \Rightarrow 1] \text{ large}
\implies
\Pr[\mathsf{Coll}^{\mathcal{C}^{\mathcal{A}, H}} \Rightarrow 1] \text{ large}
$$

C2 is C1 where $\mathcal{C}$ is not allowed to "peek" into $\mathcal{A}$, it can only run it.
The advantage bound can hold for every $\mathcal{A}$, efficient or not, while a separate running-time bound accounts for the cost of its calls to $\mathcal{A}$.
Rogaway's motivation is that you might own an attack without knowing how it works, such as a tamper-proof device or a big binary, and C2 says that is enough.
Clearly, C2 is the stronger statement, and $\text{C2} \implies \text{C1} \implies \text{C0}$.

Interestingly enough, Stinson's paper has a statement of this kind as well.
Stinson's Algorithm 4.1 turns a preimage finder into a collision finder, calling it only as a subroutine.

> **Corollary 4.4.** Let $f \colon X \to Y$ be a hash function, and suppose that FindPreimage is an $(\varepsilon, q)$ algorithm that solves Preimage for the hash function $f$ with probability $\varepsilon$ for every $y \in Y$. Then Algorithm 4.1 is an $(\varepsilon', q + 2)$ algorithm that solves Collision for the hash function $f$, where $\varepsilon' = \varepsilon (1 - |Y|/|X|)$.

Here the statement names the reduction instead of just asserting that some collision finder exists.
Theorem 4.1 can be rewritten the same way and we do not have to change its proof!

### Why only constructing the reduction helps

What makes C1 and C2 meaningful is the order of $\exists \mathcal{C}$ and $\forall H$, together with the fact that $\mathcal{C}$ is a specific algorithm defined in the proof, with a proved guarantee about what it does with $\mathcal{A}$.
In C1 and C2 the reduction is fixed before the hash function, so it has to turn every attack into a collision for every $H$, and it cannot just carry an answer, while in C0 the collision finder is chosen after $H$, so the trivial hardwired adversary is a perfectly valid adversary.

Rogaway also asks for the reduction to be given explicitly in the proof, so that the statement can be read as being about what people know.
That is what they mean by formalising human ignorance, since the statement tells us how to get a collision from an attack, while whether anyone can find one is left to what we know.

### Which $\varepsilon$ do we believe?

In practice we use C0 and C2 by plugging in a number $\varepsilon$ that we believe bounds the advantage of collision finders running in time $t'$ (Appendix A writes this out).
In C0, $\varepsilon$ has to bound every such collision finder, printer included, so for an unkeyed $H$ the only true bound is $\varepsilon = 1$.
In C2 (and C1), it only has to bound $\mathcal{C}^{\mathcal{A}}$, which can be built whenever $\mathcal{A}$ can.
Better collision finders may exist, the printer among them, but the theorem never mentions them, so we can use what we actually believe for SHA3-256, namely that nobody can *build* a collision finder in time $t'$ with advantage more than $\varepsilon$.

In that sense, $\mathcal{C}^{\mathcal{A}}$ is a lower bound that we never get to use.
If someone could build a good $\mathcal{A}$ against $\Pi^H$, they could build $\mathcal{C}^{\mathcal{A}}$ too, which we believe nobody can, so the reduction shows not that no good adversary exists, but that we cannot build one.
That $\varepsilon$ is itself not proved, though, and for SHA3-256 it comes from the cost of the best known attacks, so the explicit reduction does not remove the conjecture, it only makes it a conjecture about what people can build.

## Mixing the models

There is one more distinction that I think is easy to lose sight of (I often do).
When we say that a *security proof* is uniform or non-uniform, which part of the proof are we actually talking about?

A typical reductionist proof has three moving parts: an adversary $\mathcal{A}$ against a construction $\Pi$, a reduction $\mathcal{R}$ that uses $\mathcal{A}$ to attack some underlying primitive $F$, and an assumption that $F$ is hard to break.
Informally, we prove

$$
\mathcal{A} \text{ breaks } \Pi
\quad\Longrightarrow\quad
\mathcal{R}^{\mathcal{A}} \text{ breaks } F.
$$

Then we use the assumed security of $F$ to conclude that $\Pi$ is secure.
But there are really two questions here: what kind of adversary $\mathcal{A}$ are we trying to rule out, and what kind of adversary against $F$ does the reduction produce?

### Uniform reductions and non-uniform adversaries

Suppose $\mathcal{R}$ is a uniform reduction, meaning that it is one fixed efficient algorithm that works for every security parameter.
If $\mathcal{A}$ is uniform, then $\mathcal{R}^{\mathcal{A}}$ is also uniform.
So uniform security of $F$ suffices to prove uniform security of $\Pi$.

But what if $\mathcal{A}$ is non-uniform?
Nothing stops us from using the same reduction.
At each length $\ell$, it runs the corresponding circuit $\mathcal{A}_\ell$, producing another circuit that attacks $F$.
The resulting family is generally non-uniform, even though $\mathcal{R}$ itself is uniform.
Therefore, assuming the reduction has polynomial overhead and preserves non-negligible advantage, it gives us both statements:

$$
\begin{array}{l}
F \text{ uniformly secure}
\implies \Pi \text{ uniformly secure} \\[6pt]
F \text{ non-uniformly secure}
\implies \Pi \text{ non-uniformly secure}.
\end{array}
$$

Notice what we **cannot** conclude: uniform security of $F$ does not, by itself, give non-uniform security of $\Pi$.
This is consistent with non-uniform security implying uniform security.
The non-uniform assumption is stronger because it rules out more adversaries.
We cannot assume only the weaker uniform hardness of $F$ and somehow obtain the stronger conclusion about $\Pi$.

### When the reduction itself needs advice

Now suppose the reduction contains a non-constructive step.
For example, a proof might show that some choice of random coins makes the reduction successful, and then fix those coins.
If we cannot efficiently find a good choice, the reduction may need to hard-code a different one for every security parameter.
Even if the original $\mathcal{A}$ is uniform, the resulting adversary against $F$ may now be non-uniform.
So our proof may establish only

$$
F \text{ non-uniformly secure}
\implies \Pi \text{ uniformly secure}.
$$

That is a perfectly valid theorem, but it is not the same theorem we would get from a uniform reduction!
This is the central point of Koblitz and Menezes's *Another Look at Non-Uniformity* [KM13a].
They argue that non-uniform reductions can be undesirable for practice-oriented security proofs, not because the mathematics is wrong, but because the **hypothesis becomes stronger too**.

At first, proving security against non-uniform adversaries looks like a better result: after all, these adversaries are more powerful.
But a reductionist theorem is conditional.
To rule out more powerful adversaries against our construction, we might also need to assume that the underlying primitive resists more powerful adversaries.
Worse, even proving only uniform security of the construction may require that stronger assumption if the reduction itself is non-uniform.

This matters when we try to interpret the theorem concretely.
Koblitz and Menezes discuss the security of NMAC/HMAC, where a coin-fixing argument leads to a non-uniform adversary against the underlying compression function [KM13a].
They describe a distinguisher that hard-codes a favourable message and test, selected using potentially enormous preprocessing, but then needs very little work to distinguish the compression function from a random function.

Such an adversary may have a much better distinguishing advantage than the best known uniform attacks.
If we estimate the theorem's concrete security by assuming that the underlying primitive is as hard to distinguish as exhaustive key search, we may be using a bound that does not hold for the adversaries the proof actually requires us to rule out.
The reduction can be correct, and yet the practical interpretation of its hypothesis can be wrong.

This is not to say that non-uniform security should not be used.
It is simply a stronger assumption, and we should be careful about what evidence supports it.

## Closing thoughts

Is any of this a real problem today?
For the proofs, not really, since almost every reduction we write is explicit and black-box, and I think many papers now state their theorems with the reduction named.
For the statements, it is a reminder (at least for me) to read a theorem for what it says and not for what its proof does, especially when it mentions a single unkeyed hash function.

That said, a proof can be explicit almost everywhere and still contain a non-constructive step.
Coin-fixing is one example: the resulting reduction may need advice that we do not know how to produce, so the argument about which $\varepsilon$ we believe can break down.
As we saw above, this is also why we need to be careful about which hardness assumption a proof actually uses.

After the references, you can find some appendices about how these statements are proved, directly with and without an explicit reduction and by contrapositive, and how Rogaway's forms relate to other formal notions of reductions.

## AI Usage

I used Claude Opus 5.5 on high effort to format the text and references and to proofread this.
Since I am someone who thinks better by talking, I also used the chat function as a bit of a back and forth to develop my ideas.

## References

- [BKR00] M. Bellare, J. Kilian, P. Rogaway. The Security of the Cipher Block Chaining Message Authentication Code. Journal of Computer and System Sciences 61(3), pp. 362–399, 2000. (Preliminary version at CRYPTO 1994.) Available at <https://doi.org/10.1006/jcss.1999.1694>.
- [HR04] C.-Y. Hsiao, L. Reyzin. Finding Collisions on a Public Road, or Do Secure Hash Functions Need Secret Coins? CRYPTO 2004, LNCS 3152, pp. 92–105. Available at <https://doi.org/10.1007/978-3-540-28628-8_6>.
- [IR89] R. Impagliazzo, S. Rudich. Limits on the Provable Consequences of One-Way Permutations. STOC 1989, pp. 44–61. Available at <https://doi.org/10.1145/73007.73012>.
- [Rog06] P. Rogaway. Formalizing Human Ignorance: Collision-Resistant Hashing without the Keys. VIETCRYPT 2006, LNCS 4341, pp. 211–228. Available at <https://doi.org/10.1007/11958239_14>.
- [RTV04] O. Reingold, L. Trevisan, S. Vadhan. Notions of Reducibility between Cryptographic Primitives. TCC 2004, LNCS 2951, pp. 1–20. Available at <https://doi.org/10.1007/978-3-540-24638-1_1>.
- [Rud76] W. Rudin. Principles of Mathematical Analysis, 3rd edition. McGraw-Hill, 1976.
- [Sim98] D. R. Simon. Finding Collisions on a One-Way Street: Can Secure Hash Functions Be Based on General Assumptions? EUROCRYPT 1998, LNCS 1403, pp. 334–345. Available at <https://doi.org/10.1007/BFb0054137>.
- [Sti06] D. R. Stinson. Some Observations on the Theory of Cryptographic Hash Functions. Designs, Codes and Cryptography 38, pp. 259–277, 2006. Preliminary version as IACR ePrint 2001/020. Available at <https://doi.org/10.1007/s10623-005-6344-y>.
- [Vel19] D. J. Velleman. How to Prove It: A Structured Approach, 3rd edition. Cambridge University Press, 2019. Available at <https://doi.org/10.1017/9781108539890>.

## Appendix A, proving these statements

For whoever wants to see where exactly the explicit reduction gets lost, here is how one might prove the conventional statement, first without and then with an explicit reduction, both directly and by contrapositive.
Everything below works the same for every $H$, so fix an arbitrary one.
Write $\mathsf{Adv}_\Pi(\mathcal{A})$ for $\Pr[\mathsf{G}^{\mathcal{A}} \Rightarrow 1]$ and $\mathsf{Adv}_H(\mathcal{C})$ for $\Pr[\mathsf{Coll}^{\mathcal{C}} \Rightarrow 1]$.
In the concrete setting, "efficient" means running in time at most $t$ for adversaries against $\Pi^H$, and at most $t'$ for collision finders.
I write $\mathcal{C}'$ for a generic collision finder, and keep $\mathcal{C}$ for the reduction.

The old way of writing concrete results uses the best advantage over all adversaries with given resources, which Bellare, Kilian and Rogaway call the **insecurity** [BKR00], and which I write as $\mathsf{InSec}$.

$$
\mathsf{InSec}_\Pi(t) = \max_{\mathcal{A} \text{ in time } \le t} \mathsf{Adv}_\Pi(\mathcal{A}),
\qquad
\mathsf{InSec}_H(t') = \max_{\mathcal{C}' \text{ in time } \le t'} \mathsf{Adv}_H(\mathcal{C}')
$$

All the flips between maxima, quantifiers and negations below come from a handful of facts that are useful to spell out.

$$
\begin{array}{l}
\max_x a(x) \le \varepsilon \iff \forall x,\ a(x) \le \varepsilon \\[4pt]
\max_x a(x) > \varepsilon \iff \exists x,\ a(x) > \varepsilon \\[4pt]
\neg\big(\forall x,\ P(x)\big) \iff \exists x,\ \neg P(x) \\[4pt]
\neg (a \le \varepsilon) \iff a > \varepsilon \\[4pt]
(P \implies Q) \iff (\neg Q \implies \neg P)
\end{array}
$$

The first two follow from the definition of a maximum, which over finitely many adversaries is their least upper bound [Rud76, Defs. 1.7 and 1.8], and the fourth is just the ordering of the real numbers [Rud76, Def. 1.5].
The third and the fifth are the quantifier negation law and the contrapositive law [Vel19, Secs. 2.2 and 1.5].

In particular, "$H$ is collision resistant", meaning that every collision finder in time $t'$ has advantage at most $\varepsilon$, is the same as $\mathsf{InSec}_H(t') \le \varepsilon$.

### A direct proof without a reduction

Written with insecurities, the conventional statement is the following.

$$
\mathsf{InSec}_\Pi(t) \le \mathsf{InSec}_H(t')
$$

Unfolding both maxima, it says that every adversary in time $t$ is matched by some collision finder in time $t'$ that does at least as well.

$$
\forall \mathcal{A} \text{ in time } \le t,\ \ \exists \mathcal{C}' \text{ in time } \le t',\quad
\mathsf{Adv}_\Pi(\mathcal{A}) \le \mathsf{Adv}_H(\mathcal{C}')
$$

So a direct proof only has to show that such a $\mathcal{C}'$ exists for every $\mathcal{A}$, and nothing forces it to be built from $\mathcal{A}$, or to be the same procedure for every $\mathcal{A}$.
In practice the matching collision finder is almost always built from $\mathcal{A}$, but for an unkeyed $H$ there is always a trivial proof.
As long as $t'$ allows writing down a collision, the printer has advantage $1$, so $\mathsf{InSec}_H(t') = 1$, and the inequality holds for every $\Pi$ without looking at $\Pi$ at all.
So the conventional statement can be proved without any reduction, which is another way of saying that it is vacuous.

### A direct proof with an explicit reduction

Now suppose we have an explicit reduction $\mathcal{C}$, fixed before $H$ as in C2, and with no loss in advantage to keep the notation light.
It turns every adversary in time $t$ into a collision finder $\mathcal{C}^{\mathcal{A}}$ in time $t'$.

$$
\text{(R)} \qquad \forall \mathcal{A} \text{ in time } \le t,\quad \mathsf{Adv}_\Pi(\mathcal{A}) \le \mathsf{Adv}_H(\mathcal{C}^{\mathcal{A}})
$$

Bounding the right-hand side by the best collision finder, and then taking the maximum on the left, gives the conventional statement back.

$$
\begin{array}{l}
\forall \mathcal{A} \text{ in time } \le t,\quad \mathsf{Adv}_\Pi(\mathcal{A}) \le \mathsf{Adv}_H(\mathcal{C}^{\mathcal{A}}) \le \mathsf{InSec}_H(t') \\[4pt]
\quad \implies \quad \mathsf{InSec}_\Pi(t) \le \mathsf{InSec}_H(t')
\end{array}
$$

The argument is fine, but the conclusion is just as vacuous as before, because the step $\mathsf{Adv}_H(\mathcal{C}^{\mathcal{A}}) \le \mathsf{InSec}_H(t')$ forgets which collision finder we had, and the maximum includes the printer.

The usual way of writing it plugs in the assumption that $H$ is collision resistant instead.

$$
\begin{array}{l}
\text{(R)} \quad + \quad \forall \mathcal{C}' \text{ in time } \le t',\ \mathsf{Adv}_H(\mathcal{C}') \le \varepsilon \\[4pt]
\quad \implies \quad \forall \mathcal{A} \text{ in time } \le t,\ \mathsf{Adv}_\Pi(\mathcal{A}) \le \mathsf{Adv}_H(\mathcal{C}^{\mathcal{A}}) \le \varepsilon \\[4pt]
\quad \implies \quad \forall \mathcal{A} \text{ in time } \le t,\ \mathsf{Adv}_\Pi(\mathcal{A}) \le \varepsilon
\end{array}
$$

By the first flip above, applied on both sides, this proves $\mathsf{InSec}_H(t') \le \varepsilon \implies \mathsf{InSec}_\Pi(t) \le \varepsilon$.
Collision resistance is only ever applied to the one collision finder $\mathcal{C}^{\mathcal{A}}$, which is why the reduction has to stay within time $t'$.
For an unkeyed $H$ the assumption is false, since $\mathsf{InSec}_H(t') = 1$, so the implication holds for free and again says nothing about $\Pi$.
The vacuity is not in the argument, it is in what we chose to conclude.

### A proof by contrapositive

Negating both sides of the last implication, with the flips above, gives its contrapositive.

$$
\begin{array}{l}
\neg\big(\forall \mathcal{A},\ \mathsf{Adv}_\Pi(\mathcal{A}) \le \varepsilon\big) \iff \exists \mathcal{A},\ \mathsf{Adv}_\Pi(\mathcal{A}) > \varepsilon \iff \mathsf{InSec}_\Pi(t) > \varepsilon \\[4pt]
\neg\big(\forall \mathcal{C}',\ \mathsf{Adv}_H(\mathcal{C}') \le \varepsilon\big) \iff \exists \mathcal{C}',\ \mathsf{Adv}_H(\mathcal{C}') > \varepsilon \iff \mathsf{InSec}_H(t') > \varepsilon
\end{array}
$$

Here $\mathcal{A}$ ranges over adversaries in time $t$ and $\mathcal{C}'$ over collision finders in time $t'$, as before.
So proving $\mathsf{InSec}_H(t') \le \varepsilon \implies \mathsf{InSec}_\Pi(t) \le \varepsilon$ is the same as proving $\mathsf{InSec}_\Pi(t) > \varepsilon \implies \mathsf{InSec}_H(t') > \varepsilon$, which is the textbook "suppose $\Pi$ is broken" proof.

$$
\begin{array}{l}
\text{(R)} \quad + \quad \exists \mathcal{A}_0 \text{ in time } \le t,\ \mathsf{Adv}_\Pi(\mathcal{A}_0) > \varepsilon \\[4pt]
\quad \implies \quad \mathsf{Adv}_H(\mathcal{C}^{\mathcal{A}_0}) \ge \mathsf{Adv}_\Pi(\mathcal{A}_0) > \varepsilon \\[4pt]
\quad \implies \quad \exists \mathcal{C}' \text{ in time } \le t',\ \mathsf{Adv}_H(\mathcal{C}') > \varepsilon
\end{array}
$$

The last step is where the witness is forgotten, since the conclusion only remembers that some $\mathcal{C}'$ exists, and for an unkeyed $H$ that conclusion is true anyway.
Both versions drop $\mathcal{C}^{\mathcal{A}}$, and the direct one just hides it better, because there $\mathcal{C}^{\mathcal{A}}$ only shows up inside the proof.

### What the proof actually shows

What both proofs really establish is the version that keeps the witness for every $\mathcal{A}$.

$$
\forall \mathcal{A} \text{ in time } \le t,\quad \mathsf{Adv}_\Pi(\mathcal{A}) > \varepsilon \implies \mathsf{Adv}_H(\mathcal{C}^{\mathcal{A}}) > \varepsilon
$$

Since the proof works for an arbitrary $H$ with the same $\mathcal{C}$, this holds for every $H$, which is exactly C2 with $\varepsilon$ in place of "large".
For an unkeyed $H$, everything else in this appendix is valid but vacuous, and this is the only statement that still says something about $\Pi$.

## Appendix B, Rogaway's forms and notions of reducibility

For whoever is interested in how Rogaway's forms compare to other formal ways of talking about reductions, here is how they relate to the notions of Reingold, Trevisan and Vadhan.
Unlike in the main text, I first drop the efficiency assumption, so adversaries only get access to $H$ through an oracle, and C0 to C2 are the versions from the main text with that oracle added, for instance C0 concludes with $\Pr[\mathsf{Coll}^{\mathcal{C}^{H}} \Rightarrow 1]$ large.
The construction and the reduction are still required to be efficient as oracle algorithms.

Reingold, Trevisan and Vadhan classify reductions between cryptographic primitives [RTV04].
Their aim is different from Rogaway's, since they want precise notions to say exactly which kind of reduction a black-box separation such as [IR89, Sim98, HR04] rules out.
Their setting is also asymptotic, and their notions quantify over every implementation $f$ of a primitive, with one construction that has to work for all of them.
In their strongest notion, a fully black-box reduction, there is an algorithm that converts every adversary breaking the construction into one that breaks $f$, using both the adversary and $f$ only as oracles.
Their weaker notions, semi-black-box and weakly black-box, only ask that an efficient attack on $f$ exists whenever an efficient attack on the construction does, and differ in whether that attack on the construction gets oracle access to $f$.

In our notation, with $\Pi^f$ a construction that uses its primitive $f$ only as an oracle, the security clauses of their three definitions are the following, with the correctness clauses assumed [RTV04, Defs. 3 to 5].

$$
\begin{array}{ll}
\textbf{fully-BB:} & \exists \Pi\ \exists \mathcal{C}\ \ \forall f\ \forall \mathcal{A},\quad \mathcal{A} \text{ breaks } \Pi^f \implies \mathcal{C}^{\mathcal{A}, f} \text{ breaks } f \\[4pt]
\textbf{semi-BB:} & \exists \Pi\ \ \forall f,\quad \big(\exists \text{ efficient } \mathcal{A},\ \mathcal{A}^{f} \text{ breaks } \Pi^f\big) \implies \big(\exists \text{ efficient } \mathcal{C},\ \mathcal{C}^{f} \text{ breaks } f\big) \\[4pt]
\textbf{weakly-BB:} & \exists \Pi\ \ \forall f,\quad \big(\exists \text{ efficient } \mathcal{A},\ \mathcal{A} \text{ breaks } \Pi^f\big) \implies \big(\exists \text{ efficient } \mathcal{C},\ \mathcal{C}^{f} \text{ breaks } f\big)
\end{array}
$$

Let us now relate them to C0, C1 and C2 step by step.

**Step 1, fixing the construction.**
We only care about one construction, so we fix $\Pi$, which removes $\exists \Pi$.
We keep $\forall f$ though, and write it as $\forall H$, as in the main text.
Writing "breaks" as a large advantage gives the following.

$$
\begin{array}{ll}
\textbf{fully-BB:} & \exists \mathcal{C}\ \ \forall H\ \forall \mathcal{A},\quad \Pr[\mathsf{G}^{\mathcal{A}} \Rightarrow 1] \text{ large} \implies \Pr[\mathsf{Coll}^{\mathcal{C}^{\mathcal{A}, H}} \Rightarrow 1] \text{ large} \\[4pt]
\textbf{semi-BB:} & \forall H,\quad \big(\exists \text{ efficient } \mathcal{A},\ \Pr[\mathsf{G}^{\mathcal{A}^{H}} \Rightarrow 1] \text{ large}\big) \implies \big(\exists \text{ efficient } \mathcal{C},\ \Pr[\mathsf{Coll}^{\mathcal{C}^{H}} \Rightarrow 1] \text{ large}\big) \\[4pt]
\textbf{weakly-BB:} & \forall H,\quad \big(\exists \text{ efficient } \mathcal{A},\ \Pr[\mathsf{G}^{\mathcal{A}} \Rightarrow 1] \text{ large}\big) \implies \big(\exists \text{ efficient } \mathcal{C},\ \Pr[\mathsf{Coll}^{\mathcal{C}^{H}} \Rightarrow 1] \text{ large}\big)
\end{array}
$$

Keeping $\forall H$ after $\exists \mathcal{C}$ is what stops the reduction in fully-BB from carrying anything specific to $H$, exactly as in C2.
If we fixed $H$ as well, the printer for that $H$ would satisfy all three.

**Step 2, comparing with C0 and C2.**
Weakly-BB is literally C0, with the same premise and the same conclusion.
Fully-BB is literally C2, with $\mathcal{C}$ fixed before $H$ and running $\mathcal{A}$ and $H$ as oracles.
Semi-BB only differs from C0 in that the adversary against $\Pi^H$ may also query $H$, and it implies C0, since an adversary that does not use $H$ is just an $\mathcal{A}^{H}$ that never queries its oracle.
The extra oracle access matters in RTV's general setting, but for a compressing $H$, both existential conclusions already hold because a collision can be hard-coded.
So without any efficiency assumption, $\text{semi} \implies \text{C0} \iff \text{weakly}$, while $\text{fully-BB} \iff \text{C2}$.

**Step 3, assuming the scheme is efficient.**
Now assume that $\Pi^H$ and $H$ are efficiently computable, as in the main text.
In semi-BB, weakly-BB and C0 the adversaries are chosen after $H$, so any efficient $\mathcal{A}^{H}$ can be replaced by an adversary $\mathcal{A}'$ that evaluates $H$ itself, with the same advantage and only a little more running time, and the same goes for $\mathcal{C}^{H}$.

$$
\begin{array}{l}
\big(\exists \text{ efficient } \mathcal{A},\ \Pr[\mathsf{G}^{\mathcal{A}^{H}} \Rightarrow 1] \text{ large}\big) \iff \big(\exists \text{ efficient } \mathcal{A}',\ \Pr[\mathsf{G}^{\mathcal{A}'} \Rightarrow 1] \text{ large}\big) \\[4pt]
\big(\exists \text{ efficient } \mathcal{C},\ \Pr[\mathsf{Coll}^{\mathcal{C}^{H}} \Rightarrow 1] \text{ large}\big) \iff \big(\exists \text{ efficient } \mathcal{C}',\ \Pr[\mathsf{Coll}^{\mathcal{C}'} \Rightarrow 1] \text{ large}\big)
\end{array}
$$

So the premises of semi-BB, weakly-BB and C0 coincide, and so do their conclusions, which makes them the same statement.
In fully-BB and C2 the oracle for $H$ stays, since there $\mathcal{C}$ is fixed before $H$ and needs some way to evaluate it.

**Step 4, where C1 fits.**
C2 implies C1, since a collision finder that gets $\langle \mathcal{A} \rangle$ and $\langle H \rangle$ as input can simply run them and do exactly what $\mathcal{C}^{\mathcal{A}, H}$ does.
C1 implies C0, since $\exists \mathcal{C}\ \forall H$ implies $\forall H\ \exists \mathcal{C}$ [Vel19, Ch. 2], so for every $H$ the collision finder $\mathcal{C}(\langle \mathcal{A} \rangle, \langle H \rangle)$ shows that one exists.
Putting it all together, for an efficient scheme we get the following.

$$
\text{C2} \iff \text{fully-BB} \implies \text{C1} \implies \text{C0} \iff \text{semi-BB} \iff \text{weakly-BB}
$$

For a compressing $H$, everything from C0 onwards is vacuous, so only fully-BB and C2, where the reduction comes before $H$, say something about the scheme.
