# System Design Document
## Project: DocBOX – AI Health Vault for Bharat

---

## 1. High-Level Architecture Overview

DocBOX utilizes a **Serverless Microservices Architecture** on AWS to ensure scalability, cost-effectiveness, and low operational overhead. The system is designed to be **Event-Driven**, decoupling the user-facing application from the heavy AI processing pipelines.

- **Frontend:** React Native / Flutter (Mobile) + React.js (Web Portal) hosted on AWS Amplify.
- **API Layer:** Amazon API Gateway routing requests to AWS Lambda functions.
- **Core Logic:** Stateless Lambda functions for business logic.
- **Data Layer:** Amazon DynamoDB for metadata and Amazon S3 for document storage.
- **AI Layer:** Asynchronous processing pipeline using Textract and Comprehend Medical.

---

## 2. Component Architecture

The system is divided into three core planes:

1.  **Orchestration Plane:** Handles authentication, API routing, and request validation.
2.  **Processing Plane:** The "Brain" of DocBOX; manages file uploads, OCR queues, and AI extraction jobs.
3.  **Data Plane:** Securely stores encrypted raw files and structured NoSQL data.

---

## 3. AI Pipeline (The "Intelligence" Engine)

The AI pipeline converts unstructured images/PDFs into structured health data.

1.  **Ingestion:** User uploads a file. Image is pre-processed (deskewing, contrast enhancement) via Lambda.
2.  **OCR/Extraction (Amazon Textract):**
    -   Extracts raw text, forms, and tables from medical reports.
    -   Handles multi-column layouts typical in lab reports.
3.  **Medical Analysis (Amazon Comprehend Medical):**
    -   Parses the text to identify `PHI` (Protected Health Information).
    -   Extracts entities: `Medication`, `Dosage`, `Diagnosis`, `Test_Name`, `Test_Value`.
    -   Links attributes (e.g., linking "500mg" to "Paracetamol").
4.  **Insight Generation:**
    -   Python-based analysis (SageMaker/Lambda) compares extracted values against standard reference ranges.
    -   Generates "Out of Range" alerts and populates the trend database.

---

## 4. AWS Cloud Architecture (Detailed)

This architecture prioritizes security, scalability, and "pay-as-you-go" efficiency.

| Service | Usage & Justification |
| :--- | :--- |
| **Amazon S3** | **Secure Storage:** Stores raw medical documents. Uses **S3 Intelligent-Tiering** for cost savings on older records. Lifecycle policies archive data to Glacier Deep Archive after 3 years. |
| **AWS Lambda** | **Compute:** Runs application logic (Node.js/Python). Scales automatically to zero when unused, perfect for handling spiky traffic patterns in healthcare. |
| **Amazon API Gateway** | **Interface:** Manages RESTful API endpoints, robust throttling to prevent DDoS, and validates API keys. |
| **Amazon Cognito** | **Identity:** Manages user sign-up/sign-in, MFA, and federated identity. Handles secure implementation of patient vs. doctor roles. |
| **Amazon DynamoDB** | **Database:** Fast, serverless NoSQL database. Stores user profiles, record metadata, and the extracted health timeline. Schema-less design adapts to varying report formats. |
| **Amazon Textract** | **OCR:** Specifically optimized for document extraction, handling tables in lab reports better than generic OCR. |
| **Amazon Comprehend Medical** | **NLP:** The core differentiator. Out-of-the-box ability to understand complicated medical terminology without training custom models from scratch. |
| **Amazon CloudWatch** | **Observability:** Logs all API calls and Lambda executions. Alarms set for error rates or high latency. |
| **AWS KMS** | **Security:** Keys Management Service to manage encryption keys (CSE-KMS) for encrypting data in S3 and DynamoDB. |
| **Amazon SNS / SES** | **Notifications:** Sends SMS (OTP) and Email (Reports/Alerts) to users. |

---

## 5. Data Flow (Step-by-Step)

### Scenario: User Uploads a Blood Test Report

1.  **Upload:** User App requests a **Presigned URL** from API Gateway + Lambda.
2.  **Transfer:** App directly uploads the encrypted image to an **S3 Bucket** (Incoming).
3.  **Trigger:** S3 Event Notification triggers `ProcessDocument-Lambda`.
4.  **Async AI:**
    -   Lambda calls **Textract** `StartDocumentAnalysis`.
    -   Textract processes file and publishes completion status to an **SNS Topic**.
5.  **Analysis:**
    -   `Analyze-Lambda` consumes the SNS message, fetches OCR results.
    -   Passes text to **Comprehend Medical**.
    -   Structures the JSON output (Heart Rate: 80, BP: 120/80).
6.  **Storage:** Structured data is written to **DynamoDB** (UserTimeline Table).
7.  **Notification:** **WebSocket API** (or Push Notification) informs the user: "Your report has been analyzed."

---

## 6. Database Design Overview

**Primary Database:** Amazon DynamoDB (Single Table Design Strategy)

-   **Partition Key (PK):** `USER#<MobileNumber>`
-   **Sort Key (SK):**
    -   `PROFILE` (User details)
    -   `RECORD#<Date>#<DocID>` (Medical record metadata)
    -   `METRIC#<Type>#<Date>` (e.g., `METRIC#GLUCOSE#2023-10-27`)

**Access Patterns:**
-   *Get Profile:* PK=`USER#123` & SK=`PROFILE`
-   *Get Timeline:* Query PK=`USER#123` & starts_with(SK, `RECORD#`)
-   *Get Glucose Trend:* Query PK=`USER#123` & starts_with(SK, `METRIC#GLUCOSE`)

---

## 7. API Design Principles

-   **RESTful Architecture:** Standard HTTP verbs (GET, POST, PUT, DELETE).
-   **Versioning:** All endpoints prefixed with `/v1/` to allow non-breaking future updates.
-   **Statelessness:** No session storage on server; relies on JWT tokens (Cognito).
-   **Rate Limiting:** Per-user throttling to prevent abuse (e.g., max 10 uploads/minute).

---

## 8. Security Architecture

-   **Identity & Access Management (IAM):** Least privilege principle applied to all Lambda roles.
-   **Encryption:**
    -   **At Rest:** S3 buckets and DynamoDB tables encrypted using AWS KMS Customer Managed Keys.
    -   **In Transit:** SSL/TLS 1.2+ mandatory for all API communication.
-   **Temporary Access (Doctor Sharing):**
    -   Generates a unique, cryptic UUID link.
    -   Lambda validates the UUID and checks expiration (TTL).
    -   Content is served via signed CloudFront URLs, valid only for the user's session.

---

## 9. Scalability Strategy

-   **Serverless First:** Reliance on Lambda and DynamoDB means the infrastructure scales automatically with request volume without manual intervention.
-   **Async Processing:** Decoupling upload (synchronous) from AI processing (asynchronous) ensures the UI remains snappy even if backend processing takes time.
-   **Content Delivery:** Use **Amazon CloudFront** to cache static assets (web portal) and serve document previews with low latency across India.

---

## 10. Deployment Architecture

-   **CI/CD Pipeline:** AWS CodePipeline + AWS CodeBuild.
-   **Infrastructure as Code (IaC):** AWS CloudFormation or CDK (Cloud Development Kit) to define the entire stack. This ensures identical environments for Dev, Staging, and Prod.
-   **Blue/Green Deployment:** AWS Lambda traffic shifting to ensure zero downtime during updates.

---

## 11. Monitoring & Logging

-   **Distributed Tracing:** **AWS X-Ray** integrated into Lambdas to trace requests from API Gateway through to DynamoDB, identifying bottlenecks.
-   **Dashboards:** specific CloudWatch Dashboards for business metrics (e.g., "Total Documents Processed", "AI Confidence Score Avg").
-   **Alerting:** CloudWatch Alarms for "Lambda Error Rate > 1%" or "Textract Throttling".

---

## 12. Cost Optimization Strategy

To impress AWS Judges:
1.  **S3 Intelligent-Tiering:** Automatically moves infrequently accessed health records to cheaper storage classes.
2.  **Lambda Power Tuning:** Optimize memory allocation to find the sweet spot between cost and speed.
3.  **Compute Savings Plans:** Committed use for Lambda if steady-state traffic grows.
4.  **Free Tier Usage:** Leverage the generous AWS Free Tier for Cognito, Lambda requests, and DynamoDB throughput during the hackathon/pilot phase.
5.  **Direct Uploads:** Uploading directly to S3 (vs. passing through Lambda) saves on Lambda execution time/cost for large files.

---

## 13. Future AI Enhancements

-   **Predictive Analytics:** Train custom **SageMaker** models on the aggregated (anonymized) Indian dataset to predict regional disease outbreaks (e.g., Dengue hotspots).
-   **Voice-to-Text:** Use **Amazon Transcribe Medical** to allow doctors to dictate notes directly into the patient's record.
-   **Chatbot Assistant:** Implement **Amazon Bedrock/Claude** to allow patients to ask natural language questions ("What was my last cholesterol level?") and get answers from their own medical history.
