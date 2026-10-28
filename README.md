# ConstructPro ERP - Documents

[![CI](https://github.com/ConstructPro-ERP/documents/actions/workflows/ci.yml/badge.svg?branch=develop)](https://github.com/ConstructPro-ERP/documents/actions/workflows/ci.yml) [![Project coverage evidence](https://img.shields.io/badge/coverage-report%20available-blue)](https://github.com/ConstructPro-ERP/documents/blob/develop/testing/coverage-sprint8.pdf)

**Live application:** https://construct-pro-teal.vercel.app/  
**API documentation (Swagger):** https://construct-pro-api-gateway.vercel.app/api/docs  
**Project organization:** https://github.com/ConstructPro-ERP  
**Project board:** https://github.com/orgs/ConstructPro-ERP/projects/2

## Project Description

This repository stores the documentation and evaluation evidence for **ConstructPro ERP**, a cloud-based construction management system developed for **Ishara Homes Pvt. Ltd.**, Sri Lanka, as part of **SENG 34213 – System Development Project**, University of Kelaniya.

The system supports the complete lead-to-cash workflow: sales leads, quotations, construction projects, weighted milestones, tasks, project expenses, invoices, payments, project documents, dashboards, AI-based risk predictions, Google Calendar integration, role-based administration, and a read-only client portal.

The frontend uses Next.js; the backend has **14 NestJS applications: one API Gateway and 13 domain services**. Prisma 7 manages a shared Neon PostgreSQL database; services are deployed through Vercel.

**Broader project:** [ConstructPro ERP organization](https://github.com/ConstructPro-ERP)

## Demonstration Accounts

Visit the **live application landing page** linked above, explore the website, and sign in using a role-specific demonstration account.

| Role               | Email                       | Password             |
| ------------------ | --------------------------- | -------------------- |
| Administrator      | `admin.demo@example.com`    | `DemoAdministrator`  |
| Sales Manager      | `sales.demo@example.com`    | `DemoSalesManager`   |
| Project Manager    | `projects.demo@example.com` | `DemoProjectManager` |
| Accountant         | `accounts.demo@example.com` | `DemoAccountant`     |
| Client Portal User | `client.demo@example.com`   | `DemoClient`         |

The client portal provides ownership-scoped, read-only views of projects, milestones, quotations and documents. The client-facing billing page is not yet connected to invoice/payment browsing. Project documents are managed within project details rather than the standalone Documents dashboard page.

## Document Index

| Location                                                                                 | Contents                                  |
| ---------------------------------------------------------------------------------------- | ----------------------------------------- |
| [`srs/srs-final.pdf`](srs/srs-final.pdf)                                                 | Software Requirements Specification (SRS) |
| [`sds/sds-final.pdf`](sds/sds-final.pdf)                                                 | Software Design Specification (SDS)       |
| [`requirements/functional_workflows.md`](requirements/functional_workflows.md)           | Business and functional workflows         |
| [`architecture/system_architecture.md`](architecture/system_architecture.md)             | Architecture reference                    |
| [`architecture/deployment_infrastructure.md`](architecture/deployment_infrastructure.md) | Deployment and infrastructure design      |
| [`architecture/database_schema.md`](architecture/database_schema.md)                     | Prisma/database architecture              |
| [`design/security_model.md`](design/security_model.md)                                   | Security design                           |
| [`design/ui_ux_accessibility.md`](design/ui_ux_accessibility.md)                         | Interface and accessibility               |
| [`adr/README.md`](adr/README.md)                                                         | Architectural decision record index       |
| [`standards/project_standards.md`](standards/project_standards.md)                       | Project coding and process standards      |
| [`retrospectives/`](retrospectives/)                                                     | Sprint 5–8 retrospective records          |
| [`testing/test-register.pdf`](testing/test-register.pdf)                                 | Test case register and recorded results   |
| [`testing/coverage-sprint8.pdf`](testing/coverage-sprint8.pdf)                           | Sprint 8 test coverage evidence           |
| [`testing/performance-report.pdf`](testing/performance-report.pdf)                       | Recorded performance testing              |
| [`security/owasp-checklist.pdf`](security/owasp-checklist.pdf)                           | OWASP review evidence                     |

The current implementation is maintained in the source repositories. Earlier architecture documents or proposed decisions may describe planned designs; use current code and accepted decisions for implementation-specific details.

## Prerequisites

- Git to clone the repository
- A PDF reader for the PDF reports and deliverables
- A Markdown viewer or editor for `.md` documents

No Node.js installation, running backend service or database setup is needed simply to view these files.

## Installation and Viewing Instructions

1. **Clone the repository:**

   ```bash
   git clone https://github.com/ConstructPro-ERP/documents.git
   cd documents
   ```

2. **Read the documents:** Open Markdown files with GitHub or your editor; open the PDF files using a PDF viewer. Follow the index above to locate specifications, design notes and testing evidence.

This is a documentation repository, so there is no application `npm start`, database migration or build command.

## Recorded Testing and Coverage Evidence

The **Final Development Report (October 2026)** records the following verification results:

| Scope                                       | Recorded result                                            |
| ------------------------------------------- | ---------------------------------------------------------- |
| Backend Jest unit tests                     | 1,535 passed (84 suites)                                   |
| Backend Jest integration tests              | 546 passed, 2 skipped (27 suites)                          |
| Frontend tests                              | 319 passed (173 Node-based and 146 Vitest component tests) |
| Local Playwright execution, 25 October 2026 | 83 passed, 7 failed, 16 skipped, 1 not run (107 total)     |

**Backend unit-test coverage** is 76.4% of lines, 76.3% of statements, 71.5% of branches and 71.6% of functions. This measurement is specific to backend unit tests, not the merged coverage of all repositories.

The reported **frontend scoped line-coverage** measurements are 94.6% for quotation tests, 75.4% for the dashboard, and 71.4% for the client portal; there is no single verified frontend-wide percentage.

Detailed results are maintained in the test register and coverage report linked above. The Playwright figures describe a recorded **local** run and do not automatically establish the same outcome on the Vercel deployment.

## CI/CD

[Documents CI](.github/workflows/ci.yml) invokes `ConstructPro-ERP/infra/.github/workflows/documents-ci.yml@v1`. The shared workflow verifies that the repository contains Markdown files and checks Markdown content for unresolved merge-conflict markers.

A passing documentation CI badge means that these checks passed. It does not rerun the application test suites or produce a new source-code coverage percentage.

## Deployed Application

This repository does not have its own separately hosted website. The actual ConstructPro ERP application and API documentation are linked at the top of this README. Start from the live application **landing page** to view the product and access role-based demonstrations.

## Repository Maintenance

Update documents when an implemented change affects the stated requirements, behavior, architecture, APIs, results, decisions or known limitations. Keep implementation-specific descriptions consistent with the actual source code, recorded testing evidence and the accepted release scope.

## Related Repositories

| Repository                                                  | Purpose                                                 |
| ----------------------------------------------------------- | ------------------------------------------------------- |
| [Frontend](https://github.com/ConstructPro-ERP/frontend)    | Next.js application and frontend tests                  |
| [Backend](https://github.com/ConstructPro-ERP/backend)      | NestJS API gateway, domain services and database        |
| [Tests](https://github.com/ConstructPro-ERP/tests)          | Playwright API and browser E2E tests                    |
| [Infrastructure](https://github.com/ConstructPro-ERP/infra) | Reusable CI workflows and performance tooling           |
| [Documents](https://github.com/ConstructPro-ERP/documents)  | Project specification, design and verification evidence |

## Academic Project

Developed as part of **SENG 34213 – System Development Project**, Bachelor of Science (Hons.) in Software Engineering, University of Kelaniya.
