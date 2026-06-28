---
permalink: /blog/transport-regimes-boltzmann-quantum/
title: "Boltzmann Transport and Why It Breaks Down"
excerpt: "A concise map of Boltzmann transport, regime boundaries by length scales, and how DFT+NEGF tools such as SIESTA/TranSIESTA compute quantum transport."
author_profile: true
---

# Boltzmann Transport and Why It Breaks Down

This note is a compact map for deciding which transport model is appropriate. I use four scales: mean free path $\ell_{\mathrm{mfp}}$, phase-coherence length $L_\phi$, Fermi wavelength $\lambda_F$ (equivalently $k_F=2\pi/\lambda_F$), and device/bulk length $L$.

## 1. Boltzmann Transport in One Page

The semiclassical Boltzmann transport equation (BTE) evolves the distribution $f(\mathbf r,\mathbf k,t)$:

<div class="math-display">
$$
\frac{\partial f}{\partial t}
+\mathbf v_{\mathbf k}\cdot\nabla_{\mathbf r}f
+\frac{q\mathbf E}{\hbar}\cdot\nabla_{\mathbf k}f
=
\left.\frac{\partial f}{\partial t}\right|_{\mathrm{coll}}.
$$
</div>

With the relaxation-time approximation (RTA),

<div class="math-display">
$$
\left.\frac{\partial f}{\partial t}\right|_{\mathrm{coll}}
=-\frac{f-f_0}{\tau},
\qquad
\ell_{\mathrm{mfp}}=v_F\tau.
$$
</div>

In linear response, this gives Drude-like conductivity:

<div class="math-display">
$$
\sigma=\frac{nq^2\tau}{m^\ast},
\qquad
\mu=\frac{q\tau}{m^\ast}.
$$
</div>

For thermoelectrics, we keep the full energy dependence and explicitly linearize around equilibrium:

<div class="math-display">
$$
f_{n\mathbf k}=f_0(\varepsilon_{n\mathbf k})+\delta f_{n\mathbf k},
\qquad
|\delta f_{n\mathbf k}|\ll f_0.
$$
</div>

For steady, weakly non-uniform transport (linear response), keep only first-order terms:

<div class="math-display">
$$
\frac{\partial f}{\partial t}=0,\qquad
\mathbf v\cdot\nabla_{\mathbf r}\delta f\approx 0,\qquad
\left.\frac{\partial f}{\partial t}\right|_{\mathrm{coll}}
=-\frac{\delta f}{\tau_{n\mathbf k}}.
$$
</div>

So BTE becomes

<div class="math-display">
$$
\mathbf v_{n\mathbf k}\cdot\nabla_{\mathbf r} f_0
+\frac{(-e)\mathbf E}{\hbar}\cdot\nabla_{\mathbf k} f_0
=-\frac{\delta f_{n\mathbf k}}{\tau_{n\mathbf k}}.
$$
</div>

Use chain rules

<div class="math-display">
$$
\nabla_{\mathbf k} f_0
=\frac{\partial f_0}{\partial \varepsilon}\nabla_{\mathbf k}\varepsilon_{n\mathbf k}
=\hbar \mathbf v_{n\mathbf k}\frac{\partial f_0}{\partial \varepsilon},
$$
</div>

and (taking $\nabla\mu=0$ in the common BoltzTraP-style setup)

<div class="math-display">
$$
\nabla_{\mathbf r} f_0
=\frac{\partial f_0}{\partial T}\nabla T
=\frac{\varepsilon_{n\mathbf k}-\mu}{T}
\left(-\frac{\partial f_0}{\partial \varepsilon}\right)\nabla T.
$$
</div>

Substituting these into BTE gives the perturbation

<div class="math-display">
$$
\delta f_{n\mathbf k}
=
-\tau_{n\mathbf k}\,
\mathbf v_{n\mathbf k}\!\cdot\!
\left[
-e\mathbf E
+\frac{\varepsilon_{n\mathbf k}-\mu}{T}\nabla T
\right]
\left(-\frac{\partial f_0}{\partial \varepsilon}\right),
$$
</div>

Why this directly gives transport properties: electric and heat currents are moments of the nonequilibrium part $\delta f$:

<div class="math-display">
$$
J_\alpha
=
\frac{-e}{N_k\Omega}\sum_{n\mathbf k} v_{n\mathbf k,\alpha}\,\delta f_{n\mathbf k},
\qquad
J_{Q,\alpha}
=
\frac{1}{N_k\Omega}\sum_{n\mathbf k}(\varepsilon_{n\mathbf k}-\mu)\,v_{n\mathbf k,\alpha}\,\delta f_{n\mathbf k}.
$$
</div>

Insert $\delta f_{n\mathbf k}$, collect terms proportional to $E_\beta$ and $-\nabla_\beta T$, and define the common moments

<div class="math-display">
$$
\mathcal L_{\alpha\beta}^{(m)}
=
\frac{1}{N_k\Omega}
\sum_{n\mathbf k}
\tau_{n\mathbf k}\,
v_{n\mathbf k,\alpha}v_{n\mathbf k,\beta}
(\varepsilon_{n\mathbf k}-\mu)^m
\left(-\frac{\partial f_0}{\partial \varepsilon}\right).
$$
</div>

Then

<div class="math-display">
$$
J_\alpha
=
e^2\mathcal L_{\alpha\beta}^{(0)}E_\beta
+\frac{e}{T}\mathcal L_{\alpha\beta}^{(1)}(-\nabla_\beta T),
$$
</div>

<div class="math-display">
$$
J_{Q,\alpha}
=
e\mathcal L_{\alpha\beta}^{(1)}E_\beta
+\frac{1}{T}\mathcal L_{\alpha\beta}^{(2)}(-\nabla_\beta T).
$$
</div>

So the "$2\times2$ matrix" is just the compact Onsager form of these two coupled linear equations (charge and heat currents vs electric and thermal driving forces):

Using Onsager form for charge/heat currents,

<div class="math-display">
$$
\begin{pmatrix}\mathbf J\\ \mathbf J_Q\end{pmatrix}
=
\begin{pmatrix}
e^2\mathcal L^{(0)} & \dfrac{e}{T}\mathcal L^{(1)} \\
e\mathcal L^{(1)} & \dfrac{1}{T}\mathcal L^{(2)}
\end{pmatrix}
\begin{pmatrix}\mathbf E\\ -\nabla T\end{pmatrix},
$$
</div>

the electronic thermoelectric tensors are obtained by taking $\sigma$ from $\mathbf J$ vs $\mathbf E$ at $\nabla T=0$, $S$ from the open-circuit condition $\mathbf J=0$, and $\kappa_e$ from $\mathbf J_Q$ vs $-\nabla T$ at $\mathbf J=0$:

<div class="math-display">
$$
\sigma_{\alpha\beta}=e^2\mathcal L_{\alpha\beta}^{(0)},
\qquad
S_{\alpha\beta}
=-\frac{1}{eT}
\left(\mathcal L^{(0)-1}\mathcal L^{(1)}\right)_{\alpha\beta},
$$
</div>

<div class="math-display">
$$
\kappa_{e,\alpha\beta}
=\frac{1}{T}
\left[
\mathcal L^{(2)}
-\mathcal L^{(1)}\mathcal L^{(0)-1}\mathcal L^{(1)}
\right]_{\alpha\beta},
\qquad
\mathrm{PF}_{\alpha}=S_{\alpha}^2\sigma_{\alpha}.
$$
</div>

Finally, the figure of merit is

<div class="math-display">
$$
zT_{\alpha}
=
\frac{S_{\alpha}^2\sigma_{\alpha}T}
{\kappa_{e,\alpha}+\kappa_{L,\alpha}}.
$$
</div>

Use BTE when carriers can be treated as wave packets with well-defined momentum between scattering events.

## 2. Practical Boltzmann Implementations

For electronic materials from first principles, the two common BTE pipelines are:

- **BoltzTraP/BoltzTraP2:** interpolate DFT bands and compute transport in constant-$\tau$ or model-$\tau$ settings.
- **EPW (electron-phonon Wannier):** compute $e$-ph matrix elements and scattering rates, then solve BTE with ab initio lifetimes.

Both are still semiclassical BTE frameworks; they differ mainly in how accurately $\tau$ and scattering are modeled.

For an ab initio thermoelectric workflow (RTA):

1. **Electronic structure from DFT:** compute $\varepsilon_{n\mathbf k}$ on dense meshes (often via Wannier/Fourier interpolation).
2. **Band velocities:** evaluate $\mathbf v_{n\mathbf k}=\hbar^{-1}\nabla_{\mathbf k}\varepsilon_{n\mathbf k}$.
3. **Lifetimes $\tau_{n\mathbf k}$:**
   - constant-$\tau$ / fitted-$\tau$ (BoltzTraP-style), or
   - explicit $e$-ph scattering from supercell finite-displacement data (e.g., VASP + phono3py style) or Wannier interpolation (EPW-style), e.g. from Fermi's golden rule.
4. **Transport moments:** build $\mathcal L^{(0,1,2)}(\mu,T)$ and obtain $\sigma$, $S$, $\kappa_e$, and PF.
5. **Lattice thermal conductivity:** compute $\kappa_L$ from phonon BTE using 2nd/3rd-order IFCs (typically finite-displacement supercells).
6. **Figure of merit:** combine to get $zT(\mu,T)$ and map optimal carrier concentration/temperature windows.

This is the standard ab initio route used in recent RTA thermoelectric studies (including the methodology of arXiv:2511.15249), with differences mainly in how $\tau_{n\mathbf k}$ and $\kappa_L$ are treated.

## 3. Quantum Transport (Landauer + NEGF)

For coherent two-terminal transport, current is

<div class="math-display">
$$
I(V)=\frac{2e}{h}\int dE\;T(E,V)\bigl[f_L(E)-f_R(E)\bigr].
$$
</div>

In NEGF, transmission is

<div class="math-display">
$$
T(E)=\mathrm{Tr}\!\left[\Gamma_L G^r \Gamma_R G^a\right],
$$
</div>

with

<div class="math-display">
$$
G^r(E)=\bigl[E S-H_C-\Sigma_L^r-\Sigma_R^r\bigr]^{-1},
\qquad
\Gamma_\alpha=i\left(\Sigma_\alpha^r-\Sigma_\alpha^a\right).
$$
</div>

$H_C$ and $S$ are the device-region Hamiltonian and overlap (non-orthogonal basis in many DFT codes), and $\Sigma_{L/R}$ encode open-boundary coupling to semi-infinite electrodes.

## 4. How SIESTA/TranSIESTA-Style Quantum Transport Works

SIESTA provides localized-orbital DFT Hamiltonians; TranSIESTA performs NEGF transport on top.

<div class="algorithm-block">
  <div class="algorithm-title">Workflow. DFT+NEGF transport calculation</div>
  <ol>
    <li>Run electrode DFT for left/right bulk leads to obtain converged lead Hamiltonians.</li>
    <li>Build a device region (left lead layers + scatterer + right lead layers).</li>
    <li>Compute lead self-energies $\Sigma_{L/R}(E)$ from electrode surface Green's functions.</li>
    <li>Solve the open-device Green's function $G^r(E)$ self-consistently with charge density.</li>
    <li>Evaluate $T(E)$, then integrate to get $I(V)$ and differential conductance.</li>
  </ol>
</div>

Important interpretation:

- **DFT** gives the effective single-particle Hamiltonian.
- **NEGF** imposes open boundaries and nonequilibrium occupations.
- **Landauer formula** converts transmission into measurable current.

This framework naturally captures tunneling, resonances, contact effects, and atomistic chemistry that semiclassical BTE cannot represent.

## 5. Three Breakdown Tests for Boltzmann

Let $L$ be device length, $\ell_{\mathrm{mfp}}$ mean free path, and $L_\phi$ coherence length.

1. **Ioffe-Regel / quasiparticle test:**  
   $\lambda_F\ll \ell_{\mathrm{mfp}}$ (equivalently $k_F\ell_{\mathrm{mfp}}\gg1$) supports Fermi-liquid quasiparticles and BTE.  
   If $k_F\ell_{\mathrm{mfp}}\sim1$, Boltzmann breaks down (strong disorder/localization) and quantum methods are required.

2. **Diffusive test:**  
   $L\gg \ell_{\mathrm{mfp}}$ gives diffusive transport (bulk BTE/Drude regime).  
   If this fails ($L\lesssim\ell_{\mathrm{mfp}}$), transport is ballistic: bulk diffusive BTE is no longer right, boundary conditions dominate, and for nanoscale electrons Landauer/NEGF is typically the clean choice.

3. **Coherence test:**  
   $L\gg L_\phi$ gives semiclassical incoherent transport.  
   If this fails ($L_\phi\gtrsim L$), phase coherence survives across the device and quantum transport is needed.

In short, Boltzmann is safest when

<div class="math-display">
$$
\lambda_F\ll \ell_{\mathrm{mfp}}\ll L,
\qquad
L_\phi\ll L,
\qquad
k_F\ell_{\mathrm{mfp}}\gg1.
$$
</div>

## 6. Scattering: Elastic vs Inelastic

Different scattering mechanisms contribute differently to $\ell_{\mathrm{mfp}}$ and $\tau_\phi$:

**Elastic scattering** (impurities, defects, boundaries):
- Randomizes momentum $\Rightarrow$ contributes to $1/\ell_{\mathrm{mfp}}$.
- Preserves phase coherence (no energy loss).
- Does **not** reduce $\tau_\phi$ or $L_\phi$.

**Inelastic scattering** (electron-phonon, electron-electron, magnons, etc.):
- Randomizes momentum $\Rightarrow$ contributes to $1/\ell_{\mathrm{mfp}}$.
- Breaks phase coherence (energy/information loss) $\Rightarrow$ defines dephasing time $\tau_\phi$ and thus $L_\phi$.

In BTE frameworks like BoltzTraP or EPW, electron-phonon scattering provides the dominant inelastic contribution to lifetimes and thus to $\tau_\phi$.

## 7. How to Estimate $L_\phi$

Use

<div class="math-display">
$$
L_\phi=\sqrt{D\tau_\phi},
$$
</div>

with $D$ the diffusion constant and $\tau_\phi$ the dephasing time from inelastic processes.

**Getting the diffusion constant $D$:**

From Einstein relation,
<div class="math-display">
$$
D=\frac{\sigma}{e\cdot N(E_F)}=\frac{v_F^2\tau}{d}=\frac{v_F\ell_{\mathrm{mfp}}}{d},
$$
</div>

where $\sigma$ is conductivity, $e$ is elementary charge, $N(E_F)$ is density of states at Fermi level, $d$ is dimensionality, and $\ell_{\mathrm{mfp}}=v_F\tau$. Experimentally, extract from resistivity and Hall effect, or compute from BTE/ab initio (BoltzTraP, EPW).

**Getting the dephasing time $\tau_\phi$ via Matthiessen's rule:**

Inelastic scattering rates add inversely:

<div class="math-display">
$$
\frac{1}{\tau_\phi}=\sum_i\frac{1}{\tau_i^{\phi}},
$$
</div>

where $\tau_i^{\phi}$ are individual dephasing times ($e$-$e$, $e$-ph, magnon-scattering, etc.). Each temperature-dependent:

- **Electron-electron ($e$-$e$):** $1/\tau_{ee}^{\phi}\propto T$ (in 2D) or $\propto T^2$ (in 3D).
- **Electron-phonon ($e$-$ph$):** typically weaker; $\propto T$ at high $T$, stronger at low $T$ in some materials.

Then compute $L_\phi=\sqrt{D\tau_\phi}$ combining the Einstein $D$.

**Typical experimental extraction routes:**

1. Weak localization / anti-localization magnetoconductance fits (HLN-type).
2. Universal conductance fluctuation correlation field.
3. Aharonov-Bohm oscillation visibility vs temperature/size.
4. Microscopic dephasing rates ($e$-$e$, $e$-ph) combined with $L_\phi=\sqrt{D\tau_\phi}$.

## References

- N. W. Ashcroft and N. D. Mermin, *Solid State Physics*
- S. Datta, *Electronic Transport in Mesoscopic Systems*
- S. Datta, *Quantum Transport: Atom to Transistor*
- G. K. H. Madsen and D. J. Singh, [BoltzTraP](https://doi.org/10.1016/j.cpc.2006.03.007)
- J. Ponce et al., [EPW: electron-phonon coupling and transport](https://doi.org/10.1016/j.cpc.2016.07.028)
- M. Brandbyge et al., [Density-functional method for nonequilibrium electron transport](https://doi.org/10.1103/PhysRevB.65.165401)
- M. Paulsson and M. Brandbyge, [Transmission eigenchannels from NEGF](https://doi.org/10.1103/PhysRevB.76.115117)
