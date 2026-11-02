---
permalink: /blog/hamiltonian-projection-downfolding/
title: "Hamiltonian Projection and Downfolding: One Framework for Many Techniques"
excerpt: "A unified guide to basis transformations, subspace projection, Löwdin/Feshbach downfolding, Schrieffer-Wolff transformations, Wannierization, and TAPW, with a small moiré example."
author_profile: true
---

# Hamiltonian Projection and Downfolding: One Framework for Many Techniques

Microscopic Hamiltonians are usually much larger than the question we want to answer. A first-principles calculation may retain many atoms, orbitals, spins, and bands, while the physics near the Fermi level lives in a small subspace. A quantum-optics model may contain several excited levels even though only two states form the qubit. A Hubbard model may contain doubly occupied configurations even when the low-energy regime is primarily spin physics.

The goal is therefore simple to state:

> Construct a smaller Hamiltonian that preserves the physics of interest.

The vocabulary is less simple. Papers speak of projection, truncation, downfolding, Löwdin partitioning, Feshbach projection, Schrieffer-Wolff transformations, Wannierization, disentanglement, $k\cdot p$ theory, and reduced-basis methods. These terms are sometimes used almost interchangeably even though they describe different operations.

The cleanest taxonomy has only three entries:

<div class="math-display">
$$
\boxed{\text{change basis}}
\qquad
\boxed{\text{select a subspace}}
\qquad
\boxed{\text{integrate out its complement}}.
$$
</div>

Most named methods are combinations of these three operations. This note develops that statement and uses a moiré continuum-model workflow as a compact example.

## 1. Operation I: Change the Basis

Let $H$ be an $N\times N$ Hamiltonian. If $U$ is a square unitary matrix,

<div class="math-display">
$$
U^\dagger U=UU^\dagger=I_N,
\qquad
H'=U^\dagger H U.
$$
</div>

This is a change of coordinates in the same Hilbert space. The dimension remains $N$, no state is discarded, and the spectrum is unchanged. Examples include rotating from real orbitals $(d_{xz},d_{yz})$ to angular-momentum combinations $(d_{xz}\pm i d_{yz})/\sqrt2$, moving between spin quantization axes, and changing the gauge among an isolated set of Bloch bands.

A useful diagnostic is therefore:

<div class="math-display">
$$
\text{square unitary transformation}\neq\text{downfolding}.
$$
</div>

The matrix may become sparse, symmetry adapted, or physically transparent, but it has not become smaller.

## 2. Operation II: Project onto a Smaller Subspace

Now choose $r<N$ orthonormal states $\{|v_a\rangle\}_{a=1}^r$ and collect them as the columns of an $N\times r$ matrix

<div class="math-display">
$$
V=\begin{pmatrix}|v_1\rangle&\cdots&|v_r\rangle\end{pmatrix},
\qquad V^\dagger V=I_r.
$$
</div>

The projector onto their span is

<div class="math-display">
$$
P=VV^\dagger,
\qquad P^2=P,
\qquad P^\dagger=P.
$$
</div>

There are two equivalent-looking formulas that emphasize different objects:

<div class="math-display">
$$
PHP
\qquad\text{and}\qquad
H_{\rm proj}=V^\dagger H V.
$$
</div>

$PHP$ is an operator in the original $N$-dimensional representation whose action is restricted to the selected subspace. $V^\dagger H V$ is the actual $r\times r$ matrix in the reduced basis.

This is genuine dimension reduction, but it is only **projection or truncation**. It retains the matrix elements of $H$ inside the chosen space and discards the rest. It does not automatically preserve the virtual influence of the discarded states.

### What determines the reduced basis?

Many apparently different methods share the formula $V^\dagger H V$. Their real difference is how they choose the columns of $V$.

| Method | Columns of $V$ | Main purpose |
|---|---|---|
| Orbital truncation | selected atomic or molecular orbitals | simple interpretable model |
| Band projection | selected eigenvectors | reproduce chosen bands exactly |
| Symmetry-adapted projection | irreducible-representation states | make symmetry explicit |
| $k\cdot p$ model | band-edge states | continuum expansion near a momentum |
| Wannier disentanglement | optimized Bloch subspace | smooth localized description |
| TAPW | selected atomic plane waves | compress a huge moiré Hamiltonian |
| SVD/POD reduced basis | dominant singular vectors of snapshots | data-driven numerical compression |

This leads to one of the most useful questions in effective-model construction:

<div class="math-display">
$$
\boxed{\text{What physical or numerical principle was used to choose }V?}
$$
</div>

## 3. Why Projection Can Miss Low-Energy Physics

Consider the two-state Hamiltonian

<div class="math-display">
$$
H=
\begin{pmatrix}
0&t\\
t&\Delta
\end{pmatrix},
\qquad |t|\ll |\Delta|.
$$
</div>

Suppose we retain only the first state. Direct projection gives the $1\times1$ Hamiltonian

<div class="math-display">
$$
H_{\rm proj}=0.
$$
</div>

But the exact low-energy eigenvalue is not zero. Expanding it for $|t/\Delta|\ll1$ gives

<div class="math-display">
$$
E_{\rm low}=-\frac{t^2}{\Delta}
+O\!\left(\frac{t^4}{\Delta^3}\right).
$$
</div>

The shift comes from a virtual path

<div class="math-display">
$$
|L\rangle\longrightarrow|H\rangle\longrightarrow|L\rangle.
$$
</div>

Although the high-energy state should not appear explicitly in the final model, its influence should. This is the distinction between **discarding** a state and **integrating it out**.

## 4. Operation III: Integrate Out the Complement

Let $P$ project onto the retained subspace and define

<div class="math-display">
$$
Q=I-P,
\qquad P+Q=I,
\qquad PQ=QP=0.
$$
</div>

In bases adapted to $P$ and $Q$,

<div class="math-display">
$$
H=
\begin{pmatrix}
H_{PP}&H_{PQ}\\
H_{QP}&H_{QQ}
\end{pmatrix},
\qquad
|\Psi\rangle=
\begin{pmatrix}
|\psi_P\rangle\\
|\psi_Q\rangle
\end{pmatrix}.
$$
</div>

The Schrödinger equation gives

<div class="math-display">
$$
\begin{aligned}
H_{PP}|\psi_P\rangle+H_{PQ}|\psi_Q\rangle
&=E|\psi_P\rangle,\\
H_{QP}|\psi_P\rangle+H_{QQ}|\psi_Q\rangle
&=E|\psi_Q\rangle.
\end{aligned}
$$
</div>

When $E-H_{QQ}$ is invertible,

<div class="math-display">
$$
|\psi_Q\rangle=(E-H_{QQ})^{-1}H_{QP}|\psi_P\rangle.
$$
</div>

Substitution into the first equation produces

<div class="math-display">
$$
\boxed{
H_{\rm eff}(E)
=H_{PP}+H_{PQ}(E-H_{QQ})^{-1}H_{QP}
}.
$$
</div>

This is the central formula of Hamiltonian downfolding. The second term describes propagation out of $P$, through the eliminated $Q$ space, and back:

<div class="math-display">
$$
P\longrightarrow Q\longrightarrow P.
$$
</div>

It modifies diagonal energies and can also generate new off-diagonal couplings between retained states.

## 5. Löwdin, Feshbach, Brillouin-Wigner, and the Schur Complement

The preceding derivation has several names because it appears in several mathematical languages.

- **Löwdin partitioning** emphasizes the division of a quantum-mechanical basis into interacting classes.
- **Feshbach projection** emphasizes the complementary projectors $P$ and $Q$.
- **Brillouin-Wigner perturbation theory** emphasizes the energy-dependent effective eigenvalue problem.
- **Resolvent downfolding** emphasizes $(E-H_{QQ})^{-1}$.
- **Schur complementation** is the block-linear-algebra name for the same elimination.

For example, applying block Gaussian elimination to $H-EI$ produces the Schur complement

<div class="math-display">
$$
H_{PP}-E-H_{PQ}(H_{QQ}-E)^{-1}H_{QP},
$$
</div>

which is precisely $H_{\rm eff}(E)-E$. These are not unrelated tricks. They are different interpretations of the same block-matrix identity.

## 6. Exact but Energy Dependent

$H_{\rm eff}(E)$ is formally exact wherever the inverse exists, but it is not an ordinary linear eigenvalue problem because the Hamiltonian depends on the eigenvalue being sought:

<div class="math-display">
$$
H_{\rm eff}(E)|\psi_P\rangle=E|\psi_P\rangle.
$$
</div>

There are several standard ways forward.

1. **Solve the nonlinear problem.** This preserves the exact energy dependence but may be inconvenient.
2. **Evaluate at a reference energy.** Use $H_{\rm eff}(E_{\rm ref})$ for a target energy window.
3. **Expand the resolvent.** If the $P$ and $Q$ sectors are well separated, expand about a convenient energy and truncate.
4. **Construct an energy-independent block diagonalization.** This leads naturally to Schrieffer-Wolff and related canonical transformations.

If $H_{QQ}$ lies roughly an energy $\Delta$ above the retained sector, then schematically

<div class="math-display">
$$
(E-H_{QQ})^{-1}\sim-\frac{1}{\Delta},
\qquad
\Delta H_{\rm eff}\sim-\frac{H_{PQ}H_{QP}}{\Delta}.
$$
</div>

This is the universal second-order pattern $-V^2/\Delta$.

## 7. Schrieffer-Wolff: Rotate Until the Blocks Decouple

The Schrieffer-Wolff approach organizes the same virtual physics differently. Write

<div class="math-display">
$$
H=H_0+V_{\rm d}+V_{\rm od},
$$
</div>

where $V_{\rm d}$ is block diagonal in the $P/Q$ split and $V_{\rm od}$ couples the two blocks. Seek an anti-Hermitian generator $S^\dagger=-S$ such that

<div class="math-display">
$$
\widetilde H=e^SHe^{-S}
$$
</div>

is block diagonal order by order. Expanding with the Baker-Campbell-Hausdorff formula,

<div class="math-display">
$$
\widetilde H
=H+[S,H]+\frac{1}{2}[S,[S,H]]+\cdots.
$$
</div>

At leading order one chooses

<div class="math-display">
$$
[S,H_0]=-V_{\rm od}.
$$
</div>

The low-energy block is then

<div class="math-display">
$$
H_{\rm eff}^{\rm SW}
=P(H_0+V_{\rm d})P
+\frac12P[S,V_{\rm od}]P
+O(V^3).
$$
</div>

The commutator term contains the same $P\rightarrow Q\rightarrow P$ virtual excursions as the second-order expansion of the Löwdin/Feshbach resolvent.

The conceptual distinction is useful:

- Löwdin/Feshbach **eliminates amplitudes in $Q$** and naturally produces an energy-dependent Hamiltonian.
- Schrieffer-Wolff **rotates the full Hamiltonian** so that $P$ and $Q$ decouple perturbatively, producing an energy-independent low-energy block.

They need not yield literally identical matrices at every finite truncation or in every gauge, but properly constructed effective Hamiltonians reproduce the same low-energy observables to the claimed perturbative order.

## 8. A Familiar Example: Superexchange

Consider two sites with hopping $t$ and a large onsite repulsion $U$. At half filling, states with one electron per site form the low-energy sector. Double occupation costs energy $U$ and belongs to the eliminated sector.

An electron can hop to form a virtual doubly occupied state and then hop back. Integrating out those charge fluctuations produces a spin interaction

<div class="math-display">
$$
H_{\rm spin}=J\,\mathbf S_1\cdot\mathbf S_2,
\qquad
J\simeq\frac{4t^2}{U}.
$$
</div>

The doubly occupied configurations disappear from the effective Hilbert space, but their virtual effect becomes the antiferromagnetic exchange $J$. This is downfolding in its most recognizable form.

The same logic gives ligand-mediated hopping in solids:

<div class="math-display">
$$
d_i\longrightarrow p\longrightarrow d_j,
\qquad
t_{dd}^{\rm eff}\sim
\frac{t_{dp}^2}{\epsilon_d-\epsilon_p}.
$$
</div>

Removing the ligand orbital without this correction would miss the effective metal-metal hopping.

## 9. When Plain Projection Is Already Exact

Projection fails only when the retained space couples to its complement. If the range of $P$ is invariant under $H$,

<div class="math-display">
$$
QHP=0,
$$
</div>

then $H_{QP}=H_{PQ}=0$ for Hermitian $H$, and the downfolding correction vanishes. In this case

<div class="math-display">
$$
H_{\rm eff}=V^\dagger H V
$$
</div>

is exact.

Selecting exact eigenvectors is the obvious example. If $V=(|\psi_1\rangle,\ldots,|\psi_r\rangle)$, then

<div class="math-display">
$$
V^\dagger H V=\operatorname{diag}(E_1,\ldots,E_r).
$$
</div>

Why not always do this? Because the exact eigenvector basis can be a poor **modeling basis**. Its gauge may vary irregularly with momentum, and its states may not have stable layer, sublattice, orbital, valley, or spin labels. A compact model should often reproduce not only eigenvalues but also symmetries, matrix elements, locality, topology, and physical interpretation.

## 10. Wannierization: Subspace Selection Plus Gauge Choice

Wannier construction illustrates why projection and basis transformation often appear together. Starting from Bloch states $|\psi_{n\mathbf k}\rangle$, define

<div class="math-display">
$$
|w_{m\mathbf k}\rangle
=\sum_n|\psi_{n\mathbf k}\rangle U_{nm}(\mathbf k),
$$
</div>

and Fourier transform to real space:

<div class="math-display">
$$
|w_{m\mathbf R}\rangle
=\frac{1}{N_{\mathbf k}}
\sum_{\mathbf k}e^{-i\mathbf k\cdot\mathbf R}
|w_{m\mathbf k}\rangle.
$$
</div>

For an isolated group with the same number of input and output bands, $U(\mathbf k)$ is a unitary gauge transformation within that band subspace. For entangled bands, one first extracts a lower-dimensional smooth subspace and then chooses a gauge inside it. Wannierization can therefore contain both

<div class="math-display">
$$
\text{subspace selection}
+
\text{unitary gauge optimization}.
$$
</div>

It does not, by itself, imply that remote-band virtual effects have been added through a Löwdin or Schrieffer-Wolff correction. Sometimes those effects are already encoded in the selected exact band subspace; sometimes a separately constructed orbital model needs explicit downfolding.

## 11. Numerical Reduced-Basis Methods

Not every subspace is chosen from orbitals or bands. Suppose representative wavefunctions or solution snapshots are arranged as columns of

<div class="math-display">
$$
\Psi=\begin{pmatrix}|\psi_1\rangle&|\psi_2\rangle&\cdots\end{pmatrix}.
$$
</div>

An SVD,

<div class="math-display">
$$
\Psi=U\Sigma W^\dagger,
$$
</div>

identifies the dominant directions. Keeping the first $r$ left singular vectors $U_r$ gives

<div class="math-display">
$$
H_r=U_r^\dagger H U_r.
$$
</div>

This appears as proper orthogonal decomposition, reduced-order modeling, low-rank projection, or a reduced-basis method. It is again a projection. The difference is that the subspace is selected by approximation quality on training data rather than by an orbital or energy label.

## 12. Nonorthogonal Bases and the Two Meanings of “Löwdin”

Localized atomic orbitals are often nonorthogonal. Their coefficients satisfy a generalized eigenproblem

<div class="math-display">
$$
Hc=ESc,
\qquad
S_{ij}=\langle\phi_i|\phi_j\rangle.
$$
</div>

After projection with $V$,

<div class="math-display">
$$
H_{\rm red}=V^\dagger HV,
\qquad
S_{\rm red}=V^\dagger SV,
$$
</div>

and the reduced problem remains

<div class="math-display">
$$
H_{\rm red}c=E S_{\rm red}c.
$$
</div>

Symmetric Löwdin orthogonalization defines

<div class="math-display">
$$
\widetilde V
=V(V^\dagger S V)^{-1/2},
\qquad
\widetilde V^\dagger S\widetilde V=I.
$$
</div>

The orthonormal Hamiltonian is $\widetilde V^\dagger H\widetilde V$.

Two distinct procedures unfortunately carry Löwdin's name:

- **Löwdin orthogonalization** removes the overlap metric $S$.
- **Löwdin partitioning** integrates out a complementary subspace.

The first changes the representation and metric. The second constructs an effective Hamiltonian. They should not be conflated.

## 13. TAPW and Moiré Models as a Running Example

Large moiré supercells make the hierarchy of operations especially visible. A localized-orbital calculation may contain thousands or tens of thousands of basis functions, while low-energy states near a chosen valley involve a restricted set of momentum components.

The **Truncated Atomic Plane Wave** construction forms coherent combinations schematically like

<div class="math-display">
$$
|\psi^{\rm TAPW}_{\ell\alpha\mathbf G}(\mathbf k)\rangle
=\frac{1}{\sqrt N}
\sum_i
e^{i(\mathbf k+\mathbf G)\cdot\boldsymbol\tau_{\ell i}}
|\ell,i,\alpha\rangle,
$$
</div>

where $\ell$ labels a layer, $i$ an atomic or monolayer cell inside the moiré cell, $\alpha$ internal orbital/spin structure, and $\mathbf G$ a selected moiré reciprocal vector near the target valley.

Writing these basis states as columns of $X$, the first compression is

<div class="math-display">
$$
H_{\rm TAPW}=X^\dagger H_{\rm loc}X,
\qquad
S_{\rm TAPW}=X^\dagger S_{\rm loc}X.
$$
</div>

This is primarily a physically designed **projection**. It replaces an enormous real-space basis by a much smaller valley-centered plane-wave basis. With nonorthogonal atomic orbitals, one next constructs

<div class="math-display">
$$
H_{\rm TAPW}^{\rm orth}
=S_{\rm TAPW}^{-1/2}
H_{\rm TAPW}
S_{\rm TAPW}^{-1/2}.
$$
</div>

That is **Löwdin orthogonalization**, not yet the elimination of remote TAPW states.

The TAPW space can then be split into selected low-energy states $L$ and excluded states $H$. A fixed-reference-energy reduction has the form

<div class="math-display">
$$
H_{\rm eff}(\mathbf k;E_{\rm ref})
=H_{LL}(\mathbf k)
+H_{LH}(\mathbf k)
\left[E_{\rm ref}I-H_{HH}(\mathbf k)\right]^{-1}
H_{HL}(\mathbf k).
$$
</div>

This second reduction is the genuine **Löwdin/Feshbach downfolding** step. Finally, a physically interpretable gauge and the symmetry representations are used to express the result in a compact continuum basis,

<div class="math-display">
$$
H_{\rm continuum}(\mathbf k)
=\sum_j\theta_jY_j(\mathbf k),
$$
</div>

where the $Y_j$ are symmetry-allowed operators.

The whole pipeline can therefore be read as

<div class="math-display">
$$
H_{\rm localized}
\xrightarrow[\text{compression}]{\text{TAPW projection}}
H_{\rm TAPW}
\xrightarrow{\text{orthogonalization}}
H_{\rm TAPW}^{\rm orth}
\xrightarrow[\text{remote-state effects}]{\text{Löwdin downfolding}}
H_{\rm eff}
\xrightarrow[\text{gauge and symmetry}]{\text{operator expansion}}
H_{\rm continuum}.
$$
</div>

This example is useful precisely because it contains all three fundamental operations without treating them as synonyms.

## 14. What Can Be Generated by Downfolding?

Virtual excursions through eliminated states can produce much more than a constant energy shift. Depending on the problem, they generate

- renormalized onsite energies and dispersions;
- longer-range or symmetry-allowed hopping;
- superexchange and other spin couplings;
- spin-orbit interactions;
- correlated hopping and multibody terms;
- screening and frequency-dependent interactions;
- corrections to observables as well as to the Hamiltonian.

The last point matters. If a unitary transformation is used to derive $H_{\rm eff}$, observables should in principle be transformed too:

<div class="math-display">
$$
O_{\rm eff}=P e^S O e^{-S}P.
$$
</div>

An effective Hamiltonian can reproduce low-energy energies while a bare projected observable still gives inaccurate matrix elements.

## 15. When Is a Reduction Trustworthy?

A reduced model should come with diagnostics, not only a small matrix. Useful questions include:

1. **Energy separation:** Is the retained manifold separated from the discarded states over the full parameter and momentum range?
2. **Hybridization:** Is $\|H_{PQ}\|/\Delta$ small enough for the chosen perturbative order?
3. **Resolvent poles:** Does the target energy approach the spectrum of $H_{QQ}$?
4. **Subspace quality:** Do the retained states have large overlap with the target exact states?
5. **Gauge smoothness:** Does the basis vary smoothly with momentum and parameters?
6. **Symmetry:** Does projection or numerical truncation preserve the required representations?
7. **Topology:** Is a globally smooth localized basis obstructed by band topology?
8. **Induced terms:** Were relevant longer-range, multibody, or frequency-dependent terms discarded after downfolding?
9. **Reference-energy sensitivity:** Does changing $E_{\rm ref}$ appreciably change the predicted bands or observables?
10. **Operator consistency:** Were observables transformed consistently with the Hamiltonian when necessary?

Near a crossing between retained and eliminated states, a small-denominator expansion is not controlled. The cure is often to enlarge $P$, not to compute more terms in a failing series.

## 16. A Compact Decision Guide

| Goal | Natural operation |
|---|---|
| Express the same problem in a clearer basis | square unitary transformation |
| Keep chosen orbitals or numerical modes | rectangular projection $V^\dagger HV$ |
| Reproduce an exactly isolated set of bands | projection onto its invariant subspace |
| Obtain localized orbitals | Wannier subspace selection and gauge optimization |
| Compress a large moiré atomistic model | TAPW or another physics-informed projection |
| Preserve virtual effects of remote states exactly | energy-dependent Löwdin/Feshbach reduction |
| Obtain a static perturbative low-energy model | Schrieffer-Wolff/canonical transformation |
| Build a data-efficient simulator | SVD/POD/reduced-basis projection |

Often the correct workflow is a sequence rather than a single choice: first construct a meaningful reduced basis, then integrate out its remaining remote sector, then impose symmetry and simplify.

## 17. The Unified Picture

The apparent zoo of methods reduces to three equations:

<div class="math-display">
$$
\boxed{
\begin{aligned}
\text{Change representation:}\quad
&H' = U^\dagger H U,\\[3pt]
\text{Select a subspace:}\quad
&H_{\rm proj}=V^\dagger H V,\\[3pt]
\text{Integrate out }Q:\quad
&H_{\rm eff}(E)
=H_{PP}+H_{PQ}(E-H_{QQ})^{-1}H_{QP}.
\end{aligned}}
$$
</div>

The first preserves the full problem. The second makes it smaller by discarding states. The third makes it smaller while retaining their virtual influence.

Wannierization, TAPW, $k\cdot p$, tight-binding construction, Löwdin partitioning, Feshbach projection, and Schrieffer-Wolff transformations are best understood as different answers to three modeling questions:

1. Which subspace contains the physics we care about?
2. Which basis makes that physics interpretable and symmetry transparent?
3. How accurately must the discarded sector be remembered?

Once those questions are separated, “projection” and “downfolding” stop being mysterious names and become a controlled sequence of linear-algebra and perturbation-theory choices.

## References

- P.-O. Löwdin, "On the Non-Orthogonality Problem Connected with the Use of Atomic Wave Functions in the Theory of Molecules and Crystals," *J. Chem. Phys.* **18**, 365 (1950): [DOI:10.1063/1.1747632](https://doi.org/10.1063/1.1747632).
- P.-O. Löwdin, "A Note on the Quantum-Mechanical Perturbation Theory," *J. Chem. Phys.* **19**, 1396 (1951): [DOI:10.1063/1.1748067](https://doi.org/10.1063/1.1748067).
- H. Feshbach, "Unified Theory of Nuclear Reactions," *Ann. Phys.* **5**, 357-390 (1958): [DOI:10.1016/0003-4916(58)90007-1](https://doi.org/10.1016/0003-4916(58)90007-1).
- J. R. Schrieffer and P. A. Wolff, "Relation between the Anderson and Kondo Hamiltonians," *Phys. Rev.* **149**, 491 (1966): [DOI:10.1103/PhysRev.149.491](https://doi.org/10.1103/PhysRev.149.491).
- S. Bravyi, D. P. DiVincenzo, and D. Loss, "Schrieffer-Wolff Transformation for Quantum Many-Body Systems," *Ann. Phys.* **326**, 2793-2826 (2011): [arXiv:1105.0675](https://arxiv.org/abs/1105.0675).
- N. Marzari and D. Vanderbilt, "Maximally Localized Generalized Wannier Functions for Composite Energy Bands," *Phys. Rev. B* **56**, 12847 (1997): [DOI:10.1103/PhysRevB.56.12847](https://doi.org/10.1103/PhysRevB.56.12847).
- I. Souza, N. Marzari, and D. Vanderbilt, "Maximally Localized Wannier Functions for Entangled Energy Bands," *Phys. Rev. B* **65**, 035109 (2001): [DOI:10.1103/PhysRevB.65.035109](https://doi.org/10.1103/PhysRevB.65.035109).
- W. Miao, C. Li, X. Han, D. Pan, and X. Dai, "Truncated Atomic Plane Wave Method for the Subband Structure Calculations of Moiré Systems" (2022): [arXiv:2210.02026](https://arxiv.org/abs/2210.02026).
- Y. Zhang et al., "MoireKP: An Automated Workflow for Symmetry-Constrained Moiré Continuum Models" (2026): [arXiv:2610.37760](https://arxiv.org/abs/2610.37760).
