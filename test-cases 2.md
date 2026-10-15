# Verification Test Cases

Test cases are based on the scenario's command path: command received → operator authenticated → role authorised → command executed or rejected → status reported → event logged. Any numeric thresholds not specified by the scenario must be confirmed by the safety authority.

| Test ID | Requirement / risk | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-01 | FR-01 | Robot is powered off; system is healthy. | Power on the robot. | Robot enters IDLE and waits for a delivery request. State change is logged. |
| TC-02 | FR-02, CR-03 | Robot is IDLE; test operator has valid credentials and delivery permission. | Submit a valid delivery request; authenticate and authorise the operator. | Command is accepted, robot enters NAVIGATING, status is reported and event is logged. |
| TC-03 | FR-03, HR-04 | Robot is NAVIGATING on a simulated route. | Introduce an obstacle. | Normal navigation suspends and robot enters AVOIDING_OBSTACLE; event is logged. |
| TC-04 | FR-04 | Robot is AVOIDING_OBSTACLE. | Clear the obstacle and provide the clear event. | Robot returns to NAVIGATING and resumes the route. |
| TC-05 | FR-05 | Robot reaches the designated clinic; delivery is safe and authorised. | Trigger DestinationReached and complete delivery. | Robot enters DELIVERING, completes the package handover, then enters RETURNING. |
| TC-06 | CR-01, HR-01, NFR-01 | Robot is in an active state; battery and temperature sensors are simulated. | Inject critical chamber temperature, critical battery temperature, and emergency-low battery in separate runs; measure time to Safe Mode. | Safe Mode is entered within 3 seconds for each specified trigger; trigger and transition are logged. |
| TC-07 | FR-07 | Chamber sensors are connected. | Power on and inspect temperature, humidity, vibration, door and power monitoring. Then simulate one essential sensor failure. | Readings are monitored; normal monitoring is not declared ready when an essential sensor fails. Fault is reported. |
| TC-08 | FR-08 | Chamber has a loaded batch profile and a simulated out-of-range reading. | Cause a temperature/humidity excursion; allow correction; provide readings that return to range, then repeat with correction failure. | Success is declared only after sensor readings confirm recovery. If recovery period expires, protection mode starts, light exposure is reduced and operator is alerted. |
| TC-09 | FR-09, CR-03 | Test accounts exist: authorised, unauthorised role, and invalid credentials. | Submit commands using each account type. | Only authenticated and role-authorised commands are accepted; others are rejected and all decisions are logged. |
| TC-10 | FR-10 | Mission Control and a robot are connected. | Send a command; inspect robot status, telemetry and event log. Repeat with a rejected command. | Latest received status is displayed; accepted/rejected command and relevant event are recorded. |
| TC-11 | CR-02, NFR-02 | Mission Control test environment is available. | Connect 20 robots simultaneously and exchange status messages. | All 20 connections are supported; status and logs remain associated with the correct robot. |
| TC-12 | NFR-04, HR-04 | Robot is navigating; link can be delayed or disconnected. | Delay messages by several seconds, then disconnect the link. | Local safety functions remain active; stale commands are not executed; link loss is indicated and logged. |
| TC-13 | HR-02 | Crossing simulator provides train, barrier and approach signals. | Try entry with barrier closed; with train approaching; and with barrier open and no train approaching. | Robot is blocked in the first two cases and may enter only when the barrier is open and no train is approaching. |
| TC-14 | Task 4, C-11 | Chamber contains a batch. | Open the door during conservation; attempt batch removal while protection response is active; then restore safe conditions and verify sensors. | Normal operation suspends on door opening. Removal is denied until the chamber is safe and no protection response is active. |
| TC-15 | Task 5, sensor failure | Crossing sensor failure can be simulated. | Inject an invalid or failed train-detection sensor reading. | System treats crossing status as unsafe/unknown, prevents robot entry, raises an alarm and logs the fault. |

## Test execution record template

For each run, record:
- Test ID and date
- Tester
- Software/build version
- Input conditions
- Actual result
- Pass / Fail
- Evidence (screenshot, log excerpt or simulator output)
- Defect ID if failed

## Regression after changes
- **CR-01:** rerun TC-06, TC-07, TC-08 and TC-12.
- **CR-02:** rerun TC-11, TC-10 and TC-12.
- **CR-03:** rerun TC-09, TC-02 and TC-10.
