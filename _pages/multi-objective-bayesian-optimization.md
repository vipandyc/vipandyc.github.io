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
g_j(x)\le 0,\quad j=1,\ldots,c.
$$
</div>

We take every objective to be maximized. If some objective is naturally minimized, replace it by its negative.

A feasible point $x$ is Pareto dominated by $x'$ when

<div class="math-display">
$$
f_i(x')\ge f_i(x)\quad\forall i,
\qquad
f_j(x')>f_j(x)\quad\text{for at least one }j.
$$
</div>

The nondominated objective vectors form the Pareto front $P$. Bayesian optimization fits a surrogate to observations $\mathcal D_t$ and maximizes an acquisition function.

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

so error decreases as $S^{-1/2}$. Using the same samples, the improvement from a candidate $y$ is estimated by

<div class="math-display">
$$
\widehat{\operatorname{HVI}}_S(y)
=
\frac{V_B}{S}\sum_{s=1}^S
\mathbf 1\{y\succeq Z_s\}
\mathbf 1\{P\not\succeq Z_s\}.
$$
</div>

This is ordinary classical Monte Carlo integration, unrelated to quantum Monte Carlo.

## 2. Expected Hypervolume Improvement

At an unevaluated $x$, the surrogate makes $Y=f(x)$ random. Expected hypervolume improvement is

<div class="math-display">
$$
\alpha_{\mathrm{EHVI}}(x)
=
\mathbb E\!\left[
\operatorname{HVI}(Y;P,r)
\mid\mathcal D_t
\right].
$$
</div>

Let $A(P,r)$ be the region above $r$ not already dominated by $P$. Then

<div class="math-display">
$$
\operatorname{EHVI}(x)
=
\int_{A(P,r)}
\Pr(Y\succeq z\mid\mathcal D_t)\,\mathrm dz.
$$
</div>

Suppose $A(P,r)$ is partitioned into disjoint boxes $[\ell^{(k)},u^{(k)}]$ and the objective posteriors are independent,

<div class="math-display">
$$
Y_i\sim\mathcal N(\mu_i,\sigma_i^2).
$$
</div>

Then the integral factorizes:

<div class="math-display">
$$
\operatorname{EHVI}(x)
=
\sum_k\prod_{i=1}^m
\int_{\ell_i^{(k)}}^{u_i^{(k)}}
\Phi\!\left(\frac{\mu_i-z}{\sigma_i}\right)\mathrm dz.
$$
</div>

Writing $\psi(a)=a\Phi(a)+\phi(a)$, each one-dimensional factor is

<div class="math-display">
$$
\int_{\ell}^{u}\Phi\!\left(\frac{\mu-z}{\sigma}\right)\mathrm dz
=
\sigma\left[
\psi\!\left(\frac{\mu-\ell}{\sigma}\right)
-
\psi\!\left(\frac{\mu-u}{\sigma}\right)
\right].
$$
</div>

Thus analytic EHVI is mainly geometry: construct the box decomposition, then sum Gaussian terms.

For a joint batch $X=(x_1,\ldots,x_q)$, a practical estimator is

<div class="math-display">
$$
\widehat{q\operatorname{EHVI}}_S(X)
=
\frac{1}{S}\sum_{s=1}^S
\left[
\operatorname{HV}\!\left(P\cup f^{(s)}(X);r\right)
-
\operatorname{HV}(P;r)
\right],
$$
</div>

where $f^{(s)}(X)$ is a joint posterior draw. Joint sampling accounts for correlation between candidates, so redundant batch members add little. Implementations reuse the box decomposition and differentiate this sample average.

Objective normalization and the reference point matter: rescaling one objective rescales volume, while a poor $r$ can make the acquisition nearly flat.

## 3. ParEGO

ParEGO converts the vector objective into a randomly changing scalar objective. For normalized minimization objectives, draw $\lambda$ from the simplex and define

<div class="math-display">
$$
s_\lambda(y)
=
\max_i\lambda_i(y_i-z_i^\star)
+
\rho\sum_i\lambda_i(y_i-z_i^\star),
\qquad \rho>0.
$$
</div>

Fit a scalar surrogate to $s_\lambda(f(x))$ and maximize expected improvement. Resampling $\lambda$ sweeps the front; the Tchebycheff form can recover non-convex parts that a weighted sum misses.

| Method | Acquisition target | Main cost |
|---|---|---|
| EHVI | Expected new dominated volume | Hypervolume geometry |
| ParEGO | Expected improvement of a sampled scalarization | Repeated scalar searches |

## 4. Bandits and UCB

For a $K$-armed stochastic bandit, UCB1 selects

<div class="math-display">
$$
a_t
=
\arg\max_a
\left[
\widehat\mu_a(t)
+
\sqrt{\frac{2\log t}{n_a(t)}}
\right].
$$
</div>

The mean exploits and the radius explores. GP-UCB uses the analogous score

<div class="math-display">
$$
\alpha_{\mathrm{GP\text{-}UCB}}(x)
=
\mu_t(x)+\sqrt{\beta_t}\,\sigma_t(x).
$$
</div>

For multiple objectives, UCB still needs a scalarization or a Pareto rule. A second use is meta-optimization: treat EHVI, ParEGO, GP-UCB, random search, and an LLM proposer as arms, with a reward such as

<div class="math-display">
$$
R_t
=
\frac{\Delta\operatorname{HV}_t}{\text{evaluation cost}_t}.
$$
</div>

## 5. Feasibility Seeking and qLogPoF

For a Gaussian constraint surrogate

<div class="math-display">
$$
g_j(x)\mid\mathcal D_t
\sim
\mathcal N(\mu_j(x),\sigma_j^2(x)),
$$
</div>

the single-constraint feasibility probability is

<div class="math-display">
$$
p_j(x)
=
\Pr(g_j(x)\le0)
=
\Phi\!\left(-\frac{\mu_j(x)}{\sigma_j(x)}\right).
$$
</div>

With independent constraint posteriors,

<div class="math-display">
$$
\operatorname{PoF}(x)=\prod_{j=1}^c p_j(x),
\qquad
\log\operatorname{PoF}(x)=\sum_{j=1}^c\log p_j(x).
$$
</div>

The logarithm preserves the optimizer but avoids underflow when feasibility is rare.

For a batch $X=(x_1,\ldots,x_q)$, define a smooth feasibility indicator for posterior sample $s$,

<div class="math-display">
$$
I_\eta^{(s)}(x_k)
=
\prod_{j=1}^c
\operatorname{sigmoid}\!\left(
-\frac{g_j^{(s)}(x_k)}{\eta}
\right).
$$
</div>

A typical qLogPoF estimator has the structure

<div class="math-display">
$$
q\operatorname{LogPoF}(X)
\approx
\log\left[
\frac{1}{S}\sum_{s=1}^S
\max_{1\le k\le q}I_\eta^{(s)}(x_k)
\right],
$$
</div>

with smooth approximations to $\max$. This favors batches likely to contain a feasible point. After finding one, switch to constrained EHVI or feasibility-weighted improvement.

PoF is a posterior probability, not a safety guarantee.

## 6. Where an LLM Fits

An LLM can propose candidates from textual or physical knowledge, while the acquisition function supplies the quantitative decision rule:

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

A compact policy is

<div class="math-display">
$$
\alpha_t=
\begin{cases}
q\operatorname{LogPoF}, & \text{no feasible observation},\\[3pt]
q\operatorname{LogEHVI}\ \text{or ParEGO}, & \text{otherwise},
\end{cases}
$$
</div>

Optionally, a bandit allocates budget among acquisition rules and proposal generators.

## References

- Knowles, "ParEGO: A Hybrid Algorithm with On-Line Landscape Approximation for Expensive Multiobjective Optimization Problems" (2006): [author page](https://staff.cs.manchester.ac.uk/~jknowles/parego/).
- Auer, Cesa-Bianchi, and Fischer, "Finite-time Analysis of the Multiarmed Bandit Problem" (2002): [Machine Learning](https://doi.org/10.1023/A:1013689704352).
- Srinivas, Krause, Kakade, and Seeger, "Gaussian Process Optimization in the Bandit Setting" (2010): [arXiv:0912.3995](https://arxiv.org/abs/0912.3995).
- Daulton, Balandat, and Bakshy, "Differentiable Expected Hypervolume Improvement for Parallel Multi-Objective Bayesian Optimization" (2020): [NeurIPS](https://papers.neurips.cc/paper/2020/hash/6fec24eac8f18ed793f5eaad3dd7977c-Abstract.html).
- Ament et al., "Unexpected Improvements to Expected Improvement for Bayesian Optimization" (2023): [arXiv:2310.20708](https://arxiv.org/abs/2310.20708).
- BoTorch implementation of `qLogProbabilityOfFeasibility`: [source](https://github.com/meta-pytorch/botorch/blob/main/botorch/acquisition/logei.py).
