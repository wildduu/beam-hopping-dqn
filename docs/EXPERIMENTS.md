# Experiments

# Experimental Objectives

The experiments aim to answer the following questions.

Q1.

Can the simulation environment produce reasonable scheduling behavior?

Q2.

Can DQN outperform heuristic scheduling policies?

Q3.

Can sequential beam selection reduce decision complexity while maintaining performance?

Q4.

Can multi-exit inference reduce computation cost while maintaining performance?

Q5.

How does the proposed method scale with increasing cell numbers?

---

# Experimental Scenarios

Three scenario scales are considered.

## Debug Scenario

Purpose:

* Environment verification
* DQN debugging

Parameters:

| Item           | Value     |
| -------------- | --------- |
| Visible cells  | 19        |
| Active beams   | 4         |
| Episode length | 128 slots |

---

## Main Scenario

Purpose:

* Main performance comparison

Parameters:

| Item           | Value      |
| -------------- | ---------- |
| Visible cells  | 61         |
| Active beams   | 4          |
| Episode length | 1000 slots |

---

## Scalability Scenarios

Purpose:

* Complexity evaluation
* Scalability evaluation

Parameters:

| Item           | Value      |
| -------------- | ---------- |
| Visible cells  | 96         |
| Active beams   | 4          |
| Episode length | 1000 slots |

and

| Item           | Value      |
| -------------- | ---------- |
| Visible cells  | 128        |
| Active beams   | 4          |
| Episode length | 1000 slots |

---

# Evaluation Length

All evaluation rollouts use:

```text
2000 slots
```

unless explicitly stated otherwise.

---

# Random Seeds

Every experiment must be repeated using multiple seeds.

Recommended seeds:

```text
0
1
2
3
4
```

Minimum requirement:

```text
3 seeds
```

Preferred:

```text
5 seeds
```

Report:

* Mean
* Standard deviation

---

# Baseline Algorithms

## Random

Randomly select K legal cells.

Abbreviation:

```text
RAND
```

---

## Max Queue

Select cells with largest backlog.

Abbreviation:

```text
MAXQ
```

---

## Max Delay

Select cells with largest RT urgency.

Abbreviation:

```text
MAXD
```

---

## Max Rate

Select cells with highest instantaneous rate.

Abbreviation:

```text
MAXR
```

---

## DQN

Standard DQN baseline.

Abbreviation:

```text
DQN
```

---

## Sequential DQN

Sequential beam selection.

Abbreviation:

```text
SEQ-DQN
```

---

## Multi-Exit Sequential DQN

Proposed method.

Abbreviation:

```text
ME-SEQ-DQN
```

---

# Performance Metrics

## Throughput

Measure:

```text
Average served bits per slot
```

Units:

```text
Mbps
```

Store:

```text
throughput.csv
```

---

## RT Delay

Measure:

```text
Average waiting time
```

Units:

```text
slots
```

Store:

```text
rt_delay.csv
```

---

## RT Packet Drop

Measure:

```text
RT traffic lost due to TTL expiration
```

Units:

```text
bits
```

Store:

```text
rt_drop.csv
```

---

## NRT Packet Drop

Measure:

```text
NRT traffic lost due to buffer overflow
```

Units:

```text
bits
```

Store:

```text
nrt_drop.csv
```

---

## Fairness

Metric:

```text
Jain Fairness Index
```

Range:

```text
0 to 1
```

Store:

```text
fairness.csv
```

---

## Average Gateway Backlog

Measure:

```text
Mean gateway queue size
```

Units:

```text
bits
```

Store:

```text
gateway_backlog.csv
```

---

## Average Satellite Backlog

Measure:

```text
Mean satellite queue size
```

Units:

```text
bits
```

Store:

```text
satellite_backlog.csv
```

---

# Computational Metrics

The following metrics are mandatory.

---

## Runtime

Measure:

```text
Average decision time per slot
```

Units:

```text
milliseconds
```

Store:

```text
runtime.csv
```

---

## FLOPs

Measure:

```text
Estimated inference FLOPs
```

Store:

```text
flops.csv
```

---

## Model Parameters

Measure:

```text
Number of trainable parameters
```

Store:

```text
parameters.csv
```

---

## Memory Usage

Measure:

```text
Model size
```

Units:

```text
MB
```

Store:

```text
memory.csv
```

---

# DQN Experiments

## Experiment D1

Objective:

Verify DQN beats heuristic baselines.

Compare:

* RAND
* MAXQ
* MAXD
* MAXR
* DQN

Scenario:

```text
61 cells
```

---

## Experiment D2

Objective:

Training convergence.

Metrics:

* Episode reward
* Loss
* Epsilon

Store:

```text
training_curve.png
```

---

# Sequential DQN Experiments

## Experiment S1

Objective:

Performance comparison.

Compare:

* DQN
* SEQ-DQN

Metrics:

* Throughput
* Delay
* Packet drop

---

## Experiment S2

Objective:

Complexity comparison.

Compare:

* DQN
* SEQ-DQN

Metrics:

* Runtime
* FLOPs
* Parameters

---

# Multi-Exit Experiments

## Experiment M1

Objective:

Performance comparison.

Compare:

* SEQ-DQN
* ME-SEQ-DQN

Metrics:

* Throughput
* Delay
* Packet drop

---

## Experiment M2

Objective:

Inference cost reduction.

Metrics:

* Runtime
* FLOPs

---

## Experiment M3

Objective:

Exit usage statistics.

Metrics:

* Exit0 ratio
* Exit1 ratio
* Exit2 ratio

Store:

```text
exit_statistics.csv
```

---

# Ablation Studies

## A1

Exit0 removed.

Compare:

* Exit1 + Exit2
* Exit0 + Exit1 + Exit2

---

## A2

Exit1 removed.

Compare:

* Exit2 only
* Exit1 + Exit2

---

## A3

Different cell numbers.

Compare:

* 19
* 61
* 96
* 128

---

## A4

Different traffic loads.

Traffic settings:

### Low

RT:

```text
2 Mbps
```

NRT:

```text
10 Mbps
```

---

### Medium

RT:

```text
6 Mbps
```

NRT:

```text
40 Mbps
```

---

### High

RT:

```text
10 Mbps
```

NRT:

```text
70 Mbps
```

---

# Figures Required For Paper

The final implementation should automatically generate the following figures.

Figure 1:

Training reward curve.

Figure 2:

Throughput comparison.

Figure 3:

Delay comparison.

Figure 4:

Packet drop comparison.

Figure 5:

Fairness comparison.

Figure 6:

Runtime comparison.

Figure 7:

FLOPs comparison.

Figure 8:

Scalability comparison.

Figure 9:

Exit ratio distribution.

---

# Final Experimental Goal

Demonstrate that:

1. DQN outperforms heuristic policies.

2. Sequential DQN reduces output-space complexity.

3. Multi-exit DQN reduces inference cost.

4. The proposed method remains effective as the number of cells increases.
