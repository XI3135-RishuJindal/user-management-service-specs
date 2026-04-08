Implementation Tasks

Foundations and Contracts
- [ ] Import or author OpenAPI spec for User Management Service (align with LLD endpoints).
- [ ] Define response models: User, ErrorResponse, Pagination.
- [ ] Add authentication models: LoginRequest, LoginResponse (JWT and expiry).
- [ ] Document security scheme: Bearer JWT in OpenAPI.
- [ ] Validate spec with spectral/openapi-cli.

Data and Persistence
- [ ] Create database schema migrations (Flyway/Liquibase):
  - [ ] users table (id UUID, name, email unique, password_hash, created_at, updated_at, status)
  - [ ] roles table (role_id, name unique)
  - [ ] user_roles table (user_id, role_id, unique(user_id, role_id))
  - [ ] user_sessions table (id, user_id, issued_at, expires_at, jti, status)
- [ ] Add required DB indexes (users.email, users.id).
- [ ] Implement JPA/Repository interfaces (UserRepository, RoleRepository, UserSessionRepository).

Domain and Services
- [ ] Implement UserManagementService:
  - [ ] Create user with validation, hashing, and unique email enforcement.
  - [ ] Read/update/delete user; restrict updates to owner or admin.
  - [ ] Publish UserCreatedEvent (stub/event bus integration TBD).
- [ ] Implement UserVerificationService:
  - [ ] Verify credentials on login.
  - [ ] (Optional) Code-based verification flows per VerificationRequest/Response (scope TBD).
- [ ] Implement Role management helpers (assign/revoke roles).

Security
- [ ] Configure Spring Security with OAuth2/JWT:
  - [ ] JWT signing/verification keys via secrets.
  - [ ] AuthenticationManager and PasswordEncoder (bcrypt/argon2).
  - [ ] Method and endpoint-level authorization rules (USER vs ADMIN).
- [ ] Implement AuthController:
  - [ ] POST /api/v1/auth/login -> issues JWT, create session record.
  - [ ] POST /api/v1/auth/logout -> revoke session by jti/user (return 204 or 200).
  - [ ] TODO: Decide and implement /api/v1/auth/refresh if approved.
- [ ] Add security headers and CORS config as required.

Web/Controllers
- [ ] Implement UserController:
  - [ ] GET /api/v1/users (ADMIN only; paginated)
  - [ ] GET /api/v1/users/{id} (owner or ADMIN)
  - [ ] POST /api/v1/users (public)
  - [ ] PUT /api/v1/users/{id} (owner or ADMIN)
  - [ ] DELETE /api/v1/users/{id} (ADMIN)
- [ ] Implement HealthController: GET /health (return service status)

Error Handling and Validation
- [ ] Global exception handler -> ErrorResponse mapping.
- [ ] Bean validation on DTOs (name, email format, password policy).
- [ ] Consistent HTTP status codes per LLD: 201 create, 200 get/update, 204 delete.

Observability
- [ ] Add OpenTelemetry instrumentation (HTTP, DB).
- [ ] Expose Prometheus metrics endpoint.
- [ ] Structured JSON logging with correlation IDs; mask email and PII in logs.

Testing
- [ ] Unit tests for services (user creation, login, role checks).
- [ ] Web/MockMVC tests for controllers with security contexts.
- [ ] Integration tests with Testcontainers (PostgreSQL); verify migrations and repositories.
- [ ] Security tests: invalid tokens, expired tokens, privilege escalation attempts.
- [ ] Performance smoke tests for login and list users.

Packaging and Deployment
- [ ] Create Dockerfile (distroless/alpine base, non-root user).
- [ ] Helm chart or K8s manifests:
  - [ ] Deployment with readiness/liveness probes (/health)
  - [ ] Service, HPA, ConfigMap, Secret, NetworkPolicy
- [ ] Configure environment variables (DB URL, creds, JWT keys, log level).
- [ ] CI/CD pipeline:
  - [ ] Build, unit/integration tests
  - [ ] SAST/Dependency scan
  - [ ] Container image scan
  - [ ] Deploy to dev/staging with manual approval to prod

Documentation and Runbooks
- [ ] API docs generated from OpenAPI.
- [ ] Operational runbook: key alerts, dashboards, on-call procedures.
- [ ] Security checklist: key rotation, password policy, token TTLs.

Open Items / Clarifications
- [ ] Decide on token refresh strategy (/api/v1/auth/refresh) vs short-lived tokens only.
- [ ] Finalize role taxonomy (e.g., USER, ADMIN) and map to endpoints.
- [ ] Define File Storage Service integration use cases and adapter interface.
- [ ] Confirm data retention and deletion policies for PDPL/ISO 27001.