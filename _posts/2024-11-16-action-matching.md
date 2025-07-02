---
layout: post
title: Action Matching
description: Notes on Action Matching
date: 2024-11-16
related_posts: false  
---
My notes on <a href="https://arxiv.org/pdf/2210.06662">Action Matching: Learning Stochastic Dynamics from Samples</a>.

## TL;DR
Given dynamics/densities $$q_t$$, there are many different vector fields $$v_t$$ that provide us with ODEs or SDEs to simulate these dynamics. Action Matching proposes to choose $$v_t$$ to be the unique gradient field (of the form $$\nabla s_t$$) in each case. The paper provides tractable objectives (if one can sample from $$q_t$$), extensions, and connections to optimal transport.

## Continuity Equation
Suppose that we have a continuous path of densities $$(q_t)_{t \geq 0}$$. We may want a vector field
that pushes particles to follow these densities. In particular, suppose that a particle is initialized as
$$x_0 \sim q_0$$. We can evolve the particle to follow some time-dependent vector field $$v_t$$ as follows

$$
  \frac{d}{dt}x(t) = v_t(x(t)), \quad x(0) = x_0.
$$

The densities $$q_t$$ are equal to the laws of $$x(t)$$ iff the
<em>continuity equation</em> holds:

$$
  \frac{\partial}{\partial t}q_t(x) = - \text{div}(q_t v_t)(x) = - \nabla \cdot (q_t v_t)(x).
$$

Recall the intuition that $$\text{div}(q_t v_t)(x)$$ tells us how much outward flow the density-weighted vector field
$$v_t q_t$$ induces at a point $$x$$, and so the density will change opposite to that.

## Helmholtz Decomposition
The standard <a href="https://en.wikipedia.org/wiki/Helmholtz_decomposition">Helmholtz Decomposition</a> states that a vector field $$v$$ can be decomposed as

$$
  \begin{aligned}
    v(x) &= \nabla s(x) + w(x), \\
    \text{div}(w)(x) &= 0.
  \end{aligned}
$$

In particular, this means that $$v$$ can be written as the sum of a gradient field $$\nabla s$$ and a divergence-free term $$w$$. Under some conditions, the decomposition is unique (up to a constant for $$s$$).

This result can be extended to consider the divergence operator with respect to some density $$q$$. In particular,
any vector field $$v$$ can be uniquely decomposed as

$$
  v(x) = \nabla s(x) + w(x)
$$

where $$w$$ is divergence free w.r.t. $$q$$ in the sense that $$\text{div}(qw)(x) = 0$$.

## Action Matching
For a given path of densities $$q_t$$, there are many vector fields $$v_t$$ that will satisfy the continuity equation,
and thus keep the dynamics evolving as $$q_t$$. In particular, adding any $$a_t$$ to $$v_t$$ such that
$$\text{div}(q_ta_t) = 0$$ will still satisfy the continuity equation.

However, the Helmholtz Decomposition tells us that all $$v_t$$ that satisfy the continuity equation will have
the same unique gradient field $$\nabla s_t$$. In particular, if $$v^{(1)}_t, v^{(2)}_t$$ both satisfy the
continuity equation and their Helmholtz Decompositions are $$v^{(i)}_t = \nabla s^{(i)}_t + w^{(i)}_t$$, then

$$
  \begin{aligned}
    0 &= \frac{\partial}{\partial t}(q_t-q_t) \\
      &= \text{div}(q_t(\nabla s^{(1)}_t-\nabla s^{(2)}_t)) + \text{div}(q_tw^{(1)}_t) -\text{div}(q_tw^{(2)}_t) \\
      &= \text{div}(q_t(\nabla s^{(1)}_t-\nabla s^{(2)}_t)) + 0 - 0.
  \end{aligned}
$$

Hence, the Helmholtz Decomposition is unique only if $$\nabla s^{(1)}_t-\nabla s^{(2)}_t = 0$$. Otherwise, we could
add and subtract a multiple of $$\nabla s^{(1)}_t-\nabla s^{(2)}_t$$ from $$\nabla s^{(i)}_t$$ and $$w^{(i)}_t$$ respectively, without changing $$v^{(i)}_t$$.

Therefore Action Matching (AM) suggests choosing $$v_t$$ in a canonical way, by setting $$v_t = \nabla s_t$$, the unique gradient field satisfying the continuity equation. The scalar function $$s_t$$ (unique up to a constant) is called the <em>action</em>.

Theorem 2.2 provides a tractable objective for learning an approximation $$\hat{s}_t$$ to the true action $$s_t$$. Once
$$\hat{s}_t$$ is trained, sampling can be performed by evolving an ODE as follows:

$$
  \frac{d}{dt}x(t) = \nabla \hat{s}_t(x(t)), \quad x(0) \sim q_0.
$$

## Extensions

The paper also introduces entropic Action Matching (eAM). Instead of evolving an ODE to satisfy the dynamics
$$q_t$$, one could also evolve an SDE:

$$
  dX(t) = v_t(X(t))dt + \sigma_t dB_t, \quad X(0) \sim q_0.
$$

The evolution of such a density is characterized by the Fokker-Planck equation which states that the law of
$$X(t)$$ has density $$q_t$$ iff

$$
  \frac{\partial}{\partial t}q_t = - \text{div}(q_t v_t) + \frac{\sigma_t^2}{2}\Delta q_t.
$$

The intuition behind $$\frac{\sigma_t^2}{2}\Delta q_t$$ is that the Laplacian $$\Delta q_t(x)$$ tells us how much
more density there is around a point $$x$$ than at $$x$$. So, if this value is high, the diffusion will increase the
amount of density at $$x$$.

Fixing $$\sigma_t$$, there are once again many different $$v_t$$ that satisfy the above (we can add any divergence-free
term to $$v_t$$). However, eAM suggests choosing
$$v_t$$ to be the unique scalar field $$v_t = \nabla s_t$$ that satisfies the Fokker-Planck equation. Proposition 3.2
provides a tractable objective.

The paper also introduces unbalanced Action Matching where particles can be created and destroyed according to some growth rate $$g_t(x)$$ for position $$x$$ at time $$t$$. Such dynamics are governed by the ODE
$$\frac{\partial}{\partial t} q_t = - \text{div}(q_t v_t) + q_t g_t$$.

The paper extends AM to more general convex costs and discusses connections to optimal transport.
