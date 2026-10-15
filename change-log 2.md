# Task 2 — Change Requests and Version-Controlled Change Log

## 2.1 Change log

| Change ID | Requirement ID | Status | Old wording | New wording | Reason |
|---|---|---|---|---|---|
| CR-01 | HR-01 / NFR-01 | Modified | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | Adds measurable timing and explicit trigger conditions, enabling a timed test. |
| CR-02 | NFR-02 | Modified | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots. | Replaces the vague phrase “multiple robots” with a measurable pilot capacity. |
| CR-03 | FR-09 / NFR-03 | Modified | Only authenticated operators shall be permitted to issue robot commands. | The system shall require authenticated and role-authorised operators before accepting any robot command. | Makes role-based authorisation explicit as well as authentication and applies it to every command. |

## 2.2 Why each change is more testable

### CR-01 — Safe Mode timing
The revised requirement defines the triggering conditions and a 3-second deadline. A tester can inject a critical temperature or battery condition, measure elapsed time, and check whether Safe Mode is entered within the limit.

### CR-02 — Simultaneous robots
The revised requirement gives a minimum capacity of 20 connected robots. A load test can connect 20 robots at once and verify that the system supports them.

### CR-03 — Authentication and authorisation
The revised requirement requires two separate checks before any command is accepted: authentication proves the operator's identity, while role authorisation checks permission for the requested command. Positive and negative tests can verify both checks.

## 2.3 Version-control procedure
For each change, commit the revised requirement and this log together. Use a clear commit message such as:
- `CR-01: define Safe Mode triggers and 3-second limit`
- `CR-02: set minimum capacity to 20 robots`
- `CR-03: require role authorisation for every command`

Do not delete the old wording; retain it in this change log so the history remains auditable.
