---
permalink: /blog/x-ray-absorption-spectroscopy/
title: "Reading X-ray Absorption Spectra: Edges, Regions, and First-Principles Models"
excerpt: "A guide to pre-edge, XANES, EXAFS, core-level labels, and the calculations behind Materials Project spectra."
author_profile: true
---

# Reading X-ray Absorption Spectra

An X-ray absorption spectrum looks simple at first: tune the photon energy and record how strongly a sample absorbs. Yet one scan can say which element absorbed the photon, which core orbital the electron left, how that atom is bonded, and how its neighbors are arranged. The trick is that different parts of the spectrum speak different dialects.

Experimentally, the absorption coefficient follows the Beer-Lambert law,

<div class="math-display">
$$
I(E,t)=I_0(E)e^{-\mu(E)t},
$$
</div>

where $t$ is the sample thickness. Microscopically, absorption promotes a localized core electron into an unoccupied state. For an $N$-electron system in the electric-dipole approximation,

<div class="math-display">
$$
\mu(\omega)\propto
\sum_F
\left\lvert
\left\langle \Psi_F^N \right\rvert \hat D
\left\lvert \Psi_0^N \right\rangle
\right\rvert^2
\delta(E_F^N-E_0^N-\hbar\omega),
\qquad
\hat D=\sum_{i=1}^{N}\boldsymbol{\epsilon}\cdot\mathbf r_i.
$$
</div>

This compact expression already contains most of XAS. The initial state $\lvert\Psi_0^N\rangle$ contains the occupied core orbital, the many-body final states $\lvert\Psi_F^N\rangle$ contain the excited electron *and* the core hole, and the polarization $\boldsymbol{\epsilon}$ selects orbital directions. Element specificity comes from choosing which localized core orbital is excited.

## What does “Cr K-edge” mean?

The two parts specify different things:

- **Cr** is the absorbing element. The excitation begins on a chromium atom, even if the final state is strongly hybridized with oxygen or another neighbor.
- **K** labels the initial core shell: K means $1s$. Likewise, $L_1$ means $2s$, while $L_2$ and $L_3$ are the spin-orbit-split $2p_{1/2}$ and $2p_{3/2}$ shells.

Thus the Cr K-edge is the onset of transitions out of a Cr $1s$ core state, near 5989 eV for elemental chromium. The precise edge position is not universal: oxidation, screening, and local bonding shift it, which is why edge shifts are often used as oxidation-state fingerprints.

For a dipole transition, $\Delta l=\pm1$, so the strongest Cr K-edge channel has $1s\rightarrow p$ character. A weak $1s\rightarrow3d$ pre-edge feature can still appear through electric-quadrupole coupling or through $3d$-$4p$ mixing when local symmetry allows it. In other words, the edge name identifies the *starting shell*, not a single destination orbital.

<figure style="margin: 1.5rem 0; text-align: center;">
  <img src="/images/xas-regions.svg" alt="Schematic XAS curve divided into pre-edge, XANES, and EXAFS regions" style="width: 100%; max-width: 900px;">
  <figcaption style="font-size: 0.9em; color: #555;">A schematic division of XAS. The boundaries are useful conventions, not sharp laws of nature.</figcaption>
</figure>

## The three regions

### 1. Pre-edge: the quiet features with loud implications

The pre-edge lies just below the main threshold. Its peaks correspond to bound or quasi-bound final states and are usually much weaker than the main edge. Because their intensity depends strongly on selection rules and local symmetry, they can reveal oxidation state, spin state, coordination geometry, and inversion symmetry.

For a transition-metal K-edge, a centrosymmetric octahedral site often has a weak $1s\rightarrow3d$ pre-edge. Distortion or tetrahedral coordination mixes metal $p$ character into the $d$-dominated states, making the same feature dipole-accessible and much stronger. This is a good reminder that XAS is not simply an unoccupied density of states: matrix elements decide what becomes bright.

### 2. XANES: the near-edge many-body neighborhood

XANES (X-ray absorption near-edge structure), roughly the first 30-50 eV above $E_0$, contains the edge jump, possible white line, and several resonances. Here the photoelectron has a long wavelength and scatters strongly from several nearby atoms. At the same time, the positively charged core hole attracts the excited electron and can form a core exciton. Both multiple scattering and electron-hole correlation can matter.

This region is especially sensitive to oxidation state and coordination geometry. Higher oxidation often shifts an edge upward, but the line shape also depends on covalency, screening, spin-orbit coupling, and experimental broadening. A peak should therefore not be assigned from the projected density of states alone.

### 3. EXAFS: a local interferometer

Farther above the edge, usually beyond about 30-50 eV, the excited electron behaves more like a propagating photoelectron. Its outgoing wave scatters from neighboring atoms and returns to interfere at the absorber. Converting energy to photoelectron wave number,

<div class="math-display">
$$
k=\frac{\sqrt{2m_e(E-E_0)}}{\hbar},
\qquad
\chi(k)=\frac{\mu(k)-\mu_0(k)}{\mu_0(k)},
$$
</div>

exposes the oscillatory fine structure. A standard single-scattering expression is

<div class="math-display">
$$
\chi(k)=\sum_j
\frac{N_jS_0^2F_j(k)}{kR_j^2}
e^{-2R_j/\lambda(k)}e^{-2\sigma_j^2k^2}
\sin\!\left[2kR_j+\delta_j(k)\right].
$$
</div>

Each neighbor shell $j$ contributes through its coordination number $N_j$, distance $R_j$, scattering amplitude $F_j$, disorder $\sigma_j^2$, and phase shift $\delta_j$. The mean free path $\lambda$ damps long trajectories, while $S_0^2$ accounts for many-electron amplitude loss. Fourier transforming $k^n\chi(k)$ produces peaks near coordination-shell distances, but phase shifts mean those peaks are **not** uncorrected radial-distribution functions.

## Which method is reliable in which region?

The energy boundaries are approximate, and an unusually strong exciton can move them. Still, this is a useful working map.

**Pre-edge and the first 10-20 eV above $E_0$.** Bound states, core excitons, dipole-forbidden channels, and multiplets are most visible here. For periodic solids, use a core-level **BSE** implementation such as **OCEAN** or **exciting** when peak positions and oscillator-strength transfer matter. For molecules and finite complexes, restricted-window linear-response **TDDFT** in **NWChem** is often practical; real-time TDDFT is available in **NWChem** and **Octopus**. **FDMNES** is valuable when full-potential shape, polarization, or quadrupole transitions control a K-edge pre-edge. A static core-hole DFT calculation can reproduce broad trends but is less trustworthy for a tightly bound exciton or several coherently mixed transitions.

**Main XANES, roughly 10-50 eV above $E_0$.** Several-scattering paths overlap and the photoelectron wavelength is comparable to interatomic distances. **FEFF** with self-consistent potentials and full multiple scattering is a strong default for K-edge structure and rapid comparison of many candidate geometries. **FDMNES** is preferable when the muffin-tin potential is questionable or detailed dichroism matters. **XSpectra** in Quantum ESPRESSO and core-hole calculations in **VASP** provide plane-wave supercell alternatives. If an independent-particle calculation misses a sharp onset, white line, or strong intensity redistribution, move to **OCEAN/exciting BSE** rather than merely shifting the energy axis.

**EXAFS, normally beyond about 50 eV.** The path expansion is controlled because the photoelectron wavelength is short and inelastic damping suppresses very long paths. **FEFF** is the standard scattering engine; **Larch/Larix** or **Athena/Artemis** performs normalization, background removal, Fourier transforms, and fitting of FEFF paths. This is the most quantitatively reliable region for extracting bond lengths and disorder. Coordination numbers are less independent because they correlate with $S_0^2$ and $\sigma^2$. BSE or TDDFT is unnecessary and inefficient for hundreds of eV of EXAFS oscillations.

**Transition-metal $L_{2,3}$ and rare-earth $M$ edges need an extra warning.** The $2p$ or $3d$ core spin-orbit splitting and open-shell multiplets can dominate even if the photon energy is high. BSE codes can include spin-orbit coupling and some multiplet interactions, but strongly localized $d$ or $f$ shells may require ligand-field or charge-transfer multiplet calculations with **Quanty** or **CTM4XAS**. “Near edge” is therefore a statement about energy *relative to that edge*, not about hard versus soft X-rays.

## From the exact many-body spectrum to a Green's function

Let $E_\gamma=\hbar\omega$ be the photon energy, $n_{\rm abs}$ the number density of equivalent absorbers, and $\alpha$ the fine-structure constant. In atomic units, the dipole absorption cross section is

<div class="math-display">
$$
\sigma(E_\gamma)=4\pi^2\alpha E_\gamma
\sum_F
\left\lvert
\left\langle \Psi_F^N \right\rvert \hat D
\left\lvert \Psi_0^N \right\rangle
\right\rvert^2
\delta(E_F^N-E_0^N-E_\gamma),
\qquad
\mu(E_\gamma)=n_{\rm abs}\sigma(E_\gamma).
$$
</div>

Here $\lvert\Psi_0^N\rangle$ is the interacting ground state, $\lvert\Psi_F^N\rangle$ is an exact final state with one core hole, and $\hat D$ is the many-electron dipole operator. Using $\delta(x)=-\pi^{-1}\operatorname{Im}(x+i0^+)^{-1}$ converts the explicit sum over final states into a resolvent:

<div class="math-display">
$$
\sigma(E_\gamma)=
-4\pi\alpha E_\gamma\,
\operatorname{Im}
\left\langle \Psi_0^N \right\rvert
\hat D^\dagger
\frac{1}{E_0^N+E_\gamma-\hat H+i\Gamma}
\hat D
\left\lvert \Psi_0^N \right\rangle .
$$
</div>

$\hat H$ is the full electronic Hamiltonian. The positive broadening $\Gamma$ represents the core-hole lifetime, photoelectron lifetime, instrumental resolution, or a convolution of them; formally one takes $\Gamma\rightarrow0^+$. This equation is still exact. Its propagator is a many-body electron-hole response, so core excitons, multiplets, shake-up, and shake-off channels are present if the final states are solved exactly. FEFF, TDDFT, and BSE differ mainly in how they approximate this propagator.

## FEFF: a one-photoelectron Green's function

FEFF reduces the many-electron resolvent to a quasiparticle moving in the potential of the nuclei, the other electrons, and a screened core hole. If $\phi_c$ is the selected core orbital and $d=\boldsymbol{\epsilon}\cdot\mathbf r$ is its one-electron dipole operator, the working form is

<div class="math-display">
$$
\mu(E_\gamma)\propto
-E_\gamma\operatorname{Im}
\left\langle \phi_c\right\rvert
d^\dagger G^R(E)d
\left\lvert\phi_c\right\rangle,
\qquad
G^R(E)=
\left[E-h_{\rm ch}-\Sigma(E)+i\Gamma_c\right]^{-1}.
$$
</div>

$E=E_c+E_\gamma$ is the final one-electron energy in the code's energy convention, $h_{\rm ch}$ is the effective Hamiltonian including the screened core-hole potential, $\Gamma_c$ is the intrinsic core-level width, and the complex self-energy $\Sigma(E)$ describes quasiparticle shifts and inelastic losses. Its real part moves spectral features; its imaginary part limits the mean free path. The matrix element projects the full final-state propagator onto waves that can be launched from, and return to, the absorbing atom.

FEFF builds $G^R$ in real space from single-site scattering matrices $t_i$:

<div class="math-display">
$$
G^R=G_c+G_c tG_c+G_c tG_c tG_c+\cdots .
$$
</div>

$G_c$ propagates in the central-atom reference potential, and each $t_i$ scatters the photoelectron from site $i$. Near the edge, FEFF sums this series by a full multiple-scattering matrix inversion over a finite cluster. At high kinetic energy it reorganizes the same series into geometrical paths, which leads to the EXAFS equation above.

The efficiency comes with identifiable approximations: a one-active-electron or sudden picture; self-consistent but usually spherical muffin-tin scattering potentials; a static screened core hole; model or many-pole $GW$-like self-energies; phenomenological or calculated lifetime and vibrational damping; and an amplitude factor $S_0^2$ for spectral weight lost to many-electron channels. FEFF therefore handles continuum propagation and EXAFS exceptionally well, and often gives useful K-edge XANES, but it does not explicitly diagonalize a correlated electron-hole Hamiltonian. Deep bound excitons, strong charge-transfer multiplets, and delicate pre-edge intensities are the places to be cautious.

This is also how the XAS data associated with the **Materials Project** were generated. The high-throughput workflow used crystal structures from MP, selected every symmetrically distinct absorbing site, built a local cluster, and ran FEFF9. The original release contained hundreds of thousands of site-resolved K-edge XANES spectra. So the MP curves are first-principles real-space multiple-scattering calculations—not TDDFT trajectories. They are excellent for comparing candidate structures and local environments, but finite-temperature disorder, defects, surface chemistry, and energy alignment can still separate a calculated ideal crystal from an experiment.

## TDDFT: approximate the density-response kernel

TDDFT keeps the many-electron response at the density level. The interacting density-response function $\chi$ satisfies the Dyson-like equation

<div class="math-display">
$$
\chi(\omega)=\chi_s(\omega)
+\chi_s(\omega)\left[v_C+f_{\rm xc}(\omega)\right]\chi(\omega),
\qquad
f_{\rm xc}(\mathbf r,\mathbf r',\omega)
=\frac{\delta v_{\rm xc}(\mathbf r,\omega)}
{\delta n(\mathbf r',\omega)}.
$$
</div>

$\chi_s$ is the independent Kohn-Sham response, $v_C$ is the bare Coulomb interaction, and $f_{\rm xc}$ is the exchange-correlation kernel. Spatial integrations between neighboring factors are implicit. The polarization-resolved polarizability is obtained by projecting $\chi$ with the dipole operator,

<div class="math-display">
$$
\alpha_{\epsilon\epsilon}(\omega)
=-\int d\mathbf r\,d\mathbf r'
(\boldsymbol{\epsilon}\cdot\mathbf r)
\chi(\mathbf r,\mathbf r',\omega)
(\boldsymbol{\epsilon}\cdot\mathbf r'),
\qquad
\mu(\omega)\propto\omega\operatorname{Im}\alpha_{\epsilon\epsilon}(\omega).
$$
</div>

In linear-response TDDFT, one commonly restricts the transition space to a chosen core orbital $c$ and unoccupied orbitals $a$. In the Tamm-Dancoff form,

<div class="math-display">
$$
\sum_{c'a'}A_{ca,c'a'}X_{c'a'}^S=\Omega_S X_{ca}^S,
\qquad
A_{ca,c'a'}=(\varepsilon_a-\varepsilon_c)
\delta_{cc'}\delta_{aa'}+K^{\rm Hxc}_{ca,c'a'}.
$$
</div>

$K^{\rm Hxc}$ contains matrix elements of $v_C+f_{\rm xc}$, so TDDFT shifts and mixes the bare Kohn-Sham core transitions. Restricted excitation window or core-valence separation removes the enormous number of irrelevant valence excitations; this is the **REW-TDDFT** route implemented in NWChem.

Real-time TDDFT computes the same linear response differently. After a weak impulsive field $\delta v_{\rm ext}(t)=-\kappa\,\boldsymbol{\epsilon}\cdot\mathbf r\,\delta(t)$, it propagates

<div class="math-display">
$$
i\frac{\partial\varphi_n(t)}{\partial t}
=\left[-\frac{\nabla^2}{2}+v_{\rm ext}(t)
+v_H[n(t)]+v_{\rm xc}[n(t)]\right]\varphi_n(t),
$$
</div>

and Fourier transforms the induced dipole,

<div class="math-display">
$$
\alpha_{\epsilon\epsilon}(\omega)=
\frac{1}{\kappa}\int_0^T dt\,
e^{i\omega t-\eta t}\left[d_\epsilon(t)-d_\epsilon(0)\right].
$$
</div>

One kick yields a continuous spectrum for that polarization. The cost is a very small time step for high-frequency core motion and a long propagation for fine energy resolution.

TDDFT's weak point is now visible: the answer depends on $f_{\rm xc}$. Adiabatic semilocal kernels often underestimate core excitation energies, suffer self-interaction error, and poorly represent a long-range electron-core-hole attraction; they also miss genuine double excitations and many shake-up satellites. Hybrid or range-separated functionals, a relaxed core-hole reference, and scalar or fully relativistic Hamiltonians improve matters. In practice, TDDFT is strongest for molecular K-edge XANES and time-resolved finite-system problems. It is not the normal tool for long-range EXAFS.

## BSE: propagate an explicit electron-hole pair

BSE starts from single-particle orbitals, usually DFT states corrected toward quasiparticle energies by $GW$ or an approximate scissors/self-energy model. A core exciton $S$ is expanded in core-to-conduction transitions,

<div class="math-display">
$$
\left\lvert S\right\rangle=
\sum_{ca\mathbf k}X^S_{ca\mathbf k}
a^\dagger_{a\mathbf k}a_c
\left\lvert 0\right\rangle.
$$
</div>

$c$ labels a localized core spinor, $a\mathbf k$ an empty Bloch state, and $X^S_{ca\mathbf k}$ the correlated transition amplitude. In the Tamm-Dancoff approximation the amplitudes obey

<div class="math-display">
$$
\sum_{c'a'\mathbf k'}
H^{\rm BSE}_{ca\mathbf k,c'a'\mathbf k'}
X^S_{c'a'\mathbf k'}
=\Omega_S X^S_{ca\mathbf k},
$$
</div>

with

<div class="math-display">
$$
H^{\rm BSE}_{ca\mathbf k,c'a'\mathbf k'}=
(\varepsilon^{\rm QP}_{a\mathbf k}-\varepsilon_c)
\delta_{cc'}\delta_{aa'}\delta_{\mathbf k\mathbf k'}
+K^x_{ca\mathbf k,c'a'\mathbf k'}
-K^d_{ca\mathbf k,c'a'\mathbf k'}.
$$
</div>

$K^d$ is the matrix element of the statically screened Coulomb interaction $W(0)=\epsilon^{-1}(0)v_C$ and attracts the excited electron to the core hole. $K^x$ uses the bare Coulomb interaction and describes exchange and local-field effects. Their off-diagonal elements mix independent transitions. The spectrum is

<div class="math-display">
$$
\mu(E_\gamma)\propto E_\gamma
\sum_S
\left\lvert
\sum_{ca\mathbf k}X^S_{ca\mathbf k}
d_{ca\mathbf k}
\right\rvert^2
L_{\Gamma_S}(E_\gamma-\Omega_S),
\qquad
d_{ca\mathbf k}=\left\langle a\mathbf k\right\rvert d
\left\lvert c\right\rangle.
$$
</div>

The amplitudes add *before* squaring. This coherent sum is why an exciton can borrow intensity, split a peak, or make a transition dark; a projected density of states cannot do that. $L_{\Gamma_S}$ is a normalized Lorentzian or other broadening function containing the core-hole width, excited-electron damping, and experimental resolution.

**OCEAN** combines plane-wave DFT, PAW-reconstructed core transition matrix elements, a core BSE solver, spin-orbit coupling, and a many-pole $GW$ self-energy model. **exciting** solves core BSE in an all-electron LAPW framework. Their main controlled errors are convergence with empty bands, $k$ points, screening cutoff, and transition subspace. Their physical approximations include static screening, usually the Tamm-Dancoff approximation, a quasiparticle picture, and incomplete vibrational or multi-electron satellite physics. BSE is generally the most systematic of these methods for bound core excitons and near-edge oscillator strengths, but it is more expensive than FEFF and is not designed for long-range EXAFS fitting.

## A compact decision rule

- For a **Cr K-edge oxide**, begin with FEFF or FDMNES for geometry and broad XANES; use OCEAN/exciting BSE if the pre-edge or white-line intensity is central.
- For a **molecular C, N, or O K-edge**, REW-TDDFT in NWChem is economical; test the functional and absolute-energy shift against a standard.
- For a **strongly correlated transition-metal $L_{2,3}$ edge**, expect spin-orbit and multiplet physics; compare BSE with Quanty or another charge-transfer multiplet calculation.
- For **bond lengths, coordination shells, and thermal disorder**, use FEFF paths fitted to EXAFS with Larch or Artemis.
- For **Materials Project XAS**, remember that the underlying method is FEFF real-space multiple scattering, not TDDFT or BSE.

## Takeaway

XAS is local because the excitation begins in a compact core orbital. The pre-edge emphasizes symmetry and weak bound transitions; XANES emphasizes electronic structure, core-hole physics, and multiple scattering; EXAFS turns the photoelectron into an interferometer for neighbor distances and disorder. The regions overlap, but the change of language—from orbitals, to correlated near-edge states, to scattering paths—is what makes the whole spectrum readable.

## References and starting points

- [Materials Project: how its XAS spectra are calculated](https://docs.materialsproject.org/methodology/materials-methodology/x-ray-absorption-spectra)
- [Mathew et al., high-throughput computational XAS](https://doi.org/10.1038/sdata.2018.151)
- [FEFF documentation and theory overview](https://feff.phys.washington.edu/feff/wiki/static/f/e/f/FEFF_Synopsis_0bea.html)
- [Rehr et al., parameter-free X-ray spectra with FEFF9](https://doi.org/10.1039/B926434E)
- [exciting tutorial: XAS using the BSE](https://www.exciting-code.org/uploads/exciting/tutorial_notebooks/xray_absorption_spectra_using_bse.html)
- [FDMNES theory overview](https://fdmnes.neel.cnrs.fr/theory/)
- [OCEAN at NIST](https://www.nist.gov/services-resources/software/ocean)
- [Vinson et al., core-level BSE theory](https://doi.org/10.1103/PhysRevB.83.115106)
- [NWChem restricted-window TDDFT documentation](https://nwchemgit.github.io/Excited-State-Calculations.html)
- [Lopata et al., linear-response and real-time TDDFT for XAS](https://www.pnnl.gov/publications/linear-response-and-real-time-time-dependent-density-functional-theory-studies-core)
- [Quanty $L_{2,3}$-edge multiplet tutorial](https://www.quanty.org/documentation/tutorials/nio_crystal_field/xas_l23)
- [Quantum ESPRESSO user guide (XSpectra)](https://www.quantum-espresso.org/wp-content/uploads/2022/03/user_guide.pdf)
- [Larch documentation for XAFS analysis](https://xraypy.github.io/xraylarch/)
- [Lightshow multi-code input generator](https://doi.org/10.21105/joss.05182)
- [Chromium edge-energy reference](https://www.esrf.fr/UsersAndScience/Experiments/CRG/BM30B/Mendeleev/24-Cr.html)
