# Test Plan

## 1. Purpose

This document defines the testing approach for the TutorNation application based on the frontend SwiftUI views and backend Spring Boot controllers provided. It covers the student flow, tutor flow, authentication, authorization, profile management, lessons, calendars, onboarding, and core API behavior.

## 2. Scope

### In Scope
- Authentication and token lifecycle
- Role/persona selection
- Student onboarding
- Student lessons, lesson questions, calendar, and profile
- Tutor students list, student detail, subject detail, calendar, add-student flow, and profile
- Backend student, tutor, meeting, auth, and user update endpoints
- Authorization rules enforced by JWT and Spring Security
- Basic UI state behavior (loading, error, success, disabled states)

### Out of Scope
- Performance/load testing
- Penetration testing
- Push notification delivery infrastructure
- App Store/device compatibility matrix beyond basic iOS functional behavior
- Database migration testing
- Full visual design QA for spacing, typography, and animation polish

## 3. Test Objectives

- Verify that critical user flows work end to end
- Confirm that the frontend reacts correctly to loading, success, and failure states
- Validate backend request/response behavior for major endpoints
- Verify that authentication and authorization rules are enforced correctly
- Catch regressions across student and tutor workflows

## 4. Test Types

### Functional Testing
Verifies that features behave according to intended user workflows and endpoint contracts.

### UI Testing
Verifies buttons, navigation, disabled states, loading indicators, alerts, and rendered content.

### API Testing
Validates backend status codes, response payloads, update behavior, and data integrity.

### Integration Testing
Checks frontend screens against expected backend responses and role-dependent navigation.

### Negative Testing
Confirms that invalid inputs, missing inputs, unauthorized requests, and duplicates are handled safely.

### Regression Testing
Ensures existing flows continue to work after changes.

## 5. Test Environment

| Item | Value |
|------|-------|
| Frontend | SwiftUI iOS app |
| Backend | Spring Boot |
| Auth | JWT access + refresh tokens |
| Database Access | JDBC / relational database |
| Environment | Local and/or staging |
| API Client | App API client and manual API tool (Postman/cURL) |
| Platform | iOS simulator and/or physical iPhone |
| Browsers | N/A for app UI; API docs/testing in browser optional |

## 6. Roles Covered

- Unauthenticated user
- Student
- Tutor
- Parent (backend/security coverage only where applicable)

## 7. Modules Under Test

### Frontend
- MainPage
- StudentOnboardingView
- StudentHomePage
- StudentLessonsView
- LessonPage
- StudentCalendarView / shared calendar components
- StudentProfileView
- TutorHomePage
- StudentListView
- AddStudentSheet
- StudentDetailView
- SubjectLessonsView
- TutorProfileView
- Shared UI components such as cards, headers, and badges

### Backend
- AuthController
- UserController
- StudentController
- TutorController
- MeetingController
- ParentController
- JwtAuthenticationFilter
- SecurityConfig
- JwtService

## 8. Entry Criteria

Testing may begin when:
- App builds successfully
- Backend starts successfully
- Database is reachable
- JWT secret and expiration settings are configured
- Seed users and sample student/tutor data are available
- Required JSON resources are included in the app target

## 9. Exit Criteria

Testing is complete when:
- All critical and high-priority cases have been executed
- All blocker issues are resolved or documented
- Student and tutor happy-path flows pass
- Auth and role-based access tests pass
- Regression checklist is completed

## 10. Assumptions

- The provided files reflect current intended behavior
- APIClient methods match the controller endpoints and payload expectations
- Existing models correctly map response payloads into app view models
- JWT tokens are stored and reused correctly by the app

## 11. Risks

- Frontend and backend contracts may drift if API payload keys change
- Role update flows may fail if refreshed tokens are not persisted client-side
- Calendar and lesson data may render incorrectly if backend data is missing or malformed
- Some controllers return plain strings while others return structured JSON, which may create inconsistency in API handling
- Security rules may allow broader access than intended if role coverage is not tested carefully

## 12. Strategy by Area

### Authentication
Test registration, login, Google login, refresh token handling, invalid credentials, and invalid refresh tokens.

### Authorization
Test protected routes with no token, invalid token, valid token, and wrong-role token.

### Student Flow
Test role selection, onboarding steps, home tab loading, lessons, calendars, profile update, and logout.

### Tutor Flow
Test tutor onboarding, tutor home loading, student list, add-student flow, student detail drill-down, subject detail drill-down, calendar, and logout.

### API Data Integrity
Test create/update/delete flows, duplicate prevention, lookup behavior, and empty result handling.

## 13. Prioritization

### Critical
- Register/login
- Role selection
- Student onboarding
- Tutor onboarding
- Student/tutor data fetch
- JWT-protected endpoint access
- Add-student flow
- Profile update
- Meeting retrieval

### High
- Lesson question interactions
- Calendar rendering
- Subject drill-down
- Refresh token flow
- Duplicate handling

### Medium
- Styling-related reusable components
- Informational/test endpoints
- Empty-state and copy validation

## 14. Deliverables

- TEST_PLAN.md
- TEST_CASES.md
- Defect list / issue tracker entries
- Execution results with pass/fail status

## 15. Pass / Fail Rules

A test passes when:
- The observed behavior matches the expected result exactly
- The correct screen, payload, status, or authorization behavior occurs
- No unintended side effects occur

A test fails when:
- The wrong screen, message, data, or status is returned
- A required action is not performed
- A protected resource is accessible when it should not be
- The app crashes, hangs, or shows inconsistent state

## 16. Traceability Map

| Area | Test Case Prefix |
|------|------------------|
| Authentication | TC-AUTH |
| Authorization / Security | TC-SEC |
| Role Selection / Persona | TC-ROLE |
| Student Onboarding | TC-ONB |
| Student Home / Tabs | TC-STU |
| Lessons / Quiz | TC-LESSON |
| Calendar | TC-CAL |
| Student Profile | TC-SPROF |
| Tutor Home / Students | TC-TUTOR |
| Add Student | TC-ADD |
| Tutor Profile | TC-TPROF |
| Student API | TC-API-STU |
| Tutor API | TC-API-TUT |
| Meeting API | TC-API-MTG |
| User Update API | TC-API-USER |
| Parent API | TC-API-PARENT |
| Shared UI Components | TC-UI |

## 17. Notes for Execution

- For API tests, record request body, headers, response code, and response body.
- For auth/security tests, test with:
  - no token
  - malformed token
  - expired token if possible
  - valid token without role
  - valid token with STUDENT/TUTOR/PARENT role
- For app tests, verify both UI state and resulting backend side effects where applicable.
