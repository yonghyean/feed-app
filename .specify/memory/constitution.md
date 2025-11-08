<!--
SYNC IMPACT REPORT
==================
Version Change: [TEMPLATE] → 1.0.0
Type: INITIAL - First constitution ratification

Modified Principles:
- NEW: I. Architecture - Feature-Sliced Design
- NEW: II. Code Standards
- NEW: III. API Conventions
- NEW: IV. State Management
- NEW: V. Platform & Infrastructure
- NEW: VI. Documentation Language

Added Sections:
- Release & Deployment Policy

Templates Updated:
- ✅ plan-template.md: Constitution Check section already supports dynamic gates
- ✅ spec-template.md: No changes needed - templates are technology-agnostic
- ✅ tasks-template.md: No changes needed - templates are technology-agnostic

Follow-up TODOs:
- None - all placeholders filled

Generated: 2025-11-09
-->

# Feed App Constitution

## Core Principles

### I. Architecture - Feature-Sliced Design

**MUST**: All features follow the Next.js + Feature-Sliced Design (FSD) structure.

- Every feature MUST belong to exactly one of: `entities`, `features`, `widgets`, `pages`, `shared`, `app`
- Dependencies flow ONLY downward (higher layers → lower layers): `app` → `pages` → `widgets` → `features` → `entities` → `shared`
- Cross-layer or upward dependencies are strictly PROHIBITED
- Within each feature slice, code MUST be organized by type:
  - `ui/` - React components and presentation logic
  - `model/` - State management, business logic, and data models
  - `lib/` - Utility functions and helper libraries

**Rationale**: FSD provides a scalable, maintainable architecture that prevents circular dependencies and ensures clear separation of concerns. This structure makes it easy to understand feature boundaries and dependencies at a glance.

### II. Code Standards

**Language**: TypeScript is the ONLY language for application code.

**Naming Conventions** (NON-NEGOTIABLE):
- Components: PascalCase (e.g., `FeedList`, `UserProfile`)
- Functions/Variables: camelCase (e.g., `fetchPosts`, `userId`)
- Files: kebab-case (e.g., `feed-list.tsx`, `user-profile.tsx`)
- Hooks: `use` prefix REQUIRED (e.g., `useFeedQuery`, `useToggleModal`)
- Exports: Explicit named exports ONLY (e.g., `export { FeedList } from './ui/feed-list'`)

**Code Quality**:
- ESLint and Prettier configurations MUST be enforced
- All code MUST pass linting before commit
- Comments should explain WHY (intent, constraints, trade-offs), not WHAT (code should be self-documenting)

**Documentation Language**:
- Human-reviewed documents (specs, plans, meeting notes, in-code comments): **Korean (한국어)**
- Machine-generated documents (auto-generated API docs, test outputs, logs): English permitted

**Rationale**: Consistent naming and formatting reduces cognitive load and makes code review more efficient. Korean documentation ensures all team members can fully participate in design discussions and understand critical system constraints.

### III. API Conventions

**MUST**: All backend APIs follow REST architectural style.

**Request/Response**:
- Content-Type: `application/json`
- Methods: `GET` (read), `POST` (create), `PUT` (full update), `PATCH` (partial update), `DELETE` (remove)
- Resource paths: Plural nouns (e.g., `/posts`, `/users`, `/comments`)

**Status Codes** (HTTP standard):
- `2xx` - Success (200 OK, 201 Created, 204 No Content)
- `400` - Bad Request (validation errors)
- `401` - Unauthorized (missing/invalid auth)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `500` - Internal Server Error

**Authentication**:
- Authorization header REQUIRED: `Authorization: Bearer <token>`
- Tokens MUST be validated on all protected endpoints

**Rationale**: REST provides a widely-understood, predictable API contract. Standardized error codes simplify client-side error handling and debugging.

### IV. State Management

**Server State**: React Query is the ONLY library for server state management.
- Use for: API data fetching, caching, synchronization
- Queries for reads, Mutations for writes
- Leverage built-in cache invalidation and refetching

**Local State**: Zustand for shared client-side state.
- Use for: UI state that needs to be shared across components (modals, filters, etc.)
- Keep stores small and feature-focused

**State Minimization Principle** (NON-NEGOTIABLE):
- Global state MUST be minimized
- Feature-local state is preferred over global state
- Component state (useState) is preferred for UI-only state
- Only promote to Zustand when sharing across distant components is necessary

**Rationale**: React Query eliminates boilerplate for server state and provides excellent DX. Zustand is lightweight and prevents prop-drilling without the complexity of Redux. Keeping state local improves testability and reduces coupling.

### V. Platform & Infrastructure

**Push Notifications**: Firebase Cloud Messaging (FCM)
- MUST use FCM for all push notification delivery
- Background and foreground notification handling REQUIRED

**File Storage**: Firebase Storage
- MUST use Firebase Storage for user-generated content (images, videos, etc.)
- Implement proper access control rules

**Cross-Platform**: Capacitor
- MUST use Capacitor for native mobile capabilities (camera, file system, etc.)
- Progressive enhancement: web-first, enhance with native features where needed

**Deployment**:
- Frontend: Vercel (Next.js optimized hosting)
- CI/CD: GitHub Actions for automated testing and deployment

**Rationale**: Firebase provides a cohesive ecosystem with excellent SDKs and documentation. Capacitor enables code reuse across web and mobile. Vercel offers zero-config deployments for Next.js with automatic previews.

### VI. Release & Deployment Policy

**Branch Strategy** (NON-NEGOTIABLE):
- `main` branch MUST always be in a deployable state
- All features developed in feature branches
- Feature branch naming: `###-feature-name` (e.g., `001-user-auth`)

**Pull Request Requirements**:
- ALL code MUST be merged via Pull Request
- PRs MUST include:
  - Clear description of changes
  - Link to feature spec (if applicable)
  - Manual testing verification
- No direct commits to `main`

**Deployment Pipeline**:
- All merges to `main` trigger automatic deployment via GitHub Actions
- Failed builds BLOCK deployment
- Rollback capability MUST be maintained

**Rationale**: PR-based workflow ensures code review and knowledge sharing. Automated deployments reduce human error and enable fast iteration. Keeping `main` deployable at all times enables continuous delivery.

## Governance

**Constitution Authority**: This constitution supersedes all other development practices and guidelines.

**Amendment Process**:
1. Proposed changes MUST be documented with rationale
2. Team approval REQUIRED before adoption
3. Version MUST be incremented according to semantic versioning:
   - MAJOR: Breaking changes to core principles or architectural decisions
   - MINOR: New principles added or existing principles expanded
   - PATCH: Clarifications, wording improvements, typo fixes
4. Migration plan REQUIRED for breaking changes

**Compliance**:
- All Pull Requests MUST verify compliance with constitutional principles
- Violations MUST be justified in writing (documented in plan.md Complexity Tracking section)
- Complexity and architectural deviations require explicit approval

**Living Document**:
- Constitution should evolve with project needs
- Regular reviews recommended at major milestones
- Amendments should be rare and deliberate

**Version**: 1.0.0 | **Ratified**: 2025-11-09 | **Last Amended**: 2025-11-09
