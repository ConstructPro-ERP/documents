# System Architecture

ConstructPro ERP is designed on a **hybrid layered architecture** incorporating a centralized monolithic backend, a decoupled React-based frontend, and dedicated microservices for asynchronous operations. This architectural approach was selected to balance team development velocity with operational scalability as the user base grows.

---

## 1. Architectural Philosophy & Justification

The architecture was designed around the following core principles:

| Principle                  | Architectural Decision                                                                                            |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Separation of Concerns** | Frontend (Next.js), Backend (NestJS), and Data (PostgreSQL/Cloudinary) are independently deployable units.        |
| **Scalability**            | Vercel Serverless Functions automatically scale the backend horizontally without manual provisioning.             |
| **Security**               | All layers enforce security independently (API Gateway rate-limiting, NestJS Guards, PostgreSQL RLS).             |
| **Maintainability**        | Module-based NestJS structure mirrors the business domain (Sales, Projects, Finance, Documents).                  |
| **Cost Efficiency**        | Serverless hosting (Vercel + Neon) eliminates idle server costs — the system pays only for compute actually used. |

---

## 2. High-Level Architecture – Layer Breakdown

The system is organized into **seven logical layers**:

```
┌──────────────────────────────────────────────────────────────┐
│                        USER LAYER                            │
│  Admin | Sales Manager | Project Manager | Accountant |      │
│  Client Portal User                                          │
└──────────────────────────┬───────────────────────────────────┘
                           │ HTTPS / TLS 1.3
┌──────────────────────────▼───────────────────────────────────┐
│                    API GATEWAY LAYER                         │
│  Rate Limiting | SSL Termination | CORS | Load Balancing     │
│  (Vercel Edge Network)                                       │
└──────────────────────────┬───────────────────────────────────┘
                           │
┌──────────────────────────▼───────────────────────────────────┐
│               PRESENTATION LAYER (FRONTEND)                  │
│  Next.js 14 (App Router) | React | TailwindCSS               │
│  Hosted on Vercel Global CDN                                 │
└──────────────────────────┬───────────────────────────────────┘
                           │ REST API Calls (JSON)
┌──────────────────────────▼───────────────────────────────────┐
│            BACKEND SERVICE LAYER (NestJS API)                │
│  Authentication | Business Logic | DTO Validation (Zod)      │
│  RBAC Guards | Prisma ORM | API Controllers                  │
│  Hosted on Vercel Serverless Functions                       │
└──────────┬───────────────────────────┬───────────────────────┘
           │                           │
┌──────────▼──────────┐   ┌────────────▼──────────────────────┐
│  MICROSERVICES LAYER│   │          DATA LAYER               │
│  Notification Svc   │   │  PostgreSQL on Neon               │
│  PDF Generation Svc │   │  (Primary Database)               │
│  Calendar Sync Svc  │   │                                   │
│  AI Forecast Engine │   │  Cloudinary                       │
│  (LangChain RAG)    │   │  (File & Media Storage)           │
└──────────┬──────────┘   └───────────────────────────────────┘
           │
┌──────────▼──────────────────────────┐
│         EXTERNAL SERVICES           │
│  Google Calendar API                │
│  SendGrid / SMTP (Email)            │
└─────────────────────────────────────┘
```

---

## 3. Layer-by-Layer Description

### 3.1 User Layer

Five distinct user personas interact with the system, each with a unique access scope enforced by RBAC:

| Persona                | Primary Responsibilities                                                          | Access Scope                |
| ---------------------- | --------------------------------------------------------------------------------- | --------------------------- |
| **Administrator**      | User management, role assignment, system configuration                            | Full system access          |
| **Sales Manager**      | Lead capture, quotation drafting and sending                                      | Sales Management Module     |
| **Project Manager**    | Milestone definition, task tracking, progress updates, document uploads           | Project Management Module   |
| **Accountant**         | Invoice generation, payment recording, financial reports                          | Financial Management Module |
| **Client Portal User** | View-only access to their own project status, quotations, invoices, and documents | Read-only, own records only |

---

### 3.2 API Gateway Layer

Provided by **Vercel's Edge Network**, which functions as the system's API gateway. It handles:

- **SSL/TLS Termination**: All traffic is HTTPS-enforced. HTTP connections are permanently redirected to HTTPS (301).
- **Rate Limiting**: Configurable limits per route to prevent brute-force attacks and API abuse.
- **CORS Configuration**: Strict origin whitelist — only the registered frontend domain is permitted.
- **CDN & Caching**: Static assets (JS bundles, CSS, images) are served from Vercel's global edge nodes for low-latency delivery worldwide.

---

### 3.3 Presentation Layer – Next.js Frontend

| Aspect                | Technology / Approach                                                     |
| --------------------- | ------------------------------------------------------------------------- |
| **Framework**         | Next.js 14 with App Router                                                |
| **Language**          | TypeScript (strict mode)                                                  |
| **State Management**  | React Query (TanStack Query) for server state, Zustand for local UI state |
| **API Communication** | Axios with interceptors for token refresh and error handling              |
| **Form Handling**     | React Hook Form + Zod for client-side validation                          |
| **Styling**           | Tailwind CSS with a custom design token system                            |
| **Code Standards**    | Airbnb ESLint Style Guide + Prettier for automated formatting             |
| **Hosting**           | Vercel with automatic previews on pull requests                           |

The frontend is organized into **feature modules** that mirror the business domain:

- `/dashboard` — Admin KPI overview
- `/sales` — Lead management, quotation builder
- `/projects` — Project listing, milestone tracker, task board
- `/finance` — Invoice listing, payment entry
- `/documents` — Document upload, categorized file browser
- `/analytics` — Charts and AI forecast reports
- `/client-portal` — Restricted view for Client Portal users

---

### 3.4 Backend Service Layer – NestJS API

The NestJS application follows a **modular monolith** pattern, where each business domain is encapsulated in its own NestJS module with controllers, services, DTOs, and entities:

| Module                | Key Responsibilities                                                            |
| --------------------- | ------------------------------------------------------------------------------- |
| `AuthModule`          | JWT issuance, refresh token rotation, bcrypt password hashing, logout blocklist |
| `UsersModule`         | User CRUD, role assignment                                                      |
| `SalesModule`         | Lead CRUD, lead status transitions, quotation management                        |
| `ProjectsModule`      | Project lifecycle, milestone and task management, calendar sync                 |
| `FinanceModule`       | Invoice generation, payment recording, outstanding balance calculations         |
| `DocumentsModule`     | Multipart file upload to Cloudinary, document metadata persistence              |
| `AnalyticsModule`     | KPI aggregation queries (revenue, project completion rates, payment delays)     |
| `AiModule`            | LangChain RAG pipeline orchestration for risk and delay predictions             |
| `NotificationsModule` | Email dispatch via SendGrid, in-app notification queuing                        |

**Request Pipeline:**

```
HTTP Request → NestJS Controller → JWT Guard → RBAC Guard → DTO Validation (Zod Pipe) → Service → Prisma ORM → PostgreSQL
```

---

### 3.5 Microservices Layer

Independent background services handle workloads that are asynchronous or computationally intensive:

| Service                    | Technology                          | Function                                                                                                  |
| -------------------------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Notification Service**   | SendGrid API / SMTP                 | Sends email alerts for quotation approvals, invoice due dates, milestone completions                      |
| **PDF Generation Service** | Puppeteer / React-PDF               | Generates and uploads branded invoice and quotation PDFs to Cloudinary                                    |
| **Calendar Sync Service**  | Google Calendar API                 | Synchronizes project milestones and task deadlines to assigned users' Google Calendars                    |
| **AI Forecasting Engine**  | LangChain + RAG + OpenAI Embeddings | Ingests historical project data, runs semantic retrieval, and produces risk and payment delay predictions |

---

### 3.6 Data Layer

| Component               | Technology         | Role                                                              |
| ----------------------- | ------------------ | ----------------------------------------------------------------- |
| **Relational Database** | PostgreSQL on Neon | Primary system of record for all transactional data               |
| **ORM**                 | Prisma ORM         | Type-safe database access, schema migrations, query building      |
| **File Storage**        | Cloudinary         | Stores uploaded documents, generated PDF invoices, and quotations |

**Neon PostgreSQL** provides:

- Serverless autoscaling (scales to zero during off-hours, scales up on demand)
- Point-in-time recovery (PITR) for disaster recovery
- Branching for staging/development environments
- Row-Level Security (RLS) at the database engine level

---

### 3.7 External Services

| Service                  | Provider                   | Purpose                                                                        |
| ------------------------ | -------------------------- | ------------------------------------------------------------------------------ |
| **Email Delivery**       | SendGrid                   | Transactional email for notifications, quotation submissions, invoice delivery |
| **Calendar Integration** | Google Calendar API        | Project timeline integration for all stakeholders                              |
| **AI Language Model**    | OpenAI API (via LangChain) | Embedding generation and LLM inference for the AI Predictive Analysis Module   |

---

## 4. Key Technology Stack Summary

| Layer              | Technology         | Version |
| ------------------ | ------------------ | ------- |
| Frontend Framework | Next.js (React)    | 14.x    |
| Frontend Language  | TypeScript         | 5.x     |
| Backend Framework  | NestJS (Node.js)   | 10.x    |
| Backend Language   | TypeScript         | 5.x     |
| ORM                | Prisma             | 5.x     |
| Database           | PostgreSQL (Neon)  | 16.x    |
| File Storage       | Cloudinary         | SDK v2  |
| AI/ML              | LangChain + OpenAI | Latest  |
| Authentication     | JWT + bcrypt       | —       |
| Hosting            | Vercel             | —       |

---

## 5. Architecture Decision Records (ADRs)

### ADR-01: Monolith Backend over Full Microservices

**Decision**: NestJS modular monolith instead of fully distributed microservices.  
**Rationale**: The team size (4–6 developers) and project timeline make a full microservices architecture operationally prohibitive. The NestJS module system provides logical separation without the overhead of inter-service networking, distributed tracing, and message broker management. The architecture can be extracted into true microservices in a future phase.

### ADR-02: Serverless Hosting (Vercel + Neon)

**Decision**: Vercel for frontend/backend, Neon for PostgreSQL — all serverless.  
**Rationale**: Eliminates server provisioning and maintenance. Auto-scaling handles traffic spikes. Pay-per-use cost model is aligned with an early-stage system. Neon's branching feature supports parallel development environments.

### ADR-03: PostgreSQL over NoSQL

**Decision**: PostgreSQL relational database over MongoDB or similar.  
**Rationale**: The system's core domain (Projects, Finance, Quotations) is highly relational. PostgreSQL's foreign key enforcement, transaction support, and Row-Level Security directly satisfy the system's data integrity and multi-tenancy requirements.
