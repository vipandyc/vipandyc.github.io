---
permalink: /blog/multi-objective-bayesian-optimization/
title: "Multi-Objective Bayesian Optimization: Hypervolume, ParEGO, UCB, and Feasibility"
excerpt: "A compact mathematical guide to hypervolume computation, expected hypervolume improvement, ParEGO, UCB, qLogPoF, and LLM-guided candidate generation."
author_profile: true
---

# Multi-Objective Bayesian Optimization: Hypervolume, ParEGO, UCB, and Feasibility

Let $x\in\mathcal X$ be a costly experiment, with objectives and outcome constraints

<div class="math-display">
$$
f(x)=(f_1(x),\ldots,f_m(x)),
\qquad
c_j(x)\le 0,\quad j=1,\ldots,c.
$$
</div>

We take every objective to be maximized. If some objective is naturally minimized, replace it by its negative. Each $c_j(x)$ is a measured constraint margin; nonpositive means feasible.

A feasible point $x$ is Pareto dominated by $x'$ when

<div class="math-display">
$$
f_i(x')\ge f_i(x)\quad\forall i,
\qquad
f_j(x')>f_j(x)\quad\text{for at least one }j.
$$
</div>

We write $P$ for the **Pareto front**: the set of objective vectors $f(x)$ from feasible evaluated points that are not dominated by another evaluated point. Bayesian optimization fits a surrogate to observations $\mathcal D_t$ and maximizes an acquisition function.

## 1. Hypervolume

Choose a reference point $r\in\mathbb R^m$ worse than all objective values of interest. For $y\succeq r$, define the axis-aligned box

<div class="math-display">
$$
[r,y]=\prod_{i=1}^m[r_i,y_i].
$$
</div>

The dominated hypervolume is the Lebesgue measure of a union of boxes:

<div class="math-display">
$$
\operatorname{HV}(P;r)
=
\lambda_m\!\left(\bigcup_{y\in P}[r,y]\right).
$$
</div>

For a new objective vector $y$,

<div class="math-display">
$$
\operatorname{HVI}(y;P,r)
=
\operatorname{HV}(P\cup\{y\};r)-\operatorname{HV}(P;r).
$$
</div>

HVI is zero for a dominated point. It combines convergence and coverage of the front in one scalar.

### Exact computation in two dimensions

Let the nondominated points be sorted so that

<div class="math-display">
$$
y_1^{(1)}>y_1^{(2)}>\cdots>y_1^{(n)},
\qquad
y_2^{(1)}<y_2^{(2)}<\cdots<y_2^{(n)},
$$
</div>

and set $y_1^{(n+1)}=r_1$. A vertical sweep gives

<div class="math-display">
$$
\boxed{
\operatorname{HV}(P;r)
=
\sum_{k=1}^n
\bigl(y_1^{(k)}-y_1^{(k+1)}\bigr)
\bigl(y_2^{(k)}-r_2\bigr).
}
$$
</div>

Sorting costs $O(n\log n)$ and the sweep costs $O(n)$. For example, with $r=(0,0)$ and $P=\{(4,1),(2,3)\}$,

<div class="math-display">
$$
\operatorname{HV}(P;r)
=(4-2)(1-0)+(2-0)(3-0)=8.
$$
</div>

Adding $(3,2)$ fills one missing unit square, so $\operatorname{HVI}=1$.

### More than two objectives

The apparent simplicity disappears because the boxes overlap. Exact algorithms therefore partition the dominated region into disjoint cells, or recursively slice one coordinate:

<div class="math-display">
$$
\operatorname{HV}_m(P;r)
=
\sum_k \Delta z_k\,
\operatorname{HV}_{m-1}(P_k;r_{1:m-1}).
$$
</div>

Algorithms such as dimension sweep and WFG differ in how they construct $P_k$ and prune dominated subproblems. The number of cells grows rapidly with objective count $m$ and front size $n$: exact hypervolume is routine for two or three objectives, but can dominate the cost in many-objective problems.

### Monte Carlo estimate

When exact decomposition is too costly, choose a box $B=[r,u]$ containing the dominated region, with volume $V_B=\prod_i(u_i-r_i)$. For independent samples $Z_s\sim\operatorname{Unif}(B)$,

<div class="math-display">
$$
\widehat{\operatorname{HV}}_S(P;r)
=
\frac{V_B}{S}\sum_{s=1}^S
\mathbf 1\!\left\{\exists y\in P:y\succeq Z_s\right\}.
$$
</div>

This is unbiased. If $p=\operatorname{HV}(P;r)/V_B$, then

<div class="math-display">
$$
\operatorname{SE}\!\left(\widehat{\operatorname{HV}}_S\right)
=
V_B\sqrt{\frac{p(1-p)}{S}},
$$
</div>

so error decreases as $S^{-1/2}$. The same sampled points can estimate HVI: count the points dominated by the new candidate but not by the old front. This is ordinary classical Monte Carlo integration, unrelated to quantum Monte Carlo.

## 2. Expected Hypervolume Improvement

### Where the Gaussian process enters

After $t$ experiments, let $\mathcal D_t$ mean the complete table of tested settings and measured outcomes. Fit one Gaussian process (GP) to each objective. At a new setting $x$, the $i$th GP returns

<div class="math-display">
$$
f_i(x)\mid\mathcal D_t
\sim
\mathcal N\!\left(m_i(x),s_i^2(x)\right).
$$
</div>

Here $m_i(x)$ is the predicted objective and $s_i(x)$ is its uncertainty. The GP starts with a Gaussian-process prior over possible functions; conditioning on $\mathcal D_t$ gives the predictive distribution above. EHVI is an acquisition function placed on top of that posterior—it is not another prior.

The easiest way to understand EHVI is by posterior sampling. Draw $S$ plausible objective vectors from the fitted GPs and average how much hypervolume each one would add:

<div class="math-display">
$$
\widehat{\operatorname{EHVI}}(x)
=
\frac{1}{S}\sum_{s=1}^S
\operatorname{HVI}\!\left(f^{(s)}(x);P,r\right).
$$
</div>

This gives the actual sequential algorithm:

1. Fit or update the objective GPs using $\mathcal D_t$.
2. For every candidate $x$, sample plausible objective values from the GP posterior.
3. Compute the hypervolume gain of each sample and average it.
4. Evaluate the $x$ with largest EHVI.
5. Add the new measurements to $\mathcal D_t$ and repeat.

High predicted objectives give exploitation. Large uncertainty gives exploration because some posterior draws may produce a large new Pareto region. A candidate in a part of objective space already covered by the front receives little credit.

For $q$ experiments run in parallel, qEHVI samples the $q$ candidates jointly and scores the hypervolume gained by their union. Two nearly identical candidates are redundant, so the joint score naturally favors a diverse batch.

Objective normalization and the reference point matter: rescaling one objective rescales volume, while a poor $r$ can make the acquisition nearly flat.

## 3. ParEGO

ParEGO avoids computing hypervolume during candidate selection. Its idea is: **temporarily choose one trade-off, solve that scalar problem, then choose a different trade-off next time.**

For two objectives, a weight vector $\lambda=(0.8,0.2)$ emphasizes objective 1; the next iteration might draw $(0.25,0.75)$ and emphasize objective 2. The observations are not resampled or discarded. Only the preference direction changes.

The algorithm is:

1. Normalize all objective values to comparable scales.
2. Draw nonnegative random weights with $\sum_i\lambda_i=1$.
3. Convert every previously observed objective vector into one scalar score.
4. Fit a single GP to that scalar score.
5. Use ordinary expected improvement (EI)—the posterior-average reduction below the best (smallest) scalar score—to choose the next $x$.
6. Evaluate all objectives at $x$, append the result, draw new weights, and repeat.

The standard scalar score is an augmented Tchebycheff distance. For normalized minimization objectives,

<div class="math-display">
$$
s_\lambda(y)
=
\max_i\lambda_i\lvert y_i-z_i^\star\rvert
+
\rho\sum_i\lambda_i\lvert y_i-z_i^\star\rvert.
$$
</div>

$z^\star$ is an ideal objective vector and $\rho$ is a small tie-breaking constant. The maximum term asks, “under this weighting, which objective is furthest from ideal?” Improving the worst weighted deviation pulls the search toward the selected part of the front. Random weights make those selected parts move over time. Unlike a plain weighted sum, this scalarization can reach non-convex parts of the Pareto front.

| Method | Acquisition target | Main cost |
|---|---|---|
| EHVI | Expected new dominated volume | Hypervolume geometry |
| ParEGO | Expected improvement of a sampled scalarization | Repeated scalar searches |

## 4. Bandits and UCB

A multi-armed bandit is the simplest repeated exploration problem. Imagine $K$ actions, or “arms.” Arm $a$ gives a noisy reward with an unknown average. At each round you choose one arm and observe only its reward. The goal is not merely to identify the best arm eventually; it is to collect high total reward while learning.

This creates the central tension:

- **exploit:** choose the arm with the best observed average;
- **explore:** choose an uncertain arm that might actually be better.

Let $\bar r_a(t)$ be the average reward observed from arm $a$, and $n_a(t)$ the number of times it has been tried by round $t$. UCB chooses the largest optimistic score,

<div class="math-display">
$$
a_t
=
\arg\max_a
\left[
\bar r_a(t)
+
c\sqrt{\frac{\log t}{n_a(t)}}
\right].
$$
</div>

The first term is evidence; the second is an uncertainty bonus. Rarely tried arms receive a large bonus. As $n_a$ grows, the bonus shrinks and a mediocre arm stops being attractive.

The constant $c>0$ sets how aggressively UCB explores.

There are two useful connections to Bayesian optimization.

**Candidate-level UCB.** Treat every possible setting $x$ as an arm. A GP shares information between nearby settings, producing

<div class="math-display">
$$
\alpha_{\mathrm{GP\text{-}UCB}}(x)
=
m(x)+\kappa s(x).
$$
</div>

The GP mean $m(x)$ exploits and its uncertainty $s(x)$ explores; $\kappa>0$ controls the exploration strength. For multiple objectives, one must still scalarize the objectives or define a Pareto-aware UCB rule.

**Optional: use a bandit to choose the search method.** Here an arm is not a candidate $x$; it is an entire proposal method such as EHVI, ParEGO, GP-UCB, random search, or an LLM. At each iteration, the bandit chooses one method, that method proposes the next experiment, and the observed result gives the method a reward such as

<div class="math-display">
$$
\text{reward}
=
\frac{\text{new hypervolume gained}}{\text{evaluation cost}}.
$$
</div>

UCB can then allocate more future iterations to methods that have produced larger gains, while occasionally retrying the others. This optional outer controller does not replace the GP or acquisition functions; it only chooses which one to use on each iteration.

## 5. Feasibility Seeking and qLogPoF

Suppose an experiment is allowed only when a measured quantity stays below a limit. Define the constraint output directly, for example

<div class="math-display">
$$
c_j(x)=\text{temperature}(x)-T_{\max},
\qquad
\text{feasible when }c_j(x)\le0.
$$
</div>

Fit a GP to each measured constraint just as for an objective. At a proposed $x$, let $m_j(x)$ be its predicted constraint value and $s_j(x)$ its predictive standard deviation. Under a Gaussian predictive distribution,

<div class="math-display">
$$
\operatorname{PoF}_j(x)
=
\Pr(c_j(x)\le0)
=
\Phi\!\left(\frac{-m_j(x)}{s_j(x)}\right).
$$
</div>

Here $\Phi$ is the standard normal cumulative distribution. The intuition is immediate: a safely negative predicted mean raises PoF; large uncertainty pushes it back toward ambiguity. If several constraints are modeled independently, multiply their probabilities:

<div class="math-display">
$$
\operatorname{PoF}(x)=\prod_j\operatorname{PoF}_j(x).
$$
</div>

The name **qLogPoF** can be read literally:

- **q:** choose $q$ candidates together;
- **PoF:** maximize probability of feasibility;
- **Log:** compute in log space so tiny probabilities and gradients remain numerically usable.

If candidates were independent, the probability that a batch contains at least one feasible point would be

<div class="math-display">
$$
1-\prod_{k=1}^q\left[1-\operatorname{PoF}(x_k)\right].
$$
</div>

Actual qLogPoF implementations use joint GP samples and a smooth maximum over the $q$ candidates: roughly a differentiable Monte Carlo version of “at least one is feasible.” Correlations between batch members are therefore retained. The operational rule is simple: if no feasible point has been found, temporarily ignore objective improvement and choose the batch most likely to produce a feasible result. After feasibility is established, switch to constrained EHVI or another improvement acquisition weighted by feasibility.

PoF is a posterior probability, not a safety guarantee.

## 6. Where an LLM Fits

An LLM can propose candidates from textual or physical knowledge. It is especially useful for discrete, structured designs that are awkward for a continuous optimizer. The surrogate then supplies calibrated uncertainty and the acquisition function makes the quantitative decision:

<div class="math-display">
$$
\underbrace{\text{LLM proposals}}_{\mathcal C_t}
\;\longrightarrow\;
\arg\max_{x\in\mathcal C_t}
\underbrace{\alpha_t(x)}_{\text{EHVI, ParEGO, UCB, or PoF}}
\;\longrightarrow\;
\text{experiment}.
$$
</div>

A practical division of labor is:

- before finding feasibility, use qLogPoF;
- afterward, use constrained qLogEHVI or ParEGO;
- let an LLM enlarge the candidate pool when domain knowledge matters;
- optionally use a bandit to allocate budget among these search strategies.

## References

- Knowles, "ParEGO: A Hybrid Algorithm with On-Line Landscape Approximation for Expensive Multiobjective Optimization Problems" (2006): [author page](https://staff.cs.manchester.ac.uk/~jknowles/parego/).
- Auer, Cesa-Bianchi, and Fischer, "Finite-time Analysis of the Multiarmed Bandit Problem" (2002): [Machine Learning](https://doi.org/10.1023/A:1013689704352).
- Srinivas, Krause, Kakade, and Seeger, "Gaussian Process Optimization in the Bandit Setting" (2010): [arXiv:0912.3995](https://arxiv.org/abs/0912.3995).
- Daulton, Balandat, and Bakshy, "Differentiable Expected Hypervolume Improvement for Parallel Multi-Objective Bayesian Optimization" (2020): [NeurIPS](https://papers.neurips.cc/paper/2020/hash/6fec24eac8f18ed793f5eaad3dd7977c-Abstract.html).
- Ament et al., "Unexpected Improvements to Expected Improvement for Bayesian Optimization" (2023): [arXiv:2310.20708](https://arxiv.org/abs/2310.20708).
- BoTorch implementation of `qLogProbabilityOfFeasibility`: [source](https://github.com/meta-pytorch/botorch/blob/main/botorch/acquisition/logei.py).
