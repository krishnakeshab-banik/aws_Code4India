# Software Requirements Specification (SRS)
## Project: DocBOX – AI Health Vault for Bharat

---

## 1. Project Overview

### 1.1 Background
India's healthcare system is monumental in scale but fragmented in data management. A vast majority of the 1.4 billion population relies on physical paper records for prescriptions, diagnostic reports, and medical history. These documents are often lost, damaged, or forgotten, leading to:
- **Redundant testing:** Patients repeat expensive tests because previous reports are unavailable.
- **Dangerous drug interactions:** Doctors lack visibility into a patient's complete medication history.
- **Delayed emergency care:** Critical health data (allergies, blood type, chronic conditions) is inaccessible during accidents or emergencies.

### 1.2 Problem Statement
There is no unified, patient-centric digital platform for the "Bharat" demographic (Tier 2/3 cities and rural areas) that aggregates fragmented medical records into a secure, retrievable, and intelligent timeline. Existing solutions are often siloed within expensive hospital networks or lack vernacular accessibility.

### 1.3 Objectives
- **Centralize Health Records:** Create a secure, cloud-based vault for storing and organizing medical documents for every Indian citizen.
- **Democratize Health Intelligence:** Utilize AI to parse complex medical medical data into simple, actionable insights for the common man.
- **Enable Interoperability:** Facilitate seamless, permission-based data sharing between patients and healthcare providers.
- **Bridge the Rural-Urban Divide:** Ensure the platform is accessible via low-bandwidth connections and regional languages.

### 1.4 Social Impact for Bharat
DocBOX empowers the "Next Billion Users" by giving them ownership of their health data. It reduces out-of-pocket healthcare expenses caused by lost records, improves diagnosis accuracy, and potentially saves lives through instant emergency profile access.

---

## 2. Stakeholders

| Stakeholder | Role & Interest |
| :--- | :--- |
| **Patients** | Primary users who store, manage, and share their health data. |
| **Doctors / Clinics** | View patient history to provide accurate diagnosis and treatment. |
| **Diagnostic Labs** | Upload verified reports directly to patient vaults (future scope). |
| **Emergency Responders** | Access critical "Break-Glass" health data during emergencies. |
| **Government/Regulators** | Ensure data privacy, compliance (DISHA/NDHM alignment), and public health monitoring. |

---

## 3. User Personas

### 3.1 The Rural Patient: "Ramesh"
- **Profile:** 45-year-old farmer in Madhya Pradesh. Limited literacy, uses a budget smartphone.
- **Pain Point:** Travels 50km to the district hospital but often forgets old prescriptions.
- **Goal:** Wants to carry his "medical file" on his phone without needing to type or organize complex folders. Needs voice-based navigation in Hindi.

### 3.2 The Urban Professional: "Priya"
- **Profile:** 29-year-old IT professional in Bengaluru. Tech-savvy, health-conscious.
- **Pain Point:** Manages health records for her aging parents and her own preventive checkups. Finds it tedious to track trends like cholesterol or blood sugar over time from PDF reports.
- **Goal:** A centralized dashboard to visualize family health trends and receive AI-driven alerts.

### 3.3 The Healthcare Provider: "Dr. Mehta"
- **Profile:** General Practitioner at a busy clinic.
- **Pain Point:** Spends 5-10 minutes per patient just gathering past medical history from disorganized physical files.
- **Goal:** A quick, one-click overview of the patient's timeline to make faster, evidence-based decisions.

---

## 4. Functional Requirements

### 4.1 Record Management
- **Smart Upload:** Support for uploading images (JPG/PNG) and documents (PDF) primarily via mobile camera.
- **Auto-Categorization:** System must automatically classify documents (e.g., "Prescription", "Lab Report", "Discharge Summary").

### 4.2 AI Document Processing
- **OCR & Extraction:** Extract text from handwritten and printed prescription/reports.
- **Key Entity Recognition:** Identify and index key medical entities (Medicines, Dosages, Diagnosis, Vital Signs) from unstructured text.

### 4.3 Health Timeline
- **Chronological View:** Display a visual timeline of medical events, visits, and reports.
- **Search & Filter:** Allow retrieval of records by doctor name, date, speciality, or disease (e.g., "Show all skin requests from 2023").

### 4.4 Access & Sharing
- **Doctor Check-in:** Generate temporary QR codes/Links for doctors to view records without permanent access.
- **Time-bound Access:** Access tokens must expire automatically after a set duration (e.g., 24 hours).

### 4.5 Insights Dashboard
- **Vitals Tracking:** Graphing taken from extracted data (e.g., HbA1c trends over 2 years).
- **Risk Alerts:** AI-analyzed flags for potential health risks based on historical data.

### 4.6 Feature: Emergency Profile
- **"Break-Glass" Access:** A universally accessible public URL (via QR on lock screen) showing only critical info: Blood Group, Allergies, Emergency Contacts.

### 4.7 Privacy Mode
- **Sensitive Data Masking:** Capability to hide specific sensitive records (e.g., mental health, sexual health) from general shared views.

---

## 5. Non-Functional Requirements

### 5.1 Security & Compliance
- **Data Encryption:** AES-256 for data at rest (S3/DB) and TLS 1.3 for data in transit.
- **Access Control:** Strict IAM policies; MFA for all sensitive access.
- **Audit Trails:** Immutable logs of every file access and sharing event.

### 5.2 Scalability & Availability
- **Elastic Scale:** Architecture must support auto-scaling to handle millions of simultaneous uploads (burst traffic).
- **High Availability:** 99.9% uptime SLA using multi-AZ deployments.

### 5.3 Performance & Usability
- **Low-Bandwidth Optimization:** App must function on 2G/3G networks; uses lightweight image compression.
- **Response Time:** API latency < 200ms; OCR processing < 5 seconds for standard documents.
- **Localization:** UI and AI extraction support for top Indian languages (Hindi, Tamil, Telugu, etc.).

---

## 6. Data Privacy & Compliance

- **Consent Manager:** Granular user consent is mandatory for any third-party data sharing.
- **Data Residency:** All data resides strictly within AWS India Regions (Mumbai/Hyderabad) to comply with data localization laws.
- **Regulation Adherence:** Designed with principles from HIPAA and India's Digital Personal Data Protection (DPDP) Act.

---

## 7. Success Metrics (KPIs)

- **User Adoption:** No. of verified users with > 5 records uploaded.
- **AI Accuracy:** > 90% accuracy in entity extraction (drug names, values).
- **Engagement:** Daily Active Users (DAU) accessing the "Health Timeline".
- **Provider Trust:** Number of "Doctor Check-ins" (successful record shares).

---

## 8. Assumptions & Constraints

- **Connectivity:** Users will have intermittent internet access; offline-first capability is desirable but secondary for MVP.
- **Smartphone Penetration:** Target users have access to at least an entry-level smartphone.
- **Handwriting Quality:** AI accuracy depends on legibility; extremely poor handwriting may require "human-in-the-loop" or manual tagging fallback.

---

## 9. Future Enhancements

- **Telemedicine Integration:** Direct "Book Appointment" feature with doctors from the timeline.
- **Wearable Integration:** Sync data from smartwatches/bands directly into the health vault.
- **Family Accounts:** Sub-profiles for children and elderly led by a primary guardian.
- **ABHA Integration:** Full integration with Ayushman Bharat Health Account (ABHA) ecosystem.
