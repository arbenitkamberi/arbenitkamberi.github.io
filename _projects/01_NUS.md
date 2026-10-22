---
layout: page
title: "Verified LTL Model Checking"
description: Research internship at the National University of Singapore. Lasso, a formally verified alternative to the TLA+ model checker that matches it on performance.
img: assets/img/NUS.jpg
importance: 1
category: work
related_publications: false
---

A research internship at the [VERSE Lab](https://verse-lab.github.io/),
National University of Singapore, advised by Prof. Dr. Ilya Sergey. The result
is **Lasso**, a formally verified
alternative to TLC, the model checker behind
[TLA<sup>+</sup>](https://lamport.azurewebsites.net/tla/tla.html): a model checker for
Linear Temporal Logic implemented *and proved correct* inside the
[Lean](https://lean-lang.org/) proof assistant, and built from the start to
match TLC on performance. No
previously verified LTL checker had managed that, and the one prior effort runs
about an order of magnitude slower than the heavily optimized but unverified tools. 
A paper on the work is currently under submission.

**Why this is hard.** The interesting properties of distributed protocols are
liveness properties ("something good eventually happens"), and they almost never
hold outright: they hold only under *fairness* assumptions that rule out runs
starving some component forever. The textbook recipe translates the whole
formula, fairness included, into a Büchi automaton. 
Fairness assumptions cause that translation to explode exponentially: 
a conjunction of $$n$$ strong-fairness assumptions is a
conjunction of $$n$$ disjunctions, so the translator faces $$2^n$$ possible combinations.

$$
\begin{aligned}
\phi_{\mathit{fair}}
  &= \bigwedge_{i<n} \texttt{SF}(\mathtt{e}_i, \mathtt{t}_i) \\
  &\equiv \bigwedge_{i<n} \big( \mathbf{G}\mathbf{F}\, \mathtt{e}_i \rightarrow \mathbf{G}\mathbf{F}\, \mathtt{t}_i \big) \\
  &\equiv \bigwedge_{i<n} \big( \mathbf{F}\mathbf{G}\, \neg \mathtt{e}_i \vee \mathbf{G}\mathbf{F}\, \mathtt{t}_i \big)
\end{aligned}
$$

On one of my benchmark families the standard translation produces 356 343
automaton states and 12.8 million transitions at $$n=12$$ strong fairness assumptions, taking 3 h 14 m
for the automata translation alone, before running any checks.
The cost is entirely in the fairness. This is
also why liveness sometimes even gets skipped in practice, and why the
model checkers that do handle it are large, aggressively optimized programs whose
results are hard to trust. During the evaluation, one of the tools we compared against returned a *wrong
verdict* on two of my models, reproduced on two machines and two architectures,
and reported upstream.

**The approach: get fairness out of the formula.** Lasso never hands the
fairness to the LTL-to-Büchi automata translator. It classifies the conjuncts of the negated formula,
recognises which ones are fairness assumptions, normalises each into fair normal
form, and *hoists* them out, so that only the remaining property is translated. 
The hoisted assumptions become a condition that an accepting strongly connected component of the product
must additionally satisfy, checked on the fly by an SCC-based emptiness check
refining Tarjan's procedure.

**Pruning.** The standard way to discharge that condition enumerates all
$$2^n$$ combinations for n strong-fairness assumptions and runs a separate search for each, 
meaning it stays exponential and runs four million searches at $$n=22$$. 
I built a *recursive prune* based approach instead. The insight is that a fairness
assumption is a restriction on an SCC, so any candidate SCC is either already
fair, contains a smaller fair subgraph, or can never yield a fair counterexample
at all. 
The prune therefore does not commit to any of the combinations up front, it searches once,
and reduces a candidate SCC into its fair subgraph. When a candidate SCC is not yet fair according
to the assumptions a user wrote, only then does it commit to the option that
could still repair it, and recurse.
For standard fairness notions like weak and strong fairness, at most one
option could ever repair it, so it never branches at all, keeping the check polynomial
in $$n$$. For the earlier example of $$n=22$$, **Lasso** finishes it in $$< 50$$ ms.

**Verification.** Every component — the formula rewriting, the translation
through very weak alternating automata, the SCC enumeration, the prune — is
written and verified in Lean, in roughly 60 000 lines with no `sorry` anywhere.
Each entry point carries a matched pair of theorems: *soundness*, an empty
result means every run of the system satisfies the formula, and *completeness*,
a reported counterexample really is a run of the system that violates it. The
trusted base is reduced to Lean's three axioms and its kernel. 
No test suite is enough to make such a statement.

**Usability.** Lasso is integrated into
[Veil](https://github.com/verse-lab/veil), a foundational verifier embedded in
Lean for specifying and verifying distributed protocols, which already had a
safety model checker and now has a liveness one behind the same front end. I extended
Veil's syntax with LTL properties, including TLA-style *enabled* and *taken*
action primitives, of all the other tools I compared against only TLC supports these
natively.

**Results.** I evaluated against seven model checkers — TLC, SPOT, PAT, SPIN, NuSMV,
LTSmin and CAVA — on the official TLA<sup>+</sup> examples and on families built
to vary the fairness cost along each axis independently, plus an ablation across
six routes through my own pipeline so that each design decision is isolated with
everything else held fixed. Porting every model by hand meant validating each
port against TLC's own state graph. On TLA<sup>+</sup>'s own examples Lasso
tracks TLC closely and is faster at the median. Where fairness drives the cost
it is faster by an order of magnitude at the median and slightly slower on only two of
forty-seven cases. Across both suites it decides every benchmark TLC decides within the timeout and
twelve that TLC does not, with no disagreement anywhere. Against CAVA, the only
other fully verified LTL model checker, it is orders of magnitude faster.

**What this is for.** Previously, the field accepted a trade-off: verified *or* usable.
This work shows that we can, in fact, eat our cake and have it too.
A model checker can be machine-checked correct and still land in the same bracket 
as the industry-standard tool used for decades. And because Lasso lives in Lean, the specification you model
check is the specification you can go on to *prove*, with Mathlib and the rest
of the ecosystem available for the parts no finite check can settle.
