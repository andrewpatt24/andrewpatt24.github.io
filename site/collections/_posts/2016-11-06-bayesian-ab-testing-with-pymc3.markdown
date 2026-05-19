---
date: 2016-11-06
title: Bayesian A/B testing with PyMC3
description: >-
  How to compare conversion rates across variants using a hierarchical Beta–Bernoulli
  model and MCMC — with code from python_bayesAB.
tags:
  - ml
  - python
  - statistics
image: /images/post-5.jpg
---

Product teams run **A/B tests** to decide whether a change (new headline, checkout flow, pricing) beats the status quo. You show variant A to some users and variant B to others, count conversions, and ask: *which variant is better, and by how much?*

Classical (frequentist) tests answer that with p-values and fixed sample sizes. A **Bayesian** approach instead gives you a **distribution over the true conversion rate** for each variant — and, crucially, a **probability that one variant beats another**. That matches how many practitioners actually want to make decisions.

This post walks through a small Python library I built for that workflow: [python_bayesAB](https://github.com/andrewpatt24/python_bayesAB). It uses [PyMC3](https://docs.pymc.io/) to fit a hierarchical Beta–Bernoulli model and sample posteriors with MCMC.

## The problem in one sentence

You have **K variants** (A, B, C, …). For each variant *i* you observe:

- **nᵢ** — users exposed  
- **cᵢ** — conversions (clicks, sign-ups, purchases, etc.)

You want posterior beliefs about each variant’s true conversion rate **pᵢ**, and about **differences** like p_B − p_A.

## Frequentist vs Bayesian (briefly)

| | Frequentist | Bayesian |
|---|-------------|----------|
| **Output** | p-value, confidence interval | Posterior distribution, P(A beats B) |
| **Stopping** | Fixed n often assumed | Can peek; priors encode prior knowledge |
| **Multi-arm** | Corrections (Bonferroni, etc.) | Natural via joint model on all arms |

Neither is universally “better”; Bayesian methods shine when you want **direct statements about parameters** (“there’s a 94% chance B beats A”) and when you’re comparing **more than two** variants in one model.

## Model: hierarchical Beta–Bernoulli

The core class is `BayesABConversion`. For conversion data it assumes each user outcome is Bernoulli with variant-specific rate **pᵢ**.

**Hierarchical prior** (default `model_type='heirarchical'`):

1. Hyperparameters **μ** and **σ** get weak uniform priors on (0, 1) and a sensible scale for Beta spread.  
2. Each **pᵢ** is drawn from `Beta(μ, σ)` — variants share structure but can differ.  
3. Observations are **Bernoulli(pᵢ)** per user, pooled into a long binary vector with an index for variant.

For every pair of variants *(i, j)*, the model also defines a deterministic **δᵢⱼ = pⱼ − pᵢ**. After MCMC, those deltas are what you use to rank variants.

Conceptually:

```text
μ, σ  ~  Uniform priors
pᵢ    ~  Beta(μ, σ)     for each variant i
y     ~  Bernoulli(p[idx])   per user
δᵢⱼ   =  pⱼ − pᵢ           for i < j
```

A **simple** alternative (`model_type='simple'`) puts independent `Uniform(0, 1)` priors on each **pᵢ** with no hierarchy. The repo implements both builders; the default runner currently wires up the hierarchical path.

## From counts to PyMC3

You pass cohort summaries, not raw clickstreams:

```python
from BayesAB.bayes_AB_conversion import BayesABConversion

bab = BayesABConversion()
bab.fit(n=[1000, 1000, 1000], c=[500, 450, 400])
```

- **n** — list of exposure counts per variant  
- **c** — list of conversion counts per variant  

Internally, `fit` expands that into binary **obs** (1 = converted, 0 = not) and **idx** (which variant each row belongs to), then runs Metropolis sampling (default 50,000 draws).

The trace object is standard PyMC3 — use `bab.traceplot()` or PyMC3’s plotting utilities to inspect **p**, **hyper_mu**, **hyper_sd**, and **delta_*** chains.

### Example interpretation

Suppose variant 0 is control and variant 1 is treatment. After sampling, look at the posterior of `delta_01` (= p₁ − p₀):

- Mass mostly **above 0** → treatment likely improves conversion.  
- Narrow distribution → you’re relatively sure; wide → need more data or weaker priors.

For three arms you get deltas for (0,1), (0,2), (1,2), so you can compare any pair without running separate two-arm tests.

## Simulating data

`BayesAB/bayes_AB_random_data.py` includes a `BernoulliIterator` helper to grow synthetic cohorts under known true rates **p** — useful for sanity-checking the sampler before you point it at production logs.

## Design choices in this repo

**Why hierarchy?** When you test several similar UI tweaks, true rates are often correlated. Partial pooling via hyper-**μ** and **σ** borrows strength across arms instead of treating each variant as unrelated.

**Why Metropolis?** PyMC3’s default NUTS is usually preferred today; this code uses `pm.Metropolis()` for a straightforward, dependency-light path on small Bernoulli models. For production you’d likely move to NUTS or a conjugate Beta–Binomial update where closed form applies.

**Scope today.** The top-level `BayesAB` class is a stub; conversion testing lives in `BayesABConversion`. The README describes the project as a “Bayesian AB framework”; extending it to revenue (Normal/Gamma likelihoods) or retention curves would be natural next steps.

## Practical checklist

1. **Define the metric** — binary conversion (this repo) vs continuous revenue (not implemented here).  
2. **Pass n and c** per variant; check for zero exposures.  
3. **Run MCMC**; inspect trace plots for mixing.  
4. **Read off δ posteriors** (or compute P(p_B > p_A) from samples of `p`).  
5. **Decide** using your org’s bar (e.g. deploy if P(treatment wins) > 0.95).

## Code and context

- Repository: [github.com/andrewpatt24/python_bayesAB](https://github.com/andrewpatt24/python_bayesAB)  
- Stack: Python, NumPy, PyMC3  
- Note: the source uses Python 2-era syntax (`xrange`, etc.); modernizing to Python 3 and PyMC (v4+) is a straightforward migration if you want to run it today.

If you’re new to Bayesian experimentation, Chris Stucchio’s [Bayesian A/B testing](https://www.chrisstucchio.com/blog/2014/bayesian_ab_testing.html) and the [PyMC3 docs](https://docs.pymc.io/) are solid companions to this code-first tour.

---

*Questions or ideas for extensions (revenue models, Thompson sampling, sequential tests)? Open an issue on the repo or reach out via the links on this site.*
