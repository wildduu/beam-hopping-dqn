# Environment Specification

This document defines the environment update logic used by the simulator.

The purpose of this file is implementation guidance.

All queues, capacities, throughput, service amounts, and packet drops use bits as the internal unit.

Packet arrivals may be sampled as packet counts but must be converted into bits before queue updates.

---

# 1. Environment State

The environment state contains the following components.

## Traffic State

For each cell i:

* RT arrival rate lambda_RT_i
* NRT arrival rate lambda_NRT_i

---

## Gateway State

For each cell i:

* Gateway RT backlog Q_gw_RT_i
* Gateway NRT backlog Q_gw_NRT_i

---

## Satellite State

For each cell i:

* RT waiting-time queue Phi_i
* Satellite NRT queue Q_sat_NRT_i

---

## Channel State

For each cell i:

* Distance d_i
* Off-axis angle theta_i
* Ka rain attenuation A_rain_Ka_i

Global variables:

* V-band rain attenuation A_rain_V
* Feeder capacity C_feed

---

## Geometry State

* Satellite position
* Cell coordinates
* Visibility mask

---

# 2. Environment Timeline

Every slot executes the following sequence.

Step 1:

Generate new RT and NRT traffic.

Step 2:

Add traffic into gateway queues.

Step 3:

Update V-band feeder channel.

Step 4:

Calculate feeder capacity.

Step 5:

Upload traffic from gateway queues to satellite queues.

Step 6:

Update satellite position.

Step 7:

Update Ka-band channels.

Step 8:

Apply beam scheduling action.

Step 9:

Serve selected cells.

Step 10:

Update RT waiting layers.

Step 11:

Update NRT queue.

Step 12:

Calculate packet drops.

Step 13:

Calculate reward.

Step 14:

Generate next state.

---

# 3. Traffic Generation

## Inputs

For each cell:

* lambda_RT_i
* lambda_NRT_i

Parameters:

* Packet size = 2000 bits

---

## Traffic Rates

Initial RT arrival rate:

Uniform(2,10) Mbps

Initial NRT arrival rate:

Uniform(10,70) Mbps

---

## Traffic Update

Every 200 slots:

RT:

lambda_RT_i += Uniform(-1,1)

Clip into:

[2,10] Mbps

NRT:

lambda_NRT_i += Uniform(-5,5)

Clip into:

[10,70] Mbps

---

## Packet Generation

RT packets:

Poisson arrival

NRT packets:

Poisson arrival

Generated packet counts must be converted into bits.

Outputs:

* A_RT_i
* A_NRT_i

---

# 4. Gateway Queues

## Variables

Q_gw_RT_i

Q_gw_NRT_i

---

## Update

New arrivals are added:

Q_gw_RT_i += A_RT_i

Q_gw_NRT_i += A_NRT_i

---

## Notes

Gateway backlog is not counted as packet drop.

Gateway backlog does not enter reward calculation.

Gateway backlog is reported as an evaluation metric.

---

# 5. Feeder Link

## Purpose

Transfer traffic from gateway queues to satellite queues.

---

## Parameters

Frequency:

50 GHz

Bandwidth:

1 GHz

Clear-sky SNR:

20 dB

---

## V-band Rain

Initial value:

Uniform(0,6) dB

Every 200 slots:

Rain += Uniform(-1,1)

Clip into:

[0,20] dB

---

## Feeder Capacity

Inputs:

* feeder bandwidth
* feeder SNR

Process:

1. Calculate current feeder SNR.
2. Calculate Shannon capacity.
3. Convert capacity into bits available during one slot.

Output:

feeder_bits_available

---

# 6. Feeder Upload Rule

RT traffic has priority.

---

## RT Upload

Available feeder bits are first allocated to RT traffic.

Allocation is proportional to RT gateway backlog.

Output:

uploaded_RT_i

---

## NRT Upload

Remaining feeder bits are allocated to NRT traffic.

Allocation is proportional to NRT gateway backlog.

Output:

uploaded_NRT_i

---

## Queue Update

Gateway queues decrease.

Satellite queues receive uploaded traffic.

---

# 7. Satellite RT Queue

## Structure

RT queue uses waiting-time layers.

Number of layers:

40

Layer index:

0 ~ 39

---

## New Traffic

New RT traffic enters layer 0.

---

## Waiting-Time Update

Every slot:

All unserved RT traffic shifts one layer forward.

---

## RT Drop

Traffic remaining in layer 39 after waiting update is dropped.

RT drop reason:

TTL expiration.

---

## Output

RT dropped bits.

---

# 8. Satellite NRT Queue

## Structure

FIFO queue.

Maximum buffer:

50 Mbit

---

## New Traffic

New NRT traffic enters the queue.

---

## Overflow

Traffic exceeding buffer limit is dropped.

Drop reason:

Buffer overflow.

---

## Output

NRT dropped bits.

---

# 9. Satellite Motion

## Orbit

Altitude:

550 km

Inclination:

53 degrees

---

## Motion Model

Use local tangent approximation.

Within one scheduling episode:

Satellite moves along a straight ground track.

---

## Update

Every slot update:

* satellite position
* distance to every cell
* off-axis angle to every cell

Outputs:

* d_i
* theta_i

---

# 10. Visibility

Main experiment assumption:

All candidate cells are visible.

Visibility mask still exists.

Purpose:

Future extension.

---

# 11. Ka-band Rain

## Initial Value

For every cell:

Uniform(0,3) dB

---

## Update

Every 200 slots:

Rain += Uniform(-0.5,0.5)

Clip into:

[0,10] dB

---

## Output

A_rain_Ka_i

---

# 12. Antenna Pattern

## Parameters

Maximum antenna gain:

29.43 dBi

3 dB beamwidth:

5.25 degrees

3 dB half-angle:

2.625 degrees

---

## Inputs

Off-axis angle.

---

## Output

Transmit antenna gain.

Gain decreases as off-axis angle increases.

---

# 13. Channel Gain

## Inputs

For every cell:

* distance
* off-axis angle
* Ka rain attenuation

---

## Process

Calculate:

1. Free-space path loss.
2. Antenna gain.
3. Atmospheric loss.
4. Ka rain attenuation.

Combine them into large-scale channel gain.

---

## Output

channel_gain_i

---

# 14. Interference

Only active beams generate interference.

Interference depends on:

* beam directions
* off-axis gains
* channel gains

---

## Output

interference_i

---

# 15. SINR

Inputs:

* signal power
* interference power
* noise power

Noise power:

Computed from:

* bandwidth
* temperature
* noise figure

Output:

SINR_i

---

# 16. Downlink Rate

Inputs:

SINR_i

Bandwidth:

200 MHz

Process:

Shannon formula.

Output:

rate_i

Unit:

bits/s

---

# 17. Service

Only selected cells receive service.

For selected cell i:

service_bits_i

=

rate_i × slot_duration

---

## RT Service

Serve oldest RT traffic first.

Priority:

Layer 39 → Layer 0

---

## NRT Service

Remaining service capacity is allocated to NRT queue.

---

# 18. GNN Graph

## Node

One node = one cell.

---

## Edge

One edge = potential interference relationship.

---

## Edge Weight

Static normalized interference coefficient.

Not affected by rain attenuation.

Not affected by queue state.

---

# 19. Reward

Reward contains four components.

---

## Throughput Term

Encourage higher throughput.

Positive reward.

---

## RT Delay Term

Penalize large RT waiting time.

Negative reward.

---

## Packet Drop Term

Penalize:

* RT TTL drop
* NRT overflow drop

Negative reward.

---

## Fairness Term

Encourage balanced service among cells.

Positive reward.

---

## Final Reward

Weighted sum of:

* throughput
* delay
* packet drop
* fairness

Output:

r_t

---

# 20. Evaluation Metrics

The simulator must record:

Performance metrics:

* throughput
* RT delay
* RT drop
* NRT drop
* fairness

Queue metrics:

* gateway backlog
* satellite backlog

Complexity metrics:

* runtime
* FLOPs
* parameter count
* memory usage

Additional metrics:

* DQN loss
* episode reward

Multi-exit metrics:

* Exit0 ratio
* Exit1 ratio
* Exit2 ratio
* average inference latency
* average FLOPs

All metrics must be exportable to CSV files.
