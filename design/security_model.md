# Security Model

Security in ConstructPro ERP is a multi-layered concern enforced at every tier — from the API gateway through the application layer down to the database engine. The security model is designed to comply with the **OWASP Top 10** vulnerability guidelines and to protect sensitive construction project, financial, and client data.

---

## 1. Authentication

### 1.1 Mechanism – JSON Web Token (JWT) Stateless Authentication

ConstructPro uses a **stateless JWT authentication** scheme, meaning the server does not store session state. Instead, a cryptographically signed token is issued to the client upon successful login and must be included in every subsequent request.

**Login Flow:**

```
Client → POST /auth/login (email + password)
       → NestJS validates credentials
       → bcrypt.compare(plaintext, stored_hash)
       → If valid: issue Access Token + Refresh Token
       → Return tokens in HTTP response
```

### 1.2 Password Hashing – bcrypt

All user passwords are hashed using **bcrypt** before being stored in the database. Plaintext passwords are **never stored at any point**.

| Parameter                     | Value                   | Rationale                                                                                                         |
| ----------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Algorithm**                 | bcrypt                  | Industry-standard adaptive hashing function with a configurable work factor                                       |
| **Cost Factor (Work Factor)** | `12`                    | Provides ~300ms hashing time — sufficient to deter brute-force attacks while being acceptable for user experience |
| **Salt**                      | Auto-generated per hash | Prevents rainbow table attacks; each password has a unique salt embedded in the hash                              |

### 1.3 Token Lifecycle

| Token Type        | Lifetime   | Storage Location                          | Purpose                                       |
| ----------------- | ---------- | ----------------------------------------- | --------------------------------------------- |
| **Access Token**  | 15 minutes | JavaScript memory (not localStorage)      | Authorizes API requests                       |
| **Refresh Token** | 7 days     | HTTP-only, Secure, SameSite=Strict cookie | Renews expired access tokens without re-login |

**Token Refresh Flow:**

```
Access Token expires (401 response)
     → Axios interceptor automatically sends POST /auth/refresh
     → Server validates HTTP-only Refresh Token cookie
     → Issues new Access Token (rotation)
     → Retries original failed request transparently
```

**Security Properties:**

- **HTTP-only cookie for Refresh Token** prevents JavaScript (XSS attacks) from reading the token.
- **SameSite=Strict** prevents Cross-Site Request Forgery (CSRF) from sending the refresh token cross-origin.
- **Secure flag** ensures cookies are only transmitted over HTTPS connections.

### 1.4 Logout & Token Invalidation

Since JWTs are stateless, a **blocklist** mechanism handles revocation:

- On logout, the Access Token's JTI (JWT ID) is inserted into a `token_blocklist` table in PostgreSQL.
- Every incoming request checks whether its JTI is in the blocklist before processing.
- Blocklist entries are automatically purged after the token's natural expiry (15 minutes) to prevent unbounded growth.

---

## 2. Authorization – Role-Based Access Control (RBAC)

### 2.1 Implementation

RBAC is enforced via **NestJS Guards** applied at the route level. Every protected API endpoint declares the minimum role required to access it via a `@Roles()` decorator.

**Authorization Flow:**

```
HTTP Request → JwtAuthGuard (validates token) → RolesGuard (checks user role vs. required role) → Controller Handler
```

If either guard fails, the request is rejected with an appropriate HTTP error code:

- `401 Unauthorized` — no valid token present
- `403 Forbidden` — valid token, but insufficient role

### 2.2 Role Definitions & Permission Matrix

| Permission / Module         | Admin   | Sales Manager | Project Manager | Accountant | Client Portal User |
| --------------------------- | ------- | ------------- | --------------- | ---------- | ------------------ |
| User Management             | ✅ Full | ❌            | ❌              | ❌         | ❌                 |
| Lead Management             | ✅ Full | ✅ Full       | 👁️ Read         | ❌         | ❌                 |
| Quotation Management        | ✅ Full | ✅ Full       | 👁️ Read         | 👁️ Read    | 👁️ Own Only        |
| Project Management          | ✅ Full | 👁️ Read       | ✅ Full         | 👁️ Read    | 👁️ Own Only        |
| Task & Milestone Management | ✅ Full | ❌            | ✅ Full         | ❌         | ❌                 |
| Invoice Management          | ✅ Full | ❌            | 👁️ Read         | ✅ Full    | 👁️ Own Only        |
| Payment Recording           | ✅ Full | ❌            | ❌              | ✅ Full    | ❌                 |
| Document Management         | ✅ Full | ✅ Upload     | ✅ Upload       | 👁️ Read    | 👁️ Own Only        |
| Analytics & Reports         | ✅ Full | 👁️ Sales      | 👁️ Projects     | 👁️ Finance | ❌                 |
| AI Predictions              | ✅ Full | 👁️ Read       | 👁️ Read         | 👁️ Read    | ❌                 |
| System Configuration        | ✅ Full | ❌            | ❌              | ❌         | ❌                 |

> **Legend**: ✅ Full = Create/Read/Update/Delete, 👁️ Read = Read-Only, ❌ = No Access

### 2.3 Principle of Least Privilege

Every role is granted the minimum permissions required to perform its business function. Roles cannot be self-escalated — only an Administrator can modify a user's role assignment.

---

## 3. Row-Level Security (RLS) – Database-Level Isolation

PostgreSQL **Row-Level Security (RLS)** provides a **defence-in-depth** layer for the **Client Portal User** role, enforcing data isolation at the database engine level — independent of the application layer.

### 3.1 Enforced Policies

```sql
-- Example: Client Portal users can only SELECT their own projects
CREATE POLICY client_project_isolation ON "Project"
  FOR SELECT
  TO client_portal_user
  USING (client_id = current_setting('app.current_client_id')::UUID);

-- Write operations are completely blocked for Client Portal users
CREATE POLICY client_readonly ON "Project"
  FOR INSERT TO client_portal_user USING (false);

CREATE POLICY client_readonly_update ON "Project"
  FOR UPDATE TO client_portal_user USING (false);
```

### 3.2 Tables Protected by RLS

| Table       | RLS Policy                                   |
| ----------- | -------------------------------------------- |
| `Project`   | SELECT own records only (by `client_id`)     |
| `Quotation` | SELECT own records only (by `client_id`)     |
| `Invoice`   | SELECT own records only (by `client_id`)     |
| `Document`  | SELECT documents linked to own projects only |

---

## 4. Data Protection

### 4.1 Data in Transit

| Protection            | Mechanism                                                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Protocol**          | TLS 1.2 / TLS 1.3 enforced on all connections. TLS 1.0 and 1.1 are disabled.                                             |
| **Certificate**       | Automatically provisioned and renewed by Vercel via Let's Encrypt                                                        |
| **HSTS**              | HTTP Strict Transport Security (HSTS) headers (`max-age=31536000; includeSubDomains`) prevent protocol downgrade attacks |
| **HTTP Redirect**     | All HTTP requests receive a permanent 301 redirect to HTTPS                                                              |
| **API Communication** | All frontend-to-backend API calls are made exclusively over HTTPS                                                        |

### 4.2 Data at Rest

| Component                           | Encryption Method                                                                           |
| ----------------------------------- | ------------------------------------------------------------------------------------------- |
| **PostgreSQL on Neon**              | AES-256 volume-level encryption. Encryption keys are managed by Neon's KMS.                 |
| **Cloudinary File Storage**         | Server-Side Encryption (SSE) for all uploaded documents, PDFs, and media                    |
| **Environment Variables / Secrets** | Stored in Vercel's encrypted environment variable store — never committed to source control |

---

## 5. Input Validation & Sanitization

ConstructPro implements **defense-in-depth input validation** at both the frontend and backend layers to prevent injection attacks and malformed data from reaching the database.

### 5.1 Frontend Validation

- **React Hook Form + Zod**: Client-side validation schemas provide immediate user feedback and prevent obviously invalid payloads from being submitted.
- Field constraints (e.g., max length, regex patterns for email/phone) are enforced in the UI before any API call.

### 5.2 Backend Validation – Zod Validation Pipes (NestJS)

- Every API route uses a **Zod validation pipe** that parses and validates the request body, query parameters, and route parameters against a defined schema.
- If validation fails, a `400 Bad Request` response is returned with structured error details — malformed data **never reaches service or ORM layers**.

```typescript
// Example: Create Lead DTO validation schema
const CreateLeadSchema = z.object({
  companyName: z.string().min(2).max(200),
  contactEmail: z.string().email(),
  contactPhone: z
    .string()
    .regex(/^\+?[0-9\s\-]{7,20}$/)
    .optional(),
  inquiryDetails: z.string().max(2000).optional(),
});
```

### 5.3 SQL Injection Prevention – Prisma ORM Parameterized Queries

- All database queries are executed through **Prisma ORM**, which uses parameterized queries exclusively.
- User-supplied data is **never interpolated** directly into SQL strings.
- Raw query execution (`$queryRaw`) is prohibited in the codebase by ESLint rules.

### 5.4 XSS Prevention

- **Content Security Policy (CSP)** headers are set via Next.js middleware, restricting allowed script sources to the application's own origin.
- All user-generated content rendered in the UI is escaped by React's default rendering behavior.
- `dangerouslySetInnerHTML` is prohibited by ESLint rules.

---

## 6. OWASP Top 10 Compliance

| OWASP Vulnerability                | ConstructPro Mitigation                                                    |
| ---------------------------------- | -------------------------------------------------------------------------- |
| **A01: Broken Access Control**     | NestJS RBAC Guards + PostgreSQL RLS + JWT validation                       |
| **A02: Cryptographic Failures**    | TLS 1.3 in transit, AES-256 at rest, bcrypt for passwords                  |
| **A03: Injection**                 | Prisma parameterized queries + Zod input validation                        |
| **A04: Insecure Design**           | RBAC by design, least-privilege principle enforced                         |
| **A05: Security Misconfiguration** | Environment variables in Vercel vault, CORS whitelist, HSTS headers        |
| **A06: Vulnerable Components**     | `npm audit` run in CI pipeline; Dependabot alerts enabled                  |
| **A07: Authentication Failures**   | JWT with short expiry, bcrypt cost 12, refresh token rotation, blocklist   |
| **A08: Data Integrity Failures**   | Zod DTO validation, Prisma type-safe queries                               |
| **A09: Logging Failures**          | Vercel function logs, Sentry error tracking (planned)                      |
| **A10: SSRF**                      | No server-side URL fetching from user input; Cloudinary signed upload URLs |

---

## 7. Security Headers

The following HTTP security headers are configured in the Next.js application via `next.config.js`:

| Header                      | Value                                 | Purpose                                    |
| --------------------------- | ------------------------------------- | ------------------------------------------ |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Enforces HTTPS for 1 year                  |
| `X-Content-Type-Options`    | `nosniff`                             | Prevents MIME-type sniffing                |
| `X-Frame-Options`           | `DENY`                                | Prevents clickjacking via iframe embedding |
| `X-XSS-Protection`          | `1; mode=block`                       | Legacy XSS protection (legacy browsers)    |
| `Referrer-Policy`           | `strict-origin-when-cross-origin`     | Limits referrer information leakage        |
| `Content-Security-Policy`   | Strict origin whitelist               | Prevents XSS by restricting script sources |
