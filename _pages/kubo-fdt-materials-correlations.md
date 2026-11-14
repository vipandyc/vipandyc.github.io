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

Here $\Gamma=(\mathbf r_1,\ldots,\mathbf r_N;\mathbf p_1,\ldots,\mathbf p_N)$ contains every particle's position and momentum. Thus $\mathrm d\Gamma=\prod_{i=1}^N\mathrm d^3r_i\,\mathrm d^3p_i$ integrates over the full phase space.

For $H'=-fA$ and $\rho_{\rm eq}(\Gamma)=Z^{-1}e^{-\beta H(\Gamma)}$, phase-space integration by parts gives

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

Insert energy eigenstates $H\lvert n\rangle=E_n\lvert n\rangle$ into the greater correlation:

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

For 1–4 and 6, assume classical equilibrium and isotropic three-dimensional transport. The cross-observable version of Eq. (2) is

<div class="math-display">
$$
\chi_{AB}^{\rm cl}(t)
=\beta\Theta(t)\langle\delta A(t)\,\delta\dot B(0)\rangle.
$$
</div>

Choose transport polarizations whose time derivatives are the currents. Equation (2) gives

<div class="math-display">
$$
\dot{\mathbf B}_b=\mathbf J_b,\qquad
\chi_{\mathbf J_a\mathbf B_b}^{\rm cl}(t)
=\beta\Theta(t)\langle\mathbf J_a(t)\cdot\mathbf J_b(0)\rangle.
$$
</div>

The entropy-production rate identifies the two driving forces:

<div class="math-display">
$$
\dot s=\mathbf j_e\cdot\frac{\mathbf E}{T}
-\mathbf j_Q\cdot\frac{\nabla T}{T^2}.
$$
</div>

For total currents $\mathbf J_a=V\mathbf j_a$, the zero-frequency Kubo coefficients therefore give

<div class="math-display">
$$
\begin{aligned}
L_{ab}&=\frac{1}{3Vk_{\rm B}}\int_0^\infty
\langle\mathbf J_a(t)\cdot\mathbf J_b(0)\rangle\,\mathrm dt,\\
\begin{pmatrix}\mathbf j_e\\\mathbf j_Q\end{pmatrix}
&=\begin{pmatrix}L_{ee}&L_{eQ}\\L_{Qe}&L_{QQ}\end{pmatrix}
\begin{pmatrix}\mathbf E/T\\-\nabla T/T^2\end{pmatrix}.
\end{aligned}
$$
</div>

The second vector contains the electric and thermal driving forces. In 3D, $1/3$ averages the three directions; anisotropic materials use tensor components.

### 3.1 Self-diffusivity

With $H'=-\mathbf F\cdot\mathbf r$, insert $A=v_x$, $B=x$, and $\dot B=v_x$ into the classical Kubo formula:

<div class="math-display">
$$
\begin{aligned}
\mu&=\int_0^\infty\chi_{v_xx}^{\rm cl}(t)\,\mathrm dt
=\beta\int_0^\infty\langle v_x(t)v_x(0)\rangle\,\mathrm dt,\\
D&=k_{\rm B}T\mu
=\frac13\int_0^\infty\langle\mathbf v(t)\cdot\mathbf v(0)\rangle\,\mathrm dt.
\end{aligned}
$$
</div>

The equivalent displacement formula follows by integrating velocity twice:

<div class="math-display">
$$
\langle\lVert\Delta\mathbf r(t)\rVert^2\rangle
=2\int_0^t(t-s)\langle\mathbf v(s)\cdot\mathbf v(0)\rangle\,\mathrm ds
\;\Longrightarrow\;
D=\lim_{t\to\infty}\frac{\langle\lVert\Delta\mathbf r(t)\rVert^2\rangle}{6t}.
$$
</div>

### 3.2 Shear viscosity

Use the total momentum current $J_p=V\delta P_{xy}$ and its Helfand polarization $B_p$ with $\dot B_p=J_p$. A shear rate supplies $f_p=-\partial_y u_x$ in $H'=-f_pB_p$, so $\delta P_{xy}=\eta f_p$:

<div class="math-display">
$$
\begin{aligned}
\chi_{J_pB_p}^{\rm cl}(t)&=\beta\Theta(t)
\langle J_p(t)J_p(0)\rangle,\\
\eta&=\frac1V\int_0^\infty\chi_{J_pB_p}^{\rm cl}(t)\,\mathrm dt
=\frac{V}{k_{\rm B}T}\int_0^\infty
\langle\delta P_{xy}(t)\delta P_{xy}(0)\rangle\,\mathrm dt.
\end{aligned}
$$
</div>

### 3.3 Ionic conductivity

The electric field couples to the total dipole. Insert $B=\mathbf P_e$ and $A=\mathbf I$:

<div class="math-display">
$$
\begin{aligned}
H'&=-\mathbf E\cdot\mathbf P_e,\qquad
\mathbf P_e=\sum_a q_a\mathbf r_a,\qquad
\dot{\mathbf P}_e=\mathbf I=\sum_a q_a\mathbf v_a,\\
\sigma_{\rm ion}&=\frac{L_{ee}}{T}
=\frac{1}{3Vk_{\rm B}T}\int_0^\infty
\langle\mathbf I(t)\cdot\mathbf I(0)\rangle\,\mathrm dt,\\
\langle\mathbf I(t)\cdot\mathbf I(0)\rangle
&=\sum_{a,b}q_aq_b
\langle\mathbf v_a(t)\cdot\mathbf v_b(0)\rangle.
\end{aligned}
$$
</div>

### 3.4 Thermal conductivity

Insert $a=b=Q$ in the transport matrix, then compare $\mathbf j_Q=-\kappa\nabla T$:

<div class="math-display">
$$
\begin{aligned}
\mathbf j_Q&=L_{QQ}\left(-\frac{\nabla T}{T^2}\right),\qquad
\mathbf J_Q=V\mathbf j_Q,\\
\kappa&=\frac{L_{QQ}}{T^2}
=\frac{1}{3Vk_{\rm B}T^2}\int_0^\infty
\langle\mathbf J_Q(t)\cdot\mathbf J_Q(0)\rangle\,\mathrm dt.
\end{aligned}
$$
</div>

This form assumes no coupled charge or particle flow. [MD example](https://docs.lammps.org/compute_heat_flux.html).

### 3.5 Electronic conductivity

Use the electronic current and a vector potential, with $\mathbf E(\omega)=i\omega\mathbf A^{\rm ext}(\omega)$. The contact term comes from the field dependence of the current:

<div class="math-display">
$$
\begin{aligned}
H'&=-J_\alpha A_\alpha^{\rm ext},\qquad
\sigma_{\alpha\alpha}(\omega)
=\frac{\chi_{J_\alpha J_\alpha}(\omega)+K_{\rm contact}}{iV\omega},\\
\operatorname{Re}\sigma_{\alpha\alpha}(\omega)
&=\frac{\chi''_{J_\alpha J_\alpha}(\omega)}{V\omega}
=\frac{\pi}{V\hbar\omega}\sum_{n,m}(p_n-p_m)
|\langle m|J_\alpha|n\rangle|^2
\delta\!\left(\omega-\frac{E_m-E_n}{\hbar}\right)
\quad(\omega>0).
\end{aligned}
$$
</div>

The second line inserts $A=B=J_\alpha$ into the Lehmann/FDT result of Section 2; $p_n=e^{-\beta E_n}/Z$. Replacing many-body transitions by band energies and occupations gives Kubo–Greenwood. The DC value is $\lim_{\omega\to0^+}\operatorname{Re}\sigma(\omega)$ when finite. [Optical-properties note](/blog/optical-properties-kubo-dft-gw/).

### 3.6 Thermoelectric response

Use the *off-diagonal* entry $a=e$, $b=Q$ of the same transport matrix:

<div class="math-display">
$$
\begin{aligned}
L_{eQ}&=\frac{1}{3Vk_{\rm B}}\int_0^\infty
\langle\mathbf J_e(t)\cdot\mathbf J_Q(0)\rangle\,\mathrm dt,\\
\mathbf j_e&=\frac{L_{ee}}{T}\mathbf E
-\frac{L_{eQ}}{T^2}\nabla T,\\
\mathbf j_e=0&\quad\Longrightarrow\quad
S\equiv\frac{E}{\nabla T}
=\frac{L_{eQ}}{T L_{ee}}.
\end{aligned}
$$
</div>

### 3.7 Electric or magnetic susceptibility

Choose the moment $A$ conjugate to the applied field $h$; then set $A=B$ in the quantum Kubo formula:

<div class="math-display">
$$
\begin{aligned}
H'&=-hA,\qquad A=M_{\rm electric}\ \text{or}\ M_{\rm magnetic},\\
\chi_{AA}(\omega)&=\frac{i}{\hbar}\int_0^\infty e^{i\omega t}
\langle[A(t),A(0)]\rangle\,\mathrm dt,\\
\chi''_{AA}(\omega)&=
\frac{1-e^{-\beta\hbar\omega}}{2\hbar}S_A^>(\omega).
\end{aligned}
$$
</div>

For a classical static field, integrate Eq. (2), assuming $C_A(t)\to0$:

<div class="math-display">
$$
\chi_{AA}^{\rm cl}(0)
=-\beta\int_0^\infty\dot C_A(t)\,\mathrm dt
=\beta C_A(0)=\beta\operatorname{Var}(A).
$$
</div>

Divide by $V$ for a bulk susceptibility, with the appropriate electromagnetic unit factor.

### 3.8 Infrared absorption

Set $A=B=M$, the electric dipole along the light polarization. The field does work at rate $E(t)\dot M(t)$:

<div class="math-display">
$$
\begin{aligned}
H'&=-E(t)M,\qquad \delta M(\omega)=\chi_{MM}(\omega)E(\omega),\\
\overline{P}_{\rm abs}
&=\overline{E(t)\dot M(t)}
=\frac{\omega}{2}|E_0|^2\chi''_{MM}(\omega),\\
I_{\rm IR}(\omega)&\propto\omega\chi''_{MM}(\omega)
=\frac{\omega(1-e^{-\beta\hbar\omega})}{2\hbar}S_M^>(\omega).
\end{aligned}
$$
</div>

In MD, calculate the dipole correlation; in harmonic DFPT, calculate its mode matrix elements through effective charges.

### 3.9 Nonresonant Raman scattering

For nonresonant light, the quadratic light–matter coupling selects the projected polarizability $a$:

<div class="math-display">
$$
\begin{aligned}
H'_{\rm light}&=-\tfrac12\mathbf E\cdot\boldsymbol\alpha\cdot\mathbf E,
\qquad a=\mathbf e_{\rm out}\cdot\boldsymbol\alpha\cdot\mathbf e_{\rm in},\\
S_a^>(\omega)&=2\pi\sum_{n,m}p_n
|\langle m|\delta a|n\rangle|^2
\delta\!\left(\omega-\frac{E_m-E_n}{\hbar}\right),\\
I_{\rm Raman}(\omega)&\propto
(\omega_L-\omega)^4\sum_{n,m}p_n|\langle m|\delta a|n\rangle|^2
\delta\!\left(\omega-\frac{E_m-E_n}{\hbar}\right)\\
&=\frac{(\omega_L-\omega)^4}{2\pi}S_a^>(\omega),\\
\chi''_{aa}(\omega)&=\frac{1-e^{-\beta\hbar\omega}}{2\hbar}S_a^>(\omega).
\end{aligned}
$$
</div>

The middle line is the Lehmann spectrum from Section 2 with $A=a$; the last line uses the same FDT. The fourth power comes from the emitted dipole radiation. [DFPT example](https://quantum-espresso.org/Doc/INPUT_DYNMAT.html).

### 3.10 Neutron or X-ray scattering

The probe couples to density at momentum $\mathbf q$. Insert $A=\rho_{\mathbf q}$ and $B=\rho_{-\mathbf q}$ into the density response:

<div class="math-display">
$$
\begin{aligned}
H'&=-f_{\mathbf q}(t)\rho_{-\mathbf q},\qquad
\rho_{\mathbf q}=\sum_j e^{-i\mathbf q\cdot\mathbf r_j},\\
S_{\rm inel}(\mathbf q,\omega)
&=\frac{1}{2\pi N}\int_{-\infty}^{\infty}e^{i\omega t}
\langle\delta\rho_{\mathbf q}(t)\delta\rho_{-\mathbf q}(0)\rangle\,\mathrm dt,\\
\chi''_{\rho_{\mathbf q}\rho_{-\mathbf q}}(\omega)
&=\frac{\pi N}{\hbar}
(1-e^{-\beta\hbar\omega})S_{\rm inel}(\mathbf q,\omega),\\
\frac{\mathrm d^2\sigma}{\mathrm d\Omega\,\mathrm d\omega}
&\propto S_{\rm inel}(\mathbf q,\omega).
\end{aligned}
$$
</div>

The response relation follows from Section 2's FDT and the structure factor's normalization. Add the elastic Bragg peak separately if needed; for different atoms, include scattering lengths or form factors in the density operator.

### 3.11 Phonon or vibrational DOS

Let $a$ run over all atoms in an $N_q$-cell harmonic sample. Expanding mass-weighted velocities into normal modes and using equipartition gives

<div class="math-display">
$$
\begin{aligned}
C_v(t)&\equiv\sum_a m_a
\langle\mathbf v_a(t)\cdot\mathbf v_a(0)\rangle
=k_{\rm B}T\sum_{\mathbf q\nu}\cos(\omega_{\mathbf q\nu}t),\\
\int_{-\infty}^{\infty}e^{i\omega t}\cos(\omega_{\mathbf q\nu}t)\,\mathrm dt
&=\pi\bigl[\delta(\omega-\omega_{\mathbf q\nu})
+\delta(\omega+\omega_{\mathbf q\nu})\bigr],\\
g_{\rm ph}(\omega)
&=\frac1{N_q}\sum_{\mathbf q\nu}\delta(\omega-\omega_{\mathbf q\nu})
=\frac{1}{\pi N_qk_{\rm B}T}
\int_{-\infty}^{\infty}e^{i\omega t}C_v(t)\,\mathrm dt
\quad(\omega>0).
\end{aligned}
$$
</div>

This is a mode-counting spectrum, not a current-response coefficient. The MD equality assumes classical harmonic motion; anharmonic motion broadens the peaks. [Phonon calculation](https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html); [MD spectrum](https://manual.gromacs.org/current/onlinehelp/gmx-dos.html).

### 3.12 Electronic DOS

Use the **fermionic one-particle** retarded Green function. For a noninteracting band, its pole gives a delta function:

<div class="math-display">
$$
\begin{aligned}
G^R_{ij}(t)&=-\frac{i}{\hbar}\Theta(t)
\langle\{c_i(t),c_j^\dagger(0)\}\rangle,\\
G^R_{n\mathbf k}(E)&=\frac{1}{E-\epsilon_{n\mathbf k}+i0^+},
\qquad
-\frac1\pi\operatorname{Im}G^R_{n\mathbf k}(E)
=\delta(E-\epsilon_{n\mathbf k}),\\
g_{\rm KS}(E)&=\frac1{N_k}\sum_{n\mathbf k}
\delta(E-\epsilon_{n\mathbf k}),\\
g(E)&=-\frac{1}{\pi N_k}\sum_{\mathbf k}
\operatorname{Im}\operatorname{Tr}_nG^R(\mathbf k,E).
\end{aligned}
$$
</div>

The last line remains valid with an interacting self-energy. Its anticommutator Green function differs from the commutator response Green function in Section 2. [DFT DOS example](https://quantum-espresso.org/Doc/pp_user_guide/node9.html).

## 4. MD, DFT, and the Practical Workflow

**MD:** record the relevant time series, subtract equilibrium means when needed, average correlations over many time origins, and integrate or Fourier-transform. Check the time window, sample spacing, finite-size effects, and uncertainty. Transport estimates need a converged running integral; spectra need enough duration for frequency resolution. [GROMACS' correlation-function guide](https://manual.gromacs.org/current/reference-manual/analysis/correlation-function.html) discusses diffusion and viscosity examples.

**DFT and DFPT:** ground-state DFT provides electronic states and forces; DFPT supplies harmonic phonons, effective charges, and response tensors. A static DFT calculation does not itself produce a long real-time trajectory. Ab initio MD supplies nuclear trajectories for classical correlation estimates; electronic Kubo/optical calculations and harmonic phonon calculations instead assemble response or spectra from states, modes, and matrix elements.

**Before using a formula:** identify the applied field and its conjugate observable, decide whether you need a time integral or a finite-frequency spectrum, and check whether classical or quantum statistics govern the experiment. That choice determines the correlation to compute—and whether a Kubo coefficient, a scattering spectrum, or a DOS is the right object at all.
