---
permalink: /blog/optical-properties-kubo-dft-gw/
title: "Simulating Optical Properties: From Kubo Formula to GW-BSE"
excerpt: "A self-contained derivation of how optical spectra are computed in periodic solids, starting from the Kubo linear-response formula, through DFT-level independent-particle and RPA approximations, up to many-body GW and BSE corrections, with practical VASP, BerkeleyGW, and GPAW recipes."
author_profile: true
---

# Simulating Optical Properties: From Kubo to GW-BSE

Optical spectroscopy probes the electronic structure of materials across a broad energy range.
This note builds the theoretical framework from scratch: starting from the Kubo linear-response
formula, then specialising it to the independent-particle (IP) and random-phase approximations
(RPA) as implemented in VASP and GPAW, and finally adding many-body corrections via the GW
approximation and Bethe-Salpeter equation (BSE) as implemented in BerkeleyGW.

**Notation used throughout.**

| Symbol | Meaning |
|--------|---------|
| $\hbar$ | reduced Planck constant (kept explicit) |
| $n,m$ | band indices; $v$ = valence, $c$ = conduction |
| $\mathbf{k}$ | Bloch wavevector |
| $\mathbf{q}$ | wavevector transfer |
| $\mathbf{G},\mathbf{G}'$ | reciprocal lattice vectors (bold) |
| $G(\mathbf{r},\mathbf{r}',\omega)$ | single-particle Green's function (roman) |
| $\epsilon_{n\mathbf{k}}$ | Kohn-Sham eigenvalue |
| $\psi_{n\mathbf{k}}(\mathbf{r})$ | Kohn-Sham orbital |
| $f_{n\mathbf{k}}$ | Fermi-Dirac occupation |
| $\Omega$ | crystal volume; $N_k$ number of $\mathbf{k}$-points |
| $\eta\to 0^+$ | positive infinitesimal broadening |
| $v(\mathbf{r})$ | bare Coulomb potential $e^2/\|\mathbf{r}\|$ (Gaussian units) |
| $W(\mathbf{r},\mathbf{r}',\omega)$ | screened Coulomb interaction |
| $\Sigma(\mathbf{r},\mathbf{r}',\omega)$ | electron self-energy |
| $V_{xc}(\mathbf{r})$ | DFT exchange-correlation potential |

---

## 1. From Maxwell to the Dielectric Function

The macroscopic response of a material to an electromagnetic perturbation is encoded in the
complex dielectric function $\varepsilon(\omega) = \varepsilon_1(\omega)+i\varepsilon_2(\omega)$.
The optical constants—complex refractive index $\tilde{n} = n + i\kappa$—follow from
$\tilde{n}^2 = \varepsilon$:

<div class="math-display">
$$
n(\omega) = \sqrt{\tfrac{1}{2}\!\left(\sqrt{\varepsilon_1^2+\varepsilon_2^2}+\varepsilon_1\right)},
\qquad
\kappa(\omega) = \sqrt{\tfrac{1}{2}\!\left(\sqrt{\varepsilon_1^2+\varepsilon_2^2}-\varepsilon_1\right)}.
$$
</div>

The experimentally accessible absorption coefficient and normal-incidence reflectance are then:

<div class="math-display">
$$
\alpha(\omega) = \frac{2\omega\kappa(\omega)}{c},
\qquad
R(\omega) = \frac{(n-1)^2+\kappa^2}{(n+1)^2+\kappa^2}.
$$
</div>

Causality enforces the **Kramers-Kronig** (KK) relations between $\varepsilon_1$ and $\varepsilon_2$:

<div class="math-display">
$$
\varepsilon_1(\omega) = 1 + \frac{2}{\pi}\,\mathcal{P}\!\int_0^\infty
\frac{\omega'\varepsilon_2(\omega')}{\omega'^2-\omega^2}\,d\omega'.
$$
</div>

Because of KK, computing the absorptive part $\varepsilon_2(\omega)$ from first principles is
sufficient to fully determine $\varepsilon(\omega)$.

---

## 2. Kubo Linear-Response Theory

### 2.1 Optical Conductivity and the Kubo Formula

Apply a weak external electric field
$\mathbf{E}(\mathbf{r},t) = \mathbf{E}_0\,e^{i(\mathbf{q}\cdot\mathbf{r}-\omega t)}+\text{c.c.}$
to a many-electron system. Linear-response theory (Kubo 1957) states that the induced current
density is:

<div class="math-display">
$$
J_\alpha(\omega) = \sigma_{\alpha\beta}(\omega)\,E_\beta(\omega),
$$
</div>

where the optical conductivity tensor is given by the retarded current-current correlation
function:

<div class="math-display">
$$
\sigma_{\alpha\beta}(\omega)
= \frac{1}{i(\omega+i\eta)\,\Omega}
  \Bigl[\Pi^R_{\alpha\beta}(\omega) - \Pi^R_{\alpha\beta}(0)\Bigr]
+ \frac{ne^2\delta_{\alpha\beta}}{im_e(\omega+i\eta)},
$$
</div>

<div class="math-display">
$$
\Pi^R_{\alpha\beta}(\omega)
= -i\int_0^\infty dt\,e^{i(\omega+i\eta)t}
  \langle[\hat{J}_\alpha(t),\hat{J}_\beta(0)]\rangle_0.
$$
</div>

The second (diamagnetic) term in $\sigma_{\alpha\beta}$ cancels exactly against a corresponding
part of $\Pi^R$ at $\mathbf{q}=0$. The dielectric function is related to the conductivity by:

<div class="math-display">
$$
\varepsilon(\omega) = 1 + \frac{i\sigma(\omega)}{\varepsilon_0\omega}
= 1 - \frac{4\pi\Pi^R(\omega)}{\omega^2\,\Omega}
\qquad\text{(Gaussian units)}.
$$
</div>

### 2.2 Density-Density Response and the Dielectric Matrix

At finite momentum transfer $\mathbf{q}$, it is more natural to work with the density-density
response function (polarizability) $\chi(\mathbf{r},\mathbf{r}',\omega)$.  In a periodic solid
one expands in the plane-wave basis $e^{i(\mathbf{q}+\mathbf{G})\cdot\mathbf{r}}$, giving a
matrix in $(\mathbf{G},\mathbf{G}')$:

<div class="math-display">
$$
\varepsilon_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)
= \delta_{\mathbf{G}\mathbf{G}'} - v_\mathbf{G}(\mathbf{q})\,\chi^0_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega),
\qquad
v_\mathbf{G}(\mathbf{q}) = \frac{4\pi e^2}{|\mathbf{q}+\mathbf{G}|^2}.
$$
</div>

**Why $\varepsilon$ is directly proportional to $\chi^0$ in DFT:**  
Apply Poisson to the density perturbation:

<div class="math-display">
$$
\delta n(\mathbf{r}) = \chi^0(\mathbf{r},\mathbf{r}',\omega)[\phi_{\text{ext}}(\mathbf{r}')+\phi_{\text{ind}}(\mathbf{r}')],
\qquad
\phi_{\text{ind}} = v * \delta n,
$$
</div>

where $*$ denotes convolution.  Combining these:

<div class="math-display">
$$
\delta n = \chi^0(\phi_{\text{ext}} + v*\chi^0\phi_{\text{ext}})
= (1 - v*\chi^0)^{-1}\chi^0\phi_{\text{ext}}.
$$
</div>

The dielectric response then follows from $\delta n = \chi \phi_{\text{ext}}$, giving

<div class="math-display">
$$
\varepsilon = 1 - v*\chi^0.
$$
</div>

In the plane-wave basis this becomes $\varepsilon_{\mathbf{GG}'}(\mathbf{q},\omega) = \delta_{\mathbf{GG}'} - v_\mathbf{G}(\mathbf{q})\chi^0_{\mathbf{GG}'}(\mathbf{q},\omega)$.

The macroscopic dielectric function governing optical measurements is obtained by inverting
the full matrix and taking the $\mathbf{G}=\mathbf{G}'=\mathbf{0}$ element:

<div class="math-display">
$$
\varepsilon_M(\omega) = \frac{1}{\bigl[\varepsilon^{-1}\bigr]_{\mathbf{00}}(\mathbf{q}\to\mathbf{0},\omega)}.
$$
</div>

This inversion step is crucial: neglecting the off-diagonal elements $(\mathbf{G}\neq\mathbf{G}')$
gives the **no-local-fields** result; retaining them introduces **local-field effects (LFE)**,
which account for the spatial inhomogeneity of the crystal.

---

## 3. Independent-Particle and RPA at the DFT Level

### 3.1 Kohn-Sham Starting Point

DFT maps the interacting problem onto non-interacting electrons satisfying:

<div class="math-display">
$$
\left[-\frac{\hbar^2\nabla^2}{2m_e} + V_{KS}(\mathbf{r})\right]\psi_{n\mathbf{k}}(\mathbf{r})
= \epsilon_{n\mathbf{k}}\,\psi_{n\mathbf{k}}(\mathbf{r}),
$$
</div>

where $V_{KS} = V_{ext} + V_H + V_{xc}$ includes the external, Hartree, and
exchange-correlation potentials. The corresponding retarded single-particle Green's function is:

<div class="math-display">
$$
G^0(\mathbf{r},\mathbf{r}',\omega)
= \sum_{n\mathbf{k}}
  \frac{\psi_{n\mathbf{k}}(\mathbf{r})\,\psi_{n\mathbf{k}}^*(\mathbf{r}')}
       {\hbar\omega - \epsilon_{n\mathbf{k}} + i\eta\,\mathrm{sgn}(\epsilon_{n\mathbf{k}}-\mu)}.
$$
</div>

### 3.2 Independent-Particle Polarizability (Adler-Wiser Formula)

The bubble diagram is built from two $G^0$ propagators connected by density vertices. The retarded density-density correlator:

<div class="math-display">
$$
\chi^0(\mathbf{r},\mathbf{r}',\omega) = -\frac{i}{\hbar}\int_{-\infty}^\infty \frac{dt}{2\pi}\,e^{i\omega t}
\langle[\hat{n}(\mathbf{r},t), \hat{n}(\mathbf{r}',0)]\rangle_>,
\qquad
\hat{n}(\mathbf{r}) = \psi^{\dagger}(\mathbf{r})\psi(\mathbf{r}).
$$
</div>

Wick's theorem decomposes the four-operator commutator into products of two single-particle propagators. Fourier-expanding in plane waves $e^{\pm i(\mathbf{q}+\mathbf{G})\cdot\mathbf{r}}$:

<div class="math-display">
$$
\chi^0_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)
= \frac{1}{\hbar i}\int d\omega' \int \frac{d\mathbf{r}\,d\mathbf{r}'}{(2\pi)^6}
  e^{-i(\mathbf{q}+\mathbf{G})\cdot\mathbf{r}}
  G^0_{>}(\mathbf{r},\mathbf{r}',\omega+\omega')
  G^0_{<}(\mathbf{r}',\mathbf{r},\omega')
  e^{i(\mathbf{q}+\mathbf{G}')\cdot\mathbf{r}'}.
$$
</div>

Spectral representation—inserting complete sets of KS eigenstates and evaluating the $\omega'$ integral—yields the **Adler-Wiser** formula:

<div class="math-display">
$$
\chi^0_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)
= \frac{2}{\Omega N_k}\sum_{n\mathbf{k}}\sum_{m,\mathbf{k}+\mathbf{q}}
  \frac{f_{n\mathbf{k}} - f_{m,\mathbf{k}+\mathbf{q}}}
       {\hbar\omega - \bigl(\epsilon_{m,\mathbf{k}+\mathbf{q}}-\epsilon_{n\mathbf{k}}\bigr) + i\hbar\eta}
  \,\rho_{nm\mathbf{k}}(\mathbf{q}+\mathbf{G})\,
  \bigl[\rho_{nm\mathbf{k}}(\mathbf{q}+\mathbf{G}')\bigr]^*,
$$
</div>

where the pair density matrix element is:

<div class="math-display">
$$
\rho_{nm\mathbf{k}}(\mathbf{q}+\mathbf{G})
= \langle n\mathbf{k}\,|\,e^{-i(\mathbf{q}+\mathbf{G})\cdot\hat{\mathbf{r}}}\,|\,m,\mathbf{k}+\mathbf{q}\rangle.
$$
</div>

The factor of 2 accounts for spin degeneracy.

In the **optical limit** $\mathbf{q}\to\mathbf{0}$, the $\mathbf{G}=\mathbf{0}$ matrix element for
$n\neq m$ reduces via the commutator $[\hat{H}_{KS},\hat{\mathbf{r}}] = i\hbar\hat{\mathbf{p}}/m_e$ to:

<div class="math-display">
$$
\rho_{nm\mathbf{k}}(\mathbf{q}\to\mathbf{0})\big|_{n\neq m}
\;\longrightarrow\;
\frac{i\hbar\,\mathbf{q}\cdot\langle n\mathbf{k}|\hat{\mathbf{p}}|m\mathbf{k}\rangle}
     {m_e\,(\epsilon_{m\mathbf{k}}-\epsilon_{n\mathbf{k}})},
$$
</div>

leading to the absorptive part of the macroscopic dielectric function:

<div class="math-display">
$$
\varepsilon_2(\omega)
= \frac{4\pi^2 e^2}{m_e^2\omega^2}
  \frac{2}{\Omega N_k}
  \sum_{n,m,\mathbf{k}} f_{n\mathbf{k}}(1-f_{m\mathbf{k}})
  \bigl|\hat{\mathbf{e}}\cdot\langle n\mathbf{k}|\hat{\mathbf{p}}|m\mathbf{k}\rangle\bigr|^2
  \,\delta\!\bigl(\epsilon_{m\mathbf{k}}-\epsilon_{n\mathbf{k}}-\hbar\omega\bigr).
$$
</div>

Here $\hat{\mathbf{e}}$ is the light polarisation direction and $\hat{\mathbf{p}} = -i\hbar\nabla$
is the momentum operator. This expression has a transparent physical meaning: it is the
**joint density of states (JDOS)** weighted by the squared optical transition matrix elements
(oscillator strengths).  The real part $\varepsilon_1(\omega)$ then follows from the KK relation.

### 3.3 Random Phase Approximation and Local-Field Effects

The IP result above uses $\chi^0$ without allowing the induced density to feed back. Including
this feedback (the Dyson equation for the polarizability) defines the **random-phase approximation**:

<div class="math-display">
$$
\chi_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)
= \chi^0_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)
  + \sum_{\mathbf{G}''}\chi^0_{\mathbf{G}\mathbf{G}''}(\mathbf{q},\omega)\,
    v_{\mathbf{G}''}(\mathbf{q})\,
    \chi_{\mathbf{G}''\mathbf{G}'}(\mathbf{q},\omega),
$$
</div>

or in compact matrix notation $\chi = (1 - \chi^0 v)^{-1}\chi^0$.  The macroscopic dielectric
function with full local-field effects then requires inverting the full $\mathbf{G}\mathbf{G}'$
matrix:

<div class="math-display">
$$
\varepsilon_M(\omega)
= \Bigl[\bigl(1 - \chi^0(\mathbf{q}\to\mathbf{0},\omega)\,v\bigr)^{-1}\Bigr]_{\mathbf{00}}^{-1}.
$$
</div>

Local-field effects are most important in inhomogeneous systems (surfaces, nanostructures,
low-dimensional materials) and in strongly polarisable insulators.

### 3.4 DFT Implementations

**VASP** computes optical matrix elements from the PAW orbitals.

```
LOPTICS = .TRUE.    ! compute optical transition matrix elements
NBANDS  = 200       ! must include sufficient empty bands
NEDOS   = 2000      ! frequency grid density
CSHIFT  = 0.05      ! Lorentzian broadening η (eV)
```

By default VASP uses the IP approximation (no local fields).  The full RPA dielectric matrix
requires `ALGO = CHI`.  Results are stored in `vasprun.xml` under the `dielectricfunction` tag;
`vaspkit` (task 71x) or `sumo` can post-process them.

**GPAW** provides the `DielectricFunction` class in `gpaw.response`:

```python
from gpaw.response.df import DielectricFunction
df = DielectricFunction(
    'gs.gpw',
    frequencies={'type': 'nonlinear', 'domega0': 0.005},  # eV
    eta=0.05,    # broadening η (eV)
    ecut=150,    # G-vector cutoff for χ⁰ (eV)
    hilbert=True # use Hilbert transform for ω integration
)
df.get_dielectric_function(filename='df.csv')
```

The `ecut` parameter controls the $\mathbf{G}$-vector plane-wave cutoff for the response matrix
(separate from and typically smaller than the ground-state wavefunction cutoff).  Setting
`local_field_contributions=True` includes LFE.

---

## 4. Self-Energy Corrections: The GW Approximation

### 4.1 Why DFT Band Gaps Are Underestimated

KS eigenvalue differences are not true quasiparticle (QP) excitation energies.  The KS
exchange-correlation potential $V_{xc}$ is a multiplicative (semi-)local operator, whereas the
true self-energy $\Sigma(\mathbf{r},\mathbf{r}',\omega)$ is non-local and frequency-dependent.
In LDA/GGA, band gaps in semiconductors and insulators are typically underestimated by 30–50%.
Because $\varepsilon_2(\omega)$ is dominated by valence-to-conduction transitions, a wrong gap
shifts the entire optical spectrum.

### 4.2 Hedin's Equations

Hedin (1965) derived an exact, closed set of equations for the interacting Green's function
$G(\mathbf{r},\mathbf{r}',\omega)$, which is the propagator for adding or removing an electron
from the $N$-particle ground state.  The Dyson equation reads:

<div class="math-display">
$$
G = G^0 + G^0\,\Sigma\,G,
$$
</div>

where the self-energy $\Sigma$ captures all exchange-correlation physics beyond the Hartree level.
Hedin's self-consistent set is:

<div class="math-display">
$$
\Sigma(\mathbf{r},\mathbf{r}',\omega)
= \frac{i}{2\pi}\int d\omega'\,
  G(\mathbf{r},\mathbf{r}',\omega+\omega')\,W(\mathbf{r},\mathbf{r}',\omega'),
$$
</div>

<div class="math-display">
$$
W(\mathbf{r},\mathbf{r}',\omega)
= \int d\mathbf{r}''\,\varepsilon^{-1}(\mathbf{r},\mathbf{r}'',\omega)\,v(\mathbf{r}''-\mathbf{r}'),
$$
</div>

<div class="math-display">
$$
P(\mathbf{r},\mathbf{r}',\omega)
= -\frac{i}{2\pi}\int d\omega'\,
  G(\mathbf{r},\mathbf{r}',\omega')\,G(\mathbf{r}',\mathbf{r},\omega'-\omega),
$$
</div>

where $P$ is the irreducible polarisation operator, $W = \varepsilon^{-1}v$ is the
**screened Coulomb interaction**, and $v(\mathbf{r}-\mathbf{r}') = e^2/|\mathbf{r}-\mathbf{r}'|$
is the bare Coulomb potential.  In RPA one sets $\varepsilon = 1 - vP$ and $P = -iGG$ (no
vertex corrections), making Hedin's equations tractable.

### 4.3 The G₀W₀ Approximation

Full self-consistent GW is expensive.  The most widely used approximation is **G₀W₀** (one-shot
GW): evaluate the self-energy once using the KS Green's function $G^0$ and the RPA screened
interaction $W^0 = \varepsilon^{-1}_\text{RPA}\,v$:

<div class="math-display">
$$
\Sigma^{G_0W_0}(\mathbf{r},\mathbf{r}',\omega)
= \frac{i}{2\pi}\int d\omega'\,
  G^0(\mathbf{r},\mathbf{r}',\omega+\omega')\,W^0(\mathbf{r},\mathbf{r}',\omega').
$$
</div>

QP energies are obtained by first-order perturbation theory on top of KS eigenvalues:

<div class="math-display">
$$
\epsilon^{QP}_{n\mathbf{k}}
= \epsilon_{n\mathbf{k}}
  + \langle n\mathbf{k}|\,
    \Sigma^{G_0W_0}(\epsilon^{QP}_{n\mathbf{k}}) - V_{xc}\,
    |n\mathbf{k}\rangle.
$$
</div>

Since $\Sigma$ depends on $\epsilon^{QP}_{n\mathbf{k}}$ itself, this equation is usually solved by
**linearisation**:

<div class="math-display">
$$
\epsilon^{QP}_{n\mathbf{k}}
\approx \epsilon_{n\mathbf{k}}
  + Z_{n\mathbf{k}}\,
    \langle n\mathbf{k}|\,\Sigma^{G_0W_0}(\epsilon_{n\mathbf{k}}) - V_{xc}\,|n\mathbf{k}\rangle,
$$
</div>

where the **quasiparticle renormalisation factor** is:

<div class="math-display">
$$
Z_{n\mathbf{k}}
= \left[1 - \left.
  \frac{\partial\,\mathrm{Re}\,\Sigma_{n\mathbf{k}}(\omega)}{\partial\omega}
  \right|_{\omega=\epsilon_{n\mathbf{k}}}\right]^{-1},
\qquad 0 < Z_{n\mathbf{k}} \leq 1.
$$
</div>

$Z_{n\mathbf{k}} < 1$ reflects the transfer of spectral weight from the quasiparticle peak to
incoherent satellites (plasmons, etc.).

### 4.4 Screened Interaction in the Plane-Wave Basis

In a periodic solid the screened Coulomb matrix is:

<div class="math-display">
$$
W_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)
= \varepsilon^{-1}_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)\,v_{\mathbf{G}'}(\mathbf{q}),
\qquad
v_\mathbf{G}(\mathbf{q}) = \frac{4\pi e^2}{|\mathbf{q}+\mathbf{G}|^2}.
$$
</div>

The self-energy matrix element required for QP corrections is:

<div class="math-display">
$$
\Sigma_{n\mathbf{k}}(\omega)
= \frac{i}{2\pi\Omega N_k}
  \sum_{m,\mathbf{q}}\sum_{\mathbf{G}\mathbf{G}'}
  \int d\omega'\,
  \frac{M^n_{m,\mathbf{k}-\mathbf{q}}(\mathbf{G})\,
       \bigl[M^n_{m,\mathbf{k}-\mathbf{q}}(\mathbf{G}')\bigr]^*}
       {\hbar\omega+\hbar\omega' - \epsilon_{m,\mathbf{k}-\mathbf{q}} + i\eta_m}\,
  W_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega'),
$$
</div>

where $M^n_{m,\mathbf{k}-\mathbf{q}}(\mathbf{G})
= \langle n\mathbf{k}|e^{i(\mathbf{q}+\mathbf{G})\cdot\hat{\mathbf{r}}}|m,\mathbf{k}-\mathbf{q}\rangle$
are the plane-wave matrix elements.

The self-energy splits naturally into **exchange** and **correlation** parts,
$\Sigma = \Sigma_X + \Sigma_C$:

<div class="math-display">
$$
\Sigma_X(\mathbf{r},\mathbf{r}')
= -\sum_{n\mathbf{k}} f_{n\mathbf{k}}\,
  \psi_{n\mathbf{k}}(\mathbf{r})\,v(\mathbf{r}-\mathbf{r}')\,\psi_{n\mathbf{k}}^*(\mathbf{r}')
\qquad\text{(static, exact HF exchange)},
$$
</div>

<div class="math-display">
$$
\Sigma_C = \Sigma - \Sigma_X
\qquad\text{(frequency-dependent correlation, involves } W-v\text{)}.
$$
</div>

### 4.5 Practical GW: VASP and BerkeleyGW

**VASP G₀W₀:**

```
ALGO    = GW0        ! one-shot GW (ALGO=EVGW0 for eigenvalue self-consistent)
NOMEGA  = 50         ! frequency integration grid points
ENCUTGW = 200        ! plane-wave cutoff for W (eV), typically 2/3 of ENCUT
NBANDS  = 300        ! many empty bands required; test convergence
```

VASP uses either contour deformation (accurate, slow) or the plasmon-pole model (fast) for the
$\omega'$ integration in $\Sigma_C$.  QP corrections appear in `OUTCAR` and `EIGENVAL`.

**BerkeleyGW** (open-source, plane-wave pseudopotential) performs GW in three steps, taking
Kohn-Sham wavefunctions from VASP, Quantum ESPRESSO, or GPAW (via `WFN` files):

**Step 1 — Epsilon** (`epsilon.x`): build $\chi^0_{\mathbf{G}\mathbf{G}'}(\mathbf{q},\omega)$
via Adler-Wiser and invert to obtain $\varepsilon^{-1}_{\mathbf{G}\mathbf{G}'}$.

```
# epsilon.inp
epsilon_cutoff  10.0    # Ry; controls size (and cost) of dielectric matrix
number_bands    300
frequency_dependence 2  # 0=static, 1=Godby-Needs GPP, 2=full-frequency
```

**Step 2 — Sigma** (`sigma.x`): evaluate $\Sigma_{n\mathbf{k}}$ and write QP energies.

```
# sigma.inp
screened_coulomb_cutoff  10.0    # Ry; matches epsilon_cutoff
bare_coulomb_cutoff      40.0    # Ry; same as wavefunction cutoff
number_bands             300
```

**Convergence checklist for GW:**

| Parameter | Effect | How to converge |
|-----------|--------|-----------------|
| `number_bands` / `NBANDS` | completeness of $\chi^0$ and $\Sigma$ | increase until $\epsilon^{QP}$ stable |
| `epsilon_cutoff` / `ENCUTGW` | size of $\varepsilon_{\mathbf{GG}'}$ matrix | converge dielectric function |
| $\mathbf{k}$-mesh | BZ sampling of $\chi^0$ and $\Sigma$ | double mesh, check gap |
| Frequency grid | $\omega'$ integral in $\Sigma_C$ | increase `NOMEGA` or switch to full-frequency |

---

## 5. Optical Spectra Beyond DFT: The Bethe-Salpeter Equation

### 5.1 Why Quasiparticle Corrections Are Not Enough

GW corrects the single-particle spectrum but does not capture **excitons**—bound electron-hole
pairs formed when an electron is optically excited across the gap.  In most semiconductors
excitonic binding energies are 0.01–0.1 eV; in wide-gap oxides and 2D materials they can exceed
1 eV.  Excitons shift spectral weight to below the QP gap, create sharp resonances, and
dramatically redistribute oscillator strength.

### 5.2 The BSE Eigenvalue Problem

The Bethe-Salpeter equation for the two-particle electron-hole amplitude
$A^\lambda_{vc\mathbf{k}}$ of excitation $\lambda$ reads:

<div class="math-display">
$$
\sum_{v'c'\mathbf{k}'}
H^{BSE}_{vc\mathbf{k},\,v'c'\mathbf{k}'}\,A^\lambda_{v'c'\mathbf{k}'}
= E^\lambda\,A^\lambda_{vc\mathbf{k}}.
$$
</div>

In the **Tamm-Dancoff approximation** (TDA, resonant transitions only) the BSE Hamiltonian is:

<div class="math-display">
$$
H^{BSE}_{vc\mathbf{k},\,v'c'\mathbf{k}'}
= \underbrace{
    \bigl(\epsilon^{QP}_{c\mathbf{k}} - \epsilon^{QP}_{v\mathbf{k}}\bigr)
    \delta_{vv'}\delta_{cc'}\delta_{\mathbf{k}\mathbf{k}'}
  }_{\text{QP transition energies}}
+ \underbrace{
    2\,\bar{K}^x_{vc\mathbf{k},\,v'c'\mathbf{k}'}
  }_{\substack{\text{exchange} \\ \text{(repulsive)}}}
- \underbrace{
    K^d_{vc\mathbf{k},\,v'c'\mathbf{k}'}
  }_{\substack{\text{direct/screened} \\ \text{(attractive)}}}.
$$
</div>

The two interaction kernels are defined in terms of QP orbitals $\psi_{n\mathbf{k}}$:

<div class="math-display">
$$
\bar{K}^x_{vc\mathbf{k},\,v'c'\mathbf{k}'}
= \int d\mathbf{r}\,d\mathbf{r}'\,
  \psi^*_{c\mathbf{k}}(\mathbf{r})\,\psi_{v\mathbf{k}}(\mathbf{r})\,
  \bar{v}(\mathbf{r},\mathbf{r}')\,
  \psi^*_{v'\mathbf{k}'}(\mathbf{r}')\,\psi_{c'\mathbf{k}'}(\mathbf{r}'),
$$
</div>

<div class="math-display">
$$
K^d_{vc\mathbf{k},\,v'c'\mathbf{k}'}
= \int d\mathbf{r}\,d\mathbf{r}'\,
  \psi^*_{c\mathbf{k}}(\mathbf{r})\,\psi_{c'\mathbf{k}'}(\mathbf{r})\,
  W(\mathbf{r},\mathbf{r}',\omega{=}0)\,
  \psi_{v\mathbf{k}}(\mathbf{r}')\,\psi^*_{v'\mathbf{k}'}(\mathbf{r}'),
$$
</div>

where $\bar{v}$ is the bare Coulomb potential with the $\mathbf{G}=\mathbf{0}$ (long-range) term
removed to avoid double-counting the macroscopic Hartree shift, and $W(\omega=0)$ is the
statically screened Coulomb interaction (the frequency dependence of $W$ in the kernel is a
commonly adopted approximation).

The physical picture: the **direct term** $K^d$ provides the attractive electron-hole interaction
that binds excitons; the **exchange term** $\bar{K}^x$ is repulsive and, for singlet excitons,
produces the longitudinal-transverse splitting of degenerate exciton levels.

### 5.3 Optical Spectrum from BSE

Once the excitation energies $\{E^\lambda\}$ and amplitudes $\{A^\lambda_{vc\mathbf{k}}\}$ are
known from diagonalising $H^{BSE}$, the macroscopic imaginary dielectric function is:

<div class="math-display">
$$
\varepsilon_2(\omega)
= \frac{16\pi^2 e^2}{m_e^2\omega^2\Omega}
  \sum_\lambda
  \left|
    \sum_{vc\mathbf{k}} A^\lambda_{vc\mathbf{k}}\,
    \hat{\mathbf{e}}\cdot\langle v\mathbf{k}|\hat{\mathbf{p}}|c\mathbf{k}\rangle
  \right|^2
  \delta(\hbar\omega - E^\lambda).
$$
</div>

Comparing with the DFT-IPA result in Section 3.2, the many-body amplitude
$A^\lambda_{vc\mathbf{k}}$ replaces the product of individual oscillator strengths.
Excitons appear as sharp peaks **below** the QP gap, and the oscillator strength of band-edge
transitions is strongly redistributed.

### 5.4 Practical BSE: BerkeleyGW and VASP

**BerkeleyGW** performs BSE in two additional steps after GW:

**Step 3 — Kernel** (`kernel.x`): compute $K^d$ and $\bar{K}^x$ on a coarse $\mathbf{k}$-grid.

```
# kernel.inp
number_val_bands  8
number_cond_bands 8
use_wfn_hdf5       # use HDF5 wavefunction files for efficiency
```

**Step 4 — Absorption** (`absorption.x`): interpolate kernel onto a fine $\mathbf{k}$-grid
(via `inteqp`), build and diagonalise $H^{BSE}$, and assemble $\varepsilon_2(\omega)$.

```
# absorption.inp
number_val_bands_coarse   8
number_cond_bands_coarse  8
number_val_bands_fine     8
number_cond_bands_fine    8
use_tamm_dancoff             # TDA (recommended for most materials)
broadening  0.10 eV
energy_resolution 0.01 eV
```

Interpolation from a $6\times6\times6$ coarse grid to a $24\times24\times24$ fine grid is
routine and dramatically reduces cost while recovering the smooth optical line shape.

**VASP BSE** (after GW):

```
ALGO   = BSE
NBANDS = 200
ANTIRES = 0      ! Tamm-Dancoff; ANTIRES=2 for full BSE including anti-resonant part
```

VASP constructs $H^{BSE}$ on top of its GW quasiparticle wavefunctions and diagonalises it
by default using a direct algorithm for small matrices or iterative methods for larger ones.

---

## 6. Full Computational Pipeline

<div class="algorithm-block">
  <div class="algorithm-title">Workflow: Optical spectrum at the GW-BSE level</div>
  <ol>
    <li>
      <strong>DFT ground state (self-consistent field).</strong>
      Converge charge density on a standard $\mathbf{k}$-mesh.
      Use a norm-conserving or PAW pseudopotential.
    </li>
    <li>
      <strong>Wavefunction generation (non-self-consistent).</strong>
      Fix the charge density; diagonalise on a denser $\mathbf{k}$-mesh with many empty bands
      (convergence of $\chi^0$ and $\Sigma$ requires $\sim$100–300 bands above the Fermi level).
    </li>
    <li>
      <strong>Dielectric matrix $\varepsilon^{-1}_{\mathbf{GG}'}(\mathbf{q},\omega)$.</strong>
      Compute $\chi^0_{\mathbf{GG}'}$ via Adler-Wiser (Eq. in §3.2) and invert.
      Convergence: <code>epsilon_cutoff</code>, number of bands, $\mathbf{q}$-sampling.
    </li>
    <li>
      <strong>GW self-energy $\Sigma_{n\mathbf{k}}$.</strong>
      Evaluate $\Sigma = iG^0W^0$; obtain QP energies
      $\epsilon^{QP}_{n\mathbf{k}} = \epsilon_{n\mathbf{k}} + Z_{n\mathbf{k}}(\Sigma_{n\mathbf{k}} - \langle V_{xc}\rangle_{n\mathbf{k}})$.
    </li>
    <li>
      <strong>BSE kernel $H^{BSE}$.</strong>
      Construct the direct $K^d$ and exchange $\bar{K}^x$ kernels on a coarse $\mathbf{k}$-grid
      using QP energies and orbitals.
    </li>
    <li>
      <strong>Interpolation and diagonalisation.</strong>
      Interpolate $H^{BSE}$ onto a fine $\mathbf{k}$-grid; diagonalise to obtain
      $\{E^\lambda, A^\lambda_{vc\mathbf{k}}\}$.
    </li>
    <li>
      <strong>Optical spectrum.</strong>
      Assemble $\varepsilon_2(\omega)$ from BSE excitations; recover $\varepsilon_1(\omega)$
      via Kramers-Kronig; compute $\alpha(\omega)$, $R(\omega)$, EELS, etc.
    </li>
  </ol>
</div>

---

## 7. Summary and Approximation Hierarchy

| Level | Polarisability $\chi$ | Self-energy | Electron-hole | Typical gap accuracy |
|-------|----------------------|-------------|---------------|----------------------|
| DFT-IPA | $\chi^0$ (KS, no LFE) | $V_{xc}$ | None | $-30\%$ to $-50\%$ |
| DFT-RPA | $(1-\chi^0 v)^{-1}\chi^0$ (with LFE) | $V_{xc}$ | None | $-30\%$ to $-50\%$ |
| G₀W₀ | $\chi^0$ (KS) | $\Sigma = iG^0W^0$ | None | $<5\%$ |
| G₀W₀+BSE | $\chi^0$ (KS) | $\Sigma = iG^0W^0$ | $K^d+\bar{K}^x$ | $<5\%$; excitonic peaks |

**Practical guidance:**

- **DFT-IPA** is the right first step for selection rules, oscillator-strength maps, and
  qualitative peak assignments.  It is computationally cheap but gaps and peak positions are
  systematically red-shifted.
- **DFT-RPA** adds local-field effects at no extra DFT cost; important for surfaces,
  nanostructures, and anisotropic insulators.
- **G₀W₀** corrects QP gaps and dispersions.  For materials where KS already gives a
  reasonable gap (e.g., metals), G₀W₀ is often unnecessary.
- **G₀W₀+BSE** is the state-of-the-art for quantitative optical spectra.  It is mandatory
  whenever excitonic effects are expected—which, in practice, includes most semiconductors and
  all insulators.

---

## References

1. Kubo, R. (1957). Statistical-mechanical theory of irreversible processes. *J. Phys. Soc. Jpn.* **12**, 570–586.
2. Adler, S. L. (1962). Quantum theory of the dielectric constant in real solids. *Phys. Rev.* **126**, 413; Wiser, N. (1963). *Phys. Rev.* **129**, 62.
3. Hedin, L. (1965). New method for calculating the one-particle Green's function with application to the electron-gas problem. *Phys. Rev.* **139**, A796.
4. Hybertsen, M. S. & Louie, S. G. (1986). Electron correlation in semiconductors and insulators: Band gaps and quasiparticle energies. *Phys. Rev. B* **34**, 5390.
5. Strinati, G. (1988). Application of the Green's functions method to the study of the optical properties of semiconductors. *Riv. Nuovo Cimento* **11**, 1–86.
6. Onida, G., Reining, L. & Rubio, A. (2002). Electronic excitations: density-functional versus many-body Green's-function approaches. *Rev. Mod. Phys.* **74**, 601.
7. VASP Wiki — Optical properties: [https://www.vasp.at/wiki/index.php/Optical_properties](https://www.vasp.at/wiki/index.php/Optical_properties)
8. VASP Wiki — GW calculations: [https://www.vasp.at/wiki/index.php/GW_calculations](https://www.vasp.at/wiki/index.php/GW_calculations)
9. BerkeleyGW Manual: [https://berkeleygw.org/documentation/](https://berkeleygw.org/documentation/)
10. GPAW — Dielectric response: [https://wiki.fysik.dtu.dk/gpaw/documentation/tddft/dielectric_response.html](https://wiki.fysik.dtu.dk/gpaw/documentation/tddft/dielectric_response.html)
