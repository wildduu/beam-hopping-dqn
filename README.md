# README.md

# LEO Beam Hopping DQN

This repository studies lightweight DQN-based beam hopping scheduling for a Starlink-like LEO single-satellite system.

The main goal is to first build a reliable simulation environment and DQN baseline, then extend it with sequential beam selection and multi-exit inference.

---

## Scenario Summary

The system is a single LEO multi-beam satellite serving multiple earth-fixed traffic aggregation cells.

At each slot, the satellite activates at most K = 4 Ka-band user beams and selects 4 visible cells for downlink service.

The feeder link is modeled separately as a V-band gateway-to-satellite uplink. It only controls how much traffic enters the satellite-side queues.

The simulation is cell-level:

* one node = one traffic cell
* one action = selecting one or more cells
* one queue = one cell-level RT/NRT queue
* one channel = beam-to-cell channel
* one graph edge = potential inter-beam interference

---

## Main Parameters

| Item                    |      Value |
| ----------------------- | ---------: |
| Satellite number        |          1 |
| Altitude                |     550 km |
| Inclination             | 53 degrees |
| User downlink band      |    Ka-band |
| User downlink frequency |     20 GHz |
| User downlink bandwidth |    200 MHz |
| Feeder uplink band      |     V-band |
| Feeder uplink frequency |     50 GHz |
| Feeder uplink bandwidth |      1 GHz |
| Slot duration           |       2 ms |
| Active beams per slot   |          4 |
| Debug visible cells     |         19 |
| Main visible cells      |         61 |
| Scalability cells       |   96 / 128 |
| Cell radius             |      25 km |
| Packet size             |  2000 bits |
| RT TTL                  |   40 slots |
| NRT max buffer          |    50 Mbit |

---

## Development Stages

### Stage 1: Documentation

Prepare:

* `AGENTS.md`
* `docs/SCENARIO.md`
* `docs/EQUATIONS.md`
* `docs/IMPLEMENTATION_PLAN.md`
* `docs/EXPERIMENTS.md`

### Stage 2: Environment

Implement:

* cell geometry
* satellite local tangent motion
* Ka-band downlink channel
* V-band feeder uplink
* RT/NRT queues
* rain attenuation
* action mask
* reward

### Stage 3: Tests

Add unit tests for:

* environment reset
* environment step
* queue update
* action mask
* channel and SINR
* reward calculation

### Stage 4: Heuristic Baselines

Implement:

* random beam hopping
* max-queue policy
* max-delay policy
* max-rate policy

### Stage 5: DQN Baseline

Implement a basic DQN baseline.

For large cell numbers, avoid full combination DQN because the action space grows combinatorially.

### Stage 6: Sequential DQN

Implement sequential cell selection: select one cell per step and repeat K times.

### Stage 7: Multi-Exit Sequential DQN

Add shallow/deep exits and Exit0 reuse.

---

## Important Notes

The first version does not include:

* ISL routing
* handover
* dynamic visibility loss
* Stop action
* Rician fading
* shadowed-Rician fading
* power allocation
* bandwidth allocation

The first version focuses on a clean and testable beam hopping environment.
