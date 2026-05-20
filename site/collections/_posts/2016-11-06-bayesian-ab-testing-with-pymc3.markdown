---
date: 2016-11-06
title: Bayesian A/B testing with PyMC3
description: >-
  Variant counts to posterior distributions: hierarchical Beta-Bernoulli model,
  MCMC in PyMC3, pairwise deltas. Code in python_bayesAB.
tags:
  - ml
  - python
  - statistics
image: /images/post-5.jpg
---

You run an **A/B test**: split traffic, count conversions, then someone asks for **P(B beats A)** in addition to whether the difference cleared α = 0.05.

**Bayesian** runs give **posteriors on true conversion rates** and **probabilities that one arm beats another** from those posteriors. [python_bayesAB](https://github.com/andrewpatt24/python_bayesAB) implements a hierarchical Beta-Bernoulli model in [PyMC3](https://docs.pymc.io/) with MCMC.

## Setup

**K variants.** Per variant *i*:

- **nᵢ** exposures  
- **cᵢ** conversions  

Target: posteriors for **pᵢ** and differences **p_B − p_A**.

## Frequentist vs Bayesian

| | Frequentist | Bayesian |
|---|-------------|----------|
| Output | p-value, CI | Posterior, P(A beats B) |
| Stopping | Fixed n common | Priors; peeking handled differently |
| Multi-arm | Bonferroni, etc. | One joint model |

Useful when you want **statements about parameters** and **several arms in one fit**.

## Hierarchical Beta-Bernoulli

Class: `BayesABConversion`. Bernoulli outcomes, variant rate **pᵢ**.

Default `model_type='heirarchical'` (repo spelling):

1. **μ**, **σ** hyperpriors (weak uniforms on (0, 1) and Beta spread).  
2. **pᵢ** ~ `Beta(μ, σ)`.  
3. Pooled **Bernoulli(pᵢ)** with variant index **idx**.  
4. **δᵢⱼ = pⱼ − pᵢ** for each pair.

```text
μ, σ  ~  Uniform priors
pᵢ    ~  Beta(μ, σ)
y     ~  Bernoulli(p[idx])
δᵢⱼ   =  pⱼ − pᵢ
```

`model_type='simple'`: independent `Uniform(0, 1)` on each **pᵢ**. Both builders exist; default uses hierarchy.

## Fit

Cohort summaries only:

```python
from BayesAB.bayes_AB_conversion import BayesABConversion

bab = BayesABConversion()
bab.fit(n=[1000, 1000, 1000], c=[500, 450, 400])
```

`fit` builds **obs** and **idx**, runs Metropolis (default 50,000 draws). Inspect **p**, **hyper_mu**, **hyper_sd**, **delta_*** via `bab.traceplot()` or PyMC3 plots.

### `delta_01`

Control = 0, treatment = 1. Posterior mass of `delta_01` (= p₁ − p₀) mostly above 0 suggests a lift. Width reflects data and prior. Three arms yield deltas for (0,1), (0,2), (1,2) in one run.

## Simulation

`BayesAB/bayes_AB_random_data.py`: `BernoulliIterator` for synthetic cohorts under known **p**.

## Repo notes

**Hierarchy:** similar UI tweaks often share structure; hyper-**μ** and **σ** pool strength.

**Metropolis:** fine for small Bernoulli models here; production might use NUTS or conjugate Beta-Binomial.

**Scope:** top-level `BayesAB` is a stub; conversion logic lives in `BayesABConversion`.

## Checklist

1. Binary conversion (this repo) vs other metrics (out of scope here).  
2. Pass **n**, **c**; avoid zero exposure.  
3. Run MCMC; check mixing.  
4. Read **δ** or P(p_B > p_A) from **p** samples.  
5. Ship against a bar (e.g. P(treatment wins) > 0.95).

## Links

[github.com/andrewpatt24/python_bayesAB](https://github.com/andrewpatt24/python_bayesAB) · Python, NumPy, PyMC3 · Python 2 syntax (`xrange`) in source; Python 3 + PyMC v4 migration is mechanical.

[Chris Stucchio on Bayesian A/B](https://www.chrisstucchio.com/blog/2014/bayesian_ab_testing.html) · [PyMC3 docs](https://docs.pymc.io/)

---

*Revenue models, Thompson sampling, sequential tests: open an issue on the repo.*
