---
permalink: /blog/discrete-diffusion-uniform-masked/
title: "Discrete Diffusion: Uniform Noise and Masked Noise"
excerpt: "A compact methods note on categorical diffusion with uniform and absorbing-state corruption, including schedules, training objectives, sampling algorithms, and the connection between the two views."
author_profile: true
---

# Discrete Diffusion: Uniform Noise and Masked Noise

<style>
.algorithm-box {
  border-top: 1px solid #555;
  border-bottom: 1px solid #555;
  margin: 1.25rem 0;
  padding: 0.65rem 0.85rem;
  font-size: 0.92em;
}
.algorithm-title {
  text-align: center;
  font-weight: 700;
  margin-bottom: 0.45rem;
}
.algorithm-box p {
  margin: 0.25rem 0;
}
.algorithm-box ol {
  margin: 0.35rem 0 0.15rem 1.35rem;
}
.algorithm-box li {
  margin: 0.2rem 0;
}
.algorithm-rule {
  border-top: 1px solid #aaa;
  margin: 0.45rem 0;
}
</style>

Diffusion on images is easy to picture: add Gaussian noise, learn how to remove it, then reverse the noising process. Diffusion on tokens is less obvious. There is no token halfway between `cat` and `the`, and adding a real-valued Gaussian to a vocabulary index is usually the wrong geometry.

The clean way to move the idea to discrete data is to replace Gaussian kernels by categorical transition matrices. Let a sequence be $x_0=(x_0^1,\ldots,x_0^L)$ with each token in a vocabulary of size $K$. For now assume the forward process factorizes over positions. At each diffusion step,

<div class="math-display">
$$
q(x_t\mid x_{t-1})
=
\prod_{\ell=1}^L
\operatorname{Cat}\!\left(x_t^\ell;\; x_{t-1}^\ell Q_t\right),
$$
</div>

where $Q_t$ is a row-stochastic matrix and a token is represented as a one-hot row vector. The cumulative transition is

<div class="math-display">
$$
\bar Q_t=Q_1Q_2\cdots Q_t,
\qquad
q(x_t\mid x_0)
=
\prod_{\ell=1}^L
\operatorname{Cat}\!\left(x_t^\ell;\; x_0^\ell \bar Q_t\right).
$$
</div>

The product over $\ell$ is an assumption about the **forward corruption**, not a claim that tokens are semantically independent. During training, the corruption is sampled token-wise independently, but the denoising network can look at the whole corrupted environment around a token: surrounding words in a sentence, neighboring nodes in a graph, residues in a protein sequence, or whatever context the architecture exposes. The factorization says each site has its own categorical transition once that context has been encoded.

This note focuses on two useful choices:

- **Uniform diffusion:** a token is gradually replaced by a random vocabulary token.
- **Masked diffusion:** a token is gradually replaced by a special absorbing `[MASK]` token.

The first feels closest to "add noise everywhere." The second feels closest to "hide more and more of the sequence." Both are discrete diffusion models; they differ mostly in the geometry of the forward corruption.

## 1. Reference Map

A short lineage is useful before the formulas:

- Ho, Jain, and Abbeel, "Denoising Diffusion Probabilistic Models" (2020), gave the now-standard DDPM recipe for continuous data.
- Hoogeboom, Nielsen, Jaini, Forre, and Welling, "Argmax Flows and Multinomial Diffusion" (2021), introduced multinomial diffusion for categorical variables.
- Austin, Johnson, Ho, Tarlow, and van den Berg, "Structured Denoising Diffusion Models in Discrete State-Spaces" (2021), introduced D3PMs and made the transition matrix $Q_t$ the central design object. Their examples include uniform-like, structured, and absorbing-state corruptions.
- Lou, Meng, and Ermon, "Discrete Diffusion Modeling by Estimating the Ratios of the Data Distribution" (2023; ICML 2024), introduced Score Entropy Discrete Diffusion (SEDD), which learns discrete data-density ratios rather than continuous scores.
- Sahoo et al., "Simple and Effective Masked Diffusion Language Models" (2024), and Shi et al., "Simplified and Generalized Masked Diffusion for Discrete Data" (2024), clarified masked/absorbing diffusion objectives and sampling recipes for language-like data.

If you remember only one sentence: D3PM is the general finite-state Markov-chain view; uniform and masked diffusion are two especially useful choices of $Q_t$.

## 2. The D3PM Posterior

Training usually needs the exact forward posterior $q(x_{t-1}\mid x_t,x_0)$. For one token, if $x_0=i$, $x_{t-1}=j$, and $x_t=k$, Bayes' rule gives

<div class="math-display">
$$
\begin{aligned}
q(x_{t-1}=j\mid x_t=k,x_0=i)
&=
\frac{
q(x_t=k\mid x_{t-1}=j)\,
q(x_{t-1}=j\mid x_0=i)
}{
q(x_t=k\mid x_0=i)
} \\
&=
\frac{[\bar Q_{t-1}]_{ij}[Q_t]_{jk}}
{[\bar Q_t]_{ik}}.
\end{aligned}
$$
</div>

The denominator is the total probability of reaching $k$ from $i$ in $t$ steps:

<div class="math-display">
$$
[\bar Q_t]_{ik}
=
\sum_j [\bar Q_{t-1}]_{ij}[Q_t]_{jk}.
$$
</div>

For a sequence, apply this independently per position when the forward noising factorizes. This formula is the discrete analogue of the Gaussian posterior used in DDPM. It is also the object the learned reverse kernel tries to approximate.

A common parameterization is:

1. train a network $p_\theta(\hat x_0\mid x_t,t)$ that predicts the clean token distribution;
2. combine it with the analytic posterior above:

<div class="math-display">
$$
p_\theta(x_{t-1}\mid x_t)
=
\sum_{\hat x_0}
q(x_{t-1}\mid x_t,\hat x_0)\,
p_\theta(\hat x_0\mid x_t,t).
$$
</div>

This full-sequence sum is the formal expression. For length $L$ and vocabulary size $K$, summing over all $\hat x_0\in\{1,\ldots,K\}^L$ is prohibitive. The token-wise forward posterior is what makes the practical computation local.

In a contextual model, this prediction is still made site by site but not in isolation. More explicitly, one often writes

<div class="math-display">
$$
p_\theta(\hat x_0\mid x_t,t)
=
\prod_{\ell=1}^L
p_\theta(\hat x_0^\ell\mid x_t,t,\ell),
$$
</div>

where the factor for site $\ell$ is produced after the network has processed the full corrupted sentence, graph, or sequence. Then the reverse distribution is assembled by a vocabulary-sized sum at each site:

<div class="math-display">
$$
p_\theta(x_{t-1}^\ell=b\mid x_t)
=
\sum_{a=1}^K
q(x_{t-1}^\ell=b\mid x_t^\ell,\hat x_0^\ell=a)\,
p_\theta(\hat x_0^\ell=a\mid x_t,t,\ell).
$$
</div>

This is the practical reason the per-token reverse probabilities are meaningful: the output distribution factorizes, while the features used to compute each factor do not have to. The model avoids a global sum over clean sequences and only sums over possible token identities at each position.

## 3. Uniform Diffusion

Let $u=(1/K,\ldots,1/K)$ be the uniform distribution over the vocabulary. A simple uniform corruption matrix is

<div class="math-display">
$$
Q_t
=
\alpha_t I+(1-\alpha_t)\mathbf 1 u^T,
\qquad
\alpha_t=1-\beta_t .
$$
</div>

From a token $i$, this means

<div class="math-display">
$$
q(x_t=j\mid x_{t-1}=i)
=
\alpha_t\,\mathbf 1\{j=i\}
+
(1-\alpha_t)\frac{1}{K}.
$$
</div>

So with probability $\alpha_t$ the token is copied, and with the remaining mass it is redrawn from the whole vocabulary. The cumulative matrix has the same form:

<div class="math-display">
$$
\bar Q_t
=
\bar\alpha_t I+(1-\bar\alpha_t)\mathbf 1u^T,
\qquad
\bar\alpha_t=\prod_{s=1}^t \alpha_s .
$$
</div>

Here is the quick proof. Let $J=\mathbf 1u^T$. Since $u^T\mathbf 1=1$, we have $J^2=J$. Therefore two uniform kernels multiply as

<div class="math-display">
$$
\begin{aligned}
(\alpha I+(1-\alpha)J)(\eta I+(1-\eta)J)
&=
\alpha\eta I+
\left[
\alpha(1-\eta)+(1-\alpha)\eta+(1-\alpha)(1-\eta)
\right]J \\
&=
\alpha\eta I+(1-\alpha\eta)J.
\end{aligned}
$$
</div>

Applying this identity inductively over $Q_1,\ldots,Q_t$ gives the same form with $\bar\alpha_t=\prod_{s=1}^t\alpha_s$.

At $t=0$, $\bar\alpha_0=1$ and the token is clean. At large $t$, $\bar\alpha_t\approx 0$ and the token is nearly uniform. A schedule can be chosen directly through $\bar\alpha_t$, just as continuous DDPMs often schedule $\bar\alpha_t$ or signal-to-noise ratio. The D3PM inverse-linear choice $\beta_t=1/(T-t+1)$ is a simple discrete schedule that reaches the uniform distribution exactly at the final step because $\beta_T=1$.

### Training

The full variational objective is the discrete diffusion ELBO. For one reverse step it contains

<div class="math-display">
$$
D_{\mathrm{KL}}
\left(
q(x_{t-1}\mid x_t,x_0)
\;\Vert\;
p_\theta(x_{t-1}\mid x_t)
\right).
$$
</div>

In practice D3PM-style training often adds an auxiliary clean-token cross entropy,

<div class="math-display">
$$
\mathcal L_{\mathrm{ce}}
=
-\log p_\theta(x_0\mid x_t,t),
$$
</div>

because predicting $x_0$ directly gives a strong denoising signal. A compact training loop is:

<div class="algorithm-box" markdown="1">
<div class="algorithm-title">Algorithm 1. Uniform Discrete Diffusion Training</div>

**Input:** data law $p_{\rm data}$, transition matrices $\{Q_t\}_{t=1}^T$, timestep law $\pi(t)$, loss weights $\lambda_t,\gamma_t$.

**Repeat until convergence:**

1. Draw $x_0\sim p_{\rm data}$ and $t\sim\pi(t)$.
2. Corrupt directly with the cumulative kernel:
   $x_t\sim q(x_t\mid x_0)=\prod_{\ell=1}^L\operatorname{Cat}(x_t^\ell;\,x_0^\ell\bar Q_t)$.
3. Predict clean-token probabilities:
   $\rho_\theta^\ell(a\mid x_t,t)=p_\theta(\hat x_0^\ell=a\mid x_t,t,\ell)$.
4. Construct each reverse token kernel by marginalizing over vocabulary values:
   $p_\theta(x_{t-1}^\ell=b\mid x_t)=\sum_{a=1}^Kq(x_{t-1}^\ell=b\mid x_t^\ell,\hat x_0^\ell=a)\rho_\theta^\ell(a\mid x_t,t)$.
5. Take a gradient step on
   $\lambda_tD_{\rm KL}\!\left(q(x_{t-1}\mid x_t,x_0)\Vert p_\theta(x_{t-1}\mid x_t)\right)-\gamma_t\sum_{\ell=1}^L\log\rho_\theta^\ell(x_0^\ell\mid x_t,t)$.
</div>

Uniform diffusion is attractive when all tokens are allowed to become all other tokens. It is also natural for categorical images, segmentation maps, amino-acid sequences, or any finite alphabet where a wrong visible symbol should be treated as noisy evidence rather than as a missing value.

### Inference

Sampling starts from the stationary noise distribution and runs the learned reverse chain:

<div class="algorithm-box" markdown="1">
<div class="algorithm-title">Algorithm 2. Uniform Discrete Diffusion Sampling</div>

**Input:** trained predictor $\rho_\theta(\hat x_0\mid x_t,t)$, transition matrices $\{Q_t\}_{t=1}^T$.

1. Initialize $x_T^\ell\sim\operatorname{Unif}(\{1,\ldots,K\})$ independently for $\ell=1,\ldots,L$.
2. **For** $t=T,T-1,\ldots,1$ **do**
   1. Compute $\rho_\theta^\ell(a\mid x_t,t)$ for each position $\ell$ and vocabulary value $a$.
   2. Form $p_\theta(x_{t-1}^\ell=b\mid x_t)=\sum_{a=1}^Kq(x_{t-1}^\ell=b\mid x_t^\ell,\hat x_0^\ell=a)\rho_\theta^\ell(a\mid x_t,t)$.
   3. Sample $x_{t-1}\sim\prod_{\ell=1}^Lp_\theta(x_{t-1}^\ell\mid x_t)$.
3. **Return** $x_0$.
</div>

The model sees a fully populated sequence at every step. Even at high noise, every position contains some token. That makes the problem different from masked language modeling: the model must decide which visible symbols are meaningful and which are accidental substitutions.

## 4. Masked Diffusion

Masked diffusion adds a special absorbing state $m=[\mathrm{MASK}]$. For an ordinary token $i$, define

<div class="math-display">
$$
q(x_t=i\mid x_{t-1}=i)=\alpha_t,
\qquad
q(x_t=m\mid x_{t-1}=i)=1-\alpha_t,
$$
</div>

and make the mask absorbing:

<div class="math-display">
$$
q(x_t=m\mid x_{t-1}=m)=1.
$$
</div>

Equivalently, with $\beta_t=1-\alpha_t$, every unmasked token is independently masked with probability $\beta_t$ at step $t$. The cumulative marginal is especially simple:

<div class="math-display">
$$
q(x_t^\ell=x_0^\ell\mid x_0^\ell)=\bar\alpha_t,
\qquad
q(x_t^\ell=m\mid x_0^\ell)=1-\bar\alpha_t.
$$
</div>

Unlike uniform diffusion, a visible token is never changed into another visible token. It is either the original token or the mask. This is why absorbing diffusion is so close to iterative masked language modeling.

The exact posterior for a masked position is also simple. If $x_t=m$, then

<div class="math-display">
$$
q(x_{t-1}=x_0\mid x_t=m,x_0)
=
\frac{\bar\alpha_{t-1}\beta_t}{1-\bar\alpha_t},
\qquad
q(x_{t-1}=m\mid x_t=m,x_0)
=
\frac{1-\bar\alpha_{t-1}}{1-\bar\alpha_t}.
$$
</div>

If $x_t$ is visible, then $x_{t-1}=x_t$ with probability one.

### Training

The most direct objective is masked-token prediction. Sample a noise level, mask each token with probability $1-\bar\alpha_t$, and train the network to recover the clean identities of the masked tokens:

<div class="math-display">
$$
\mathcal L_{\mathrm{mask}}
=
E_{x_0,t,x_t}
\left[
w(t)
\sum_{\ell:x_t^\ell=m}
-\log p_\theta(x_0^\ell\mid x_t,t)
\right].
$$
</div>

The weight $w(t)$ depends on whether one uses a discrete-time ELBO, a continuous-time limit, or a Rao-Blackwellized variant. The important structural point is stable: the prediction target is the original clean token at masked sites. This is why modern masked diffusion objectives can look like mixtures of classical masked language modeling losses, with the mask rate playing the role of diffusion time.

<div class="algorithm-box" markdown="1">
<div class="algorithm-title">Algorithm 3. Masked Discrete Diffusion Training</div>

**Input:** data law $p_{\rm data}$, survival schedule $\bar\alpha_t$, timestep law $\pi(t)$, weight $w(t)$.

**Repeat until convergence:**

1. Draw $x_0\sim p_{\rm data}$ and $t\sim\pi(t)$.
2. For each position $\ell$, sample $b_\ell\sim\operatorname{Bernoulli}(\bar\alpha_t)$ and set
   $x_t^\ell=b_\ell x_0^\ell+(1-b_\ell)m$, where $m=[\mathrm{MASK}]$.
3. Predict clean tokens at masked positions:
   $\rho_\theta^\ell(\cdot\mid x_t,t)=p_\theta(x_0^\ell=\cdot\mid x_t,t)$.
4. Take a gradient step on
   $\displaystyle w(t)\sum_{\ell:x_t^\ell=m}-\log\rho_\theta^\ell(x_0^\ell\mid x_t,t)$.
</div>

This objective is usually easier to implement than the full uniform D3PM posterior. It also matches the inductive bias of bidirectional Transformers: condition on all currently visible context and predict the missing pieces.

### Inference

The ancestral sampler uses the posterior probabilities above. A practical sampler often reveals a controlled number of tokens per step, sometimes choosing the most confident predictions first.

<div class="algorithm-box" markdown="1">
<div class="algorithm-title">Algorithm 4. Masked Discrete Diffusion Sampling</div>

**Input:** trained predictor $\rho_\theta^\ell(\hat x_0^\ell\mid x_t,t)$, decreasing survival schedule $\bar\alpha_T,\ldots,\bar\alpha_0$.

1. Initialize $x_T^\ell=m$ for all positions $\ell=1,\ldots,L$.
2. **For** $t=T,T-1,\ldots,1$ **do**
   1. Compute $\rho_\theta^\ell(\hat x_0^\ell\mid x_t,t)$ for each masked position.
   2. Set the reveal probability
      $r_t=q(x_{t-1}=x_0\mid x_t=m,x_0)=\bar\alpha_{t-1}\beta_t/(1-\bar\alpha_t)$.
   3. Choose a reveal set $R_t\subseteq\{\ell:x_t^\ell=m\}$ by Bernoulli sampling with rate $r_t$, or by matching the target mask rate $1-\bar\alpha_{t-1}$.
   4. For $\ell\in R_t$, sample $x_{t-1}^\ell\sim\rho_\theta^\ell(\cdot\mid x_t,t)$; for $\ell\notin R_t$, set $x_{t-1}^\ell=m$.
   5. Keep visible coordinates fixed: if $x_t^\ell\ne m$, set $x_{t-1}^\ell=x_t^\ell$.
3. **Return** $x_0$.
</div>

The simplest absorbing sampler is monotone: once a token is revealed, it stays revealed. More flexible samplers allow remasking and resampling, which gives the model a way to revise earlier decisions. That revision step moves masked diffusion closer in spirit to uniform diffusion, where every visible token can be questioned throughout the reverse chain.

## 5. Connecting the Two Setups

Uniform and masked diffusion are not separate theories. They are two choices of a categorical noising matrix.

Uniform diffusion has a stationary distribution over ordinary tokens. As time increases, the observation becomes a dense sequence of random symbols. The reverse model must perform denoising from corrupted evidence.

Masked diffusion has an absorbing state. As time increases, the observation loses coordinates rather than replacing them by random alternatives. The reverse model performs conditional infilling from partial evidence.

The same D3PM posterior formula covers both. The difference is what information survives in $x_t$:

| Setup | High-noise state | What a visible token means | Natural training signal |
|---|---|---|---|
| Uniform | random vocabulary tokens | noisy evidence, possibly wrong | denoise substitutions / predict $x_0$ |
| Masked | mostly `[MASK]` | reliable evidence if not masked | predict missing clean tokens |

There is also a useful modeling tradeoff. Uniform diffusion gives every position a value at every step, so the model can revise all positions throughout sampling. But the network must learn under noisy false tokens. Masked diffusion gives cleaner conditioning context, but a monotone sampler can commit too early unless confidence scheduling or remasking is used.

For language, masked diffusion often feels more natural because text already has a strong infilling interpretation. For scientific categorical states, uniform diffusion can be more symmetric: one spin, atom type, residue, or cluster label can be corrupted into another without introducing a special missing symbol. In applications, the best choice is often about what "noise" should mean physically.

## 6. Relation to Discrete Scores

Continuous diffusion learns the score $\nabla_x\log p_t(x)$. On a finite state space there is no ordinary gradient with respect to token identity. One replacement is to learn probability ratios between neighboring states, for example quantities like

<div class="math-display">
$$
s_t(x,y)\approx \frac{p_t(y)}{p_t(x)}.
$$
</div>

This is the perspective behind SEDD. It is especially clean when the discrete state space has a graph structure: $y$ is a one-token edit, a nearest neighbor, or another allowed move from $x$. The reverse dynamics can then be written in terms of learned ratios rather than a direct clean-token predictor.

The clean-token-prediction view and the ratio view are connected. Both are ways to parameterize the reverse process. Predicting $x_0$ asks, "which clean state could have produced this corrupted state?" Ratio estimation asks, "which nearby discrete moves increase the noised data probability?" In continuous diffusion these viewpoints collapse toward score-based denoising identities. In discrete diffusion they remain visibly different because the state space has jumps rather than infinitesimal directions.

## 7. Practical Schedule Choices

A schedule should make the endpoint easy and the intermediate tasks learnable.

For uniform diffusion:

- choose $\bar\alpha_0=1$ and $\bar\alpha_T\approx 0$;
- sample $x_t$ from $\bar Q_t=\bar\alpha_tI+(1-\bar\alpha_t)\mathbf 1u^T$;
- use a schedule with enough mid-noise steps that the model sees partially informative corruptions;
- consider auxiliary clean-token cross entropy because the exact KL alone can be weak or awkward.

For masked diffusion:

- choose a survival probability $\bar\alpha_t$ decreasing from $1$ to $0$;
- train across mask rates, not just one fixed BERT-style mask rate;
- use loss weights if matching a specific ELBO or continuous-time objective;
- during sampling, decide whether tokens are revealed monotonically or may be remasked and revised.

One nice mental model is:

<div class="math-display">
$$
\text{uniform diffusion: corrupt by substitution,}
\qquad
\text{masked diffusion: corrupt by deletion of information.}
$$
</div>

Both become generative models by learning how to reverse that corruption.

## References

- Ho, Jain, and Abbeel, "Denoising Diffusion Probabilistic Models" (2020): [arXiv:2006.11239](https://arxiv.org/abs/2006.11239).
- Hoogeboom, Nielsen, Jaini, Forre, and Welling, "Argmax Flows and Multinomial Diffusion: Learning Categorical Distributions" (2021): [arXiv:2102.05379](https://arxiv.org/abs/2102.05379).
- Austin, Johnson, Ho, Tarlow, and van den Berg, "Structured Denoising Diffusion Models in Discrete State-Spaces" (2021): [NeurIPS](https://papers.nips.cc/paper_files/paper/2021/hash/958c530554f78bcd8e97125b70e6973d-Abstract.html).
- Lou, Meng, and Ermon, "Discrete Diffusion Modeling by Estimating the Ratios of the Data Distribution" (2023; ICML 2024): [arXiv:2310.16834](https://arxiv.org/abs/2310.16834).
- Sahoo, Arriola, Schiff, Gokaslan, Marroquin, Chiu, Rush, and Kuleshov, "Simple and Effective Masked Diffusion Language Models" (2024): [arXiv:2406.07524](https://arxiv.org/abs/2406.07524).
- Shi, Han, Wang, Doucet, and Titsias, "Simplified and Generalized Masked Diffusion for Discrete Data" (2024): [arXiv:2406.04329](https://arxiv.org/abs/2406.04329).
