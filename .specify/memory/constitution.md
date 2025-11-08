<!--
SYNC IMPACT REPORT
==================
Version Change: 1.0.0 → 1.1.0
Type: MINOR - Expanded Release & Deployment Policy with phase-based workflow and Git conventions

Modified Principles:
- UPDATED: VI. Release & Deployment Policy
  - Added: Phase-based branching strategy (feature branches per phase)
  - Added: Git commit message conventions (Conventional Commits)
  - Added: Branch naming conventions (feature/descriptive-name, kebab-case)
  - Clarified: Feature branch workflow for implementation phases
  - Updated: Branch names focus on feature description, not phase numbers
- UPDATED: V. Platform & Infrastructure
  - Added: pnpm as mandatory package manager (npm/yarn prohibited)

Added Sections:
- Git Commit Message Format (subsection under VI)
- Phase-Based Development Workflow (subsection under VI)

Templates Updated:
- ⚠️ commands/speckit.implement.md: Should create feature branches with descriptive names
- ✅ plan-template.md: No changes needed - already phase-agnostic
- ✅ spec-template.md: No changes needed
- ✅ tasks-template.md: No changes needed

Follow-up TODOs:
- Update /speckit.implement command to create phase branches automatically with descriptive names
- Consider adding Git hooks for commit message validation (commitlint)
- Update CLAUDE.md to reflect pnpm requirement

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

**Package Management**:
- MUST use pnpm as the package manager
- npm and yarn are NOT permitted

**Rationale**: Firebase provides a cohesive ecosystem with excellent SDKs and documentation. Capacitor enables code reuse across web and mobile. Vercel offers zero-config deployments for Next.js with automatic previews. pnpm offers faster installs, better disk space efficiency, and stricter dependency resolution.

### VI. Release & Deployment Policy

**Branch Strategy** (NON-NEGOTIABLE):
- `main` branch MUST always be in a deployable state
- All features developed in feature branches
- Feature branch naming: `feature/descriptive-name` (kebab-case, describes the actual work)
  - Examples:
    - `feature/project-setup` (프로젝트 초기 설정)
    - `feature/user-authentication` (사용자 인증 기능)
    - `feature/post-creation` (포스트 생성 기능)
    - `feature/feed-pagination` (피드 페이지네이션)
    - `feature/comment-system` (댓글 시스템)

**Phase-Based Development Workflow**:

For large features with multiple implementation phases (as defined in tasks.md):
1. **Each Phase gets its own feature branch with descriptive name**:
   - Name branches by what they implement, not phase numbers
   - Examples:
     - Phase 1 (Setup) → `feature/project-setup`
     - Phase 2 (Foundational) → `feature/foundational-infrastructure`
     - Phase 3 (Post Creation) → `feature/post-creation`
     - Phase 4 (Feed View) → `feature/feed-view`
     - Phase 5 (Comments) → `feature/comment-system`
     - Phase 6 (Notifications) → `feature/push-notifications`

2. **Phase Completion Workflow**:
   - Complete all tasks in the phase
   - Commit with descriptive message following conventions (see below)
   - Create Pull Request with:
     - Title: `feat(scope): [description]` (e.g., `feat(setup): Next.js 프로젝트 초기화 및 의존성 설치`)
     - Body: Phase summary, completed tasks checklist, testing notes
     - Link to spec.md and tasks.md
   - Review and merge to `main` (or integration branch like `develop`)
   - Create next phase branch from latest `main`/`develop`

3. **Benefits of Phase-Based PRs**:
   - Incremental review reduces cognitive load
   - Earlier feedback on architecture/approach
   - Ability to deploy intermediate milestones
   - Clear project progress tracking
   - Easier rollback to last stable phase

**Git Commit Message Format** (Conventional Commits):

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

**Type** (REQUIRED):
- `feat`: 새로운 기능 추가
- `fix`: 버그 수정
- `docs`: 문서 변경
- `style`: 코드 포맷팅 (기능 변경 없음)
- `refactor`: 리팩토링 (기능 변경 없음)
- `test`: 테스트 추가/수정
- `chore`: 빌드, 설정 파일 수정

**Scope** (OPTIONAL): 변경 범위 (e.g., `auth`, `feed`, `comment`, `phase-1`)

**Subject** (REQUIRED): 50자 이내 요약 (명령형, 한국어)

**Examples**:
```
feat(phase-1): Next.js 프로젝트 초기화 및 FSD 구조 생성

feat(auth): NextAuth.js 로그인 및 회원가입 구현

fix(feed): 무한 스크롤 페이지네이션 버그 수정

docs: README에 개발 환경 설정 가이드 추가

refactor(post): 이미지 업로드 로직을 별도 hook으로 분리
```

**Commit Body** (OPTIONAL):
- 72자 단위로 줄바꿈
- 변경 이유, 주요 변경 내역, 영향 범위 설명
- Korean preferred for human communication

**Commit Footer** (OPTIONAL):
- Breaking changes: `BREAKING CHANGE: <description>`
- Issue references: `Closes #123`, `Fixes #456`
- Co-authors for pair programming

**Pull Request Requirements**:
- ALL code MUST be merged via Pull Request
- PRs MUST include:
  - Clear description of changes (Korean)
  - Link to feature spec (if applicable)
  - Completed tasks checklist from tasks.md
  - Manual testing verification
  - Screenshots/videos for UI changes
- No direct commits to `main`
- PR title MUST follow Conventional Commits format

**Deployment Pipeline**:
- All merges to `main` trigger automatic deployment via GitHub Actions
- Failed builds BLOCK deployment
- Rollback capability MUST be maintained
- Preview deployments for all PRs

**Rationale**: PR-based workflow ensures code review and knowledge sharing. Phase-based branching enables incremental delivery and reduces risk. Conventional Commits provide clear, parseable commit history for automation (changelog generation, semantic versioning). Automated deployments reduce human error and enable fast iteration. Keeping `main` deployable at all times enables continuous delivery.

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

**Version**: 1.1.0 | **Ratified**: 2025-11-09 | **Last Amended**: 2025-11-09
