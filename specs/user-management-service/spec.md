Feature/Service: User Management Service (S-01)

What and Why
- What
  - Provide REST APIs to manage users (CRUD), authenticate users (JWT issuance), and support role-based access.
  - Expose a health endpoint for runtime checks.
- Why
  - Centralizes identity and session concerns for the platform, enabling secure, compliant, and observable access for frontend/BFF and downstream services.
  - Aligns with system architecture patterns (BFF, microservices) to keep the frontend light while ensuring consistent security.

In-Scope
- User CRUD via:
  - GET /api/v1/users
  - GET /api/v1/users/{id}
  - POST /api/v1/users
  - PUT /api/v1/users/{id}
  - DELETE /api/v1/users/{id}
- Authentication
  - POST /api/v1/auth/login (issue JWT on successful authentication)
  - POST /api/v1/auth/logout (invalidate/terminate session) per service_definition
- Health
  - GET /health
- Data models (as per LLD):
  - User, UserSession; DTOs such as UserDTO
- Role management support (responsibility indicates “Support user role management”)

Out of Scope
- Onboarding flows and multi-step draft persistence (handled by Onboarding Service).
- CMS content operations (Headless CMS).
- Frontend/BFF behavior and rate limiting (enforced at BFF).
- Analytics aggregation (separate Analytics Service).

Users and Actors
- End users via frontend (through BFF).
- Admin operators (role-based privileged operations).
- System/BFF client calling User Management Service APIs.

User Stories
- As a visitor, I can create an account so I can access protected features. (POST /api/v1/users)
- As a user, I can log in with my credentials and receive a token to access my dashboard. (POST /api/v1/auth/login)
- As a user, I can retrieve and update my profile. (GET/PUT /api/v1/users/{id})
- As an admin, I can list users and deactivate/delete accounts when necessary. (GET /api/v1/users, DELETE /api/v1/users/{id})
- As the BFF, I can verify that the service is healthy. (GET /health)

Functional Requirements
- Authentication
  - Accept credentials and return a signed JWT on success.
  - Enforce Bearer authentication on protected operations.
- Authorization
  - Enforce role-based access for admin operations.
- User management
  - Create, read, update, delete users with server-side validation.
  - Unique constraint on user email.
  - Provide pagination for listing.
- Error handling
  - Use standardized ErrorResponse for error conditions.
- Observability
  - Expose /health for probes; emit metrics/traces.

Non-Functional Requirements and Constraints
- Security: OAuth2/JWT, TLS, minimal PII exposure, PDPL/ISO 27001 compliance.
- Availability: 99.99% SLA target.
- Performance: Low latency for login and profile retrieval; efficient DB queries and indices.
- Scalability: Stateless service, horizontally scalable; DB scaled appropriately.
- Compatibility: Follow OpenAPI contract; keep responses stable across minor versions.
- Logging/Tracing: Structured logs with correlation IDs; traces across controller->service->repository.
- Compliance: Access controls and audit-friendly logging (TODO: confirm audit log retention window).

Data Model (Conceptual)
- User: id, name, email, createdAt
- UserSession: session or token metadata (conceptual)
- Role/Assignments: role management support (conceptual; details TBD)

Dependencies and Integrations
- PostgreSQL for user data.
- BFF as primary caller; integrates with Identity/Onboarding at system level (per HLD).
- File Storage Service dependency noted in LLD (usage TBD).
- Observability stack per HLD (OpenTelemetry, Prometheus/Grafana).

Assumptions
- BFF enforces client-facing rate limits and CSRF; service enforces OAuth2/JWT.
- Email uniqueness is required.
- Logout semantics will invalidate server-side session or mark token unusable for subsequent requests (implementation detail clarified in plan).

Open Questions / TODO
- Token refresh endpoint: not included in current API spec; identity-api section suggests refresh route. Decision: add /api/v1/auth/refresh or delegate to another service? (TODO)
- Role taxonomy and admin boundary details (TODO).
- File Storage Service integration use cases (avatars, KYC documents) and required scopes (TODO).