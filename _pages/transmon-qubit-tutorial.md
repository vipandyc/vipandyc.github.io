---
permalink: /blog/transmon-qubit-tutorial/
title: "A Tutorial Introduction to the Transmon Qubit"
excerpt: "A compact lecture note on Josephson junctions, transmon circuit quantization, circuit-QED cavity coupling, universal gates, and using superconducting qubits to simulate spin Hamiltonians."
author_profile: true
---

# A Tutorial Introduction to the Transmon Qubit

The transmon is the workhorse superconducting qubit. It is not a microscopic two-level atom; it is an engineered nonlinear LC oscillator made from a Josephson junction and a large shunting capacitance. The trick is to make the oscillator only weakly anharmonic, so it is insensitive to charge noise, while still keeping enough anharmonicity to address the lowest two levels as a qubit.

This note builds the story in five steps. First, a Josephson junction gives a nondissipative nonlinear inductance. Second, adding capacitance gives a quantized circuit Hamiltonian. Third, the transmon regime protects against charge noise. Fourth, coupling to a microwave cavity enables control, readout, and qubit-qubit interactions. Fifth, single- and two-qubit gates let us simulate spin Hamiltonians.

I will keep $\hbar$ explicit in Hamiltonians. Frequencies such as $\omega_q$ are angular frequencies; ordinary frequencies are written as $f_q=\omega_q/2\pi$. The Pauli convention is $Z\lvert0\rangle=+\lvert0\rangle$ and $Z\lvert1\rangle=-\lvert1\rangle$ unless stated otherwise.

## 1. Josephson Junction: A Nonlinear Inductor

A Josephson junction consists of two superconducting islands separated by a thin insulating barrier. The macroscopic superconducting phases on the two sides are $\theta_1$ and $\theta_2$. The important coordinate is the gauge-invariant phase difference, with $\Phi_0=h/(2e)$:

<div class="math-display">
$$
\varphi=\theta_1-\theta_2-\frac{2\pi}{\Phi_0}\int_1^2 \mathbf A\cdot d\mathbf l .
$$
</div>

The phase $\varphi$ controls coherent Cooper-pair tunneling. The two Josephson relations are central:

<div class="math-display">
$$
I=I_c\sin\varphi,
\qquad
V=\frac{\Phi_0}{2\pi}\dot\varphi.
$$
</div>

The first equation says the current is dissipationless but nonlinear. The second says voltage is the time derivative of phase. With $P=IV$,

<div class="math-display">
$$
\frac{dU_J}{dt}=IV
=I_c\sin\varphi\frac{\Phi_0}{2\pi}\dot\varphi
=\frac{d}{dt}\left[-E_J\cos\varphi\right],
\qquad
E_J=\frac{\Phi_0 I_c}{2\pi}.
$$
</div>

Thus the junction contributes $U_J(\varphi)=-E_J\cos\varphi$. Near a minimum, $\cos\varphi\approx 1-\varphi^2/2+\varphi^4/24-\cdots$, so the junction behaves like an inductor plus nonlinear corrections:

<div class="math-display">
$$
-E_J\cos\varphi
\approx
-E_J+\frac{E_J}{2}\varphi^2-\frac{E_J}{24}\varphi^4+\cdots.
$$
</div>

The quadratic part is equivalent to a Josephson inductance $L_J=(\Phi_0/2\pi)^2/E_J=\Phi_0/(2\pi I_c)$ near $\varphi=0$. More generally, the small-signal inductance around a biased phase is $L_J(\varphi_0)=\Phi_0/[2\pi I_c\cos\varphi_0]$.

The nonlinear terms are the essential ingredient. A perfectly harmonic oscillator has evenly spaced levels, so driving $\lvert0\rangle\to\lvert1\rangle$ also drives $\lvert1\rangle\to\lvert2\rangle$. A Josephson oscillator has unequal spacings, letting us isolate two levels.

## 2. Canonical Variables: Phase and Cooper-Pair Number

For a superconducting island, the charge is quantized in Cooper-pair units: $n=Q/(2e)$. The phase $\varphi$ and number $n$ are conjugate variables, $[\varphi,n]=i$, so in the phase basis $n=-i\partial_\varphi$. This is the circuit analogue of $[x,p]=i\hbar$.

It is often cleaner to start from the node flux $\Phi=(\Phi_0/2\pi)\varphi$. For a capacitance $C$, the Lagrangian kinetic energy is $C\dot\Phi^2/2$, the conjugate charge is $Q=\partial L/\partial\dot\Phi=C\dot\Phi$, and quantization gives $[\Phi,Q]=i\hbar$. Dividing by $\Phi_0/2\pi$ and $2e$ gives $[\varphi,n]=i$.

The uncertainty relation $\Delta\varphi\,\Delta n\gtrsim 1/2$ is why charge qubits and phase-like qubits have complementary noise sensitivities. A state localized in charge has uncertain phase; a state localized in phase has uncertain charge.

## 3. The Cooper-Pair Box Hamiltonian

Start with one superconducting island connected to ground through a Josephson junction and biased by a gate capacitance. The electrostatic energy is $E_{\rm ch}=(Q-Q_g)^2/(2C_\Sigma)$, where $C_\Sigma=C_J+C_g$ and $Q_g=C_gV_g$. Using $Q=2en$, $E_C=e^2/(2C_\Sigma)$, and $n_g=Q_g/(2e)$ gives the Cooper-pair box Hamiltonian

<div class="math-display">
$$
H_{\rm CPB}=4E_C(n-n_g)^2-E_J\cos\varphi.
$$
</div>

In the charge basis $\lvert n\rangle$, the charging term $4E_C(n-n_g)^2$ is diagonal, while the Josephson term hops by one Cooper pair because $e^{\pm i\varphi}\lvert n\rangle=\lvert n\pm1\rangle$:

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

When $E_J/E_C$ is small, the eigenstates strongly depend on $n_g$. In the two-charge-state limit near $n_g=1/2$, the CPB is approximately $H\simeq -2E_C(1-2n_g)\tau_z-(E_J/2)\tau_x$, so small offset-charge fluctuations directly move the qubit frequency. The transmon solves this by increasing $C_\Sigma$, thereby decreasing $E_C$.

## 4. The Transmon Circuit

The transmon is the same Hamiltonian as the Cooper-pair box, $H_{\rm transmon}=4E_C(n-n_g)^2-E_J\cos\varphi$, but operated in the regime $E_J/E_C\gg1$, typically $E_J/E_C\sim20$ to $100$.

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

The quadratic terms form a harmonic oscillator with plasma energy $\hbar\omega_p=\sqrt{8E_JE_C}$. The quartic term makes the oscillator weakly anharmonic. Perturbation theory gives approximately

<div class="math-display">
$$
E_m
\approx
-E_J+\sqrt{8E_JE_C}\left(m+\frac{1}{2}\right)
-\frac{E_C}{12}(6m^2+6m+3).
$$
</div>

Therefore

<div class="math-display">
$$
\hbar\omega_{01}=E_1-E_0\approx \sqrt{8E_JE_C}-E_C,
\qquad
\hbar\alpha=(E_2-E_1)-(E_1-E_0)\approx -E_C.
$$
</div>

The negative anharmonicity means the $\lvert1\rangle\to\lvert2\rangle$ transition is lower in frequency than the $\lvert0\rangle\to\lvert1\rangle$ transition. If you know a desired $f_{01}=\omega_{01}/2\pi$ and anharmonicity $\alpha/2\pi<0$, a quick design estimate is $E_C/h\approx \lvert\alpha\rvert/2\pi$ and $E_J/h\approx (f_{01}+E_C/h)^2/[8(E_C/h)]$.

The stronger statement is about charge dispersion. For level $m$, the residual modulation with offset charge is exponentially small:

<div class="math-display">
$$
\epsilon_m
\sim
(-1)^m E_C
\frac{2^{4m+5}}{m!}\sqrt{\frac{2}{\pi}}
\left(\frac{E_J}{2E_C}\right)^{m/2+3/4}
e^{-\sqrt{8E_J/E_C}} .
$$
</div>

The important scaling is the exponential $e^{-\sqrt{8E_J/E_C}}$: charge sensitivity dies rapidly, while the anharmonicity only shrinks algebraically as $\alpha\approx -E_C/\hbar$. Typical numbers are

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

The transmon is a good qubit because it sits in the sweet spot: charge noise is strongly reduced, but $\lvert\alpha\rvert$ is still large compared with pulse bandwidths, so the qubit transition remains spectrally resolvable.

## 5. Quantizing the Transmon as a Weakly Anharmonic Oscillator

It is often useful to write the transmon in ladder-operator form. For the harmonic part,

<div class="math-display">
$$
\varphi=\varphi_{\rm zpf}(b+b^\dagger),
\qquad
n=i n_{\rm zpf}(b^\dagger-b),
\qquad
\varphi_{\rm zpf}n_{\rm zpf}=\frac{1}{2}.
$$
</div>

with zero-point fluctuations $\varphi_{\rm zpf}=(2E_C/E_J)^{1/4}$ and $n_{\rm zpf}=(E_J/32E_C)^{1/4}$. In the transmon regime, $\varphi_{\rm zpf}\ll1$ and $n_{\rm zpf}\gg1$: the wavefunction is phase-localized and charge-delocalized.

Keeping only the dominant Kerr nonlinearity, one writes

<div class="math-display">
$$
H_{\rm transmon}
\approx
\hbar\omega_q b^\dagger b
+\frac{\hbar\alpha}{2}b^\dagger b^\dagger b b,
\qquad
\alpha<0.
$$
</div>

Equivalently, if $N=b^\dagger b$, then $E_N/\hbar\approx \omega_qN+\alpha N(N-1)/2$. This form makes leakage transparent: $\omega_{12}=\omega_{01}+\alpha$, so a pulse with bandwidth comparable to $\lvert\alpha\rvert$ can unintentionally drive $\lvert1\rangle\to\lvert2\rangle$.

If we restrict to the two lowest levels, $\lvert0\rangle$ and $\lvert1\rangle$, then the qubit Hamiltonian is $H_q=-(\hbar\omega_q/2)Z$ under the convention $Z\lvert0\rangle=+\lvert0\rangle$ and $Z\lvert1\rangle=-\lvert1\rangle$, up to an irrelevant constant. Many papers write $+\hbar\omega_q\sigma_z/2$ with the opposite ground/excited eigenvalue convention; either is fine if used consistently. The full oscillator model is still important in simulations because strong pulses can populate $\lvert2\rangle$ and higher states.

## 6. Why Couple the Transmon to a Cavity?

A bare transmon is a small circuit element. We need a way to drive it, measure it, and connect it to other qubits without wiring directly into every degree of freedom. Circuit QED does this by coupling the transmon to a microwave resonator, usually a coplanar waveguide or 3D cavity mode.

At the simplest level, the cavity is a harmonic oscillator, $H_r=\hbar\omega_r a^\dagger a$. Capacitive coupling between the resonator voltage and the transmon charge gives

<div class="math-display">
$$
H_{\rm int}\propto V_r n
\propto (a+a^\dagger)n.
$$
</div>

More explicitly, $V_r=V_{\rm zpf}(a+a^\dagger)$ and $H_{\rm int}=2e\beta V_{\rm zpf}(a+a^\dagger)n$, where $\beta$ is a capacitance-divider factor. The adjacent-level coupling is $g_j=(2e\beta V_{\rm zpf}/\hbar)\langle j\rvert n\lvert j+1\rangle$. In a two-level approximation and after the rotating-wave approximation, this becomes the Jaynes-Cummings Hamiltonian:

<div class="math-display">
$$
H_{\rm JC}
=
\hbar\omega_r a^\dagger a
-\frac{\hbar\omega_q}{2}Z
+\hbar g(a^\dagger\sigma_-+a\sigma_+).
$$
</div>

The cavity is useful in several distinct ways.

### Readout

When the qubit and cavity are detuned, $\Delta=\omega_q-\omega_r$ and $\lvert\Delta\rvert\gg g$, they do not resonantly exchange excitations. Instead, the qubit shifts the cavity frequency. The effective dispersive Hamiltonian is

<div class="math-display">
$$
H_{\rm disp}
\approx
\hbar(\omega_r-\chi Z)a^\dagger a
-\frac{\hbar}{2}(\omega_q+\chi)Z.
$$
</div>

For a weakly anharmonic transmon with $\alpha=\omega_{12}-\omega_{01}<0$,

<div class="math-display">
$$
\chi\approx \frac{g^2\alpha}{\Delta(\Delta+\alpha)}.
$$
</div>

Thus the cavity resonance is different depending on whether the qubit is in $\lvert0\rangle$ or $\lvert1\rangle$: roughly $\omega_r\pm\chi$. Sending in a microwave probe and measuring the reflected or transmitted phase gives a qubit measurement. The same coupling creates a Purcell decay channel, roughly $\Gamma_{\rm Purcell}\approx\kappa(g/\Delta)^2$ in the two-level estimate, where $\kappa$ is the cavity linewidth. One also keeps the photon number below $n_{\rm crit}\approx\Delta^2/(4g^2)$ so the dispersive approximation does not break down.

### Control

The cavity or feedline also provides a microwave port. A drive near $\omega_q$ produces rotations of the qubit. In the lab frame, a minimal model is $H_d(t)=\hbar\Omega(t)\cos(\omega_d t+\phi_d)X$. In a frame rotating at the drive frequency and under the rotating-wave approximation,

<div class="math-display">
$$
H_{\rm rot}
=
\frac{\hbar\delta}{2}Z
+\frac{\hbar\Omega(t)}{2}
\left(\cos\phi_d\,X+\sin\phi_d\,Y\right),
\qquad
\delta=\omega_d-\omega_q.
$$
</div>

The microwave amplitude sets the rotation angle $\Theta=\int\Omega(t)\,dt$; the microwave phase selects the rotation axis in the $xy$ plane. Because a transmon has $\lvert2\rangle$, practical pulses often use DRAG-style quadrature correction, approximately $\Omega_y(t)\propto-\dot\Omega_x(t)/\alpha$, to reduce leakage and phase errors.

### Coupling Qubits

A resonator can act as a bus. If two qubits couple to the same cavity, virtual photons mediate an effective interaction. For two detuned qubits coupled to one bus, a rough exchange scale is $J_{12}\sim g_1g_2(1/\Delta_1+1/\Delta_2)/2$, where $\Delta_i=\omega_i-\omega_r$. Depending on the device architecture and drive scheme, the useful low-energy terms can look like exchange,

<div class="math-display">
$$
H_{\rm ex}
=
\hbar J(\sigma_1^+\sigma_2^-+\sigma_1^-\sigma_2^+)
=
\frac{\hbar J}{2}(X_1X_2+Y_1Y_2),
$$
</div>

or conditional phase interactions,

<div class="math-display">
$$
H_{ZZ}=\frac{\hbar\zeta}{4}Z_1Z_2,
\qquad
U_{ZZ}(t)=e^{-i\zeta t Z_1Z_2/4}.
$$
</div>

Modern processors may also use direct capacitive coupling plus tunable couplers, but the cavity-QED language remains the cleanest conceptual starting point: resonators turn small artificial atoms into addressable, measurable, connectable quantum systems.

## 7. Single-Qubit Gates

A single-qubit pure state is $\lvert\psi\rangle=\cos(\theta/2)\lvert0\rangle+e^{i\phi}\sin(\theta/2)\lvert1\rangle$. On the Bloch sphere, arbitrary single-qubit gates are rotations $R_{\hat n}(\Theta)=\exp[-i(\Theta/2)\hat n\cdot\boldsymbol\sigma]$.

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

Special cases are $X=R_x(\pi)$, $Y=R_y(\pi)$, and $X_{\pi/2}=R_x(\pi/2)$. $Z$ rotations can be physical detuning pulses, but in most superconducting processors they are implemented virtually: a frame update $R_z(\lambda)=e^{-i\lambda Z/2}$ is equivalent to shifting the phase of subsequent microwave pulses. Virtual $Z$ gates are fast and nearly error-free because they are classical bookkeeping.

Any single-qubit unitary can be decomposed using Euler angles, for example $U=e^{i\gamma}R_z(\alpha)R_x(\beta)R_z(\delta)$. So microwave $X/Y$ rotations plus virtual $Z$ rotations give arbitrary one-qubit control.

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

A useful phase form is

<div class="math-display">
$$
\mathrm{CZ}
\sim
e^{-i\pi(I-Z_1)(I-Z_2)/4},
\qquad
U_{\rm iSWAP}(\theta)
=
\exp\!\left[-i\frac{\theta}{2}(X_1X_2+Y_1Y_2)\right],
$$
</div>

where $\sim$ means equal up to single-qubit and global phases.

Why is this universal? Any $N$-qubit unitary can be decomposed into a sequence of one-qubit gates and CNOTs. Equivalently, arbitrary single-qubit rotations plus any entangling two-qubit gate generate a universal gate set. In practical language: single-qubit rotations plus one entangling two-qubit gate imply universal quantum gates.

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

A transmon qubit already is a spin-$1/2$ after truncation: identify $\lvert0\rangle\equiv\lvert\uparrow\rangle$ and $\lvert1\rangle\equiv\lvert\downarrow\rangle$. A local field term is just a single-qubit rotation generator, while a spin-spin term is an entangling generator.

There are two broad simulation modes.

### Digital Simulation

Digital simulation approximates time evolution $U(t)=e^{-iH_{\rm spin}t}$ as a product of elementary gates. If $H=\sum_\ell H_\ell$, a first-order Trotter formula is

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

The leading error comes from noncommuting terms: schematically $\lVert U-U_{\rm Trotter}\rVert=O[t^2 r^{-1}\sum_{\ell<m}\lVert[H_\ell,H_m]\rVert]$. A common symmetric second-order step is

<div class="math-display">
$$
e^{-iH\Delta t}
\approx
\prod_{\ell=1}^L e^{-iH_\ell\Delta t/2}
\prod_{\ell=L}^1 e^{-iH_\ell\Delta t/2}
+O(\Delta t^3).
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

The first two factors are single-qubit rotations: $e^{-ih_iX_i\Delta t}=R_x(2h_i\Delta t)$ and $e^{-im_iZ_i\Delta t}=R_z(2m_i\Delta t)$.

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

The general Pauli-string recipe is worth remembering. For $P=P_1P_2\cdots P_k$ with each $P_j\in\{X,Y,Z\}$,

<div class="math-display">
$$
e^{-i\theta P}
=
B^\dagger
\left[
\mathrm{CNOT\ ladder}\; R_z(2\theta)\; \mathrm{uncompute}
\right]
B,
$$
</div>

where $B$ rotates every non-$Z$ Pauli into $Z$: use $H X H=Z$ and $H S^\dagger Y S H=Z$ up to phase convention. This is the digital-simulation workhorse: basis change, parity compute, phase rotate, uncompute.

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
-\sum_i \frac{\hbar\omega_i}{2}Z_i
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

In a rotating frame, driven transmons are especially flexible. A drive detuning gives a controllable $Z$ field, a resonant quadrature gives $X/Y$ fields, and tunable exchange gives $XX+YY$. This is why the same device can act either as a gate-model computer or as a programmable spin simulator.

## 10. Practical Simulation Workflow

A compact workflow for simulating a spin Hamiltonian on transmons is:

1. choose the spin model and encode each spin as one transmon qubit;
2. split $H$ into terms that are easy to compile, such as $X_i$, $Z_i$, $Z_iZ_j$, $X_iX_j+Y_iY_j$;
3. choose digital, analog, or hybrid simulation;
4. for digital simulation, Trotterize $e^{-iHt}$ and compile each term into native gates;
5. include hardware constraints: connectivity, gate duration, leakage, crosstalk, readout error, and decoherence;
6. measure observables by rotating into the desired basis and reading out $Z$.

For an observable such as $\langle X_iX_j\rangle$, apply $H$ gates to qubits $i$ and $j$ before measurement, then measure $Z_iZ_j$. For $\langle Y_iY_j\rangle$, rotate $Y$ to $Z$ with an $S^\dagger H$-type basis change, then measure. More generally, for a Pauli string $P$, estimate $\langle P\rangle=\sum_b p_b\,\lambda_b$ after basis rotation, where $b$ is a measured bitstring and $\lambda_b=\pm1$ is the corresponding Pauli eigenvalue product.

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

Then evolve $\lvert\psi(t)\rangle=e^{-iHt}\lvert\psi(0)\rangle$. For open-system checks, replace this with the Lindblad master equation $\dot\rho=-i[H,\rho]+\sum_\mu(L_\mu\rho L_\mu^\dagger-\{L_\mu^\dagger L_\mu,\rho\}/2)$, using $L_1=\sqrt{1/T_1}\,\sigma_-$ for relaxation and $L_\phi=\sqrt{1/(2T_\phi)}\,Z$ for pure dephasing.

A real device implements the same logic indirectly through calibrated pulses and gates. The clean mathematical model is the spin Hamiltonian; the hardware layer turns each term into microwave pulses on anharmonic superconducting oscillators.

## 12. Extended Reading

For a first serious pass through the original literature, these are the references I would read in roughly this order.

| Topic | Reference |
|---|---|
| Transmon design | J. Koch et al., ["Charge-insensitive qubit design derived from the Cooper pair box"](https://doi.org/10.1103/PhysRevA.76.042319), Phys. Rev. A 76, 042319 (2007). |
| Circuit QED theory | A. Blais et al., ["Cavity quantum electrodynamics for superconducting electrical circuits"](https://doi.org/10.1103/PhysRevA.69.062320), Phys. Rev. A 69, 062320 (2004). |
| First strong-coupling cQED experiment | A. Wallraff et al., ["Strong coupling of a single photon to a superconducting qubit using circuit quantum electrodynamics"](https://doi.org/10.1038/nature02851), Nature 431, 162-167 (2004). |
| Modern cQED review | A. Blais, A. L. Grimsmo, S. M. Girvin, and A. Wallraff, ["Circuit quantum electrodynamics"](https://doi.org/10.1103/RevModPhys.93.025005), Rev. Mod. Phys. 93, 025005 (2021). |
| Superconducting circuits review | M. H. Devoret, A. Wallraff, and J. M. Martinis, ["Superconducting qubits: A short review"](https://arxiv.org/abs/cond-mat/0411174), arXiv:cond-mat/0411174. |
| Quantum gates and universality | M. A. Nielsen and I. L. Chuang, [*Quantum Computation and Quantum Information*](https://doi.org/10.1017/CBO9780511976667), especially the one-qubit/CNOT universality construction. |
| Universal quantum simulation | S. Lloyd, ["Universal Quantum Simulators"](https://doi.org/10.1126/science.273.5278.1073), Science 273, 1073-1078 (1996). |
| Quantum simulation review | I. M. Georgescu, S. Ashhab, and F. Nori, ["Quantum simulation"](https://doi.org/10.1103/RevModPhys.86.153), Rev. Mod. Phys. 86, 153 (2014). |

## 13. Big Picture

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
