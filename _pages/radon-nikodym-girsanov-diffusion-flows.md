---
permalink: /blog/radon-nikodym-girsanov-diffusion-flows/
title: "Radon-Nikodym, Girsanov, and Diffusion Flow Cheat Sheet"
excerpt: "An intuition-first note on change of measure, likelihood ratios, score SDEs, probability-flow ODEs, DDIM, flow matching, and Fokker-Planck equations."
author_profile: true
---

# Radon-Nikodym, Girsanov, and Diffusion Flow Cheat Sheet

This note is for the common situation where the same random object is being described by two different probability laws. In physics language, think of two ensembles on the same configuration space. The Radon-Nikodym derivative is the pointwise reweighting factor between them. Girsanov's theorem is the path-space version for diffusion processes: it tells us how much probability weight changes when we change the drift of an SDE but keep the noise strength fixed.

The goal here is not measure-theory elegance. The goal is to make the object computable, while keeping just enough rigor to avoid misleading pictures. Then we connect it to the formulas that appear in score-based diffusion, DDIM/probability-flow ODEs, flow matching, Fokker-Planck equations, continuity equations, and reverse-time SDEs.

## 1. Radon-Nikodym Derivative

Suppose two probability laws $P$ and $Q$ live on the same measurable space $(\Omega,\mathcal F)$. Here $\Omega$ is the sample space and $\mathcal F$ is the collection of events we are allowed to assign probabilities to.

The Radon-Nikodym derivative

<div class="math-display">
$$
\frac{\mathrm dP}{\mathrm dQ}(x)
$$
</div>

is the local density ratio: how much more strongly $P$ weights the point $x$ compared with $Q$.

The upright $\mathrm d$ is the convention many books use for measure/integration notation:

<div class="math-display">
$$
\mathrm dP(x)
=
\frac{\mathrm dP}{\mathrm dQ}(x)\,\mathrm dQ(x).
$$
</div>

So $\mathrm dP/\mathrm dQ$ should not be read as an ordinary derivative with respect to a coordinate. It means "the density of the measure $P$ relative to the measure $Q$." In informal writing people often type $dP/dQ$, but the upright $\mathrm d$ version is the cleaner notation.

If both laws have ordinary densities $p(x)$ and $q(x)$ with respect to the same base measure $\mathrm dx$, then nothing mysterious is happening:

<div class="math-display">
$$
\frac{\mathrm dP}{\mathrm dQ}(x)=\frac{p(x)}{q(x)}
\qquad
\text{where }q(x)>0.
$$
</div>

The extra condition is that $Q$ must not assign zero probability to regions where $P$ assigns positive probability. In symbols,

<div class="math-display">
$$
P\ll Q
\qquad
\Longleftrightarrow
\qquad
Q(A)=0\Rightarrow P(A)=0.
$$
</div>

Computably, this just means: do not try to reweight from samples that never visit the important states.

### Reweighting Identity

The main use is expectation conversion:

<div class="math-display">
$$
E_P[f(X)]
=
E_Q\left[f(X)\frac{\mathrm dP}{\mathrm dQ}(X)\right].
$$
</div>

This is importance sampling in its cleanest form.

More explicitly,

<div class="math-display">
$$
\int_\Omega f(x)\,\mathrm dP(x)
=
\int_\Omega f(x)\frac{\mathrm dP}{\mathrm dQ}(x)\,\mathrm dQ(x).
$$
</div>

### Physics Picture

For canonical ensembles with energies $E_0(x)$ and $E_1(x)$,

<div class="math-display">
$$
p_i(x)=\frac{e^{-\beta E_i(x)}}{Z_i}.
$$
</div>

Then

<div class="math-display">
$$
\frac{\mathrm dP_1}{\mathrm dP_0}(x)
=
\frac{Z_0}{Z_1}e^{-\beta(E_1(x)-E_0(x))}.
$$
</div>

So the Radon-Nikodym derivative is the Boltzmann reweighting factor plus a normalization constant. If the two ensembles barely overlap, the ratio has huge variance, and Monte Carlo becomes painful.

### Log Form

Most computations use the log ratio:

<div class="math-display">
$$
\log\frac{\mathrm dP}{\mathrm dQ}(x)=\log p(x)-\log q(x).
$$
</div>

This is the quantity that appears in likelihood-ratio estimators, variational bounds, KL divergence, and path-probability ratios.

## 2. The Radon-Nikodym Theorem

The theorem says:

If $P$ is absolutely continuous with respect to $Q$, meaning $Q(A)=0$ implies $P(A)=0$, then there exists a nonnegative measurable function $r(x)$ such that

<div class="math-display">
$$
P(A)=\int_A r(x)\,\mathrm dQ(x)
\qquad
\text{for all events }A.
$$
</div>

That function is unique up to $Q$-almost everywhere equality, and is written

<div class="math-display">
$$
r(x)=\frac{\mathrm dP}{\mathrm dQ}(x).
$$
</div>

The phrase "$Q$-almost everywhere" just means that changing $r$ on a set with $Q$-probability zero does not change any integral against $Q$. Intuitively: if $P$ never puts probability mass where $Q$ sees nothing, then $P$ can be built by locally reweighting $Q$.

### Computable Versions

- Discrete states: $\mathrm dP/\mathrm dQ(x)=P(x)/Q(x)$.
- Continuous densities: $\mathrm dP/\mathrm dQ(x)=p(x)/q(x)$.
- Path measures: $\mathrm dP/\mathrm dQ(\omega)$ is a likelihood ratio for the whole trajectory $\omega=\{X_t:0\le t\le T\}$.
- Monte Carlo: sample from $Q$, weight by $\mathrm dP/\mathrm dQ$.
- Training objective: log-likelihood ratios are usually easier than raw likelihood ratios.

## 3. Girsanov Theorem

Girsanov is the Radon-Nikodym theorem for changing the drift of a diffusion.

Start with a reference process

<div class="math-display">
$$
dX_t=b_0(X_t,t)\,dt+\sigma(t)\,dW_t.
$$
</div>

Now compare it with another process with the same diffusion matrix but different drift:

<div class="math-display">
$$
dX_t=b_1(X_t,t)\,dt+\sigma(t)\,dW_t.
$$
</div>

Let

<div class="math-display">
$$
u(X_t,t)=\sigma(t)^{-1}\bigl(b_1(X_t,t)-b_0(X_t,t)\bigr).
$$
</div>

Then, under standard regularity/integrability conditions, the path probability ratio is

<div class="math-display">
$$
\frac{\mathrm dP_1}{\mathrm dP_0}
=
\exp\left(
\int_0^T u_t\cdot dW_t
-
\frac12\int_0^T \lVert u_t\rVert^2\,dt
\right).
$$
</div>

Here $P_0$ and $P_1$ are probability measures on path space, not just densities at a single time. The random variable inside the exponential depends on the whole trajectory. That is the main conceptual jump from ordinary density ratios to stochastic-process density ratios.

The important interpretation:

<div class="math-display">
$$
\text{changing drift}
\quad\Longleftrightarrow\quad
\text{reweighting path probabilities}.
$$
</div>

This is directly related to Girsanov's theorem. Radon-Nikodym gives the language of density ratios between probability laws. Girsanov gives the explicit density ratio between two diffusion path measures.

### Discrete-Time Intuition

Euler discretize with step $\Delta t$:

<div class="math-display">
$$
X_{k+1}=X_k+b(X_k,t_k)\Delta t+\sigma\sqrt{\Delta t}\,\epsilon_k,
\qquad
\epsilon_k\sim N(0,I).
$$
</div>

Changing $b_0$ to $b_1$ shifts the mean of each Gaussian transition. The full trajectory likelihood ratio is the product of Gaussian transition-density ratios. Taking the continuous-time limit gives the Girsanov exponential.

### Why Same Noise Matters

Girsanov changes the drift while keeping the quadratic variation fixed. In physics language, you can bias the force field, but you cannot secretly change the temperature/noise amplitude and expect the same theorem to apply in the same simple form.

## 4. Score-Based Diffusion Cheat Sheet

Let $p_t(x)$ be the marginal density at time $t$, and define the score

<div class="math-display">
$$
s_t(x)=\nabla_x\log p_t(x).
$$
</div>

Forward SDE:

<div class="math-display">
$$
dX_t=f(X_t,t)\,dt+g(t)\,dW_t.
$$
</div>

Fokker-Planck equation:

<div class="math-display">
$$
\partial_t p_t(x)
=
-\nabla\cdot\bigl(f(x,t)p_t(x)\bigr)
+
\frac12 g(t)^2\Delta p_t(x).
$$
</div>

Reverse-time SDE:

<div class="math-display">
$$
dX_t=
\left[
f(X_t,t)-g(t)^2\nabla_x\log p_t(X_t)
\right]dt
+
g(t)\,d\bar W_t,
\qquad t:T\to0.
$$
</div>

Here $dt<0$ if we parameterize the solver backward in the original time variable. Many codebases instead introduce a new positive reverse time variable.

Probability-flow ODE:

<div class="math-display">
$$
\frac{dX_t}{dt}
=
f(X_t,t)
-
\frac12 g(t)^2\nabla_x\log p_t(X_t).
$$
</div>

Same marginals as the forward/reverse SDE, but deterministic trajectories.

Score matching denoising target, common VP-style form:

<div class="math-display">
$$
X_t=\alpha_t X_0+\sigma_t\epsilon,
\qquad
\epsilon\sim N(0,I),
$$
</div>

<div class="math-display">
$$
\nabla_{x_t}\log p(x_t|x_0)
=
-\frac{x_t-\alpha_t x_0}{\sigma_t^2}
=
-\frac{\epsilon}{\sigma_t}.
$$
</div>

Denoising score matching loss:

<div class="math-display">
$$
L(\theta)
=
E_{t,x_0,\epsilon}
\left[
\lambda(t)
\left\lVert
s_\theta(x_t,t)
+
\frac{\epsilon}{\sigma_t}
\right\rVert^2
\right].
$$
</div>

Noise-prediction parameterization:

<div class="math-display">
$$
s_\theta(x_t,t)\approx-\frac{\epsilon_\theta(x_t,t)}{\sigma_t}.
$$
</div>

$x_0$-prediction relation:

<div class="math-display">
$$
\hat x_0(x_t,t)
=
\frac{x_t-\sigma_t\epsilon_\theta(x_t,t)}{\alpha_t}.
$$
</div>

## 5. VP, VE, and Sub-VP Common Forms

Variance-preserving SDE:

<div class="math-display">
$$
dX_t=-\frac12\beta(t)X_t\,dt+\sqrt{\beta(t)}\,dW_t.
$$
</div>

VP marginal:

<div class="math-display">
$$
X_t=\alpha_tX_0+\sigma_t\epsilon,
\qquad
\alpha_t=\exp\left(-\frac12\int_0^t\beta(s)\,ds\right),
\qquad
\sigma_t^2=1-\alpha_t^2.
$$
</div>

VP reverse SDE:

<div class="math-display">
$$
dX_t=
\left[
-\frac12\beta(t)X_t
-
\beta(t)s_t(X_t)
\right]dt
+
\sqrt{\beta(t)}\,d\bar W_t.
$$
</div>

VP probability-flow ODE:

<div class="math-display">
$$
\frac{dX_t}{dt}
=
-\frac12\beta(t)X_t
-
\frac12\beta(t)s_t(X_t).
$$
</div>

Variance-exploding SDE:

<div class="math-display">
$$
dX_t=g(t)\,dW_t,
\qquad
\sigma_t^2=\int_0^t g(s)^2\,ds.
$$
</div>

VE reverse SDE:

<div class="math-display">
$$
dX_t=-g(t)^2s_t(X_t)\,dt+g(t)\,d\bar W_t.
$$
</div>

VE probability-flow ODE:

<div class="math-display">
$$
\frac{dX_t}{dt}
=
-\frac12g(t)^2s_t(X_t).
$$
</div>

Sub-VP SDE, one common form:

<div class="math-display">
$$
dX_t=-\frac12\beta(t)X_t\,dt
+
\sqrt{\beta(t)\bigl(1-e^{-2\int_0^t\beta(s)\,ds}\bigr)}\,dW_t.
$$
</div>

## 6. DDPM and DDIM Relations

Discrete forward noising:

<div class="math-display">
$$
q(x_t|x_0)=N(\sqrt{\bar\alpha_t}x_0,(1-\bar\alpha_t)I).
$$
</div>

Noise form:

<div class="math-display">
$$
x_t=\sqrt{\bar\alpha_t}x_0+\sqrt{1-\bar\alpha_t}\epsilon.
$$
</div>

Predict clean sample:

<div class="math-display">
$$
\hat x_0=
\frac{x_t-\sqrt{1-\bar\alpha_t}\epsilon_\theta(x_t,t)}
{\sqrt{\bar\alpha_t}}.
$$
</div>

DDPM mean:

<div class="math-display">
$$
\mu_\theta(x_t,t)
=
\frac{1}{\sqrt{\alpha_t}}
\left(
x_t
-
\frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\epsilon_\theta(x_t,t)
\right).
$$
</div>

DDPM sampling:

<div class="math-display">
$$
x_{t-1}=\mu_\theta(x_t,t)+\sigma_t z,
\qquad
z\sim N(0,I).
$$
</div>

DDIM deterministic update:

<div class="math-display">
$$
x_{t-1}
=
\sqrt{\bar\alpha_{t-1}}\hat x_0
+
\sqrt{1-\bar\alpha_{t-1}}\epsilon_\theta(x_t,t).
$$
</div>

DDIM with stochasticity parameter $\eta$:

<div class="math-display">
$$
x_{t-1}
=
\sqrt{\bar\alpha_{t-1}}\hat x_0
+
\sqrt{1-\bar\alpha_{t-1}-\sigma_t^2}\epsilon_\theta
+
\sigma_t z.
$$
</div>

The deterministic DDIM update is a discrete non-Markovian sampler related to the probability-flow ODE. It transports samples without injecting fresh noise at every step.

## 7. Continuity Equation and Flow Matching

For a deterministic flow

<div class="math-display">
$$
\frac{dX_t}{dt}=v_t(X_t),
$$
</div>

the density evolves by the continuity equation

<div class="math-display">
$$
\partial_t p_t(x)+\nabla\cdot(p_t(x)v_t(x))=0.
$$
</div>

This is the zero-diffusion version of Fokker-Planck.

Flow matching trains a velocity field $v_\theta(x,t)$ to match a target conditional velocity $u_t(x|x_0,x_1)$ along a chosen interpolation path:

<div class="math-display">
$$
L(\theta)
=
E_{t,x_0,x_1,x_t}
\left[
\left\lVert v_\theta(x_t,t)-u_t(x_t|x_0,x_1)\right\rVert^2
\right].
$$
</div>

Linear interpolation path:

<div class="math-display">
$$
x_t=(1-t)x_0+tx_1,
\qquad
u_t=x_1-x_0.
$$
</div>

Gaussian conditional path:

<div class="math-display">
$$
x_t=\mu_t(x_0,x_1)+\sigma_t\epsilon.
$$
</div>

Conditional velocity from the path:

<div class="math-display">
$$
u_t(x_t|x_0,x_1)
=
\dot\mu_t
+
\frac{\dot\sigma_t}{\sigma_t}(x_t-\mu_t).
$$
</div>

Sampling ODE:

<div class="math-display">
$$
\frac{dX_t}{dt}=v_\theta(X_t,t),
\qquad
X_0\sim p_{\rm base},
\qquad
X_1\sim p_{\rm data}.
$$
</div>

## 8. Fokker-Planck, Current, and Physics Notation

General Ito SDE:

<div class="math-display">
$$
dX_t=f(X_t,t)\,dt+G(X_t,t)\,dW_t.
$$
</div>

Diffusion tensor:

<div class="math-display">
$$
D(x,t)=\frac12G(x,t)G(x,t)^T.
$$
</div>

Fokker-Planck:

<div class="math-display">
$$
\partial_t p
=
-\sum_i\partial_i(f_i p)
+
\sum_{i,j}\partial_i\partial_j(D_{ij}p).
$$
</div>

If $D$ is constant:

<div class="math-display">
$$
\partial_t p=-\nabla\cdot(fp)+D:\nabla\nabla p.
$$
</div>

Probability current form:

<div class="math-display">
$$
\partial_t p+\nabla\cdot J=0,
\qquad
J=fp-D\nabla p
\quad
\text{for constant }D.
$$
</div>

Using the score:

<div class="math-display">
$$
J=p\bigl(f-D\nabla\log p\bigr).
$$
</div>

So diffusion can be viewed as a score-driven current down density gradients.

## 9. Reverse SDE and Probability-Flow ODE Side by Side

Forward:

<div class="math-display">
$$
dX_t=f\,dt+g\,dW_t.
$$
</div>

Reverse SDE:

<div class="math-display">
$$
dX_t=(f-g^2s_t)\,dt+g\,d\bar W_t,
\qquad
s_t=\nabla\log p_t.
$$
</div>

Probability-flow ODE:

<div class="math-display">
$$
dX_t=\left(f-\frac12g^2s_t\right)dt.
$$
</div>

Same one-time marginals:

<div class="math-display">
$$
p_t^{\rm reverse\ SDE}=p_t^{\rm probability\ flow\ ODE}.
$$
</div>

Different trajectories:

<div class="math-display">
$$
\text{SDE paths are stochastic,}
\qquad
\text{ODE paths are deterministic.}
$$
</div>

Likelihood from probability-flow ODE:

<div class="math-display">
$$
\frac{d}{dt}\log p_t(X_t)
=
-\nabla\cdot v_t(X_t),
\qquad
v_t=f-\frac12g^2s_t.
$$
</div>

Therefore

<div class="math-display">
$$
\log p_0(x_0)
=
\log p_T(x_T)
+
\int_0^T \nabla\cdot v_t(X_t)\,dt,
$$
</div>

with sign depending on whether the ODE is integrated forward or backward.

## 10. Where Radon-Nikodym and Girsanov Enter Diffusion Models

- Score models learn $\nabla\log p_t$, not the full density $p_t$.
- Reverse SDE formulas come from comparing forward and backward path measures.
- Girsanov gives the likelihood ratio when one drift is replaced by another drift.
- Classifier guidance and control terms can often be interpreted as drift changes.
- Path-space KLs between controlled and uncontrolled diffusions are quadratic control costs:

<div class="math-display">
$$
{\rm KL}(P_u\Vert P_0)
=
\frac12E_{P_u}\int_0^T\lVert u_t\rVert^2\,dt
$$
</div>

up to initial/terminal density terms and convention choices.

- Schrodinger bridge methods use this idea directly: find the most likely drift control that transforms one marginal distribution into another while staying close to a reference diffusion.
- Probability-flow ODEs turn stochastic density evolution into deterministic transport with the same marginals.
- Flow matching starts from the deterministic transport side and learns the velocity field directly.

## Big Picture

Radon-Nikodym derivative:

<div class="math-display">
$$
\frac{\mathrm dP}{\mathrm dQ}
=
\text{density ratio between two probability laws}.
$$
</div>

Girsanov theorem:

<div class="math-display">
$$
\frac{\mathrm dP_{\rm new\ drift}}{\mathrm dP_{\rm old\ drift}}
=
\text{exponential path-weight ratio}.
$$
</div>

Score-based diffusion:

<div class="math-display">
$$
\text{learn }s_t=\nabla\log p_t
\quad\Rightarrow\quad
\text{run reverse SDE or probability-flow ODE}.
$$
</div>

Flow matching:

<div class="math-display">
$$
\text{learn }v_t
\quad\Rightarrow\quad
\text{solve a continuity-equation transport ODE}.
$$
</div>

Physics translation:

<div class="math-display">
$$
\text{Radon-Nikodym}=\text{ensemble reweighting},
\qquad
\text{Girsanov}=\text{trajectory reweighting},
$$
</div>

<div class="math-display">
$$
\text{Fokker-Planck}=\text{density current balance},
\qquad
\text{score}=\text{log-density force}.
$$
</div>
