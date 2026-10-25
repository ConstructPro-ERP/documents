# Database Schema & Normalization

The ConstructPro ERP database is structured using **PostgreSQL** (hosted on **Neon**), a fully managed serverless relational database service. PostgreSQL was selected for its proven ACID compliance, support for Row-Level Security (RLS), and strong compatibility with the Prisma ORM used in the NestJS backend.

---

## 1. Normalization Strategy

The database design strictly adheres to **Third Normal Form (3NF)** to eliminate data redundancy, ensure referential integrity, and simplify maintenance.

### 1NF – First Normal Form

- Every table has a surrogate **UUID primary key** (e.g., `user_id`, `lead_id`, `project_id`).
- All columns contain **atomic, indivisible values** — no lists or nested objects stored within a single field.
- Each row within a table is uniquely identifiable.

### 2NF – Second Normal Form

- All non-key attributes are **fully functionally dependent** on the entire primary key.
- Because every table uses a single-column UUID primary key, there are no composite key partial dependencies by design.
- For example, `project_name` and `start_date` depend entirely on `project_id` — not on any subset of the key.

### 3NF – Third Normal Form

- **No transitive dependencies** exist.
- Reference data (e.g., role names, document categories, payment statuses) is normalized into dedicated lookup tables and referenced via foreign keys.
- For example, `role_name` is not stored directly on the `User` table; instead, a foreign key (`role_id`) references the `Role` table, preventing any update anomaly if a role name changes.

---

## 2. Entity Descriptions and Table Structures

### 2.1 User & Role

| Column          | Type         | Constraints        | Description                             |
| --------------- | ------------ | ------------------ | --------------------------------------- |
| `user_id`       | UUID         | PRIMARY KEY        | Unique identifier for each user         |
| `full_name`     | VARCHAR(150) | NOT NULL           | User's display name                     |
| `email`         | VARCHAR(255) | UNIQUE, NOT NULL   | Login email address                     |
| `password_hash` | TEXT         | NOT NULL           | bcrypt-hashed password (cost factor 12) |
| `role_id`       | UUID         | FK → Role(role_id) | Foreign key to Role table               |
| `is_active`     | BOOLEAN      | DEFAULT true       | Soft-delete / deactivation flag         |
| `created_at`    | TIMESTAMP    | DEFAULT NOW()      | Account creation timestamp              |
| `updated_at`    | TIMESTAMP    | AUTO-UPDATE        | Last modification timestamp             |

| Column        | Type         | Constraints      | Description                                                                 |
| ------------- | ------------ | ---------------- | --------------------------------------------------------------------------- |
| `role_id`     | UUID         | PRIMARY KEY      | Unique role identifier                                                      |
| `role_name`   | VARCHAR(100) | UNIQUE, NOT NULL | e.g., Admin, Sales Manager, Project Manager, Accountant, Client Portal User |
| `description` | TEXT         | NULLABLE         | Human-readable description of role permissions                              |

> **RBAC Note**: Every API request validates the authenticated user's `role_id` against the required permission for the requested route via NestJS Guards and Decorators.

---

### 2.2 Client & Lead

**Lead** records are created when a prospective client is first entered into the system. Once a quotation is approved, the `Lead` can be promoted to a formal `Client`.

| Column            | Type         | Constraints        | Description                                          |
| ----------------- | ------------ | ------------------ | ---------------------------------------------------- |
| `lead_id`         | UUID         | PRIMARY KEY        | Unique lead identifier                               |
| `company_name`    | VARCHAR(200) | NOT NULL           | Prospect company name                                |
| `contact_person`  | VARCHAR(150) | NOT NULL           | Key contact at the prospect company                  |
| `contact_email`   | VARCHAR(255) | NOT NULL           | Contact email                                        |
| `contact_phone`   | VARCHAR(20)  | NULLABLE           | Contact phone number                                 |
| `inquiry_details` | TEXT         | NULLABLE           | Initial project requirements/notes                   |
| `status`          | ENUM         | NOT NULL           | `NEW`, `CONTACTED`, `QUALIFIED`, `CONVERTED`, `LOST` |
| `created_by`      | UUID         | FK → User(user_id) | Sales team member who logged the lead                |
| `created_at`      | TIMESTAMP    | DEFAULT NOW()      | Lead entry date                                      |

| Column            | Type         | Constraints        | Description                                    |
| ----------------- | ------------ | ------------------ | ---------------------------------------------- |
| `client_id`       | UUID         | PRIMARY KEY        | Unique client identifier                       |
| `lead_id`         | UUID         | FK → Lead(lead_id) | Originating lead (nullable for direct clients) |
| `company_name`    | VARCHAR(200) | NOT NULL           | Client company name                            |
| `billing_address` | TEXT         | NOT NULL           | Formal billing address                         |
| `contact_email`   | VARCHAR(255) | UNIQUE, NOT NULL   | Primary contact email                          |
| `contact_phone`   | VARCHAR(20)  | NULLABLE           | Primary phone number                           |
| `created_at`      | TIMESTAMP    | DEFAULT NOW()      | Client record creation date                    |

---

### 2.3 Quotation & Quotation Items

A `Quotation` is a formal cost estimate issued to a `Client` or `Lead`. It is composed of one or more `Quotation_Item` line entries, supporting the normalization of itemized costs.

| Column         | Type          | Constraints            | Description                              |
| -------------- | ------------- | ---------------------- | ---------------------------------------- |
| `quotation_id` | UUID          | PRIMARY KEY            | Unique quotation identifier              |
| `client_id`    | UUID          | FK → Client(client_id) | Client this quotation belongs to         |
| `created_by`   | UUID          | FK → User(user_id)     | Sales Manager who created the quotation  |
| `status`       | ENUM          | NOT NULL               | `DRAFT`, `SENT`, `APPROVED`, `REJECTED`  |
| `total_amount` | DECIMAL(15,2) | NOT NULL               | Total calculated value of all line items |
| `valid_until`  | DATE          | NULLABLE               | Quotation expiry date                    |
| `notes`        | TEXT          | NULLABLE               | Additional terms or comments             |
| `created_at`   | TIMESTAMP     | DEFAULT NOW()          | Creation timestamp                       |
| `updated_at`   | TIMESTAMP     | AUTO-UPDATE            | Last updated timestamp                   |

| Column         | Type          | Constraints                  | Description                                                 |
| -------------- | ------------- | ---------------------------- | ----------------------------------------------------------- |
| `item_id`      | UUID          | PRIMARY KEY                  | Unique line item identifier                                 |
| `quotation_id` | UUID          | FK → Quotation(quotation_id) | Parent quotation                                            |
| `description`  | TEXT          | NOT NULL                     | Line item description (e.g., "Concrete works - Foundation") |
| `quantity`     | DECIMAL(10,2) | NOT NULL                     | Quantity of the item                                        |
| `unit_price`   | DECIMAL(15,2) | NOT NULL                     | Price per unit                                              |
| `line_total`   | DECIMAL(15,2) | GENERATED                    | `quantity * unit_price` (computed on insertion)             |

---

### 2.4 Project, Milestone & Task

An approved `Quotation` is converted into a formal `Project`. The `Project` entity is the central entity in the system — it links to Milestones, Tasks, Documents, Invoices, and Payments.

| Column               | Type          | Constraints                  | Description                                                    |
| -------------------- | ------------- | ---------------------------- | -------------------------------------------------------------- |
| `project_id`         | UUID          | PRIMARY KEY                  | Unique project identifier                                      |
| `quotation_id`       | UUID          | FK → Quotation(quotation_id) | Source approved quotation                                      |
| `client_id`          | UUID          | FK → Client(client_id)       | Associated client                                              |
| `project_manager_id` | UUID          | FK → User(user_id)           | Assigned Project Manager                                       |
| `project_name`       | VARCHAR(200)  | NOT NULL                     | Project title                                                  |
| `description`        | TEXT          | NULLABLE                     | Project scope summary                                          |
| `status`             | ENUM          | NOT NULL                     | `PLANNING`, `IN_PROGRESS`, `ON_HOLD`, `COMPLETED`, `CANCELLED` |
| `start_date`         | DATE          | NOT NULL                     | Project kick-off date                                          |
| `end_date`           | DATE          | NULLABLE                     | Planned completion date                                        |
| `budget`             | DECIMAL(15,2) | NOT NULL                     | Approved project budget                                        |
| `created_at`         | TIMESTAMP     | DEFAULT NOW()                | Record creation date                                           |

| Column                  | Type         | Constraints              | Description                                      |
| ----------------------- | ------------ | ------------------------ | ------------------------------------------------ |
| `milestone_id`          | UUID         | PRIMARY KEY              | Unique milestone identifier                      |
| `project_id`            | UUID         | FK → Project(project_id) | Parent project                                   |
| `title`                 | VARCHAR(200) | NOT NULL                 | Milestone name (e.g., "Foundation Complete")     |
| `due_date`              | DATE         | NOT NULL                 | Milestone target date                            |
| `completion_percentage` | INTEGER      | DEFAULT 0                | Progress indicator (0–100)                       |
| `status`                | ENUM         | NOT NULL                 | `PENDING`, `IN_PROGRESS`, `COMPLETED`, `OVERDUE` |

| Column         | Type         | Constraints                  | Description                   |
| -------------- | ------------ | ---------------------------- | ----------------------------- |
| `task_id`      | UUID         | PRIMARY KEY                  | Unique task identifier        |
| `milestone_id` | UUID         | FK → Milestone(milestone_id) | Parent milestone              |
| `assigned_to`  | UUID         | FK → User(user_id)           | Team member assigned          |
| `title`        | VARCHAR(200) | NOT NULL                     | Task description              |
| `status`       | ENUM         | NOT NULL                     | `TODO`, `IN_PROGRESS`, `DONE` |
| `due_date`     | DATE         | NULLABLE                     | Task deadline                 |

---

### 2.5 Finance – Invoice & Payment

| Column         | Type          | Constraints                  | Description                                    |
| -------------- | ------------- | ---------------------------- | ---------------------------------------------- |
| `invoice_id`   | UUID          | PRIMARY KEY                  | Unique invoice identifier                      |
| `project_id`   | UUID          | FK → Project(project_id)     | Related project                                |
| `milestone_id` | UUID          | FK → Milestone(milestone_id) | Triggering milestone                           |
| `created_by`   | UUID          | FK → User(user_id)           | Accountant who generated invoice               |
| `amount_due`   | DECIMAL(15,2) | NOT NULL                     | Total amount billed                            |
| `amount_paid`  | DECIMAL(15,2) | DEFAULT 0                    | Total amount received against this invoice     |
| `status`       | ENUM          | NOT NULL                     | `PENDING`, `PARTIALLY_PAID`, `PAID`, `OVERDUE` |
| `due_date`     | DATE          | NOT NULL                     | Payment due date                               |
| `issued_date`  | DATE          | DEFAULT NOW()                | Invoice issue date                             |
| `pdf_url`      | TEXT          | NULLABLE                     | Cloudinary URL for generated PDF invoice       |

| Column             | Type          | Constraints              | Description                       |
| ------------------ | ------------- | ------------------------ | --------------------------------- |
| `payment_id`       | UUID          | PRIMARY KEY              | Unique payment record identifier  |
| `invoice_id`       | UUID          | FK → Invoice(invoice_id) | Invoice being paid                |
| `client_id`        | UUID          | FK → Client(client_id)   | Paying client                     |
| `amount`           | DECIMAL(15,2) | NOT NULL                 | Payment amount                    |
| `payment_date`     | DATE          | NOT NULL                 | Date payment was received         |
| `payment_method`   | VARCHAR(50)   | NOT NULL                 | e.g., Bank Transfer, Cheque, Cash |
| `reference_number` | VARCHAR(100)  | NULLABLE                 | Bank reference or cheque number   |
| `recorded_by`      | UUID          | FK → User(user_id)       | User who recorded the payment     |

---

### 2.6 Document Management

| Column         | Type         | Constraints                         | Description                          |
| -------------- | ------------ | ----------------------------------- | ------------------------------------ |
| `document_id`  | UUID         | PRIMARY KEY                         | Unique document identifier           |
| `project_id`   | UUID         | FK → Project(project_id)            | Associated project                   |
| `uploaded_by`  | UUID         | FK → User(user_id)                  | Uploader                             |
| `category_id`  | UUID         | FK → Document_Category(category_id) | Document classification              |
| `file_name`    | VARCHAR(255) | NOT NULL                            | Original filename                    |
| `file_url`     | TEXT         | NOT NULL                            | Cloudinary secure URL                |
| `file_size_kb` | INTEGER      | NOT NULL                            | File size in KB                      |
| `mime_type`    | VARCHAR(100) | NOT NULL                            | e.g., `application/pdf`, `image/png` |
| `uploaded_at`  | TIMESTAMP    | DEFAULT NOW()                       | Upload timestamp                     |

| Column          | Type         | Constraints      | Description                               |
| --------------- | ------------ | ---------------- | ----------------------------------------- |
| `category_id`   | UUID         | PRIMARY KEY      | Category identifier                       |
| `category_name` | VARCHAR(100) | UNIQUE, NOT NULL | e.g., Contract, Drawing, Approval, Report |

---

## 3. Key Relationships Summary

```
Role ───────────────< User
User ───────────────< Lead (created_by)
Lead ───────────────> Client (conversion)
Client ─────────────< Quotation
Quotation ──────────< Quotation_Item
Quotation ──────────> Project (on approval)
Project ────────────< Milestone
Milestone ──────────< Task
Project ────────────< Invoice
Invoice ────────────< Project_Payment
Project ────────────< Document
Document_Category ──< Document
```

---

## 4. Indexing Strategy

To maintain query performance at scale, the following indexes are applied:

| Table       | Indexed Column(s)              | Reason                                 |
| ----------- | ------------------------------ | -------------------------------------- |
| `User`      | `email`                        | Unique lookup for login authentication |
| `Lead`      | `status`, `created_by`         | Dashboard filter queries               |
| `Quotation` | `client_id`, `status`          | Sales module listing                   |
| `Project`   | `project_manager_id`, `status` | PM dashboard queries                   |
| `Invoice`   | `project_id`, `status`         | Finance module outstanding bills       |
| `Document`  | `project_id`, `category_id`    | Document module filtering              |

---

## 5. Row-Level Security (RLS)

PostgreSQL **Row-Level Security policies** are enforced specifically for the **Client Portal User** role. These policies restrict data visibility so that:

- A client can only `SELECT` rows in `Project`, `Quotation`, `Invoice`, and `Document` where their own `client_id` matches.
- All write operations (`INSERT`, `UPDATE`, `DELETE`) from the Client Portal are blocked at the database level, providing defence-in-depth beyond the application-layer guards.
