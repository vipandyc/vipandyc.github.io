# REACT Algorithm Walkthrough: Committor Learning via Stochastic Optimal Control

This note summarizes the algorithmic core of the paper "Rare Event Analysis via Stochastic Optimal Control." The goal is to explain the method without the rigorous proof machinery.

## 1. Problem Setup

You have stochastic dynamics:

$dX_t = b(X_t)dt + \sigma dW_t$

For overdamped Langevin dynamics:

$dX_t = -\nabla U(X_t)dt + \sqrt{2\beta^{-1}}dW_t$

Define two metastable sets:

- $A$: reactant basin
- $B$: product basin

The goal is to learn the committor:

$q(x)=P_x(\text{hit }B\text{ before }A)$

Concrete meaning:

- $q(x)=0$: the trajectory almost surely goes back to $A$
- $q(x)=1$: the trajectory almost surely reaches $B$
- $q(x)=0.5$: transition-state surface

The paper does not directly learn $q$. It learns a regularized value function:

$\Phi(x)=-\log q_\xi(x)$

Here $q_\xi$ is a softened committor:

- $q_\xi(x)=\xi$ in $A$
- $q_\xi(x)=1-\xi$ in $B$

Usually $\xi$ is small, for example $0.01$. This avoids the singularity from $\log 0$.

After training:

$q_\xi(x)=e^{-\Phi(x)}$

and the original committor is recovered by:

$q(x)=\frac{q_\xi(x)-\xi}{1-2\xi}$

## 2. Neural Parameterization

The network should satisfy the boundary conditions automatically. The paper uses a parameterization like:

$q_\theta(x)=\xi+(1-2\xi)\frac{d_A(x)\psi_1(x)}{d_A(x)\psi_1(x)+d_B(x)\psi_2(x)}$

where:

- $d_A(x)$ is distance to set $A$
- $d_B(x)$ is distance to set $B$
- $\psi_1(x),\psi_2(x)>0$ are neural network outputs, e.g. softplus outputs

Check the boundary behavior:

- If $x\in A$, then $d_A(x)=0$, so $q_\theta(x)=\xi$
- If $x\in B$, then $d_B(x)=0$, so $q_\theta(x)=1-\xi$

Then define:

$\Phi_\theta(x)=-\log q_\theta(x)$

This is the quantity trained by the algorithm.

## 3. Core Control Idea

Given the current value function $\Phi_\theta$, define a controlled drift:

$b_\theta(x)=b(x)-2D\nabla\Phi_\theta(x)$

Since $\Phi=-\log q_\xi$, this is equivalent to:

$b_\theta(x)=b(x)+2D\nabla\log q_\xi(x)$

Here:

$D=\frac{1}{2}\sigma\sigma^T$

For overdamped Langevin:

$D=\beta^{-1}I$

Interpretation: the learned committor creates a force field that pushes trajectories toward successful rare transitions.

## 4. Algorithm A: REACT-DBP

DBP means direct backpropagation. This is the straightforward version.

At each training step:

1. Sample starting points $X_0\sim \mu_0$.
2. Simulate the controlled SDE:

   $dX_t=b(X_t)dt-2D\nabla\Phi_\theta(X_t)dt+\sigma dW_t$

3. Stop at $\tau_T=\min(\tau,T)$, where $\tau$ is the first hitting time of $A\cup B$.
4. Minimize the trajectory cost:

   $L_{\rm DBP}(\theta)=E[\int_0^{\tau_T} |\nabla\Phi_\theta(X_t)|_D^2 dt + \Phi_{\rm stop}(X_{\tau_T})]$

Here $\Phi_{\rm stop}$ means the terminal value is detached from gradient flow.

### DBP Pseudocode

```python
for step in training_steps:
    x = sample_initial_batch()

    loss_running = 0

    for k in range(num_time_steps):
        grad_phi = grad(Phi_theta(x), x)
        drift = b(x) - 2 * D @ grad_phi

        x = x + drift * dt + sigma @ normal() * sqrt(dt)

        loss_running += dot(grad_phi, D @ grad_phi) * dt

        if x hits A or B:
            break

    terminal = stop_gradient(Phi_theta(x))
    loss = mean(loss_running + terminal)
    update theta by backprop through trajectory
```

DBP is simple, but it has two practical problems:

- It requires differentiating through the full simulated trajectory.
- If the current controller is poor, it may still generate poor training trajectories.

## 5. Algorithm B: REACT-VM

VM means Value Matching. This is the more important contribution.

Instead of requiring trajectories to come exactly from the current controller, VM can train from off-policy trajectories. That means the sampler can be chosen for convenience, while the learning objective still targets the correct committor.

The sampled trajectory follows:

$dX_t^{v,\kappa}=v(X_t^{v,\kappa})dt+\sqrt{\kappa}\sigma dW_t$

where:

- $v$ is the chosen sampling drift
- $\kappa$ controls the noise level
- $\kappa=1$ means normal noise
- $\kappa<1$ makes paths more directed and barrier crossing easier

The ideal sampling drift would use the true committor. During training, the paper uses the current network estimate instead.

For reversible overdamped dynamics, a practical choice is approximately:

$v_\theta(x)=b(x)-2D\nabla\bar{\Phi}_\theta(x)$

where $\bar{\Phi}_\theta$ is detached from gradients. In plain language:

- Use the current network to generate data.
- Do not backpropagate through the data-generation drift.

VM trains $\Phi_\theta$ using a residual along the trajectory:

$L_{\rm VM}(\theta)=E[R_\theta(X_{0:\tau_T})^2]$

The residual $R_\theta$ measures whether the candidate value function is self-consistent along the sampled path.

For $\kappa=1$, a simplified residual is:

$R_\theta = \int_0^{\tau_T} \langle \nabla\Phi_\theta(X_t), dX_t-(b(X_t)-D\nabla\Phi_\theta(X_t))dt\rangle + \Phi_\theta(X_0)-G(X_{\tau_T})$

The terminal target $G$ is:

- $G(x)=-\log\xi$ if $x\in A$
- $G(x)=-\log(1-\xi)$ if $x\in B$
- $G(x)=\Phi_\theta(x)$ if the trajectory does not hit before $T$

The exact paper adds correction terms for $\kappa\neq 1$, but the intuition is simple: the correct value function should make this path residual zero.

### VM Pseudocode

```python
for step in training_steps:
    # 1. Detach current network for sampling
    Phi_bar = stop_gradient(Phi_theta)

    # 2. Sample trajectory off-policy
    x = sample_initial_batch()
    residual = Phi_theta(x)

    for k in range(num_time_steps):
        grad_phi_train = grad(Phi_theta(x), x)
        grad_phi_sample = grad(Phi_bar(x), x)

        # Sampling drift; detached
        v = b(x) - 2 * D @ grad_phi_sample

        noise = normal()
        dx = v * dt + sqrt(kappa) * sigma @ noise * sqrt(dt)
        x_next = x + dx

        # Value-matching residual increment
        residual += inner(
            grad_phi_train,
            (1 / kappa) * dx + (-1 / kappa * b(x) + D @ grad_phi_train) * dt
        )

        x = x_next

        if x hits A or B:
            break

    if x in A:
        terminal = -log(xi)
    elif x in B:
        terminal = -log(1 - xi)
    else:
        terminal = Phi_theta(x)

    residual -= terminal
    loss = mean(residual ** 2)
    update theta
```

## 6. What Makes VM Different From DBP?

DBP says:

> Generate trajectories using my current controller and backpropagate through the whole simulation.

VM says:

> Generate trajectories using a convenient sampler, then force the value function to satisfy the correct stochastic-control identity along those trajectories.

This makes VM more flexible. In principle, the off-policy data could come from:

- current controlled dynamics
- annealed dynamics with $\kappa<1$
- biased initial states
- reactive-density-like sampling
- PTMCMC-generated configurations or trajectories
- diffusion-sampler-generated configurations, if paired with a usable dynamical interpretation

## 7. What Does $\kappa$ Do?

VM samples trajectories using:

$dX_t=v(X_t)dt+\sqrt{\kappa}\sigma dW_t$

If $\kappa<1$, the noise is reduced. The paper also adjusts the drift so the process follows the reactive current more strongly.

Intuition:

- Normal dynamics wander.
- Controlled dynamics push toward reactive paths.
- $\kappa<1$ makes the path less diffusive and more streamline-like.

In the limit $\kappa\to 0$, the dynamics approach deterministic reactive-current flow.

This can help metastable systems because fewer trajectories are wasted wandering inside basins.

## 8. Full Practical Pipeline

For a toy implementation:

1. Define $A$, $B$, $b(x)$, $\sigma$, and $D$.
2. Build neural network $q_\theta(x)$ with hard boundary conditions.
3. Set $\Phi_\theta(x)=-\log q_\theta(x)$.
4. Choose initial distribution $\mu_0$.
   - Simple option: sample near the transition region, if known.
   - Better option: use PT, umbrella sampling, or a previous sampler.
5. Train with REACT-VM:
   - detach the old $\Phi_\theta$ to define the sampling drift,
   - simulate capped trajectories,
   - compute the VM residual,
   - minimize the squared residual.
6. Recover the committor $q(x)$.
7. Compute derived quantities:
   - transition surface: $q(x)=0.5$
   - reaction rate: $\int \rho(x)\nabla q(x)\cdot D\nabla q(x)dx$
   - reactive density for reversible systems: $\rho_R(x)\propto \rho(x)q(x)(1-q(x))$
8. Generate reactive paths using Doob dynamics:

   $dX_t=b(X_t)dt+2D\nabla\log q_\theta(X_t)dt+\sigma dW_t$

## 9. Clean Mental Model

- $q_\theta$ is the learned probability of reaching $B$ before $A$.
- $\nabla\log q_\theta$ is the learned rare-event force.
- DBP trains this force by differentiating through its own rollouts.
- VM trains it by checking pathwise consistency, so it can use better off-policy sampling.
- $\kappa$ gives a knob to make rare transitions easier without simply destroying the reactive physics.

For glass sampling, the attractive part is VM: PTMCMC, biased structural ensembles, or diffusion-sampler outputs could potentially provide off-policy data for learning a committor-like or transition-current-like object.
