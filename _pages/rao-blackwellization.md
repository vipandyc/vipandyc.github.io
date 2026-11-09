---
permalink: /blog/rao-blackwellization/
title: "Rao–Blackwellization: Average Out the Noise You Can"
excerpt: "An applied introduction to conditional expectation as variance reduction, with a worked Monte Carlo example and applications to latent-variable models and particle filters."
author_profile: true
---

# Rao–Blackwellization: Average Out the Noise You Can

Suppose a simulation draws two random quantities, $X$ and $Y$, and we want the mean of an output $G(X,Y)$. One way to estimate it is to run the simulation many times and average the outputs. But if, after drawing $X$, we can **calculate the average over $Y$**, why also sample $Y$? Rao–Blackwellization replaces that avoidable random draw with a conditional expectation.

The idea appears in classical statistics as a way to improve an estimator. In applied math it is also a practical recipe for reducing Monte Carlo noise: **sample the difficult part; integrate out the tractable part**.

## 1. The Basic Construction

Let $G=G(X,Y)$ have a finite second moment, and define

<div class="math-display">
$$
\mu=\mathbb E[G(X,Y)],
\qquad
m(X)=\mathbb E[G(X,Y)\mid X].
$$
</div>

The conditional expectation $m(x)$ means: hold $X=x$ fixed and average over the remaining randomness in $Y\mid X=x$. For a discrete $Y$ it is a sum; for a continuous $Y$ it is an integral:

<div class="math-display">
$$
m(x)=\sum_y G(x,y)\,p(y\mid x)
\quad\text{or}\quad
m(x)=\int G(x,y)\,p(y\mid x)\,\mathrm dy.
$$
</div>

The **tower property** says $\mathbb E[m(X)]=\mathbb E[G]=\mu$. Thus, with independent draws $X_1,\ldots,X_N$, both

<div class="math-display">
$$
\widehat\mu_{\rm raw}=\frac1N\sum_{i=1}^N G(X_i,Y_i),
\qquad
\widehat\mu_{\rm RB}=\frac1N\sum_{i=1}^N m(X_i)
$$
</div>

estimate the same mean. The second estimator has averaged away the randomness in $Y$ before the sample average is taken.

## 2. Why the Variance Falls

Here $G=G(X,Y)$. The **law of total variance** separates the variation across $X$ from the variation left within each fixed $X$:

<div class="math-display">
$$
\operatorname{Var}(G)
=\operatorname{Var}_X\bigl(\mathbb E_Y[G\mid X]\bigr)
+\mathbb E_X\bigl[\operatorname{Var}_Y(G\mid X)\bigr].
$$
</div>

Think of each fixed value of $X$ as a group. The first term is the **variance between group means**; the second is the **average variance within groups**.

Rao–Blackwellization keeps the group means and removes the within-group sampling noise. Because the within-group term is nonnegative, conditioning cannot increase the variance.

To see why, write $G=m(X)+R$ with $R=G-m(X)$. Since $\mathbb E[R\mid X]=0$, we have $\mathbb E[m(X)R]=\mathbb E[m(X)\mathbb E[R\mid X]]=0$: the two parts are uncorrelated, so their variances add. Equality holds only when $G$ already has no randomness left after $X$ is known.

For independent samples, each estimator's variance is its single-draw variance divided by $N$. Rao–Blackwellization therefore reduces the Monte Carlo error at the same sample count. Since the mean is preserved, it also cannot increase mean squared error for estimating $\mu$.

The classical **Rao–Blackwell theorem** takes an estimator of an unknown parameter and conditions it on a *sufficient statistic* $T$. Sufficiency means that, once $T$ is known, the conditional distribution of the original data does not depend on the unknown parameter. This makes the improved estimator $\mathbb E[\widehat\theta\mid T]$ usable without knowing $\theta$. In Monte Carlo problems the sampling law is usually specified, so we can condition on any convenient $X$ for which $m(X)$ is computable; sufficiency is not required. See [Blackwell's original paper](https://doi.org/10.1214/aoms/1177730497) for the statistical result.

## 3. A Small Example with Exact Numbers

Draw $X$ uniformly from $[0,1]$, then draw $Y\mid X=x$ as a Bernoulli random variable with success probability $x$. We want $\mu=\mathbb E[Y]$. A raw simulation records either $0$ or $1$, but after drawing $X=x$ we already know that $\mathbb E[Y\mid X=x]=x$. The Rao–Blackwellized simulation records $X$ itself.

Both methods have mean $1/2$. Their variances differ:

<div class="math-display">
$$
\begin{aligned}
\operatorname{Var}(Y)&=\tfrac14,\\
\operatorname{Var}(\mathbb E[Y\mid X])
&=\operatorname{Var}(X)=\tfrac1{12},\\
\mathbb E[\operatorname{Var}(Y\mid X)]
&=\mathbb E[X(1-X)]=\tfrac16.
\end{aligned}
$$
</div>

Indeed, $1/4=1/12+1/6$, exactly as the total-variance identity predicts. At the same $N$, averaging the $X$ values has **one-third the variance** of averaging the $Y$ values, or a standard error smaller by $\sqrt3$. The example is deliberately simple; in a real model the conditional average might be a nontrivial but manageable calculation.

## 4. Where the Trick Is Useful

**Conditional Monte Carlo.** To estimate a rare-event probability $\Pr(Y>a)$, a raw simulation averages the indicators $\mathbf 1_{\{Y>a\}}$. If the conditional distribution of $Y\mid X$ is known, average $\Pr(Y>a\mid X)$ instead. Each simulated $X$ then contributes a probability between $0$ and $1$, rather than a noisy yes/no outcome.

**Latent-variable models.** If a model contains an easy-to-integrate latent variable $Z$, replace a sampled contribution $G(X,Z)$ by $\int G(X,z)p(z\mid X)\,\mathrm dz$. This is useful when that integral is analytic or can be evaluated cheaply and accurately, for example with a Gaussian conditional distribution.

**Particle filters.** In a state-space model with a nonlinear component and a conditionally linear-Gaussian component, particles can represent the nonlinear part while a Kalman filter integrates the Gaussian part for each particle. This is a *Rao–Blackwellized particle filter*: it spends sampling effort on the part that cannot be handled analytically. A [particle-filter study by Doucet and colleagues](https://arxiv.org/abs/1301.3853) develops this construction for dynamic Bayesian networks.

## 5. The Computational Tradeoff

The variance inequality compares estimators at the **same number of samples**. If computing $m(X)$ is expensive, compare error at the same *time budget* instead. Suppose one conditional-average draw costs $c$ times as much as one raw draw. For independent draws and a large fixed budget, the Rao–Blackwellized method wins on variance when roughly

<div class="math-display">
$$
c\,\operatorname{Var}(m(X))<\operatorname{Var}(G).
$$
</div>

In the Bernoulli example, the variance falls by a factor of three, so the conditional-average draw could cost up to about three times as much and still be competitive. Setup costs and correlated samples can change this comparison.

The practical workflow is: identify the random variables in an estimator, find a conditional average that can be computed **exactly or to controlled accuracy**, then compare variance *per unit of computation*. If the conditional average is only approximated, its error must be included; the exact unbiasedness and variance guarantee do not automatically carry over. The useful mental model is simple: when part of a simulation's randomness can be averaged analytically, do that calculation before spending samples on it.
