---
permalink: /blog/td-nnqmc-lvm-afqmc/
title: "Time-Dependent NNQMC, LVM, and Canonical AFQMC"
excerpt: "A compact tutorial on real-time neural quantum Monte Carlo, linear subspace dynamics, and canonical auxiliary-field QMC."
author_profile: true
---

# Time-Dependent NNQMC, LVM, and Canonical AFQMC

This note is a method-first tutorial. The model can be a spin system, a fermion lattice Hamiltonian, or a continuum many-electron problem after choosing a basis. The common goal is to compute either real-time dynamics or the corresponding canonical thermal benchmark.

The three algorithms play different roles. Time-dependent NNQMC represents $\lvert\psi(t)\rangle$ by a neural state $\lvert\psi_{\theta(t)}\rangle$ and evolves $\theta(t)$ by a time-dependent variational principle. The Linear Variational Method (LVM) represents $\lvert\psi(t)\rangle=\sum_k c_k(t)\lvert\phi_k\rangle$ in a fixed subspace and solves projected dynamics. Canonical AFQMC samples finite-temperature equilibrium in a fixed particle-number or symmetry sector.

## Notation

Let $x$ be a computational-basis configuration and $\psi_\theta(x)=\langle x\vert\psi_\theta\rangle$. Neural quantum Monte Carlo samples $p_\theta(x)=\lvert\psi_\theta(x)\rvert^2/\sum_y\lvert\psi_\theta(y)\rvert^2$.

For any operator $A$, the local estimator is $A_{\rm loc}(x)=\langle x\vert A\vert\psi_\theta\rangle/\psi_\theta(x)=\sum_y A_{xy}\psi_\theta(y)/\psi_\theta(x)$. Thus $\langle A\rangle=E_{x\sim p_\theta}[A_{\rm loc}(x)]$.

The practical primitive is simple: for each sampled $x$, enumerate all connected configurations $y$ with $A_{xy}\ne0$, compute $\psi_\theta(y)/\psi_\theta(x)$, multiply by $A_{xy}$, and sum. For fermions, this is where operator-ordering signs enter.

## 1. Time-Dependent NNQMC

Exact real-time evolution obeys $d\lvert\psi(t)\rangle/dt=-iH(t)\lvert\psi(t)\rangle$. TD-NNQMC restricts the state to a variational manifold $\lvert\psi_{\theta(t)}\rangle$ and chooses $\dot\theta$ so that $\sum_j\dot\theta_j\lvert\partial_j\psi_\theta\rangle$ is as close as possible to $-iH\lvert\psi_\theta\rangle$.

### Log-Derivative Geometry

Define logarithmic derivatives $O_j(x)=\partial_{\theta_j}\log\psi_\theta(x)$. Then $\partial_{\theta_j}\psi_\theta(x)=O_j(x)\psi_\theta(x)$. Remove norm/phase redundancy by centering: $\Delta O_j(x)=O_j(x)-E[O_j]$ and $\Delta E(x)=E_{\rm loc}(x)-E[E_{\rm loc}]$.

The TDVP equations are $S\dot\theta=-iF$, with $S_{ij}=E[\Delta O_i^\ast\Delta O_j]$ and $F_i=E[\Delta O_i^\ast E_{\rm loc}]$. For real parameters, split complex quantities into real and imaginary parts; many implementations use ${\rm Re}(S)\dot\theta={\rm Im}(F)$, up to sign conventions.

Equivalently, $\dot\theta$ minimizes the residual $r(x)=\sum_j\dot\theta_j\Delta O_j(x)+i\Delta E(x)$ in mean square. This residual is the most useful interpretation: TDVP projects the Schrodinger vector field onto the neural tangent space.

### Monte Carlo Estimators

Draw $M$ samples $x_a\sim p_\theta$. Estimate $\bar O_j=M^{-1}\sum_a O_j(x_a)$ and $\bar E=M^{-1}\sum_a E_{\rm loc}(x_a)$. Then $S_{ij}\approx M^{-1}\sum_a (O_i(x_a)-\bar O_i)^\ast(O_j(x_a)-\bar O_j)$ and $F_i\approx M^{-1}\sum_a (O_i(x_a)-\bar O_i)^\ast(E_{\rm loc}(x_a)-\bar E)$.

Directly forming and inverting $S$ is expensive because $S$ is $P\times P$, where $P$ is the number of neural-network parameters.

### Sample-Space Solve

Let $Y_{aj}=(O_j(x_a)-\bar O_j)/\sqrt M$ and $e_a=(E_{\rm loc}(x_a)-\bar E)/\sqrt M$. Then $S=Y^\dagger Y$ and $F=Y^\dagger e$.

Instead of solving in parameter space, use $(Y^\dagger Y+\lambda I)^{-1}Y^\dagger=Y^\dagger(YY^\dagger+\lambda I)^{-1}$. The TDVP velocity becomes $\dot\theta=-iY^\dagger(YY^\dagger+\lambda I)^{-1}e$.

This is the key computational trick. The matrix $YY^\dagger$ is $M\times M$, the empirical neural tangent kernel on the Monte Carlo batch. Usually $M\ll P$, so this solve is feasible even for large networks.

Regularization can be a diagonal shift $\lambda I$ or a spectral cutoff: diagonalize $YY^\dagger$, discard eigenvalues smaller than $r_{\rm cond}\lambda_{\max}$, and invert only the retained modes.

### Time Integration

TDVP gives an ODE $\dot\theta=f(\theta,t)$. A robust low-cost integrator is Heun:

```text
k1 = f(theta_n, t_n)
theta_pred = theta_n + dt*k1
k2 = f(theta_pred, t_n + dt)
theta_{n+1} = theta_n + dt*(k1 + k2)/2
```

For a time-dependent Hamiltonian, a common midpoint variant holds $H$ fixed at $H(t_n+dt/2)$ during the step:

```text
tH = t_n + dt/2
k1 = f(theta_n, tH)
theta_pred = theta_n + dt*k1
k2 = f(theta_pred, tH)
theta_{n+1} = theta_n + dt*(k1 + k2)/2
```

Choose $dt$ so the largest local energy scale times $dt$ is small. In a static Hamiltonian, energy drift is a direct check of time-step and variational error.

### Sampling and Measurements

Use a sampler that stays in the intended Hilbert-space sector. For spin systems, this may mean magnetization-preserving spin exchanges. For fermions, propose particle moves from occupied to empty orbitals while preserving $N_\uparrow$ and $N_\downarrow$.

At each measurement time, sample from $\lvert\psi_\theta\rvert^2$, evaluate $A_{\rm loc}(x)$, and average. For Hermitian $A$, keep the real part but monitor the imaginary part as a noise diagnostic.

The TDVP residual should be estimated on an independent validation batch: $L_{\rm val}=E_{\rm val}[\lvert\sum_j\dot\theta_j\Delta O_j(x)+i\Delta E(x)\rvert^2]$. Using the same batch as the linear solve can underestimate the error because the solve may fit sampling noise.

### TD-NNQMC Algorithm

```text
input: logpsi(theta,x), Hamiltonian connections, theta0, time grid, sector-preserving sampler

theta = theta0
for each time step:
  sample x_a from |psi_theta(x)|^2
  compute E_loc(x_a)
  compute O_j(x_a) or JVP/VJP equivalents
  center O_j and E_loc
  build K = Y Y^dagger
  solve alpha = (K + lambda I)^(-1) e
  compute dot_theta = -i Y^dagger alpha
  advance theta with Heun or RK4
  measure observables and validation residual on fresh samples
  save selected snapshots theta(t)
```

For large networks, avoid explicitly storing the full Jacobian if possible. The solve only needs $YY^\dagger$ and $Y^\dagger\alpha$, which can be obtained through vector-Jacobian and Jacobian-vector products.

## 2. Linear Variational Method

LVM approximates dynamics in a fixed basis $\lvert\phi_k\rangle$: $\lvert\psi(t)\rangle=\sum_{k=1}^{M_b}c_k(t)\lvert\phi_k\rangle$. The basis can be TD-NNQMC snapshots, Krylov vectors, Lanczos vectors, Chebyshev vectors, or separately optimized variational states.

Define the overlap matrix $S_{ij}=\langle\phi_i\vert\phi_j\rangle$ and projected Hamiltonian $H_{ij}=\langle\phi_i\vert H\vert\phi_j\rangle$. Projecting Schrodinger evolution gives $S\dot c=-iHc$. In an orthonormal basis this reduces to $\dot c=-iHc$, but neural snapshots are generally nonorthogonal.

### Matrix Elements by Monte Carlo

For neural basis states, choose a sampling distribution with support over all basis states, for example $\Pi(x)=\sum_k w_k\lvert\phi_k(x)\rvert^2$ with $w_k>0$. Then $S_{ij}=Z_\Pi E_\Pi[\phi_i^\ast(x)\phi_j(x)/\Pi(x)]$.

For an operator $A$, define the transition local estimator $A_{\rm loc}^{(j)}(x)=\langle x\vert A\vert\phi_j\rangle/\phi_j(x)$. Then $A_{ij}=Z_\Pi E_\Pi[\phi_i^\ast(x)\phi_j(x)A_{\rm loc}^{(j)}(x)/\Pi(x)]$.

Use the same logic for $H_{ij}$. After Monte Carlo estimation, symmetrize Hermitian matrices: $S\leftarrow(S+S^\dagger)/2$, $H\leftarrow(H+H^\dagger)/2$, and $A\leftarrow(A+A^\dagger)/2$.

### Orthonormalization

Do not invert $S$ blindly. Diagonalize $S=UsU^\dagger$, discard directions with $s_\alpha<\epsilon_Ss_{\max}$, and define $B=U_{\rm kept}s_{\rm kept}^{-1/2}$.

The retained orthonormal basis is $\lvert\chi_\alpha\rangle=\sum_i B_{i\alpha}\lvert\phi_i\rangle$. Transform matrices as $H_o=B^\dagger HB$ and $A_o=B^\dagger AB$.

If $\lvert\psi\rangle=\sum_i c_i\lvert\phi_i\rangle$, its coefficients in the orthonormal retained basis are $d=B^\dagger Sc$. Conversely, within the retained subspace, $c=Bd$.

### Finite-Time and Infinite-Time Dynamics

For time-independent $H$, evolve $d(t)=e^{-iH_ot}d(0)$. Observables are $\langle A(t)\rangle=d^\dagger(t)A_od(t)/d^\dagger(t)d(t)$.

For the infinite-time average, diagonalize $H_ou_\alpha=E_\alpha u_\alpha$ and expand $d(0)=\sum_\alpha a_\alpha u_\alpha$. If there are no degeneracies, $\bar A_\infty=\sum_\alpha \lvert a_\alpha\rvert^2u_\alpha^\dagger A_ou_\alpha$. With degeneracies, keep all pairs satisfying $E_\alpha=E_\beta$: $\bar A_\infty=\sum_{\alpha,\beta:E_\alpha=E_\beta}a_\alpha^\ast a_\beta u_\alpha^\dagger A_ou_\beta$.

In finite precision, group energies using a tolerance. The tolerance should exceed numerical diagonalization noise but not merge physically distinct frequencies.

### LVM Algorithm

```text
input: basis states phi_k, Hamiltonian connections, observable A, initial state

choose Pi(x) = sum_k w_k |phi_k(x)|^2
sample x_a from Pi
evaluate phi_k(x_a) for all k
estimate S_ij, H_ij, and A_ij by reweighting
Hermitize S, H, and A
diagonalize S and discard small-overlap directions
build B = U_kept s_kept^(-1/2)
transform H_o = B^dagger H B and A_o = B^dagger A B
project the initial state into the retained basis
evolve d(t) or diagonalize H_o for infinite-time averages
repeat with larger bases and independent samples
```

For dynamics generated from time snapshots, a natural sequence is $\lvert\phi_k\rangle\approx(e^{-iH\Delta t})^k\lvert\psi_0\rangle$. Increasing the final snapshot time or the number of basis states controls the subspace approximation.

## 3. Canonical AFQMC

AFQMC is an imaginary-time finite-temperature method. Grand-canonical AFQMC samples ${\rm Tr}\,e^{-\beta(H-\mu N)}$, while canonical AFQMC samples fixed-particle-number traces such as ${\rm Tr}^{(N)}e^{-\beta H}$, or fixed up/down particle-number sectors. Canonical sampling is the right benchmark when the real-time dynamics conserves particle number or magnetization.

### BSS Structure

Write the Hamiltonian as a one-body part plus interactions that can be Hubbard-Stratonovich decoupled. Discretize $\beta=L_\tau\Delta\tau$ and use a symmetric Trotter step. After HS decoupling, each time slice is a one-body propagator $B_\ell(s_\ell)$, and a full auxiliary-field configuration $s$ gives $U_s=B_{L_\tau}(s_{L_\tau})\cdots B_1(s_1)$.

For one species, the grand-canonical field weight is $W_{\rm GC}(s)=\det(I+U_s)$. For spin species, multiply the determinants over spin sectors. If $W$ is not positive, sample $\lvert W\rvert$ and reweight by the sign or phase.

### Exact Canonical Projection

Introduce a fugacity $z$. For fixed fields, $\det(I+zU_s)=\sum_N z^N W_{N}(s)$. The canonical weight is the coefficient $W_{N}(s)=[z^N]\det(I+zU_s)$.

For two spin sectors, $W_{N_\uparrow,N_\downarrow}(s)=[z_\uparrow^{N_\uparrow}]\det(I+z_\uparrow U_{s,\uparrow})[z_\downarrow^{N_\downarrow}]\det(I+z_\downarrow U_{s,\downarrow})$.

One way to compute the coefficient is Fourier projection: $W_{N}(s)=N_{\phi}^{-1}\sum_{m=0}^{N_{\phi}-1}e^{-iN\phi_m}\det(I+e^{i\phi_m}U_s)$, with $\phi_m=2\pi m/N_{\phi}$ and $N_{\phi}\ge N_{\rm orb}+1$ for exact projection in exact arithmetic.

Another way is to use eigenvalues $\lambda_a$ of $U_s$: $\det(I+zU_s)=\prod_a(1+z\lambda_a)$. The coefficient of $z^N$ is the degree-$N$ elementary symmetric polynomial in $\{\lambda_a\}$, computed recursively by updating coefficients in descending order.

At low temperature, $U_s$ is ill-conditioned, so practical codes use QR/SVD stabilization and stabilized coefficient extraction.

### Canonical Green's Functions

At fugacity $z$, the grand-canonical one-body density matrix for fixed fields is $\rho(z)=zU_s(I+zU_s)^{-1}$. The canonical one-body density matrix is obtained by projecting the numerator and denominator separately:

$\langle c_i^\dagger c_j\rangle_{s,N}=\frac{[z^N]\det(I+zU_s)\rho(z)_{ji}}{[z^N]\det(I+zU_s)}$.

Using Fourier projection, this is the weighted sum over phases $z=e^{i\phi_m}$. Two-body canonical estimators can be derived by adding sources and differentiating the canonical generating function, or by library-provided canonical measurement routines. Do not blindly use grand-canonical Wick contractions after fixing $N$.

### Penalty-Enforced Canonical Sampling

A practical alternative is to simulate an augmented Hamiltonian $H_{\rm pen}=H+\lambda_N(\hat N-N_0)^2+\lambda_S(\hat S_z-S_{z,0})^2$. Large penalties exponentially suppress unwanted sectors while allowing reuse of grand-canonical AFQMC machinery.

The procedure is empirical but controlled: increase $\lambda_N$ and $\lambda_S$ until $\langle(\hat N-N_0)^2\rangle$ and $\langle(\hat S_z-S_{z,0})^2\rangle$ vanish within error bars. Then measure observables in the effectively canonical ensemble.

### Updates, Stabilization, and Measurements

A local update proposes changing one auxiliary field. The acceptance ratio is the ratio of canonical weights, $R=W_{N}(s_{\rm new})/W_{N}(s)$, or the corresponding penalty-Hamiltonian grand-canonical ratio. Accept with probability $\min(1,\lvert R\rvert)$ and reweight signs if needed.

The product $U_s=B_{L_\tau}\cdots B_1$ must be stabilized by QR or SVD factorizations. Store products in factorized form, track log determinants, and periodically recompute Green's functions from stabilized matrices.

Measurements use $\langle A\rangle^{(N)}=\sum_s W^{(N)}(s)A^{(N)}(s)/\sum_s W^{(N)}(s)$. With signs, accumulate the reweighted ratio $\langle A\rangle^{(N)}=\langle A^{(N)}(s)\,{\rm sgn}^{(N)}(s)\rangle/\langle{\rm sgn}^{(N)}(s)\rangle$. Use blocking or jackknife because Markov-chain samples are correlated.

### Canonical AFQMC Algorithm

```text
input: K, interaction channels, target N or (N_up,N_down), beta, Delta tau, HS decomposition

initialize auxiliary fields s
for each Monte Carlo sweep:
  propose local or global field updates
  compute canonical weight ratio by coefficient projection
  accept/reject using min(1, |R|)
  periodically stabilize B_L ... B_1 products
  after warmup:
    compute canonical Green's function
    measure observables
    accumulate sign-reweighted statistics

postprocess:
  blocking or jackknife errors
  Delta tau extrapolation
  beta/temperature interpolation
```

For a thermal benchmark to a real-time calculation, compute the thermal energy $E_{\rm th}(T)$ over a temperature grid, fit or interpolate it, solve $E_{\rm th}(T_{\rm eff})=E_{\rm dyn}$, and evaluate the observable at $T_{\rm eff}$ in the same canonical sector.

## How They Fit Together

A common pipeline is:

```text
choose conserved sector
prepare initial state
run TD-NNQMC and save snapshots
build an LVM basis from snapshots
extract finite-time or infinite-time observables
run canonical AFQMC in the same sector
match temperature by conserved energy
compare dynamical and thermal observables
```

TD-NNQMC and LVM approximate unitary real-time dynamics. AFQMC samples imaginary-time equilibrium. AFQMC does not show that a system thermalizes; it gives the value that real-time dynamics should approach if thermalization holds.

## References

- Time-dependent VMC for correlated electrons: [Ido, Ohgoe, and Imada](https://arxiv.org/abs/1507.00274)
- TDVP and variational quantum simulation: [Yuan et al.](https://arxiv.org/abs/1812.08767)
- Large-scale stochastic reconfiguration identity: [Rende et al.](https://arxiv.org/abs/2310.05715)
- Time-dependent Neural Galerkin / LVM: [Sinibaldi et al.](https://arxiv.org/abs/2412.11778)
- Canonical finite-temperature AFQMC: [Wang, Assaad, and Parisen Toldin](https://arxiv.org/abs/1706.01874)
- BSS AFQMC: [Blankenbecler, Scalapino, and Sugar](https://doi.org/10.1103/PhysRevD.24.2278)
- ALF AFQMC documentation: [Assaad et al.](https://arxiv.org/abs/2012.11914)
