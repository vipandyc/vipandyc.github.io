---
permalink: /blog/long-step-gradients-checkpointing-adjoints-implicit/
title: "Long-Step Gradients: Checkpointing, Adjoints, Implicit Differentiation, and Envelope Theorems"
excerpt: "A methods note on what to do when direct backpropagation through many steps is too expensive: rematerialization, chunking, neural ODE adjoints, fixed-point implicit gradients, and argmin/envelope gradients."
author_profile: true
---

# Long-Step Gradients: Checkpointing, Adjoints, Implicit Differentiation, and Envelope Theorems

Many modern algorithms are secretly long computations wrapped inside a loss. A neural network is applied for hundreds of diffusion steps. A simulator takes many time steps. An optimizer runs inside a meta-objective. A fixed-point solver iterates until self-consistency. A variational inner problem is minimized before the outer parameters are updated.

The naive answer is always the same: unroll everything and let reverse-mode automatic differentiation accumulate the gradient. This is the cleanest mental model, and for short computations it is usually the right engineering choice. But for long-step problems, direct backpropagation stores too much intermediate state, differentiates through too many fragile numerical details, or spends most of its time on a path whose endpoint is the only thing that matters.

This note is a map of the common escape routes:

- checkpointing and rematerialization, when the unrolled computation is right but memory is the bottleneck;
- chunked gradients, when the long computation can be cut into manageable blocks;
- neural ODE-style adjoints, when the computation is better viewed as a continuous trajectory;
- implicit differentiation, when the final state is defined by a fixed point or root rather than by a particular iteration history;
- envelope-theorem gradients, when an inner variable is an exact or approximate argmin.

The unifying idea is simple: decide what mathematical object defines the output. If the output is the result of a particular finite program, differentiate the program, perhaps with checkpointing. If the output is the solution of an equation, differentiate the equation. If the output is the value of an optimized objective, use stationarity so the derivative of the optimizer path does not have to appear.

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

This formula is not the problem. It is the storage. To compute the vector-Jacobian product at step $k$, the backward pass usually needs the forward state $x_k$ and local intermediates created while evaluating $F_k$. If $T$ is large, the memory cost is roughly proportional to the full trajectory.

There is also a numerical issue. The gradient of a long composition contains products of Jacobians. Some modes grow, some vanish, and some are dominated by discretization or solver details. Long-step gradients are not merely expensive; they often ask the wrong question if the finite path is only a method for reaching an equilibrium, a root, or an optimum.

So the first diagnostic question is:

**Do I care about the whole path, or only about the object reached at the end?**

If the path itself is the model, such as an actual recurrent network or a finite diffusion sampler, unrolled gradients are faithful. If the path is just a solver for something else, implicit or envelope gradients are often cleaner.

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

This is the right move when the finite unroll is semantically important. For example:

- a Transformer with many layers whose exact layerwise computation defines the model;
- an unrolled diffusion sampler trained through a fixed number of denoising steps;
- a differentiable simulator where the discretized trajectory is the object being optimized;
- a meta-learning inner loop where the number of inner steps is part of the algorithm.

Checkpointing does not change the gradient. It only changes the memory schedule. That is why it is so useful as a first rescue attempt: the math stays the same.

### Choosing Checkpoints

Uniform chunks are the easiest policy. If each step costs about the same and has about the same activation size, choose a chunk size that fits in memory. A small chunk uses more memory but less recomputation; a large chunk uses less memory but more recomputation.

There are more optimal schedules. Recursive checkpointing algorithms can achieve much better memory-compute tradeoffs than uniform chunking, especially when $T$ is large and memory is tight. But the intuition remains the same: keep a sparse skeleton of the trajectory, and rebuild local flesh only when the backward pass arrives there.

The practical rule is:

**Use checkpointing when backpropagation through the actual steps is still the desired derivative, but saved activations are too expensive.**

## 3. Chunked Backpropagation and Truncated Gradients

The word "chunk" can mean two different things, and it is worth keeping them separate.

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

This produces a biased gradient for the full $T$-step objective, but it can be a good algorithm. Recurrent networks, long-horizon control, online learning, and some simulation-calibration problems all use truncated gradients because exact long-range credit assignment is noisy, unstable, or too expensive.

The distinction is philosophical as much as technical:

- exact chunking asks for the true derivative of the long computation;
- truncated chunking asks for a local training signal that may be good enough.

If the goal is a reliable derivative of a scientific objective, truncation should be treated carefully. It can optimize a different problem than the one written down. If the goal is representation learning or control, the biased signal may be acceptable because the final trained behavior matters more than exact differentiation of the bookkeeping.

## 4. Neural ODE Adjoints: Backpropagating Through Time Without Storing Time

For a continuous-time model,

<div class="math-display">
$$
\frac{dz(t)}{dt}=f(z(t),t,\theta),
\qquad
z(0)=z_0,
\qquad
L=\ell(z(T)),
$$
</div>

one can view the forward pass as an ODE solve rather than a very deep residual network. The adjoint method introduces

<div class="math-display">
$$
a(t)=\frac{\partial L}{\partial z(t)}.
$$
</div>

The continuous adjoint equation is

<div class="math-display">
$$
\frac{da(t)}{dt}
=
-
\left(\partial_z f(z(t),t,\theta)\right)^T a(t),
\qquad
a(T)=\nabla_{z(T)}\ell.
$$
</div>

The parameter gradient is accumulated along the reverse-time trajectory:

<div class="math-display">
$$
\frac{dL}{d\theta}
=
\int_T^0
-
a(t)^T\partial_\theta f(z(t),t,\theta)\,dt.
$$
</div>

Equivalently, integrating from $0$ to $T$,

<div class="math-display">
$$
\frac{dL}{d\theta}
=
\int_0^T
a(t)^T\partial_\theta f(z(t),t,\theta)\,dt.
$$
</div>

The memory appeal is obvious. Instead of saving every internal solver state, solve the state-adjoint system backward. In the ideal mathematical picture, the forward trajectory can be reconstructed by reversing the ODE from $z(T)$.

This is elegant, but the elegance hides practical traps. Numerical ODE solvers are not perfectly reversible. Adaptive solvers choose step sizes based on local error estimates, and those choices are themselves part of the discretized computation. Chaotic or stiff dynamics can make backward reconstruction unstable. In those cases, the continuous adjoint may be a gradient of the ideal ODE flow, while direct autodiff through the solver gives a gradient of the discrete numerical algorithm.

Neither answer is universally "more correct." They answer different questions:

- continuous adjoint: gradient of the underlying continuous-time model;
- differentiating the solver: gradient of the actual discretized computation.

Many robust implementations use a hybrid: save occasional checkpoints along the forward ODE solve, then integrate adjoints backward between checkpoints. This keeps memory under control without relying on reconstructing the entire trajectory from the final state.

The neural ODE lesson generalizes beyond ODEs:

**If the long computation approximates a continuous evolution, adjoint equations may be the natural gradient object. But numerical stability decides how much checkpointing you still need.**

## 5. Fixed Points and the Implicit Function Theorem

Now suppose the endpoint is not interesting because it came from $T$ iterations. It is interesting because it solves a self-consistency equation:

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

but the iteration is only a numerical route to the fixed point. If $z_\ast$ is the real object, differentiating through all solver steps is often wasteful.

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

This is the workhorse formula. It replaces "store every iteration and backpropagate through it" with "solve one linear system involving Jacobian-vector products." The Jacobian never has to be materialized. Krylov methods, conjugate gradients in symmetric cases, or fixed-point iterations for the adjoint can use vector-Jacobian products supplied by AD at the converged solution.

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

If the solver is converged, the implicit gradient is independent of how the solver got there. Newton, Anderson acceleration, simple iteration, or a warm start should give the same derivative, up to numerical tolerances. That is exactly what we want when the fixed point is a physical self-consistency condition or an equilibrium definition.

There are caveats. If the fixed point is not unique, the chosen branch matters. If $I-\partial_z\Phi$ is nearly singular, the gradient can be huge or ill-conditioned. If the solver has not converged, the implicit gradient may describe a nearby equation solution rather than the actual finite iterate. But these caveats are usually visible in residual norms and linear-solve diagnostics, which is much better than silently storing a thousand-step tape.

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

For constrained optimization, $G$ may be the KKT system. For a nonlinear PDE solve, $G$ may be the discretized residual. For electronic-structure self-consistency, $G$ may be a density or potential residual. The gradient asks how the solution changes when the parameters change, not how a particular iterative method wandered toward that solution.

This is also why implicit differentiation is so common in scientific machine learning. Many scientific codes already have residuals, solvers, and preconditioners. Rather than make the entire solver differentiable as a giant program trace, one can expose enough linear algebra to differentiate the defining equations.

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

This is not a cute cancellation. It is a major computational simplification. If the inner problem is a variational Monte Carlo optimization, a relaxed structure, a fitted auxiliary model, a free-energy variational bound, or a control/action minimization, differentiating through every optimizer step may be unnecessary when the outer objective is the optimized value itself.

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

But the envelope theorem says that for the value $v(\theta)=f(\theta,y_\ast(\theta))$, this sensitivity does not appear because it is multiplied by the zero first-order condition. So:

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

## 8. How to Choose the Strategy

Here is the decision tree I actually find useful.

If the finite computation is the model, differentiate the finite computation. If memory is too large, use checkpointing or exact chunking. This is the default for deep networks, finite unrolled samplers, and algorithms whose step count is part of the definition.

If the finite computation is only a solver for a fixed point or root, differentiate the fixed-point or residual equation. Store the converged solution, not the whole history. The backward pass becomes a linear solve involving Jacobian-vector or vector-Jacobian products.

If the finite computation is an optimizer and the outer objective is the optimized value, try the envelope theorem first. Do not pay for $dy_\ast/d\theta$ unless the outer loss actually depends on $y_\ast$ in a way that survives stationarity.

If the computation approximates a continuous-time flow, decide whether the target is the continuous model or the discretized solver. Continuous adjoints are natural for the former. Solver autodiff is natural for the latter. In difficult dynamics, checkpointed adjoints are often the practical middle ground.

The four strategies are not rivals. They are different answers to "what should be differentiated?"

## 9. A Small Translation Table

For a long unrolled computation,

<div class="math-display">
$$
x_T=F_{T-1}\circ\cdots\circ F_0(x_0,\theta),
$$
</div>

use full reverse-mode AD when $T$ is moderate and memory is fine. Use checkpointing when $T$ is large but the exact unrolled derivative is wanted. Use truncation when a biased local signal is acceptable.

For an ODE,

<div class="math-display">
$$
\dot z=f(z,t,\theta),
$$
</div>

use adjoint equations when the continuous trajectory is the mathematical model. Add checkpoints when reversibility or stiffness makes pure backward reconstruction unreliable.

For a fixed point,

<div class="math-display">
$$
z_\ast=\Phi(z_\ast,\theta),
$$
</div>

solve

<div class="math-display">
$$
\left(I-\partial_z\Phi\right)^Tv=\nabla_z\ell
$$
</div>

and compute the parameter gradient from local derivatives at $z_\ast$.

For an inner minimization,

<div class="math-display">
$$
v(\theta)=\min_y f(\theta,y),
$$
</div>

use

<div class="math-display">
$$
\nabla_\theta v=\partial_\theta f(\theta,y_\ast)
$$
</div>

when the optimum is regular and the outer objective is the optimized value.

## 10. The Main Moral

Long-step gradients become manageable when we stop treating every algorithm as a tape that must be replayed backward.

Sometimes the tape is real, and checkpointing is the right memory strategy. Sometimes the tape is a numerical route to an endpoint, and the endpoint is defined by an equation; then the implicit function theorem is the right language. Sometimes the endpoint is the value of an optimized problem, and the envelope theorem removes the derivative of the optimizer. Sometimes the path is continuous, and adjoint equations express the gradient more naturally than a stack of discrete activations.

The common pattern is to replace "differentiate everything that happened" with "differentiate the object that matters." That small shift is often the difference between an elegant method on paper and a method that can actually run.
