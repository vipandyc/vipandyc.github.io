---
permalink: /blog/doob-h-transform/
title: "The Doob h-Transform: From Conditioned Random Walks to Guided SDEs"
excerpt: "A derivation of the Doob h-transform from Bayes' rule, through Markov chains and jump processes, to diffusion drifts, Brownian bridges, committors, and rare-event sampling."
author_profile: true
---

# The Doob $h$-Transform: From Conditioned Random Walks to Guided SDEs

Suppose a stochastic process usually fails at a task, but we want to study the paths on which it succeeds. A particle might cross an energy barrier before returning to its starting basin, or Brownian motion might arrive at a specified endpoint at a specified time. Repeatedly simulating the original process and discarding failures is exact, but can be hopelessly expensive when success is rare.

The **Doob $h$-transform** turns the future success condition into a local rule for the next step. For a Markov chain, it changes transition probabilities. For a diffusion, it adds a state-dependent drift. When $h$ is the *exact* probability of the desired event, the new process produces the original process's paths **conditioned on that event**. The randomness is still there; only the path law has changed.

This note derives that statement rather than treating the added drift as a recipe to memorize. We will move from a six-state random walk to continuous-time jumps, then to SDEs and the committor used in rare-event sampling.

## 1. Start with Bayes' Rule

Let $X_0,X_1,\ldots$ be a Markov chain with transition matrix $P_{ij}=\Pr(X_{n+1}=j\mid X_n=i)$. Let $E$ be a future event, such as "hit $B$ before $A$." Write its probability for a chain started at state $i$ as

<div class="math-display">
$$
h(i)=\Pr(E\mid X_0=i).
$$
</div>

For a hitting event, before the chain has reached either boundary, the Markov property gives

<div class="math-display">
$$
\Pr(E\mid X_n=i,X_{n+1}=j)=h(j).
$$
</div>

Bayes' rule now answers the central question: **given that success will occur, what is the probability of the next move?**

<div class="math-display">
$$
\begin{aligned}
P_{\rm cond}(j\mid i)
&=\Pr(X_{n+1}=j\mid X_n=i,E)\\
&=\frac{\Pr(X_{n+1}=j,E\mid X_n=i)}{\Pr(E\mid X_n=i)}\\
&=P_{ij}\frac{h(j)}{h(i)},\qquad h(i)>0.
\end{aligned}
$$
</div>

The factor $h(j)/h(i)$ rewards moves into states from which success is more likely. This is a probability kernel because the law of total probability gives

<div class="math-display">
$$
h(i)=\sum_j P_{ij}h(j),
\qquad\text{so}\qquad
\sum_j P_{\rm cond}(j\mid i)=1.
$$
</div>

The averaging equation says that $h$ is **harmonic** for the chain in the region where the event has not yet been decided. For a hitting event, we stop at the first visit to $A$ or $B$, set $h=0$ on $A$ and $h=1$ on $B$, and use the averaging equation only before either set is reached. States with $h=0$ cannot be starting states of the conditioned process.

### Why the *whole path* has the right distribution

For a path $i_0\to i_1\to\cdots\to i_m$, multiply its transformed step probabilities:

<div class="math-display">
$$
\begin{aligned}
\Pr_{\rm cond}(\text{path}\mid X_0=i_0)
&=\prod_{k=0}^{m-1}P_{i_k i_{k+1}}\frac{h(i_{k+1})}{h(i_k)}\\
&=\left(\prod_{k=0}^{m-1}P_{i_k i_{k+1}}\right)
\frac{h(i_m)}{h(i_0)}.
\end{aligned}
$$
</div>

All intermediate $h$ factors cancel. If the path stops on success, $h(i_m)=1$, and its probability is the original path probability divided by $\Pr(E\mid X_0=i_0)$. That is exactly the probability of the path conditional on $E$. If it stops on failure, $h(i_m)=0$, so the transformed path has zero probability.

This telescoping calculation is the reason the transform is more than a heuristic force pointing toward $B$: with the correct $h$, it preserves the *relative probabilities among successful paths*.

## 2. A Random Walk You Can Calculate by Hand

Take the simple symmetric random walk on $\{0,1,2,3,4,5\}$. States $0$ and $5$ are absorbing, and each interior step goes left or right with probability $1/2$. We want the paths that reach $5$ before $0$.

The success probability $h(i)$ obeys

<div class="math-display">
$$
h(0)=0,\qquad h(5)=1,\qquad
h(i)=\tfrac12h(i-1)+\tfrac12h(i+1),\quad 1\le i\le4.
$$
</div>

The interior equation says the first difference $h(i+1)-h(i)$ is constant. The boundary values then give $h(i)=i/5$. Starting at $i=2$, success has probability $2/5$. The conditioned step probabilities are

<div class="math-display">
$$
P_{\rm cond}(3\mid2)=\frac12\frac{h(3)}{h(2)}=\frac34,
\qquad
P_{\rm cond}(1\mid2)=\frac12\frac{h(1)}{h(2)}=\frac14.
$$
</div>

Moving right is now favored, but moving left is still possible. At state $1$, however, the step to $0$ has probability zero: conditioned on eventual success, the process cannot take a path that has already failed.

As a check, consider the particular path $2\to3\to4\to5$. Under the original walk it has probability $1/8$. Conditional on success, its probability is $(1/8)/(2/5)=5/16$. The transformed walk gives the same answer directly:

<div class="math-display">
$$
P_{\rm cond}(3\mid2)P_{\rm cond}(4\mid3)P_{\rm cond}(5\mid4)
=\frac34\cdot\frac23\cdot\frac58
=\frac5{16}.
$$
</div>

The same pathwise calculation also shows why an approximate $h_{\rm approx}$ does **not** automatically give exact conditional paths. If $h_{\rm approx}(i)\ne\sum_jP_{ij}h_{\rm approx}(j)$, the numbers $P_{ij}h_{\rm approx}(j)/h_{\rm approx}(i)$ do not even sum to one without further normalization. An approximation can still be a useful proposal for enhanced sampling, but it needs a correction or path weights when exact expectations are required.

## 3. A Deadline Makes $h$ Time Dependent

Now ask a different question: will the chain satisfy a terminal condition at step $N$? Write $g(X_N)$ for its terminal payoff; for an event, $g$ is its indicator. The appropriate function is

<div class="math-display">
$$
h_n(i)=\mathbb E[g(X_N)\mid X_n=i],
\qquad h_N(i)=g(i).
$$
</div>

Working backward one step gives the **backward equation**

<div class="math-display">
$$
h_n(i)=\sum_jP_{ij}h_{n+1}(j).
$$
</div>

Therefore the conditioned transition at step $n$ is

<div class="math-display">
$$
P_{\rm cond}(j\mid i,n)=P_{ij}\frac{h_{n+1}(j)}{h_n(i)}.
$$
</div>

Its explicit dependence on $n$ matters. If the deadline is close, the next move may need to be more strongly directed than it was earlier. Multiplying the kernels again telescopes to $g(i_N)/h_0(i_0)$ times the original path probability. For an indicator $g$, this is ordinary conditioning; for a nonnegative payoff $g$, it is a **terminally tilted** path law.

## 4. Continuous-Time Markov Jump Processes

There is a useful middle step between discrete chains and SDEs. Let a continuous-time chain jump from $i$ to $j$ at rate $k_{ij}$. Its generator $L$ gives the average instantaneous rate of change of a function $f$:

<div class="math-display">
$$
(Lf)(i)=\sum_{j\ne i}k_{ij}\,[f(j)-f(i)].
$$
</div>

For a hitting committor $h$, $Lh=0$ in the interior. The transformed generator is

<div class="math-display">
$$
(L_{\rm cond} f)(i)=\frac{L(hf)(i)-f(i)Lh(i)}{h(i)}.
$$
</div>

Expanding both sums shows exactly what changes:

<div class="math-display">
$$
\begin{aligned}
(L_{\rm cond} f)(i)
&=\sum_{j\ne i}k_{ij}\frac{h(j)}{h(i)}[f(j)-f(i)],\\
k_{\rm cond}(i,j)&=k_{ij}\frac{h(j)}{h(i)}.
\end{aligned}
$$
</div>

Thus the discrete probability ratio becomes a **jump-rate ratio**. A jump toward a state with larger success probability is accelerated; a jump into $h=0$ is removed. For a harmonic hitting committor, the *total* exit rate actually stays fixed:

<div class="math-display">
$$
\sum_{j\ne i}k_{\rm cond}(i,j)
=\sum_{j\ne i}k_{ij}+\frac{Lh(i)}{h(i)}
=\sum_{j\ne i}k_{ij}.
$$
</div>

So in this case the waiting time at $i$ is unchanged, while the destination of the next jump is reweighted. A time-dependent or nonharmonic transform can also change the waiting time.

## 5. The Diffusion Limit: Why the Extra Drift Is $a\nabla\log h$

Consider an Itô diffusion in $\mathbb R^d$,

<div class="math-display">
$$
\mathrm dX_t=b(t,X_t)\,\mathrm dt+\sigma(t,X_t)\,\mathrm dW_t,
\qquad a(t,x)=\sigma(t,x)\sigma(t,x)^\top.
$$
</div>

Its generator, acting on a smooth test function $f$, is

<div class="math-display">
$$
(L_t f)(x)=b(t,x)\cdot\nabla f(x)
+\frac12\sum_{i,j}a_{ij}(t,x)\,\partial_i\partial_j f(x).
$$
</div>

For a terminal event $E$, let $h(t,x)=\Pr(E\mid X_t=x)$, or use a transition **density** when $E$ specifies an exact continuous endpoint. Assume $h>0$ and smooth in the region of interest. Its backward equation is

<div class="math-display">
$$
\partial_t h+L_t h=0,
$$
</div>

with terminal or boundary data set by the event. The finite-time transition kernel changes by the same Bayes factor as before:

<div class="math-display">
$$
p_{\rm cond}(t,y\mid s,x)
=p(t,y\mid s,x)\frac{h(t,y)}{h(s,x)},\qquad s<t.
$$
</div>

To identify the SDE, expand the transformed generator. The ordinary product rule gives $\nabla(hf)=h\nabla f+f\nabla h$. For the second-order term, the two cross derivatives combine because $a$ is symmetric:

<div class="math-display">
$$
L_t(hf)=hL_t f+fL_t h+(a\nabla h)\cdot\nabla f.
$$
</div>

Consequently,

<div class="math-display">
$$
\begin{aligned}
L_{\rm cond} f
&=\frac{L_t(hf)-fL_t h}{h}\\
&=\left(b+a\frac{\nabla h}{h}\right)\cdot\nabla f
+\frac12\sum_{i,j}a_{ij}\partial_i\partial_j f\\
&=\left(b+a\nabla\log h\right)\cdot\nabla f
+\frac12\sum_{i,j}a_{ij}\partial_i\partial_j f.
\end{aligned}
$$
</div>

The second-order term is unchanged. Reading off the first-order term, the transformed process satisfies

<div class="math-display">
$$
\boxed{\mathrm dX_t=\left[b(t,X_t)+a(t,X_t)\nabla_x\log h(t,X_t)\right]\mathrm dt
+\sigma(t,X_t)\,\mathrm dW_t^{\rm cond}.}
$$
</div>

$W^{\rm cond}$ is Brownian motion under the conditioned law. The noise covariance $a$ stays the same; only the drift changes. If one writes $D=a/2$, the extra drift is $2D\nabla\log h$. This factor of two is easy to lose when switching between $a$ and the physics convention $D$.

The backward equation ensures that the kernel above is normalized. The subtraction of $fL_t h$ in the generator removes a zero-order term; it is useful when deriving the formula even in a region where $L_t h$ is not separately zero. For time-dependent $h$, the relevant harmonic equation is $\partial_t h+L_t h=0$, not $L_t h=0$ by itself.

### The same drift from a path likelihood ratio

There is a second derivation that connects the transform to Girsanov's theorem. Apply Itô's formula to $\log h(t,X_t)$ under the original SDE. Since $(\partial_t+L_t)h=0$,

<div class="math-display">
$$
\mathrm d\log h(t,X_t)
=u_t\cdot\mathrm dW_t-\tfrac12(u_t\cdot u_t)\,\mathrm dt,
\qquad
u_t=\sigma(t,X_t)^\top\nabla\log h(t,X_t).
$$
</div>

Integrate from $0$ to $t$ and exponentiate:

<div class="math-display">
$$
\frac{h(t,X_t)}{h(0,X_0)}
=\exp\!\left(
\int_0^t u_s\cdot\mathrm dW_s
-\frac12\int_0^t(u_s\cdot u_s)\,\mathrm ds
\right).
$$
</div>

The left side is the likelihood ratio for the conditioned path up to time $t$. Under appropriate martingale and integrability conditions, Girsanov's theorem says that $W_t^{\rm cond}=W_t-\int_0^t u_s\,\mathrm ds$ is Brownian motion under this new law. Substituting $\mathrm dW_t=\mathrm dW_t^{\rm cond}+u_t\,\mathrm dt$ into the original SDE adds $\sigma u_t=\sigma\sigma^\top\nabla\log h=a\nabla\log h$ to its drift, agreeing with the generator calculation. For a hitting event, read these identities up to the stopping time; for an exact endpoint bridge, use times $t<T$.

## 6. Brownian Bridge: Conditioning on an Endpoint

Let $\mathrm dX_t=\mathrm dW_t$, start at $X_0=x_0$, and require $X_T=y$. An exact point has probability zero under a continuous density, so here $h$ is the transition *density* to $y$:

<div class="math-display">
$$
h(t,x)=p(T,y\mid t,x)
=\frac{1}{\sqrt{2\pi(T-t)}}
\exp\!\left[-\frac{(y-x)^2}{2(T-t)}\right],\qquad t<T.
$$
</div>

Differentiate its logarithm with respect to $x$:

<div class="math-display">
$$
\partial_x\log h(t,x)=\frac{y-x}{T-t}.
$$
</div>

Since $b=0$ and $a=1$, the bridge SDE is

<div class="math-display">
$$
\mathrm dX_t=\frac{y-X_t}{T-t}\,\mathrm dt+\mathrm dW_t^{\rm cond},
\qquad t<T.
$$
</div>

Early in the path, the pull toward $y$ is mild. As $t\uparrow T$, the remaining time shrinks and the pull grows. The path remains random at every time before $T$, but its endpoint is pinned. The same formula follows from the Gaussian conditional law: given $X_t=x$ and $X_T=y$, the expected displacement over a short interval $\Delta t$ is approximately $(y-x)\Delta t/(T-t)$.

Notice the difference from the random walk example. The Brownian bridge targets a **specified time and endpoint**, so its $h(t,x)$ depends on time. A committor targets **which set is hit first**, with no fixed deadline, so its $h(x)$ can be time independent.

### Conditional diffusion generation

The same construction explains the ideal form of guidance in a generative diffusion sampler. Write the sampler in a time coordinate that **increases in the direction it is simulated**, with base drift $b$ and covariance $a$. If $Y$ is a desired property of the final sample, define $h(t,x)=\Pr(Y=y\mid X_t=x)$, using a conditional density when $Y$ is continuous. The exact conditional sampler adds

<div class="math-display">
$$
a(t,x)\nabla_x\log h(t,x)
=a(t,x)\nabla_x\log\Pr(Y=y\mid X_t=x)
$$
</div>

to the base drift. This is the mathematical version of classifier guidance: the gradient points toward intermediate states more likely to produce the requested property at the end. The formula is exact only for the true conditional likelihood and the correct time convention. An approximate classifier, or an arbitrarily scaled guidance term, changes the sampling law and generally loses the exact conditioning guarantee.

## 7. The Committor and Reactive Paths

Let $A$ and $B$ be disjoint sets. The **forward committor** is

<div class="math-display">
$$
q(x)=\Pr(\text{reach }B\text{ before }A\mid X_0=x).
$$
</div>

It solves the boundary-value problem

<div class="math-display">
$$
Lq=0\quad\text{outside }A\text{ and }B,
\qquad q(x)=0\ (x\in A),\qquad q(x)=1\ (x\in B).
$$
</div>

Why $Lq=0$? Over a short time $\Delta t$, the Markov property says $q(x)=\mathbb E[q(X_{\Delta t})\mid X_0=x]$ before hitting either set. Subtract $q(x)$, divide by $\Delta t$, and take $\Delta t\to0$. By the definition of the generator, the result is $Lq(x)=0$.

For overdamped Langevin dynamics with scalar mobility absorbed into the time unit,

<div class="math-display">
$$
\mathrm dX_t=-\nabla U(X_t)\,\mathrm dt
+\sqrt{2\beta^{-1}}\,\mathrm dW_t,
\qquad \beta^{-1}=k_{\rm B}T,
$$
</div>

we have $a=2\beta^{-1}I$. Taking $h=q$ therefore gives

<div class="math-display">
$$
\boxed{\mathrm dX_t=
\left[-\nabla U(X_t)+2\beta^{-1}\nabla\log q(X_t)\right]\mathrm dt
+\sqrt{2\beta^{-1}}\,\mathrm dW_t^{\rm cond}.}
$$
</div>

This is the precise sense in which the Doob transform is **enhanced sampling guided by the committor**. It samples the original dynamics conditioned on reaching $B$ before $A$, stopping when $B$ is first reached. It targets a distribution of *paths*, not merely a flatter equilibrium distribution of configurations.

### One-dimensional committor with an explicit solution

Let $A=\{a\}$ and $B=\{b\}$, with $a<x<b$, and use $U=U(x)$. The backward equation becomes

<div class="math-display">
$$
-U'(x)q'(x)+\beta^{-1}q''(x)=0,
\qquad q(a)=0,\quad q(b)=1.
$$
</div>

Multiply by $\beta$ and rearrange:

<div class="math-display">
$$
q''-\beta U'q'=0
\quad\Longleftrightarrow\quad
\frac{\mathrm d}{\mathrm dx}\!\left(e^{-\beta U}q'\right)=0.
$$
</div>

Hence $q'(x)=C e^{\beta U(x)}$. Integrating and applying both boundary conditions yields

<div class="math-display">
$$
\boxed{q(x)=
\frac{\displaystyle\int_a^x e^{\beta U(z)}\,\mathrm dz}
{\displaystyle\int_a^b e^{\beta U(z)}\,\mathrm dz}.}
$$
</div>

The conditioned drift is then fully explicit:

<div class="math-display">
$$
b_{\rm cond}(x)=-U'(x)
+2\beta^{-1}\frac{e^{\beta U(x)}}{\displaystyle\int_a^x e^{\beta U(z)}\,\mathrm dz}.
$$
</div>

As a simple check, set $U=0$ and choose unit Brownian noise, $\mathrm dX_t=\mathrm dW_t$, on $(0,L)$. Then $q(x)=x/L$, $a=1$, and the transformed drift is $1/x$. Near the failure boundary $x=0$, it strongly repels the process. This is the continuum analogue of the random walk's zero probability of stepping from state $1$ to state $0$.

The drift becomes singular where $q=0$. The ideal conditioned process is defined where $q>0$; starting *exactly* on the boundary of $A$ requires an entrance-law or limiting construction. In simulations one usually starts just outside $A$ or uses a regularized committor, while remembering that regularization changes the exact law.

## 8. What Is Gained, and What Is Not

The exact $q$-transform makes successful trajectories typical under the new dynamics. For every event $F$ determined by the path before stopping,

<div class="math-display">
$$
\Pr_{\rm cond}(F\mid X_0=x)
=\Pr(F\mid\text{reach }B\text{ before }A,\ X_0=x).
$$
</div>

This does **not** mean that a collection of $q$-transformed paths alone tells us the original transition frequency. The conditioning has divided out the success probability $q(x)$, just as the random walk path probability was divided by $2/5$. A rate calculation also needs information about how often trajectories leave $A$, their starting distribution or flux, and the success probability under the original dynamics. Those are separate kinetic quantities.

There is also a computational catch: finding the exact committor in a high-dimensional system can be as hard as the rare-event problem itself. A learned $q_{\rm approx}$ can make a very good proposal, but the exact-path claim no longer holds automatically. To estimate unbiased observables, one must account for the actual proposal law, for example with likelihood-ratio reweighting or another valid correction. Near $q=0$, even a small absolute error can produce a large error in $\nabla\log q$.

The relation to change of measure is clean. For a positive-probability event $E$, an initial point $x$, and a path event $F$ observed by time $t$, Bayes' rule gives

<div class="math-display">
$$
\Pr(F\mid E,X_0=x)
=\frac{\Pr(F\mid X_0=x)\,
\mathbb E[h(t,X_t)\mid F,X_0=x]}{h(0,x)},
$$
</div>

up to the relevant stopping time. In words, a path segment is reweighted by its remaining chance of success, $h(t,X_t)$, divided by the initial chance $h(0,x)$. The SDE drift $a\nabla\log h$ is the local change of dynamics that realizes this reweighting. For a soft terminal weight $g(X_T)>0$, replace $h(t,x)$ by $\mathbb E[g(X_T)\mid X_t=x]$; the path law is then weighted by $g(X_T)/h(0,x)$ rather than conditioned on a binary event.

One can also see why this is an optimal-control construction. Let $P$ be the original path distribution, $P_{\rm cond}$ its distribution conditional on $E$, and $Q$ any other path distribution that always satisfies $E$. A successful path has conditional probability equal to its original probability divided by $\Pr(E)$, so

<div class="math-display">
$$
\mathrm{KL}(Q\|P)
=\mathrm{KL}(Q\|P_{\rm cond})-\log\Pr(E)
\ge -\log\Pr(E).
$$
</div>

Here $\mathrm{KL}$ is the Kullback-Leibler divergence, which is nonnegative. Equality holds at $Q=P_{\rm cond}$. Among path laws that enforce the event, the conditional law changes the original path measure by the least relative entropy. When realizable by a controlled diffusion with the same noise, its drift correction is precisely $a\nabla\log h$.

## 9. A Short Map of the Formulas

| Setting | Future information in $h$ | Local rule after conditioning |
|---|---|---|
| Discrete chain | $h(i)=\Pr(E\mid X_0=i)$ | $P_{\rm cond}(j\mid i)=P_{ij}h(j)/h(i)$ |
| Chain with deadline | $h_n(i)=\mathbb E[g(X_N)\mid X_n=i]$ | $P_{\rm cond}(j\mid i,n)=P_{ij}h_{n+1}(j)/h_n(i)$ |
| Continuous-time jumps | $Lh=0$ in the interior | $k_{\rm cond}(i,j)=k_{ij}h(j)/h(i)$ |
| Diffusion | $\partial_t h+L_t h=0$ | $b_{\rm cond}=b+a\nabla\log h$ |
| Reactive paths | $q(x)=\Pr(\text{reach }B\text{ before }A\mid X_0=x)$ | Add $a\nabla\log q$ until $B$ is hit |

The through-line is Bayes' rule. A move becomes more likely in proportion to how much it improves the chance of the future condition. The backward equation makes those local probabilities consistent; telescoping makes the resulting *entire path law* the correct conditional law.

## References and Related Notes

- E and Vanden-Eijnden, ["Towards a Theory of Transition Paths"](https://collaborate.princeton.edu/en/publications/towards-a-theory-of-transition-paths/) (2006), develops the committor-based statistics of reactive trajectories.
- Lu and Nolen, ["Reactive Trajectories and the Transition Path Process"](https://doi.org/10.1007/s00440-014-0547-y) (2015), constructs the transition-path process using the Doob transform.
- Metzner, Schütte, and Vanden-Eijnden, ["Transition Path Theory for Markov Jump Processes"](https://epubs.siam.org/doi/10.1137/070699500) (2009), develops the discrete-state setting.
- Gao, Liu, and Tse, ["Optimal Control Formulation of Transition Path Problems for Markov Jump Processes"](https://arxiv.org/abs/2311.07795) (2023), discusses the entropy-optimal control view.
- For background on path likelihood ratios, see [Radon-Nikodym, Girsanov, and Diffusion Flow Cheat Sheet](/blog/radon-nikodym-girsanov-diffusion-flows/).
