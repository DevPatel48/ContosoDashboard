<!-- SYNC IMPACT REPORT
  Version change: N/A → v1.0.0 (initial)
  Modified principles: N/A (initial creation)
  Added sections:
    - Core Principles (5): Security-First, Clean Architecture, Offline-First,
      Test-First, Spec-Driven Development
    - Technology Stack
    - Security Requirements
    - Development Workflow
    - Code Quality Gates
    - Governance
  Templates requiring updates:
    - .specify/templates/plan-template.md: ✅ Constitution Check section references
      constitution principles generically — no changes needed
    - .specify/templates/spec-template.md: ✅ No constitution-specific references — no changes needed
    - .specify/templates/tasks-template.md: ✅ No constitution-specific references — no changes needed
  Follow-up TODOs: None
-->

# ContosoDashboard Constitution

## Core Principles

### I. Security-First

Every feature MUST implement defense-in-depth security from the outset. Security is never an afterthought or a final checklist item.

**Non-negotiable rules:**
- All data access MUST pass service-level authorization checks, not rely solely on UI-level guards
- IDOR (Insecure Direct Object Reference) protection is mandatory — every service method MUST verify the requesting user has permission for the requested resource
- All Blazor pages with sensitive data MUST carry `[Authorize]` attributes with appropriate role policies
- Security headers (CSP, X-Frame-Options, X-Content-Type-Options, X-XSS-Protection, Referrer-Policy) MUST be applied globally via middleware
- Cookies MUST use secure flags with sliding expiration; sensitive data MUST never be stored in client-side storage
- Role-based access control (RBAC) policies MUST be defined in `Program.cs` and enforced consistently across all layers

**Rationale:** The project serves as a training example for secure development patterns. Demonstrating proper security practices is a primary educational objective.

### II. Clean Architecture

The codebase MUST maintain strict separation of concerns across architectural layers.

**Non-negotiable rules:**
- **Models** (`Models/`) contain only domain entities — no business logic, no UI concerns
- **Services** (`Services/`) encapsulate all business logic and data access rules — no direct UI coupling
- **Data** (`Data/`) contains only EF Core context and configuration — no business rules
- **Pages** (`Pages/`) contain only presentation logic — they MUST delegate to services, never access the database directly
- Interfaces MUST define service contracts; implementations MUST depend on abstractions, not concrete types
- Circular dependencies between layers are strictly forbidden

**Rationale:** Clean architecture enables independent testing, clear code ownership, and straightforward refactoring. For a training project, demonstrating proper layering is essential.

### III. Offline-First

The application MUST function entirely without external service dependencies.

**Non-negotiable rules:**
- No external API calls, CDN dependencies for core functionality, or cloud service integrations in the codebase
- Database MUST use SQL Server LocalDB — accessible without network connectivity
- Authentication MUST use the mock cookie-based system — no Azure AD, OAuth, or external identity providers
- All abstractions (interfaces, service contracts) MUST be designed to allow future cloud migration without breaking changes
- Feature designs MUST evaluate offline compatibility before introducing any external dependency

**Rationale:** The training application must be accessible to all learners regardless of network conditions. The offline-first constraint also forces cleaner abstraction boundaries.

### IV. Test-First (NON-NEGOTIABLE)

Every new feature MUST have tests written and approved before implementation code.

**Non-negotiable rules:**
- Tests MUST be written before implementation code — Red-Green-Refactor cycle is mandatory
- Service layer tests MUST verify authorization logic, including negative cases (unauthorized access attempts)
- Integration tests MUST cover the full request path: page → service → data context
- Unit tests MUST cover model validation and business rule enforcement
- Test coverage for new features MUST reach the same quality bar as existing code
- Tests that are commented out, skipped, or marked `Ignore` MUST be justified with a TODO and a deadline

**Rationale:** Test-first discipline ensures features are designed for testability and that security-critical paths are verified before code is considered complete.

### V. Spec-Driven Development

Every feature MUST originate from a specification before any code is written.

**Non-negotiable rules:**
- All features MUST follow the Spec Kit workflow: `/speckit.specify` → `/speckit.clarify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement`
- A feature specification (`spec.md`) MUST exist and be approved before planning begins
- An implementation plan (`plan.md`) MUST include architecture decisions, data model changes, and constitution compliance checks
- Task lists (`tasks.md`) MUST be dependency-ordered and grouped by user story
- No code changes are permitted without a corresponding spec and plan in `/specs/`
- Feature branches MUST follow the naming convention `###-feature-name` with sequential numbering

**Rationale:** Spec-driven development prevents scope creep, ensures alignment, and produces documentation that serves as training material for learners.

## Technology Stack

**Framework**: ASP.NET Core 8.0 with Blazor Server rendering model
**UI**: Bootstrap 5.3 for styling, Bootstrap Icons for iconography
**Database**: SQL Server LocalDB via Entity Framework Core 8.0
**Authentication**: Cookie-based mock authentication (training implementation)
**Authorization**: Claims-based identity with role-based access control policies
**Build System**: .NET SDK with MSBuild; `dotnet build`, `dotnet run`, `dotnet test`
**Package Management**: NuGet for all third-party dependencies

**Stack Constraints:**
- Target framework MUST remain .NET 8.0 LTS unless a constitution amendment approves upgrade
- New NuGet packages MUST be justified in the feature spec and approved during planning
- Frontend dependencies are limited to Bootstrap 5.x; no JavaScript framework additions without constitution amendment

## Security Requirements

**Authentication:**
- Mock authentication system uses cookie-based sessions with 8-hour sliding expiration
- Four training user accounts with distinct roles: Administrator, ProjectManager, TeamLead, Employee
- Login/logout implemented as Razor Pages for proper HTTP request handling
- Custom `AuthenticationStateProvider` bridges cookie auth to Blazor Server's authorization system

**Authorization:**
- Role hierarchy: Administrator > ProjectManager > TeamLead > Employee
- Authorization policies defined in `Program.cs` via `AddAuthorization`
- Every service method MUST validate the caller's role against the resource ownership
- Pages MUST use `[Authorize]` with role-specific policies where access is restricted

**Data Protection:**
- IDOR protection: Service methods verify user-resource relationship before returning data
- User isolation: Each user sees only their authorized data across all views
- No sensitive data caching in Blazor component state

**Headers & Transport:**
- HTTPS enforced via `UseHttpsRedirection` and HSTS
- Content Security Policy restricts script/style/font sources to self and CDN
- X-Frame-Options set to DENY, X-Content-Type-Options set to nosniff
- Referrer-Policy set to strict-origin-when-cross-origin

## Development Workflow

**Feature Lifecycle:**
1. **Specify** (`/speckit.specify`) — Create feature specification from user description
2. **Clarify** (`/speckit.clarify`) — Resolve ambiguities and edge cases in the spec
3. **Plan** (`/speckit.plan`) — Generate implementation plan with architecture decisions
4. **Task** (`/speckit.tasks`) — Produce dependency-ordered task list grouped by user story
5. **Implement** (`/speckit.implement`) — Execute tasks following test-first discipline

**Branch Management:**
- Feature branches follow `###-feature-name` naming convention with sequential numbers
- `/speckit.git.feature` creates a new feature branch with the next sequential number
- Main branch MUST always be in a buildable state
- Feature branches are merged after implementation and review

**Code Review Checklist:**
- Constitution compliance verified (all 5 principles satisfied)
- Test coverage for new code meets existing quality bar
- Security checks: authorization in services, `[Authorize]` on pages, no IDOR gaps
- Architecture compliance: correct layer usage, no circular dependencies
- No external dependencies introduced without spec approval

## Code Quality Gates

**Naming Conventions:**
- PascalCase for types, methods, and properties
- camelCase for local variables and private fields
- `_prefix` for private fields is NOT used; expression-bodied properties preferred
- Interface names prefixed with `I` (e.g., `IUserService`)
- File names MUST match the primary type name

**Documentation:**
- XML documentation comments on all public APIs and service interfaces
- Razor components MUST include summary comments describing purpose and authorization requirements
- Complex business logic MUST include inline comments explaining the "why," not the "what"

**Code Style:**
- C# top-level statements for `Program.cs`; traditional class structure for all other files
- `record` types for immutable data transfer objects; `class` for entities with behavior
- `null`-conditional operators (`?.`) and null-forgiving operators (`!`) used judiciously
- Pattern matching preferred over type casting and chained conditionals
- Async/await pattern for all I/O operations; synchronous blocking is prohibited

**Linting:**
- Analyzer warnings are treated as errors during builds
- `Nullable` context is enabled project-wide (`<Nullable>enable</Nullable>`)
- Unused usings and variables MUST be removed before commit

## Governance

**Authority:** This Constitution supersedes all other development practices, guidelines, and conventions for the ContosoDashboard project. When a practice conflicts with the Constitution, the Constitution prevails.

**Amendments:**
- Amendments require: (a) written proposal describing the change, (b) justification for why existing principles are insufficient, (c) impact assessment on existing code, (d) approval by project maintainers
- Version bumps follow semantic versioning: MAJOR for principle removals or redefinitions, MINOR for new principles or sections, PATCH for clarifications and wording refinements
- All amendments MUST update the `LAST_AMENDED_DATE` and increment `CONSTITUTION_VERSION`

**Compliance:**
- Every feature spec MUST include a Constitution Check section verifying alignment with all principles
- The implementation plan MUST explicitly address any principle tensions or trade-offs
- Code reviews MUST verify constitution compliance as a mandatory gate
- Violations MUST be documented with a TODO and a remediation deadline

**Training Alignment:**
- All constitution rules serve dual purposes: project governance AND training demonstration
- Security patterns, architecture decisions, and development workflows are intentionally explicit to model best practices for learners

**Version**: 1.0.0 | **Ratified**: 2026-05-22 | **Last Amended**: 2026-05-22
