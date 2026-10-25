# ConstructPro ERP - Project Standards

This document outlines the coding standards, workflows, and configurations for the ConstructPro ERP project, based on the SENG 34213 Guidelines and ConstructPro SRS.

## 1. Project Context & Architecture
- **System**: ConstructPro ERP (Cloud-based enterprise resource planning for Ishara Homes Pvt. Ltd.)
- **Modules**: Sales Management (SMM), Project Management (PMM), Financial Management (FMM), Document Management (DMM), Analytics & Reporting (ARM), AI Predictive Analysis (AIPAM - via LangChain/RAG).
- **Stack**: 
  - Frontend: Next.js (React) - Style Guide: Airbnb Style Guide + ESLint + Prettier
  - Backend: Nest.js (Node.js)
  - Database: PostgreSQL (Neon DB)
  - Infra: Docker, GitHub Actions
  - Runtime Requirement: Node.js 22+

## 2. Git & GitHub Flow
- **Branches**:
  - `main`: Production-ready. Tagged with SemVer (`vMAJOR.MINOR.PATCH`).
  - `develop`: Integration branch.
  - `feature/<ticket-id>-<slug>`: Features (short-lived, max 5 working days).
  - `fix/<ticket-id>-<slug>`: Bug fixes.
  - `hotfix/<slug>`: Emergency production fixes from main.
  - `release/<version>`: Release preparation.
- **Pull Requests & Code Review**:
  - Require at least 1 approving review.
  - Code review tags: `[blocker]` (must fix), `[suggestion]`, `[question]`, `[nit]`.
  - Must pass CI pipeline and all Quality Gates before merge.

## 3. Commit Message Convention
Use **Conventional Commits**:
- Types: `feat`, `fix`, `test`, `ci`, `build`, `perf`, `refactor`, `docs`, `chore`, `style`, `revert`.
- Format:
  ```text
  type(scope): short description

  Detailed body explaining the why and how.

  Closes #<issue_number>
  Refs <epic_label> or ADR-<number>
  ```

## 4. Coding Standards & Security
- **Principles**: SOLID, DRY, YAGNI, Clean Code.
- **Secrets**: NEVER commit secrets. Use `.env.example` and GitHub Secrets.
- **Security (OWASP Top 10)**: 
  - Enforce RBAC (Role-Based Access Control) using JWT.
  - Hash passwords with bcrypt (cost >= 12).
  - Use parameterized queries / ORM to prevent SQL injection.
  - Input validation and sanitization on all endpoints.

## 5. Testing Standards (Definition of Done)
- **Unit Tests**: Minimum 80% coverage on all new code. Use Arrange-Act-Assert (AAA) / Given-When-Then pattern.
- **Integration Tests**: 100% of API endpoints must have at least one integration test against a test DB.
- **E2E Tests**: Cover all happy paths and top 3 critical flows.

## 6. CI/CD & Quality Gates
- **Pipeline Stages**: Lint & Format -> Build -> Unit Tests -> Integration Tests -> Security Scan (e.g. npm audit) -> Deploy (Staging).
- **Quality Gates**: Zero lint errors, >= 80% coverage, no high/critical vulnerabilities, all `[blocker]` comments resolved.

## 7. GitHub Issues & Epics
- **Epics**: `epic:dev-setup`, `epic:data-layer`, `epic:api`, `epic:auth`, `epic:core-features`, `epic:ui-frontend`, `epic:integration`, `epic:testing`, `epic:ci-cd`, `epic:security`, `epic:performance`, `epic:documentation`.
- **Ticket Format**: Must include User Story, Background/Context (SRS/SDS refs), Acceptance Criteria (BDD Given-When-Then), Technical Notes, Test Requirements, and Definition of Done (DoD).
