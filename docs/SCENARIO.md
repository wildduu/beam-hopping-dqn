# docs/SCENARIO.md

# Scenario Design

## 1. Core Scenario

This project considers a Starlink-like LEO single-satellite Ka-band user downlink beam hopping scheduling scenario.

The feeder link is modeled as a V-band gateway-to-satellite uplink. It only controls how much traffic enters the satellite-side queues.

The user link is modeled as a Ka-band cell-level downlink.

All traffic, queues, channel gains, actions, graph nodes, and rewards are defined at the cell level.

---

## 2. Scenario Assumptions

| Item                             | Setting                                        |
| -------------------------------- | ---------------------------------------------- |
| Satellite number                 | 1                                              |
| Link direction                   | Forward link / downlink                        |
| Scheduling type                  | Snapshot-based single-satellite beam hopping   |
| User link                        | Ka-band downlink                               |
| Feeder link                      | V-band gateway-to-satellite uplink             |
| Cell type                        | Earth-fixed traffic aggregation cell           |
| Candidate cells                  | All visible during the short scheduling window |
| Invisibility-induced packet loss | Not considered                                 |
| ISL routing                      | Not considered                                 |
| Handover                         | Not considered                                 |
| Stop action                      | Not used in the first version                  |

---

## 3. Orbit and Motion

| Parameter          |  Symbol |      Value |
| ------------------ | ------: | ---------: |
| Altitude           |       h |     550 km |
| Inclination        |       i | 53 degrees |
| Slot duration      | Delta t |       2 ms |
| Ground-track speed |     v_g |     7 km/s |

The satellite follows a LEO orbit. Within each short scheduling episode, the satellite ground track is approximated by a local tangent line.

The satellite position is updated at every slot:

```text
x_sat(t) = x_0 + v_g * t * Delta t
y_sat(t) = 0
z_sat(t) = h
```

This approximation is used to update the satellite-cell distance and beam off-axis angle at every slot.

---

## 4. Cell Configuration

| Parameter                 | Symbol |    Value |
| ------------------------- | -----: | -------: |
| Cell radius               | r_cell |    25 km |
| Debug visible cells       |  N_vis |       19 |
| Main visible cells        |  N_vis |       61 |
| Scalability visible cells |  N_vis | 96 / 128 |
| Active beams per slot     |      K |        4 |

The ground service region is represented by a local hexagonal grid.

Each cell is an earth-fixed traffic aggregation cell. A cell is not a village, not a single user, and not an administrative region.

The main experiment uses 61 visible cells. The 19-cell setting is used for debugging and small-scale DQN validation. The 96-cell and 128-cell settings are used for scalability tests.

---

## 5. Cell Radius and Antenna

| Parameter                   |    Symbol |         Value |
| --------------------------- | --------: | ------------: |
| User link frequency         |      f_Ka |        20 GHz |
| Wavelength                  | lambda_Ka |       0.015 m |
| Satellite aperture radius   |         a |         0.1 m |
| Satellite aperture diameter |         D |         0.2 m |
| 3 dB beamwidth              | theta_3dB |  5.25 degrees |
| 3 dB half-angle             |   theta_b | 2.625 degrees |
| Satellite antenna max gain  |   G_T_max |     29.43 dBi |

The 3 dB beamwidth is calculated by:

```text
theta_3dB = 70 * lambda / D
```

With lambda = 0.015 m and D = 0.2 m:

```text
theta_3dB = 70 * 0.015 / 0.2 = 5.25 degrees
```

The cell radius is approximated by the ground projection of the 3 dB half-beamwidth:

```text
r_cell = h * tan(theta_3dB / 2)
```

With h = 550 km:

```text
r_cell = 550 * tan(2.625 degrees) ≈ 25.2 km
```

The final value is:

```text
r_cell = 25 km
```

---

## 6. User Downlink Parameters

| Parameter                     | Symbol |                Value |
| ----------------------------- | -----: | -------------------: |
| User downlink frequency       |   f_Ka |               20 GHz |
| User downlink bandwidth       | B_user |              200 MHz |
| Total satellite power         |  P_sat |               13 dBW |
| Active beams                  |      K |                    4 |
| Per-beam power                |    P_b |             6.98 dBW |
| Frequency reuse               |        | Full frequency reuse |
| User receive gain             |    G_R |               30 dBi |
| Noise temperature             |    T_n |                290 K |
| Noise figure                  |     NF |                 3 dB |
| Noise power                   | sigma2 |           -118.0 dBW |
| Fixed gas/implementation loss |  L_gas |                 1 dB |

Per-beam power:

```text
P_b = P_sat - 10 * log10(K)
P_b = 13 - 10 * log10(4) = 6.98 dBW
```

---

## 7. Ka-Band Rain Attenuation

Ka rain is modeled as a cell-level slow random walk.

| Parameter     |       Symbol |                 Value |
| ------------- | -----------: | --------------------: |
| Initial value | A_rain_Ka(0) |      Uniform(0, 3) dB |
| Update period |       T_rain |             200 slots |
| Step          |   Delta A_Ka | Uniform(-0.5, 0.5) dB |
| Bound         |              |            [0, 10] dB |

Update rule:

```text
A_rain_Ka_i(t + T_rain)
=
clip(A_rain_Ka_i(t) + Delta A_Ka_i(t), 0, 10)
```

---

## 8. Feeder Uplink Parameters

| Parameter            |     Symbol |                       Value |
| -------------------- | ---------: | --------------------------: |
| Feeder direction     |            | Gateway-to-satellite uplink |
| Feeder band          |            |                      V-band |
| Feeder frequency     |     f_feed |                      50 GHz |
| Feeder bandwidth     |     B_feed |                       1 GHz |
| Clear-sky feeder SNR | SNR_feed_0 |                       20 dB |

V-band rain is modeled as a global slow random walk.

| Parameter     |      Symbol |             Value |
| ------------- | ----------: | ----------------: |
| Initial value | A_rain_V(0) |  Uniform(0, 6) dB |
| Update period |      T_rain |         200 slots |
| Step          |   Delta A_V | Uniform(-1, 1) dB |
| Bound         |             |        [0, 20] dB |

Feeder capacity:

```text
C_feed(t)
=
B_feed * log2(1 + 10^((SNR_feed_0 - A_rain_V(t)) / 10))
```

---

## 9. Traffic Parameters

| Parameter                |           Symbol |                     Value |
| ------------------------ | ---------------: | ------------------------: |
| Traffic classes          |                  |                RT and NRT |
| RT initial arrival rate  |   lambda_RT_i(0) |  Uniform(2, 10) Mbps/cell |
| NRT initial arrival rate |  lambda_NRT_i(0) | Uniform(10, 70) Mbps/cell |
| Arrival process          |                  |                   Poisson |
| Packet size              |              L_p |                 2000 bits |
| Traffic update period    |        T_traffic |                 200 slots |
| RT random-walk step      |  Delta lambda_RT |       Uniform(-1, 1) Mbps |
| NRT random-walk step     | Delta lambda_NRT |       Uniform(-5, 5) Mbps |

RT arrival rate update:

```text
lambda_RT_i(t + 200)
=
clip(lambda_RT_i(t) + Delta lambda_RT_i(t), 2, 10)
```

NRT arrival rate update:

```text
lambda_NRT_i(t + 200)
=
clip(lambda_NRT_i(t) + Delta lambda_NRT_i(t), 10, 70)
```

---

## 10. Queue Parameters

| Parameter          |    Symbol |                       Value |
| ------------------ | --------: | --------------------------: |
| Queue unit         |           |                        bits |
| Gateway queue      |           |         RT and NRT per cell |
| Satellite queue    |           |         RT and NRT per cell |
| RT queue structure |           | 40-layer waiting-time queue |
| RT layer index     |           |                     0 to 39 |
| RT TTL             |     T_TTL |            40 slots = 80 ms |
| NRT queue type     |           |                        FIFO |
| NRT max buffer     | Q_NRT_max |                     50 Mbit |

Packet loss sources:

| Drop type         | Cause                         |
| ----------------- | ----------------------------- |
| RT drop           | TTL expiration                |
| NRT drop          | Satellite NRT buffer overflow |
| Invisibility drop | Not considered                |
| Feeder drop       | Not considered in main reward |

---

## 11. Training and Evaluation Length

| Stage                   |     Length |
| ----------------------- | ---------: |
| Debug episode           |  128 slots |
| Formal training episode | 1000 slots |
| Evaluation rollout      | 2000 slots |

The debug setting is only used for quick environment validation.

The formal training setting uses 1000 slots so that rain and traffic random walks update multiple times within one episode.

The evaluation setting uses 2000 slots to compute stable average metrics.
