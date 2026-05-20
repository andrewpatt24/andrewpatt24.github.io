---
date: 2017-06-01
title: Catching cheaters in mobile games with neural network outlier detection
description: >-
  PyData London 2017 talk recap: F2P cheat economics, client and network exploits,
  outlier detection from rules through robust geometry to replicator nets.
  NNOutlier reference code.
tags:
  - ai
  - ml
  - python
image: /images/post-3.jpg
---

Cheaters are rare accounts and loud problems: support load, broken economies, competitive distrust. Client patches and network fixes do not end the arms race. **Post-hoc detection** (review, throttle, ban) on telemetry is the complement.

Summary of PyData London 2017, *War on Cheaters: Outlier Detection methods for cheating in mobile games*, plus [NNOutlier](https://github.com/andrewpatt24/NNOutlier).

## F2P base rates

Mobile conversion sits around **1.9%** in industry ballparks. Cheaters are another thin tail with outsized impact. The modelling question: which accounts sit far from normal play once features are defined?

## Where cheats appear

**Client:** modified APKs, assistant apps, altered payloads.

**Network:** capture, edit, or replay transactions.

Fingerprints show up in **transaction streams** and **per-user aggregates** over time.

## Framing

Hand-written exploit signatures scale poorly. **Outlier detection** on a feature space scales better.

Axes from the talk:

- Transaction-level vs per-user rollups.  
- Near-live vs batch history.

Features might include win ratio, troop score delta, day-zero progression, currency. Then: distance from the bulk of legitimate accounts.

## Baselines

### One variable

Thresholds on win ratio or day-zero progression catch obvious cases. Single-axis rules are explainable and easy to evade.

### Multivariate distance

**PCA**, **Mahalanobis**, **Euclidean** on combined features. High spenders and skilled players can sit far from the centroid without cheating.

### Robust geometry

**MCD**, **MVE**: small ellipsoid around most points; flag outside. `EllipticEnvelope`, `MinCovDet`, dedicated MVE implementations.

## Replicator networks

[Hawkins et al.](http://neuro.bstu.by/ai/To-dom/My_research/Papers-0/For-research/D-mining/Anomaly-D/KDD-cup-99/NN/dawak02.pdf) (see repo link): train compress-and-reconstruct on “normal” multivariate rows. **Squared reconstruction error** scores outliers. Strict replicator activations vs relaxed autoencoder-style variants were compared in the talk for messy game logs.

### Features

Drop noise columns; keep what cheaters move (currency, battle outcomes). Domain picks or kurtosis-driven separation.

## Scores to policy

Outlier score ≠ ban. **Ranked set resampling** oversamples the top of the list for analyst review: co-firing features, labelled sets for supervised models or stable rules. Classifiers for tangled behaviour; heuristics when patterns stay flat.

Operational sequence: sensible variables; density/robust methods when unimodal; replicator or autoencoder when geometry fails; resample, visualise, human review before mass auto-ban.

## NNOutlier

Python sketch from Kaggle-style examples aligned with Hawkins. `ffneuralnet`: layer sizes, sigmoid hiddens, forward pass, reconstruction error on player vectors. Production would use a modern autoencoder stack; the repo is a starting point.

## Summary

Rare cheaters suit outlier methods. Client and network paths both leave telemetry traces. Start with robust multivariate baselines; add reconstruction models when “normal” is non-elliptical. Label before blind bans.

---

*Talk slides and repo on GitHub; production anti-cheat needs policy and legal review beyond this sketch.*
