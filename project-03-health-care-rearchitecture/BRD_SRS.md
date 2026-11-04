
# 🏥 Architecture Restructuring of the Dominican Healthcare System

This repository contains the Product Management and Systems Analysis artifacts for the architecture restructuring of the Dominican Republic's healthcare affiliation system, transforming a monolithic SQL architecture into a scalable, decoupled enterprise integration model.

---

## 📄 1. BRD — Business Requirements Document

The Dominican Healthcare System is restructuring the architecture of the platform that processes its complete affiliates database to improve **scalability, adaptability, reliability, and accountability**.

### 1.1 Context & Business Problem
* **Limited Adaptability:** All system affiliation modules are currently implemented using a fully SQL-based architecture, which reduces system adaptability by approximately 75% and severely limits the implementation of new business rules required under Health Care Law 87-01.
* **Performance Bottlenecks:** Reduced data-processing performance during batch jobs due to increasing data volumes and legacy design practices.
* **Extended Delivery Cycles:** Excessively long Time-to-Market resulting from the platform's limited adaptability and scalability.

---

### 1.2 Business Objectives

| ID | Objective | Target Metric |
| :--- | :--- | :--- |
| **BO-01** | Increase system adaptability via a decoupled architecture | **100% support** for new business rules under Law 87-01 |
| **BO-02** | Improve batch job data-processing performance | **85% performance increase** |
| **BO-03** | Reduce Time-to-Market for new business rules | From an average of 6 months to **< 4 weeks** |
| **BO-04** | Accelerate healthcare affiliation processing time | From several days to **< 24 hours** (submission to approval) |

---

### 1.3 Scope

* **Contributory Regime Affiliation Module:**
  * Holder/Principal member registration.
  * Selection of Health Risk Administrator (ARS).
  * Selection of Pension Fund Administrator (AFP).
  * Dependents registration.

### 1.4 Out of Scope

* Subsidized and Contributory-Subsidized regimes.
* Calculation and collection of Social Security Treasury (TSS) invoices.
* Pension disbursements and payments.
* Modifications to current legal/regulatory frameworks.

---

### 1.5 Stakeholders Matrix

| Stakeholder | Process Role | Main Interest |
| :--- | :--- | :--- |
| **National Social Security Council (CNSS)** | System Governing Body | Ensuring strict regulatory compliance with Law 87-01. |
| **Social Security Treasury (TSS)** | System Owner & Data Custodian | Maintaining a single, reliable, and accurate source of truth for all affiliate records. |
| **Employers** | Service Consumers | Submitting employee registration requests through a fast, transparent, and efficient process. |
| **ARS & AFP Entities** | External Service Providers | Receiving assigned affiliates via timely, accurate, and automated notifications. |

---

### 1.6 Business Rules (BRD)

* **`BR-01` Mandatory Enrollment:** Every salaried worker must be enrolled by their employer in Family Health Insurance, Old-Age, Disability, & Survivors' Insurance, and Occupational Risk Insurance.
* **`BR-02` Identity Verification:** The principal member's identity must be validated using their National ID Card (*Cédula*) or Resident Foreign Document against the official government source (JCE).
* **`BR-03` Employer Eligibility:** The employer must hold an active National Taxpayer Registry (RNC) number with DGII and be officially registered with the Social Security Treasury (TSS).
* **`BR-04` Administrator Selection:** The member selects a single Health Insurance Administrator (ARS) and a single Pension Fund Administrator (AFP). If no selection is made within the regulatory timeframe, assignment is automatically applied according to regulations.
* **`BR-05` Audit Trail & Traceability:** All approved, modified, or rejected affiliation requests must be logged, including the user ID, timestamp (date and time), and action details.

---

### 1.7 Risk Analysis & Mitigation Matrix

| Risk Event | Impact | Mitigation Strategy |
| :--- | :---: | :--- |
| **Risk 1: JCE or DGII API Unavailability** | `Medium` | Implement an asynchronous retry queue with an intermediate "Pending Validation" status. |
| **Risk 2: Historical Data Quality & Integrity** | `High` | Execute pre-migration data cleansing and enforce strict validation rules at the middleware layer. |
| **Risk 3: Employer Resistance to Change** | `Medium` | Launch a pilot program with major enterprise employers alongside interactive online user guides. |
| **Risk 4: System Integrator / Vendor Abandonment** | `Medium` | Enforce strict contractual terms, SLA milestones, and staged economic incentives. |

---

## 🛠️ 2.0 SRS — Software Requirements Specification

### 2.1 Scope & Architecture Overview

This SRS defines the software behavior for the **Contributory Regime Enrollment Module** under the new integration architecture. It serves as the single source of truth for System Design, Middleware Engineering, QA Testing, and User Acceptance Testing (UAT), directly addressing Business Objectives **BO-01** through **BO-05**.

#### **Architectural Blueprint:**
* **API Gateway:** Exposes Contributory Affiliation services via secure REST APIs.
* **Enterprise Service Bus (ESB):** Orchestrates external identity and tax validations (JCE/DGII).
* **Data Access Layer:** The ESB serves as the sole orchestrated component authorized to read from and write to the core SQL database, abstracting direct database access from external endpoints.

## 📋 3.0 Functional & Non-Functional Requirements

This section details the functional specifications to achieve business rules 01 and 02, and non-functional metrics for the **Contributory Regime Affiliation Module**.

---

### 🔐 BR-01: Employer Registration & Status Verification

### 🔹 Business Rule Overview
Every salaried worker must be enrolled by an eligible, authenticated employer. Employers must be able to track the status of registration requests efficiently and securely.

### 📑 Functional Requirements (FR)

#### `FR-01.1` Employee Registration Submission
* **Description:** The system shall allow the employer to submit a new employee registration to the contributory healthcare regime.
* **Actor / User:** User registered as an employer with a valid National Taxpayer Registry (RNC).

#### `FR-01.2` Registration Status Check
* **Description:** The system shall allow the employer to verify the status of an employee registration.
* **Status Lifecycle:**
  * `PE` — **In Progress:** Request submitted, pending external/internal validation.
  * `OK` — **Approved:** Affiliate registered successfully.
  * `RE` — **Rejected:** Registration declined due to validation failure or business rule violation.

---

### ⚡ Non-Functional Requirements (NFR)

| Requirement ID | Category | Metric / Specification |
| :--- | :--- | :--- |
| **`NFR-01.1`** | **Performance** | Employee data registration processing time must be **< 2 seconds**. |
| **`NFR-02.01`** | **Latency / SLA** | The system shall display an employee registration status within **3 seconds for 95%** of user requests. |
| **`NFR-02.02`** | **Scalability & Concurrency** | The system shall support at least **1,000 concurrent users** checking registration statuses without performance degradation. |
| **`NFR-02.03`** | **Security & Access Control** | Only **authenticated employers** shall be allowed to view the registration status of their own employees. |

---

### 🆔 BR-02: Identity Validation Engine

### 🔹 Business Rule Overview
The principal member's identity must be validated against official government sources before affiliation can be processed.

### 📑 Functional Requirements (FR)

#### `FR-02.1` Governmental ID Validation
* **Description:** The system shall validate the principal affiliate ID against the official governmental ID source (JCE).

#### `FR-02.2` Foreign Identification Support
* **Description:** The system shall validate a foreign passport as a valid identification document type for eligible resident affiliates.

---

### ⚡ Non-Functional Requirements (NFR)

| Requirement ID | Category | Metric / Specification |
| :--- | :--- | :--- |
| **`NFR-02.1`** | **Data Integrity / Sync** | The system shall perform identity validation against the **latest actualization of the official governmental ID database**. |
