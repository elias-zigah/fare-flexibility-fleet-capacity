# Fare flexibility and fleet capacity under a service reliability standard

How much operator-owned fleet capacity can regulated fare flexibility replace, when the service
must meet a probabilistic reliability standard? A large-scale simulation study of a depot-based
shared autonomous vehicle service.

> **Status.** Manuscript under preparation for submission to *Transportation Research Part E: Logistics and
> Transportation Review*. This repository currently holds the reproducibility record below. Code
> and the replication package are planned for release following acceptance, subject to
> data-sharing permissions.

---

## The question

A public operator commits to a fleet size before demand is realised, and must then meet a service
standard such as a 90th-percentile waiting time of five minutes. Vehicles are expensive and the
commitment is made in advance. Fares are the one instrument that can be adjusted afterwards, and
raising them suppresses demand, which relieves the same congestion that more vehicles would have
relieved.

Fares and fleet size are therefore partial substitutes. This study measures the exchange rate
between them, inside a regulated fare band, and then tests whether letting the fare respond to
real-time fleet state does better than the best jointly optimised fixed fare.

The estimands are the minimum compliant fleet at each permitted fare multiplier, the capacity
value of moving between two multipliers inside the band, and the fleet and net-benefit difference
between a frozen state-dependent policy and its frozen fixed-fare comparator.

## Design

| Stage | Purpose | Seeds | Runs |
| --- | --- | ---: | ---: |
| Fixed-fare benchmark | Minimum compliant fleet across the fare band | 15 | 10,200 |
| Tuning and controls | Rank and screen candidate rules | 5 | 6,545 |
| Held-out coarse | Select policy identities | 15 | 3,630 |
| Held-out refinement | Resolve fleet size within frozen identities | 15 | 2,790 |
| Independent confirmation | Evaluate frozen comparisons | 15 | 465 |

23,630 month-long simulation runs in total. Each run simulates a full calendar month of
event-level operation: stochastic request arrival, willingness-to-pay draw, fare acceptance,
dispatch feasibility and quoted wait, waiting-time acceptance, vehicle assignment and ride
pooling.

Three demand-response settings are calibrated to measured reference-fare elasticities of −1.5,
−1.0 and −0.6 while holding reference-fare acceptance fixed at 0.85, so the cases coincide at the
reference fare and diverge only as the multiplier moves away from it. Six reliability standards
are tested: 90th-percentile waiting time at 5.0, 4.5 and 4.0 minutes, and acceptance-conditioned
wait loss at 10%, 8% and 6%. Three vehicle capital costs are applied at analysis rather than
simulation, so the same operating realisations are evaluated at €600, €1,200 and €2,400 per
vehicle-month.

The state-dependent pricing rule observes the idle-fleet proportion and sets

```
m_t = clip( a + b(1 − I_t),  0.85,  1.50 )
```

raising the fare when little of the committed fleet is idle. Floor `a` and slope `b` are frozen
before evaluation.

## Headline results

**Fare flexibility substitutes for vehicles, and the exchange rate depends on elasticity.** Moving
across the regulated band from 0.85 to 1.50 under a five-minute reliability standard replaces:

| Demand response | Fleet at fare 0.85 | Fleet at fare 1.50 | Vehicles replaced |
| --- | ---: | ---: | ---: |
| Elastic (−1.5) | 30 | 9 | 21 |
| Unit-elastic (−1.0) | 29 | 15 | 14 |
| Inelastic (−0.6) | 29 | 20 | 9 |

**Tightening the standard is convex in vehicles.** Moving from 5.0 to 4.5 minutes costs three to
five additional vehicles across all 27 fare-by-elasticity combinations. Moving from 4.5 to 4.0
costs six to nine. The second increment is larger in every comparison.

**State-dependent pricing earns its place only where vehicles are expensive.** Of twenty
prespecified frozen comparisons carried to independent confirmation, ten achieve strict
confirmation, and all ten occur at €2,400 per vehicle-month. No state-dependent policy was
selected at €600 per vehicle-month or under inelastic demand at any cost. Nine of the ten
confirmed cases select a smaller fleet, by one to five vehicles, mean 3.22. Confirmed monthly
net-benefit margins range from €863 to €6,343.

Strict confirmation requires three conditions jointly: both frozen policies satisfy the applicable
reliability standard on the independent seeds, the mean state-dependent-minus-fixed margin is
positive, and the two-sided Holm-adjusted p-value is at most 0.05 across the full twenty-comparison
family.

Two elastic-demand comparisons at €1,200 have positive, statistically significant margins and
replication means inside the standard, but upper confidence limits just outside it, at 4.022
minutes against a 4.0-minute target and 6.039% against a 6% target. Both are reported as
infeasible. Counting them would have raised the headline from ten confirmations to twelve.

## Protocol

**Feasibility uses a suffix rule, not a first pass.** A fleet is feasible only if it and every
larger evaluated fleet pass the one-sided 95% upper-confidence-limit test. The reported
requirement is the smallest fleet in the feasible suffix. This prevents an isolated stochastic
pass, followed by a failure at a larger fleet, from defining the boundary. Every reported minimum
has an observed failure immediately below it and a feasible observation at or above it, resolved
at unit fleet resolution rather than read off a coarse grid.

**Selection and confirmation are separated.** Tuning ranks candidates on one seed block.
Held-out selection freezes policy identities and refines fleet size on a second. Independence
begins only once identities and fleet sizes are frozen, on a third seed block, with no
reoptimisation. The held-out coarse and refinement stages deliberately share seeds, because they
are one selection procedure rather than two.

**The random-number design is specified down to draw order.** Each run spawns two independent
generator streams from a seed sequence, one for demand and one for decisions. Within each
simulated hour the demand stream is consumed in a fixed order: sampling integer, request-minute
offsets as a single sorted vector, then per-trip party size and willingness-to-pay, then exactly
one wait-tolerance variate per request drawn before any acceptance or fleet-dependent branch.
Demand-side quantities are therefore identical across fleet sizes sharing a seed, which is what
makes the paired comparison valid.

> **Replication contract.** Matching integer seeds is not sufficient. A replication must preserve
> the bit generator, the seed-sequence spawning order, the pinned NumPy and pandas versions, the
> hourly draw order and the request order.

**The simulator is validated against known answers.** The queueing core is checked against
Erlang-C mean waiting times at 75% utilisation with a single depot and exponential service, over
200,000 requests per configuration with 20 batch means after a 25% warm-up discard:

| Servers | Erlang-C (min) | Simulated (min) | Relative error (%) |
| ---: | ---: | ---: | ---: |
| 3 | 9.084 | 9.524 | +4.8 |
| 5 | 4.433 | 4.427 | −0.1 |
| 8 | 2.142 | 2.095 | −2.2 |

Common-random-number pairing, vehicle conservation and the fare-acceptance accounting identity are
asserted every episode. Across 5,265 adjacent-fleet comparisons, seed-level monotonicity reversals
occur in 1.20% of served-volume comparisons and 0.44% of 90th-percentile-wait comparisons; the
replication-aggregated surfaces are monotone and the suffix rule prevents reversals from defining
a boundary.

## Replication

The replication package exists and its source-file hashes are recorded in the manuscript's
supplementary material. Its structure, for release:

```
environment/     pinned Python version and dependency lock
data/raw/        redistributable sources, or access instructions and source identifiers
data/processed/  hourly demand, enriched requests, depot and zone inputs, checksums
src/             config, simulator, fare rules, grid runner, calibration, validation, analysis
manifests/       exact cell manifests per stage, including the frozen confirmation manifest
results/         seed-level summaries and derived decision tables
logs/            run metadata, failures, completion checks
```

Discrete outputs are exact replication targets: the nine-fare minimum-fleet frontiers, the
selected policy and fleet identities, the twenty frozen comparison identities, feasibility
categories and vehicles-replaced values must match exactly. A requirement of 40 vehicles does not
reproduce a requirement of 41. Continuous monetary outputs must match within the greater of €1 and
0.1%; reliability means within 0.001 minute or 0.001 percentage point.

**Data availability.** Processed inputs derive from operator-supplied pilot data and are not
redistributable in full. The repository ships checksums and access instructions so a replication
can verify it is working from identical inputs. Operating and vehicle capital cost inputs come
from the ULTIMO AV Fleet and Cost Calculator (Beye et al., 2025).

## Citation

```bibtex
@unpublished{zigah_fare_fleet,
  author = {Zigah, Elias},
  title  = {How much fleet capacity can regulated fare flexibility replace
            under a service reliability standard?},
  note   = {Under review, Transportation Research Part E},
  year   = {2026}
}
```
