# Functional Workflows

The ConstructPro ERP system manages the **complete end-to-end lifecycle** of a construction business — from the first prospect contact through project delivery and final payment collection. The six core modules work in concert to provide a seamless operational flow.

---

## 1. System Modules Overview

| Module                        | Code  | Primary Actor(s)           | Core Responsibility                                                       |
| ----------------------------- | ----- | -------------------------- | ------------------------------------------------------------------------- |
| Sales Management Module       | SMM   | Sales Manager              | Lead capture, client onboarding, quotation management                     |
| Project Management Module     | PMM   | Project Manager            | Project lifecycle, milestones, task assignment, calendar sync             |
| Financial Management Module   | FMM   | Accountant                 | Invoice generation, payment recording, outstanding balance tracking       |
| Document Management Module    | DMM   | Project Manager, All Roles | Secure document upload, categorization, and retrieval                     |
| Analytics & Reporting Module  | ARM   | Admin, Management          | Real-time KPI dashboards, financial reports, project completion analytics |
| AI Predictive Analysis Module | AIPAM | Admin, Management          | Risk forecasting, payment delay prediction using historical data          |

---

## 2. Core Workflows

### 2.1 Workflow 1: Lead Capture & Quotation Generation (SMM)

**Actors**: Sales Manager, Client  
**Pre-condition**: Sales Manager is authenticated with the `Sales Manager` role.  
**Trigger**: A prospective client makes contact with the construction company.

#### Step-by-Step Activity Flow

```
[Start]
  │
  ▼
Sales Manager logs into ConstructPro
  │
  ▼
Navigates to Sales → Leads → "New Lead"
  │
  ▼
Fills in Lead Form:
  - Company Name, Contact Person, Contact Email, Phone
  - Inquiry Details (project scope description)
  │
  ▼
System validates form (Zod schema) and saves Lead (Status: NEW)
  │
  ▼
Sales Manager contacts the prospect (off-system)
  │
  ▼
Updates Lead Status to CONTACTED → QUALIFIED
  │
  ▼
Navigates to Quotations → "New Quotation" for this Lead/Client
  │
  ▼
Quotation Builder:
  1. Selects Client (from Lead or existing Client)
  2. Adds Line Items (description, quantity, unit price)
     → System auto-calculates line_total per row
     → Grand Total updates live
  3. Sets validity period and notes
  │
  ▼
Saves as DRAFT → Reviews → Clicks "Send to Client"
  │
  ▼
System generates branded PDF quotation (PDF Generation Microservice)
  → PDF uploaded to Cloudinary
  → Quotation status updated to SENT
  → Email notification sent to client via SendGrid (Notification Service)
  │
  ▼
Client reviews quotation (via email or Client Portal)
  │
  ├─── Client APPROVES ─────────────────────────────────────────────────►
  │                                                                       │
  │                                                            Sales Manager marks Quotation APPROVED
  │                                                                       │
  │                                                            System converts Quotation → Project
  │                                                            Lead Status updated to CONVERTED
  │
  └─── Client REJECTS → Sales Manager revises and re-sends
```

**Post-condition**: An approved `Quotation` exists and a new `Project` record is created, linked to the quotation and client.

#### Functional Requirements Referenced (SRS)

- `FR-SMM-01`: The system shall allow Sales Managers to capture leads with company details and inquiry information.
- `FR-SMM-02`: The system shall allow Sales Managers to build itemized quotations with auto-calculated totals.
- `FR-SMM-03`: The system shall generate a PDF quotation and dispatch it via email upon submission.
- `FR-SMM-04`: Approved quotations shall be automatically converted into Project records.

---

### 2.2 Workflow 2: Project Setup & Milestone Tracking (PMM)

**Actors**: Project Manager, Admin  
**Pre-condition**: A `Project` record exists (converted from an approved quotation).  
**Trigger**: Admin assigns a Project Manager to the newly created project.

#### Step-by-Step Activity Flow

```
[Start: Project Created from Approved Quotation]
  │
  ▼
Admin navigates to Projects → Assigns Project Manager to project
  │
  ▼
Project Manager receives notification (email + in-app)
  │
  ▼
PM opens Project → "Milestones" tab → "Add Milestone"
  - Milestone title (e.g., "Site Preparation Complete")
  - Target due date
  - Description
  │
  ▼
System saves Milestone (Status: PENDING) linked to the Project
  │
  ▼
PM creates Tasks under each Milestone:
  - Task title, description
  - Assigns to team member (User)
  - Sets due date
  │
  ▼
Task appears in assigned user's task board (TODO column)
  │
  ▼
Assigned user moves task: TODO → IN PROGRESS → DONE
  │
  ▼
PM monitors:
  - Milestone completion % updates automatically as tasks complete
  - Calendar view shows milestones on due dates
  │
  ▼
Google Calendar Sync Service pushes milestone deadlines to
  PM's and assigned users' Google Calendars
  │
  ▼
Milestone reaches 100% task completion
  │
  ▼
PM marks Milestone as COMPLETED
  │
  ▼
System triggers notification to Accountant:
  "Milestone [X] complete — ready for invoice generation"
  │
  ▼
[End: Milestone Completed → Handoff to FMM]
```

**Post-condition**: All milestones in the project progress through `PENDING → IN_PROGRESS → COMPLETED`. Project status is updated to `COMPLETED` when all milestones are done.

#### Functional Requirements Referenced (SRS)

- `FR-PMM-01`: The system shall allow Project Managers to define milestones with target dates and descriptions.
- `FR-PMM-02`: The system shall allow task creation, assignment, and status tracking within milestones.
- `FR-PMM-03`: Milestone completion percentage shall be automatically calculated based on completed tasks.
- `FR-PMM-04`: The system shall integrate with Google Calendar to sync milestone due dates.
- `FR-PMM-05`: The system shall notify the Accountant when a milestone is marked complete.

---

### 2.3 Workflow 3: Invoice Generation & Payment Recording (FMM)

**Actors**: Accountant, Client (for payment), Admin  
**Pre-condition**: A milestone has been marked COMPLETED by the Project Manager.  
**Trigger**: Accountant receives notification to generate an invoice for the completed milestone.

#### Step-by-Step Activity Flow

```
[Start: Milestone Completed Notification Received]
  │
  ▼
Accountant navigates to Finance → Invoices → "Generate Invoice"
  │
  ▼
Invoice Form:
  - Select Project and Milestone
  - Enter amount due
  - Set payment due date
  │
  ▼
System creates Invoice record (Status: PENDING)
  │
  ▼
PDF Generation Service creates branded invoice PDF
  → Uploaded to Cloudinary
  → Invoice PDF URL stored in database
  │
  ▼
Notification Service emails invoice PDF to client
  │
  ▼
Client reviews invoice and submits payment (offline — bank transfer, cheque, etc.)
  │
  ▼
Accountant navigates to Invoice → "Record Payment"
  - Enter amount, payment date, payment method, reference number
  │
  ▼
System updates Invoice:
  ├── amount_paid < amount_due → Status: PARTIALLY_PAID
  └── amount_paid = amount_due → Status: PAID
  │
  ▼
System updates outstanding balance calculations for the project
  │
  ▼
Client Portal user sees updated invoice status in their portal
  │
  ▼
[End: Invoice PAID]
```

**Overdue Invoice Handling:**

```
System scheduled job runs daily (cron)
  │
  ▼
Checks all PENDING/PARTIALLY_PAID invoices where due_date < TODAY
  │
  ▼
Updates overdue invoices to Status: OVERDUE
  │
  ▼
Sends overdue notification email to Client
Sends alert to Accountant dashboard
```

#### Functional Requirements Referenced (SRS)

- `FR-FMM-01`: The system shall allow Accountants to generate milestone-linked invoices.
- `FR-FMM-02`: The system shall generate a PDF invoice and email it to the client.
- `FR-FMM-03`: The system shall allow Accountants to record payments against invoices.
- `FR-FMM-04`: Invoice status shall automatically transition (PENDING → PARTIALLY_PAID → PAID → OVERDUE).
- `FR-FMM-05`: The system shall dispatch automated overdue payment reminder emails.

---

### 2.4 Workflow 4: Document Upload & Retrieval (DMM)

**Actors**: Project Manager, All authenticated users  
**Pre-condition**: User is authenticated. A Project exists.

#### Step-by-Step Activity Flow

```
[Start]
  │
  ▼
User navigates to Documents module (or Project → Documents tab)
  │
  ▼
Clicks "Upload Document"
  │
  ▼
Selects or drags file into upload zone
  - Supported types: PDF, DOCX, XLSX, PNG, JPG
  - Max file size: 25 MB
  │
  ▼
Selects Document Category (Contract, Drawing, Approval, Report, Other)
  │
  ▼
Frontend requests a signed Cloudinary upload URL from backend
  (POST /documents/upload-url)
  │
  ▼
Frontend uploads file directly to Cloudinary using signed URL
  (bypasses backend — no file buffering on server)
  │
  ▼
On successful Cloudinary upload:
  Frontend notifies backend with file metadata
  (POST /documents with { project_id, category_id, file_url, file_name, ... })
  │
  ▼
Backend saves Document record to database
  │
  ▼
Document appears in the project's document list immediately
  │
  ▼
[File Retrieval]
  │
  ▼
User clicks document → Backend generates time-limited signed Cloudinary URL
  → User browser opens/downloads the file
  (private Cloudinary resources — no direct public URL access)
```

#### Functional Requirements Referenced (SRS)

- `FR-DMM-01`: The system shall allow authorized users to upload documents up to 25 MB.
- `FR-DMM-02`: Documents shall be categorized by type (Contract, Drawing, Approval, Report).
- `FR-DMM-03`: The system shall use Cloudinary for secure cloud storage of all uploaded files.
- `FR-DMM-04`: Document downloads shall be served via time-limited signed URLs.
- `FR-DMM-05`: Client Portal users shall have read-only access to documents belonging to their own projects.

---

### 2.5 Workflow 5: Analytics & KPI Reporting (ARM)

**Actors**: Admin, Management  
**Pre-condition**: User has Admin role.

#### Data Flow

```
[Database: Aggregation Queries]
  │
  ├── Total Revenue (SUM of paid invoices this month)
  ├── Outstanding Receivables (SUM of pending/partially paid invoices)
  ├── Active Projects Count (Projects WHERE status = IN_PROGRESS)
  ├── Project Completion Rate (completed / total × 100)
  ├── Revenue by Month (GROUP BY month, 12-month rolling)
  └── Payment Collection Rate (paid / billed × 100)
  │
  ▼
NestJS AnalyticsModule aggregates data via Prisma queries
  │
  ▼
Frontend Dashboard renders:
  - KPI Summary Cards
  - Bar Charts (Revenue Trend)
  - Pie Charts (Project Status Distribution)
  - Data Tables (Top Clients, Outstanding Invoices)
  │
  ▼
Charts are built with Recharts / Chart.js
All charts include accessible text descriptions for screen readers
  │
  ▼
User can export reports as PDF (trigger PDF Generation Service)
```

#### Functional Requirements Referenced (SRS)

- `FR-ARM-01`: The system shall display real-time KPI metrics on the Admin dashboard.
- `FR-ARM-02`: The system shall provide revenue trend charts for the last 12 months.
- `FR-ARM-03`: The system shall provide project completion rate and status distribution charts.
- `FR-ARM-04`: The system shall allow export of reports as PDF documents.

---

### 2.6 Workflow 6: AI Predictive Analysis (AIPAM)

**Actors**: Admin, Management  
**Pre-condition**: Sufficient historical project and payment data exists in the database (minimum 10 completed projects for meaningful predictions).  
**Technology**: LangChain RAG (Retrieval-Augmented Generation) + OpenAI Embeddings

#### AI Prediction Flow

```
[Historical Data Collection]
  │
  ├── Past project completion timelines vs. planned timelines
  ├── Payment histories (days-late per invoice per client)
  ├── Milestone completion rates
  └── Seasonal patterns (construction industry delays)
  │
  ▼
Data Preprocessing (NestJS AI Module)
  - Structured records converted to text documents
  - Documents embedded via OpenAI text-embedding-ada-002
  - Embeddings stored in pgvector extension (PostgreSQL)
  │
  ▼
[Inference Request: User asks "Which projects are at risk?"]
  │
  ▼
LangChain RAG Pipeline:
  1. Query is embedded using the same embedding model
  2. Vector similarity search retrieves top-K relevant historical records
  3. Retrieved context + active project data is assembled into a prompt
  4. OpenAI LLM generates a structured prediction response
  │
  ▼
Predictions returned as structured JSON:
  {
    "project_id": "...",
    "project_name": "Tower Block A",
    "risk_type": "DELAY_RISK",
    "confidence_score": 0.87,
    "predicted_delay_days": 14,
    "reasoning": "Similar projects in Q1 historically run 2 weeks over...",
    "recommended_action": "Review subcontractor availability for Phase 3"
  }
  │
  ▼
Predictions displayed on Analytics dashboard:
  - Risk alert panel on main dashboard
  - Detailed predictions table in Analytics module
  - Confidence score displayed as a percentage
```

#### Prediction Types

| Prediction Type         | Description                                                          | Confidence Threshold     |
| ----------------------- | -------------------------------------------------------------------- | ------------------------ |
| **Delay Risk**          | Predicts likelihood of project milestone overrun                     | ≥ 70% flagged as at-risk |
| **Payment Delay Risk**  | Predicts likelihood of a client paying late based on payment history | ≥ 65% flagged            |
| **Budget Overrun Risk** | Predicts likelihood of project costs exceeding approved budget       | ≥ 75% flagged            |

#### Functional Requirements Referenced (SRS)

- `FR-AIPAM-01`: The system shall use LangChain RAG to analyze historical project data and predict delay risks.
- `FR-AIPAM-02`: The system shall predict client payment delay likelihood based on payment history.
- `FR-AIPAM-03`: AI predictions shall be displayed with a confidence score.
- `FR-AIPAM-04`: The system shall provide AI-generated recommended actions alongside risk predictions.

---

## 3. Cross-Workflow Notification Summary

| Trigger Event                  | Recipient(s)         | Channel                     |
| ------------------------------ | -------------------- | --------------------------- |
| New lead captured              | Admin                | In-app notification         |
| Quotation sent to client       | Client               | Email (SendGrid)            |
| Quotation approved by client   | Sales Manager, Admin | In-app + Email              |
| Project created from quotation | Project Manager      | In-app + Email              |
| Milestone completed            | Accountant           | In-app + Email              |
| Invoice generated and sent     | Client               | Email (with PDF attachment) |
| Invoice overdue (daily cron)   | Client, Accountant   | Email                       |
| Task assigned to user          | Assigned user        | In-app notification         |
| Task overdue                   | PM, Assigned user    | In-app + Email              |
| AI risk prediction generated   | Admin, Management    | In-app notification         |

---

## 4. End-to-End Business Lifecycle Summary

```
Lead Captured (SMM)
     │
     ▼
Quotation Drafted & Sent (SMM)
     │
     ▼
Quotation Approved → Project Created (SMM → PMM)
     │
     ▼
Milestones Defined → Tasks Assigned (PMM)
     │
     ▼
Work Executed → Tasks Completed (PMM)
     │
     ▼
Milestone Completed → Invoice Generated (PMM → FMM)
     │
     ▼
Client Pays → Payment Recorded (FMM)
     │
     ▼
All Milestones Done → Project Completed (PMM)
     │
     ▼
Final Report & Documents Archived (DMM)
     │
     ▼
Historical Data Feeds AI Predictions (AIPAM)
```
