# Test plan: Manastik Medico onboarding

## 1. Objective

Check the onboarding workflow of the Manastik Medico Android app: sign-up or login, OTP verification, personal details, role selection, medical profile stages, and the final "Verification in progress" state. Automate the repeatable parts with Maestro and report defects with evidence.

## 2. Scope

In scope
- Phone entry and validation (login and create account)
- OTP entry and validation (fixed test OTP `111111`)
- Personal details, email code, resume dialog
- Role selection (Medical, Administrative)
- Medical onboarding: documents, achievements, institution, practice details
- "Verification in progress" screen and relaunch behaviour

Out of scope
- Features after approval (the verification result is manual and takes up to 24 hours)
- Real SMS or email delivery (the test build sends none)
- Performance, security and accessibility testing
- Devices other than the Galaxy A14, and iOS

## 3. Approach

| Layer | Method |
|---|---|
| Manual exploratory and scripted cases | Executed on a physical device, results recorded in `test-cases/manual-test-cases.csv` |
| Automation | Maestro YAML flows in `tests/maestro/`, run from PowerShell, JUnit output in `evidence/reports/` |
| Defect reporting | `bug-reports/bug-reports.md`: environment, preconditions, steps, expected, actual, severity and priority, frequency, evidence |

Test design techniques: boundary and negative inputs (phone length, wrong OTP), empty-field validation, state transitions (resume, relaunch), and review of copy and UI consistency.

## 4. Environment

Samsung Galaxy A14, Android 15, app build 3.21.5+475 (debug, `com.manastik.dadtmedico`), Maestro 2.11.0, ADB, Windows PowerShell.

## 5. Test data

- Fixed OTP `111111` (from the hiring team's email). Invalid OTPs used: `000000`, `123458`.
- Phone numbers are real personal numbers. They are never stored in the repository. Automation reads them at run time with `-e TEST_PHONE`.
- Uploaded documents should be dummy files in anything published.

## 6. Entry and exit criteria

Entry: app installed, phone connected and unlocked, `adb devices` shows `device`.
Exit: flows run with recorded results, manual cases executed or marked not run with a reason, defects documented.

## 7. Risks and constraints

| Risk | Effect | Handling |
|---|---|---|
| Each completed sign-up consumes a phone number | The full sign-up flow cannot be repeated | Reset requested from the hiring team. Other flows are written to run on any account. |
| App start screen depends on server-side account state | Flows can start on different screens | Flows wait for stable anchors and handle both start screens. Reinstall is the reset method. |
| Fixed OTP and no real messages | Expiry, resend invalidation and mailbox checks cannot be tested | Listed as blocked in the test cases |
| Repeated wrong OTPs may lock the shared test account | Lost test account | Wrong-OTP flow limited to a few runs |
| One device only | Device-specific issues may be missed | Stated as a limitation |

## 8. Deliverables

Test plan, test cases (CSV), bug reports, Maestro flows, README, evidence (reports and screenshots), demo recording.

## 9. Status at submission

- Automated and passing: flows 02, 03, 04 (see README for exact scope).
- Manual cases TC-01 to TC-22 executed. TC-23 onward not run or blocked, each with a reason.
- Not done: automated end-to-end sign-up flow (needs a fresh number or reset). Retests listed in the test cases as "Not run".
