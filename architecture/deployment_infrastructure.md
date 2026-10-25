# Deployment & Infrastructure

ConstructPro ERP utilizes a **fully cloud-native, serverless architecture** optimized for reliability, global performance, horizontal scalability, and minimal operational overhead. All infrastructure components are managed services — there are no self-managed servers.

---

## 1. Infrastructure Overview

| Component            | Technology                               | Provider        |
| -------------------- | ---------------------------------------- | --------------- |
| Frontend Hosting     | Vercel (Global CDN + Edge Network)       | Vercel Inc.     |
| Backend Hosting      | Vercel Serverless Functions              | Vercel Inc.     |
| Relational Database  | PostgreSQL (Serverless)                  | Neon            |
| File & Media Storage | Cloudinary (CDN-backed)                  | Cloudinary      |
| Domain & DNS         | Custom domain with Vercel DNS management | Vercel Inc.     |
| CI/CD Pipeline       | GitHub Actions + Vercel Auto-Deployments | GitHub / Vercel |

---

## 2. Deployment Environments

### 2.1 Local Development Environment

The local environment is used by individual developers for feature development and unit testing.

| Setting               | Value                                                |
| --------------------- | ---------------------------------------------------- |
| **Frontend URL**      | `http://localhost:3000`                              |
| **Backend API URL**   | `http://localhost:5000`                              |
| **Database**          | Neon development branch (`dev/<developer-name>`)     |
| **File Storage**      | Cloudinary development folder (`/constructpro/dev/`) |
| **Secret Management** | `.env.local` files (git-ignored, never committed)    |
| **Package Manager**   | `npm` with `package-lock.json` committed             |

**Local Setup Steps:**

1. Clone the repository and install dependencies: `npm install`
2. Copy `.env.example` to `.env.local` and populate with development credentials.
3. Run database migrations: `npx prisma migrate dev`
4. Seed initial data: `npx prisma db seed`
5. Start development servers: `npm run dev` (Next.js) and `npm run start:dev` (NestJS)

---

### 2.2 Staging Environment

The staging environment mirrors production and is used for QA testing, integration validation, and client acceptance testing.

| Setting               | Value                                                                                |
| --------------------- | ------------------------------------------------------------------------------------ |
| **Trigger**           | Every push to `develop` or `feature/*` branch triggers a Vercel Preview Deployment   |
| **Frontend URL**      | Vercel auto-generated preview URL (e.g., `constructpro-git-develop-team.vercel.app`) |
| **Backend API URL**   | Vercel preview function deployment linked to staging                                 |
| **Database**          | Separate Neon PostgreSQL staging branch (`staging`) — isolated from production data  |
| **File Storage**      | Cloudinary staging folder (`/constructpro/staging/`)                                 |
| **Secret Management** | Vercel Environment Variables (staging scope)                                         |

**Staging Workflow:**

- Pull Requests automatically receive a **Vercel Preview URL** for QA review.
- Integration tests and end-to-end tests (Cypress/Playwright) are executed against the staging environment.
- Client stakeholders validate features on staging before merging to `main`.

---

### 2.3 Production Environment

The production environment serves live users with maximum reliability and security.

| Setting               | Value                                                             |
| --------------------- | ----------------------------------------------------------------- |
| **Trigger**           | Merge to `main` branch auto-deploys to production via Vercel      |
| **Frontend URL**      | `https://constructpro.vercel.app` (custom domain configured)      |
| **Backend API URL**   | Vercel Production Serverless Functions                            |
| **Database**          | Neon PostgreSQL production branch — `main`                        |
| **File Storage**      | Cloudinary production environment with production resource limits |
| **Secret Management** | Vercel Environment Variables (production scope)                   |
| **Uptime SLA**        | ≥ 99% during business operational hours (NFR-03)                  |

---

## 3. Infrastructure Component Details

### 3.1 Frontend Hosting – Vercel

Vercel is the hosting platform for the Next.js frontend application and acts as the API gateway for all backend serverless functions.

**Key Capabilities:**

- **Global Edge Network**: Static assets are distributed across Vercel's 100+ global edge locations, ensuring low-latency delivery regardless of user geography.
- **SSL/TLS Termination**: Automatic TLS certificate provisioning and renewal. All traffic is HTTPS-only; HTTP requests receive a permanent 301 redirect.
- **Automatic Preview Deployments**: Every pull request generates an isolated preview deployment with its own URL — enabling per-PR code review and QA.
- **Incremental Static Regeneration (ISR)**: Allows static pages to be regenerated on demand without a full rebuild, improving dashboard load performance.
- **Zero-Downtime Deployments**: Vercel uses an atomic deployment model — new deployments are only made live after health checks pass, ensuring no downtime during updates.

---

### 3.2 Backend Hosting – Vercel Serverless Functions

The NestJS backend is deployed as **Vercel Serverless Functions** (Node.js runtime).

**Key Capabilities:**

- **Automatic Horizontal Scaling**: Functions scale independently per endpoint. High-traffic routes (e.g., Dashboard KPIs) scale without affecting low-traffic routes.
- **Cold Start Mitigation**: Vercel's Edge Runtime and function keep-warm strategies minimize cold start latency for critical API endpoints.
- **Function Timeout Configuration**: Standard API functions are configured with a 10-second timeout. Long-running tasks (PDF generation, AI inference) are offloaded to background queue workers.

**Serverless Limitation Management:**

- Background jobs (PDF generation, calendar sync, AI forecasting) are decoupled from the request-response cycle via an async task queue, preventing timeout issues.
- Large file uploads are streamed directly from the client to Cloudinary using signed upload URLs — the backend never buffers large file payloads.

---

### 3.3 Database – Neon PostgreSQL

**Neon** provides a serverless, fully managed PostgreSQL database with the following characteristics:

| Feature                           | Detail                                                                                                                |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Serverless Scaling**            | Automatically scales compute up/down based on query load. Scales to zero during inactivity (cost optimization).       |
| **Branching**                     | Database branches for `dev`, `staging`, and `production` environments — similar to Git branches.                      |
| **Point-in-Time Recovery (PITR)** | Supports recovery to any point in the last 7 days — protecting against accidental data loss.                          |
| **High Availability**             | Multi-zone replication ensures zero data loss with automatic failover.                                                |
| **Connection Pooling**            | Neon Proxy (PgBouncer-compatible) handles connection pooling, preventing connection exhaustion under serverless load. |
| **Encryption at Rest**            | AES-256 volume-level encryption provided by Neon.                                                                     |
| **Compliance**                    | SOC 2 Type II certified infrastructure.                                                                               |

**Connection Details:**

- Prisma connects to Neon via the `DATABASE_URL` environment variable using a pooled connection string.
- Migrations are managed by Prisma Migrate, executed as part of the CI/CD pipeline during deployments.

---

### 3.4 File & Media Storage – Cloudinary

**Cloudinary** serves as the system's cloud file storage and delivery platform for all document uploads, generated PDF invoices, and quotation PDFs.

| Feature                          | Detail                                                                                                                                               |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Storage**                      | Organized folder structure: `/constructpro/{env}/{project_id}/{category}/`                                                                           |
| **Server-Side Encryption (SSE)** | All uploaded files are encrypted at rest using Cloudinary SSE.                                                                                       |
| **CDN Delivery**                 | Files are served via Cloudinary's global CDN for fast, low-latency downloads.                                                                        |
| **Signed Upload URLs**           | Backend generates short-lived signed upload URLs — the client uploads directly to Cloudinary, bypassing the backend for large file transfers.        |
| **Transformations**              | On-the-fly image resizing and format conversion (e.g., thumbnails for document preview icons).                                                       |
| **Access Control**               | Uploaded documents are set to `private` delivery type — files are only accessible via signed, time-limited Cloudinary URLs generated by the backend. |

---

## 4. CI/CD Pipeline

```
Developer Push
     │
     ▼
GitHub Repository
     │
     ├─── PR to develop ──► GitHub Actions (Lint + Unit Tests) ──► Vercel Preview Deployment
     │
     └─── Merge to main ──► GitHub Actions (Lint + Tests + Migrations) ──► Vercel Production Deployment
```

**Pipeline Steps:**

1. **Lint & Format Check**: ESLint + Prettier validation fails the build on violations.
2. **Type Check**: TypeScript strict mode compilation.
3. **Unit Tests**: Jest test suite execution (minimum 80% coverage enforced).
4. **Database Migration Check**: Verifies Prisma migration files are consistent with the schema.
5. **Build**: `next build` and NestJS compilation.
6. **Deploy**: Vercel CLI performs the deployment.
7. **Post-Deploy Health Check**: Vercel runs health check on `/api/health` endpoint.

---

## 5. Hardware & Network Requirements

### Client (Browser) Requirements

| Requirement            | Specification                                    |
| ---------------------- | ------------------------------------------------ |
| **Browser**            | Chrome 100+, Edge 100+, Firefox 100+, Safari 15+ |
| **RAM**                | Minimum 8 GB recommended                         |
| **Display Resolution** | Minimum 1366 × 768 (WXGA)                        |
| **Network**            | Minimum 5 Mbps broadband connection              |
| **JavaScript**         | Must be enabled                                  |

### Server Requirements

All server-side infrastructure is **fully abstracted** by Vercel and Neon managed services. The project team is not responsible for server provisioning, OS patching, hardware maintenance, or capacity planning. The cloud providers guarantee:

| Guarantee                          | SLA                                                   |
| ---------------------------------- | ----------------------------------------------------- |
| **Vercel Uptime**                  | 99.99% (per Vercel Enterprise SLA)                    |
| **Neon Database Availability**     | 99.95%                                                |
| **Cloudinary CDN Availability**    | 99.99%                                                |
| **Operational Hours SLA (NFR-03)** | ≥ 99% uptime during 6:00 AM – 10:00 PM business hours |

---

## 6. Monitoring & Observability

| Tool                     | Purpose                                                                               |
| ------------------------ | ------------------------------------------------------------------------------------- |
| **Vercel Analytics**     | Real-time frontend performance metrics (Core Web Vitals), request counts, error rates |
| **Vercel Function Logs** | Serverless function execution logs, error traces, latency monitoring                  |
| **Neon Console**         | Database query performance insights, connection pool metrics, storage usage           |
| **Sentry (Planned)**     | Application-level error tracking and alerting for both frontend and backend           |

---

## 7. Backup & Disaster Recovery

| Scenario                                    | Recovery Mechanism                                                                                                                         |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Database corruption / accidental delete** | Neon PITR — restore to any point in the last 7 days                                                                                        |
| **Frontend deployment failure**             | Vercel instant rollback to previous deployment (single click)                                                                              |
| **Backend deployment failure**              | Vercel instant rollback; zero-downtime atomic deployments prevent mid-deploy failures from affecting users                                 |
| **File storage failure**                    | Cloudinary's distributed storage provides built-in redundancy; critical documents are also backed up to a secondary Cloudinary environment |
