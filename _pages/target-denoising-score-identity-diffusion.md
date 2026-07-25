---
permalink: /blog/target-denoising-score-identity-diffusion/
title: "Target Score Identity and Denoising Score Identity in Diffusion Models"
excerpt: "A compact derivation of TSI and DSI for Gaussian noising, their connection to Tweedie's formula, and when each identity is useful."
author_profile: true
---

# Target Score Identity and Denoising Score Identity in Diffusion Models

Diffusion models keep asking for the same object: the score of a noised distribution,

<div class="math-display">
$$
s_t(y)=\nabla_y \log p_t(y).
$$
</div>

Here $p_t$ is the marginal law after corrupting clean data $X_0\sim p_0$ into a noisy variable $Y_t$. For the usual Gaussian forward process,

<div class="math-display">
$$
Y_t = \alpha_t X_0 + \sigma_t \varepsilon,
\qquad
\varepsilon\sim\mathcal N(0,I),
\qquad
p_{t|0}(y|x)=\mathcal N(y;\alpha_t x,\sigma_t^2 I).
$$
</div>

The marginal noised density is

<div class="math-display">
$$
p_t(y)=\int p_0(x)\,p_{t|0}(y|x)\,\mathrm dx.
$$
</div>

The problem is that $p_t$ is usually intractable, so $s_t(y)$ is not directly available. The denoising score identity (DSI) and target score identity (TSI) give two different unbiased conditional signals for the same score.

## 1. The Two Identities

For the Gaussian corruption above, define the clean target score

<div class="math-display">
$$
s_0(x)=\nabla_x\log p_0(x).
$$
</div>

Then the two identities are

<div class="math-display">
$$
\boxed{
s_t(y)
=
\mathbb E\!\left[
\nabla_y\log p_{t|0}(y|X_0)
\mid Y_t=y
\right]
}
\qquad\text{(DSI)}
$$
</div>

and

<div class="math-display">
$$
\boxed{
s_t(y)
=
\frac{1}{\alpha_t}
\mathbb E\!\left[
s_0(X_0)
\mid Y_t=y
\right]
}
\qquad\text{(TSI).}
$$
</div>

Since

<div class="math-display">
$$
\nabla_y\log p_{t|0}(y|x)
=
\frac{\alpha_t x-y}{\sigma_t^2},
$$
</div>

DSI is often written as

<div class="math-display">
$$
s_t(y)
=
\mathbb E\!\left[
\frac{\alpha_t X_0-y}{\sigma_t^2}
\mid Y_t=y
\right].
$$
</div>

The important point is not that these formulas look similar. They use different information:

- DSI uses only the known noising kernel $p_{t|0}(y|x)$ and paired samples $(X_0,Y_t)$.
- TSI uses the score of the clean target density, $s_0(x)=\nabla_x\log p_0(x)$.

So DSI is natural for ordinary data-driven diffusion training. TSI is natural when $p_0$ is an energy, posterior, Boltzmann distribution, or any differentiable target whose log-gradient is available up to a normalizing constant.

## 2. Proof of DSI

Start from the marginal density

<div class="math-display">
$$
p_t(y)=\int p_0(x)\,p_{t|0}(y|x)\,\mathrm dx.
$$
</div>

Differentiate under the integral:

<div class="math-display">
$$
\nabla_y p_t(y)
=
\int p_0(x)\,\nabla_y p_{t|0}(y|x)\,\mathrm dx.
$$
</div>

Rewrite $\nabla_y p_{t|0}=p_{t|0}\nabla_y\log p_{t|0}$:

<div class="math-display">
$$
\nabla_y p_t(y)
=
\int p_0(x)\,p_{t|0}(y|x)
\nabla_y\log p_{t|0}(y|x)\,\mathrm dx.
$$
</div>

Divide by $p_t(y)$ and recognize Bayes' posterior

<div class="math-display">
$$
p_{0|t}(x|y)
=
\frac{p_0(x)p_{t|0}(y|x)}{p_t(y)}.
$$
</div>

This gives

<div class="math-display">
$$
\nabla_y\log p_t(y)
=
\int
\nabla_y\log p_{t|0}(y|x)\,
p_{0|t}(x|y)\,\mathrm dx.
$$
</div>

That is DSI. For Gaussian noising, the conditional score is exactly $(\alpha_t x-y)/\sigma_t^2$.

## 3. Proof of TSI

TSI uses integration by parts. Assume $p_0$ is differentiable and boundary terms vanish. For isotropic Gaussian noising,

<div class="math-display">
$$
\nabla_x p_{t|0}(y|x)
=
-\alpha_t \nabla_y p_{t|0}(y|x).
$$
</div>

Now compute the posterior expectation of the clean score:

<div class="math-display">
$$
\mathbb E[s_0(X_0)\mid Y_t=y]
=
\frac{1}{p_t(y)}
\int \nabla_x\log p_0(x)\,p_0(x)\,p_{t|0}(y|x)\,\mathrm dx.
$$
</div>

Since $\nabla_x\log p_0(x)\,p_0(x)=\nabla_x p_0(x)$,

<div class="math-display">
$$
\mathbb E[s_0(X_0)\mid Y_t=y]
=
\frac{1}{p_t(y)}
\int \nabla_x p_0(x)\,p_{t|0}(y|x)\,\mathrm dx.
$$
</div>

Integrate by parts:

<div class="math-display">
$$
\int \nabla_x p_0(x)\,p_{t|0}(y|x)\,\mathrm dx
=
-\int p_0(x)\,\nabla_x p_{t|0}(y|x)\,\mathrm dx.
$$
</div>

Use $\nabla_x p_{t|0}=-\alpha_t\nabla_y p_{t|0}$:

<div class="math-display">
$$
\int \nabla_x p_0(x)\,p_{t|0}(y|x)\,\mathrm dx
=
\alpha_t
\int p_0(x)\,\nabla_y p_{t|0}(y|x)\,\mathrm dx
=
\alpha_t \nabla_y p_t(y).
$$
</div>

Therefore

<div class="math-display">
$$
\mathbb E[s_0(X_0)\mid Y_t=y]
=
\alpha_t\nabla_y\log p_t(y)
=
\alpha_t s_t(y),
$$
</div>

or

<div class="math-display">
$$
s_t(y)
=
\frac{1}{\alpha_t}
\mathbb E[s_0(X_0)\mid Y_t=y].
$$
</div>

That is TSI.

## 4. Connection to Tweedie's Formula

Tweedie's formula says that, under Gaussian corruption, the posterior mean of the clean variable is obtained from the noisy observation plus a score correction. In the simplest variance-exploding case,

<div class="math-display">
$$
Y=X+\sigma\varepsilon,
\qquad
\varepsilon\sim\mathcal N(0,I),
$$
</div>

DSI gives

<div class="math-display">
$$
s_\sigma(y)
=
\mathbb E\!\left[
\frac{X-y}{\sigma^2}
\mid Y=y
\right]
=
\frac{\mathbb E[X\mid Y=y]-y}{\sigma^2}.
$$
</div>

Rearranging,

<div class="math-display">
$$
\boxed{
\mathbb E[X\mid Y=y]
=
y+\sigma^2 s_\sigma(y).
}
$$
</div>

For the more common scaled form $Y_t=\alpha_tX_0+\sigma_t\varepsilon$,

<div class="math-display">
$$
\boxed{
\mathbb E[X_0\mid Y_t=y]
=
\frac{y+\sigma_t^2 s_t(y)}{\alpha_t}.
}
$$
</div>

This is why DSI, Tweedie's formula, and denoising are almost the same story told from different angles:

- the score tells you how to move a noisy point toward regions of higher marginal probability;
- the posterior mean tells you the Bayes-optimal denoised clean estimate;
- the residual $\mathbb E[\varepsilon\mid Y_t=y]$ satisfies

<div class="math-display">
$$
\mathbb E[\varepsilon\mid Y_t=y]
=
-\sigma_t s_t(y).
$$
</div>

In neural diffusion models, these are just different parameterizations. Predicting score, noise, clean data, and velocity are algebraically connected once $\alpha_t$ and $\sigma_t$ are fixed.

## 5. Score Matching View

Suppose we train a network $s_\theta(y,t)$ with squared loss. DSI uses the target

<div class="math-display">
$$
b(X_0,Y_t,t)
=
\nabla_y\log p_{t|0}(Y_t|X_0)
=
\frac{\alpha_tX_0-Y_t}{\sigma_t^2}.
$$
</div>

TSI uses the target

<div class="math-display">
$$
c(X_0,t)
=
\frac{1}{\alpha_t}s_0(X_0).
$$
</div>

Both have the same conditional mean:

<div class="math-display">
$$
\mathbb E[b(X_0,Y_t,t)\mid Y_t=y]
=
\mathbb E[c(X_0,t)\mid Y_t=y]
=
s_t(y).
$$
</div>

Therefore, for either target $U\in\{b,c\}$, the minimizer of

<div class="math-display">
$$
\mathbb E\left[
\lambda(t)\|s_\theta(Y_t,t)-U\|^2
\right]
$$
</div>

is the same ideal function:

<div class="math-display">
$$
s_\theta^\star(y,t)=s_t(y).
$$
</div>

The difference is variance, not the population optimum. DSI and TSI supervise the same score through different noisy labels.

## 6. When to Use DSI

Use DSI when you have samples but not the clean data score.

This is the standard image/audio/text-latent diffusion setting. You can sample $X_0$ from the dataset, sample $\varepsilon$, construct $Y_t=\alpha_tX_0+\sigma_t\varepsilon$, and compute the denoising target from the known Gaussian kernel. You do not need $\nabla_x\log p_0(x)$, which is good because for an empirical dataset it is usually not a meaningful smooth object.

DSI is especially convenient at moderate and high noise levels. But at very low noise,

<div class="math-display">
$$
b(X_0,Y_t,t)
=
-\frac{\varepsilon}{\sigma_t},
$$
</div>

up to the usual scaled-noising convention. As $\sigma_t\to 0$, this target can have large variance even though its conditional mean remains finite. This is one reason low-noise score estimation can be delicate.

## 7. When to Use TSI

Use TSI when the clean target score is available or can be estimated reliably:

- energy-based sampling, where $p_0(x)\propto \exp(-E(x))$ and $s_0(x)=-\nabla E(x)$;
- Bayesian inverse problems, where the posterior score is the gradient of log prior plus log likelihood;
- molecular, statistical physics, or Monte Carlo settings with differentiable unnormalized densities;
- score distillation or hybrid methods where a proxy clean score is available.

The normalizing constant of $p_0$ is irrelevant because

<div class="math-display">
$$
\nabla_x\log p_0(x)
=
\nabla_x\log \bar p_0(x)
$$
</div>

when $p_0(x)=\bar p_0(x)/Z$.

TSI is attractive at low noise: as $\alpha_t\approx 1$, its signal is close to the clean target score. But at very high noise in variance-preserving or Ornstein-Uhlenbeck noising, $\alpha_t\to 0$, so

<div class="math-display">
$$
c(X_0,t)=\frac{s_0(X_0)}{\alpha_t}
$$
</div>

can become high-variance or numerically unstable.

## 8. Practical Rule of Thumb

DSI and TSI are two unbiased labels for the same conditional mean. Pick the one whose random label has lower variance in the regime you care about.

| Regime / information | Prefer | Reason |
|---|---:|---|
| Only dataset samples are available | DSI | No clean target score is needed. |
| Clean density score $\nabla\log p_0$ is available | TSI at low noise | Avoids the $1/\sigma_t$ low-noise explosion of denoising targets. |
| High-noise VP/OU regime, $\alpha_t\ll 1$ | DSI | TSI has the $1/\alpha_t$ scaling. |
| Physics, Bayesian, or energy-based sampling | TSI or a blend | Target score is often cheap relative to learning a low-noise denoiser. |
| Full diffusion schedule with both signals available | Blend / control variate | Both identities have the same mean but different variance profiles. |

A useful mental model is:

<div class="math-display">
$$
\text{DSI: score from the corruption residual,}
\qquad
\text{TSI: score from the clean target force.}
$$
</div>

In a physical system, DSI says "infer the force from how the noisy observation was generated." TSI says "average the true clean force over all clean states compatible with the noisy observation."

## 9. References

- Vincent, "A Connection Between Score Matching and Denoising Autoencoders" (2011): [Neural Computation](https://doi.org/10.1162/NECO_a_00142).
- Song and Ermon, "Generative Modeling by Estimating Gradients of the Data Distribution" (2019): [arXiv:1907.05600](https://arxiv.org/abs/1907.05600).
- Song et al., "Score-Based Generative Modeling through Stochastic Differential Equations" (2021): [ICLR / OpenReview](https://openreview.net/forum?id=PxTIG12RRHS).
- De Bortoli, Hutchinson, Wirnsberger, and Doucet, "Target Score Matching" (2024): [arXiv:2402.08667](https://arxiv.org/abs/2402.08667).
- Duston and Bui-Thanh, "Variance-Reduced Diffusion Sampling via Target Score Identity" (2026): [arXiv:2601.01594](https://arxiv.org/abs/2601.01594).
