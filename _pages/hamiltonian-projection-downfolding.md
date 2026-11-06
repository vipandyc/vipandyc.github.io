---
permalink: /blog/hamiltonian-projection-downfolding/
title: "Hamiltonian Projection and Downfolding: One Framework for Many Techniques"
excerpt: "A unified guide to basis transformations, subspace projection, Löwdin/Feshbach downfolding, Schrieffer-Wolff transformations, Wannierization, and TAPW, with a small moiré example."
author_profile: true
---

# Hamiltonian Projection and Downfolding: One Framework for Many Techniques

Microscopic Hamiltonians are often much larger than the question of interest. A first-principles model may retain many orbitals and bands while the physics near the Fermi level lives in a small subspace. The goal is simple:

> Construct a smaller Hamiltonian that preserves the physics of interest.

The vocabulary is not. Projection, downfolding, Löwdin partitioning, Feshbach projection, Schrieffer-Wolff transformations, Wannierization, and reduced-basis methods are often mixed together despite describing different operations.

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

Most named methods combine these operations. A moiré continuum-model workflow will serve as a compact example.

## 1. Operation I: Change the Basis

Let $H$ be an $N\times N$ Hamiltonian. If $U$ is a square unitary matrix,

<div class="math-display">
$$
U^\dagger U=UU^\dagger=I_N,
\qquad
H'=U^\dagger H U.
$$
</div>

This changes coordinates in the same Hilbert space. The dimension and spectrum are unchanged; no state is discarded. Examples include rotating orbitals, changing the spin axis, or changing the gauge among isolated Bloch bands. Thus

<div class="math-display">
$$
\text{square unitary transformation}\neq\text{downfolding}.
$$
</div>

The matrix may become clearer, but it has not become smaller.

## 2. Operation II: Project onto a Smaller Subspace

Now choose $r<N$ orthonormal states $\{\lvert v_a\rangle\}_{a=1}^r$ and collect them as the columns of an $N\times r$ matrix

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

The projected Hamiltonian has two representations:

<div class="math-display">
$$
PHP
\qquad\text{and}\qquad
H_{\rm proj}=V^\dagger H V.
$$
</div>

$PHP$ is an $N\times N$ matrix that acts only within the selected subspace. $H_{\rm proj}=V^\dagger H V$ is its $r\times r$ matrix in the basis formed by the columns of $V$. They are related by inserting $P=VV^\dagger$:

<div class="math-display">
$$
PHP=(VV^\dagger)H(VV^\dagger)
=V\underbrace{(V^\dagger H V)}_{H_{\rm proj}}V^\dagger.
$$
</div>

So these are **representations of the same projected action**, not matrices of the same size. If a reduced state has coordinates $c\in\mathbb C^r$, its full-space vector is $Vc$, and

<div class="math-display">
$$
(PHP)(Vc)=V(H_{\rm proj}c).
$$
</div>

For a concrete $N=3$, $r=2$ example, retain the first two coordinate states:

<div class="math-display">
$$
H=\begin{pmatrix}2&1&7\\1&4&8\\7&8&9\end{pmatrix},
\quad
V=\begin{pmatrix}1&0\\0&1\\0&0\end{pmatrix},
\quad
P=VV^\dagger=\begin{pmatrix}1&0&0\\0&1&0\\0&0&0\end{pmatrix}.
$$
</div>

Now both forms are easy to see:

<div class="math-display">
$$
PHP=\begin{pmatrix}2&1&0\\1&4&0\\0&0&0\end{pmatrix}
\quad\longleftrightarrow\quad
H_{\rm proj}=V^\dagger H V=\begin{pmatrix}2&1\\1&4\end{pmatrix}.
$$
</div>

**Yes:** in a basis whose first $r$ vectors span the retained subspace, $PHP$ has the reduced $r\times r$ matrix as its only potentially nonzero block. In a general original basis, the nonzero entries need not sit in one block. For example, retain $v=(1,1)^\top/\sqrt2$ from $H=\operatorname{diag}(2,4)$. Then $r=1$, but

<div class="math-display">
$$
H_{\rm proj}=v^\dagger H v=3,
\qquad
PHP=v(3)v^\dagger=\frac32\begin{pmatrix}1&1\\1&1\end{pmatrix}.
$$
</div>

The full-space matrix has four nonzero entries despite representing a one-dimensional operator. If we rotate the full basis so that its first vector is $v$, that same $PHP$ becomes $\operatorname{diag}(3,0)$.

This is genuine dimension reduction, but it is only **projection or truncation**. It retains the matrix elements of $H$ inside the chosen space and discards the rest. It does not automatically preserve the virtual influence of the discarded states.

### What determines the reduced basis?

Many methods share $V^\dagger H V$; their real difference is how they choose $V$.

| Method | Columns of $V$ | Main purpose |
|---|---|---|
| Orbital truncation | selected atomic or molecular orbitals | simple interpretable model |
| Band projection | selected eigenvectors | reproduce chosen bands exactly |
| Symmetry-adapted projection | irreducible-representation states | make symmetry explicit |
| $k\cdot p$ model | band-edge states | continuum expansion near a momentum |
| Wannier disentanglement | optimized Bloch subspace | smooth localized description |
| TAPW | selected atomic plane waves | compress a huge moiré Hamiltonian |
| SVD/POD reduced basis | dominant singular vectors of snapshots | data-driven numerical compression |

The essential question is therefore:

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

But the exact low-energy eigenvalue is not zero. Expanding it for $\lvert t/\Delta\rvert\ll1$ gives

<div class="math-display">
$$
E_{\rm low}=-\frac{t^2}{\Delta}
+O\!\left(\frac{t^4}{\Delta^3}\right).
$$
</div>

The shift comes from the virtual path $L\rightarrow H\rightarrow L$. The high-energy state should disappear from the final basis, but not from the physics. That is the distinction between **discarding** and **integrating out** a state.

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

The same elimination appears under several names:

- **Löwdin partitioning:** interacting basis classes;
- **Feshbach projection:** complementary projectors $P$ and $Q$;
- **Brillouin-Wigner theory:** the energy-dependent eigenproblem;
- **resolvent downfolding:** $(E-H_{QQ})^{-1}$;
- **Schur complement:** the block-linear-algebra viewpoint.

For example, applying block Gaussian elimination to $H-EI$ produces the Schur complement

<div class="math-display">
$$
H_{PP}-E-H_{PQ}(H_{QQ}-E)^{-1}H_{QP},
$$
</div>

which is $H_{\rm eff}(E)-E$. The names describe different interpretations of one block-matrix identity.

## 6. Exact but Energy Dependent

$H_{\rm eff}(E)$ is formally exact wherever the inverse exists, but it is not an ordinary linear eigenvalue problem because the Hamiltonian depends on the eigenvalue being sought:

<div class="math-display">
$$
H_{\rm eff}(E)|\psi_P\rangle=E|\psi_P\rangle.
$$
</div>

One may solve this nonlinear problem, evaluate it at a reference energy $E_{\rm ref}$, expand the resolvent when $P$ and $Q$ are well separated, or construct an energy-independent block diagonalization.

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

Schrieffer-Wolff organizes the same virtual physics by writing

<div class="math-display">
$$
H=H_0+V_{\rm d}+V_{\rm od},
$$
</div>

where $V_{\rm d}$ is block diagonal and $V_{\rm od}$ couples $P$ and $Q$. Seek $S^\dagger=-S$ such that

<div class="math-display">
$$
\widetilde H=e^SHe^{-S}
$$
</div>

is block diagonal order by order. The Baker-Campbell-Hausdorff expansion gives

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

The commutator contains the same $P\rightarrow Q\rightarrow P$ excursions as the Löwdin/Feshbach expansion. The conceptual distinction is:

- Löwdin/Feshbach **eliminates amplitudes in $Q$** and naturally produces an energy-dependent Hamiltonian.
- Schrieffer-Wolff **rotates the full Hamiltonian** so that $P$ and $Q$ decouple perturbatively, producing an energy-independent low-energy block.

Finite-order matrices can differ by gauge, but properly constructed theories reproduce the same low-energy observables to their claimed order.

## 8. A Familiar Example: Superexchange

For two sites with hopping $t$ and large repulsion $U$, singly occupied states form the low-energy sector while double occupation costs $U$. Integrating out virtual double occupation gives

<div class="math-display">
$$
H_{\rm spin}=J\,\mathbf S_1\cdot\mathbf S_2,
\qquad
J\simeq\frac{4t^2}{U}.
$$
</div>

The eliminated charge configurations survive as antiferromagnetic exchange. Likewise, eliminating a ligand produces

<div class="math-display">
$$
d_i\longrightarrow p\longrightarrow d_j,
\qquad
t_{dd}^{\rm eff}\sim
\frac{t_{dp}^2}{\epsilon_d-\epsilon_p}.
$$
</div>

Both are direct manifestations of the $-V^2/\Delta$ structure.

## 9. When Plain Projection Is Already Exact

Projection fails only when the retained space couples to its complement. If the range of $P$ is invariant under $H$,

<div class="math-display">
$$
QHP=0,
$$
</div>

then $H_{QP}=H_{PQ}=0$ for Hermitian $H$, so

<div class="math-display">
$$
H_{\rm eff}=V^\dagger H V
$$
</div>

is exact. Selecting exact eigenvectors gives

<div class="math-display">
$$
V^\dagger H V=\operatorname{diag}(E_1,\ldots,E_r).
$$
</div>

Yet exact eigenvectors can be a poor **modeling basis**: their gauge may vary irregularly with momentum and obscure stable layer, orbital, valley, or spin labels. A useful model should preserve interpretation as well as eigenvalues.

## 10. Wannierization: Subspace Selection Plus Gauge Choice

Wannier construction combines projection with a gauge choice. Starting from Bloch states $\lvert\psi_{n\mathbf k}\rangle$,

<div class="math-display">
$$
|w_{m\mathbf k}\rangle
=\sum_n|\psi_{n\mathbf k}\rangle U_{nm}(\mathbf k),
$$
</div>

followed by a Fourier transform to localized $\lvert w_{m\mathbf R}\rangle$. For isolated bands, $U(\mathbf k)$ is a unitary gauge transformation. For entangled bands, one first extracts a smooth lower-dimensional subspace. Wannierization can therefore contain both

<div class="math-display">
$$
\text{subspace selection}
+
\text{unitary gauge optimization}.
$$
</div>

This does not by itself add remote-band corrections. SVD/POD methods fit the same taxonomy: they choose $V$ from dominant snapshot directions and then use $V^\dagger H V$.

## 11. Nonorthogonal Bases and the Two Meanings of “Löwdin”

Localized atomic orbitals are often nonorthogonal. Their coefficients satisfy a generalized eigenproblem

<div class="math-display">
$$
Hc=ESc,
\qquad
S_{ij}=\langle\phi_i|\phi_j\rangle.
$$
</div>

Projection gives

<div class="math-display">
$$
H_{\rm red}=V^\dagger HV,
\qquad
S_{\rm red}=V^\dagger SV,
$$
</div>

and $H_{\rm red}c=ES_{\rm red}c$. Symmetric Löwdin orthogonalization defines

<div class="math-display">
$$
\widetilde V
=V(V^\dagger S V)^{-1/2},
\qquad
\widetilde V^\dagger S\widetilde V=I.
$$
</div>

The orthonormal Hamiltonian is $\widetilde V^\dagger H\widetilde V$. Two distinct procedures carry Löwdin's name:

- **Löwdin orthogonalization** removes the overlap metric $S$.
- **Löwdin partitioning** integrates out a complementary subspace.

The first changes the metric; the second constructs an effective Hamiltonian.

## 12. TAPW and Moiré Models as a Running Example

Large moiré supercells make the hierarchy visible: an atomistic basis may contain thousands of functions, while a chosen valley uses far fewer momentum components.

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

Here $\ell$, $i$, $\alpha$, and $\mathbf G$ label layer, cell, internal state, and selected moiré reciprocal vector.

Writing these basis states as columns of $X$, the first compression is

<div class="math-display">
$$
H_{\rm TAPW}=X^\dagger H_{\rm loc}X,
\qquad
S_{\rm TAPW}=X^\dagger S_{\rm loc}X.
$$
</div>

This is a physics-informed **projection** into a valley-centered basis. Nonorthogonal orbitals next require

<div class="math-display">
$$
H_{\rm TAPW}^{\rm orth}
=S_{\rm TAPW}^{-1/2}
H_{\rm TAPW}
S_{\rm TAPW}^{-1/2}.
$$
</div>

This is **Löwdin orthogonalization**, not remote-state elimination.

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

This is the genuine **Löwdin/Feshbach downfolding** step. A physical gauge and symmetry representation then give

<div class="math-display">
$$
H_{\rm continuum}(\mathbf k)
=\sum_j\theta_jY_j(\mathbf k),
$$
</div>

with symmetry-allowed $Y_j$.

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

## 13. What Can Be Generated by Downfolding?

Eliminated states can generate renormalized energies, longer-range hopping, exchange, spin-orbit coupling, multibody terms, and frequency dependence. Observables may also require transformation:

<div class="math-display">
$$
O_{\rm eff}=P e^S O e^{-S}P.
$$
</div>

Thus correct energies do not guarantee correct bare projected matrix elements.

## 14. When Is a Reduction Trustworthy?

A reduced model should come with diagnostics, not only a small matrix. Useful questions include:

1. Is the retained manifold separated from discarded states, with $\lVert H_{PQ}\rVert/\Delta$ small enough for the chosen order?
2. Does the target energy approach a pole of $(E-H_{QQ})^{-1}$?
3. Does the retained space overlap the target exact states and vary smoothly with momentum?
4. Are symmetry and topology represented consistently?
5. Were important induced, frequency-dependent, or transformed-observable terms omitted?
6. Are predictions stable under changes of $E_{\rm ref}$ and truncation order?

Near a crossing between retained and eliminated states, a small-denominator expansion is not controlled. The cure is often to enlarge $P$, not to compute more terms in a failing series.

## 15. A Compact Decision Guide

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

## 16. The Unified Picture

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

These methods answer three modeling questions:

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
