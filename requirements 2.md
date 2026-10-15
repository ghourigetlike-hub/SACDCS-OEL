# Task 1 — Requirement Extraction

**Project:** Smart Autonomous Cold-Chain Delivery & Control System (SACDCS)

The requirements below are rewritten as testable statements based on the assignment scenario. Proposed numeric acceptance criteria are assumptions for the pilot and require approval because the scenario does not specify all exact thresholds.

## 1.1 Functional requirements

| Req. ID | Requirement | Type | Priority | Verification method |
|---|---|---|---|---|
| FR-01 | When powered on, each robot shall enter IDLE and wait for a valid delivery request. | FR | High | State-transition test |
| FR-02 | On accepting a valid delivery request, the robot shall navigate to the specified clinic destination. | FR | High | Functional test / simulation |
| FR-03 | While navigating, the robot shall detect an obstacle, suspend normal navigation and begin obstacle avoidance. | FR | Critical | Scenario test / simulation |
| FR-04 | After an obstacle is confirmed clear, the robot shall resume navigation toward the original destination. | FR | High | Transition test |
| FR-05 | On reaching the destination, the robot shall deliver the package and then return to the warehouse. | FR | High | End-to-end test |
| FR-06 | If critical battery capacity is detected during a delivery journey, the robot shall abandon the delivery and return to the warehouse; the critical condition shall also trigger Safe Mode as defined by the safety policy. | FR | Critical | Fault-injection test |
| FR-07 | The chamber shall monitor temperature, humidity, vibration, door status and power for each medicine batch. | FR | Critical | Sensor integration test |
| FR-08 | The chamber shall correct out-of-range temperature or humidity and verify recovery using subsequent sensor readings. | FR | Critical | Sensor / fault-injection test |
| FR-09 | Before accepting a robot command, the system shall authenticate the operator and verify that the operator's role is authorised for that command. | FR | Critical | Security test |
| FR-10 | The system shall transmit robot telemetry and status, record events, and log accepted or rejected commands through the remote link. | FR | High | Integration test / log inspection |

## 1.2 Non-functional requirements

These measurable targets are proposed pilot acceptance criteria, not numbers explicitly stated in the scenario.

| Req. ID | Requirement | Type | Priority | Verification method |
|---|---|---|---|---|
| NFR-01 | After a critical battery or chamber/battery temperature threshold is detected, the robot shall enter Safe Mode within 3 seconds. | NFR | Critical | Timed fault-injection test |
| NFR-02 | The system shall support at least 20 robots connected to Mission Control simultaneously. | NFR | High | Load / scalability test |
| NFR-03 | Every command shall be rejected unless both operator authentication and role authorisation succeed; the decision shall be recorded in an audit log. | NFR | Critical | Security and audit test |
| NFR-04 | When the remote link is delayed or unavailable, the robot shall maintain local safety protections and shall not execute a stale or unauthorised command. | NFR | Critical | Network-delay / link-loss test |
| NFR-05 | For the 20-robot pilot, the portal shall display the latest received status and indicate stale telemetry when no update has arrived for 10 seconds. | NFR | Medium | UI / timing test |

## 1.3 High-risk requirements

| Req. ID | High-risk requirement | Risk category | Priority | Verification method |
|---|---|---|---|---|
| HR-01 | The robot shall enter Safe Mode within 3 seconds after a critical battery or temperature condition is detected. | Safety-critical / real-time | Critical | Timed fault injection |
| HR-02 | The robot shall never enter a railway crossing unless the barrier is open and no train is approaching. | Railway crossing | Critical | Crossing simulation / safety test |
| HR-03 | The robot shall not accept or execute a command unless the operator is authenticated and authorised for that command. | Security / command control | Critical | Negative and positive security tests |
| HR-04 | Loss or delay of the remote link shall not disable local obstacle, battery, chamber or crossing safety responses. | Link-dependent / safety | Critical | Network fault-injection test |

## 1.4 Verification notes
- A requirement is testable when an observable result can be checked against a defined condition.
- The 3-second Safe Mode target and the 10-second stale-telemetry indication are proposed measurable targets for this lab; the scenario does not give those exact values except that CR-01 specifies 3 seconds.
- Exact critical battery, temperature, humidity, vibration and recovery thresholds must be supplied by the approved system safety specification.
