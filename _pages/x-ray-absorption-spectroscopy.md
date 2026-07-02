---
permalink: /blog/x-ray-absorption-spectroscopy/
title: "Reading X-ray Absorption Spectra: Edges, Regions, and First-Principles Models"
excerpt: "A compact guide to pre-edge, XANES, EXAFS, core-level labels, and the calculations behind Materials Project spectra."
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

where $t$ is the sample thickness. Microscopically, absorption promotes a localized core electron into an unoccupied state. In the electric-dipole approximation,

<div class="math-display">
$$
\mu(\omega)\propto
\sum_f \left|\langle f|\boldsymbol{\epsilon}\!\cdot\!\mathbf r|c\rangle\right|^2
\delta(E_f-E_c-\hbar\omega).
$$
</div>

This compact expression already contains most of XAS: the core state $|c\rangle$ makes the measurement element-specific, the final states $|f\rangle$ contain the local electronic and geometric environment, and the polarization $\boldsymbol{\epsilon}$ makes orbital orientation visible.

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

## How first-principles calculations see the same spectrum

The golden-rule sum over final states can be rewritten using a retarded Green's function,

<div class="math-display">
$$
\mu(E)\propto-\operatorname{Im}
\langle c|D^\dagger G(E)D|c\rangle,
\qquad D=\boldsymbol{\epsilon}\cdot\mathbf r.
$$
</div>

This is the natural language of **FEFF**. The code constructs an atomic cluster around a chosen absorber, obtains self-consistent scattering potentials, includes a screened core hole and inelastic losses, and evaluates the photoelectron Green's function. Full multiple scattering is important in XANES; a path expansion becomes intuitive and efficient in EXAFS.

This is also how the XAS data associated with the **Materials Project** were generated. The high-throughput workflow used crystal structures from MP, selected every symmetrically distinct absorbing site, built a local cluster, and ran FEFF9. The original release contained hundreds of thousands of site-resolved K-edge XANES spectra. So the MP curves are first-principles real-space multiple-scattering calculations—not TDDFT trajectories. They are excellent for comparing candidate structures and local environments, but finite-temperature disorder, defects, surface chemistry, and energy alignment can still separate a calculated ideal crystal from an experiment.

### Where BSE and TDDFT enter

Very close to the edge, treating the electron and core hole as independent particles can fail. The Bethe-Salpeter equation (BSE) instead diagonalizes an effective electron-hole Hamiltonian,

<div class="math-display">
$$
H^{\mathrm{BSE}}=H^{\mathrm{diag}}+K^x-K^d,
\qquad
\epsilon_2(\omega)\propto\sum_\lambda
\left|\sum_{ca\mathbf k}X^\lambda_{ca\mathbf k}d_{ca\mathbf k}\right|^2
\delta(\omega-\Omega_\lambda).
$$
</div>

The attractive direct term $-K^d$ binds and redshifts core excitons; off-diagonal terms mix transitions and redistribute oscillator strength. **OCEAN** and **exciting** are prominent BSE routes for near-edge core spectra.

Time-dependent DFT (TDDFT) is another linear-response route: one perturbs the electron density and computes its frequency-dependent response. It can work well, especially for molecules and selected edges, but core spectroscopy places severe demands on the exchange-correlation kernel, relativistic core levels, and core-hole treatment. **FDMNES** includes TDDFT for edges where its usual DFT treatment is insufficient. TDDFT and BSE are therefore alternatives for near-edge electron-hole physics; neither is the method behind the Materials Project FEFF database.

## A practical package map

| Goal | Useful tools | Main idea |
|---|---|---|
| Fast XANES/EXAFS simulation and path analysis | **FEFF** | Real-space Green's functions and multiple scattering |
| Full-potential near-edge simulation, dichroism | **FDMNES** | Finite differences or multiple scattering; DFT/TDDFT options |
| Plane-wave/pseudopotential XAS | **XSpectra** in Quantum ESPRESSO, **VASP** | Core-hole supercells and reconstructed transition matrix elements |
| Explicit core excitons | **OCEAN**, **exciting** | DFT plus BSE electron-hole Hamiltonian |
| Multi-code input generation | **Lightshow** | Builds consistent inputs for FEFF, VASP, OCEAN, exciting, and XSpectra |
| Experimental reduction and EXAFS fitting | **Larch/Larix**, **Athena/Artemis** | Normalize $\mu(E)$, subtract background, Fourier transform, and fit FEFF paths |

A sensible workflow is to identify the absorber and edge, align and normalize the measured spectrum, use standards or calculated XANES to test oxidation and geometry, then fit EXAFS paths for distances and disorder. If the first few eV contain unexplained sharp peaks, that is the moment to suspect core excitons and move from an independent-particle or multiple-scattering picture toward BSE or TDDFT.

## Takeaway

XAS is local because the excitation begins in a compact core orbital. The pre-edge emphasizes symmetry and weak bound transitions; XANES emphasizes electronic structure, core-hole physics, and multiple scattering; EXAFS turns the photoelectron into an interferometer for neighbor distances and disorder. The regions overlap, but the change of language—from orbitals, to correlated near-edge states, to scattering paths—is what makes the whole spectrum readable.

## References and starting points

- [Materials Project: how its XAS spectra are calculated](https://docs.materialsproject.org/methodology/materials-methodology/x-ray-absorption-spectra)
- [Mathew et al., high-throughput computational XAS](https://doi.org/10.1038/sdata.2018.151)
- [FEFF documentation and theory overview](https://feff.phys.washington.edu/feff/wiki/static/f/e/f/FEFF_Synopsis_0bea.html)
- [exciting tutorial: XAS using the BSE](https://www.exciting-code.org/uploads/exciting/tutorial_notebooks/xray_absorption_spectra_using_bse.html)
- [FDMNES theory overview](https://fdmnes.neel.cnrs.fr/theory/)
- [OCEAN at NIST](https://www.nist.gov/services-resources/software/ocean)
- [Quantum ESPRESSO user guide (XSpectra)](https://www.quantum-espresso.org/wp-content/uploads/2022/03/user_guide.pdf)
- [Larch documentation for XAFS analysis](https://xraypy.github.io/xraylarch/)
- [Lightshow multi-code input generator](https://doi.org/10.21105/joss.05182)
- [Chromium edge-energy reference](https://www.esrf.fr/UsersAndScience/Experiments/CRG/BM30B/Mendeleev/24-Cr.html)
