---
permalink: /blog/transport-regimes-boltzmann-quantum/
title: "Transport Regimes: Boltzmann, Ballistic, and Quantum"
excerpt: "A concise map of Boltzmann transport, regime boundaries by length scales, and how DFT+NEGF tools such as SIESTA/TranSIESTA compute quantum transport."
author_profile: true
---

# Transport Regimes: Boltzmann, Ballistic, and Quantum

This note is a compact map for deciding which transport model is appropriate. I use: mean free path $\ell_{\mathrm{mfp}}$, phase-coherence length $L_\phi$, Fermi wavelength $\lambda_F$, and device/bulk length $L$.

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

## 2. Regime Map: Ballistic, Diffusive, Semiclassical, Quantum

Let $L$ be the transport length (channel length, film thickness, or probe spacing).

| Regime | Length-scale condition | Dominant picture | Typical model |
|---|---|---|---|
| **Ballistic semiclassical** | $L\ll \ell_{\mathrm{mfp}}$ and $L\gg \lambda_F$ | Few/no collisions, classical trajectories | Ballistic BTE / Landauer with classical channels |
| **Diffusive semiclassical** | $\ell_{\mathrm{mfp}}\ll L$ and $L_\phi\ll L$ | Many collisions, phase randomized | BTE + RTA, drift-diffusion |
| **Coherent quantum ballistic** | $L\ll \ell_{\mathrm{mfp}}$ and $L\ll L_\phi$ | Wave interference, mode quantization | Landauer-Buttiker, NEGF |
| **Coherent quantum diffusive** | $\ell_{\mathrm{mfp}}\ll L\lesssim L_\phi$ | Multiple scattering + interference | NEGF with disorder/self-energies, weak localization theory |

Three practical boundaries:

1. **Momentum-randomizing boundary:** $L/\ell_{\mathrm{mfp}}$ separates ballistic and diffusive transport.
2. **Phase-randomizing boundary:** $L/L_\phi$ determines whether interference survives.
3. **Bulk vs mesoscopic boundary:** when all internal scales satisfy $\ell_{\mathrm{mfp}},L_\phi\ll L$, transport is effectively bulk/local; when $L$ is comparable to either, nonlocal mesoscopic effects matter.

So, **Boltzmann is strongest in the semiclassical window** $L_\phi\ll L$ with quasiparticles and local scattering. It breaks down when phase coherence, tunneling, or strong confinement dominates.

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

## 5. Quick Model Selection Rule

If $\ell_{\mathrm{mfp}}\ll L$ and $L_\phi\ll L$ (with no strong tunneling/confinement): start with **Boltzmann**.  
If $L\lesssim L_\phi$ or channels are nanoscopic/quantized: use **quantum transport (Landauer/NEGF)**.  
If both scattering and coherence are relevant: use **NEGF with scattering self-energies** or a hybrid multiscale approach.

## References

- N. W. Ashcroft and N. D. Mermin, *Solid State Physics*
- S. Datta, *Electronic Transport in Mesoscopic Systems*
- S. Datta, *Quantum Transport: Atom to Transistor*
- M. Brandbyge et al., [Density-functional method for nonequilibrium electron transport](https://doi.org/10.1103/PhysRevB.65.165401)
- M. Paulsson and M. Brandbyge, [Transmission eigenchannels from NEGF](https://doi.org/10.1103/PhysRevB.76.115117)
