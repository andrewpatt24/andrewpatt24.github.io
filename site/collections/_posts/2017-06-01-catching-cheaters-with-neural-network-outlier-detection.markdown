---
date: 2017-06-01
title: Catching cheaters in mobile games with neural network outlier detection
description: >-
  Why cheat detection matters in free-to-play games, how cheaters exploit clients
  and networks, and how replicator neural networks can flag outliers for review and bans.
tags:
  - ai
  - ml
  - python
image: /images/post-3.jpg
---

Presented at PyData London 2017 (*War on Cheaters: Outlier Detection methods for cheating in mobile games*). This write-up summarises that work and the [NNOutlier](https://github.com/andrewpatt24/NNOutlier) reference implementation.

## Why catching cheaters matters

In free-to-play (F2P) mobile games, only a small slice of players ever pay. Industry estimates put mobile conversion rates around **1.9%**—and cheaters are another rare group sitting in the same long tail. They distort economies, frustrate paying players, and undermine trust in competitive play.

There is also a practical arms race: patching every client exploit or network trick at the source is never-ending. A complementary strategy is to **detect abnormal player behaviour** after the fact and act on it—review, restrict, or ban accounts that look nothing like legitimate users.

## How cheaters cheat

Cheating tends to fall into two broad categories:

**Client-side hacks** modify the game on the device—altered APKs or “assistant” apps that change what the client sends or displays.

**Network-based hacks** sit between client and server: capturing and editing transaction packets, or replaying the same packet many times to duplicate rewards or actions.

From a data perspective, both show up as unusual patterns in the stream of game transactions and in how players progress over time.

## Framing cheat detection as outlier detection

Rather than hand-coding every exploit signature, you can treat suspicious players as **outliers** in feature space built from gameplay data.

The presentation outlined a pipeline with two axes:

- **Transaction-level analysis** vs **user-level aggregation** (rolling behaviour up per account).
- **Live / near-live detection** vs **post-hoc analysis** on historical batches.

Once features are defined—battle win ratios, troop score changes, day-zero progression curves, currency balances, and so on—the question becomes: *which accounts sit far from the bulk of normal play?*

## Approaches before neural networks

### Single-variable rules

Simple thresholds on one metric can catch obvious cases: extreme battle win ratios, or progression on day zero that no legitimate player matches. These are easy to explain but miss cheats that look normal on any one axis.

### Multivariate distance methods

Combining features (e.g. win ratio and troop score delta) enables richer comparisons. Common tools include **PCA**, **Mahalanobis distance**, and **Euclidean distance** in the reduced or original space. The catch: a point can be far from the centre of the distribution without being a cheater—legitimate skilled or high-spending players can look “weird” too.

### Robust geometry

To reduce sensitivity to extreme values, **Minimum Covariance Determinant (MCD)** and **Minimum Volume Ellipsoid (MVE)** methods fit robust boundaries: find a small ellipsoid that contains most points, optionally with points on the boundary, and flag what falls outside. Libraries such as scikit-learn’s `EllipticEnvelope` and `MinCovDet`, and dedicated MVE implementations, support this line of work.

## Replicator neural networks

**Replicator neural networks** (Hawkins et al.; see the [dawak02](http://neuro.bstu.by/ai/To-dom/My_research/Papers-0/For-research/D-mining/Anomaly-D/KDD-cup-99/NN/dawak02.pdf) paper referenced in the repo) treat outlier detection as **reconstruction error**.

The network is trained to compress and reconstruct “normal” multivariate inputs through a narrow hidden layer—similar in spirit to an autoencoder. Inliers reconstruct well; outliers produce **high squared error** between input and output. That error is the outlier score.

The talk contrasted strict replicator activation functions with **relaxed assumptions** closer to modern autoencoders, which can be easier to train on messy game telemetry.

### Variable selection

Not every column in a log helps. Some fields (e.g. display names) are irrelevant; others (currency balances, battle outcomes) are directly tied to what cheaters manipulate. Selection can be:

- **Intuition- or research-driven**—ask what a cheater would need to change.
- **Algorithm-driven**—choose variables that maximise kurtosis or separation between likely inliers and outliers.

Getting features right matters as much as picking the detector.

## From outliers to bans

Finding outliers is only the start—you are looking for a needle in a haystack. **Ranked set resampling** helps: sort accounts by outlier score, sample heavily from the top of the list, and build a labelled set for analysts. Goals include:

- Understanding **vectors of cheating** (which features fire together).
- Building a **ground-truth dataset** for supervised models or policy rules.

Downstream you either train a **classifier** when behaviour is complex, or codify **heuristics** when patterns are simple and stable.

The summary slide from the talk still rings true: select sensible variables, pick density-based methods when data is roughly unimodal/normal, and reach for **replicator NNs / autoencoders** when the structure is messier—then resample, visualise, and operationalise bans or flags.

## Reference implementation

The [NNOutlier](https://github.com/andrewpatt24/NNOutlier) repository is a Python sketch of replicator-network ideas for outlier detection, adapted from public Kaggle neural-network examples and aligned with the Hawkins paper above. The core `ffneuralnet` class wires up layer sizes, sigmoid activations in hidden layers, and forward propagation so reconstruction error can be computed over multivariate player feature vectors.

For production-scale systems, the same pattern maps cleanly to frameworks such as TensorFlow’s `DNNClassifier` or any autoencoder stack; the talk noted that a full replicator implementation was planned to be open-sourced—this repo is that starting point.

## Takeaways

- F2P games have **rare cheaters and rare spenders**—outlier methods match the base rate.
- **Client and network cheats** both leave traces in transactional and aggregated user data.
- **Classical multivariate and robust estimators** are strong baselines; **replicator neural networks** add flexibility when “normal” play is hard to describe with simple geometry.
- Detection should feed a **human-in-the-loop review path** (ranked sampling, labels) before automated bans at scale.

If you are exploring cheat detection on game telemetry, start with clear features, benchmark simple elliptic-envelope or MCD baselines, then experiment with reconstruction-error models when linear separability is not enough.
