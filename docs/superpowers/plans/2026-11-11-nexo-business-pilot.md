# NEXO Business Pilot Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Deliver a real, mobile-first, installable pilot for one service provider per company, with login and tenant-isolated clients, services and appointments.

**Architecture:** Vite/TypeScript PWA frontend in NEXO-BUSINESS; Node/PostgreSQL backend through Aureon Base if its deployed instance and SDK pass readiness and isolation checks. If not, pause integration and use a separately approved managed backend; no fake localStorage “production” data. A project-level tenant is not necessarily an individual business: introduce a company scope for all NEXO records and test cross-company access even within the same SaaS project.

**Tech Stack:** Node >=20, TypeScript, Vite, Vitest, Playwright, PostgreSQL; Aureon Base Node/Express SDK/API if verified. Pin dependencies and commit lockfile.

**Spec:** `docs/superpowers/specs/2026-11-11-nexo-business-design.md`

## Global Constraints

- Initial segment: “Serviços com agendamento”; one bookable resource per company.
- Product name NEXO Business; credit “por AUREON SaaS”; pt-BR, BRL, company timezone.
- No payments, WhatsApp automation, marketplace, AI app generator, global admin, multi-unit, inventory, store orders or third-party calendar in pilot.
- Public repo: no passwords, private keys, customer data or service-role credentials in code.
- Pilot is not a paid release. Paid release requires privacy/data rights, backup/restore and operational validation.

## Review Focus

- Same project but different companies: direct API access must return 403/404, never another company's data.
- Concurrent appointments at same time: second write must fail server-side without race.
- Repeated submit/network retry: no duplicate client/service/appointment.
- Invalid daylight-saving/local times: reject or normalize clearly in company timezone.
- Password reset and expired session: protected data remains inaccessible and unsaved changes are surfaced.

---

## File map

- `package.json`, `package-lock.json`, `vite.config.ts`, `tsconfig.json`: frontend/test toolchain.
- `src/main.ts`, `src/app.ts`, `src/styles.css`: shell and mobile navigation.
- `src/auth.ts`, `src/api.ts`: session and authenticated API client.
- `src/onboarding.ts`, `src/clients.ts`, `src/services.ts`, `src/appointments.ts`: focused screens/features.
- `src/types.ts`: shared domain types.
- `public/manifest.webmanifest`, `public/icons/*`: PWA metadata/assets.
- `backend/migrations/*`, `backend/routes/nexo*.js`, `backend/test/nexo*.test.js`: only if Aureon Base integration is approved and compatible; backend changes live in Aureon-Base repository, not copied into frontend.
- `tests/*.test.ts`, `e2e/*.spec.ts`: unit and browser acceptance tests.
- `docs/DEPLOYMENT.md`, `README.md`: setup, environment, release gates.

### Task 1: Backend readiness and tenant contract

**Files:** Create `docs/backend-contract.md`; test notes `docs/backend-evidence.md`.

**Interfaces:** Produce endpoint contract: `register/login/refresh/logout/reset`, `companies`, `clients`, `services`, `appointments` with company ID, role and error shape.

- [ ] **Step 1: Write a failing contract checklist** asserting deployed /ready, auth flow, email reset readiness, company-level authorization and durable storage.
- [ ] **Step 2: Run read-only smoke checks** against configured non-production endpoint. Expected: failures documented if missing; do not send real customer data.
- [ ] **Step 3: Audit Aureon Base README, schema, SDK and policies** for company-vs-project tenant boundary. Decide whether to add company isolation in its repository. Document exact endpoints/status codes and deployment prerequisite.
- [ ] **Step 4: Run isolated two-company black-box tests** with test users and verify denials on direct reads/writes. Expected: no cross-company access.
- [ ] **Step 5: Commit contract and evidence.** Stop before any live customer onboarding if backend is not deployed/verified.

### Task 2: Secure company and domain API

**Files:** Create additive migrations and routes/tests in `AUREON-TECH/Aureon-Base` after its own repo inspection; update `docs/backend-contract.md`.

**Interfaces:** `GET/POST/PATCH /api/nexo/companies`, `GET/POST/PATCH /api/nexo/clients`, `GET/POST/PATCH /api/nexo/services`, `GET/POST/PATCH /api/nexo/appointments`; every record carries `company_id`, every request validates membership server-side. Exact route shape finalized in Task 1 contract.

- [ ] **Step 1: Write failing HTTP/SQL tests** for owner creation, persisted CRUD, cross-company IDOR, unauthenticated access, duplicate requests, conflicting appointment and invalid time.
- [ ] **Step 2: Run tests.** Expected: fail due to missing routes/schema.
- [ ] **Step 3: Add minimal additive schema, constraints, RLS and routes** using transaction-safe conflict enforcement; no destructive migration.
- [ ] **Step 4: Run API/integration tests with PostgreSQL.** Expected: all pass, including same-project separate-company isolation.
- [ ] **Step 5: Run migration from empty and existing Base schema, security checks and commit.**

### Task 3: Auth and onboarding frontend

**Files:** Create `package.json`, lockfile, `index.html`, `src/main.ts`, `src/app.ts`, `src/auth.ts`, `src/api.ts`, `src/onboarding.ts`, `src/types.ts`, `src/styles.css`, `tests/auth.test.ts`, `e2e/onboarding.spec.ts`; preserve existing `Index.html` until migration is verified.

**Interfaces:** `AuthClient.signIn/signOut/register/requestReset`, `ApiClient.request<T>(path,options)`; `Company` type from contract.

- [ ] **Step 1: Write failing tests** for registration, login, logout, reset request, expired session, company creation and no data when signed out.
- [ ] **Step 2: Run unit/browser tests.** Expected: fail for missing implementation.
- [ ] **Step 3: Implement accessible mobile shell, auth and company onboarding** against Task 1/2 API; no localStorage as source of business records.
- [ ] **Step 4: Run tests/build and inspect narrow mobile viewport.** Expected: all pass; no horizontal overflow.
- [ ] **Step 5: Commit.**

### Task 4: Clients and services

**Files:** Create `src/clients.ts`, `src/services.ts`, `tests/clients.test.ts`, `tests/services.test.ts`, `e2e/catalog.spec.ts`.

**Interfaces:** `Client {id, companyId, name, contact?, notes?}`; `Service {id, companyId, name, durationMinutes, priceCents?, active}`.

- [ ] **Step 1: Write failing tests** for required name, optional contact, search/edit, positive duration, optional nonnegative price, deactivated service and duplicate submit.
- [ ] **Step 2: Run tests.** Expected: fail.
- [ ] **Step 3: Implement forms/list screens** with API persistence and safe error/empty states.
- [ ] **Step 4: Run tests and reload browser.** Expected: records persist and remain company-scoped.
- [ ] **Step 5: Commit.**

### Task 5: Agenda and dashboard

**Files:** Create `src/appointments.ts`, `src/dashboard.ts`, `tests/appointments.test.ts`, `e2e/agenda.spec.ts`.

**Interfaces:** `Appointment {id, companyId, clientId, serviceId, startsAt, endsAt, status, notes?}`; status `scheduled | completed | cancelled`.

- [ ] **Step 1: Write failing tests** for create/edit/cancel, today/upcoming dashboard, service duration, server conflict, timezone boundary, repeat submit and stale session.
- [ ] **Step 2: Run tests.** Expected: fail.
- [ ] **Step 3: Implement calendar/list and dashboard** using server-confirmed writes; show conflict and save errors.
- [ ] **Step 4: Run tests and manual mobile flow** create→reload→edit→cancel; expected: persisted and accessible.
- [ ] **Step 5: Commit.**

### Task 6: PWA, security and pilot handoff

**Files:** Create `public/manifest.webmanifest`, icons, `README.md`, `docs/DEPLOYMENT.md`, `e2e/security.spec.ts`; configure build/CI.

**Interfaces:** Pilot URL and API origin as environment configuration; no private key in frontend.

- [ ] **Step 1: Write failing checks** for manifest/icons, installability, routes after refresh, secret scan, two-company direct API isolation, login recovery and outage state.
- [ ] **Step 2: Run checks.** Expected: fail where missing.
- [ ] **Step 3: Add PWA assets, safe cache strategy, CI and deployment guide** with environment-specific callback/CORS configuration and pilot-only label.
- [ ] **Step 4: Run unit, integration, browser, accessibility, build and secret checks; inspect phone screenshot.** Expected: all pass.
- [ ] **Step 5: Commit evidence and hand off pilot.** Do not describe as paid-production ready until backup/restore, privacy/data rights and operations are verified.
