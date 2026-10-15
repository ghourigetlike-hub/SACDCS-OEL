# Task 5 — Constraints, Formalisation & Violations

## 5.1 Constraint list

| Constraint ID | Constraint in simple English | Why necessary |
|---|---|---|
| C-01 | While an obstacle is detected, normal navigation is suspended; navigation resumes only after the path is confirmed clear. | Prevents collision and unsafe movement. |
| C-02 | A critical battery or chamber/battery temperature condition triggers Safe Mode within 3 seconds. | Limits harm from critical faults. |
| C-03 | A robot command is accepted only if the operator is authenticated and role-authorised. | Prevents unauthorised control. |
| C-04 | Command decisions and safety-critical events are logged. | Supports audit, investigation and verification. |
| C-05 | Mission Control supports at least 20 simultaneously connected robots for the pilot. | Meets the pilot scale requirement from CR-02. |
| C-06 | Loss or delay of the remote link must not disable local safety functions or allow stale commands to execute. | Protects the robot when communications are unreliable. |
| C-07 | The robot may enter a railway crossing only when the barrier is open and no train is approaching. | Prevents collision with a train. |
| C-08 | The barrier remains closed while a train is passing and opens only after the crossing is confirmed clear. | Prevents premature barrier opening. |
| C-09 | If train-detection or barrier status is failed, invalid or unknown, robot crossing entry is prohibited. | Fails safely when sensors are unreliable. |
| C-10 | A robot in IDLE must not enter DELIVERING without an accepted delivery request and arrival at the destination. | Prevents an invalid state transition. |
| C-11 | The robot must not perform delivery while the chamber door is open or a protection response is active. | Protects medicine integrity and operator safety. |
| C-12 | Normal conservation is not permitted unless essential sensors work and a valid batch profile is loaded. | Avoids uncontrolled storage conditions. |
| C-13 | Temperature/humidity recovery is confirmed only by valid sensor readings within the approved limits. | A command alone does not prove recovery. |
| C-14 | A batch may be removed only when the chamber is safe and no protection response is active. | Prevents unsafe batch handling. |
| C-15 | Significant vibration suspends risky chamber activities until vibration stays below the threshold for the stabilisation period. | Avoids damage or unsafe handling. |

## 5.2 Logical formalisation

Let:
- `Train_Present`: a train is present or approaching.
- `Barrier_Open`: crossing barrier is open.
- `Robot_Entering`: robot is entering the crossing.
- `Sensors_Valid`: crossing sensors provide valid readings.
- `Operator_Authenticated`: operator identity has been verified.
- `Role_Authorised`: operator has permission for the requested command.
- `Command_Accepted`: system accepts the command.
- `Critical_Condition`: critical battery or temperature trigger is active.
- `Safe_Mode`: robot is in Safe Mode.
- `Obstacle_Detected`: obstacle is detected.
- `Normal_Navigation`: robot is performing normal navigation.
- `Chamber_Door_Open`: chamber door is open.
- `Delivering`: robot is delivering a package.
- `Protection_Active`: chamber protection response is active.
- `Profile_Loaded`: valid batch profile is loaded.
- `Sensors_Healthy`: essential chamber sensors are healthy.
- `Recovery_Confirmed`: readings confirm temperature/humidity are in range.
- `Correction_Success`: correction is declared successful.
- `Crossing_Clear`: crossing is confirmed clear.
- `Sensor_Failure`: a crossing sensor has failed.
- `Stale_Command`: command is too old to execute safely.

| Constraint | Logical expression |
|---|---|
| C-01 | `Obstacle_Detected → ¬Normal_Navigation` |
| C-02 | `Critical_Condition → (Safe_Mode within 3 seconds)` |
| C-03 | `Command_Accepted → (Operator_Authenticated ∧ Role_Authorised)` |
| C-07 | `Robot_Entering → (Barrier_Open ∧ ¬Train_Present)` |
| C-08 | `Train_Present → ¬Barrier_Open` |
| C-09 | `Sensor_Failure → ¬Robot_Entering` |
| C-10 | `Delivering → (Delivery_Request_Accepted ∧ Destination_Reached)` |
| C-11 | `Chamber_Door_Open → ¬Delivering` and `Protection_Active → ¬Delivering` |
| C-12 | `Normal_Conservation → (Sensors_Healthy ∧ Profile_Loaded ∧ ¬Chamber_Door_Open)` |
| C-13 | `Correction_Success → Recovery_Confirmed` |
| C-14 | `Batch_Removal_Allowed → (Chamber_Safe ∧ ¬Protection_Active)` |
| C-06 | `Link_Lost → Local_Safety_Active` and `Stale_Command → ¬Command_Executed` |
| C-15 | `Vibration_Above_Threshold → ¬Risky_Activity` |

**Note:** “within 3 seconds” is a timed temporal condition; it is written in words because ordinary propositional logic alone does not represent elapsed time.

## 5.3 Violation scenarios and expected responses

| Scenario | Constraint violated / variable values | Detection | Expected response |
|---|---|---|---|
| VS-01 | C-01: `Obstacle_Detected=TRUE`, `Normal_Navigation=TRUE` | Obstacle monitor compares sensor flag with navigation command. | Suspend navigation, begin obstacle handling, raise/log fault if motion continues. |
| VS-02 | C-02: `Critical_Condition=TRUE`, `Safe_Mode=FALSE` after 3 seconds | Safety timer detects missed deadline. | Trigger emergency safety response, alert Mission Control/operator and log deadline violation. |
| VS-03 | C-03: `Operator_Authenticated=TRUE`, `Role_Authorised=FALSE`, `Command_Accepted=TRUE` | Authorisation audit detects an accepted forbidden command. | Reject/stop command where possible, alert security operator and log the violation. |
| VS-04 | C-07: `Robot_Entering=TRUE`, `Barrier_Open=FALSE`, `Train_Present=TRUE` | Crossing interlock sees unsafe entry request. | Stop robot before the crossing, keep barrier closed, alarm and log. |
| VS-05 | C-08: `Train_Present=TRUE`, `Barrier_Open=TRUE` | ARLCCS compares train detection and barrier position. | Command barrier closed, activate warnings, prevent robot entry and alert operator. |
| VS-06 | C-09: `Sensor_Failure=TRUE`, `Robot_Entering=TRUE` | Sensor diagnostics mark crossing state invalid. | Treat crossing as unsafe/unknown, stop robot, alarm and log sensor fault. |
| VS-07 | C-10: `Delivering=TRUE`, `Delivery_Request_Accepted=FALSE` or `Destination_Reached=FALSE` | State-transition monitor checks required guards. | Block delivery transition, return to a safe valid state and log invalid transition. |
| VS-08 | C-11: `Chamber_Door_Open=TRUE`, `Delivering=TRUE` | Door sensor and robot state monitor detect conflict. | Suspend delivery, protect batch, notify operator and log. |
| VS-09 | C-12: `Normal_Conservation=TRUE`, `Sensors_Healthy=FALSE` | Self-check/health monitor reports sensor failure. | Suspend normal conservation, enter protection response and alert operator. |
| VS-10 | C-13: `Correction_Success=TRUE`, `Recovery_Confirmed=FALSE` | Verification logic compares correction result with subsequent readings. | Do not declare recovery; continue recovery until deadline, then activate protection response and alert. |
| VS-11 | C-14: `Batch_Removal_Allowed=TRUE`, `Protection_Active=TRUE` | Removal interlock checks protection flag. | Deny removal, display reason, alert if necessary and log attempt. |
| VS-12 | C-06: `Link_Lost=TRUE`, `Local_Safety_Active=FALSE` or `Stale_Command=TRUE`, `Command_Executed=TRUE` | Watchdog and command-age validation detect the failure. | Enter the defined safe communication response, reject stale command, alert Mission Control when possible and log locally. |

## 5.4 Robot–chamber and communication links
- C-11 directly links robot delivery state with chamber door and protection status.
- C-12 links chamber operation to sensor health and the loaded medicine profile.
- C-06 covers communication loss and requires local safety functions to remain active.
- C-09 ensures crossing sensor failure blocks entry rather than being interpreted as a safe crossing.

## 5.5 Verification note
Each constraint should be monitored or tested using observable values. Thresholds for temperature, humidity, vibration, sensor timeout, stabilisation period and recovery period must be defined in the approved system specification; they are not numerically supplied by the assignment scenario.
