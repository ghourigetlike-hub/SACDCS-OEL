# Task 4 — Conservation Chamber Operations

## 4.1 Identified operations

Operation names describe actions, not state names.

1. `OP-01 InitialiseChamber`
2. `OP-02 RunSensorSelfCheck`
3. `OP-03 IdentifyMedicineBatch`
4. `OP-04 LoadConservationProfile`
5. `OP-05 ReadEnvironmentalSensors`
6. `OP-06 CorrectTemperature`
7. `OP-07 CorrectHumidity`
8. `OP-08 VerifyEnvironmentalRecovery`
9. `OP-09 DetectAndRespondToVibration`
10. `OP-10 DetectDoorOpening`
11. `OP-11 RestoreOperationAfterDoorClosure`
12. `OP-12 SwitchToEmergencyPower`
13. `OP-13 RecordPowerLossAndShutdownSafely`
14. `OP-14 ActivateProtectionResponse`
15. `OP-15 AlertOperator`
16. `OP-16 VerifySafeBatchRemoval`

## 4.2 Operation schemas

### OP-01 — InitialiseChamber
- **Description:** Starts chamber checks after power-on.
- **Inputs:** Power-on signal, sensor health results.
- **Outputs:** Initialisation result.
- **Reads:** Sensor status, power status, chamber configuration.
- **Changes:** Sets readiness only if all essential sensors work.
- **Sends/Alerts:** Reports failed self-check to the controller/operator.
- **Assumes (precondition):** Chamber receives power and the controller is running.
- **Result (postcondition):** Chamber is marked ready only when essential sensors pass the self-check.

### OP-03 — IdentifyMedicineBatch
- **Description:** Identifies the placed batch.
- **Inputs:** Batch identifier / scan.
- **Outputs:** Batch identity and lookup result.
- **Reads:** Batch record and medicine profile.
- **Changes:** Associates the identified batch with the chamber.
- **Sends/Alerts:** Reports unknown or invalid batch identity.
- **Assumes (precondition):** Batch is presented and identification equipment is available.
- **Result (postcondition):** A valid batch is associated with the chamber; an unknown batch is not treated as validated.

### OP-04 — LoadConservationProfile
- **Description:** Loads required temperature and humidity limits for a batch.
- **Inputs:** Batch ID and approved conservation profile.
- **Outputs:** Loaded profile and validation result.
- **Reads:** Batch record, temperature/humidity limits.
- **Changes:** Stores the active batch profile.
- **Sends/Alerts:** Reports missing or invalid profile.
- **Assumes (precondition):** Batch has been identified.
- **Result (postcondition):** The correct limits are available for monitoring; active conservation starts only when the door is closed and the profile is loaded.

### OP-05 — ReadEnvironmentalSensors
- **Description:** Collects current chamber readings.
- **Inputs:** Temperature, humidity, vibration, door and power sensor signals.
- **Outputs:** Time-stamped readings and sensor-health status.
- **Reads:** All chamber sensors.
- **Changes:** Updates the latest reading record.
- **Sends/Alerts:** Reports missing, invalid or out-of-range readings.
- **Assumes (precondition):** Monitoring controller and relevant sensors are powered.
- **Result (postcondition):** Current readings and their validity are recorded; invalid readings are not treated as proof of safe conditions.

### OP-06 — CorrectTemperature
- **Description:** Adjusts chamber temperature when it is outside the loaded profile.
- **Inputs:** Current temperature and target range.
- **Outputs:** Cooling/heating control request.
- **Reads:** Temperature reading and loaded profile.
- **Changes:** Adjusts temperature-control output.
- **Sends/Alerts:** Reports correction failure if recovery period expires.
- **Assumes (precondition):** A valid profile is loaded and temperature sensor data is valid.
- **Result (postcondition):** A correction action is issued; temperature is not considered recovered until OP-08 confirms it through sensor readings.

### OP-08 — VerifyEnvironmentalRecovery
- **Description:** Confirms that temperature and humidity have returned to permitted ranges.
- **Inputs:** New sensor readings, configured ranges and recovery deadline.
- **Outputs:** Recovery confirmed / recovery failed.
- **Reads:** Current sensor readings, profile limits, recovery timer.
- **Changes:** Updates recovery status.
- **Sends/Alerts:** On failure, requests protection response and operator alert.
- **Assumes (precondition):** A temperature or humidity excursion has been detected.
- **Result (postcondition):** Recovery is confirmed only if valid readings remain within the approved range according to the approved verification rule; otherwise failure is recorded.

### OP-10 — DetectDoorOpening
- **Description:** Detects a door opening during conservation.
- **Inputs:** Door sensor status.
- **Outputs:** Door-open event.
- **Reads:** Door sensor and current chamber activity.
- **Changes:** Suspends normal conservation activities.
- **Sends/Alerts:** Reports door opening to the robot/controller and operator as configured.
- **Assumes (precondition):** Door sensor is available.
- **Result (postcondition):** Normal operation is suspended; it cannot resume until the door is closed and conditions and sensors are verified.

### OP-12 — SwitchToEmergencyPower
- **Description:** Transfers the chamber to emergency power after main power loss, when available.
- **Inputs:** Main-power failure signal and emergency-power availability.
- **Outputs:** Transfer success/failure.
- **Reads:** Main-power and emergency-power status.
- **Changes:** Selects emergency power if available.
- **Sends/Alerts:** Reports power loss and transfer result.
- **Assumes (precondition):** Power monitoring is functioning.
- **Result (postcondition):** Emergency power is confirmed only when its status is verified; if unavailable, OP-13 is performed.

### OP-14 — ActivateProtectionResponse
- **Description:** Protects a batch when correction fails within the recovery period.
- **Inputs:** Recovery failure, sensor/fault status.
- **Outputs:** Protection response status.
- **Reads:** Recovery timer, environmental readings, chamber status.
- **Changes:** Enters protection mode and reduces light exposure.
- **Sends/Alerts:** Alerts the operator and logs the reason.
- **Assumes (precondition):** A qualifying failure or unsafe condition is detected.
- **Result (postcondition):** Protection mode is active, light exposure is reduced, and the event is recorded.

### OP-16 — VerifySafeBatchRemoval
- **Description:** Decides whether a batch can be removed.
- **Inputs:** Door status, sensor health, chamber conditions and protection status.
- **Outputs:** Removal permitted / denied.
- **Reads:** All relevant safety readings and protection-response status.
- **Changes:** Records the removal decision.
- **Sends/Alerts:** Denial may be reported to the operator.
- **Assumes (precondition):** A removal request is made.
- **Result (postcondition):** Removal is permitted only when the chamber is safe and no protection response is active.

## 4.3 Links to the robot state model
- If a critical chamber temperature or other defined critical fault occurs during NAVIGATING, the robot transitions to SAFE_MODE under CR-01.
- If the chamber door opens during DELIVERING, the chamber suspends normal conservation and the robot must not treat delivery as safe until chamber conditions are verified. A transition to SAFE_MODE or a controlled safe-hold response is required by the safety policy.
- If power is lost and emergency power is unavailable, the chamber records the incident and shuts down safely; the robot/control system must be informed so it can take the appropriate safe response.

## 4.4 Operations requiring sensor verification
1. **CorrectTemperature / CorrectHumidity:** issuing a control command does not prove the actual temperature or humidity changed. New valid sensor readings must confirm the value is back within the required range.
2. **RestoreOperationAfterDoorClosure:** closing the door does not prove that the chamber is safe. The system must verify sensor health and environmental conditions before resuming normal operation.
