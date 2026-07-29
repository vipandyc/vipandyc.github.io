---
permalink: /blog/transmon-qubit-tutorial/
title: "A Tutorial Introduction to the Transmon Qubit"
excerpt: "A compact lecture note on Josephson junctions, transmon circuit quantization, circuit-QED cavity coupling, universal gates, and using superconducting qubits to simulate spin Hamiltonians."
author_profile: true
---

# A Tutorial Introduction to the Transmon Qubit

The transmon is the workhorse superconducting qubit. It is not a microscopic two-level atom; it is an engineered nonlinear LC oscillator made from a Josephson junction and a large shunting capacitance. The trick is to make the oscillator only weakly anharmonic, so it is insensitive to charge noise, while still keeping enough anharmonicity to address the lowest two levels as a qubit.

This note builds the story in five steps:

1. a Josephson junction gives a nondissipative nonlinear inductance;
2. adding capacitance gives a quantized circuit Hamiltonian;
3. the transmon regime protects against charge noise;
4. coupling to a microwave cavity enables control, readout, and qubit-qubit interactions;
5. single- and two-qubit gates let us simulate spin Hamiltonians.

## 1. Josephson Junction: A Nonlinear Inductor

A Josephson junction consists of two superconducting islands separated by a thin insulating barrier. The macroscopic superconducting phases on the two sides are $\theta_1$ and $\theta_2$. The important coordinate is the gauge-invariant phase difference

<div class="math-display">
$$
\varphi=\theta_1-\theta_2-\frac{2\pi}{\Phi_0}\int_1^2 \mathbf A\cdot d\mathbf l,
\qquad
\Phi_0=\frac{h}{2e}.
$$
</div>

The phase $\varphi$ controls coherent Cooper-pair tunneling. The two Josephson relations are

<div class="math-display">
$$
I=I_c\sin\varphi,
\qquad
V=\frac{\Phi_0}{2\pi}\dot\varphi.
$$
</div>

The first equation says the current is dissipationless but nonlinear. The second says voltage is the time derivative of phase. Because power is $P=IV$, the Josephson energy satisfies

<div class="math-display">
$$
\frac{dU_J}{dt}=IV
=I_c\sin\varphi\frac{\Phi_0}{2\pi}\dot\varphi
=\frac{d}{dt}\left[-E_J\cos\varphi\right],
\qquad
E_J=\frac{\Phi_0 I_c}{2\pi}.
$$
</div>

Thus the junction contributes the potential energy

<div class="math-display">
$$
U_J(\varphi)=-E_J\cos\varphi.
$$
</div>

Near a minimum, $\cos\varphi\approx 1-\varphi^2/2+\varphi^4/24-\cdots$, so the junction behaves like an inductor plus nonlinear corrections:

<div class="math-display">
$$
-E_J\cos\varphi
\approx
-E_J+\frac{E_J}{2}\varphi^2-\frac{E_J}{24}\varphi^4+\cdots.
$$
</div>

The quadratic part is equivalent to an inductance

<div class="math-display">
$$
L_J=\left(\frac{\Phi_0}{2\pi}\right)^2\frac{1}{E_J}.
$$
</div>

The nonlinear terms are the essential ingredient. A perfectly harmonic oscillator has evenly spaced levels, so driving $\lvert0\rangle\to\lvert1\rangle$ also drives $\lvert1\rangle\to\lvert2\rangle$. A Josephson oscillator has unequal spacings, letting us isolate two levels.

## 2. Canonical Variables: Phase and Cooper-Pair Number

For a superconducting island, the charge is quantized in Cooper-pair units. Let

<div class="math-display">
$$
n=\frac{Q}{2e}
$$
</div>

be the number of excess Cooper pairs on the island. The phase $\varphi$ and number $n$ are conjugate variables:

<div class="math-display">
$$
[\varphi,n]=i.
$$
</div>

This is the circuit analogue of $[x,p]=i\hbar$. In the phase basis,

<div class="math-display">
$$
n=-i\frac{\partial}{\partial\varphi}.
$$
</div>

The uncertainty relation

<div class="math-display">
$$
\Delta\varphi\,\Delta n\gtrsim \frac{1}{2}
$$
</div>

is why charge qubits and phase-like qubits have complementary noise sensitivities. A state localized in charge has uncertain phase; a state localized in phase has uncertain charge.

## 3. The Cooper-Pair Box Hamiltonian

Start with one superconducting island connected to ground through a Josephson junction and biased by a gate capacitance. The electrostatic energy of the island is

<div class="math-display">
$$
E_{\rm ch}=\frac{(Q-Q_g)^2}{2C_\Sigma},
\qquad
C_\Sigma=C_J+C_g,
\qquad
Q_g=C_g V_g.
$$
</div>

Using $Q=2en$ and defining

<div class="math-display">
$$
E_C=\frac{e^2}{2C_\Sigma},
\qquad
n_g=\frac{Q_g}{2e},
$$
</div>

the Hamiltonian is

<div class="math-display">
$$
H_{\rm CPB}=4E_C(n-n_g)^2-E_J\cos\varphi.
$$
</div>

This is the Cooper-pair box. In the charge basis $\lvert n\rangle$,

<div class="math-display">
$$
4E_C(n-n_g)^2
$$
</div>

is diagonal, while the Josephson term hops by one Cooper pair:

<div class="math-display">
$$
-E_J\cos\varphi
=-\frac{E_J}{2}\left(e^{i\varphi}+e^{-i\varphi}\right),
\qquad
e^{\pm i\varphi}\lvert n\rangle=\lvert n\pm1\rangle.
$$
</div>

So in the charge basis,

<div class="math-display">
$$
H_{\rm CPB}
=
\sum_n 4E_C(n-n_g)^2\lvert n\rangle\langle n\rvert
-\frac{E_J}{2}\sum_n
\left(\lvert n+1\rangle\langle n\rvert+\lvert n\rangle\langle n+1\rvert\right).
$$
</div>

When $E_J/E_C$ is small, the eigenstates strongly depend on $n_g$. That makes the qubit sensitive to fluctuating offset charge. The transmon solves this by adding a large shunt capacitor.

## 4. The Transmon Circuit

The transmon is the same Hamiltonian as the Cooper-pair box,

<div class="math-display">
$$
H_{\rm transmon}=4E_C(n-n_g)^2-E_J\cos\varphi,
$$
</div>

but operated in the regime

<div class="math-display">
$$
\frac{E_J}{E_C}\gg 1,
\qquad
\text{typically } E_J/E_C\sim 20\text{--}100.
$$
</div>

The large capacitance makes $E_C=e^2/(2C_\Sigma)$ small. Physically, the wavefunction becomes broad in charge and localized in phase near a minimum of the cosine. This exponentially suppresses sensitivity to offset charge $n_g$.

To see the spectrum, expand the cosine near $\varphi=0$:

<div class="math-display">
$$
H
\approx
4E_C n^2+\frac{E_J}{2}\varphi^2
-E_J-\frac{E_J}{24}\varphi^4+\cdots.
$$
</div>

The quadratic terms form a harmonic oscillator with plasma frequency

<div class="math-display">
$$
\hbar\omega_p=\sqrt{8E_JE_C}.
$$
</div>

The quartic term makes the oscillator weakly anharmonic. Perturbation theory gives approximately

<div class="math-display">
$$
E_m
\approx
-E_J+\sqrt{8E_JE_C}\left(m+\frac{1}{2}\right)
-\frac{E_C}{12}(6m^2+6m+3).
$$
</div>

Therefore the qubit transition and anharmonicity are

<div class="math-display">
$$
\hbar\omega_{01}=E_1-E_0\approx \sqrt{8E_JE_C}-E_C,
$$
</div>

<div class="math-display">
$$
\hbar\alpha
=
(E_2-E_1)-(E_1-E_0)
\approx
-E_C.
$$
</div>

The negative anharmonicity means the $\lvert1\rangle\to\lvert2\rangle$ transition is lower in frequency than the $\lvert0\rangle\to\lvert1\rangle$ transition. Typical numbers are

| Quantity | Common scale |
|---|---:|
| $\omega_{01}/2\pi$ | $4$ to $8$ GHz |
| $\alpha/2\pi$ | $-100$ to $-300$ MHz |
| $E_J/E_C$ | $20$ to $100$ |
| gate pulses | $5$ to $100$ ns |

The design compromise is now clear:

| Increasing $E_J/E_C$ does this | Consequence |
|---|---|
| suppresses charge dispersion | better dephasing time |
| decreases relative anharmonicity | more leakage to $\lvert2\rangle$ |
| localizes phase | better oscillator picture |

The transmon is a good qubit because it sits in the sweet spot: charge noise is strongly reduced, but $\alpha$ is still large enough to spectrally resolve the qubit transition.

## 5. Quantizing the Transmon as a Weakly Anharmonic Oscillator

It is often useful to write the transmon in ladder-operator form. For the harmonic part,

<div class="math-display">
$$
\varphi=\varphi_{\rm zpf}(b+b^\dagger),
\qquad
n=i n_{\rm zpf}(b^\dagger-b),
$$
</div>

with zero-point fluctuations

<div class="math-display">
$$
\varphi_{\rm zpf}=\left(\frac{2E_C}{E_J}\right)^{1/4},
\qquad
n_{\rm zpf}=\left(\frac{E_J}{32E_C}\right)^{1/4}.
$$
</div>

Keeping only the dominant Kerr nonlinearity, one writes

<div class="math-display">
$$
H_{\rm transmon}
\approx
\hbar\omega_q b^\dagger b
\frac{\hbar\alpha}{2}b^\dagger b^\dagger b b.
$$
</div>

If we restrict to the two lowest levels,

<div class="math-display">
$$
\lvert0\rangle,\quad \lvert1\rangle,
$$
</div>

then the qubit Hamiltonian is

<div class="math-display">
$$
H_q=\frac{\hbar\omega_q}{2}\sigma_z
$$
</div>

up to an irrelevant constant. The full oscillator model is still important in simulations because strong pulses can populate $\lvert2\rangle$ and higher states. Leakage is one of the main practical constraints in transmon gate design.

## 6. Why Couple the Transmon to a Cavity?

A bare transmon is a small circuit element. We need a way to drive it, measure it, and connect it to other qubits without wiring directly into every degree of freedom. Circuit QED does this by coupling the transmon to a microwave resonator, usually a coplanar waveguide or 3D cavity mode.

At the simplest level, the cavity is a harmonic oscillator:

<div class="math-display">
$$
H_r=\hbar\omega_r a^\dagger a.
$$
</div>

Capacitive coupling between the resonator voltage and the transmon charge gives

<div class="math-display">
$$
H_{\rm int}\propto V_r n
\propto (a+a^\dagger)n.
$$
</div>

In a two-level approximation and after the rotating-wave approximation, this becomes the Jaynes-Cummings Hamiltonian:

<div class="math-display">
$$
H_{\rm JC}
=
\hbar\omega_r a^\dagger a
+\frac{\hbar\omega_q}{2}\sigma_z
+\hbar g(a^\dagger\sigma_-+a\sigma_+).
$$
</div>

The cavity is useful in several distinct ways.

### Readout

When the qubit and cavity are detuned,

<div class="math-display">
$$
\Delta=\omega_q-\omega_r,
\qquad
|\Delta|\gg g,
$$
</div>

they do not resonantly exchange excitations. Instead, the qubit shifts the cavity frequency. The effective dispersive Hamiltonian is

<div class="math-display">
$$
H_{\rm disp}
\approx
\hbar(\omega_r+\chi\sigma_z)a^\dagger a
+\frac{\hbar}{2}(\omega_q+\chi)\sigma_z.
$$
</div>

For a weakly anharmonic transmon,

<div class="math-display">
$$
\chi\approx -\frac{g^2\alpha}{\Delta(\Delta+\alpha)}.
$$
</div>

Thus the cavity resonance is different depending on whether the qubit is in $\lvert0\rangle$ or $\lvert1\rangle$. Sending in a microwave probe and measuring the reflected or transmitted phase gives a qubit measurement.

### Control

The cavity or feedline also provides a microwave port. A drive near $\omega_q$ produces rotations of the qubit:

<div class="math-display">
$$
H_d(t)=\hbar\Omega(t)\cos(\omega_d t+\phi_d)\sigma_x.
$$
</div>

In a frame rotating at the drive frequency and under the rotating-wave approximation,

<div class="math-display">
$$
H_{\rm rot}
=
-\frac{\hbar\delta}{2}\sigma_z
+\frac{\hbar\Omega(t)}{2}
\left(\cos\phi_d\,\sigma_x+\sin\phi_d\,\sigma_y\right),
\qquad
\delta=\omega_d-\omega_q.
$$
</div>

The microwave amplitude sets the rotation angle; the microwave phase selects the rotation axis in the $xy$ plane.

### Coupling Qubits

A resonator can act as a bus. If two qubits couple to the same cavity, virtual photons mediate an effective interaction. Depending on the device architecture and drive scheme, the useful low-energy terms can look like exchange,

<div class="math-display">
$$
H_{\rm ex}
=
\hbar J(\sigma_1^+\sigma_2^-+\sigma_1^-\sigma_2^+)
=
\frac{\hbar J}{2}(\sigma_1^x\sigma_2^x+\sigma_1^y\sigma_2^y),
$$
</div>

or conditional phase interactions,

<div class="math-display">
$$
H_{ZZ}
=
\frac{\hbar\zeta}{4}\sigma_1^z\sigma_2^z.
$$
</div>

Modern processors may also use direct capacitive coupling plus tunable couplers, but the cavity-QED language remains the cleanest conceptual starting point: resonators turn small artificial atoms into addressable, measurable, connectable quantum systems.

## 7. Single-Qubit Gates

A single-qubit pure state is

<div class="math-display">
$$
\lvert\psi\rangle
=
\cos\frac{\theta}{2}\lvert0\rangle
+e^{i\phi}\sin\frac{\theta}{2}\lvert1\rangle.
$$
</div>

On the Bloch sphere, arbitrary single-qubit gates are rotations

<div class="math-display">
$$
R_{\hat n}(\Theta)
=
\exp\left[-i\frac{\Theta}{2}\hat n\cdot\boldsymbol\sigma\right].
$$
</div>

Microwave control directly implements rotations in the equatorial plane:

<div class="math-display">
$$
R_\phi(\Theta)
=
\exp\left[
-i\frac{\Theta}{2}
(\cos\phi\,\sigma_x+\sin\phi\,\sigma_y)
\right],
\qquad
\Theta=\int \Omega(t)\,dt.
$$
</div>

Special cases are

<div class="math-display">
$$
X=R_x(\pi),
\qquad
Y=R_y(\pi),
\qquad
X_{\pi/2}=R_x(\pi/2).
$$
</div>

$Z$ rotations can be physical detuning pulses, but in most superconducting processors they are implemented virtually. A frame update

<div class="math-display">
$$
R_z(\lambda)=e^{-i\lambda\sigma_z/2}
$$
</div>

is equivalent to shifting the phase of subsequent microwave pulses. Virtual $Z$ gates are fast and nearly error-free because they are classical bookkeeping.

Any single-qubit unitary can be decomposed using Euler angles, for example

<div class="math-display">
$$
U=e^{i\gamma}R_z(\alpha)R_x(\beta)R_z(\delta).
$$
</div>

So microwave $X/Y$ rotations plus virtual $Z$ rotations give arbitrary one-qubit control.

## 8. Two-Qubit Gates and Universality

Single-qubit gates alone cannot create entanglement. Universal quantum computation requires arbitrary single-qubit rotations plus at least one entangling two-qubit gate.

The most common superconducting two-qubit gates include:

| Gate | Mechanism | Ideal unitary |
|---|---|---|
| controlled-Z (CZ) | conditional phase from avoided crossings or tunable couplers | $\mathrm{diag}(1,1,1,-1)$ |
| cross-resonance (CR) | drive one fixed-frequency transmon at the other qubit's frequency | approximately $e^{-i\theta Z\otimes X/2}$ |
| iSWAP / fSim | exchange interaction between near-resonant qubits | swaps excitation with phases |

The controlled-NOT can be built from CZ and Hadamards:

<div class="math-display">
$$
\mathrm{CNOT}_{1\to2}
=
(I\otimes H)\,\mathrm{CZ}\,(I\otimes H).
$$
</div>

Why is this universal? Any $N$-qubit unitary can be decomposed into a sequence of one-qubit gates and CNOTs. Equivalently, arbitrary single-qubit rotations plus any entangling two-qubit gate generate a dense subgroup of $SU(2^N)$. In practical language:

<div class="math-display">
$$
\begin{aligned}
&\text{single-qubit rotations}\\
&\quad+\text{ one entangling two-qubit gate}\\
&\Longrightarrow \text{ universal quantum gates}.
\end{aligned}
$$
</div>

For transmons, the caveat is that the physical system has more than two levels. A high-fidelity gate must do the desired operation on the computational subspace while returning almost all population from leakage levels such as $\lvert2\rangle$.

## 9. From Qubits to Spin Hamiltonians

Now suppose we want to simulate a spin-$1/2$ Hamiltonian,

<div class="math-display">
$$
H_{\rm spin}
=
\sum_i h_i^x\sigma_i^x
+\sum_i h_i^z\sigma_i^z
+\sum_{i<j}
\left(
J_{ij}^x\sigma_i^x\sigma_j^x
+J_{ij}^y\sigma_i^y\sigma_j^y
+J_{ij}^z\sigma_i^z\sigma_j^z
\right).
$$
</div>

A transmon qubit already is a spin-$1/2$ after truncation:

<div class="math-display">
$$
\lvert0\rangle\equiv \lvert\uparrow\rangle,
\qquad
\lvert1\rangle\equiv \lvert\downarrow\rangle.
$$
</div>

There are two broad simulation modes.

### Digital Simulation

Digital simulation approximates time evolution

<div class="math-display">
$$
U(t)=e^{-iH_{\rm spin}t}
$$
</div>

as a product of elementary gates. If $H=\sum_\ell H_\ell$, a first-order Trotter formula is

<div class="math-display">
$$
e^{-iHt}
\approx
\left[
\prod_\ell e^{-iH_\ell \Delta t}
\right]^r,
\qquad
\Delta t=\frac{t}{r}.
$$
</div>

For example, for a transverse-field Ising model,

<div class="math-display">
$$
H_{\rm TFIM}
=
\sum_i h_i X_i
+\sum_i m_i Z_i
+\sum_{\langle i,j\rangle} J_{ij} Z_iZ_j,
$$
</div>

one Trotter step is

<div class="math-display">
$$
U(\Delta t)
\approx
\prod_i e^{-ih_i X_i\Delta t}
\prod_i e^{-im_i Z_i\Delta t}
\prod_{\langle i,j\rangle} e^{-iJ_{ij}Z_iZ_j\Delta t}.
$$
</div>

The first two factors are single-qubit rotations:

<div class="math-display">
$$
e^{-ih_i X_i\Delta t}=R_x(2h_i\Delta t),
\qquad
e^{-im_i Z_i\Delta t}=R_z(2m_i\Delta t).
$$
</div>

The $ZZ$ interaction can be compiled using CNOTs:

<div class="math-display">
$$
e^{-i\theta Z_iZ_j}
=
\mathrm{CNOT}_{i\to j}
\left(I_i\otimes R_z(2\theta)_j\right)
\mathrm{CNOT}_{i\to j}.
$$
</div>

So a spin model becomes a circuit made of rotations and entangling gates.

### Basis Changes for Other Couplings

If hardware naturally gives $ZZ$ gates, we can still simulate $XX$ and $YY$ interactions by rotating basis before and after the $ZZ$ evolution:

<div class="math-display">
$$
e^{-i\theta X_iX_j}
=
(H_iH_j)
e^{-i\theta Z_iZ_j}
(H_iH_j),
$$
</div>

because $H Z H=X$. Similarly,

<div class="math-display">
$$
e^{-i\theta Y_iY_j}
=
(S_i^\dagger H_i S_j^\dagger H_j)
e^{-i\theta Z_iZ_j}
(H_i S_i H_j S_j),
$$
</div>

up to equivalent phase-convention choices. The main idea is simple: rotate the measurement axis of the qubit, apply the native entangling phase, then rotate back.

### Analog and Floquet Simulation

In analog simulation, we tune the hardware Hamiltonian itself to resemble the target model. Coupled transmons naturally produce terms such as exchange,

<div class="math-display">
$$
H_{\rm XY}
=
\sum_i \frac{\hbar\omega_i}{2}Z_i
+\sum_{i<j}\frac{\hbar J_{ij}}{2}(X_iX_j+Y_iY_j),
$$
</div>

which is already an $XY$ spin model. Parametric modulation, tunable couplers, and microwave drives can dress this into effective Hamiltonians with adjustable signs, strengths, and rotating-frame fields.

Floquet engineering goes one step further: apply periodic drives so the stroboscopic evolution over one period $T$ is

<div class="math-display">
$$
U_F=\mathcal T e^{-i\int_0^T H(t)\,dt}
=e^{-iH_{\rm eff}T}.
$$
</div>

Then the simulated Hamiltonian is the effective generator $H_{\rm eff}$ rather than the instantaneous circuit Hamiltonian.

## 10. Practical Simulation Workflow

A compact workflow for simulating a spin Hamiltonian on transmons is:

1. choose the spin model and encode each spin as one transmon qubit;
2. split $H$ into terms that are easy to compile, such as $X_i$, $Z_i$, $Z_iZ_j$, $X_iX_j+Y_iY_j$;
3. choose digital, analog, or hybrid simulation;
4. for digital simulation, Trotterize $e^{-iHt}$ and compile each term into native gates;
5. include hardware constraints: connectivity, gate duration, leakage, crosstalk, readout error, and decoherence;
6. measure observables by rotating into the desired basis and reading out $Z$.

For an observable such as

<div class="math-display">
$$
\langle X_iX_j\rangle,
$$
</div>

apply $H$ gates to qubits $i$ and $j$ before measurement, then measure $Z_iZ_j$. For

<div class="math-display">
$$
\langle Y_iY_j\rangle,
$$
</div>

rotate $Y$ to $Z$ with an $S^\dagger H$-type basis change, then measure.

## 11. Minimal Numerical Model

For a small number of qubits, the exact state-vector simulation is straightforward. The building blocks are Pauli strings. For example,

<div class="math-display">
$$
H
=
h\sum_i X_i
+J\sum_i Z_iZ_{i+1}.
$$
</div>

Construct each many-qubit operator by tensor products:

<div class="math-display">
$$
X_i
=
I\otimes\cdots\otimes X\otimes\cdots\otimes I,
\qquad
Z_iZ_{i+1}
=
I\otimes\cdots\otimes Z\otimes Z\otimes\cdots\otimes I.
$$
</div>

Then evolve

<div class="math-display">
$$
\lvert\psi(t)\rangle=e^{-iHt}\lvert\psi(0)\rangle.
$$
</div>

A real device implements the same logic indirectly through calibrated pulses and gates. The clean mathematical model is the spin Hamiltonian; the hardware layer turns each term into microwave pulses on anharmonic superconducting oscillators.

## 12. Big Picture

The transmon is a carefully engineered compromise:

| Ingredient | Role |
|---|---|
| Josephson junction | nonlinear inductance and anharmonic spectrum |
| shunt capacitance | reduces charge noise by lowering $E_C$ |
| lowest two levels | define the computational qubit |
| microwave drive | implements single-qubit rotations |
| cavity or resonator | enables readout, control ports, and mediated coupling |
| entangling gates | make computation and spin simulation universal |

In one line:

<div class="math-display">
$$
\begin{array}{c}
\text{Josephson nonlinearity}\\
+\ \text{capacitance and microwave control}\\
+\ \text{cavity-mediated measurement/coupling}\\[2mm]
\Downarrow\\[1mm]
\text{programmable spin-1/2 quantum hardware}
\end{array}
$$
</div>

That is why the transmon sits at the center of superconducting quantum computing: it turns circuit parameters into a controllable Hamiltonian, and controllable Hamiltonians are exactly what we need for gates, measurement, and quantum simulation.
