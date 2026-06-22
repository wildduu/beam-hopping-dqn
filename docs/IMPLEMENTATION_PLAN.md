# Implementation Plan

# Project Goal

Build a complete simulation platform for LEO satellite beam hopping scheduling.

The implementation must proceed incrementally:

1. Environment
2. Tests
3. Heuristic baselines
4. DQN baseline
5. Sequential DQN
6. Multi-exit DQN

The objective is to ensure that every stage is validated before adding additional complexity.

---

# Stage 0: Documentation

## Objective

Create a clear and stable specification for future development.

## Required Files

* AGENTS.md
* SCENARIO.md
* EQUATIONS.md
* IMPLEMENTATION_PLAN.md
* EXPERIMENTS.md

## Completion Criteria

* All scenario parameters fixed.
* Queue definitions fixed.
* Reward definition fixed.
* Evaluation metrics fixed.
* No implementation ambiguity remains.

---

# Stage 1: Environment Construction

## Objective

Build the beam hopping simulation environment.

## Required Components

### Geometry Module

Implement:

* Cell coordinates
* Hexagonal grid generation
* Satellite position update
* Distance calculation
* Off-axis angle calculation

Files:

```text
src/utils/geometry.py
```

---

### Traffic Module

Implement:

* RT arrival generation
* NRT arrival generation
* Traffic random walk updates

Files:

```text
src/utils/traffic.py
```

---

### Rain Module

Implement:

* Ka rain attenuation
* V-band rain attenuation
* Random walk updates

Files:

```text
src/utils/channel.py
```

---

### Queue Module

Implement:

* Gateway RT queue
* Gateway NRT queue
* Satellite RT queue
* Satellite NRT queue
* RT TTL expiration
* NRT overflow

Files:

```text
src/utils/queue_manager.py
```

---

### Feeder Module

Implement:

* Feeder capacity calculation
* RT-priority feeder upload
* NRT upload

Files:

```text
src/utils/feeder.py
```

---

### Environment Module

Implement:

* reset()
* step()
* reward calculation
* state generation
* action validation

Files:

```text
src/envs/beam_hopping_env.py
```

## Completion Criteria

Environment can:

* Reset correctly.
* Run at least 100 slots.
* Produce finite rewards.
* Produce finite SINR.
* Produce finite throughput.

---

# Stage 2: Unit Tests

## Objective

Verify environment correctness.

## Required Tests

### Environment Tests

File:

```text
tests/test_env.py
```

Verify:

* reset works
* step works
* output shapes correct

---

### Queue Tests

File:

```text
tests/test_queue.py
```

Verify:

* RT TTL expiration
* NRT overflow
* feeder upload

---

### Channel Tests

File:

```text
tests/test_channel.py
```

Verify:

* positive distance
* finite gain
* finite SINR
* finite rate

---

### Action Tests

File:

```text
tests/test_action.py
```

Verify:

* duplicate actions rejected
* invalid actions rejected
* action mask works

## Completion Criteria

All tests pass.

---

# Stage 3: Heuristic Baselines

## Objective

Verify environment behavior before introducing reinforcement learning.

## Baseline 1: Random

Select K legal cells uniformly.

File:

```text
src/baselines/random_policy.py
```

---

## Baseline 2: Max Queue

Select cells with largest backlog.

File:

```text
src/baselines/max_queue.py
```

---

## Baseline 3: Max Delay

Select cells with largest RT urgency.

File:

```text
src/baselines/max_delay.py
```

---

## Baseline 4: Max Rate

Select cells with largest achievable rate.

File:

```text
src/baselines/max_rate.py
```

---

## Evaluation

Run:

* 19-cell scenario
* 61-cell scenario

Store:

```text
results/baselines/
```

Metrics:

* Throughput
* RT delay
* RT drop
* NRT drop
* Fairness
* Runtime

## Completion Criteria

Results are reproducible and saved.

---

# Stage 4: DQN Baseline

## Objective

Implement a standard DQN benchmark.

## Network

Input:

Environment state vector.

Output:

One score per candidate cell.

Example:

```text
61 cells -> 61 outputs
```

Top-K selection:

Select K highest-scoring cells.

## Required Components

Files:

```text
src/agents/dqn.py
src/models/mlp_qnet.py
src/train_dqn.py
```

Implement:

* Replay buffer
* Target network
* Epsilon-greedy
* Checkpoint saving
* TensorBoard logging

## Completion Criteria

Training converges.

Reward improves over Random.

---

# Stage 5: Sequential DQN

## Objective

Reduce output dimension by sequential beam selection.

## Method

Instead of selecting K cells simultaneously:

Step 1:

Select one cell.

Step 2:

Mask selected cell.

Step 3:

Select next cell.

Repeat K times.

## Required Components

Files:

```text
src/agents/sequential_dqn.py
```

## New Metrics

Record:

* Decision latency
* FLOPs
* Throughput

## Completion Criteria

Performance close to DQN baseline.

---

# Stage 6: Multi-Exit Sequential DQN

## Objective

Reduce inference cost.

## Network Structure

Exit0:

Reuse previous action.

No network execution.

---

Exit1:

Shallow network.

---

Exit2:

Full network.

---

## Required Components

Files:

```text
src/models/multi_exit_qnet.py
src/agents/multi_exit_dqn.py
```

Implement:

* Exit classifier
* Exit routing
* Exit statistics

## Metrics

Record:

* Exit0 ratio
* Exit1 ratio
* Exit2 ratio
* Average FLOPs
* Average inference latency
* Throughput
* RT delay
* Packet drop

## Completion Criteria

Average inference cost reduced.

Performance degradation remains acceptable.

---

# Stage 7: Ablation Study

## Experiments

1. Random
2. Max Queue
3. Max Delay
4. Max Rate
5. DQN
6. Sequential DQN
7. Multi-Exit Sequential DQN

---

## Additional Ablations

### Exit Study

Compare:

* Exit2 only
* Exit1 + Exit2
* Exit0 + Exit1 + Exit2

---

### Scale Study

Compare:

* 19 cells
* 61 cells
* 96 cells
* 128 cells

---

### Traffic Study

Compare:

* Low load
* Medium load
* High load

---

# Final Goal

Produce a complete experimental pipeline that supports:

* Traditional DQN
* Sequential DQN
* Multi-exit DQN

under a common LEO satellite beam hopping environment.
