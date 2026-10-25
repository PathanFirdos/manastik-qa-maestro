# Bug reports: Manastik Medico onboarding

Environment for all reports: Samsung Galaxy A14, Android 15, app build 3.21.5+475 (debug, `com.manastik.dadtmedico`), test OTP `111111`. Severity and priority are my proposal. Phone numbers and emails in evidence are masked.

| ID | Title | Severity | Priority |
|---|---|---|---|
| BUG-01 | App closes on the first relaunch after being closed during onboarding | Medium | Medium |
| BUG-02 | "Verification in progress" screen has no way to log out, refresh or go back | Medium | Medium |
| BUG-03 | "User already registered with Manastik" shown on Personal details even for a new email | Medium | Medium |
| BUG-05 | Languages Spoken field shows no red border when empty on Stage 7 | Low | Low |

---

## BUG-01: App closes on the first relaunch after being closed during onboarding

**Preconditions:** Account in onboarding (Personal details screen open), app not yet submitted.

**Steps**
1. Open the app and reach the Personal details screen.
2. Close the app from the Recents screen.
3. Open the app again.

**Expected:** The app opens and returns to the saved step (or shows the resume dialog).
**Actual:** The app closes on this first relaunch. A second launch works.
**Frequency:** Seen on the first relaunch after closing during onboarding. Number of repeats not recorded.

**Evidence**
- `evidence/reports/bug01_exit_info.txt` (Android exit history for the package). The entries reviewed list only user-requested exits (remove task, force stop). No crash reason appears in them.
- A crash was therefore not recorded by Android, so this may be the app finishing itself rather than crashing. That is not confirmed.
- Screen recording or screenshot: add file name here.

---

## BUG-02: "Verification in progress" screen has no way to log out, refresh or go back

**Preconditions:** Account has completed onboarding and is waiting for verification.

**Steps**
1. Complete the Medical onboarding path to the end.
2. On "Verification in progress", look for Logout, Log out, Refresh, back or home controls.
3. Close and reopen the app.

**Expected:** A way to leave the screen (log out) or refresh the status.
**Actual:** No such control is present. Reopening the app returns to the same screen every time. The only way to start over is to uninstall and reinstall.
**Frequency:** 100% (3 of 3 automated runs, plus manual runs).
**Note:** This may be intended behaviour while verification is pending. It is reported because a user cannot switch account or log out.

**Evidence:** `tests/maestro/04_verification_screen_no_exit.yaml`, reports `evidence/reports/04.xml`, `04_run2.xml`, `04_run3.xml` (all passed, meaning none of the three controls was found), and screenshots `evidence/screenshots/04_verification_no_exit.png` and `TC-21_verification_in_progress.png` (the screen shows only the message and a clock icon).

---

## BUG-03: "User already registered with Manastik" shown on Personal details even for a new email

**Preconditions:** Phone number that has already been used to register.

**Steps**
1. Log in or sign up with the already-used phone number and OTP `111111`.
2. On Personal details, enter an email and tap Next.
3. Repeat with a different email each time (four different emails were tried).

**Expected:** An error that says what is wrong (the phone number is already registered), or the email being accepted if it is unused.
**Actual:** The same message "User already registered with Manastik" appeared for all four emails (10:13 to 10:21). With a different phone number, the email that was rejected earlier was accepted. The message therefore appears to depend on the phone number, not the email, so it is misleading on this screen.
**Also observed once:** an earlier attempt on the same screen showed the toast "phone should not be empty".
**Frequency:** 4 of 4 attempts with the same phone number.

**Evidence:** `evidence/screenshots/BUG-03_user_already_registered.png` (10:21 attempt; the message appears as a toast over Personal details; email masked).

---

## BUG-05: Languages Spoken field shows no red border when empty on Stage 7

**Preconditions:** Medical path, Stage 7 (Practice details), all fields empty.

**Steps**
1. Reach Stage 7.
2. Tap Next with every field empty.

**Expected:** All five required fields show an error state.
**Actual:** The error text "Please select language spoken" appears under the field, but its border stays grey. The other four fields (Consultation Type, Specialised Procedure, Consultation Fee, Select Availability) get a red border and error text.
**Frequency:** Seen on the manual run (TC-20).

**Evidence:** `evidence/screenshots/TC-20_BUG-05_stage7_empty_fields.png`. Linked test case: TC-20.

---

## Observations to confirm (not yet reported as bugs)

These came from the screenshots and the manual run. Each is a test case marked "Not run" that needs one more check before it becomes a report.

| Observation | Evidence | Linked case |
|---|---|---|
| Screens go from Stage 4 (achievements, experience) to Stage 6 (institute). No Stage 5 screen was seen in the run. | `TC-17_stage4_achievements_none_added.png`, `TC-18_stage6_institute_search.png` | TC-29 |
| The experience date picker lets days after today (25 to 31 Oct 2026) be selected. Whether the app rejects the date afterwards is not checked. | `TC-25_stage4_experience_date_picker.png` | TC-25 |
| Dates of birth 25/10/2008 and 28/10/2008 (17 years old on the test date) were accepted on Personal details. The age policy is unknown. | `TC-21_03_stage1_personal_details.png`, `BUG-03_user_already_registered.png` | TC-24 |
| Login with a number that may have had no account moved on to onboarding Stage 1 after OTP 111111, instead of showing "not registered". Needs a brand-new number to confirm. | `TC-21_01_login_phone_entered.png`, `TC-21_02_otp_entry.png`, `TC-21_03_stage1_personal_details.png` | TC-30 |
