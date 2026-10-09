# Hospital Managment - DevOps Project

## 1. Project overview

## 2. Requirement analysis

### 2.1. Funtional requirements

### 2.2. Non-functional requirements

## 3. Roles and access control

The Receptionist manages patient appointments and basic data entry. The Doctor accesses medical records, inputs clinical diagnoses, and prescribes treatments. The Administrator configures system settings, manages user accounts, and oversees security.

| Role | Primary Responsibility | Access Level |
| --- | --- | --- |
| Receptionist | Appointment scheduling and patient registration | Low (Front desk operations) |
| Doctor | Medical record management and clinical care | Medium (Patient health data) |
| Administrator | System configuration and user access control | High (Full system access) |

## 4. Product backlog

### 4.1. Receptionist

- **US-01:** As a receptionist, I want to register a new patient with their name, date of birth and contact details, so that the patient can be booked for an appointment.
- **US-02:** As a receptionist, I want to search for registered patients by name or national identification number, so that I can quickly retrieve their profile and avoid duplicate records.
- **US-03:** As a receptionist, I want to schedule an appointment for a patient with a specific doctor and time slot, so that patient visits are properly organized without scheduling conflicts.
- **US-04:** As a receptionist, I want to cancel or reschedule an existing appointment, so that the calendar reflects real-time changes when patients modify their visits.
- **US-05:** As a receptionist, I want to mark a patient as checked-in upon arrival, so that the attending doctor is notified and the waiting room queue is updated.

### 4.2. Doctor

- **US-06:** As a doctor, I want to view my daily consultation schedule and patient queue, so that I can organize my working day and attend to patients in a timely manner.
- **US-07:** As a doctor, I want to view and update a patient's medical records and clinical diagnoses, so that I can prescribe the appropriate treatment and ensure continuous clinical care.
- **US-08:** As a doctor, I want to record and view patient allergies and chronic conditions, so that I can avoid prescribing contraindicated medications or treatments.
- **US-09:** As a doctor, I want to issue electronic prescriptions specifying dosage and duration, so that patients receive clear medication instructions and pharmacies can dispense them accurately.
- **US-10:** As a doctor, I want to order laboratory and diagnostic tests for a patient, so that I can obtain necessary clinical data to confirm a diagnosis.

### 4.3. Administrator

- **US-11:** As an administrator, I want to create and manage user accounts with role-based access control, so that system security is maintained and sensitive patient data is protected.
- **US-12:** As an administrator, I want to deactivate user accounts and immediately revoke access privileges, so that ex-employees or unauthorized individuals cannot access medical systems.
- **US-13:** As an administrator, I want to configure hospital departments, consultation rooms, and operating hours, so that scheduling aligns with the hospital's operational capacity.
- **US-14:** As an administrator, I want to inspect system audit logs and data access records, so that I can ensure compliance with healthcare privacy regulations and detect unauthorized access.
- **US-15:** As an administrator, I want to configure automated database backups and recovery routines, so that critical hospital and patient records can be restored in case of a system failure.

## 5. Process and ceremonies
