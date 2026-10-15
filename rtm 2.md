# Task 2 — Multi-Layer Requirements Traceability Matrix (RTM)

The RTM links requirements to robot/chamber behaviour, operations, constraints, tests and risk. IDs refer to the other files in this repository.

| Requirement | State / transition | Operation | Constraint | Test case | Risk |
|---|---|---|---|---|---|
| FR-01 | ST-01: PowerOn → IDLE | OP-01 Initialise robot | C-10: no delivery from IDLE without request | TC-01 | Medium |
| FR-02 | ST-02: IDLE → NAVIGATING on DeliveryRequest | OP-02 Validate and accept delivery request | C-10: delivery requires an accepted request | TC-02 | High |
| FR-03 | ST-03: NAVIGATING → AVOIDING_OBSTACLE | OP-03 Detect and handle obstacle | C-01: obstacle handling suspends navigation | TC-03 | Critical |
| FR-04 | ST-04: AVOIDING_OBSTACLE → NAVIGATING on ObstacleCleared | OP-04 Resume route | C-01: resume only after obstacle clear | TC-04 | High |
| FR-05 | ST-05: NAVIGATING → DELIVERING → RETURNING | OP-05 Deliver package; OP-06 Return to warehouse | C-11: delivery requires safe robot and chamber | TC-05 | High |
| FR-06 | ST-06: NAVIGATING / AVOIDING_OBSTACLE → SAFE_MODE / RETURNING | OP-07 Handle critical battery | C-02: critical condition triggers Safe Mode | TC-06 | Critical |
| FR-07 | Chamber monitoring operation; door/power events alter operation | OP-08 Read chamber sensors | C-12: sensor failure prevents normal conservation | TC-07 | Critical |
| FR-08 | Chamber correction / protection response | OP-09 Correct and verify temperature/humidity | C-13: correction success requires sensor confirmation | TC-08 | Critical |
| FR-09 | Command accepted only after identity and role checks | OP-10 Authenticate and authorise command | C-03: unauthenticated or unauthorised command rejected | TC-09 | Critical |
| FR-10 | Telemetry, status reporting and event logging | OP-11 Report status; OP-12 Record event | C-04: command decision and safety events logged | TC-10 | High |
| NFR-01 / HR-01 | Any permitted active state → SAFE_MODE within 3 s of trigger | OP-07 Handle critical condition | C-02 | TC-06 | Critical |
| NFR-02 | Mission Control connected-robot capacity | OP-13 Manage robot connections | C-05: at least 20 simultaneous robot connections | TC-11 | High |
| NFR-04 / HR-04 | Active state → LINK_LOST when communication is lost | OP-14 Handle link loss | C-06: local safety remains active during link loss | TC-12 | Critical |
| HR-02 | Approach crossing → hold until safe-to-enter condition | OP-15 Request crossing clearance | C-07: crossing entry interlock | TC-13 | Critical |
| Task 4 | Chamber operation is suspended on unsafe door/vibration/power condition | OP-16 Verify chamber safe for batch removal | C-11 | TC-14 | Critical |

## Regression tests required by each change request

| Change request | Tests to re-run | Reason |
|---|---|---|
| CR-01 | TC-06, TC-07, TC-08, TC-12 | Re-check Safe Mode timing, thermal/battery triggers, chamber protection and link-loss interaction. |
| CR-02 | TC-11, TC-10, TC-12 | Re-check 20-robot capacity, telemetry/logging and behaviour under communication load. |
| CR-03 | TC-09, TC-02, TC-10 | Re-check command acceptance, delivery-request commands and audit records for allowed and rejected commands. |

## Traceability maintenance
Whenever a requirement, state, operation, constraint or test changes, update the affected RTM row in the same commit. A requirement with no linked test is an unverified requirement and must be reviewed before release.
