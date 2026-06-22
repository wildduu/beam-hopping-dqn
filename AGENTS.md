# AGENTS.md

## Project Role

This repository implements a research simulation environment for Starlink-like LEO single-satellite beam hopping scheduling with DQN-based algorithms.

The project must proceed in stages:

1. Documentation
2. Environment implementation
3. Unit tests
4. Heuristic baselines
5. DQN baseline
6. Sequential DQN
7. Multi-exit sequential DQN

Do not implement later stages before earlier stages are complete and tested.

---

## Hard Rules

### Do Not Change Core Scenario Parameters

Do not change the following parameters unless explicitly instructed:

* Satellite number: 1
* Orbit altitude: 550 km
* Orbit inclination: 53 degrees
* User downlink frequency: 20 GHz Ka-band
* User downlink bandwidth: 200 MHz
* Feeder uplink frequency: 50 GHz V-band
* Feeder uplink bandwidth: 1 GHz
* Slot duration: 2 ms
* Active beams per slot: K = 4
* Debug visible cells: 19
* Main visible cells: 61
* Scalability visible cells: 96 and 128
* Debug episode length: 128 slots
* Formal training episode length: 1000 slots
* Evaluation length: 2000 slots
* Cell radius: 25 km
* Packet size: 2000 bits
* RT TTL: 40 slots
* NRT buffer limit: 50 Mbit

---

## Modeling Rules

* The whole simulation is cell-level.
* Actions select cells.
* Queue states are cell-level.
* Channel gains are beam-to-cell / cell-level.
* GNN nodes are cells.
* GNN edges represent normalized potential inter-beam interference.
* Reward is computed from cell-level throughput, RT delay, packet drop, and fairness.
* All internal queue and capacity units must be bits.
* Packet arrivals may be sampled as packet counts, but must be converted to bits before queue updates.
* Small-scale fading such as Rician or shadowed-Rician must not be implemented in the first version.
* The user link is Ka-band downlink.
* The feeder link is V-band gateway-to-satellite uplink.
* The feeder link only controls how much gateway backlog enters satellite queues.
* Feeder gateway backlog must not enter the main reward.
* No ISL routing, handover, or invisibility-induced packet loss in the first version.
* All candidate cells are visible in the main experiment, but a visibility mask must still be implemented for future extension.
* Do not add Stop action in the first version.
* Each slot must select exactly K legal cells.

---

## Satellite Motion

The satellite follows a LEO orbit, but within the short scheduling window its ground track is approximated by a local tangent line.

At every slot, update:

* satellite ground-track position
* satellite-cell distance
* beam off-axis angle
* antenna gain
* channel gain
* SINR
* downlink rate

This local tangent approximation is used only to capture short-term LEO channel dynamics.

---

## Testing Rules

Before claiming a task is complete:

* Run all available tests.
* Verify that reset and step functions work.
* Verify all tensor and array shapes.
* Verify queue updates.
* Verify RT TTL expiration.
* Verify NRT buffer overflow.
* Verify action masks.
* Verify channel gains and SINR are finite.
* Verify reward is finite.
* Verify random rollout runs without crashing.

---

## Implementation Order

Do not implement DQN before the environment and tests are complete.

The required order is:

1. Write documentation.
2. Implement environment only.
3. Add unit tests.
4. Implement heuristic baselines.
5. Implement DQN baseline.
6. Implement sequential DQN.
7. Implement multi-exit sequential DQN.

---

## Output Rules

For every experiment, save:

* config file used
* random seed
* metrics CSV
* model checkpoint, if applicable
* training curve, if applicable
* evaluation summary

Never report performance without saving the raw metrics.
