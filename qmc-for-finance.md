Why Pricing and Risk Desks Should Care About Quasi-Monte Carlo
================================================================

*A practitioner's translation of five years spent building faster numerical integration methods — and why the same machinery sits underneath option pricing and risk simulation.*

If you've ever waited for an overnight VaR run to finish, or watched a pricing engine chew through millions of simulated paths for a basket option, you've run into the same mathematical problem my PhD spent five years on: **how do you approximate a high-dimensional integral without needing an astronomical number of points to do it accurately?**

Most of the time this problem is invisible — it's buried under "Monte Carlo simulation" in a risk engine's documentation. I want to unpack what's actually happening in there, and why a research area called quasi-Monte Carlo (QMC) methods — the subject of my PhD in Computational Mathematics — is a direct, practical answer to it.

## Pricing and risk are integration problems in disguise

Strip away the finance vocabulary and a lot of quant work reduces to the same task: computing the expected value of some payoff or loss function, where that function depends on many underlying random variables.

- Pricing a basket option depends on the joint behavior of every asset in the basket.
- Pricing a path-dependent option (Asian, barrier, American-style) depends on the asset's value at every simulated time step along the path.
- Computing portfolio VaR or CVaR depends on the joint distribution of every risk factor in the book.

Each of these is, mathematically, an integral over a high-dimensional space — often tens, hundreds, or effectively unbounded dimensions once you discretize a time-continuous path. And these integrals essentially never have a closed-form solution. They have to be approximated numerically.

## Why standard Monte Carlo is the default — and where it hurts

The standard approach is Monte Carlo (MC) simulation: draw random scenarios, evaluate the payoff or loss on each one, and average. It's simple, it's dimension-agnostic, and it's why every pricing library ships an MC engine.

The catch is convergence speed. The error of a standard Monte Carlo estimate shrinks proportionally to **1/√N**, where N is the number of simulated paths — and this holds *regardless of dimension*. That sounds fine until you do the arithmetic: to cut your pricing error in half, you need **4x** as many simulated paths. To get one more decimal digit of accuracy, you need **100x** as many. For a real-time pricing engine, or a risk system that has to revalue a large book overnight, that's compute cost and latency you're paying directly.

## What quasi-Monte Carlo does differently

QMC methods replace random sampling with **deterministically chosen, well-distributed point sets** — constructed so they cover the integration domain more evenly than random points ever will by chance. Two well-studied families of these point sets are **lattice rules** and **(polynomial) lattice point sets**, and under the right smoothness assumptions on the function being integrated, they can converge dramatically faster than 1/√N — in some regimes, arbitrarily close to O(1/N), independent of dimension.

That's the appeal in one sentence: **the same accuracy for a fraction of the scenarios**, or a much tighter estimate for the same compute budget.

## The catch that became my PhD

Good QMC point sets don't just exist — someone has to construct them, and constructing a *good* one for a specific dimension and specific integrand is the hard part. The best-known method is the **component-by-component (CBC) algorithm**, which builds a good lattice rule one dimension at a time. But two practical problems limit it:

1. Most CBC-style methods need you to know the smoothness of your integrand *in advance* to get the right construction — which you often don't, especially for the kinds of nonlinear, path-dependent payoffs common in derivatives pricing.
2. Naively, construction cost itself can scale badly with dimension and number of points, eating into the speed advantage you're trying to gain.

My thesis, *Quasi-Monte Carlo Methods: Component-by-Component Algorithms* (Johannes Kepler University, 2022), developed and analyzed construction algorithms — a component-by-component digit-by-digit algorithm and a variant CBC algorithm — that address both problems. They recover the convergence rate associated with an integrand's actual smoothness **without needing that smoothness specified upfront**, and, under standard conditions on how much each dimension is "weighted" in the problem, the resulting error bounds hold **independent of the dimension d**. Critically, both can be implemented efficiently, reducing construction cost to **O(dN log N)** — fast enough to actually use, not just prove theorems about.

This work was published across three peer-reviewed papers, with coauthors Adrian Ebert, Peter Kritzer, Dirk Nuyens, and Tetiana Stepaniuk:

- *Digit-by-digit and component-by-component construction of lattice rules for periodic functions with unknown smoothness* — Journal of Complexity, 66, 101555 (2021)
- *Construction of good polynomial lattice rules in weighted Walsh spaces by an alternative component-by-component construction* — Mathematics and Computers in Simulation, 192, 399–419 (2022)
- *Component-by-component digit-by-digit construction of good polynomial lattice rules in weighted Walsh spaces* — Constructive Approximation (2022)

## Why this is more than academic

I'll be direct about scope: my thesis is pure computational mathematics — it doesn't run a derivatives book or backtest a risk model. But the object it produces (a fast, adaptive way to construct high-quality low-discrepancy point sets) is exactly the input a QMC-based pricing or risk engine needs. Where a bank's quant library already uses lattice rules or Sobol sequences for Monte Carlo variance reduction, the construction method behind those point sets is precisely the kind of problem this research addresses — and unknown or varying integrand smoothness is the realistic case for the exotic, path-dependent payoffs that make MC necessary in the first place.

Concretely, faster and more reliable convergence in this setting means:

- Fewer simulated scenarios needed to hit a target pricing or risk tolerance — real infrastructure cost savings at scale.
- Tighter, more defensible error bounds on risk figures like VaR/CVaR, which matters as much for model validation and regulatory scrutiny as it does for speed.
- Adaptive methods that don't require assuming smoothness properties of a payoff you haven't fully characterized yet — which is the normal state of affairs for path-dependent and exotic instruments.

## Where I sit now

I've since moved from proving convergence rates to building the data and ML systems that consume this kind of numerical rigor day to day — ELT pipelines, forecasting models, monitored ML in production. But the standard hasn't changed: I still want the error bars, not just the point estimate. If you're working on pricing, risk simulation, or anywhere numerical integration meets a deadline, I'd enjoy comparing notes.

*Onyekachi (Kachi) Emenike, PhD — [kachiann.github.io/portfolio](https://kachiann.github.io/portfolio) · [GitHub](https://github.com/kachiann) · [LinkedIn](https://www.linkedin.com/in/onyekachi-osisiogu/)*
