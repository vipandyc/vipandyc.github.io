---
permalink: /blog/kubo-fdt-materials-correlations/
title: "From Fluctuations to Materials Properties: Kubo, FDT, and Correlations"
excerpt: "A practical map from equilibrium correlations to diffusion, viscosity, charge and heat conductivity, susceptibility, vibrational spectra, and electronic or phonon density of states."
author_profile: true
---

# From Fluctuations to Materials Properties: Kubo, FDT, and Correlations

An equilibrium material is never microscopically still. Atoms move, currents fluctuate, and dipoles and spins change in time. A correlation function asks how much a fluctuation now resembles one a time $t$ later. **Kubo linear response** connects those equilibrium fluctuations to what happens when we apply a weak field; the **fluctuation–dissipation theorem** (FDT) connects their spectrum to energy absorption.

The practical question is: *which quantity should I record?* Velocity gives diffusion, stress gives viscosity, charge and heat currents give conductivities, and dipole or polarizability fluctuations give vibrational spectra. Electronic and phonon density of states (DOS) fit the broader spectral picture, although they are not themselves Green–Kubo transport coefficients.

## 1. One Response Formula, Many Observables

Apply a weak perturbation $H'(t)=-f(t)B$ to a system at equilibrium. The field $f$ couples to $B$; we observe the change in $A$. To first order,

<div class="math-display">
$$
\delta\langle A(t)\rangle
=\int_{-\infty}^{t}\chi_{AB}(t-s)f(s)\,\mathrm ds,
\qquad
\chi_{AB}(t)
=\frac{i}{\hbar}\Theta(t)
\left\langle[A(t),B(0)]\right\rangle_{\rm eq}.
$$
</div>

This is the **quantum Kubo formula**. The step function $\Theta(t)$ enforces causality: a probe cannot affect the past. If $A=B$, the response is tied to an autocorrelation; if they differ, a cross-correlation describes a coupled response. We use the Fourier convention $F(\omega)=\int_{-\infty}^{\infty}e^{i\omega t}F(t)\,\mathrm dt$ and write $\chi''=\operatorname{Im}\chi$.

For classical equilibrium dynamics, ordinary products replace operator commutators. Write $\beta=1/(k_{\rm B}T)$ and $\delta A=A-\langle A\rangle$. The connected **cross-correlation** is $C_{AB}(t)=\langle\delta A(t)\delta B(0)\rangle$; setting $B=A$ gives the **autocorrelation** $C_A(t)=C_{AA}(t)$. For the field conjugate to $A$, it obeys

<div class="math-display">
$$
\chi_{AA}^{\rm cl}(t)
=-\beta\Theta(t)\frac{\mathrm d C_A(t)}{\mathrm dt}.
$$
</div>

For $H'=-fA$ and $\rho_{\rm eq}=Z^{-1}e^{-\beta H}$, phase-space integration by parts gives

<div class="math-display">
$$
\begin{aligned}
\chi_{AA}^{\rm cl}(t)
&=-\Theta(t)\langle\{A(t),A(0)\}_{\rm PB}\rangle\\
&=\Theta(t)\int\mathrm d\Gamma\,A(t)
  \{\rho_{\rm eq},A(0)\}_{\rm PB}\\
&=\beta\Theta(t)\langle\delta A(t)\dot A(0)\rangle\\
&=-\beta\Theta(t)\frac{\mathrm d}{\mathrm dt}
  \langle\delta A(t)\delta A(0)\rangle.
\end{aligned}
$$
</div>

The last equality uses equilibrium time-translation symmetry:
$\langle\delta A(t)\dot A(0)\rangle=-\mathrm dC_A(t)/\mathrm dt$.

Thus the relaxation of an equilibrium fluctuation fixes the response to a weak field.

## 2. Quantum and Classical FDT

### 2.1 Quantum Green functions and correlation spectra

Let $A$ be Hermitian, with $\delta A=A-\langle A\rangle$. Three two-time Green functions separate causal response from the two operator orderings:

<div class="math-display">
$$
\begin{aligned}
G^R_{AA}(t)&=-i\Theta(t)\langle[\delta A(t),\delta A(0)]\rangle,\\
\chi_{AA}(t)&=-G^R_{AA}(t)/\hbar,\\
G^>_{AA}(t)&=-i\langle\delta A(t)\delta A(0)\rangle,\\
G^<_{AA}(t)&=-i\langle\delta A(0)\delta A(t)\rangle.
\end{aligned}
$$
</div>

The retarded function is the causal difference of the two orderings. Their Fourier spectra are

<div class="math-display">
$$
\begin{aligned}
G^R_{AA}(t)&=\Theta(t)\bigl[G^>_{AA}(t)-G^<_{AA}(t)\bigr],\\
S_A^>(\omega)&=iG^>_{AA}(\omega)
=\int_{-\infty}^{\infty}e^{i\omega t}
\langle\delta A(t)\delta A(0)\rangle\,\mathrm dt,\\
S_A^<(\omega)&=iG^<_{AA}(\omega)=S_A^>(-\omega).
\end{aligned}
$$
</div>

### 2.2 Deriving quantum FDT

Insert energy eigenstates $H|n\rangle=E_n|n\rangle$ into the greater correlation:

<div class="math-display">
$$
S_A^>(\omega)=\frac{2\pi}{Z}\sum_{n,m}e^{-\beta E_n}
|\langle m|\delta A|n\rangle|^2
\delta\!\left(\omega-\frac{E_m-E_n}{\hbar}\right).
$$
</div>

For negative frequency, swap $n$ and $m$. The delta function then sets $E_m-E_n=\hbar\omega$:

<div class="math-display">
$$
\begin{aligned}
S_A^>(-\omega)
&=\frac{2\pi}{Z}\sum_{n,m}e^{-\beta E_m}
|\langle m|\delta A|n\rangle|^2
\delta\!\left(\omega-\frac{E_m-E_n}{\hbar}\right)\\
&=e^{-\beta\hbar\omega}S_A^>(\omega),\\
S_A^<(\omega)&=S_A^>(-\omega)
=e^{-\beta\hbar\omega}S_A^>(\omega),\\
2\hbar\chi''_{AA}(\omega)
&=S_A^>(\omega)-S_A^<(\omega)\\
&=(1-e^{-\beta\hbar\omega})S_A^>(\omega).
\end{aligned}
$$
</div>

This is the **quantum FDT**: the equilibrium spectrum determines the dissipative response $\chi''$. The factor $1-e^{-\beta\hbar\omega}$ records the unequal probabilities of absorbing and emitting energy $\hbar\omega$.

### 2.3 Classical FDT and the useful connection

For the symmetrized spectrum $S_A^{\rm sym}=(S_A^{>}+S_A^{<})/2$, the same result becomes

<div class="math-display">
$$
S_A^{\rm sym}(\omega)
=\hbar\coth\!\left(\frac{\beta\hbar\omega}{2}\right)\chi''_{AA}(\omega)
\;\xrightarrow{\;\beta\hbar\omega\ll1\;}\;
S_A^{\rm cl}(\omega)=\frac{2k_{\rm B}T}{\omega}\chi''_{AA}(\omega).
$$
</div>

Here $S_A^{\rm cl}(\omega)=\int e^{i\omega t}C_A(t)\,\mathrm dt$ is the classical equilibrium autocorrelation spectrum. It also follows directly from Eq. (2). Since $C_A(t)=C_A(-t)$,

<div class="math-display">
$$
\begin{aligned}
\chi''_{AA}(\omega)
&=-\beta\int_0^\infty\sin(\omega t)\dot C_A(t)\,\mathrm dt\\
&=\beta\omega\int_0^\infty\cos(\omega t)C_A(t)\,\mathrm dt
=\frac{\beta\omega}{2}S_A^{\rm cl}(\omega).
\end{aligned}
$$
</div>

This is the frequency-domain **classical FDT**.

The link is useful in both directions: simulate or measure equilibrium fluctuations to predict absorption, or infer fluctuations from a response experiment. Causality also fixes the dispersive part:

<div class="math-display">
$$
\chi'_{AA}(\omega)=\frac1\pi\mathcal P\int_{-\infty}^{\infty}
\frac{\chi''_{AA}(\omega')}{\omega'-\omega}\,\mathrm d\omega'.
$$
</div>

Classical MD applies when nuclear motion is adequately classical; it misses quantum detailed balance and zero-point motion. [Kubo's original paper](https://doi.org/10.1143/JPSJ.12.570) and [FDT review](https://www.zib.de/userpage/donati/stochastics2023/06/references/Kubo1966.pdf) give the broader theory.

## 3. Property Map: What Do We Correlate?

For 1–4 and 6, use equilibrium, isotropic three-dimensional classical systems. Here $V$ is volume, $T$ temperature, $\delta P_{xy}=P_{xy}-\langle P_{xy}\rangle$, and all currents are **total** currents, not current densities. For anisotropic materials, use tensor components. Proportionality signs below omit probe geometry and unit factors.

### 3.1 Self-diffusivity

<div class="math-display">
$$
D=\frac13\int_0^\infty\langle\mathbf v(t)\cdot\mathbf v(0)\rangle\,\mathrm dt
=\lim_{t\to\infty}\frac{\langle\lVert\mathbf r(t)-\mathbf r(0)\rVert^2\rangle}{6t}.
$$
</div>

**Why:** Persistent velocity produces long-time displacement.

### 3.2 Shear viscosity

<div class="math-display">
$$
\eta=\frac{V}{k_{\rm B}T}\int_0^\infty
\langle\delta P_{xy}(t)\delta P_{xy}(0)\rangle\,\mathrm dt.
$$
</div>

**Why:** $P_{xy}$ carries $x$-momentum across a $y$-plane; its relaxation sets shear resistance.

### 3.3 Ionic conductivity

<div class="math-display">
$$
\mathbf I=\sum_a q_a\mathbf v_a,\qquad
\sigma_{\rm ion}=\frac{1}{3Vk_{\rm B}T}\int_0^\infty
\langle\mathbf I(t)\cdot\mathbf I(0)\rangle\,\mathrm dt.
$$
</div>

**Why:** An electric field drives charge current. The total-current correlation includes motion of *different* ions together.

### 3.4 Thermal conductivity

<div class="math-display">
$$
\kappa=\frac{1}{3Vk_{\rm B}T^2}\int_0^\infty
\langle\mathbf J_Q(t)\cdot\mathbf J_Q(0)\rangle\,\mathrm dt,
\qquad \mathbf J_Q=V\mathbf j_Q.
$$
</div>

**Why:** $\mathbf J_Q$ carries heat; in an insulator this is often mainly phonon transport. The formula assumes no coupled charge or particle flow. [MD example](https://docs.lammps.org/compute_heat_flux.html).

### 3.5 Electronic conductivity

<div class="math-display">
$$
\operatorname{Re}\sigma_{\alpha\alpha}(\omega)
=\frac{\chi''_{J_\alpha J_\alpha}(\omega)}{V\omega}\quad(\omega>0),
\qquad
\sigma_{\rm dc}=\lim_{\omega\to0^+}\operatorname{Re}\sigma(\omega).
$$
</div>

**Why:** The electric field drives electronic current $\mathbf J$. DFT Kubo–Greenwood uses band energies and current matrix elements; the full complex response includes a contact term. [Optical-properties note](/blog/optical-properties-kubo-dft-gw/).

### 3.6 Thermoelectric response

<div class="math-display">
$$
L_{eQ}=\frac{1}{3Vk_{\rm B}T}\int_0^\infty
\langle\mathbf J_e(t)\cdot\mathbf J_Q(0)\rangle\,\mathrm dt,
\qquad
\mathbf j_e=\sigma\mathbf E-\frac{L_{eQ}}{T}\nabla T.
$$
</div>

**Why:** Charge–heat *cross*-correlation gives the charge current driven by a temperature gradient; at zero current, $S=L_{eQ}/(T\sigma)$ with $S=E/\nabla T$.

### 3.7 Electric or magnetic susceptibility

<div class="math-display">
$$
\chi_{AA}(\omega)=\frac{i}{\hbar}\int_0^\infty e^{i\omega t}
\langle[A(t),A(0)]\rangle\,\mathrm dt,
\qquad A=M_{\rm electric}\ \text{or}\ M_{\rm magnetic}.
$$
</div>

**Why:** The applied field couples to the electric or magnetic moment $A$; $\operatorname{Im}\chi$ measures absorption.

### 3.8 Infrared absorption

<div class="math-display">
$$
I_{\rm IR}(\omega)\propto
\omega\chi''_{MM}(\omega)
\propto\omega(1-e^{-\beta\hbar\omega})S_M^>(\omega),
\qquad M=\text{electric dipole}.
$$
</div>

**Why:** IR-active vibrations change $M$. In MD, use its dipole autocorrelation; in DFPT, use mode effective charges.

### 3.9 Nonresonant Raman scattering

<div class="math-display">
$$
I_{\rm Raman}(\omega)\propto(\omega_L-\omega)^4 S_a^>(\omega),
\qquad
a(t)=\mathbf e_{\rm out}\cdot\boldsymbol\alpha(t)\cdot\mathbf e_{\rm in}.
$$
</div>

**Why:** Raman-active vibrations change polarizability $\boldsymbol\alpha$, selected by light polarizations. [DFPT example](https://quantum-espresso.org/Doc/INPUT_DYNMAT.html).

### 3.10 Neutron or X-ray scattering

<div class="math-display">
$$
S(\mathbf q,\omega)=\frac{1}{2\pi N}\int_{-\infty}^{\infty}
e^{i\omega t}\langle\rho_{\mathbf q}(t)\rho_{-\mathbf q}(0)\rangle\,\mathrm dt,
\qquad \rho_{\mathbf q}=\sum_j e^{-i\mathbf q\cdot\mathbf r_j}.
$$
</div>

**Why:** A probe transfers momentum $\mathbf q$ and energy $\hbar\omega$ to density fluctuations; peaks trace collective modes and phonon dispersion. For different atoms, weight $\rho_{\mathbf q}$ by scattering lengths or form factors.

### 3.11 Phonon or vibrational DOS

<div class="math-display">
$$
g_{\rm ph}(\omega)=\frac{1}{N_q}\sum_{\mathbf q\nu}
\delta(\omega-\omega_{\mathbf q\nu}),\qquad
g_{\rm MD}(\omega)\propto\int_{-\infty}^{\infty}e^{i\omega t}
\sum_a m_a\langle\mathbf v_a(t)\cdot\mathbf v_a(0)\rangle\,\mathrm dt.
$$
</div>

**Why:** Each normal mode adds a peak at its frequency; the mass-weighted MD spectrum recovers this in the classical harmonic limit. [Phonon calculation](https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html); [MD spectrum](https://manual.gromacs.org/current/onlinehelp/gmx-dos.html).

### 3.12 Electronic DOS

<div class="math-display">
$$
g_{\rm KS}(E)=\frac{1}{N_k}\sum_{n\mathbf k}\delta(E-\epsilon_{n\mathbf k}),
\qquad
g(E)=-\frac1\pi\operatorname{Im}\operatorname{Tr}G^R(E).
$$
</div>

**Why:** The first counts DFT band states; the second counts interacting one-particle spectral weight. Here $G^R(E)$ is the fermionic one-particle Green function, distinct from the response Green function in Section 2. [DFT DOS example](https://quantum-espresso.org/Doc/pp_user_guide/node9.html).

## 4. MD, DFT, and the Practical Workflow

**MD:** record the relevant time series, subtract equilibrium means when needed, average correlations over many time origins, and integrate or Fourier-transform. Check the time window, sample spacing, finite-size effects, and uncertainty. Transport estimates need a converged running integral; spectra need enough duration for frequency resolution. [GROMACS' correlation-function guide](https://manual.gromacs.org/current/reference-manual/analysis/correlation-function.html) discusses diffusion and viscosity examples.

**DFT and DFPT:** ground-state DFT provides electronic states and forces; DFPT supplies harmonic phonons, effective charges, and response tensors. A static DFT calculation does not itself produce a long real-time trajectory. Ab initio MD supplies nuclear trajectories for classical correlation estimates; electronic Kubo/optical calculations and harmonic phonon calculations instead assemble response or spectra from states, modes, and matrix elements.

**Before using a formula:** identify the applied field and its conjugate observable, decide whether you need a time integral or a finite-frequency spectrum, and check whether classical or quantum statistics govern the experiment. That choice determines the correlation to compute—and whether a Kubo coefficient, a scattering spectrum, or a DOS is the right object at all.
