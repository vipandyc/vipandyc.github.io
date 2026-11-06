---
permalink: /blog/finite-group-irreps-characters-orthogonality/
title: "Finite Groups: Irreps, Characters, and Orthogonality Cheat Sheet"
excerpt: "The main identities for complex finite-group representations: dimensions, character tables, row and column orthogonality, multiplicities, symmetry projectors, and tensor products."
author_profile: true
---

# Finite Groups: Irreps, Characters, and Orthogonality Cheat Sheet

This is a working sheet for the equations one repeatedly needs when decomposing a representation or checking a character table. **Convention:** $G$ is finite, all representations are finite-dimensional over $\mathbb C$, and one representative of each inequivalent irreducible representation (irrep) is labeled by $\alpha$. Over $\mathbb C$, averaging an inner product makes every representation unitary, and every representation decomposes into irreps.

Write $n=|G|$, $D^{(\alpha)}(g)$ for the $d_\alpha\times d_\alpha$ matrix of irrep $\alpha$, and $\chi_\alpha(g)=\operatorname{tr}D^{(\alpha)}(g)$. Let $C_j$ be a conjugacy class with representative $g_j$ and size $c_j$. Every sum over $\alpha$ below includes each inequivalent irrep once.

## 1. Character identities

Traces do not change under similarity, so characters are constant on conjugacy classes. Unitarity and the rules for traces give

<div class="math-display">
$$
\begin{aligned}
\chi_\alpha(e)&=d_\alpha,
&\chi_\alpha(hgh^{-1})&=\chi_\alpha(g),
&\chi_\alpha(g^{-1})&=\overline{\chi_\alpha(g)},\\
\chi_{V\oplus W}(g)&=\chi_V(g)+\chi_W(g),
&\chi_{V\otimes W}(g)&=\chi_V(g)\chi_W(g),
&\chi_{V^*}(g)&=\overline{\chi_V(g)}.
\end{aligned}
$$
</div>

In particular, $|\chi_\alpha(g)|\leq d_\alpha$: the eigenvalues of a finite-order unitary matrix lie on the unit circle. The last identity describes the dual representation $V^*$, whose matrices are $D_V(g^{-1})^{\mathsf T}$.

## 2. The source of orthogonality: Schur's lemma

For irreps $\alpha,\beta$ and a linear map $X:V_\beta\to V_\alpha$, average it over the group:

<div class="math-display">
$$
\mathcal A(X)=\frac1n\sum_{g\in G}
D^{(\alpha)}(g)X D^{(\beta)}(g)^{-1}
=\begin{cases}
(\operatorname{tr}X/d_\alpha)I_{d_\alpha},&\alpha=\beta,\\
0,&\alpha\ne\beta.
\end{cases}
$$
</div>

The average intertwines the two group actions. Schur's lemma therefore makes it zero between inequivalent irreps and a scalar identity within one irrep; taking its trace fixes that scalar. Choosing $X$ to have just one nonzero entry yields **matrix-element orthogonality** in unitary bases:

<div class="math-display">
$$
\frac1n\sum_{g\in G}
D^{(\alpha)}(g)_{ij}
\overline{D^{(\beta)}(g)_{k\ell}}
=\frac{\delta_{\alpha\beta}\delta_{ik}\delta_{j\ell}}{d_\alpha}.
$$
</div>

Summing the diagonal entries gives the character identities below. Matrix-element orthogonality is stronger: it resolves individual rows and columns of the representation matrices, not only their traces.

## 3. Row and column orthogonality of the character table

For class functions define $\langle f,h\rangle_G=n^{-1}\sum_{g\in G}\overline{f(g)}h(g)$. Taking traces in the preceding identity gives **row orthogonality**:

<div class="math-display">
$$
\langle\chi_\alpha,\chi_\beta\rangle_G
=\frac1n\sum_{g\in G}\overline{\chi_\alpha(g)}\chi_\beta(g)
=\frac1n\sum_j c_j\overline{\chi_\alpha(g_j)}\chi_\beta(g_j)
=\delta_{\alpha\beta}.
$$
</div>

The class-size factor $c_j$ matters: a character table has one column per *class*, but the inner product sums over *elements*. Since $\overline{\chi_\alpha(g)}=\chi_\alpha(g^{-1})$, the inverse-character version of this formula is identical. There are as many irreps as conjugacy classes: the center of $\mathbb C[G]$ has a basis of class sums, while $\mathbb C[G]\cong\bigoplus_\alpha\operatorname{End}(V_\alpha)$ gives one central scalar block per irrep.

Thus the weighted character table $U_{\alpha j}=\sqrt{c_j/n}\,\chi_\alpha(g_j)$ is square and unitary. Its column orthogonality is

<div class="math-display">
$$
\sum_\alpha\overline{\chi_\alpha(g_j)}\chi_\alpha(g_k)
=\frac{n}{c_j}\,\delta_{jk}
=|C_G(g_j)|\,\delta_{jk},
$$
</div>

where $C_G(g_j)=\{x\in G:xg_j=g_jx\}$ is the centralizer. Equivalently, the sum is zero for elements in different conjugacy classes and equals the centralizer size for elements in the same class. Rows distinguish irreps; columns distinguish conjugacy classes.

## 4. Dimensions, the regular representation, and multiplicities

The regular representation acts on basis vectors labeled by group elements. A nonidentity element permutes that basis with no fixed *basis vector*, whereas the identity fixes all $n$ basis vectors. Hence $\chi_{\rm reg}(e)=n$ and $\chi_{\rm reg}(g)=0$ for $g\ne e$. Row orthogonality then gives the multiplicity of irrep $\alpha$ in the regular representation as $d_\alpha$:

<div class="math-display">
$$
\mathbb C[G]\cong\bigoplus_\alpha d_\alpha V_\alpha,
\qquad
\sum_\alpha d_\alpha^2=n,
\qquad
\sum_\alpha d_\alpha\chi_\alpha(g)=n\delta_{g,e}.
$$
</div>

For any representation $V\cong\bigoplus_\alpha m_\alpha V_\alpha$, characters add, and row orthogonality extracts each nonnegative integer multiplicity:

<div class="math-display">
$$
m_\alpha=\langle\chi_\alpha,\chi_V\rangle_G
=\frac1n\sum_j c_j\overline{\chi_\alpha(g_j)}\chi_V(g_j),
\qquad
\dim V=\sum_\alpha m_\alpha d_\alpha.
$$
</div>

Useful checks follow immediately: $V$ is irreducible exactly when $\langle\chi_V,\chi_V\rangle_G=1$, and two complex representations are isomorphic exactly when their characters agree. More generally, $\langle\chi_V,\chi_W\rangle_G=\dim\operatorname{Hom}_G(V,W)=\sum_\alpha m_\alpha(V)m_\alpha(W)$.

## 5. Projectors that extract symmetry sectors

If $D(g)$ is the representation acting on your space, group averaging projects onto the invariant (trivial-irrep) subspace:

<div class="math-display">
$$
P_{\rm inv}=\frac1n\sum_{g\in G}D(g),
\qquad
\dim V^G=\operatorname{tr}P_{\rm inv}
=\frac1n\sum_{g\in G}\chi_V(g).
$$
</div>

For an irrep, the character sum is therefore $\sum_g\chi_\alpha(g)=n$ for the trivial irrep and $0$ for every other irrep.

To keep **all copies** of a chosen irrep $\alpha$, use its character projector:

<div class="math-display">
$$
P_\alpha=\frac{d_\alpha}{n}
\sum_{g\in G}\overline{\chi_\alpha(g)}D(g),
\qquad
P_\alpha P_\beta=\delta_{\alpha\beta}P_\alpha,
\qquad
\sum_\alpha P_\alpha=I.
$$
</div>

Why does the coefficient work? On an irreducible block $V_\beta$, the sum commutes with every $D^{(\beta)}(h)$, so it is a scalar by Schur's lemma. Its trace is $d_\alpha\langle\chi_\alpha,\chi_\beta\rangle_G=d_\alpha\delta_{\alpha\beta}$, making the scalar $1$ for $\beta=\alpha$ and $0$ otherwise. Thus $\operatorname{rank}P_\alpha=m_\alpha d_\alpha$. It selects an entire *isotypic sector*, not one particular copy when $m_\alpha>1$.

## 6. Building symmetry-adapted basis functions

Suppose $G$ acts on functions by $(D(g)f)(x)=f(g^{-1}x)$. Given a trial function $f$, the character projector above produces $P_\alpha f$, its component in symmetry sector $\alpha$. It may be zero; otherwise normalize the result. For a multi-dimensional irrep, characters select the whole sector but do not distinguish its partner functions.

If the matrices $D^{(\alpha)}(g)$ are known, **matrix-unit operators** isolate those partners:

<div class="math-display">
$$
\begin{aligned}
E_{ij}^{(\alpha)}
&=\frac{d_\alpha}{n}\sum_{g\in G}
\overline{D^{(\alpha)}(g)_{ij}}D(g),\\
E_{ij}^{(\alpha)}E_{k\ell}^{(\beta)}
&=\delta_{\alpha\beta}\delta_{jk}E_{i\ell}^{(\alpha)}.
\end{aligned}
$$
</div>

Matrix-element orthogonality makes $E_{ij}^{(\alpha)}f_k=\delta_{jk}f_i$ on each copy of irrep $\alpha$, and $\sum_i E_{ii}^{(\alpha)}=P_\alpha$. The diagonal $E_{ii}$ are projectors; an off-diagonal $E_{ij}$ transfers partner $j$ to partner $i$. Apply these to trial functions, discard zero or dependent results, and normalize the survivors to obtain a symmetry-adapted basis.

For the two-element group $G=\{e,s\}$ acting by reflection, $D(s)f(x)=f(-x)$, the construction is simply

<div class="math-display">
$$
\begin{aligned}
P_\pm&=\tfrac12(I\pm D(s)),\\
f_\pm(x)&=(P_\pm f)(x)=\tfrac12\bigl[f(x)\pm f(-x)\bigr].
\end{aligned}
$$
</div>

These are the even and odd parts of $f$. With a reflection-invariant inner product they are orthogonal, since $P_+P_-=0$ and $P_\pm^\dagger=P_\pm$. If an operator $H$ respects the symmetry, $[H,D(g)]=0$ for every $g$, then $P_\alpha H P_\beta=0$ for $\alpha\ne\beta$. In a symmetry-adapted basis, $H$ splits into blocks $\bigoplus_\alpha(h_\alpha\otimes I_{d_\alpha})$; $h_\alpha$ can still mix different copies of the same irrep.

## 7. Products and symmetric or antisymmetric states

The tensor-product character is a product, so the number of copies of $V_\gamma$ in $V_\alpha\otimes V_\beta$ is

<div class="math-display">
$$
N_{\alpha\beta}^{\gamma}
=\langle\chi_\gamma,\chi_\alpha\chi_\beta\rangle_G
=\frac1n\sum_j c_j
\overline{\chi_\gamma(g_j)}\chi_\alpha(g_j)\chi_\beta(g_j).
$$
</div>

For two identical factors, the swap operator separates symmetric and antisymmetric tensors. Its trace on $D(g)\otimes D(g)$ is $\chi_V(g^2)$, giving

<div class="math-display">
$$
\chi_{\operatorname{Sym}^2 V}(g)
=\frac{\chi_V(g)^2+\chi_V(g^2)}2,
\qquad
\chi_{\wedge^2 V}(g)
=\frac{\chi_V(g)^2-\chi_V(g^2)}2.
$$
</div>

## 8. Worked check: $S_3$

The six permutations of three objects form three conjugacy classes. The trivial, sign, and standard representations give this character table:

| Class representative | $e$ | $(12)$ | $(123)$ |
|:--|--:|--:|--:|
| Class size $c_j$ | $1$ | $3$ | $2$ |
| Trivial, $d=1$ | $1$ | $1$ | $1$ |
| Sign, $d=1$ | $1$ | $-1$ | $1$ |
| Standard, $d=2$ | $2$ | $0$ | $-1$ |

The three-dimensional permutation representation has character $(3,1,0)$: its trace counts fixed objects. Subtracting the constant invariant vector gives the standard character $(2,0,-1)$. If $f_1,f_2,f_3$ are basis functions permuted by $S_3$, the character projectors make this decomposition concrete:

<div class="math-display">
$$
P_{\rm triv}f_1=\frac{f_1+f_2+f_3}{3},\qquad
P_{\rm std}f_1=\frac{2f_1-f_2-f_3}{3},\qquad
P_{\rm sign}f_1=0.
$$
</div>

The first function is symmetric; the second lies in the two-dimensional sum-zero standard sector, where $f_2-f_3$ supplies another independent basis function. The character-table checks are then quick:

<div class="math-display">
$$
1^2+1^2+2^2=6,
\qquad
\frac{1(2)^2+3(0)^2+2(-1)^2}{6}=1,
\qquad
\frac{1(2)+3(0)+2(-1)}{6}=0.
$$
</div>

These are the dimension sum, the standard row's norm, and its inner product with the trivial row. Column orthogonality gives $1^2+(-1)^2+0^2=2=6/3$ for the transposition column. Finally, the pointwise square $(2,0,-1)^2=(4,0,1)$ equals the sum of the three rows, so $V_{\rm std}\otimes V_{\rm std}\cong V_{\rm triv}\oplus V_{\rm sign}\oplus V_{\rm std}$.

For an abelian group every irrep is one-dimensional. For $G=\mathbb Z_m$, the characters $\chi_k(\ell)=e^{2\pi i k\ell/m}$ turn row orthogonality into the discrete Fourier identity $m^{-1}\sum_{\ell=0}^{m-1}e^{2\pi i(k-k')\ell/m}=\delta_{kk'}$.

## References

- [Pavel Etingof, *Introduction to Representation Theory*, Chapter 3, MIT OpenCourseWare](https://ocw.mit.edu/courses/18-712-introduction-to-representation-theory-fall-2010/resources/mit18_712f10_ch3/).
- [MIT 6.734J, *Applications of Group Theory to the Physics of Solids*, Chapter 4: Basis Functions](https://web.mit.edu/course/6/6.734j/www/group-full02.pdf).
- [Katherine Christianson, *Representation Theory Notes*, Columbia University](https://www.math.columbia.edu/~khovanov/finite2016/RepTheoryFinite2.pdf).
