# [PROJECT_NAME] Constitution
<!-- Example: Spec Constitution, TaskFlow Constitution, etc. -->

## Core Principles

### [PRINCIPLE_1_NAME]
<!-- Example: I. Library-First -->
[PRINCIPLE_1_DESCRIPTION]
<!-- Example: Every feature starts as a standalone library; Libraries must be self-contained, independently testable, documented; Clear purpose required - no organizational-only libraries -->

### [PRINCIPLE_2_NAME]
<!-- Example: II. CLI Interface -->
[PRINCIPLE_2_DESCRIPTION]
<!-- Example: Every library exposes functionality via CLI; Text in/out protocol: stdin/args → stdout, errors → stderr; Support JSON + human-readable formats -->

### [PRINCIPLE_3_NAME]
<!-- Example: III. Test-First (NON-NEGOTIABLE) -->
[PRINCIPLE_3_DESCRIPTION]
<!-- Example: TDD mandatory: Tests written → User approved → Tests fail → Then implement; Red-Green-Refactor cycle strictly enforced -->

### [PRINCIPLE_4_NAME]
<!-- Example: IV. Integration Testing -->
[PRINCIPLE_4_DESCRIPTION]
<!-- Example: Focus areas requiring integration tests: New library contract tests, Contract changes, Inter-service communication, Shared schemas -->

### [PRINCIPLE_5_NAME]
<!-- Example: V. Observability, VI. Versioning & Breaking Changes, VII. Simplicity -->
[PRINCIPLE_5_DESCRIPTION]
<!-- Example: Text I/O ensures debuggability; Structured logging required; Or: MAJOR.MINOR.BUILD format; Or: Start simple, YAGNI principles -->

## [SECTION_2_NAME]
<!-- Example: Additional Constraints, Security Requirements, Performance Standards, etc. -->

[SECTION_2_CONTENT]
<!-- Example: Technology stack requirements, compliance standards, deployment policies, etc. -->

## [SECTION_3_NAME]
<!-- Example: Development Workflow, Review Process, Quality Gates, etc. -->

[SECTION_3_CONTENT]
<!-- Example: Code review requirements, testing gates, deployment approval process, etc. -->

## Governance
<!-- Example: Constitution supersedes all other practices; Amendments require documentation, approval, migration plan -->

# BookIt — Project Constitution

## What this document contains
- Project overview and scope
- Core principles that guide design and delivery
- Key architecture decisions and rationale
- Development workflow and required steps
- Quality standards, testing, and CI gating
- Branch strategy and PR / code-review rules
- Enforcement, exceptions, and onboarding pointers

---

## 1. Project Overview

Name: BookIt

Purpose: Enable service providers to manage availability and allow customers to discover and book appointments easily and reliably.

Tech stack:
- Backend: ASP.NET Core Web API
- Frontend: Next.js 14
- Database: SQLite (via Entity Framework Core)
- Authentication: JWT (JSON Web Tokens)
- Data access: Entity Framework Core

Scope:
- Provider account and availability management
- Customer booking flow (search, book, cancel, reschedule)
- Notifications (email/webhooks)
- Admin tools for reporting and management

Audience:
- Service providers (businesses, freelancers)
- End customers booking appointments

Repo conventions:
- All feature and API specifications live in `/specs`
- Database migrations under the EF migrations folder
- API surface documented via Swagger (OpenAPI)

---

## 2. Core Principles

1. Spec-driven development
	 - Every feature begins with a spec stored in `/specs` (see "Spec template" below).
	 - Specs must be reviewed and approved before implementation.

2. Test coverage
	 - Minimum 80% coverage for business logic (unit & integration where meaningful).
	 - Coverage measured per-feature and enforced by CI gate.

3. API-first design
	 - Backend API endpoints must be defined and implemented (and minimally validated) before frontend depends on them.
	 - OpenAPI/Swagger spec is the contract between frontend and backend.

4. User experience
	 - Booking flow must be simple, fast, and unambiguous.
	 - Strive for minimal steps to complete a booking and clear messaging for errors and confirmations.

5. Code quality
	 - Clean, expressive names.
	 - Small functions, single responsibility.
	 - Robust error handling, explicit validation, and clear logs.
	 - No commented-out dead code in PRs.

---

## 3. Architecture Decisions (with rationale)

- RESTful API design
	- Use standard HTTP verbs and resources for clarity and cacheability. Keep endpoints resource-oriented.
- JWT for authentication
	- Stateless auth suitable for REST APIs; supports short-lived access tokens and optional refresh tokens.
- Entity Framework Core (EF Core)
	- Productivity + migrations support. EF Core works with SQLite for local development, lightweight deployments, and CI tests.
- SQLite as the primary database for this constitution
	- SQLite is chosen for simplicity, ease of local setup, and deterministic CI behavior. For production scenarios requiring high concurrency or distributed deployments, the team may adopt a server RDBMS (e.g., PostgreSQL) and document migration steps.
- Repository pattern for data access
	- Encapsulate data access, enable easier unit testing and swapability.
- Separation of concerns
	- Controllers → Services → Repositories. Controllers orchestrate, Services implement business logic, Repositories handle DB.
- Background processing
	- Use hosted services or background job framework for async tasks like sending notifications and long-running jobs.
- Configuration & secrets
	- Use `appsettings.{Environment}.json` + environment variables / secrets manager; never check secrets into source.

---

## 4. Development Workflow (required steps)

1. Write a spec in `/specs`
	 - Include API endpoints, request/response schemas, data model changes, UI acceptance criteria, and test cases.
	 - Add the spec to the repo and open a discussion/PR for approval if required by the team.

2. Get approval
	 - A product-owner or designated reviewer must approve the spec before work starts.

3. Create implementation issues
	 - Convert the approved spec into issues/tasks with clear acceptance criteria and test expectations.

4. Feature branch per issue
	 - Branch naming: `feature/<issue-number>-short-name` or `fix/<issue-number>-short-name`.
	 - Keep branches focused and small.

5. Implement feature
	 - Backend first. Add endpoint(s), DTOs, migrations, repository logic, services, and unit tests.
	 - Ensure Swagger/OpenAPI updated for new endpoints.
	 - Implement frontend only after API surface is stable.

6. PR with tests and docs
	 - Include unit tests and integration tests for critical flows.
	 - Add or update Swagger docs and any README or changelog entries.
	 - Self-review with the checklist (see Code Review Requirements).

7. Code review before merge
	 - At least one approver other than the author. All CI checks must pass.

8. Merge to `main` only after approval and CI
	 - Deploy to `staging` automatically (or via a gated pipeline).
	 - Run smoke tests on staging prior to release.

---

## 5. Quality Standards & Enforcement

- Swagger/OpenAPI
	- All public endpoints must have Swagger documentation and example schemas.
- Database changes
	- All schema changes must be applied via EF Core migrations committed to the repo.
- Secrets and configuration
	- No hard-coded secrets. Use environment variables or a secrets manager. Add `.env.example` if helpful.
- HTTP status codes
	- Use semantics consistently: 200 (OK), 201 (Created), 204 (No Content), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 409 (Conflict), 422 (Unprocessable Entity) where applicable, 500 for unexpected server errors.
- Input validation
	- Validate all user inputs at the API boundary (DTOs with validation attributes + explicit checks).
- Error handling
	- Return consistent error payloads. Log details server-side, return minimal safe messages to clients.
- Logging and monitoring
	- Use structured logging. Include request ids/correlation ids for tracing. Integrate basic metrics and error reporting.
- Performance
	- Keep operations within reasonable response-time budgets. Profiling and optimization required for slow endpoints.

Enforcement:
- CI pipeline gates: build, lint, unit tests, test coverage, migration validation, Swagger validation.
- PRs failing gates cannot be merged.

---

## 6. Branch Strategy

- `main`
	- Always deployable production-ready code. Merges to `main` must be done via PR only and pass CI.
- `feature/*`
	- For new features. Example: `feature/123-calendar-integration`
- `fix/*`
	- For bug fixes and hotfixes. Example: `fix/456-reschedule-bug`
- `chore/*`
	- Non-functional changes (deps, CI config, docs).
- No direct commits to `main`. Use PRs and code review.

Release flow:
- Merges to `main` trigger a staging deploy. After staging verification, promote to production (manual or automated per environment policy).

---

## 7. Code Review Requirements (PR checklist)

Before requesting review, the author must complete the self-review checklist:

- [ ] Spec in `/specs` exists and is approved (link included in PR).
- [ ] PR references the related issue(s) and acceptance criteria.
- [ ] Tests added: unit tests for business logic and integration tests for critical flows.
- [ ] Coverage: business logic coverage ≥ 80% for changed modules.
- [ ] Swagger/OpenAPI updated for new/changed endpoints.
- [ ] Database migrations included for schema changes.
- [ ] No hard-coded secrets or credentials.
- [ ] Proper HTTP status codes and consistent error payloads.
- [ ] Code compiles and all tests pass locally.
- [ ] Linting and formatting checks pass.
- [ ] No console logs, warnings, or debug-only code left.
- [ ] README or docs updated when public behavior changed.

Reviewer responsibilities:
- Confirm that tests meaningfully cover changed behavior.
- Validate security and correctness of auth flows (JWT usage).
- Confirm API shape and documentation.
- Request changes for ambiguous names, complex methods, or missing error handling.

Merging:
- At least one approving review required.
- CI gates (build, tests, coverage, lint) must be green.
- Squash or rebase per team preference (documented in repo README).

---

## 8. Testing & CI

Minimum CI pipeline steps (each PR and main branch):
1. Restore/build
2. Lint and static analysis
3. Unit tests
4. Integration tests (where quick and deterministic)
5. Test coverage report and fail if business-logic coverage < 80%
6. Run EF Core migration validation (no pending migrations mismatch)
7. Swagger validation (ensures OpenAPI is consistent)
8. Optional: run a small set of smoke/integration tests against a disposable DB

Notes:
- For CI and local development, use SQLite (file or in-memory) for fast, deterministic tests. When integration tests require features not available in SQLite, run them in a separate job with a server RDBMS.
- Tests should be deterministic and fast; split long-running integration tests to a separate pipeline.
- Use a CI-provided SQLite instance or file-based DB for integration tests when possible.

---

## 9. Security & Data Privacy

- Authentication: JWT with short lifetimes and refresh token strategy where needed.
- Authorization: Role-based and resource-based checks in services; never rely solely on frontend.
- Data protection: Encrypt sensitive data at rest if required by policy; protect backups.
- Secrets management: Environment variables and/or secrets manager. Do not commit secrets.
- GDPR/Privacy: Minimize stored personal data, provide deletion workflows for user accounts.

---

## 10. Migrations & Backwards Compatibility

- Migrations only via EF Core migrations files; checked into source control.
- When changing APIs or DB schemas:
	- Prefer additive changes and graceful migration paths.
	- Maintain backward compatibility where possible; if breaking, document migration steps and communicate timing.
- Migration checklist:
	- Add migration file and tests that validate the migration (if feasible).
	- Test both forward and rollback/restore scenarios for critical migrations.

---

## 11. Spec Template (place in `/specs/template.md`)

A recommended spec template to standardize proposals:

- Title:
- Author:
- Related issue(s):
- Summary:
- Why:
- Goals (acceptance criteria, measurable):
- API contract (endpoints, request/response JSON examples):
- Database changes (models, migrations):
- UI/UX notes (screens, flows):
- Tests required (unit/integration and scenarios):
- Rollout plan and backward compatibility considerations:
- Security/privacy concerns:
- Open questions / alternatives considered:
- Approval (reviewers & date):

---

## 12. Exceptions & Governance

- Exceptions to this constitution may be granted by (role/team lead) and must be logged with a rationale and duration.
- Periodic review: Revisit this constitution at major milestones (major release or after ~6 months).
- Enforcement: CI gates and code owners help enforce. Repeated violations may trigger stricter review or rollback.

---

## 13. Onboarding & References

Key locations:
- Specs: `/specs`
- API docs: `/swagger` or runtime Swagger at `/swagger/index.html`
- Migrations: EF migrations folder within the backend project
- Config: `appsettings.json`, `appsettings.Development.json`, environment variables
- README: Root README with setup instructions
- PR template and Issue template: `.github/PULL_REQUEST_TEMPLATE.md` and `.github/ISSUE_TEMPLATE.md` (recommended)

Suggested repo files to add if not present:
- `CONSTITUTION.md` (this document)
- `/specs/template.md` (spec template)
- `.env.example` (non-sensitive required vars)
- `README.md` with local setup and test commands
- CI config with coverage gate and migration checks

---

## Completion Summary

What this document contains:
- A professional project constitution covering Project Overview, Core Principles, Architecture Decisions, Development Workflow, Quality Standards, Branch Strategy, Code Review Requirements, Testing & CI, Security, Migrations, Spec Template, Exceptions & Onboarding.

Next suggested steps (optional):
- Commit this doc to the repo root (already added).
- Add `/specs/template.md` to the repo.
- Add a PR template and CI coverage gate if not already present.

If you want, I can create `/specs/template.md` and a PR template and add a recommended CI snippet next. Which would you like me to add next?
