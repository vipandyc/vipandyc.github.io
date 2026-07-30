---
permalink: /blog/calculate-free-energy-ti-fep-mbar-sscha/
title: "How to Calculate Free Energy: TI, FEP, MBAR, and SSCHA"
excerpt: "A compact methods note on thermodynamic integration, free-energy perturbation, MBAR, alchemical transformations, and the variational route to SSCHA."
author_profile: true
---

# How to Calculate Free Energy: TI, FEP, MBAR, and SSCHA

Free energy is the thermodynamic quantity that tells us what a system prefers after energy and entropy have both had their say. In atomistic simulation it is also awkward: the partition function is usually unknown, phase space is high dimensional, and the important quantity is often a difference between two states rather than an absolute number.

This note is a compact map of four related tools: thermodynamic integration (TI), free-energy perturbation (FEP), the multistate Bennett acceptance ratio (MBAR), and the stochastic self-consistent harmonic approximation (SSCHA). The first three estimate free-energy differences by sampling ensembles. SSCHA starts from a variational principle and turns anharmonic lattice free energy into an optimized harmonic reference problem.

## 1. The Common Object

Let a classical system have coordinates $x$, potential energy $U(x)$, inverse temperature $\beta=1/(k_BT)$, and configurational partition function $Z=\int dx\,\exp[-\beta U(x)]$. The Helmholtz free energy is

<div class="math-display">
$$
F=-\beta^{-1}\log Z .
$$
</div>

We almost never need $F$ alone. We need $\Delta F=F_B-F_A$: a phase stability, a binding free energy, a defect formation free energy, a correction from one Hamiltonian to another, or the free-energy cost of turning one chemical species into another.

Introduce a path of potentials $U_\lambda(x)$ with $\lambda\in[0,1]$, where $U_0=U_A$ and $U_1=U_B$. The normalized ensemble at $\lambda$ is $p_\lambda(x)=\exp[-\beta U_\lambda(x)]/Z_\lambda$, and averages over it are written $\langle\cdots\rangle_\lambda$.

## 2. Thermodynamic Integration

TI differentiates the free energy along the path. Since $F_\lambda=-\beta^{-1}\log Z_\lambda$, the whole method comes from differentiating the partition function rather than estimating $Z_\lambda$ itself.

**Proof note.**
Differentiate under the integral sign, then substitute into $\partial_\lambda F_\lambda=-\beta^{-1}(\partial_\lambda Z_\lambda/Z_\lambda)$. The ratio that appears is exactly a Boltzmann average in the $\lambda$ ensemble:

<div class="math-display">
$$
\begin{aligned}
\partial_\lambda Z_\lambda
&=
\int dx\,[-\beta\,\partial_\lambda U_\lambda(x)]e^{-\beta U_\lambda(x)},\\
\partial_\lambda F_\lambda
&=
\frac{1}{Z_\lambda}
\int dx\,\partial_\lambda U_\lambda(x)e^{-\beta U_\lambda(x)} .
\end{aligned}
\notag
$$
</div>

This gives the TI integrand:

<div class="math-display">
$$
\frac{dF_\lambda}{d\lambda}
=
\left\langle \frac{\partial U_\lambda}{\partial\lambda}\right\rangle_\lambda .
$$
</div>

The free-energy difference follows by integrating this derivative from $\lambda=0$ to $\lambda=1$. No new statistical identity is needed here; it is just the fundamental theorem of calculus applied to the scalar function $F_\lambda$:

<div class="math-display">
$$
\Delta F_{A\to B}
=
\int_0^1 d\lambda\,
\left\langle \partial_\lambda U_\lambda\right\rangle_\lambda .
$$
</div>

In practice, choose a schedule $\lambda_0,\ldots,\lambda_K$, sample each ensemble, estimate $\langle\partial_\lambda U_\lambda\rangle_{\lambda_k}$, then integrate numerically. If $U_\lambda=(1-\lambda)U_A+\lambda U_B$, the integrand is simply $\langle U_B-U_A\rangle_\lambda$.

TI is often the most interpretable method. You can plot the integrand and see exactly where the transformation becomes hard. Its weakness is that it needs simulations at intermediate $\lambda$ values and a smooth path. Singular endpoints, disappearing atoms, and poorly chosen alchemical paths can create sharp integrands that require many windows or soft-core potentials.

## 3. Free-Energy Perturbation

FEP estimates the same difference without explicitly integrating a path. The move is to express the target partition function $Z_B$ using the reference Boltzmann weight from $A$.

**Proof note.**
Insert and remove $U_A$ inside the exponential, then divide by $Z_A$. The normalized factor $Z_A^{-1}\exp[-\beta U_A(x)]$ turns the remaining integral into an expectation over state $A$:

<div class="math-display">
$$
\begin{aligned}
\frac{Z_B}{Z_A}
&=
\frac{1}{Z_A}\int dx\,e^{-\beta U_B(x)}\\
&=
\frac{1}{Z_A}\int dx\,e^{-\beta U_A(x)}
e^{-\beta[U_B(x)-U_A(x)]}\\
&=
\left\langle e^{-\beta(U_B-U_A)}\right\rangle_A .
\end{aligned}
\notag
$$
</div>

Taking $-\beta^{-1}\log$ gives the Zwanzig formula:

<div class="math-display">
$$
\Delta F_{A\to B}
=
-\beta^{-1}\log
\left\langle
\exp[-\beta(U_B-U_A)]
\right\rangle_A .
$$
</div>

This is the Zwanzig formula. It is exact when the average is evaluated with infinite sampling from state $A$.

Regular FEP means both states live in the same coordinate space and one potential is treated as a perturbation of the other. Examples include correcting a force field with a higher-level electronic-structure energy, comparing two exchange-correlation functionals on the same set of configurations, or evaluating a small field-induced change.

Alchemical FEP uses the same identity but lets $U_\lambda$ change the chemical identity or interaction parameters. A solute can be decoupled from solvent, an atom type can be morphed into another, or a charge distribution can be scaled. The word "alchemical" does not change the mathematics. It changes the practical danger: endpoint ensembles may have little phase-space overlap, especially when atoms appear, disappear, or change charge.

FEP is very efficient when $A$ already samples the important configurations of $B$. It fails brutally when it does not. The exponential average is dominated by rare low-work configurations, so the estimator can look precise while being biased. Reverse FEP, from $B$ to $A$, is a useful diagnostic: if forward and reverse estimates disagree beyond uncertainty, overlap is poor.

## 4. BAR and MBAR

Bennett's acceptance ratio (BAR) improves on one-direction FEP by using samples from both states. MBAR generalizes this idea to many states and is now the standard estimator when simulations are available at several $\lambda$ windows.

Suppose we sample $K$ reduced potentials $u_k(x)=\beta U_k(x)$ and collect $N_k$ samples from each state. MBAR treats all saved configurations as pooled samples from a mixture distribution, then asks how much each configuration should count for each target state.

**Proof note.**
Define the unnormalized density $q_k(x)=\exp[-u_k(x)]$ and the normalized density $p_k(x)=q_k(x)/Z_k=\exp[f_k-u_k(x)]$, where $f_k=-\log Z_k$. If we pool all samples, the effective sampling distribution is the mixture of the sampled state densities:

<div class="math-display">
$$
p_{\mathrm{mix}}(x)
=
\frac{1}{N}\sum_{k=1}^{K}N_k p_k(x)
=
\frac{1}{N}\sum_{k=1}^{K}N_k\exp[f_k-u_k(x)] .
\notag
$$
</div>

Now estimate the target normalization constant $Z_i=\int dx\,q_i(x)$ by importance sampling from this pooled distribution. Insert $p_{\mathrm{mix}}(x)/p_{\mathrm{mix}}(x)$ into the integral:

<div class="math-display">
$$
\begin{aligned}
Z_i
&=
\int dx\,p_{\mathrm{mix}}(x)
\frac{q_i(x)}{p_{\mathrm{mix}}(x)}\\
&\approx
\frac{1}{N}\sum_{n=1}^{N}
\frac{\exp[-u_i(x_n)]}{p_{\mathrm{mix}}(x_n)}\\
&=
\sum_{n=1}^{N}
\frac{\exp[-u_i(x_n)]}
{\sum_{k=1}^{K}N_k\exp[f_k-u_k(x_n)]}.
\end{aligned}
\notag
$$
</div>

This is why each configuration contributes its target Boltzmann weight divided by the pooled probability of having sampled that configuration. Since $Z_i=\exp[-f_i]$, enforcing this relation for every state gives the MBAR self-consistency equations:

<div class="math-display">
$$
\hat f_i
=
-\log
\sum_{n=1}^{N}
\frac{\exp[-u_i(x_n)]}
{\sum_{k=1}^{K} N_k \exp[\hat f_k-u_k(x_n)]}.
$$
</div>

The free-energy difference is $\Delta F_{ij}=\beta^{-1}(\hat f_j-\hat f_i)$. The formula looks heavier than FEP, but the idea is simple: every sampled configuration contributes to every state according to its statistical weight. If a configuration is plausible under state $i$, it helps estimate $f_i$; if it is implausible, MBAR automatically downweights it.

This weighting is not just a convenient heuristic. MBAR can be derived from extended bridge sampling or maximum likelihood, and in the large-sample limit it is asymptotically unbiased and has the lowest variance among the standard estimators that combine equilibrium samples from multiple states. The practical translation is: once the simulations have already been run, MBAR is close to the statistically best way to reuse all of those samples, provided the states have adequate overlap.

For alchemical calculations, MBAR is especially natural. One simulates a ladder of intermediate Hamiltonians $U_{\lambda_k}$, evaluates all reduced potentials $u_i(x_n)$ for all saved samples, and lets MBAR combine information across the ladder. This is usually more stable than doing independent FEP estimates window by window, because MBAR uses the entire overlap graph rather than only neighboring pairs.

**Flexible generalization.**
The index $k$ in MBAR does not have to mean an alchemical coupling value. It labels any thermodynamic state whose reduced potential $u_k(x)$ can be evaluated on the pooled samples. For replica-exchange or parallel-tempering MCMC, the physical potential may be fixed while the temperature changes:

<div class="math-display">
$$
u_k(x)=\beta_k U(x).
\notag
$$
</div>

Then MBAR combines samples from many temperatures to estimate free-energy differences, expectations, heat capacities, or results at intermediate temperatures. Alchemical MBAR is the corresponding case $u_k(x)=\beta U_{\lambda_k}(x)$, and mixed grids are possible too:

<div class="math-display">
$$
u_k(x)=\beta_k U_{\lambda_k}(x)+b_k(x),
\notag
$$
</div>

where $b_k(x)$ could be an umbrella bias or another known biasing term. The requirement is not that the states differ by a potential parameter only; the requirement is overlap plus the ability to evaluate the reduced potential of each saved configuration in each target state.

## 5. TI Compared with FEP and MBAR

TI estimates an integral of ensemble averages. FEP estimates an exponential reweighting identity. MBAR estimates the same partition-function ratios by optimally combining samples from multiple ensembles. They agree in the infinite-sampling limit if they use the same end states and a valid path.

The practical difference is where the pain appears. In TI, the pain appears as a difficult integrand: sharp peaks, endpoint curvature, or noisy $\partial_\lambda U_\lambda$. In FEP, the pain appears as poor overlap and exponential bias. In MBAR, the pain appears as a disconnected overlap graph: adjacent windows do not share enough statistically important configurations.

TI is excellent when $\partial_\lambda U_\lambda$ is cheap and smooth. FEP is excellent for small perturbations or high-quality reference sampling. MBAR is usually the best default for serious alchemical work because it uses all windows and gives overlap diagnostics, but it still cannot invent missing phase-space overlap. More windows, better path design, replica exchange, soft-core interactions, and expanded ensembles are often more important than the estimator itself.

Another useful distinction is data reuse. TI mainly needs the derivative $\partial_\lambda U_\lambda$ sampled at each window. FEP and MBAR need energies of sampled configurations under one or more other states. If rerunning expensive electronic-structure calculations is hard, TI may be cheaper. If evaluating multiple Hamiltonians on saved configurations is cheap, MBAR can be extremely powerful.

## 6. Variational Free Energy

The SSCHA route starts from a different question. Instead of asking for $\Delta F$ between two sampled systems, ask: can we approximate the anharmonic free energy by the best harmonic density matrix?

For a quantum or classical system with Hamiltonian $H$, the exact equilibrium free energy is obtained by minimizing the Gibbs functional over density matrices $\rho$. At the minimum, the density is the Boltzmann density $\rho_\ast=e^{-\beta H}/Z$, but the variational form is more useful because it lets us restrict the search space:

<div class="math-display">
$$
F
=
\min_\rho
\left[
\operatorname{Tr}(\rho H)
+\beta^{-1}\operatorname{Tr}(\rho\log\rho)
\right].
$$
</div>

The upper-bound statement comes from restricting this exact minimization. Let $\rho_{\mathcal R,\Phi}$ be the exact thermal density matrix of a trial harmonic Hamiltonian with potential $V_{\mathcal R,\Phi}$.

**Proof note.**
Evaluate the Gibbs functional for the true potential $V$ at the trial harmonic density. Split $V$ into the trial potential plus a correction. The part containing the trial Hamiltonian gives the harmonic free energy $F_{\mathcal R,\Phi}$, and the leftover part is the trial average of the anharmonic correction:

<div class="math-display">
$$
\begin{aligned}
\operatorname{Tr}(\rho_{\mathcal R,\Phi}H)
+\beta^{-1}\operatorname{Tr}(\rho_{\mathcal R,\Phi}\log\rho_{\mathcal R,\Phi})
&=
F_{\mathcal R,\Phi}
+\left\langle V-V_{\mathcal R,\Phi}\right\rangle_{\mathcal R,\Phi}.
\end{aligned}
\notag
$$
</div>

Since the exact $F$ is the unrestricted minimum over all $\rho$, this trial value is an upper bound:

<div class="math-display">
$$
\mathcal F(\mathcal R,\Phi)
=
F_{\mathcal R,\Phi}
+
\left\langle
V(\mathbf R)-V_{\mathcal R,\Phi}(\mathbf R)
\right\rangle_{\mathcal R,\Phi}.
$$
</div>

This is the Gibbs-Bogoliubov variational principle. $F_{\mathcal R,\Phi}$ is the free energy of the trial harmonic system, $V$ is the true Born-Oppenheimer potential, and $V_{\mathcal R,\Phi}$ is the harmonic trial potential.

It is also useful to compare the bound with FEP. If the harmonic system is the reference, the exact reweighting correction would be

<div class="math-display">
$$
\Delta F
=
-\beta^{-1}\log
\left\langle
e^{-\beta(V-V_{\mathcal R,\Phi})}
\right\rangle_{\mathcal R,\Phi}.
\notag
$$
</div>

The variational expression replaces this logarithmic exponential average by the first cumulant $\langle V-V_{\mathcal R,\Phi}\rangle_{\mathcal R,\Phi}$. Jensen's inequality, $-\log\langle e^{-X}\rangle\le\langle X\rangle$, is what makes this first-order-looking expression an upper bound rather than merely a Taylor approximation. The best approximation is found by minimizing $\mathcal F(\mathcal R,\Phi)$ over both centroids and force constants.

SSCHA is the stochastic implementation of this minimization. It samples ionic configurations from the trial harmonic density, evaluates true energies and forces, and updates $\mathcal R$ and $\Phi$ so that the trial harmonic system becomes the best Gaussian representation of the anharmonic solid at temperature $T$.

## 7. SSCHA in the Same Language

SSCHA is not usually described as TI, FEP, or MBAR, but it belongs to the same free-energy family. The variational functional contains an exact free energy of a reference system plus a correction averaged over that reference.

**Proof note.**
Take any trial Hamiltonian $H_0$ with exact thermal density $\rho_0$ and free energy $F_0$. Plugging $\rho_0$ into the Gibbs functional for the true Hamiltonian $H$ gives $F_0+\langle H-H_0\rangle_0$. Because the exact $F$ minimizes the same functional over all densities, it must be no larger than that trial value. Equivalently, the FEP identity plus Jensen gives the same inequality:

<div class="math-display">
$$
\begin{aligned}
F
&=
F_0-\beta^{-1}\log
\left\langle e^{-\beta(H-H_0)}\right\rangle_0\\
&\le
F_0+\left\langle H-H_0\right\rangle_0 .
\end{aligned}
\notag
$$
</div>

This gives the compact variational bound:

<div class="math-display">
$$
F
\le
F_0+\langle H-H_0\rangle_0 .
$$
</div>

The inequality is both the power and the limitation. It avoids the exponential-overlap disaster of FEP and gives a stable objective for anharmonic phonons. But it only searches within the chosen trial family. A harmonic Gaussian density can capture frequency renormalization, thermal expansion, and many strong anharmonic effects, but it cannot represent every possible multimodal or strongly diffusive ionic distribution without extensions.

One can think of SSCHA as free-energy calculation by optimized reference design. TI and MBAR ask how to connect known endpoints with good sampling overlap. SSCHA asks which harmonic reference makes the anharmonic target as variationally close as possible.

## 8. A Practical Decision Map

Use TI when the derivative with respect to the coupling parameter is available, the path is smooth, and you want direct interpretability. Use FEP when the perturbation is small or when configurations from one state already represent the other well. Use BAR or MBAR when you have samples from several states and can evaluate cross reduced potentials. Use MBAR flexibly across alchemical states, temperature ladders from parallel tempering, umbrella windows, or mixed thermodynamic grids, but watch overlap diagnostics rather than trusting dense formulas.

Use SSCHA when the problem is an anharmonic vibrational free energy, especially in crystals where phonons are strongly renormalized by temperature. The central object is not a path integral over $\lambda$ but a variationally optimized harmonic ensemble.

In short: TI integrates mean generalized forces, FEP exponentiates energy differences, MBAR combines all reweighting information across many ensembles, and SSCHA minimizes a variational free-energy upper bound. They are different answers to the same question: how do we estimate a partition-function ratio without ever computing the full partition function directly?

## References

Michael R. Shirts and John D. Chodera, [Statistically optimal analysis of samples from multiple equilibrium states](https://www.choderalab.org/s/MBAR-paper.pdf), *Journal of Chemical Physics* **129**, 124105 (2008), DOI: [10.1063/1.2978177](https://doi.org/10.1063/1.2978177).
