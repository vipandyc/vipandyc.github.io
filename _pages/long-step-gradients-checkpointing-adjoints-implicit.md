---
permalink: /blog/long-step-gradients-checkpointing-adjoints-implicit/
title: "How to Tackle Long-Step Gradients"
excerpt: "A methods note on what to do when direct backpropagation through many steps is too expensive: rematerialization, chunking, neural ODE adjoints, fixed-point implicit gradients, and argmin/envelope gradients."
author_profile: true
---

# How to Tackle Long-Step Gradients

Many modern objectives contain a long computation inside the loss. A model may be evaluated for hundreds of diffusion steps, a simulator may take many time steps, an optimizer may run inside a meta-objective, or a self-consistent solver may iterate until convergence.

The direct approach is to unroll the computation and apply reverse-mode automatic differentiation. For short computations this is usually the correct baseline. For long-step problems it can become memory-limited, numerically fragile, or mathematically misaligned with the object of interest. In many cases the finite trajectory is only a method for reaching an endpoint: a continuous flow, a fixed point, or an optimized value.

This note summarizes the main alternatives:

- checkpointing and rematerialization, when the unrolled computation is right but memory is the bottleneck;
- chunked gradients, when the long computation can be cut into manageable blocks;
- neural ODE-style adjoints, when the computation is better viewed as a continuous trajectory;
- implicit differentiation, when the final state is defined by a fixed point or root rather than by a particular iteration history;
- envelope-theorem gradients, when an inner variable is an exact or approximate argmin.

The central question is what defines the output. If the output is a finite program, differentiate that program, perhaps with checkpointing. If it is the solution of an equation, differentiate the equation. If it is the value of an optimized objective, use stationarity so the derivative of the optimizer trajectory does not appear.

## 1. The Direct Gradient and Why It Hurts

Write a long computation as

<div class="math-display">
$$
x_{k+1}=F_k(x_k,\theta),
\qquad k=0,\ldots,T-1,
\qquad
L(\theta)=\ell(x_T,\theta).
$$
</div>

Reverse-mode AD propagates an adjoint variable backward from the terminal loss. If

<div class="math-display">
$$
\lambda_T=\nabla_{x_T}\ell(x_T,\theta),
$$
</div>

then the reverse recursion is

<div class="math-display">
$$
\lambda_k
=
\left(\partial_{x_k}F_k(x_k,\theta)\right)^T\lambda_{k+1},
$$
</div>

and the parameter gradient accumulates contributions from every step:

<div class="math-display">
$$
\nabla_\theta L
=
\partial_\theta \ell(x_T,\theta)
+
\sum_{k=0}^{T-1}
\left(\partial_\theta F_k(x_k,\theta)\right)^T\lambda_{k+1}.
$$
</div>

The formula is straightforward; the storage is not. To compute the vector-Jacobian product at step $k$, the backward pass usually needs $x_k$ and local intermediates from $F_k$. Thus the memory cost grows roughly with the trajectory length.

There is also a modeling issue. The gradient of a long composition contains products of Jacobians, so some modes grow, some vanish, and some are dominated by discretization or solver details. If the finite path is merely a method for reaching an equilibrium, a root, or an optimum, differentiating the entire path may not be the most appropriate derivative.

The first diagnostic question is therefore:

**Is the full path part of the model, or is it only a procedure for reaching the final object?**

If the path itself defines the model, as in a recurrent network or a finite-step sampler, unrolled gradients are faithful. If the path is only a solver, implicit or envelope gradients are often more natural.

## 2. Checkpointing: Pay Compute to Save Memory

Gradient checkpointing, also called rematerialization, keeps the same exact unrolled gradient but changes what is stored. Instead of saving every intermediate activation, save only selected checkpoints. During the backward pass, recompute the missing forward states from the nearest checkpoint.

In the simplest picture, split the trajectory into $B$ blocks of length $m$, so $T=Bm$. Store

<div class="math-display">
$$
x_0,x_m,x_{2m},\ldots,x_T.
$$
</div>

When the backward pass needs states inside block $b$, recompute

<div class="math-display">
$$
x_{bm+1},x_{bm+2},\ldots,x_{(b+1)m}
$$
</div>

from the saved boundary state $x_{bm}$, then backpropagate through that block. Memory drops from storing $O(T)$ states to storing roughly $O(B+m)$ states: the checkpoints plus one active block. Compute increases because parts of the forward pass are run again.

This is appropriate when the finite unroll is part of the model definition. Examples include:

- a Transformer with many layers whose exact layerwise computation defines the model;
- an unrolled diffusion sampler trained through a fixed number of denoising steps;
- a differentiable simulator where the discretized trajectory is the object being optimized;
- a meta-learning inner loop where the number of inner steps is part of the algorithm.

Checkpointing does not change the gradient; it changes only the memory schedule.

### Choosing Checkpoints

Uniform chunks are the easiest policy. If each step costs about the same and has about the same activation size, choose a chunk size that fits in memory. A small chunk uses more memory but less recomputation; a large chunk uses less memory but more recomputation.

Recursive checkpointing algorithms can improve the memory-compute tradeoff when $T$ is large, but the principle is unchanged: store a sparse set of states and recompute missing intervals only when the backward pass reaches them.

## 3. Chunked Backpropagation and Truncated Gradients

The term "chunk" is used in two different ways.

The first meaning is exact chunking with recomputation. This is just checkpointing: each chunk is differentiated exactly, and adjoints are passed across chunk boundaries. The final gradient is the same as full backpropagation.

The second meaning is truncated backpropagation. Here one deliberately stops the gradient at chunk boundaries:

<div class="math-display">
$$
x_{(b+1)m}
=
F_{(b+1)m-1}\circ\cdots\circ F_{bm}(x_{bm},\theta),
\qquad
\text{but } x_{bm}\text{ is treated as constant.}
$$
</div>

This produces a biased gradient for the full $T$-step objective, but it can still be a useful optimization procedure. Recurrent networks, long-horizon control, online learning, and some simulation-calibration problems use truncated gradients because exact long-range credit assignment is noisy, unstable, or too expensive.

The distinction is important:

- exact chunking asks for the true derivative of the long computation;
- truncated chunking asks for a local training signal that may be good enough.

For scientific objectives, truncation should be treated as an approximation to the intended derivative. For representation learning or control, the biased signal may be acceptable if it improves the final trained behavior.

## 4. Neural ODE Adjoints

Consider an ODE with state $z(t)\in\mathbb R^n$ and parameters $\theta\in\mathbb R^p$:

<div class="math-display">
$$
\dot z(t)=f(z(t),t,\theta),\qquad z(0)=z_0,\qquad L(\theta)=\ell(z(T)).
$$
</div>

Assume first that $z_0$ does not depend on $\theta$ and that there is no running loss. Along the forward trajectory, define

<div class="math-display">
$$
f_z(t)=\partial_z f(z(t),t,\theta)\in\mathbb R^{n\times n},
\qquad
f_\theta(t)=\partial_\theta f(z(t),t,\theta)\in\mathbb R^{n\times p}.
$$
</div>

The most direct sensitivity is the matrix

<div class="math-display">
$$
S(t)=\frac{\partial z(t)}{\partial\theta}\in\mathbb R^{n\times p}.
$$
</div>

Differentiating the ODE with respect to $\theta$ gives the forward sensitivity equation

<div class="math-display">
$$
\dot S(t)=f_z(t)S(t)+f_\theta(t),\qquad S(0)=0.
$$
</div>

The terminal gradient is

<div class="math-display">
$$
\nabla_\theta L=S(T)^T\nabla_z\ell(z(T)).
$$
</div>

This formula is correct but expensive when $p$ is large, because it propagates the full $n\times p$ sensitivity matrix. The adjoint method avoids this by introducing a vector $a(t)\in\mathbb R^n$ with terminal condition

<div class="math-display">
$$
a(T)=\nabla_z\ell(z(T)).
$$
</div>

We choose the adjoint dynamics so that the dependence on $S(t)$ cancels when differentiating $a(t)^TS(t)$. Specifically, set

<div class="math-display">
$$
\dot a(t)=-f_z(t)^T a(t).
$$
</div>

This is the reason for the adjoint equation. Indeed,

<div class="math-display">
$$
\frac{d}{dt}\bigl(a^TS\bigr)
=\dot a^TS+a^T\dot S
=(-f_z^Ta)^TS+a^T(f_zS+f_\theta)
=a^Tf_\theta.
$$
</div>

Integrating from $0$ to $T$,

<div class="math-display">
$$
a(T)^TS(T)-a(0)^TS(0)=\int_0^T a(t)^Tf_\theta(t)\,dt.
$$
</div>

Since $S(0)=0$ and $a(T)=\nabla_z\ell(z(T))$, the left side is $\nabla_\theta L^T$. Transposing gives

<div class="math-display">
$$
\boxed{\nabla_\theta L=\int_0^T f_\theta(t)^T a(t)\,dt.}
$$
</div>

Thus the backward pass solves one $n$-dimensional adjoint equation instead of propagating the full $n\times p$ sensitivity matrix. If $\ell$ depends explicitly on $\theta$, add $\partial_\theta\ell(z(T),\theta)$. If $z_0=z_0(\theta)$, add $\left(\partial_\theta z_0\right)^Ta(0)$.

For implementation it is often clearer to integrate backward in the variable $\tau=T-t$. Then

<div class="math-display">
$$
\frac{dz}{d\tau}=-f,\qquad
\frac{da}{d\tau}=f_z^Ta,\qquad
\frac{dq}{d\tau}=f_\theta^Ta,\qquad
q(0)=0.
$$
</div>

After integrating from $\tau=0$ to $\tau=T$, $q(T)=\nabla_\theta L$. This form avoids the sign ambiguity that comes from writing a negative source term in $t$ while numerically stepping backward.

The continuous adjoint gives the gradient of the ideal ODE flow. Differentiating the numerical solver instead gives the gradient of the discrete algorithm. These agree only to the extent that the solver accurately represents the flow and its sensitivities. Adaptive, stiff, or chaotic dynamics often require checkpointed adjoints: save selected forward states, then integrate the adjoint backward between checkpoints.

## 5. Fixed Points and the Implicit Function Theorem

Now suppose the endpoint is important because it solves a self-consistency equation, not because it came from a particular number of iterations:

<div class="math-display">
$$
z_\ast = \Phi(z_\ast,\theta).
$$
</div>

This happens in equilibrium models, deep equilibrium networks, self-consistent field calculations, message passing to convergence, Bellman fixed points, and steady-state solvers. One can obtain $z_\ast$ by iteration,

<div class="math-display">
$$
z_{k+1}=\Phi(z_k,\theta),
$$
</div>

but the iteration is only a numerical route to the fixed point. If $z_\ast$ is the object being modeled, differentiating through all solver steps is often unnecessary.

Rewrite the fixed point as a root:

<div class="math-display">
$$
G(z,\theta)=z-\Phi(z,\theta)=0.
$$
</div>

Assuming the Jacobian $\partial_zG$ is invertible at the solution, the implicit function theorem says $z_\ast$ changes smoothly with $\theta$, and

<div class="math-display">
$$
\frac{dz_\ast}{d\theta}
=
-
\left(\partial_zG(z_\ast,\theta)\right)^{-1}
\partial_\theta G(z_\ast,\theta).
$$
</div>

Since $G=z-\Phi$, this becomes

<div class="math-display">
$$
\frac{dz_\ast}{d\theta}
=
\left(I-\partial_z\Phi(z_\ast,\theta)\right)^{-1}
\partial_\theta\Phi(z_\ast,\theta).
$$
</div>

For a loss $L(\theta)=\ell(z_\ast,\theta)$, we do not need to form this full Jacobian. Use an adjoint vector $v$ solving

<div class="math-display">
$$
\left(I-\partial_z\Phi(z_\ast,\theta)\right)^T v
=
\nabla_z\ell(z_\ast,\theta).
$$
</div>

Then

<div class="math-display">
$$
\nabla_\theta L
=
\partial_\theta\ell(z_\ast,\theta)
+
\left(\partial_\theta\Phi(z_\ast,\theta)\right)^T v.
$$
</div>

This replaces backpropagation through every solver iteration with one linear solve involving Jacobian-vector products. The Jacobian need not be materialized; Krylov methods, conjugate gradients in symmetric cases, or fixed-point iterations for the adjoint can use vector-Jacobian products evaluated at the converged solution.

### What Changed?

Unrolled differentiation depends on the finite iteration count:

<div class="math-display">
$$
z_T=\Phi_\theta^{(T)}(z_0).
$$
</div>

Implicit differentiation depends on the equation:

<div class="math-display">
$$
z_\ast=\Phi_\theta(z_\ast).
$$
</div>

If the solver is converged, the implicit gradient is independent of the particular algorithm used to reach the solution. Newton iteration, Anderson acceleration, simple iteration, or a warm start should give the same derivative up to numerical tolerances.

If the fixed point is not unique, the chosen branch matters. If $I-\partial_z\Phi$ is nearly singular, the gradient can be ill-conditioned. If the solver has not converged, the implicit gradient describes the equation solution rather than the actual finite iterate.

## 6. Root Solvers, KKT Conditions, and the Same Trick

Fixed points are just one form of implicit definition. More generally, suppose $z_\ast(\theta)$ solves

<div class="math-display">
$$
G(z_\ast,\theta)=0.
$$
</div>

Then the same formula applies:

<div class="math-display">
$$
\partial_zG\,\frac{dz_\ast}{d\theta}
+
\partial_\theta G
=
0.
$$
</div>

For constrained optimization, $G$ may be the KKT system. For a nonlinear PDE solve, it may be the discretized residual. For electronic-structure self-consistency, it may be a density or potential residual. The gradient asks how the solution changes with the parameters, not how a particular algorithm reached it.

This is why implicit differentiation is common in scientific machine learning: many codes already expose residuals, solvers, and preconditioners, which are precisely the ingredients needed to differentiate the defining equations.

## 7. Argmin Problems and the Envelope Theorem

Now consider an inner optimization problem:

<div class="math-display">
$$
v(\theta)
=
\min_y f(\theta,y),
\qquad
y_\ast(\theta)\in\arg\min_y f(\theta,y).
$$
</div>

The naive chain rule says

<div class="math-display">
$$
\frac{dv}{d\theta}
=
\partial_\theta f(\theta,y_\ast)
+
\partial_y f(\theta,y_\ast)\frac{dy_\ast}{d\theta}.
$$
</div>

But at an unconstrained interior optimum,

<div class="math-display">
$$
\partial_y f(\theta,y_\ast)=0.
$$
</div>

So the second term disappears:

<div class="math-display">
$$
\boxed{
\frac{dv}{d\theta}
=
\partial_\theta f(\theta,y_\ast)
}
$$
</div>

This is the envelope theorem in its most useful form. To differentiate the optimized value, you do not need the derivative of the optimizer $y_\ast(\theta)$. You only need to evaluate the partial derivative of the objective with respect to $\theta$ at the optimized inner variable.

This cancellation is often a major computational simplification. If the inner problem is a variational Monte Carlo optimization, a relaxed structure, a fitted auxiliary model, a free-energy variational bound, or a control/action minimization, differentiating through every optimizer step may be unnecessary when the outer objective is the optimized value itself.

### Relation to Implicit Differentiation

The optimizer satisfies the stationarity equation

<div class="math-display">
$$
\nabla_y f(\theta,y_\ast)=0.
$$
</div>

If we need the derivative of $y_\ast$ itself, implicit differentiation gives

<div class="math-display">
$$
\frac{dy_\ast}{d\theta}
=
-
\left(\nabla_{yy}^2 f(\theta,y_\ast)\right)^{-1}
\nabla_{y\theta}^2 f(\theta,y_\ast).
$$
</div>

For the optimized value $v(\theta)=f(\theta,y_\ast(\theta))$, this sensitivity is multiplied by the zero first-order condition. Thus:

- need the optimized value gradient: use envelope;
- need how the optimizer changes: use implicit differentiation;
- need gradient through a finite optimizer as an algorithm: unroll, probably with checkpointing.

### Constraints

For constrained problems, the same idea holds with the Lagrangian. Suppose

<div class="math-display">
$$
\min_y f(\theta,y)
\qquad
\text{subject to } c(\theta,y)=0.
$$
</div>

The Lagrangian is

<div class="math-display">
$$
\mathcal L(\theta,y,\lambda)
=
f(\theta,y)+\lambda^T c(\theta,y).
$$
</div>

At a regular optimum satisfying the KKT conditions, the value derivative is

<div class="math-display">
$$
\frac{dv}{d\theta}
=
\partial_\theta \mathcal L(\theta,y_\ast,\lambda_\ast).
$$
</div>

The multipliers carry the effect of active constraints. The derivative of $y_\ast$ still does not need to be explicitly computed for the value gradient.

## 8. Choosing the Right Gradient

The strategies above correspond to different mathematical definitions of the output.

Use **checkpointing** when the finite sequence of operations is the thing you truly want to differentiate, but storing all intermediates is too expensive. This preserves the exact unrolled gradient and trades memory for recomputation.

Use **truncation** when exact long-range credit assignment is less important than a cheap local training signal. This is an optimization choice, not an exact-gradient trick.

Use **continuous adjoints** when the mathematical object is an ODE flow. Be explicit about whether you want the gradient of the continuous model or the gradient of the discretized solver; for stiff, chaotic, or adaptive solves, checkpointed adjoints are often more reliable than pure trajectory reconstruction.

Use **implicit differentiation** when the output is defined by an equation such as

<div class="math-display">
$$
G(z_\ast,\theta)=0.
$$
</div>

Then the backward pass is a linear solve at the converged solution rather than a replay of every solver iteration.

Use the **envelope theorem** when the outer quantity is an optimized value,

<div class="math-display">
$$
v(\theta)=\min_y f(\theta,y).
$$
</div>

At a regular optimum, the derivative of $y_\ast(\theta)$ drops out of $dv/d\theta$; the inner optimizer matters only through the point it returns.

The main habit is to ask what defines the output: a finite program, a continuous flow, an equation, or an optimum. Long-step gradients become manageable once the derivative is matched to that object rather than to the entire computation history.
