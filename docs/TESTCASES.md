# Test Cases

## Status Definitions

- **Not Run**
- **Pass**
- **Fail**
- **Blocked**

---

## TC-AUTH-001: Register with valid new email

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** Email does not already exist in `users`
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/register`
2. Provide valid `first_name`, `last_name`, `email`, and `password_hash`
3. Inspect response

### Expected Result
- Response indicates success
- Access token is returned
- Refresh token is returned
- User object is returned without exposing password hash
- User record is created

---

## TC-AUTH-002: Register with duplicate email

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** Email already exists
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/register` with an existing email
2. Inspect response

### Expected Result
- Request is rejected
- Response indicates failure
- Message states an account already exists for this email

---

## TC-AUTH-003: Login with valid credentials

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** User exists and password is correct
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/login`
2. Provide valid email and correct password
3. Inspect response

### Expected Result
- Response indicates success
- Access token and refresh token are returned
- User object is returned
- Password hash is not included in response

---

## TC-AUTH-004: Login with nonexistent email

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** Email does not exist
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/login` with nonexistent email
2. Inspect response

### Expected Result
- Request is rejected
- Generic invalid email/password message is returned

---

## TC-AUTH-005: Login with wrong password

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** User exists
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/login` with valid email and wrong password
2. Inspect response

### Expected Result
- Request is rejected
- Generic invalid email/password message is returned

---

## TC-AUTH-006: Google login with missing token

- **Priority:** High
- **Type:** Negative / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/google-login` without `id_token`
2. Inspect response

### Expected Result
- Request is rejected
- Response message indicates missing token

---

## TC-AUTH-007: Google login with invalid token

- **Priority:** High
- **Type:** Negative / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/google-login` with invalid token
2. Inspect response

### Expected Result
- Request is rejected
- Response message indicates invalid Google token or auth failure

---

## TC-AUTH-008: Google login for new account

- **Priority:** High
- **Type:** Functional / API
- **Preconditions:** Valid Google token for email not in system
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/google-login` with valid token
2. Inspect response and database state

### Expected Result
- New user is created with `auth_provider = google`
- Response indicates account created successfully
- Tokens are returned

---

## TC-AUTH-009: Google login for existing account

- **Priority:** High
- **Type:** Functional / API
- **Preconditions:** Valid Google token for existing user
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/google-login` with valid token
2. Inspect response

### Expected Result
- Existing user is returned
- No duplicate user is created
- Response indicates login successful
- Tokens are returned

---

## TC-AUTH-010: Refresh with valid refresh token

- **Priority:** High
- **Type:** Functional / API
- **Preconditions:** Valid refresh token exists
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/refresh` with valid refresh token
2. Inspect response

### Expected Result
- New access token is returned
- New refresh token is returned
- User object is returned
- Response indicates success

---

## TC-AUTH-011: Refresh with missing token

- **Priority:** High
- **Type:** Negative / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/refresh` without refresh token
2. Inspect response

### Expected Result
- Request is rejected with unauthorized status
- Response indicates invalid or expired refresh token

---

## TC-AUTH-012: Refresh with invalid token

- **Priority:** High
- **Type:** Negative / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Send `POST /api/auth/refresh` with malformed or expired refresh token
2. Inspect response

### Expected Result
- Request is rejected with unauthorized status
- Response indicates invalid or expired refresh token

---

## TC-SEC-001: Access protected endpoint without token

- **Priority:** Critical
- **Type:** Security / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Call protected endpoint such as `POST /api/users/update-persona` without Authorization header
2. Inspect response

### Expected Result
- Request is rejected as unauthorized

---

## TC-SEC-002: Access protected endpoint with malformed bearer token

- **Priority:** Critical
- **Type:** Security / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Call protected endpoint with malformed bearer token
2. Inspect response

### Expected Result
- Request is rejected
- No authenticated principal is established

---

## TC-SEC-003: Student token can access student endpoints

- **Priority:** Critical
- **Type:** Security / API
- **Preconditions:** Valid STUDENT token
- **Status:** Not Run

### Steps
1. Call allowed student endpoint such as `GET /api/students/get-student?email=...`
2. Inspect response

### Expected Result
- Request succeeds if data exists

---

## TC-SEC-004: Tutor token can access tutor endpoints

- **Priority:** Critical
- **Type:** Security / API
- **Preconditions:** Valid TUTOR token
- **Status:** Not Run

### Steps
1. Call allowed tutor endpoint such as `GET /api/tutors/get-tutor?email=...`
2. Inspect response

### Expected Result
- Request succeeds if data exists

---

## TC-SEC-005: Parent token can access parent endpoints

- **Priority:** High
- **Type:** Security / API
- **Preconditions:** Valid PARENT token
- **Status:** Not Run

### Steps
1. Call `GET /api/parents`
2. Inspect response

### Expected Result
- Request succeeds

---

## TC-SEC-006: Student token blocked from parent-only restricted combinations if not allowed

- **Priority:** High
- **Type:** Security / API
- **Preconditions:** Valid STUDENT token
- **Status:** Not Run

### Steps
1. Call `GET /api/parents`
2. Inspect response

### Expected Result
- Request is rejected because students are not allowed on `/api/parents/**`

---

## TC-SEC-007: Auth endpoints accessible without token

- **Priority:** High
- **Type:** Security / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Call `/api/auth/login` without token
2. Call `/api/auth/register` without token

### Expected Result
- Requests are allowed to reach controller logic

---

## TC-SEC-008: JWT filter ignores `/api/auth/**`

- **Priority:** Medium
- **Type:** Security / Integration
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Call auth endpoints with and without Authorization header
2. Observe behavior/logs

### Expected Result
- Auth endpoints are not blocked by JWT filter’s normal auth enforcement path

---

## TC-ROLE-001: Student role selection updates persona and navigates to student onboarding

- **Priority:** Critical
- **Type:** Functional / UI / Integration
- **Preconditions:** Logged-in user at MainPage
- **Status:** Not Run

### Steps
1. Open MainPage
2. Tap Student
3. Allow persona update request to complete

### Expected Result
- Loading indicator appears while request runs
- Persona is updated
- User navigates to StudentOnboardingView
- Error message is not shown on success

---

## TC-ROLE-002: Parent role selection updates persona and navigates to parent home

- **Priority:** High
- **Type:** Functional / UI / Integration
- **Preconditions:** Logged-in user at MainPage
- **Status:** Not Run

### Steps
1. Open MainPage
2. Tap Parent

### Expected Result
- Persona is updated
- User navigates to ParentHomePage

---

## TC-ROLE-003: Tutor role selection updates persona, triggers tutor onboarding, and navigates to tutor home

- **Priority:** Critical
- **Type:** Functional / UI / Integration
- **Preconditions:** Logged-in user at MainPage
- **Status:** Not Run

### Steps
1. Open MainPage
2. Tap Tutor

### Expected Result
- Persona update request succeeds
- Tutor onboarding request succeeds
- User navigates to TutorHomePage

---

## TC-ROLE-004: Role selection shows error when persona update fails

- **Priority:** High
- **Type:** Negative / UI
- **Preconditions:** Force persona update API failure
- **Status:** Not Run

### Steps
1. Open MainPage
2. Tap any role
3. Simulate failure in role update request

### Expected Result
- Error text is shown
- Navigation does not continue
- Loading state ends

---

## TC-API-USER-001: Update persona with authenticated user

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** Valid token
- **Status:** Not Run

### Steps
1. Send `POST /api/users/update-persona`
2. Include persona in request body

### Expected Result
- User persona is updated for authenticated email
- New access token is returned
- New refresh token is returned

---

## TC-API-USER-002: Update persona without token

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Send `POST /api/users/update-persona` without token

### Expected Result
- Request is rejected as unauthorized

---

## TC-API-USER-003: Update grade with authenticated user

- **Priority:** High
- **Type:** Functional / API
- **Preconditions:** Valid token
- **Status:** Not Run

### Steps
1. Send `POST /api/users/update-grade`
2. Include grade in request body

### Expected Result
- Grade is updated for authenticated user
- Success message is returned

---

## TC-API-USER-004: Update grade with invalid payload

- **Priority:** Medium
- **Type:** Negative / API
- **Preconditions:** Valid token
- **Status:** Not Run

### Steps
1. Send `POST /api/users/update-grade` with malformed or unusable data
2. Inspect response

### Expected Result
- If update fails, API returns failure response

---

## TC-ONB-001: Student onboarding step 1 allows grade selection

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** StudentOnboardingView is open
- **Status:** Not Run

### Steps
1. Open step 1
2. Change selected grade

### Expected Result
- Picker updates selected grade value

---

## TC-ONB-002: Step 1 Next updates grade and advances to step 2

- **Priority:** High
- **Type:** UI / Integration
- **Preconditions:** StudentOnboardingView at step 1
- **Status:** Not Run

### Steps
1. Select a grade
2. Tap Next

### Expected Result
- Grade update request is attempted
- View advances to step 2

---

## TC-ONB-003: Step 2 shows all subjects and supports multi-select

- **Priority:** High
- **Type:** UI
- **Preconditions:** StudentOnboardingView at step 2
- **Status:** Not Run

### Steps
1. Tap multiple subject cards
2. Observe selected count and highlighting

### Expected Result
- Subjects can be toggled on
- Selected count updates
- Selected subjects show visual confirmation

---

## TC-ONB-004: Step 2 Next disabled when no subject selected

- **Priority:** Critical
- **Type:** Validation / UI
- **Preconditions:** Step 2 with zero selected subjects
- **Status:** Not Run

### Steps
1. Reach step 2
2. Ensure no subjects are selected

### Expected Result
- Next button is disabled

---

## TC-ONB-005: Step 2 Next enabled when at least one subject selected

- **Priority:** High
- **Type:** Validation / UI
- **Preconditions:** Step 2
- **Status:** Not Run

### Steps
1. Select at least one subject

### Expected Result
- Next button becomes enabled

---

## TC-ONB-006: Step 3 loads tutor-connected classes

- **Priority:** High
- **Type:** Integration / UI
- **Preconditions:** Reach step 3 with valid logged-in email
- **Status:** Not Run

### Steps
1. Navigate to step 3
2. Wait for loadTutorClasses task

### Expected Result
- Loading indicator appears while fetching
- Matched classes populate if available

---

## TC-ONB-007: Step 3 shows empty state when no classes exist

- **Priority:** High
- **Type:** UI / Empty State
- **Preconditions:** No matched tutor classes for email
- **Status:** Not Run

### Steps
1. Navigate to step 3 for user without linked classes

### Expected Result
- Empty-state message is shown
- Finish remains available because no class selection is required

---

## TC-ONB-008: Step 3 requires class selection when classes exist

- **Priority:** Critical
- **Type:** Validation / UI
- **Preconditions:** Step 3 with one or more matched classes
- **Status:** Not Run

### Steps
1. Navigate to step 3 with matched classes
2. Do not select any class

### Expected Result
- Finish button is disabled

---

## TC-ONB-009: Step 3 Finish enabled when at least one class selected

- **Priority:** High
- **Type:** Validation / UI
- **Preconditions:** Step 3 with matched classes
- **Status:** Not Run

### Steps
1. Select one or more classes

### Expected Result
- Finish button becomes enabled

---

## TC-ONB-010: Back button returns to previous onboarding step

- **Priority:** Medium
- **Type:** UI
- **Preconditions:** On step 2 or 3
- **Status:** Not Run

### Steps
1. Tap Back

### Expected Result
- Step value decrements by one
- Prior screen content is shown

---

## TC-ONB-011: Finish submits onboarding and navigates to student home

- **Priority:** Critical
- **Type:** Functional / Integration
- **Preconditions:** Step 3 with valid state
- **Status:** Not Run

### Steps
1. Tap Finish
2. Allow onboardStudent request to complete

### Expected Result
- Student onboarding request succeeds
- Navigation moves to StudentHomePage

---

## TC-ONB-012: Onboarding continues even if update-grade request silently fails

- **Priority:** Medium
- **Type:** Negative / Behavior
- **Preconditions:** Simulate updateGrade failure on step 1
- **Status:** Not Run

### Steps
1. Tap Next on step 1 while forcing updateGrade failure

### Expected Result
- View may still advance because error is ignored with `try?`
- This should be documented as current behavior

---

## TC-STU-001: StudentHomePage loads student, subjects, and meetings on launch

- **Priority:** Critical
- **Type:** Functional / Integration
- **Preconditions:** Valid authenticated student user
- **Status:** Not Run

### Steps
1. Open StudentHomePage
2. Allow task to execute

### Expected Result
- Student record is fetched by email
- Student subjects are fetched
- Student meetings are fetched
- Tabs render with loaded data

---

## TC-STU-002: StudentHomePage shows loading state while data fetch is in progress

- **Priority:** High
- **Type:** UI
- **Preconditions:** Slow network or mocked delay
- **Status:** Not Run

### Steps
1. Open StudentHomePage
2. Observe UI during fetch

### Expected Result
- Loading indicator appears on relevant tabs

---

## TC-STU-003: StudentHomePage shows error message when load fails

- **Priority:** High
- **Type:** Negative / UI
- **Preconditions:** Force fetch failure
- **Status:** Not Run

### Steps
1. Open StudentHomePage
2. Simulate failure in one of the required API calls

### Expected Result
- Error text is shown
- Success content is not shown for that state

---

## TC-STU-004: Lessons tab displays StudentLessonsView when student loads

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** Student data loaded
- **Status:** Not Run

### Steps
1. Open StudentHomePage
2. Stay on Lessons tab

### Expected Result
- StudentLessonsView is shown with correct student data

---

## TC-STU-005: Calendar tab displays StudentCalendarView with schedule

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** Schedule loaded
- **Status:** Not Run

### Steps
1. Switch to Calendar tab

### Expected Result
- Calendar renders schedule data

---

## TC-STU-006: Profile tab displays StudentProfileView with student data

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** Student data loaded
- **Status:** Not Run

### Steps
1. Switch to Profile tab

### Expected Result
- Profile form is populated from student data

---

## TC-LESSON-001: Student lessons stats calculate completed lessons correctly

- **Priority:** High
- **Type:** UI / Calculation
- **Preconditions:** Student with subject data
- **Status:** Not Run

### Steps
1. Open StudentLessonsView
2. Compare displayed completed count with subject lesson data

### Expected Result
- Completed count equals sum of completed lessons across subjects

---

## TC-LESSON-002: Student lessons stats calculate in-progress lessons correctly

- **Priority:** High
- **Type:** UI / Calculation
- **Preconditions:** Student with subject data
- **Status:** Not Run

### Steps
1. Open StudentLessonsView
2. Compare displayed in-progress count with total minus completed across subjects

### Expected Result
- In-progress count is correct

---

## TC-LESSON-003: Student lessons stats calculate achievements correctly

- **Priority:** Medium
- **Type:** UI / Calculation
- **Preconditions:** Student with subject data
- **Status:** Not Run

### Steps
1. Open StudentLessonsView
2. Compare achievements count with fully completed subjects

### Expected Result
- Achievements count matches number of subjects where completed lessons equal total lessons

---

## TC-LESSON-004: Subject cards navigate to lesson page from StudentLessonsView

- **Priority:** High
- **Type:** Navigation / UI
- **Preconditions:** Student with at least one subject
- **Status:** Not Run

### Steps
1. Tap a subject card in StudentLessonsView

### Expected Result
- App navigates to LessonPage

---

## TC-LESSON-005: LessonPage loads questions from bundled JSON on appear

- **Priority:** High
- **Type:** Functional / UI
- **Preconditions:** `example_questions.json` included in app target
- **Status:** Not Run

### Steps
1. Open LessonPage
2. Observe question list

### Expected Result
- Questions are loaded and displayed
- No crash occurs

---

## TC-LESSON-006: LessonPage handles missing JSON file gracefully

- **Priority:** Medium
- **Type:** Negative / UI
- **Preconditions:** Remove or misconfigure JSON resource
- **Status:** Not Run

### Steps
1. Open LessonPage with missing file

### Expected Result
- File-not-found path is handled
- Empty question list is shown
- App does not crash

---

## TC-LESSON-007: Selecting correct answer marks choice with green success state

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** LessonPage loaded
- **Status:** Not Run

### Steps
1. Select the correct answer on a question

### Expected Result
- Selected row shows green background
- Checkmark icon appears

---

## TC-LESSON-008: Selecting wrong answer marks choice with red failure state

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** LessonPage loaded
- **Status:** Not Run

### Steps
1. Select an incorrect answer

### Expected Result
- Selected row shows red background
- X icon appears

---

## TC-LESSON-009: Once an answer is selected, other answers are disabled

- **Priority:** High
- **Type:** Validation / UI
- **Preconditions:** LessonPage loaded
- **Status:** Not Run

### Steps
1. Select any answer for a question
2. Attempt to select a different answer

### Expected Result
- Other answer buttons are disabled for that question

---

## TC-LESSON-010: Submit button dismisses LessonPage

- **Priority:** Medium
- **Type:** Navigation / UI
- **Preconditions:** LessonPage opened from navigation stack
- **Status:** Not Run

### Steps
1. Tap Submit

### Expected Result
- LessonPage dismisses

---

## TC-CAL-001: Student calendar renders with title “My Schedule”

- **Priority:** Medium
- **Type:** UI
- **Preconditions:** StudentCalendarView loaded
- **Status:** Not Run

### Steps
1. Open StudentCalendarView

### Expected Result
- Shared calendar view shows title “My Schedule”

---

## TC-CAL-002: Tutor calendar renders with title “My Students”

- **Priority:** Medium
- **Type:** UI
- **Preconditions:** CalendarView loaded for tutor
- **Status:** Not Run

### Steps
1. Open tutor CalendarView

### Expected Result
- Shared calendar view shows title “My Students”

---

## TC-CAL-003: Calendar highlights weekdays that have sessions

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** Sessions provided
- **Status:** Not Run

### Steps
1. Open shared calendar with sessions on known weekdays
2. Inspect month grid dots

### Expected Result
- Orange dots appear on weekdays with sessions

---

## TC-CAL-004: Selecting day updates selectedDate styling

- **Priority:** Medium
- **Type:** UI
- **Preconditions:** Calendar displayed
- **Status:** Not Run

### Steps
1. Tap a day in the month grid

### Expected Result
- Selected date changes visually
- Forecast section highlights selected day

---

## TC-CAL-005: Forecast section shows “No sessions scheduled” for empty day

- **Priority:** Medium
- **Type:** Empty State / UI
- **Preconditions:** Day in forecast has no sessions
- **Status:** Not Run

### Steps
1. Open calendar with at least one empty forecast day

### Expected Result
- Empty day row shows no sessions scheduled

---

## TC-CAL-006: Forecast section shows session cards for days with sessions

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** At least one day with sessions
- **Status:** Not Run

### Steps
1. Open calendar
2. Inspect days with sessions

### Expected Result
- Session cards render with participant, time, duration, subject, and optional goal/secondary label

---

## TC-CAL-007: Month navigation previous/next updates displayed month

- **Priority:** Medium
- **Type:** UI
- **Preconditions:** CalendarMonthView loaded
- **Status:** Not Run

### Steps
1. Tap previous month button
2. Tap next month button

### Expected Result
- Month/year title updates appropriately

---

## TC-SPROF-001: Student profile loads initial values from student model

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** StudentProfileView created with student
- **Status:** Not Run

### Steps
1. Open StudentProfileView

### Expected Result
- First name, last name, email, grade, notifications, and subjects populate from student data

---

## TC-SPROF-002: Save Changes sends updateStudent request

- **Priority:** Critical
- **Type:** Functional / Integration
- **Preconditions:** StudentProfileView loaded
- **Status:** Not Run

### Steps
1. Modify one or more profile fields
2. Tap Save Changes

### Expected Result
- Update request is sent
- Loading state appears
- Success alert appears if request succeeds

---

## TC-SPROF-003: Save Changes failure shows alert

- **Priority:** High
- **Type:** Negative / UI
- **Preconditions:** Force updateStudent failure
- **Status:** Not Run

### Steps
1. Tap Save Changes while API failure is simulated

### Expected Result
- Failure alert appears
- Error message includes failure reason

---

## TC-SPROF-004: Save button disabled while update is in progress

- **Priority:** Medium
- **Type:** UI / Validation
- **Preconditions:** Update request running
- **Status:** Not Run

### Steps
1. Tap Save Changes
2. Inspect button while request is active

### Expected Result
- Button is disabled
- Label changes to “Saving...”

---

## TC-SPROF-005: Sign Out calls logout and exits authenticated session

- **Priority:** Critical
- **Type:** Functional / UI
- **Preconditions:** Logged-in user on StudentProfileView
- **Status:** Not Run

### Steps
1. Tap Sign Out

### Expected Result
- Logout is executed
- Session/token state is cleared
- User is returned to unauthenticated flow as implemented by app container

---

## TC-TUTOR-001: TutorHomePage loads tutor, students, subjects, and meetings on launch

- **Priority:** Critical
- **Type:** Functional / Integration
- **Preconditions:** Valid authenticated tutor user
- **Status:** Not Run

### Steps
1. Open TutorHomePage
2. Allow task to execute

### Expected Result
- Tutor is fetched by email
- Tutor’s students are fetched
- Each student’s subjects are fetched
- Tutor meetings are fetched
- Tabs render with loaded data

---

## TC-TUTOR-002: TutorHomePage shows loading state during fetch

- **Priority:** High
- **Type:** UI
- **Preconditions:** Slow network or mocked delay
- **Status:** Not Run

### Steps
1. Open TutorHomePage
2. Observe while API calls are in progress

### Expected Result
- Loading indicator appears on appropriate tabs

---

## TC-TUTOR-003: TutorHomePage shows error state on failure

- **Priority:** High
- **Type:** Negative / UI
- **Preconditions:** Simulate backend failure
- **Status:** Not Run

### Steps
1. Open TutorHomePage
2. Force data fetch failure

### Expected Result
- Error message is shown

---

## TC-TUTOR-004: Students tab displays list of students

- **Priority:** High
- **Type:** UI
- **Preconditions:** Students loaded
- **Status:** Not Run

### Steps
1. Open Students tab

### Expected Result
- Student cards render for each student

---

## TC-TUTOR-005: Plus button opens AddStudentSheet

- **Priority:** High
- **Type:** UI / Navigation
- **Preconditions:** TutorHomePage loaded
- **Status:** Not Run

### Steps
1. Tap plus button in StudentListView

### Expected Result
- AddStudentSheet opens as a sheet

---

## TC-TUTOR-006: Student card navigates to StudentDetailView

- **Priority:** High
- **Type:** Navigation / UI
- **Preconditions:** At least one student available
- **Status:** Not Run

### Steps
1. Tap a student card

### Expected Result
- App navigates to StudentDetailView

---

## TC-TUTOR-007: Tutor calendar tab renders shared calendar

- **Priority:** High
- **Type:** UI
- **Preconditions:** Schedule loaded
- **Status:** Not Run

### Steps
1. Switch to Calendar tab

### Expected Result
- Shared calendar view renders tutor schedule

---

## TC-TUTOR-008: Tutor profile renders tutor info when tutor exists

- **Priority:** Medium
- **Type:** UI
- **Preconditions:** Tutor loaded
- **Status:** Not Run

### Steps
1. Switch to Profile tab

### Expected Result
- Tutor name and email are displayed
- Account information section is shown

---

## TC-TUTOR-009: Tutor profile sign out logs user out

- **Priority:** Critical
- **Type:** Functional / UI
- **Preconditions:** Tutor logged in
- **Status:** Not Run

### Steps
1. Tap Sign Out on tutor profile

### Expected Result
- Logout runs successfully

---

## TC-ADD-001: AddStudentSheet disables button for invalid email

- **Priority:** Critical
- **Type:** Validation / UI
- **Preconditions:** AddStudentSheet open
- **Status:** Not Run

### Steps
1. Enter invalid email text

### Expected Result
- Add Student button remains disabled

---

## TC-ADD-002: AddStudentSheet enables button for valid email

- **Priority:** High
- **Type:** Validation / UI
- **Preconditions:** AddStudentSheet open
- **Status:** Not Run

### Steps
1. Enter syntactically valid email

### Expected Result
- Add Student button becomes enabled

---

## TC-ADD-003: AddStudentSheet trims whitespace and lowercases email

- **Priority:** Medium
- **Type:** Validation / UI
- **Preconditions:** AddStudentSheet open
- **Status:** Not Run

### Steps
1. Enter email with uppercase letters and surrounding spaces
2. Submit request

### Expected Result
- Request uses trimmed lowercase email

---

## TC-ADD-004: AddStudent success shows success alert and closes sheet on OK

- **Priority:** High
- **Type:** Functional / UI / Integration
- **Preconditions:** Valid tutor ID and existing student email
- **Status:** Not Run

### Steps
1. Enter valid student email
2. Tap Add Student
3. Tap OK on success alert

### Expected Result
- Success alert is shown
- Sheet closes
- Parent screen can reload data

---

## TC-ADD-005: AddStudent failure shows error alert

- **Priority:** High
- **Type:** Negative / UI
- **Preconditions:** Force API failure or use unsupported email
- **Status:** Not Run

### Steps
1. Submit invalid/nonexistent/duplicate student email

### Expected Result
- Error alert is shown
- Sheet remains open

---

## TC-ADD-006: AddStudent shows loading indicator during request

- **Priority:** Medium
- **Type:** UI
- **Preconditions:** AddStudentSheet open
- **Status:** Not Run

### Steps
1. Tap Add Student
2. Observe button area during request

### Expected Result
- ProgressView replaces button text
- Button remains effectively unavailable during call

---

## TC-TUTOR-010: StudentDetailView displays computed stats

- **Priority:** Medium
- **Type:** UI / Calculation
- **Preconditions:** StudentDetailView open
- **Status:** Not Run

### Steps
1. Open StudentDetailView for a student with known data

### Expected Result
- Progress percentage, total hours, and total subjects are calculated correctly

---

## TC-TUTOR-011: StudentDetailView shows upcoming session text or fallback

- **Priority:** Medium
- **Type:** UI / Functional
- **Preconditions:** Student has completed/incomplete lessons
- **Status:** Not Run

### Steps
1. Open StudentDetailView

### Expected Result
- If incomplete lesson exists, text shows next lesson title and date
- Otherwise fallback text is shown

---

## TC-TUTOR-012: Subject card in StudentDetailView navigates to SubjectLessonsView

- **Priority:** High
- **Type:** Navigation / UI
- **Preconditions:** Student has at least one subject
- **Status:** Not Run

### Steps
1. Tap a subject card in StudentDetailView

### Expected Result
- App navigates to SubjectLessonsView

---

## TC-TUTOR-013: SubjectLessonsView separates remaining and completed lessons

- **Priority:** High
- **Type:** UI / Functional
- **Preconditions:** Subject with mix of lesson completion states
- **Status:** Not Run

### Steps
1. Open SubjectLessonsView

### Expected Result
- Remaining lessons show under “Up Next”
- Completed lessons show under “Completed”

---

## TC-TUTOR-014: SubjectLessonsView stats show completed and remaining counts

- **Priority:** Medium
- **Type:** UI / Calculation
- **Preconditions:** SubjectLessonsView open
- **Status:** Not Run

### Steps
1. Open SubjectLessonsView for known subject

### Expected Result
- Counts match lesson completion data

---

## TC-API-STU-001: Get all students

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Authorized user with allowed role
- **Status:** Not Run

### Steps
1. Send `GET /api/students/all`

### Expected Result
- List of student records is returned

---

## TC-API-STU-002: Get student by ID

- **Priority:** High
- **Type:** API
- **Preconditions:** Student exists
- **Status:** Not Run

### Steps
1. Send `GET /api/students/{id}`

### Expected Result
- Correct student record is returned

---

## TC-API-STU-003: Get student by invalid ID

- **Priority:** Medium
- **Type:** Negative / API
- **Preconditions:** Student ID does not exist
- **Status:** Not Run

### Steps
1. Send `GET /api/students/{id}` with nonexistent ID

### Expected Result
- API returns not found/error behavior from queryForMap path
- Error is handled/documented

---

## TC-API-STU-004: Add student manually

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Authorized role, valid payload
- **Status:** Not Run

### Steps
1. Send `POST /api/students/add`

### Expected Result
- Student is inserted
- Success message returned

---

## TC-API-STU-005: Update student profile successfully

- **Priority:** Critical
- **Type:** API
- **Preconditions:** Student exists
- **Status:** Not Run

### Steps
1. Send `PUT /api/students/{id}` with valid fields

### Expected Result
- Response returns success true and success message

---

## TC-API-STU-006: Update nonexistent student returns failure payload

- **Priority:** High
- **Type:** Negative / API
- **Preconditions:** ID does not exist
- **Status:** Not Run

### Steps
1. Send `PUT /api/students/{id}` with nonexistent ID

### Expected Result
- Response indicates success false
- Message indicates student not found

---

## TC-API-STU-007: Delete existing student

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Student exists
- **Status:** Not Run

### Steps
1. Send `DELETE /api/students/{id}`

### Expected Result
- Success message is returned

---

## TC-API-STU-008: Delete nonexistent student

- **Priority:** Medium
- **Type:** Negative / API
- **Preconditions:** ID does not exist
- **Status:** Not Run

### Steps
1. Send `DELETE /api/students/{id}` with nonexistent ID

### Expected Result
- Not found message is returned

---

## TC-API-STU-009: Onboard new student creates student and subject links

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** User exists in `users`, no student exists for email
- **Status:** Not Run

### Steps
1. Send `POST /api/students/onboard`
2. Provide email, grade, and subjects list

### Expected Result
- Student row is created from users table data
- Subject links are inserted into `student_subjects`
- Success payload returns student ID

---

## TC-API-STU-010: Onboard existing student updates grade and replaces subjects

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** Student already exists
- **Status:** Not Run

### Steps
1. Send `POST /api/students/onboard` for existing student with new grade/subjects

### Expected Result
- Existing student is updated
- Old student_subjects are deleted
- New subject list is inserted
- No duplicate student row is created

---

## TC-API-STU-011: Onboard student with empty subject list clears old subjects

- **Priority:** High
- **Type:** Edge Case / API
- **Preconditions:** Existing student with current subject links
- **Status:** Not Run

### Steps
1. Send `POST /api/students/onboard` with empty or null subjects

### Expected Result
- Old subject links are removed
- No new subject links are inserted

---

## TC-API-STU-012: Get student subjects

- **Priority:** High
- **Type:** API
- **Preconditions:** Student exists
- **Status:** Not Run

### Steps
1. Send `GET /api/students/{id}/subjects`

### Expected Result
- Student subject rows are returned

---

## TC-API-STU-013: Get student meetings

- **Priority:** High
- **Type:** API
- **Preconditions:** Student exists
- **Status:** Not Run

### Steps
1. Send `GET /api/students/{id}/meetings`

### Expected Result
- Meetings for student are returned

---

## TC-API-STU-014: Get tutor classes linked to student email

- **Priority:** High
- **Type:** API
- **Preconditions:** Student linked through student_tutor_classes
- **Status:** Not Run

### Steps
1. Send `GET /api/students/my-classes?email=...`

### Expected Result
- Matching tutor class records are returned with tutor name fields

---

## TC-API-STU-015: Get student by email success

- **Priority:** Critical
- **Type:** API
- **Preconditions:** Student exists for email
- **Status:** Not Run

### Steps
1. Send `GET /api/students/get-student?email=...`

### Expected Result
- Response indicates success true
- Student object is returned

---

## TC-API-STU-016: Get student by email failure

- **Priority:** High
- **Type:** Negative / API
- **Preconditions:** No student exists for email
- **Status:** Not Run

### Steps
1. Send `GET /api/students/get-student?email=...` with unknown email

### Expected Result
- Response indicates success false
- Message indicates no student found

---

## TC-API-TUT-001: Get all tutors

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Authorized role
- **Status:** Not Run

### Steps
1. Send `GET /api/tutors`

### Expected Result
- Tutor list is returned

---

## TC-API-TUT-002: Get tutor by ID

- **Priority:** High
- **Type:** API
- **Preconditions:** Tutor exists
- **Status:** Not Run

### Steps
1. Send `GET /api/tutors/{id}`

### Expected Result
- Correct tutor record is returned

---

## TC-API-TUT-003: Add tutor manually

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Valid payload
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/add`

### Expected Result
- Tutor is created
- Success message is returned

---

## TC-API-TUT-004: Get tutor classes

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Tutor exists
- **Status:** Not Run

### Steps
1. Send `GET /api/tutors/{id}/classes`

### Expected Result
- Tutor class rows are returned

---

## TC-API-TUT-005: Get tutor students

- **Priority:** Critical
- **Type:** API
- **Preconditions:** Tutor has linked students
- **Status:** Not Run

### Steps
1. Send `GET /api/tutors/{id}/students`

### Expected Result
- Distinct linked students are returned

---

## TC-API-TUT-006: Get tutor meetings

- **Priority:** High
- **Type:** API
- **Preconditions:** Tutor exists
- **Status:** Not Run

### Steps
1. Send `GET /api/tutors/{id}/meetings`

### Expected Result
- Meeting list is returned

---

## TC-API-TUT-007: Update tutor successfully

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Tutor exists
- **Status:** Not Run

### Steps
1. Send `PUT /api/tutors/{id}` with valid payload

### Expected Result
- Success message is returned

---

## TC-API-TUT-008: Update nonexistent tutor

- **Priority:** Medium
- **Type:** Negative / API
- **Preconditions:** Tutor does not exist
- **Status:** Not Run

### Steps
1. Send `PUT /api/tutors/{id}` with nonexistent ID

### Expected Result
- Not found message is returned

---

## TC-API-TUT-009: Get tutor by email success

- **Priority:** Critical
- **Type:** API
- **Preconditions:** Tutor exists
- **Status:** Not Run

### Steps
1. Send `GET /api/tutors/get-tutor?email=...`

### Expected Result
- Success true and tutor object returned

---

## TC-API-TUT-010: Get tutor by email failure

- **Priority:** High
- **Type:** Negative / API
- **Preconditions:** Tutor does not exist
- **Status:** Not Run

### Steps
1. Send `GET /api/tutors/get-tutor?email=...` with unknown email

### Expected Result
- Success false and no tutor found message returned

---

## TC-API-TUT-011: Delete tutor success

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Tutor exists
- **Status:** Not Run

### Steps
1. Send `DELETE /api/tutors/{id}`

### Expected Result
- Success message is returned

---

## TC-API-TUT-012: Delete tutor nonexistent

- **Priority:** Medium
- **Type:** Negative / API
- **Preconditions:** Tutor does not exist
- **Status:** Not Run

### Steps
1. Send `DELETE /api/tutors/{id}` with nonexistent ID

### Expected Result
- Not found message is returned

---

## TC-API-TUT-013: Add student to tutor with empty email

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** Valid tutor ID
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/{id}/add-student` with empty email

### Expected Result
- Request is rejected
- Message states email is required

---

## TC-API-TUT-014: Add student to tutor with invalid email format

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** Valid tutor ID
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/{id}/add-student` with invalid email format

### Expected Result
- Request is rejected
- Message states invalid email format

---

## TC-API-TUT-015: Add student to tutor when student does not exist

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** Tutor ID valid, no student for email
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/{id}/add-student` with unknown student email

### Expected Result
- 404-style failure response
- Message states no student found

---

## TC-API-TUT-016: Add student to tutor creates default class if needed

- **Priority:** High
- **Type:** Functional / API
- **Preconditions:** Tutor has no tutor_classes row, student exists
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/{id}/add-student` with valid student email

### Expected Result
- Default “General” class is created
- Student is linked to that class
- Success response is returned

---

## TC-API-TUT-017: Add student to tutor prevents duplicate link

- **Priority:** Critical
- **Type:** Negative / API
- **Preconditions:** Student already linked to tutor’s default class
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/{id}/add-student` again with same student email

### Expected Result
- Conflict-style response returned
- Message states student is already added

---

## TC-API-TUT-018: Add student to tutor succeeds when valid and not already linked

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** Student exists and not linked
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/{id}/add-student` with valid student email

### Expected Result
- Link row is inserted
- Success true is returned

---

## TC-API-TUT-019: Tutor onboard creates new tutor from existing user

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** User exists in users table, tutor does not yet exist
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/onboard` with email for existing user

### Expected Result
- Tutor row is created from users table names
- Success payload returns tutor ID

---

## TC-API-TUT-020: Tutor onboard returns existing tutor without duplicate creation

- **Priority:** Critical
- **Type:** Functional / API
- **Preconditions:** Tutor already exists
- **Status:** Not Run

### Steps
1. Send `POST /api/tutors/onboard` with existing tutor email

### Expected Result
- Existing tutor ID is returned
- No duplicate tutor row is created

---

## TC-API-TUT-021: Cleanup tutors endpoint succeeds for keep email

- **Priority:** Low
- **Type:** API / Maintenance
- **Preconditions:** Stored procedure available
- **Status:** Not Run

### Steps
1. Send `DELETE /api/tutors/cleanup?keep=...`

### Expected Result
- Success response indicates cleanup complete

---

## TC-API-MTG-001: Get all meetings

- **Priority:** High
- **Type:** API
- **Preconditions:** Authenticated user
- **Status:** Not Run

### Steps
1. Send `GET /api/meetings`

### Expected Result
- List of meetings is returned

---

## TC-API-MTG-002: Create meeting with valid payload

- **Priority:** High
- **Type:** Functional / API
- **Preconditions:** Authenticated user, valid student and tutor IDs
- **Status:** Not Run

### Steps
1. Send `POST /api/meetings`
2. Provide `student_id`, `tutor_id`, `subject`, `duration_minutes`, `date_time`

### Expected Result
- New meeting is created
- Returned payload includes inserted meeting fields

---

## TC-API-MTG-003: Create meeting with invalid or malformed timestamp

- **Priority:** Medium
- **Type:** Negative / API
- **Preconditions:** Authenticated user
- **Status:** Not Run

### Steps
1. Send `POST /api/meetings` with invalid `date_time`

### Expected Result
- Request fails due to timestamp cast/DB validation
- Failure is captured and documented

---

## TC-API-MTG-004: Delete existing meeting

- **Priority:** Medium
- **Type:** API
- **Preconditions:** Meeting exists
- **Status:** Not Run

### Steps
1. Send `DELETE /api/meetings/{id}`

### Expected Result
- Success message is returned

---

## TC-API-MTG-005: Delete nonexistent meeting

- **Priority:** Medium
- **Type:** Negative / API
- **Preconditions:** Meeting ID does not exist
- **Status:** Not Run

### Steps
1. Send `DELETE /api/meetings/{id}` with nonexistent ID

### Expected Result
- Not found message is returned

---

## TC-API-PARENT-001: Parent controller root returns expected message

- **Priority:** Low
- **Type:** API / Smoke
- **Preconditions:** Valid PARENT or TUTOR token
- **Status:** Not Run

### Steps
1. Send `GET /api/parents`

### Expected Result
- Response body indicates controller is working

---

## TC-API-PARENT-002: Parent hello endpoint returns expected message

- **Priority:** Low
- **Type:** API / Smoke
- **Preconditions:** Valid PARENT or TUTOR token
- **Status:** Not Run

### Steps
1. Send `GET /api/parents/hello`

### Expected Result
- Response body matches hello message

---

## TC-UI-001: AvatarView renders system symbol avatar using user.avatarSymbol

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** Valid User model
- **Status:** Not Run

### Steps
1. Render AvatarView with known avatar symbol

### Expected Result
- Circular symbol avatar is displayed with TutorNation blue styling

---

## TC-UI-002: SectionHeader renders title and optional subtitle

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Render SectionHeader with title only
2. Render SectionHeader with title and subtitle

### Expected Result
- Title always appears
- Subtitle appears only when provided

---

## TC-UI-003: StatBadge card style renders icon, value, and label

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Render StatBadge in `.card` style

### Expected Result
- Icon, value, and label are displayed correctly

---

## TC-UI-004: StatBadge banner style renders value and label without icon dependency

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** None
- **Status:** Not Run

### Steps
1. Render StatBadge in `.banner` style

### Expected Result
- Value and label are displayed in rounded banner badge

---

## TC-UI-005: SubjectCardView displays subject name, lesson ratio, progress, and next lesson

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** Valid Subject model
- **Status:** Not Run

### Steps
1. Render SubjectCardView with known subject values

### Expected Result
- All key fields display correctly

---

## TC-UI-006: StudentCardView displays student name, subject list, progress, and lesson completion text

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** Valid student and subjects
- **Status:** Not Run

### Steps
1. Render StudentCardView with known data

### Expected Result
- Student identity and progress text display correctly

---

## TC-UI-007: LessonCard banner style shows subject, title, duration, and formatted date

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** Valid lesson data
- **Status:** Not Run

### Steps
1. Render LessonCard in `.banner` style

### Expected Result
- Subject, title, duration, and date are shown

---

## TC-UI-008: LessonCard gradient style shows completion badge for completed lesson

- **Priority:** Low
- **Type:** UI Component
- **Preconditions:** Lesson marked completed
- **Status:** Not Run

### Steps
1. Render LessonCard in `.gradient` style with `isCompleted = true`

### Expected Result
- Completed label/checkmark is shown
- Card opacity is reduced as implemented

---

## TC-REG-001: Full student happy path

- **Priority:** Critical
- **Type:** Regression / End-to-End
- **Preconditions:** Fresh user account
- **Status:** Not Run

### Steps
1. Register or login
2. Select Student role
3. Complete onboarding
4. Open lessons, calendar, and profile tabs
5. Save profile update
6. Sign out

### Expected Result
- Entire student journey completes without blocker issues

---

## TC-REG-002: Full tutor happy path

- **Priority:** Critical
- **Type:** Regression / End-to-End
- **Preconditions:** Valid tutor-capable user
- **Status:** Not Run

### Steps
1. Login
2. Select Tutor role
3. Load TutorHomePage
4. Open students list
5. Add valid student
6. Open student detail and subject detail
7. Open calendar
8. Sign out

### Expected Result
- Entire tutor journey completes without blocker issues

---

## TC-REG-003: Auth and security smoke regression

- **Priority:** Critical
- **Type:** Regression / Security
- **Preconditions:** Tokens for multiple roles
- **Status:** Not Run

### Steps
1. Verify login
2. Verify refresh
3. Verify protected endpoint without token fails
4. Verify allowed roles succeed
5. Verify disallowed role on parent endpoint fails

### Expected Result
- Security behavior remains consistent with config
