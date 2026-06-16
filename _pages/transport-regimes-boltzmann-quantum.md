---
permalink: /blog/transport-regimes-boltzmann-quantum/
title: "Transport Regimes: Boltzmann, Ballistic, and Quantum"
excerpt: "A concise map of Boltzmann transport, regime boundaries by length scales, and how DFT+NEGF tools such as SIESTA/TranSIESTA compute quantum transport."
author_profile: true
---

# Transport Regimes: Boltzmann, Ballistic, and Quantum

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

Use BTE when carriers can be treated as wave packets with well-defined momentum between scattering events.

## 2. Practical Boltzmann Implementations

For electronic materials from first principles, the two common BTE pipelines are:

- **BoltzTraP/BoltzTraP2:** interpolate DFT bands and compute transport in constant-$\tau$ or model-$\tau$ settings.
- **EPW (electron-phonon Wannier):** compute $e$-ph matrix elements and scattering rates, then solve BTE with ab initio lifetimes.

Both are still semiclassical BTE frameworks; they differ mainly in how accurately $\tau$ and scattering are modeled.

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

## 6. How to Estimate $L_\phi$

Use

<div class="math-display">
$$
L_\phi=\sqrt{D\tau_\phi},
$$
</div>

with $D$ the diffusion constant and $\tau_\phi$ the dephasing time.

Typical extraction routes:

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
