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

TI differentiates the free energy along the path. Since $F_\lambda=-\beta^{-1}\log Z_\lambda$,

<div class="math-display">
$$
\frac{dF_\lambda}{d\lambda}
=
\left\langle \frac{\partial U_\lambda}{\partial\lambda}\right\rangle_\lambda .
$$
</div>

The derivation is one line: differentiate $Z_\lambda$, divide by $Z_\lambda$, and recognize the Boltzmann average. Therefore

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

FEP estimates the same difference without explicitly integrating a path. Starting from the two partition functions,

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

This is the Zwanzig formula. It is exact when the average is evaluated with infinite sampling from state $A$. The derivation is just reweighting: write $Z_B=\int dx\,\exp[-\beta U_A(x)]\exp[-\beta(U_B(x)-U_A(x))]$, then divide by $Z_A$.

Regular FEP means both states live in the same coordinate space and one potential is treated as a perturbation of the other. Examples include correcting a force field with a higher-level electronic-structure energy, comparing two exchange-correlation functionals on the same set of configurations, or evaluating a small field-induced change.

Alchemical FEP uses the same identity but lets $U_\lambda$ change the chemical identity or interaction parameters. A solute can be decoupled from solvent, an atom type can be morphed into another, or a charge distribution can be scaled. The word "alchemical" does not change the mathematics. It changes the practical danger: endpoint ensembles may have little phase-space overlap, especially when atoms appear, disappear, or change charge.

FEP is very efficient when $A$ already samples the important configurations of $B$. It fails brutally when it does not. The exponential average is dominated by rare low-work configurations, so the estimator can look precise while being biased. Reverse FEP, from $B$ to $A$, is a useful diagnostic: if forward and reverse estimates disagree beyond uncertainty, overlap is poor.

## 4. BAR and MBAR

Bennett's acceptance ratio (BAR) improves on one-direction FEP by using samples from both states. MBAR generalizes this idea to many states and is now the standard estimator when simulations are available at several $\lambda$ windows.

Suppose we sample $K$ reduced potentials $u_k(x)=\beta U_k(x)$ and collect $N_k$ samples from each state. MBAR estimates the dimensionless free energies $f_k=-\log Z_k$ through the self-consistent equations

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

For alchemical calculations, MBAR is especially natural. One simulates a ladder of intermediate Hamiltonians $U_{\lambda_k}$, evaluates all reduced potentials $u_i(x_n)$ for all saved samples, and lets MBAR combine information across the ladder. This is usually more stable than doing independent FEP estimates window by window, because MBAR uses the entire overlap graph rather than only neighboring pairs.

## 5. TI Compared with FEP and MBAR

TI estimates an integral of ensemble averages. FEP estimates an exponential reweighting identity. MBAR estimates the same partition-function ratios by optimally combining samples from multiple ensembles. They agree in the infinite-sampling limit if they use the same end states and a valid path.

The practical difference is where the pain appears. In TI, the pain appears as a difficult integrand: sharp peaks, endpoint curvature, or noisy $\partial_\lambda U_\lambda$. In FEP, the pain appears as poor overlap and exponential bias. In MBAR, the pain appears as a disconnected overlap graph: adjacent windows do not share enough statistically important configurations.

TI is excellent when $\partial_\lambda U_\lambda$ is cheap and smooth. FEP is excellent for small perturbations or high-quality reference sampling. MBAR is usually the best default for serious alchemical work because it uses all windows and gives overlap diagnostics, but it still cannot invent missing phase-space overlap. More windows, better path design, replica exchange, soft-core interactions, and expanded ensembles are often more important than the estimator itself.

Another useful distinction is data reuse. TI mainly needs the derivative $\partial_\lambda U_\lambda$ sampled at each window. FEP and MBAR need energies of sampled configurations under one or more other states. If rerunning expensive electronic-structure calculations is hard, TI may be cheaper. If evaluating multiple Hamiltonians on saved configurations is cheap, MBAR can be extremely powerful.

## 6. Variational Free Energy

The SSCHA route starts from a different question. Instead of asking for $\Delta F$ between two sampled systems, ask: can we approximate the anharmonic free energy by the best harmonic density matrix?

For a quantum or classical system with Hamiltonian $H$, the exact equilibrium free energy is obtained by minimizing the Gibbs functional over density matrices $\rho$:

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

If the minimization is restricted to a trial family $\rho_{\mathcal R,\Phi}$ generated by a harmonic Hamiltonian centered at ionic positions $\mathcal R$ with force-constant matrix $\Phi$, the result is an upper bound to the exact free energy:

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

This is the Gibbs-Bogoliubov variational principle. $F_{\mathcal R,\Phi}$ is the free energy of the trial harmonic system, $V$ is the true Born-Oppenheimer potential, and $V_{\mathcal R,\Phi}$ is the harmonic trial potential. The best approximation is found by minimizing $\mathcal F(\mathcal R,\Phi)$ over both centroids and force constants.

SSCHA is the stochastic implementation of this minimization. It samples ionic configurations from the trial harmonic density, evaluates true energies and forces, and updates $\mathcal R$ and $\Phi$ so that the trial harmonic system becomes the best Gaussian representation of the anharmonic solid at temperature $T$.

## 7. SSCHA in the Same Language

SSCHA is not usually described as TI, FEP, or MBAR, but it belongs to the same free-energy family. The variational functional contains an exact free energy of a reference system plus a correction averaged over that reference. That correction resembles FEP in spirit, but it is linear rather than exponential because it comes from a variational upper bound:

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

Use TI when the derivative with respect to the coupling parameter is available, the path is smooth, and you want direct interpretability. Use FEP when the perturbation is small or when configurations from one state already represent the other well. Use BAR or MBAR when you have samples from several states and can evaluate cross energies. Use alchemical MBAR for binding, solvation, mutation, and chemical-transformation problems, but watch overlap diagnostics rather than trusting dense formulas.

Use SSCHA when the problem is an anharmonic vibrational free energy, especially in crystals where phonons are strongly renormalized by temperature. The central object is not a path integral over $\lambda$ but a variationally optimized harmonic ensemble.

In short: TI integrates mean generalized forces, FEP exponentiates energy differences, MBAR combines all reweighting information across many ensembles, and SSCHA minimizes a variational free-energy upper bound. They are different answers to the same question: how do we estimate a partition-function ratio without ever computing the full partition function directly?
