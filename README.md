# Manastik Medico: QA Screening (Maestro automation)

QA-track submission for the Manastik internship screening. The app under test is the Android build supplied by the hiring team. No separate app was built.

## What is in this repository

```
README.md
test-plan.md
test-cases/manual-test-cases.csv
bug-reports/bug-reports.md
tests/maestro/            Maestro YAML flows
evidence/reports/         JUnit XML results from real runs
evidence/screenshots/     Screenshots (phone numbers and emails masked)
demo.mp4                  Screen recording
```

The APK is not committed (it is supplied by the hiring team, see `.gitignore`).

## Test environment

| Item | Value |
|---|---|
| App | Manastik Medico, package `com.manastik.dadtmedico`, build 3.21.5+475 (debug) |
| Device | Samsung Galaxy A14, Android 15 |
| Host | Windows, PowerShell |
| Tools | ADB, Maestro 2.11.0, Java (required by Maestro) |

## Setup

1. Install Java 17 or newer, ADB (Android platform-tools) and Maestro. Add them to `PATH`.
2. Enable USB debugging on the phone, connect it, unlock it, and keep the screen on:
   ```powershell
   adb devices
   adb shell svc power stayon usb
   ```
   `adb devices` must list the phone as `device`.
3. Place the supplied APK at `app\app-debug.apk` and install it:
   ```powershell
   adb install -r app\app-debug.apk
   ```

## Running the flows

```powershell
# Flow 02: needs no account, no test number
maestro test --format junit --output evidence\reports\02.xml tests\maestro\02_invalid_phone_length.yaml

# Flow 03: needs a phone number that has already completed registration.
# The number is read at run time, so it is never stored in the repository.
$phone = Read-Host "Used test number (10 digits)"
maestro test -e TEST_PHONE=$phone --format junit --output evidence\reports\03.xml tests\maestro\03_invalid_otp.yaml

# Flow 04: precondition, see "Known limitations" (account must be in "Verification in progress")
maestro test --format junit --output evidence\reports\04.xml tests\maestro\04_verification_screen_no_exit.yaml
```

Use `--debug-output evidence\debug` to keep Maestro's own logs and screenshots in a folder you choose.

## Flows and results

| Flow | What it checks | Result |
|---|---|---|
| `02_invalid_phone_length.yaml` | A 9-digit phone number does not open the OTP screen after tapping Send OTP | Passed (1 run) |
| `03_invalid_otp.yaml` | OTP `000000` shows "Invalid security code" | Passed (1 run) |
| `04_verification_screen_no_exit.yaml` | "Verification in progress" screen shows, and has no Logout, Log out or Refresh control | Passed (2 runs) |

How to read the results: each run writes a JUnit XML file to `evidence/reports/`. `failures="0"` means every step passed. A failed step names the assertion that did not hold.

What the flows do not prove:
- Flow 02 checks that the OTP screen did not open. It does not check that the button looks disabled. That was checked by hand (TC-01).
- Flow 04 checks three words only. It does not prove that no other exit control exists, and it does not prove the missing exit is unintended.

## Design decisions

- Flutter exposes labels as accessibility text, and the phone field label appears only as a hint. Selectors therefore use text or a regex such as `.*Phone Number.*`, and flows wait for "Send OTP" instead of the heading, because the open keyboard hides the heading.
- Flows start with a plain `launchApp` and handle both possible start screens (welcome or phone-number screen) so they do not depend on one fixed state.
- Test data is never hardcoded. The phone number is passed with `-e TEST_PHONE`. The OTP is the fixed test value `111111` from the hiring team's email.

## Assumptions

- The test build does not send real SMS or email. The hiring team's email states the OTP is `111111`. The screen text "We've sent an SMS" is therefore test-build behaviour and is not reported as a defect.
- For the same reason, mailbox existence and OTP expiry cannot be tested in this build.

## Known limitations

- **The start screen depends on account state.** On the same device the app opened on Personal details, on Login, and on "Verification in progress" at different times. `pm clear` and Maestro's `clearState` did **not** reset the app. Uninstalling and reinstalling the APK did, and showed the welcome screen. Reset between runs with:
  ```powershell
  adb uninstall com.manastik.dadtmedico
  adb install -r app\app-debug.apk
  ```
  The server-side account is not reset by this.
- **Sign-up can run once per phone number.** Completed registrations consume the number, and re-using it produces "User already registered" or "Verification in progress". Two test numbers were used up before automation started.
- **The full end-to-end sign-up flow (sign-up through "Verification in progress") has not been automated or run end to end.** It was walked through manually once in about 3 minutes (see TC-21 in the test cases). Automating it needs a fresh test number or an account reset from the hiring team. A reset has been requested.
- **Flow 04 needs an account that has already submitted onboarding.** After a reinstall, log in with a registered number and code `111111` first.
- `takeScreenshot` saves into Maestro's debug output folder, not into the project folder. Selected images were copied into `evidence/screenshots/` by hand.
- Flows ran on one device (Android 15). Other devices and Android versions were not tested.

## Privacy

Phone numbers, email addresses, names and uploaded documents were masked or replaced before publishing. No credentials or hiring-team answer key is included.
