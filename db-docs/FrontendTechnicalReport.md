Here is the detailed English technical report in Markdown format.

```markdown
# Frontend Technical Report — Online Learning Management System

## 1. Executive Summary

The frontend of this project is implemented using **React + TypeScript + Vite** and is fully integrated with the **FastAPI** backend. The user interface is Persian (Farsi) with a **Right-to-Left (RTL)** layout. All pages, authentication mechanisms, user roles (Admin / Teacher / Student), and CRUD operations operate with real backend data through API calls — there is no mock data used anywhere in the application.

## 2. Project Structure

```
edu-platform/
├── index.html
├── vite.config.ts
├── tsconfig.json
├── package.json
├── .env                              # Backend URL (VITE_API_URL)
└── src/
    ├── main.tsx                      # Entry point + AuthProvider
    ├── App.tsx                       # Routing & authentication guard
    │
    ├── api/                          # Backend communication layer
    │   ├── client.ts                 # Base fetcher + JWT management
    │   ├── auth.ts                   # Login / register / me
    │   ├── courses.ts                # Course CRUD operations
    │   ├── users.ts                  # Profile, payments, user management
    │   └── index.ts                  # Enrollments, assignments, attendance,
    │                                  # certificates, reports, views
    │
    ├── context/
    │   └── AuthContext.tsx           # Global authentication state
    │
    ├── hooks/
    │   └── useApi.ts                 # Generic fetch hook with loading/error/refetch
    │
    ├── components/
    │   ├── layout/
    │   │   ├── Layout.tsx
    │   │   ├── Sidebar.tsx / .css
    │   │   └── Header.tsx / .css
    │   ├── ui/
    │   │   ├── Modal.tsx
    │   │   └── Badge.tsx
    │   └── shared/
    │       └── StatCard.tsx
    │
    ├── pages/
    │   ├── Login.tsx / .css
    │   ├── Dashboard.tsx / .css
    │   ├── Users.tsx
    │   ├── StudentsTeachers.tsx      # Two separate pages: Students & Teachers
    │   ├── Courses.tsx
    │   ├── Enrollments.tsx
    │   ├── Assignments.tsx
    │   ├── Payments.tsx
    │   ├── Attendance.tsx
    │   ├── Certificates.tsx
    │   └── Register.tsx
    │
    ├── styles/
    │   ├── variables.css             # Design tokens (colors, spacing, shadows, fonts)
    │   ├── globals.css               # Reset + base RTL styles + animations
    │   └── components.css            # Shared component styles
    │
    └── types/
        └── index.ts                  # TypeScript types for the entire domain
```

## 3. Backend Communication Layer (`src/api/`)

### `client.ts` — Core HTTP Client

- Backend URL is read from the environment variable `VITE_API_URL` (default: `http://localhost:8000`)
- Generic `request<T>(path, options)` function:
    - Automatically builds query strings from `params`
    - Automatically adds the `Authorization: Bearer <token>` header (unless `noAuth: true`)
    - Automatically sets `Content-Type: application/json`
    - Handles `204 No Content` responses
    - On `401 Unauthorized`: clears the token and redirects the user to the login page
    - On errors: throws an `ApiError` containing the HTTP status and the `detail` message from the backend
- Helper functions `saveToken` / `clearToken` for `localStorage` management

### `auth.ts`

| Function                    | Endpoint           | Description                                      |
| --------------------------- | ------------------ | ------------------------------------------------ |
| `login(email, password)`    | `POST /auth/login` | Receives a JWT and stores it automatically       |
| `register(payload)`         | `POST /auth/register` | Registers a new user (student or teacher)     |
| `getMe()`                   | `GET /auth/me`     | Retrieves the current user‘s information from the token |

### `courses.ts`

| Function                           | Endpoint                         |
| ---------------------------------- | -------------------------------- |
| `listCourses(params?)`             | `GET /courses` (filters: status, teacher_id, min/max price, search) |
| `getCourseDetails(id)`             | `GET /courses/{id}`              |
| `createCourse(data)`               | `POST /courses`                  |
| `updateCourse(id, data)`           | `PUT /courses/{id}`              |
| `deleteCourse(id)`                 | `DELETE /courses/{id}`           |
| `getCourseAssignments(id)`         | `GET /courses/{id}/assignments`  |

### `users.ts`

| Function                                   | Endpoint                              |
| ------------------------------------------ | ------------------------------------- |
| `getProfile()`                             | `GET /users/profile`                  |
| `updateProfile(data)`                      | `PUT /users/profile`                  |
| `getMyPayments()`                          | `GET /users/payments`                 |
| `getMyGrades()`                            | `GET /users/grades`                   |
| `listUsers(role?, include_deleted?)`       | `GET /users/admin/users` (Admin only) |
| `updateUserStatus(id, data)`               | `PUT /users/admin/users/{id}/status` (Admin only) |

### `index.ts` — Enrollments, Assignments, Attendance, Certificates, Reports, Views

| Category            | Functions                                                                        |
| ------------------- | -------------------------------------------------------------------------------- |
| Enrollments         | `enrollStudent`, `cancelEnrollment`                                              |
| Assignments         | `createAssignment`, `updateAssignment`, `deleteAssignment`, `submitAssignment`, `gradeSubmission`, `getSubmissionsForGrading` |
| Attendance          | `recordAttendance`                                                               |
| Certificates        | `issueCertificates`                                                              |
| Reports             | `reportTopStudents`, `reportTeacherIncome`, `reportPopularCourses`, `reportCourseEnrollments`, `reportInactiveStudents`, `reportFailedPayments`, `reportMonthlyIncome`, `reportTeacherRanking`, `reportAttendance` |
| Views               | `getStudentTranscript`, `getTeacherDashboard`, `getCourseStatistics`, `getFinancialSummary`, `getStudentCertificates` |

### `announcements.ts` (New — based on the Announcement table in the database design)

| Function                             | Endpoint                     |
| ------------------------------------ | ---------------------------- |
| `listAnnouncements(course_id?)`      | `GET /announcements`         |
| `createAnnouncement(data)`           | `POST /announcements`        |
| `updateAnnouncement(id, data)`       | `PUT /announcements/{id}`    |
| `deleteAnnouncement(id)`             | `DELETE /announcements/{id}` |

> ⚠️ **Note:** These four endpoints have not yet been implemented on the backend and must be added. See Section 13 for details.

## 4. Authentication Management (`AuthContext.tsx`)

A global Context that wraps the entire application at the `main.tsx` level:

- **State:** `user`, `loading`
- **Methods:**
    - `login(email, password)` → calls `/auth/login`, saves the token, then calls `/auth/me`
    - `logout()` → clears the token and resets the state
- **Role flags:** `isAdmin`, `isTeacher`, `isStudent` (derived from `user.role`)
- On initial mount: if a token exists in `localStorage`, it is validated with `/auth/me`; if invalid, the token is cleared.
- `useAuth()` — a simple hook for accessing this context anywhere.

## 5. `useApi` Hook

A generic fetch hook with the following signature:

```typescript
const { data, loading, error, refetch } = useApi(() => listCourses(), [search]);
```

- Automatically manages `loading` and `error` states
- Prevents race conditions using an internal counter (responses from outdated requests are ignored)
- `refetch()` for manual refresh after create/update/delete operations
- `ApiError` messages are converted into user-friendly Persian text

## 6. Authentication Guard & Routing (`App.tsx` / `main.tsx`)

- **`main.tsx`**: The entire application is rendered inside `<AuthProvider>`
- **`App.tsx`:**
    - If `loading` is `true` → displays a splash screen with logo and spinner on a dark background
    - If there is no `user` → displays the `LoginPage`
    - Otherwise → renders `AppContent` (includes `Layout` + internal state-based router)
- Routing is **state-based** (without `react-router`) using `PageKey` and a `switch/case` statement in `renderPage()`

## 7. Layout Components

### `Sidebar.tsx`

- Right-aligned collapsible side menu
- Menu items are **filtered based on the user's role**:

| Page             | Authorized Roles               |
| ---------------- | ------------------------------ |
| Dashboard        | All                            |
| Users            | Admin                          |
| Students         | Admin, Teacher                 |
| Teachers         | Admin                          |
| Courses          | All                            |
| Announcements    | All                            |
| Enrollments      | Student                        |
| Assignments      | All                            |
| Payments         | All                            |
| Attendance       | All                            |
| Certificates     | All                            |

- Bottom section of the Sidebar: avatar, user's name and role, plus a logout button

### `Header.tsx`

- Displays the active page title (mapped from `PageKey` to Persian text)
- Shows today‘s date in Persian format (`toLocaleDateString(‘fa-IR’)`)
- Displays the user‘s role and name, avatar, and a logout button

### `Layout.tsx`

- Combines `Sidebar` + `Header` + main content area with a fade-in animation

## 8. Shared UI Components

### `Modal.tsx`

- Reusable modal component with `open`, `title`, `onClose`, `children`, and `footer` props
- Closes when clicking on the backdrop
- Includes a scale-in animation

### `Badge.tsx`

- Colored badge component with variants: `success` / `warning` / `danger` / `info` / `neutral`
- Domain-specific helper functions:
    - `enrollmentBadge` / `paymentBadge` (Successful / Pending / Failed)
    - `attendanceBadge` (Present / Absent)
    - `userRoleBadge` (Admin / Teacher / Student)

### `StatCard.tsx`

- Statistical card with an icon, title, value, and upward/downward trend indicator

## 9. Pages — Detailed Implementation

### 9.1 `Login.tsx`

- Email/password form with native validation
- Inline display of API errors (using `ApiError`)
- Dark-themed design with gradient-blur backgrounds
- Loading state displays a spinner inside the button during submission

### 9.2 `Dashboard.tsx`

Dashboard content **varies by role**:

| Card / Section                  | Data Source                                 | Displayed For            |
| ------------------------------- | ------------------------------------------- | ------------------------ |
| Active courses / Total courses  | `getCourseStatistics()`                     | Admin, Teacher           |
| Total revenue                   | `getFinancialSummary()`                     | Admin only               |
| Top student (by GPA)            | `reportTopStudents(5)`                      | Admin, Teacher           |
| Most popular course             | `reportPopularCourses()`                    | Admin, Teacher           |
| Welcome message                 | `user.fullname`                             | All                      |
| Top students table              | `reportTopStudents(5)`                      | Admin, Teacher           |
| Popular courses chart           | `reportPopularCourses()`                    | Admin, Teacher           |

- Independent skeleton loading for each section
- Empty state for missing data

### 9.3 `Users.tsx` (Admin only, accessible from Sidebar)

- Full user list from `GET /users/admin/users`
- Filter by role (Admin / Teacher / Student)
- Checkbox for “Include deleted users” (`include_deleted`)
- Columns: ID, name, email, role (badge), phone number, registration date
- Actions (Admin only):
    - **Change student status** (Active/Inactive) via a modal → `updateUserStatus`
    - **Soft delete** user (`is_deleted: true`) with confirmation
- Deleted user rows are displayed with reduced opacity
- Skeleton + error state + “Retry” button

### 9.4 `StudentsTeachers.tsx` (exports two separate components)

**`StudentsPage`**

- Data from `listUsers('Student')`
- Columns: name, email, phone number, GPA (color-coded), status (Active/Inactive)

**`TeachersPage`**

- Data from `listUsers('Teacher')`
- Columns: name, email, phone number, total income (formatted in Persian Toman)

### 9.5 `Courses.tsx`

- Course list from `listCourses({ search })` with live search
- Columns: title + description, teacher, price, start/end date, capacity bar (Capacity vs EnrolledCount), status (color-coded badge for Active/Upcoming/Completed/other)
- Actions (Admin/Teacher):
    - **Add course** → `createCourse` (converts dates to ISO format)
    - **Edit course** → `updateCourse`
    - **Delete course** → `deleteCourse` with confirmation
- Modal form includes: title, description, price, capacity, start/end date (input type=date)
- Error messages are displayed inside the modal

### 9.6 `Enrollments.tsx`

- Displays the list of all available courses (`listCourses`) with status and capacity
- **“Enroll” button** (Student only):
    - Opens a modal with the pre-selected course and auto-filled price
    - Form includes: course selection, amount (editable), optional transaction code
    - Submits → `enrollStudent({ student_id, course_id, amount, transaction_id })`
    - Green success message after enrollment (auto-dismisses after 4 seconds)
- Error handling with `ApiError`

### 9.7 `Assignments.tsx`

Two tabs:

**Tab 1: “Assignments”**

- Currently displays a helper message (the list of assignments per course is viewable from the course details page)
- **“+ New Assignment” button** (Admin/Teacher) → modal includes: course selection (from `listCourses`), title, submission deadline (datetime-local), maximum score → `createAssignment`

**Tab 2: “Submissions”**

- List from `getSubmissionsForGrading({})`
- Columns: student, assignment title, course, submission date, score (or “Not graded“ badge), feedback
- **”Grade“ button** (Admin/Teacher) → modal with score field (0 to 20, step=0.5) and feedback → `gradeSubmission`

### 9.8 `Payments.tsx`

- Data from `getMyPayments()` (role‑based return: students see only their own payments, Admins see all)
- Three `StatCard` components at the top: total successful payments (in Toman), number of successful payments, number of failed payments
- Transactions table includes a “Student” column (visible only to Admin)
- Status displayed with badge (Successful / Pending / Failed)
- Transaction ID displayed in monospace font with LTR layout

### 9.9 `Attendance.tsx`

- Filter section: course selection (from `listCourses`) + student ID (editable for Admin/Teacher, defaults to the current user for Student)
- **“View” button** → calls `reportAttendance(student_id, course_id)`
- Summary display: attendance percentage as a large circular progress bar + progress bar + number of present/absent sessions
- Attendance history table with status badges (Present/Absent)
- **“+ Record Attendance” button** (Admin/Teacher) → modal: course, student ID, session date, status → `recordAttendance`

### 9.10 `Certificates.tsx`

- Data from `getStudentCertificates()`
- Displays the last 6 certificates as **visual cards** (trophy icon, course title, student name, certificate code, issue date)
- Full table of all certificates
- Certificate issuance section (Admin/Teacher):
    - Select a course (or “All courses”)
    - **“Issue Certificates” button** → calls `issueCertificates(course_id?)` with confirmation

### 9.11 `Announcements.tsx` (New)

- Filter announcements by course (`listAnnouncements(course_id?)`)
- Each announcement is displayed as a **card** containing:
    - icon, title, course name badge, relative time (e.g., “2 hours ago”), full text
    - If edited, shows “Edited: ... ago”
- Actions (Admin/Teacher):
    - **Add new announcement** → modal with course selection, title, text (textarea) → `createAnnouncement`
    - **Edit** (course cannot be changed) → `updateAnnouncement`
    - **Delete** with confirmation → `deleteAnnouncement`
- Helper function `timeAgo()` converts dates to Persian relative time strings
- Dedicated skeleton (loading cards)
- Standard empty and error states


## 10. Design System

### `variables.css`

- Complete color palette (brand, gray, semantic: success/warning/danger/info)
- Accent colors for `StatCard` components (blue, green, amber, rose, violet, teal)
- Typography tokens (Vazirmatn font, sizes, weights)
- Spacing, border radius, shadows, transitions, z‑index, Sidebar/Header dimensions

### `globals.css`

- Complete CSS reset + `direction: rtl` on `body`
- Custom scrollbar, text selection, focus ring styles
- Common animations: `fadeIn`, `slideInRight`, `scaleIn`, `pulse`, `spin`
- Shell classes: `app-shell`, `page-content`, `main-area`

### `components.css`

Styles for all reusable elements:

- Buttons (`btn-primary/secondary/danger/ghost`, sizes)
- Cards (`card`, `card-header`, `card-body`)
- Badges (5 color variants)
- Data tables (`data-table`) with hover effect and sticky header
- Forms (`form-input`, `form-select`, `form-label`) + textarea styles
- Modal (backdrop with blur, animation)
- `StatCard`
- Page header (`page-header`, `page-title`, `page-subtitle`)
- Search bar, empty state, skeleton (shimmer animation)

## 11. TypeScript Types (`types/index.ts`)

Complete types for all domain entities (aligned with the database design):

- `User` (with `UserRole`, `UserStatus`)
- `Student`, `Teacher`
- `Course` (with `CourseStatus`)
- `Enrollment` (with `EnrollmentStatus`)
- `Assignment`, `Submission`
- `Payment` (with `PaymentStatus`)
- `Attendance` (with `AttendanceStatus`)
- `Certificate`
- UI helper types: `SidebarItem`, `StatCardData`

## 12. Backend Integration Status — Complete Endpoint Table

| Page             | Consumed Endpoint(s)                                                              | Status in Backend |
| ---------------- | --------------------------------------------------------------------------------- | ----------------- |
| Login            | `/auth/login`, `/auth/me`                                                         | ✅ Existing       |
| Dashboard        | `/reports/top-students`, `/reports/popular-courses`, `/views/financial-summary`, `/views/course-statistics` | ✅ Existing |
| Users            | `/users/admin/users`, `/users/admin/users/{id}/status`                            | ✅ Existing       |
| Students/Teachers| `/users/admin/users?role=...`                                                     | ✅ Existing       |
| Courses          | `/courses` (GET/POST/PUT/DELETE)                                                  | ✅ Existing       |
| Enrollments      | `/courses`, `/enrollments/enroll`                                                 | ✅ Existing       |
| Assignments      | `/submissions/list`, `/assignments/create`, `/assignments/grade`                  | ✅ Existing       |
| Payments         | `/users/payments`                                                                 | ✅ Existing       |
| Attendance       | `/reports/attendance`, `/attendance/record`                                       | ✅ Existing       |
| Certificates     | `/views/student-certificates`, `/certificates/issue`                              | ✅ Existing       |

## 13. Environment Variables

File `.env` in the project root:

```
VITE_API_URL=http://localhost:8000
```

## 14. Running the Frontend

```bash
cd edu-platform
npm install
npm run dev
```

The application will be available at `http://localhost:5173` (assuming the backend is running on `http://localhost:8000`).

## 15. Final Summary

✅ All pages work with **real data from the API** (no mocks)  
✅ Complete JWT authentication with role-based guards in the UI  
✅ `loading` / `error` / `empty state` handling on all pages  
✅ Unified RTL Persian design with a centralized design system (CSS Variables)  
