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

Thus response is linked to how a spontaneous fluctuation relaxes. Its time integral or its Fourier spectrum will give different physical outputs.

## 2. Quantum and Classical FDT

For a Hermitian observable $A$, define the **unsymmetrized quantum spectrum**

<div class="math-display">
$$
S_A^{>}(\omega)
=\int_{-\infty}^{\infty}e^{i\omega t}
\langle\delta A(t)\delta A(0)\rangle\,\mathrm dt.
$$
</div>

Equilibrium detailed balance and FDT give

<div class="math-display">
$$
S_A^{>}(-\omega)=e^{-\beta\hbar\omega}S_A^{>}(\omega),
\qquad
\chi''_{AA}(\omega)
=\frac{1-e^{-\beta\hbar\omega}}{2\hbar}S_A^{>}(\omega).
$$
</div>

Positive and negative frequencies therefore have different weights in a quantum spectrum. If we instead average the two orderings, $S_A^{\rm sym}=[S_A^{>}(\omega)+S_A^{>}(-\omega)]/2$, then

<div class="math-display">
$$
S_A^{\rm sym}(\omega)
=\hbar\coth\!\left(\frac{\beta\hbar\omega}{2}\right)\chi''_{AA}(\omega)
\;\xrightarrow{\;\beta\hbar\omega\ll1\;}\;
\frac{2k_{\rm B}T}{\omega}\chi''_{AA}(\omega).
$$
</div>

The rightmost expression is the **classical FDT** for the ordinary autocorrelation spectrum. Classical MD is useful when nuclear motion is adequately classical; it does not automatically reproduce quantum detailed balance, zero-point motion, or low-temperature occupation factors. These formulas use one Fourier convention throughout; changing the sign in the Fourier transform changes some intermediate signs, not the physics. [Kubo's original paper](https://doi.org/10.1143/JPSJ.12.570) and his [FDT review](https://www.zib.de/userpage/donati/stochastics2023/06/references/Kubo1966.pdf) develop the general theory.

## 3. Property Map: What Do We Correlate?

The **zero-frequency limit** of a current response often gives a transport coefficient. The **finite-frequency spectrum** gives absorption, scattering, or relaxation. Below, the Green–Kubo integrals are for equilibrium, isotropic three-dimensional systems; the heat formula assumes no coupled particle or charge flow. Let $V$ be the volume, $\mathbf v$ a tagged-particle velocity, $P_{xy}$ an off-diagonal stress, $\mathbf I=\sum_a q_a\mathbf v_a$ the **total** ionic charge current, and $\mathbf J_Q$ the **total** heat current (volume times the heat-flux density).

<div class="math-display">
$$
\begin{aligned}
D&=\frac13\int_0^\infty
  \langle\mathbf v(t)\!\cdot\!\mathbf v(0)\rangle\,\mathrm dt,\\
\eta&=\frac{V}{k_{\rm B}T}\int_0^\infty
  \langle\delta P_{xy}(t)\delta P_{xy}(0)\rangle\,\mathrm dt,\\
\sigma_{\rm ion}&=\frac{1}{3Vk_{\rm B}T}\int_0^\infty
  \langle\mathbf I(t)\!\cdot\!\mathbf I(0)\rangle\,\mathrm dt,\\
\kappa&=\frac{1}{3Vk_{\rm B}T^2}\int_0^\infty
  \langle\mathbf J_Q(t)\!\cdot\!\mathbf J_Q(0)\rangle\,\mathrm dt.
\end{aligned}
$$
</div>

These four formulas are the classical starting point, with tensor components used for anisotropic materials. The list below shows what changes when the target is a different property.

1. **Self-diffusivity.** Correlate one particle's velocity with its later velocity. In MD or ab initio MD, integrate the velocity autocorrelation above; the long-time mean-square displacement gives the equivalent Einstein route, $\langle|\mathbf r(t)-\mathbf r(0)|^2\rangle\sim6Dt$. A vibrational velocity spectrum uses a Fourier transform instead of this time integral.

2. **Shear viscosity.** Correlate an off-diagonal stress with itself. The area under the stress autocorrelation gives $\eta$; a frequency-resolved stress response gives viscoelastic behavior. Slow stress tails make the integral noisy, so a flat region in the running integral and independent uncertainty estimates matter.

3. **Ionic conductivity.** Correlate the **total charge current**, not merely each ion's self-diffusion. Expanding $\mathbf I(t)\cdot\mathbf I(0)$ includes correlations between different ions. Dropping those cross terms gives the Nernst–Einstein approximation, which can fail in concentrated electrolytes.

4. **Thermal conductivity, often phonon dominated.** Correlate the microscopic heat current. Classical equilibrium MD gives the Green–Kubo $\kappa$ above; DFT-based force constants plus phonon scattering and the Boltzmann transport equation are a different route to phonon thermal conductivity. The heat-current definition and finite-size convergence deserve special care; [LAMMPS documents a practical Green–Kubo workflow](https://docs.lammps.org/compute_heat_flux.html).

5. **Electronic conductivity.** Choose electronic current as both the perturbation-linked and measured quantity. The frequency-dependent *quantum* current response gives optical conductivity, and its low-frequency limit gives DC transport when that limit is well defined. Band energies and current matrix elements from DFT lead to independent-particle Kubo–Greenwood calculations. The full conductivity also has a diamagnetic/contact contribution, so it is more than a bare current autocorrelation. Our [optical-properties note](/blog/optical-properties-kubo-dft-gw/) develops that route.

6. **Thermoelectric coefficients.** Use **cross-correlations** between charge and heat currents. Together with the two autocorrelations, they form an Onsager response matrix from which quantities such as the Seebeck coefficient are constructed. This is why a single current autocorrelation cannot capture coupled heat and charge flow.

7. **Electric or magnetic susceptibility.** Correlate polarization with polarization, or magnetization with magnetization. The retarded response gives $\chi(\omega)$; its imaginary part describes absorption. A classical static susceptibility can also be obtained from an equal-time variance of the field-conjugate quantity (with the appropriate volume and unit factors). In quantum systems, static response generally requires more than that simple equal-time variance.

8. **Infrared (IR) absorption.** The field couples to the electric dipole or polarization $\mathbf M$. The absorptive signal tracks $\omega\chi''_{MM}(\omega)$, up to optical and geometry factors. In MD, one often Fourier-transforms a dipole or dipole-velocity autocorrelation; in harmonic DFT/DFPT, Born effective charges and phonon modes determine IR activity.

9. **Nonresonant Raman scattering.** Use the fluctuating polarizability tensor rather than the dipole. A polarization-projected polarizability correlation gives the Raman spectral shape, with quantum occupation and optical prefactors added for measured Stokes or anti-Stokes intensities. In DFPT, mode derivatives of polarizability become Raman tensors. [Quantum ESPRESSO's phonon tools](https://quantum-espresso.org/Doc/INPUT_DYNMAT.html) implement the harmonic IR/Raman route.

10. **Neutron/X-ray scattering and phonon dispersion.** Correlate density at wavevector $\mathbf q$: $S(\mathbf q,\omega)\propto\int\mathrm dt\,e^{i\omega t}\langle\rho_{\mathbf q}(t)\rho_{-\mathbf q}(0)\rangle$. Peaks reveal collective modes; their width reflects finite lifetime. This is a space-and-time correlation, rather than just a local autocorrelation.

11. **Phonon or vibrational DOS.** In a harmonic calculation, count mode frequencies: $g_{\rm ph}(\omega)\propto\sum_{\mathbf q\nu}\delta(\omega-\omega_{\mathbf q\nu})$. From MD, the Fourier transform of a **mass-weighted velocity autocorrelation** gives a vibrational spectrum related to the DOS in the classical harmonic limit; anharmonicity broadens it. [Quantum ESPRESSO's matdyn.x](https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html) computes a harmonic phonon DOS, while [GROMACS gmx dos](https://manual.gromacs.org/current/onlinehelp/gmx-dos.html) uses an MD correlation.

12. **Electronic DOS.** A DFT band calculation counts states at energy $E$, $g_{\rm KS}(E)\propto\sum_{n\mathbf k}\delta(E-\epsilon_{n\mathbf k})$. For interacting electrons the analogous object is the one-particle spectral function, $g(E)\propto-\operatorname{Im}\operatorname{Tr}G^R(E)/\pi$. It is a **one-particle Green-function spectrum**, not a charge-current Kubo coefficient. DFT codes compute the Kohn–Sham DOS directly; [Quantum ESPRESSO's DOS tools](https://quantum-espresso.org/Doc/pp_user_guide/node9.html) are one example.

The last two entries are deliberately on the list: a DOS is a spectrum, but not every spectrum is a dissipative response. Likewise, two observables can have similar peaks yet different selection rules: a mode may appear in the phonon DOS but be weak in IR and strong in Raman.

## 4. MD, DFT, and the Practical Workflow

**MD:** record the relevant time series, subtract equilibrium means when needed, average correlations over many time origins, and integrate or Fourier-transform. Check the time window, sample spacing, finite-size effects, and uncertainty. Transport estimates need a converged running integral; spectra need enough duration for frequency resolution. [GROMACS' correlation-function guide](https://manual.gromacs.org/current/reference-manual/analysis/correlation-function.html) discusses diffusion and viscosity examples.

**DFT and DFPT:** ground-state DFT provides electronic states and forces; DFPT supplies harmonic phonons, effective charges, and response tensors. A static DFT calculation does not itself produce a long real-time trajectory. Ab initio MD supplies nuclear trajectories for classical correlation estimates; electronic Kubo/optical calculations and harmonic phonon calculations instead assemble response or spectra from states, modes, and matrix elements.

**Before using a formula:** identify the applied field and its conjugate observable, decide whether you need a time integral or a finite-frequency spectrum, and check whether classical or quantum statistics govern the experiment. That choice determines the correlation to compute—and whether a Kubo coefficient, a scattering spectrum, or a DOS is the right object at all.
