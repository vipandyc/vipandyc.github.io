---
permalink: /blog/quantum-defects-platforms/
title: "Quantum Defects as Sensors, Single-Photon Emitters, and Qubits"
excerpt: "A compact tutorial on why atom-scale defects in crystals can act as quantum sensors, single-photon emitters, and spin qubits, using the negatively charged nitrogen-vacancy center in diamond as a running example."
author_profile: true
---

# Quantum Defects as Sensors, Single-Photon Emitters, and Qubits

<style>
.page__content a {
  overflow-wrap: anywhere;
}
.page__content .math-display .mjx-chtml {
  display: block !important;
  max-width: 100% !important;
  overflow-x: auto;
  overflow-y: hidden;
}
@media (max-width: 600px) {
  .page__content .math-display {
    text-align: left;
  }
  .page__content .math-display .mjx-chtml {
    width: 100% !important;
    min-width: 0 !important;
  }
  .page__content .math-display .mjx-label {
    display: none !important;
  }
}
</style>

A quantum defect is an atom-scale imperfection in an otherwise periodic crystal whose electronic states are localized strongly enough to behave like a small artificial atom. The host provides mechanical support, thermal contact, optical access, and fabrication routes. The defect provides the quantum degrees of freedom: a spin, an optical transition, a local charge state, or a combination of these.

The negatively charged nitrogen-vacancy center in diamond, NV$^-$, is the canonical example. It consists of a substitutional nitrogen next to a missing carbon atom. In the useful charge state it has a spin-triplet ground state, an optical transition in the visible, and spin-dependent fluorescence. That combination is rare: we can initialize the spin optically, manipulate it with microwaves, read it out optically, and let it interact with magnetic fields, electric fields, strain, temperature, photons, and nearby nuclear spins.

This note is organized around three applications and one design question:

1. quantum sensing: how a defect turns fields into phase shifts or resonance shifts;
2. single-photon emission: how one localized optical transition emits one photon at a time;
3. quantum computing: how a defect spin becomes a qubit and couples to memories or photons;
4. computational discovery: how screening requirements tighten from ground-state stability to excited-state dynamics.

The same defect can support all three applications, but the requirements are not equally strict. A useful sensor can be an ensemble with broad optical emission. A single-photon source must be bright, photostable, and antibunched. A scalable quantum-computing or networking defect usually needs coherent spin control, high-fidelity initialization and readout, reproducible optical transitions, and an interface to either nearby spins or photons. The funnel becomes narrow quickly.

## 1. Quantum Sensing

The basic sensing idea is simple: use a well-controlled quantum transition as a clock, then let the environment shift that clock. A magnetic field, electric field, strain field, temperature change, or nearby spin changes the defect Hamiltonian. If we can prepare a superposition, wait, and read out the accumulated phase, the defect becomes a local probe.

For the NV$^-$ center, the ground state has total spin $S=1$. A minimal spin Hamiltonian is

<div class="math-display">
$$
\begin{aligned}
H_{\rm gs}
&=
D S_z^2+\gamma_e \mathbf B\cdot \mathbf S \\
&\quad
+H_{\rm strain}+H_{\rm hf}+\cdots .
\end{aligned}
$$
</div>

Here $D/2\pi\approx 2.87$ GHz is the zero-field splitting between $m_s=0$ and $m_s=\pm1$, $\gamma_e$ is the electron gyromagnetic ratio, $H_{\rm strain}$ describes local crystal distortion and electric-field-like perturbations, and $H_{\rm hf}$ describes hyperfine coupling to nuclei such as the host $^{14}$N or nearby $^{13}$C spins. Along the NV axis, the Zeeman term shifts the $m_s=\pm1$ levels in opposite directions:

<div class="math-display">
$$
f_{\pm}
\approx
D/2\pi
\pm
\gamma_e B_z/2\pi .
$$
</div>

Measuring the microwave resonance frequency is therefore a magnetometer. The physics is a solid-state version of spectroscopy:

<div class="math-display">
$$
B_z
\longrightarrow
\Delta E_\pm(B_z)
\longrightarrow
f_\pm(B_z)
\longrightarrow
\text{measured signal}.
$$
</div>

The remaining question is practical: how do we measure the spin state of one atomic-scale defect? The answer is that light gives different photon counts for different spin projections.

For this discussion, write the useful ground-state spin levels as

<div class="math-display">
$$
\lvert 0\rangle \equiv \lvert S=1,m_s=0\rangle,
\qquad
\lvert 1\rangle \equiv \lvert S=1,m_s=-1\rangle .
$$
</div>

Here $m_s=0$ is not a singlet. It is one component of the same $S=1$ triplet. In a collinear DFT calculation one often represents the triplet by the high-spin determinant with total moment near $2\mu_B$; the experimental spin Hamiltonian then describes the $m_s=0,\pm1$ sublevels inside that triplet.

The optical cycle can be summarized by two probabilities. After green-laser excitation, the defect usually falls back to the ground triplet and emits a red photon. But the probability of emitting that photon depends on $m_s$:

<div class="math-display">
$$
p_\gamma(m_s=0) > p_\gamma(m_s=\pm1).
$$
</div>

So the expected photon number during a short readout pulse satisfies

<div class="math-display">
$$
N_\gamma(m_s=0) > N_\gamma(m_s=\pm1).
$$
</div>

That inequality is the whole reason optical readout works. A bright signal means the spin was more likely in $m_s=0$; a darker signal means it was more likely in $m_s=\pm1$.

Why are the $m_s=\pm1$ states darker? Because they have a larger chance to leave the bright optical cycle through intermediate singlet states. In a rate-picture,

<div class="math-display">
$$
k_{\rm singlet}(m_s=\pm1)
>
k_{\rm singlet}(m_s=0).
$$
</div>

The singlet path emits fewer useful red photons. It also tends to return population back to $m_s=0$. Therefore repeated laser cycles do two things at once:

<div class="math-display">
$$
\rho
\xrightarrow{\rm green\ laser}
\rho_{00}\approx 1,
\qquad
N_\gamma \text{ tells us about } \rho_{00}.
$$
</div>

In words: the laser both initializes the spin into $m_s=0$ and makes $m_s=0$ brighter than $m_s=\pm1$.

Now add microwaves. A microwave drive near the spin transition mixes $m_s=0$ with $m_s=\pm1$:

<div class="math-display">
$$
\lvert0\rangle
\leftrightarrow
\lvert m_s=\pm1\rangle,
\qquad
f_{\rm mw}\approx f_\pm .
$$
</div>

If the microwave is off resonance, the laser keeps the population mostly in bright $m_s=0$, so fluorescence is high. If the microwave is on resonance, it moves population into darker $m_s=\pm1$, so fluorescence drops:

<div class="math-display">
$$
I_{\rm PL}(f_{\rm mw})
\quad \text{is minimal at} \quad
f_{\rm mw}=f_\pm(B_z).
$$
</div>

This is optically detected magnetic resonance, or ODMR. The name sounds heavier than the experiment: shine laser light, sweep microwave frequency, and record where the red fluorescence dips. The dip position gives $f_\pm$, and $f_\pm$ gives the local field.

There are two common ways to use this physics.

### Resonance-Shift Sensing

The simplest mode measures the frequency of the fluorescence dip:

<div class="math-display">
$$
B_z
\approx
\frac{2\pi}{\gamma_e}
\left(f_+-D/2\pi\right).
$$
</div>

This works with one NV center, but it is also natural for ensembles. With many NV centers in each camera pixel, one can image a spatially varying magnetic field by fitting the dip frequency pixel by pixel. The cost is that an ensemble usually has broader lines because different defects see slightly different strain, fields, and local environments.

### Phase-Accumulation Sensing

For weaker fields, it is often better to use the spin as an interferometer. Prepare

<div class="math-display">
$$
\lvert \psi(0)\rangle
=
\frac{1}{\sqrt 2}
\left(\lvert0\rangle+\lvert1\rangle\right),
$$
</div>

wait for a time $\tau$, and then read out the spin optically. During the wait time, the field changes the relative phase:

<div class="math-display">
$$
\lvert \psi(\tau)\rangle
=
\frac{1}{\sqrt 2}
\left(
\lvert0\rangle
+e^{-i\phi(\tau)}\lvert1\rangle
\right),
\qquad
\phi(\tau)=\int_0^\tau \delta\omega(t)\,dt .
$$
</div>

The final microwave pulse converts this phase into population:

<div class="math-display">
$$
\phi
\longrightarrow
P_0
\longrightarrow
N_\gamma .
$$
</div>

So the measured photon count is ultimately a measurement of accumulated phase. Ramsey, echo, and dynamical-decoupling sequences are different choices of microwave pulses during the wait time. They decide which time-dependence of $\delta\omega(t)$ the NV is most sensitive to.

The useful chain is therefore

<div class="math-display">
$$
\begin{aligned}
B(t) &\rightarrow \delta\omega(t),
\qquad
\Delta E=\hbar\delta\omega,\\
\delta\omega(t) &\rightarrow
\phi(\tau)=\int_0^\tau \delta\omega(t)\,dt,\\
\phi &\rightarrow P_0 \rightarrow N_\gamma .
\end{aligned}
$$
</div>

The sensor does not need a perfect optical transition. For magnetometry, a broad but bright room-temperature optical cycle can be enough, as long as the photon count still depends on the spin state and the spin survives long enough to accumulate a measurable shift.

## 2. Single-Photon Emission

A single-photon emitter is a localized quantum system that emits at most one photon per excitation cycle. If a defect is in its ground state and absorbs one photon, it can be promoted to an excited state. It then relaxes by emitting one photon and returns to the ground state. Until it is excited again, it cannot emit a second photon. That saturation of one localized emitter is the physical origin of antibunching.

The standard diagnostic is the second-order correlation function

<div class="math-display">
$$
g^{(2)}(\tau)
=
\frac{\langle I(t)I(t+\tau)\rangle}{\langle I(t)\rangle^2}.
$$
</div>

For an ideal single-photon source, $g^{(2)}(0)=0$: two photons are not detected at the same time. Experimentally, $g^{(2)}(0)<1/2$ is the usual signature that the light is dominated by a single emitter rather than a classical or multi-emitter source.

For NV$^-$, the optical transition connects the triplet ground state $^3A_2$ to the triplet excited state $^3E$. At room temperature, much of the emission goes into a broad phonon sideband, while the zero-phonon line is near 637 nm. The broad sideband is fine for confirming single-photon emission and for many sensing readout schemes. It is less ideal for interference-based photonic quantum information, where photons from different emitters should be indistinguishable.

The distinction is important:

| Requirement | Ordinary single-photon source | Interference-grade spin-photon interface |
|---|---|---|
| antibunching | essential | essential |
| photostability | essential | essential |
| brightness | important | important |
| narrow optical linewidth | helpful | essential |
| high Debye-Waller factor | helpful | often essential |
| low spectral diffusion | helpful | essential |
| spin-selective optical transitions | optional | essential for spin-photon entanglement |

A localized defect can be a robust single-photon source even if it is not an ideal quantum-network node. The early NV single-photon experiments were important because they showed a stable solid-state source that did not photobleach like many molecules. But NV$^-$ also illustrates a limitation: only a modest fraction of its emission is in the zero-phonon line. For indistinguishable photons, researchers often use cavities to enhance the zero-phonon-line channel, work at cryogenic temperatures, or search for alternative defects such as group-IV vacancy centers in diamond, divacancies in SiC, rare-earth ions, or defects in two-dimensional materials.

The physical design question is therefore not just "does it glow?" It is whether a localized excited state, radiative decay, weak spectral wandering, and efficient collection can combine into useful quantum light.

Computationally, that means one has to care about optical excitation energies, relaxed excited-state geometries, electron-phonon coupling, nonradiative pathways, and how strongly the transition dipole couples to the optical mode. Ground-state defect levels are only the first filter.

## 3. Quantum Computing

For quantum computing, the defect must supply at least one controllable two-level subsystem. In NV$^-$, the natural qubit is a pair of spin sublevels in the $S=1$ ground state, often $\lvert0\rangle\equiv\lvert m_s=0\rangle$ and $\lvert1\rangle\equiv\lvert m_s=-1\rangle$ under a bias magnetic field. Microwave pulses drive rotations between them:

<div class="math-display">
$$
\begin{aligned}
H_{\rm drive}(t)
\approx&
\frac{\hbar\Omega(t)}{2}
\left[
\cos\phi(t)\,\sigma_x+\sin\phi(t)\,\sigma_y
\right] \\
&+
\frac{\hbar\Delta(t)}{2}\sigma_z .
\end{aligned}
$$
</div>

As with other qubit platforms, resonant drive amplitude sets the rotation angle, drive phase sets the rotation axis in the $xy$ plane, and detuning gives a $z$ rotation in the rotating frame.

The NV electron spin can also couple to nearby nuclear spins through hyperfine interactions:

<div class="math-display">
$$
H_{\rm hf}
=
\mathbf S\cdot \mathbf A\cdot \mathbf I .
$$
</div>

Nearby $^{13}$C nuclei or the intrinsic nitrogen nuclear spin can act as longer-lived memory qubits. The electron spin is easy to initialize, manipulate, and read out optically; nuclear spins are slower but can store coherence for longer. This gives a small quantum register around a single defect.

There are three levels of ambition.

### Local Registers

A single defect plus nearby nuclei can implement a few-qubit register. The electron spin acts as an ancilla for control and readout, while nuclear spins provide memory. This is enough for demonstrations of entanglement, simple algorithms, error-correction primitives, and nanoscale sensing protocols that use a memory qubit to preserve information while the electron spin interacts with the environment.

### Coupled Defects

Two nearby electronic spins can couple magnetically through dipole-dipole interactions. The scale is roughly

<div class="math-display">
$$
J_{\rm dd}
\sim
\frac{\mu_0}{4\pi}
\frac{(g\mu_B)^2}{\hbar r^3}.
$$
</div>

This interaction is conceptually clean but hard to scale: the coupling becomes strong only at nanometer separations, while too many nearby spins also create decoherence. Fabrication must place defects accurately and control the spin environment.

### Networked Qubits

A more scalable route is optical networking. If a defect has spin-selective optical transitions, photons can become entangled with the spin. Interfering photons from remote defects can then herald entanglement between distant spin qubits. In this mode, the optical transition is no longer just a readout channel. It becomes the quantum interconnect.

That is why the quantum-computing requirement is harsher than the sensing requirement. A magnetometer can tolerate inhomogeneous broadening if the resonance is still measurable. A networked qubit needs optical linewidths narrow enough for photon indistinguishability, stable charge states, low spectral diffusion, good spin coherence, efficient collection, and high-fidelity operations.

The full qubit checklist looks like this:

| Qubit need | Defect/material property |
|---|---|
| initialization | spin-selective optical pumping or fast measurement feedback |
| one-qubit gates | microwave, optical Raman, or strain/electric control with low leakage |
| readout | spin-dependent fluorescence or spin-to-charge conversion |
| memory | long $T_1$, long $T_2$, weak magnetic and charge noise |
| two-qubit gates | dipolar, exchange, strain, cavity, or photon-mediated coupling |
| scalability | deterministic creation, spectral uniformity, device integration |
| fault tolerance | high operation fidelity and stable calibration |

NV$^-$ is excellent for room-temperature spin control and sensing. It is also a serious qubit and network node, especially with nuclear-spin memories and cryogenic resonant excitation. But it is not perfect: optical spectral diffusion, collection efficiency, and the small zero-phonon-line fraction motivate both nanophotonic engineering and the search for new defects.

## 4. What Defect Papers Actually Compute

The computational workflow is less mysterious if we separate the outputs by what calculation produces them. Most papers do not compute "a quantum technology score." They compute a table of defect quantities, then decide which application those numbers support.

### 4.1 Ground-State Supercell Calculations

Start with a relaxed supercell containing the defect. For each charge state $q$ and spin constraint, calculate a total energy. The standard formation energy is

<div class="math-display">
$$
\begin{aligned}
E_f(D^q;E_F)
&=
E_{\rm tot}(D^q)-E_{\rm tot}({\rm bulk})-\sum_i n_i\mu_i \\
&\quad
+q(E_F+E_{\rm VBM})+E_{\rm corr}.
\end{aligned}
$$
</div>

Here $E_{\rm tot}(D^q)$ is the DFT total energy of the defective supercell, $E_{\rm tot}({\rm bulk})$ is the total energy of the pristine supercell, $n_i$ counts atoms of species $i$ added to the cell, $\mu_i$ is the atomic chemical potential, $E_F$ is the Fermi level measured from the valence-band maximum $E_{\rm VBM}$, and $E_{\rm corr}$ corrects finite-size electrostatics for charged defects.

The charge-transition level between charge states $q$ and $q'$ is the Fermi level where their formation energies are equal:

<div class="math-display">
$$
\epsilon(q/q')
=
\frac{
E_f(D^q;E_F=0)-E_f(D^{q'};E_F=0)
}{q'-q}.
$$
</div>

This answers a concrete question: for which Fermi-level range is NV$^-$ stable rather than NV$^0$ or another charge state?

For the spin state, a collinear DFT calculation usually fixes or initializes

<div class="math-display">
$$
M_z=(N_\uparrow-N_\downarrow)\mu_B,
\qquad
S_z=\frac{N_\uparrow-N_\downarrow}{2}.
$$
</div>

For the NV$^-$ triplet, the usual high-spin representative has $N_\uparrow-N_\downarrow=2$, so $M_z\approx2\mu_B$. This is how VASP sees the $S=1$ electronic configuration. The experimental labels $m_s=0,\pm1$ are then sublevels of this same triplet in the effective spin Hamiltonian.

One also checks whether the defect state is localized. A simple orbital localization measure is the inverse participation ratio

<div class="math-display">
$$
{\rm IPR}[\psi]
=
\frac{\int |\psi(\mathbf r)|^4\,d\mathbf r}
{\left(\int |\psi(\mathbf r)|^2\,d\mathbf r\right)^2}.
$$
</div>

A more localized state has larger ${\rm IPR}$. In practice, papers also plot the defect Kohn-Sham orbitals or the spin density

<div class="math-display">
$$
\sigma(\mathbf r)=n_\uparrow(\mathbf r)-n_\downarrow(\mathbf r).
$$
</div>

At this level the calculation gives: stable charge state, relaxed geometry, spin multiplicity, gap levels, and localization. This is the loose screen.

### 4.2 Spin-Hamiltonian Parameters

After the electronic state is identified, compute the parameters in a spin Hamiltonian:

<div class="math-display">
$$
H_{\rm spin}
=
\mathbf S\cdot \mathbf D\cdot \mathbf S
+\mu_B \mathbf B\cdot \mathbf g\cdot \mathbf S
+\sum_K \mathbf S\cdot \mathbf A^{(K)}\cdot \mathbf I_K .
$$
</div>

Every tensor in this equation is computable.

The zero-field-splitting tensor $\mathbf D$ describes how the spin sublevels split even when $\mathbf B=0$. For the spin-spin contribution,

<div class="math-display">
$$
D_{ab}
\approx
\frac{1}{2}\frac{\mu_0}{4\pi}g_e^2\mu_B^2
\int \rho_2(\mathbf r_1,\mathbf r_2)
\frac{r^2\delta_{ab}-3r_ar_b}{r^5}
\,d\mathbf r_1d\mathbf r_2,
\qquad
\mathbf r=\mathbf r_1-\mathbf r_2 .
$$
</div>

Here $\rho_2$ is the two-electron spin density matrix. In DFT codes this is usually approximated from occupied Kohn-Sham orbitals. For an approximately axial $S=1$ defect, this tensor is often summarized by one number:

<div class="math-display">
$$
H_{\rm ZFS}
=
D\left(S_z^2-\frac{S(S+1)}{3}\right).
$$
</div>

For NV$^-$, this is the origin of the $m_s=0$ versus $m_s=\pm1$ splitting near $2.87$ GHz.

The $g$ tensor tells how a magnetic field shifts the spin levels:

<div class="math-display">
$$
H_Z=\mu_B\mathbf B\cdot\mathbf g\cdot\mathbf S.
$$
</div>

If $\mathbf g=g_e\mathbf I$, the spin behaves like a free electron with $g_e\approx2.0023$. In a crystal, spin-orbit coupling and the local electronic structure make $\mathbf g$ slightly anisotropic. Formally,

<div class="math-display">
$$
g_{ab}
=
\frac{1}{\mu_B}
\left.
\frac{\partial^2 E}{\partial B_a\,\partial S_b}
\right|_{\mathbf B=0}.
$$
</div>

So $\mathbf g$ is the derivative of the energy with respect to magnetic field direction and spin direction. Computationally it is a relativistic response property, often treated with PAW/GIPAW or related methods.

The hyperfine tensor $\mathbf A^{(K)}$ tells how the defect electron spin couples to nucleus $K$:

<div class="math-display">
$$
H_{\rm hf}^{(K)}
=
\mathbf S\cdot \mathbf A^{(K)}\cdot \mathbf I_K .
$$
</div>

It has a contact part from spin density at the nucleus and a dipolar part from spin density around it:

<div class="math-display">
$$
\begin{aligned}
A_{ab}^{(K)}
&=
\frac{2\mu_0}{3}
g_e\mu_B g_K\mu_N
\frac{\sigma(\mathbf R_K)}{S}\delta_{ab} \\
&\quad
+\frac{\mu_0}{4\pi}
g_e\mu_B g_K\mu_N
\frac{1}{S}
\int
\frac{3r_ar_b-r^2\delta_{ab}}{r^5}
\sigma(\mathbf r+\mathbf R_K)\,d\mathbf r .
\end{aligned}
$$
</div>

Here $\mathbf R_K$ is the nuclear position, $\mathbf I_K$ is the nuclear spin operator, $g_K$ is the nuclear $g$ factor, and $\mu_N$ is the nuclear magneton. The vector $\mathbf r$ is measured from nucleus $K$. Large hyperfine couplings identify nearby nuclei that can either disturb the electron spin or act as memory qubits.

### 4.3 Optical Quantities

For single-photon emission, the next calculation is an excited-state calculation. Let $Q_g$ be the relaxed nuclear geometry of the electronic ground state, and $Q_e$ the relaxed geometry of the excited state. Then

<div class="math-display">
$$
E_{\rm ZPL}
=
E_e(Q_e)-E_g(Q_g).
$$
</div>

This is the zero-phonon-line energy: the photon energy if the electronic transition happens without changing vibrational quantum number.

The vertical absorption and emission energies are

<div class="math-display">
$$
E_{\rm abs}
=
E_e(Q_g)-E_g(Q_g),
\qquad
E_{\rm em}
=
E_e(Q_e)-E_g(Q_e).
$$
</div>

From these one gets the relaxation energies

<div class="math-display">
$$
\lambda_e=E_e(Q_g)-E_e(Q_e),
\qquad
\lambda_g=E_g(Q_e)-E_g(Q_g).
$$
</div>

The transition dipole is

<div class="math-display">
$$
\boldsymbol\mu_{eg}
=
\langle \Psi_e | e\mathbf r | \Psi_g\rangle .
$$
</div>

It controls the radiative decay rate. A common estimate is

<div class="math-display">
$$
\Gamma_{\rm rad}
\approx
\frac{n\omega^3|\boldsymbol\mu_{eg}|^2}
{3\pi\epsilon_0\hbar c^3},
\qquad
\tau_{\rm rad}=1/\Gamma_{\rm rad},
$$
</div>

where $\omega=E_{\rm ZPL}/\hbar$ and $n$ is the refractive index of the host.

The Huang-Rhys factor measures how much the atoms move between the two electronic states. In a one-effective-mode approximation,

<div class="math-display">
$$
\Delta Q^2
=
\sum_\alpha M_\alpha
\left|\mathbf R_{e,\alpha}-\mathbf R_{g,\alpha}\right|^2,
\qquad
S_{\rm HR}
=
\frac{\Omega\,\Delta Q^2}{2\hbar}.
$$
</div>

Here $\alpha$ labels atoms, $M_\alpha$ is the atomic mass, $\mathbf R_{g,\alpha}$ and $\mathbf R_{e,\alpha}$ are relaxed positions in the ground and excited states, and $\Omega$ is an effective phonon frequency. In a many-mode treatment,

<div class="math-display">
$$
S_{\rm HR}=\sum_k S_k,
\qquad
S_k=\frac{\omega_k\Delta Q_k^2}{2\hbar}.
$$
</div>

The Debye-Waller factor is the fraction of optical intensity in the zero-phonon line:

<div class="math-display">
$$
{\rm DW}\approx e^{-S_{\rm HR}}.
$$
</div>

Small $S_{\rm HR}$ means less phonon sideband and a larger zero-phonon-line fraction. For a simple single-photon source, one mainly wants a stable optical transition and antibunching. For photon-mediated quantum computing, one wants a large radiative rate into a narrow optical line, so $E_{\rm ZPL}$, $\boldsymbol\mu_{eg}$, $S_{\rm HR}$, and ${\rm DW}$ matter a lot.

### 4.4 Spin-Dependent Optical Cycle

The hardest part of NV-like physics is not "is there an optical transition?" It is whether the optical cycle distinguishes spin states. A minimal rate model uses

<div class="math-display">
$$
\Gamma_{\rm rad}^{(m_s)},
\qquad
\Gamma_{\rm nr}^{(m_s)},
\qquad
\Gamma_{\rm ISC}^{(m_s)} .
$$
</div>

Here $\Gamma_{\rm rad}$ is the photon-emitting decay rate, $\Gamma_{\rm nr}$ is a non-photon nonradiative decay rate, and $\Gamma_{\rm ISC}$ is the rate for the triplet state to cross into singlet states. The readout contrast can be summarized by

<div class="math-display">
$$
C
=
\frac{N_\gamma(m_s=0)-N_\gamma(m_s=\pm1)}
{N_\gamma(m_s=0)+N_\gamma(m_s=\pm1)}.
$$
</div>

The intersystem-crossing rate is commonly modeled using Fermi's golden rule:

<div class="math-display">
$$
\Gamma_{i\rightarrow f}
=
\frac{2\pi}{\hbar}
\left|
\langle \Psi_f | H_{\rm SO} | \Psi_i\rangle
\right|^2
F_{if}.
$$
</div>

$H_{\rm SO}$ is the spin-orbit coupling operator, and $F_{if}$ is a vibrational overlap factor between initial and final nuclear wavefunctions. This is why this final step is harder: one needs excited triplet and singlet states, spin-orbit matrix elements, and electron-phonon coupling, not just a ground-state DFT calculation.

So the practical order is:

1. compute $E_f(D^q)$ and $\epsilon(q/q')$ to find stable charge states;
2. compute $S$, $\sigma(\mathbf r)$, gap levels, and localization;
3. compute $\mathbf D$, $\mathbf g$, and $\mathbf A^{(K)}$ for the spin Hamiltonian;
4. compute $E_{\rm ZPL}$, $\boldsymbol\mu_{eg}$, $\Gamma_{\rm rad}$, $S_{\rm HR}$, and ${\rm DW}$ for optical emission;
5. only for NV-like optical spin readout, compute or model the rates $\Gamma_{\rm rad}^{(m_s)}$, $\Gamma_{\rm ISC}^{(m_s)}$, and the contrast $C$.

That is the loose-to-harsh rationale in computational language. Formation energy and spin density are relatively routine. ZFS, hyperfine, and $g$ are specialized but well-defined spin-Hamiltonian outputs. ZPL and Huang-Rhys require excited-state geometries. Spin-selective readout requires the most detailed excited-state and vibronic information.

## References

- A. Gruber, A. Drabenstedt, C. Tietz, L. Fleury, J. Wrachtrup, and C. von Borczyskowski, "Scanning Confocal Optical Microscopy and Magnetic Resonance on Single Defect Centers," *Science* 276, 2012-2014 (1997): [DOI:10.1126/science.276.5321.2012](https://doi.org/10.1126/science.276.5321.2012).
- C. Kurtsiefer, S. Mayer, P. Zarda, and H. Weinfurter, "Stable Solid-State Source of Single Photons," *Phys. Rev. Lett.* 85, 290-293 (2000): [DOI:10.1103/PhysRevLett.85.290](https://doi.org/10.1103/PhysRevLett.85.290).
- F. Jelezko, T. Gaebel, I. Popa, A. Gruber, and J. Wrachtrup, "Observation of Coherent Oscillations in a Single Electron Spin," *Phys. Rev. Lett.* 92, 076401 (2004): [DOI:10.1103/PhysRevLett.92.076401](https://doi.org/10.1103/PhysRevLett.92.076401).
- P. Neumann et al., "Multipartite Entanglement Among Single Spins in Diamond," *Science* 320, 1326-1329 (2008): [DOI:10.1126/science.1157233](https://doi.org/10.1126/science.1157233).
- J. R. Weber, W. F. Koehl, J. B. Varley, A. Janotti, B. B. Buckley, C. G. Van de Walle, and D. D. Awschalom, "Quantum Computing with Defects," *PNAS* 107, 8513-8518 (2010): [DOI:10.1073/pnas.1003052107](https://doi.org/10.1073/pnas.1003052107).
- E. Togan et al., "Quantum Entanglement Between an Optical Photon and a Solid-State Spin Qubit," *Nature* 466, 730-734 (2010): [DOI:10.1038/nature09256](https://doi.org/10.1038/nature09256).
- L. Robledo et al., "High-Fidelity Projective Read-Out of a Solid-State Spin Quantum Register," *Nature* 477, 574-578 (2011): [DOI:10.1038/nature10401](https://doi.org/10.1038/nature10401).
- M. W. Doherty, N. B. Manson, P. Delaney, F. Jelezko, J. Wrachtrup, and L. C. L. Hollenberg, "The Nitrogen-Vacancy Colour Centre in Diamond," *Physics Reports* 528, 1-45 (2013): [DOI:10.1016/j.physrep.2013.02.001](https://doi.org/10.1016/j.physrep.2013.02.001).
- C. Freysoldt, B. Grabowski, T. Hickel, J. Neugebauer, G. Kresse, A. Janotti, and C. G. Van de Walle, "First-Principles Calculations for Point Defects in Solids," *Rev. Mod. Phys.* 86, 253-305 (2014): [DOI:10.1103/RevModPhys.86.253](https://doi.org/10.1103/RevModPhys.86.253).
- A. Alkauskas, Q. Yan, and C. G. Van de Walle, "First-Principles Theory of Nonradiative Carrier Capture via Multiphonon Emission," *Phys. Rev. B* 90, 075202 (2014): [DOI:10.1103/PhysRevB.90.075202](https://doi.org/10.1103/PhysRevB.90.075202).
- C. L. Degen, F. Reinhard, and P. Cappellaro, "Quantum Sensing," *Rev. Mod. Phys.* 89, 035002 (2017): [DOI:10.1103/RevModPhys.89.035002](https://doi.org/10.1103/RevModPhys.89.035002).
- V. Ivady, I. A. Abrikosov, and A. Gali, "First Principles Calculation of Spin-Related Quantities for Point Defect Qubit Research," *npj Computational Materials* 4, 76 (2018): [DOI:10.1038/s41524-018-0132-5](https://doi.org/10.1038/s41524-018-0132-5).
- A. Gali, "Ab Initio Theory of the Nitrogen-Vacancy Center in Diamond," *Nanophotonics* 8, 1907-1943 (2019): [DOI:10.1515/nanoph-2019-0154](https://doi.org/10.1515/nanoph-2019-0154).
- G. Wolfowicz et al., "Quantum Guidelines for Solid-State Spin Defects," *Nature Reviews Materials* 6, 906-925 (2021): [DOI:10.1038/s41578-021-00306-y](https://doi.org/10.1038/s41578-021-00306-y).
- Z. Fang and Q. Yan, "Towards the Predictive Design of Quantum Defects for Next-Generation Quantum Technologies," *Communications Materials* 7, 155 (2026): [DOI:10.1038/s43246-026-01225-7](https://doi.org/10.1038/s43246-026-01225-7).
