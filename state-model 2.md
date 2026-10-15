# Task 3 — State Modelling & Transition Verification

## 3.1 State definitions

| State ID | State | Meaning | Entry condition | Exit condition |
|---|---|---|---|---|
| ST-00 | OFF | Robot is powered off. | Power is off. | PowerOn event. |
| ST-01 | IDLE | Robot is waiting for a valid delivery request. | Power-on checks complete or robot arrives at warehouse. | Valid DeliveryRequest or shutdown. |
| ST-02 | NAVIGATING | Robot is travelling toward the clinic. | Valid request accepted. | Obstacle, destination reached, critical condition, or link-loss policy. |
| ST-03 | AVOIDING_OBSTACLE | Normal route progress is suspended while obstacle is handled. | ObstacleDetected during navigation. | ObstacleCleared, critical condition, or link-loss policy. |
| ST-04 | DELIVERING | Robot performs package handover at the destination. | DestinationReached from NAVIGATING and delivery safety checks pass. | DeliveryComplete or critical condition. |
| ST-05 | RETURNING | Robot is travelling back to the warehouse. | Delivery complete or delivery abandoned due to critical battery. | WarehouseReached or safety escalation. |
| ST-06 | SAFE_MODE | Safety response limits normal operation after a critical fault. | Critical battery/temperature/fault condition. | Fault cleared and safety checks pass under an authorised recovery procedure. |
| ST-07 | LINK_LOST | Remote communication is unavailable or exceeds the approved timeout. | Link-loss detector triggers in a state requiring remote communication. | Link restored and commands/status are reconciled safely. |

## 3.2 Events and conditions

- `PowerOn`: robot power becomes available.
- `DeliveryRequest`: a delivery request is received.
- `ObstacleDetected`: obstacle sensor reports an obstacle.
- `ObstacleCleared`: sensors confirm the path is clear.
- `DestinationReached`: destination geofence/location is reached.
- `DeliveryComplete`: package handover is confirmed.
- `CriticalBattery`: battery capacity falls below the emergency level.
- `CriticalTemperature`: chamber or battery temperature exceeds its critical threshold.
- `WarehouseReached`: robot reaches the warehouse.
- `LinkTimeout`: no valid communication is received within the configured timeout.
- `LinkRestored`: communication is restored and validated.
- `FaultCleared`: a fault is cleared and recovery checks pass.

**Difference:** A state describes the robot's current mode over a period of time. An event is an occurrence or detected condition that may trigger a transition between states.

## 3.3 State transition table

| Current state | Event / guard | Next state | Requirement | Valid? | Verification evidence |
|---|---|---|---|---|---|
| OFF | PowerOn and self-check passes | IDLE | FR-01 | Yes | TC-01 |
| IDLE | Valid, authorised DeliveryRequest | NAVIGATING | FR-02, FR-09 | Yes | TC-02 |
| IDLE | DestinationReached without a request | DELIVERING | FR-02 | No | Negative transition test |
| NAVIGATING | ObstacleDetected | AVOIDING_OBSTACLE | FR-03 | Yes | TC-03 |
| AVOIDING_OBSTACLE | ObstacleCleared and path verified clear | NAVIGATING | FR-04 | Yes | TC-04 |
| AVOIDING_OBSTACLE | DeliveryStart / direct handover request | DELIVERING | FR-03 | No | Negative transition test |
| NAVIGATING | DestinationReached and safety checks pass | DELIVERING | FR-05 | Yes | TC-05 |
| DELIVERING | DeliveryComplete | RETURNING | FR-05 | Yes | TC-05 |
| NAVIGATING | CriticalBattery | SAFE_MODE, then controlled return if safety policy permits | FR-06, CR-01 | Yes | TC-06 |
| AVOIDING_OBSTACLE | CriticalBattery | SAFE_MODE; preserve obstacle safety and determine safe return | FR-06, CR-01 | Yes; must be explicitly implemented | TC-06 |
| DELIVERING | CriticalBattery / CriticalTemperature | SAFE_MODE; suspend delivery and protect batch | FR-06, CR-01 | Yes; must be explicitly implemented | TC-06, TC-08 |
| RETURNING | WarehouseReached and safety checks pass | IDLE | FR-01 | Yes | State test |
| NAVIGATING | LinkTimeout | LINK_LOST with local safety active | NFR-04 | Yes | TC-12 |
| AVOIDING_OBSTACLE | LinkTimeout | LINK_LOST with obstacle handling maintained locally | NFR-04 | Yes | TC-12 |
| DELIVERING | LinkTimeout | LINK_LOST / local safe hold; do not assume handover succeeded | NFR-04 | Yes | TC-12 |
| LINK_LOST | LinkRestored and status/commands reconciled | Previous safe operational state or controlled recovery state | NFR-04 | Conditional | TC-12 |
| SAFE_MODE | FaultCleared and authorised safety checks pass | IDLE or controlled recovery state | CR-01 | Conditional | TC-06 |

## 3.4 Behavioural verification findings

### (a) Can IDLE → DELIVERING happen?
No. The robot must first receive and accept a valid delivery request, navigate to the destination, and verify arrival. A direct IDLE → DELIVERING transition violates FR-02 and the stated rule that a waiting robot must never begin delivering directly.

### (b) What if AVOIDING_OBSTACLE has no transition back?
The robot can remain stuck in AVOIDING_OBSTACLE and never resume navigation or reach the destination. This is a liveness/deadlock problem. Add a transition on `ObstacleCleared` after the path is verified clear, and define a timeout/fault response if the obstacle cannot be cleared.

### (c) Can AVOIDING_OBSTACLE → DELIVERING occur?
No. It violates the rule that the robot must never deliver while handling an obstacle. The robot must return to NAVIGATING after obstacle clearance and only enter DELIVERING after DestinationReached and safety checks.

### (d) What happens on CriticalBattery while AVOIDING_OBSTACLE or DELIVERING?
If no transition is defined, the behaviour is unspecified and potentially unsafe. Define critical-condition transitions from every operational state to SAFE_MODE, then apply an approved controlled-return or safe-stop policy. During obstacle handling, the robot must not blindly move into an unsafe path. During delivery, it must suspend handover and protect the medicine batch.

## 3.5 Safe Mode and Link Lost rules
- Any active state must be able to trigger SAFE_MODE when a critical battery or temperature condition occurs.
- Under CR-01, entry into SAFE_MODE must occur within 3 seconds of the specified trigger.
- Link loss must not disable local obstacle detection, chamber monitoring or safety interlocks.
- SAFE_MODE and LINK_LOST are safety/control conditions, not permission to bypass existing guards.
- Exact recovery rules should be confirmed by the system safety authority.
