# Hospital Management — DevOps Project

## 1. Project overview

This project delivers a streamlined **Hospital Administration Web Application** designed to manage core healthcare operational workflows, including patient administration, doctor scheduling, appointment lifecycle management, and consultation records under role-based access control.

> **Key Architectural Principle:** The hospital web application serves as a functional vehicle; the **automated DevOps delivery pipeline is the primary deliverable**.

The system is developed and graded on the establishment of an automated, resilient engineering pipeline encompassing:

- **GitHub Issues & Projects:** Transparent backlog management, task decomposition, and Definition of Done (DoD) tracking.
- **GitHub Actions (CI/CD):** Continuous Integration automating code linting, unit testing, and end-to-end integration tests on every pull request.
- **GitHub Container Registry (GHCR):** Automated container build, tagging, and publication triggered whenever verified changes merge into the `main` branch.

---

## 2. Requirement analysis

### 2.1 Functional requirements

The system implements functional capabilities organized across six core domain groups derived from the operational scenario.

#### Group 1: Patients

- **FR-01 (Patient Registration):** The system shall allow receptionists and administrators to register new patients with mandatory attributes: full name, date of birth, and contact details.
- **FR-02 (Patient Search & Profile):** The system shall allow receptionists and administrators to search for patients by full name or national identification number and display their demographic details.
- **FR-03 (Patient Record Modification):** The system shall allow receptionists and administrators to update existing patient demographic and contact information.
- **FR-04 (Patient Deactivation):** The system shall allow receptionists and administrators to deactivate a patient record while preserving all past appointments and medical history (soft deactivation; records are never purged).

#### Group 2: Doctors

- **FR-05 (Doctor Registration):** The system shall allow administrators to register medical doctors specifying their name, clinical specialisation, contact information, and availability windows.
- **FR-06 (Doctor Profile & Availability Update):** The system shall allow administrators to modify a doctor's profile, clinical specialisation, and working schedule.
- **FR-07 (Doctor Deactivation):** The system shall allow administrators to deactivate doctors who are temporarily unavailable or no longer practicing, ensuring past consultation records remain linked and intact.

#### Group 3: Appointments

- **FR-08 (Appointment Scheduling):** The system shall allow receptionists and administrators to schedule appointments by linking an active patient, an available doctor, date, time slot, and reason for consultation.
- **FR-09 (Conflict Prevention / No Double-Booking):** The system shall enforce a strict validation rule preventing double-booking; no doctor may have more than one appointment scheduled at the same date and time slot.
- **FR-10 (Appointment Status Lifecycle):** The system shall manage appointment states across three defined values: `Scheduled`, `Completed`, and `Cancelled`.
- **FR-11 (Appointment Modification & Cancellation):** The system shall allow receptionists, administrators, and the assigned doctor to cancel or reschedule appointments to an alternative available slot.

#### Group 4: Medical Records

- **FR-12 (Clinical Entry Recording):** The system shall allow doctors to record clinical notes, visit dates, and medical diagnoses following or during an appointment.
- **FR-13 (Medical Record Retention & Access):** The system shall permanently retain medical records and restrict read access strictly to authorized clinical personnel.

#### Group 5: Authentication

- **FR-14 (User Authentication):** The system shall require all users to authenticate with credentials (username/email and password) before accessing any protected feature or route.
- **FR-15 (Role Differentiation):** The system shall authenticate and distinguish between three separate staff roles: Receptionist, Doctor, and Administrator.

#### Group 6: Access and Data Export

- **FR-16 (Role-Based Route Restriction):** The system shall enforce server-side and client-side authorization ensuring users only reach functions permitted by their assigned role.
- **FR-17 (Clinical Data Export):** The system shall allow doctors to export authorized clinical consultation and diagnosis summaries as a downloadable CSV report.

---

### 2.2 Non-functional requirements

Every non-functional requirement specifies an objective threshold, numerical target, or concrete technical mechanism.

#### Stated Requirements

- **NFR-01 (Performance):** 95% of standard web requests and API endpoints must complete within two seconds ($\le 2.0\text{ s}$) under a demonstration workload of 20 concurrent simulated users against a seeded test dataset.
- **NFR-02 (Availability & Fault Recovery):** The application service must recover automatically from an unhandled process termination or container crash within 30 seconds, enforced via Docker Compose restart policies (`restart: unless-stopped`).
- **NFR-03 (Security — Credential Storage):** Passwords must never be stored in plain text; they must be hashed using `bcrypt` with a minimum work/cost factor of 10 prior to database persistence.
- **NFR-04 (Security — Access Enforcement):** Authentication and authorization must be evaluated on every protected route; unauthenticated requests must return HTTP `401 Unauthorized`, and authenticated requests lacking sufficient role privileges must return HTTP `403 Forbidden`.

#### Pipeline-Implied DevOps Requirements

- **NFR-05 (Deployability):** Any validated commit merged into the `main` branch must trigger an automated CI/CD workflow that builds a container image and publishes it to GitHub Container Registry (GHCR) without manual intervention.
- **NFR-06 (Testability):** Automated unit and integration test suites must run in GitHub Actions on every pull request targeting `main`; branch protection rules must block merges if any test fails.
- **NFR-07 (Observability):** The application must expose a dedicated health endpoint (`/health`) returning HTTP 200 and basic runtime status metrics for operational monitoring.
- **NFR-08 (Maintainability):** The repository must enforce GitHub Flow (short-lived feature branches merged exclusively through Pull Requests) and Semantic Versioning (`vMAJOR.MINOR.PATCH`) for all published container images.

---

## 3. Roles and access control

### 3.1 Role Descriptions

- **Receptionist (Front Desk):** Owns patient records and the appointment calendar. Registers, searches, edits, and deactivates patients; schedules, updates, and cancels appointments. Never accesses or alters clinical notes.
- **Doctor (Clinical):** Manages clinical care for patients scheduled with them. Records consultation dates, clinical diagnoses, and treatment notes; reviews medical history of their patients; exports authorized clinical records to CSV.
- **Administrator (System Management):** Manages user accounts, doctor registration, and system settings. Registers, updates, and deactivates doctors; assigns role-based permissions. Does not participate in clinical care.

> **Explicit Role Boundary — The Patient is Data, Not a User:**  
> Patients do not have system credentials, cannot log in, cannot book appointments online, and cannot view their medical records directly. A separate patient user role was evaluated during requirement analysis and explicitly ruled out to maintain an internal hospital administrative scope.

### 3.2 RBAC Matrix

| Capability | Receptionist | Doctor | Administrator |
| :--- | :---: | :---: | :---: |
| Register, search and edit a patient | **Yes** | No | **Yes** |
| Deactivate a patient record (soft delete) | **Yes** | No | **Yes** |
| Register, update or deactivate a doctor | No | No | **Yes** |
| Schedule an appointment | **Yes** | No | **Yes** |
| View appointments | **All** | **Own only** | **All** |
| Update or cancel an appointment | **Yes** | **Yes** *(Decided)* | **Yes** |
| Record diagnosis and visit notes | No | **Yes** | No |
| Read a patient's medical history | No | **Own patients** | **No** *(Decided)* |
| Export authorised data to CSV | No | **Yes** | **No** *(Decided)* |
| Manage user accounts and roles | No | No | **Yes** |

### 3.3 Resolution of Ambiguous Decisions

- **Update or cancel an appointment (Doctor $\rightarrow$ Yes):** Doctors need autonomy to cancel or reschedule appointments on their own roster in case of urgent surgeries or sudden medical emergencies without depending on receptionist availability.
- **Read a patient's medical history (Administrator $\rightarrow$ No):** Administrators manage infrastructure, user accounts, and technical settings; to comply with healthcare privacy regulations (GDPR/HIPAA), non-clinical administrative staff are strictly prohibited from inspecting confidential medical histories and clinical notes.
- **Export authorised data to CSV (Administrator $\rightarrow$ No):** Clinical data exports contain sensitive diagnostic details intended exclusively for treating physicians. Administrators are restricted to system-level configuration and cannot export patient health summaries.

---

## 4. Product backlog

### 4.1 Prioritised Backlog Sequence

The backlog represents a single, strictly ordered sequence prioritized according to **Dependency**, **Risk**, and **Value**.

1. **US-01 (Authentication & Access):** As any staff member, I want to log in with my username and password, so that I can securely reach only the functionality permitted by my role.
2. **US-02 (Register Doctor):** As an administrator, I want to register a doctor with their medical specialisation and schedule, so that they can be assigned to clinical consultations.
3. **US-03 (Register Patient):** As a receptionist, I want to register a new patient with their name, date of birth, and contact details, so that they can be booked for appointments.
4. **US-04 (Schedule Appointment & Conflict Prevention):** As a receptionist, I want to schedule an appointment for a patient with a designated doctor and time slot, so that patient visits are arranged without double-booking conflicts.
5. **US-05 (View Doctor Queue):** As a doctor, I want to view my scheduled appointments for the day, so that I can organize my consultation workflow and attend to patients promptly.
6. **US-06 (Record Diagnosis and Visit Notes):** As a doctor, I want to record visit dates, clinical diagnoses, and consultation notes, so that an accurate and permanent medical record is preserved.
7. **US-07 (Review Patient Medical History):** As a doctor, I want to review previous consultation histories and clinical records of patients assigned to me, so that I can make informed diagnosis and treatment decisions.
8. **US-08 (Search Patient):** As a receptionist, I want to search for registered patients by name or identification number, so that I can quickly retrieve their profiles and avoid creating duplicate records.
9. **US-09 (Edit Patient Details):** As a receptionist, I want to edit existing patient contact details, so that hospital records remain current and communication channels valid.
10. **US-10 (Update or Cancel Appointment):** As a receptionist, I want to reschedule or cancel an existing appointment upon patient request, so that schedule changes are reflected in real time.
11. **US-11 (Doctor Appointment Cancellation):** As a doctor, I want to cancel or reschedule an appointment on my own calendar during medical emergencies, so that affected patients are notified and clinic schedules updated.
12. **US-12 (Export Clinical Data to CSV):** As a doctor, I want to export my authorized consultation and diagnosis records to a CSV report, so that I can analyze patient treatment trends and satisfy reporting obligations.
13. **US-13 (Track Appointment Status):** As a receptionist, I want to update an appointment status to `Completed` or `Cancelled`, so that reception queues accurately reflect real-time attendance.
14. **US-14 (Manage User Accounts):** As an administrator, I want to create user accounts and assign roles (`Receptionist`, `Doctor`, `Administrator`), so that authorized personnel receive appropriate system credentials.
15. **US-15 (Deactivate User Accounts):** As an administrator, I want to deactivate user accounts immediately upon staff departure, so that former personnel cannot access sensitive hospital systems.
16. **US-16 (Update Doctor Information):** As an administrator, I want to update doctor profile details and availability windows, so that scheduling reflects accurate medical staff availability.
17. **US-17 (Deactivate Doctor Record):** As an administrator, I want to deactivate an unavailable doctor while retaining their historical consultations, so that medical audits and appointment history remain valid.
18. **US-18 (Deactivate Patient Record):** As a receptionist, I want to deactivate a patient record while preserving all past appointments and clinical notes, so that inactive records are archived without violating record retention policies.

### 4.2 Backlog Ordering Justification

> **Top Three Justification:**  
> **US-01 (Authentication)** must be delivered first because no role-based functionality can be verified or secured without an active user identity context. **US-02 (Doctor Registration)** and **US-03 (Patient Registration)** must precede scheduling because an appointment has a strict relational dependency on both an active physician and a registered patient. Establishing this "walking skeleton" vertical slice early mitigates integration risk and enables early containerized deployment testing.

### 4.3 Acceptance Criteria on Top Items

#### US-01: User Login and Role-Based Redirection

- **Scenario 1 — Successful authentication and role routing:**
  - **Given** an active user registered with the role `Doctor` and valid credentials,
  - **When** the user submits their username and password via `/login`,
  - **Then** the system authenticates the credentials, issues a session/token, and redirects the user to the `/doctor/dashboard` interface.
- **Scenario 2 — Rejection of unauthenticated protected requests:**
  - **Given** an unauthenticated visitor,
  - **When** the visitor attempts to navigate directly to `/doctor/dashboard` or perform a `GET` request to `/api/patients`,
  - **Then** the application denies access, issues an HTTP `401 Unauthorized` status, and redirects the browser to `/login`.

#### US-04: Schedule Appointment with Double-Booking Prevention

- **Scenario 1 — Successful appointment booking:**
  - **Given** Doctor Smith has no existing appointments at 10:00 on 14 October and Patient Doe is active,
  - **When** the receptionist books an appointment for Patient Doe with Doctor Smith for 10:00 on 14 October,
  - **Then** the system records the appointment with status `Scheduled` and displays it on the doctor's calendar.
- **Scenario 2 — Prevention of double-booking conflict:**
  - **Given** Doctor Smith already has an appointment scheduled at 10:00 on 14 October,
  - **When** the receptionist attempts to book another patient with Doctor Smith at 10:00 on 14 October,
  - **Then** the system rejects the booking request, displays a clear conflict warning indicating the time slot is unavailable, and makes no change to existing records.

#### US-06: Record Clinical Diagnosis and Consultation Notes

- **Scenario 1 — Clinical note entry by attending doctor:**
  - **Given** an authenticated doctor attending a scheduled appointment with Patient Doe,
  - **When** the doctor inputs the diagnosis and clinical consultation notes and clicks "Save Record",
  - **Then** the record is persisted with the current timestamp and author ID, and the appointment status transitions to `Completed`.
- **Scenario 2 — Unauthorized clinical edit prevention:**
  - **Given** an authenticated user logged in as a `Receptionist`,
  - **When** the user submits a `POST` or `PUT` request to `/api/records`,
  - **Then** the server rejects the request with an HTTP `403 Forbidden` error and creates no record.

### 4.4 Definition of Done (DoD)

A backlog item is declared **Done** and eligible for release only when all of the following conditions are satisfied:

1. **Pull Request Workflow:** Code is committed on a feature branch and merged into `main` strictly through an approved Pull Request (no direct pushes to `main`).
2. **Automated Unit Testing:** Unit tests verifying business logic and validation rules are written, executed, and passing in GitHub Actions CI.
3. **End-to-End Test Suite:** Automated end-to-end and integration tests covering the affected user flow pass green in the pipeline.
4. **Container Build and Publish:** The application container image builds successfully and is automatically pushed to GitHub Container Registry (GHCR).
5. **Documentation Alignment:** The `README.md` and related technical specifications are updated if any requirement, API contract, or architectural decision was changed.

---

## 5. Process and ceremonies

### 5.1 Sprint Length and Cadence

- **Sprint Length:** 2 weeks (10 working days).
- **Number of Sprints:** 4 sprints mapped across the semester project timeline.
- **Justification:** A two-week cadence provides ample time to build and verify a complete vertical slice of functionality from database schema to containerized delivery, while allowing four iterative opportunities to inspect and adapt development before final project defense (avoiding the high process overhead of 1-week cycles and the lack of course correction of 1-month cycles).

### 5.2 Sprint Roadmap Outline

- **Sprint 1 — Walking Skeleton:** Establish repository conventions, database migrations, authentication (`US-01`), branching model, and the automated CI pipeline running linting and unit tests.
- **Sprint 2 — Core Records:** Deliver patient and doctor registers (`US-02`, `US-03`, `US-08`, `US-14`, `US-18`) end-to-end; configure automated container image publishing to GHCR.
- **Sprint 3 — Appointments and Clinical Notes:** Implement appointment booking with double-booking prevention (`US-04`), clinical notes recording (`US-05`, `US-06`, `US-07`), CSV export (`US-12`), and continuous delivery configuration.
- **Sprint 4 — Harden and Observe:** Implement end-to-end testing in CI, configure container health checks and monitoring endpoints (`/health`), perform role authorization regression testing, and finalize documentation for defense.

### 5.3 Scrum Ceremonies

As this is an individual DevOps project, ceremonies represent disciplined role transitions ("changing hats") rather than multi-person meetings:

1. **Sprint Planning:**
   - **Timing:** First day of the sprint (30 to 60 minutes).
   - **Role:** Product Owner hat.
   - **Input:** Prioritised Product Backlog.
   - **Action:** Select top priority stories based on velocity, break them into technical tasks on the GitHub Projects board, and establish a clear Sprint Goal.
   - **Output:** Sprint Goal and committed Sprint Backlog.

2. **Sprint Review:**
   - **Timing:** Last day of the sprint (approximately 30 minutes).
   - **Role:** Product Owner and Demonstrator.
   - **Input:** Running, working software deployed in containers.
   - **Action:** Demonstrate working features live to an external party (classmate or instructor).
   - **Output:** Stakeholder feedback documented as new or re-ordered backlog items.

3. **Sprint Retrospective:**
   - **Timing:** Immediately following the Sprint Review (20 minutes).
   - **Role:** Scrum Master hat.
   - **Input:** Observations on pipeline execution, test stability, and sprint velocity.
   - **Action:** Analyze what went well, what caused friction (e.g., CI failures, estimation issues), and identify workflow adjustments.
   - **Output:** Exactly one concrete, actionable improvement committed to the repository for implementation in the subsequent sprint.

> **Note on Daily Stand-up:**  
> In an individual project, daily stand-up meetings are replaced by a **dated work log** maintained in the repository commit history and GitHub Issue tracking, ensuring total transparency of daily technical progress.

### 5.4 Backlog Refinement Triggers

Backlog refinement is continuous rather than a rigid calendar event. Refinement is explicitly triggered when:
1. **New or Changed Requirement:** A course requirement or stakeholder request introduces new constraints or features.
2. **Story Too Large (Epic):** An item cannot realistically be completed inside a single sprint and must be decomposed into smaller vertical stories before sprint planning.
3. **Missing Acceptance Criteria:** A story lacks concrete, testable verification steps and cannot yet be pulled into a sprint.
4. **Spillover:** A story fails to satisfy the Definition of Done by the end of a sprint; it is re-estimated and reprioritized rather than silently rolled over.
5. **Discovery During Implementation:** A technical limitation, containerization constraint, or hidden dependency is uncovered during development.
6. **Feedback from Sprint Review:** Demonstration feedback shifts stakeholder priorities or reveals usability gaps.
