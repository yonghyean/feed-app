# Tasks: 소셜 피드 (Social Feed)

**Feature**: 001-social-feed
**Input**: Design documents from `/specs/001-social-feed/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: Tests are NOT explicitly requested in spec.md, therefore NO test tasks are included.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `- [ ] [ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Initialize Next.js 14+ project with TypeScript in project root
- [ ] T002 [P] Install core dependencies (React 18+, Next.js 14+, TypeScript 5.x) in package.json
- [ ] T003 [P] Install state management dependencies (React Query, Zustand) in package.json
- [ ] T004 [P] Install Firebase SDK (Storage + FCM) in package.json
- [ ] T005 [P] Install Prisma and PostgreSQL client dependencies in package.json
- [ ] T006 [P] Install NextAuth.js v5 in package.json
- [ ] T007 [P] Install UI dependencies (Tailwind CSS, shadcn/ui, Radix UI) in package.json
- [ ] T008 [P] Configure ESLint and Prettier in .eslintrc.js and .prettierrc
- [ ] T009 [P] Configure TypeScript in tsconfig.json
- [ ] T010 [P] Configure Next.js in next.config.js (images, Firebase Storage domain)
- [ ] T011 Create FSD directory structure (src/app/, src/entities/, src/features/, src/widgets/, src/shared/)
- [ ] T012 Create environment variables template in .env.example

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Database & ORM

- [ ] T013 Initialize Prisma in prisma/schema.prisma with PostgreSQL datasource
- [ ] T014 Define User model in prisma/schema.prisma (per data-model.md)
- [ ] T015 Define Post model in prisma/schema.prisma (per data-model.md)
- [ ] T016 Define Comment model in prisma/schema.prisma (per data-model.md)
- [ ] T017 Define Notification model with NotificationType enum in prisma/schema.prisma (per data-model.md)
- [ ] T018 Create initial Prisma migration in prisma/migrations/
- [ ] T019 Generate Prisma Client with `pnpm prisma generate`
- [ ] T020 [P] Create seed script in prisma/seed.ts (2 users, 3 posts, 5 comments for testing)

### Authentication & Authorization

- [ ] T021 Configure NextAuth.js in src/app/api/auth/[...nextauth]/route.ts
- [ ] T022 Setup JWT strategy and Credentials provider in NextAuth config
- [ ] T023 Create auth middleware in src/middleware.ts (protect /api/* routes)
- [ ] T024 Create User entity types in src/entities/user/model/types.ts
- [ ] T025 Create User validation schemas in src/entities/user/model/schema.ts (Zod)
- [ ] T026 Implement password hashing utilities in src/entities/user/lib/auth.ts (bcrypt)

### Firebase Setup

- [ ] T027 Initialize Firebase client in src/shared/lib/firebase.ts
- [ ] T028 Configure Firebase Storage rules in Firebase Console (per quickstart.md)
- [ ] T029 Configure Firebase Admin SDK in src/shared/lib/firebase-admin.ts
- [ ] T030 Create Firebase Storage upload helper in src/shared/lib/firebase-storage.ts

### Shared Infrastructure

- [ ] T031 Setup React Query client in src/shared/lib/query-client.ts
- [ ] T032 Create API client wrapper in src/shared/api/client.ts (fetch with auth headers)
- [ ] T033 Create root layout in src/app/layout.tsx (providers, fonts, metadata)
- [ ] T034 Create error handling utilities in src/shared/lib/error-handler.ts
- [ ] T035 [P] Create base UI components: Button in src/shared/ui/button.tsx
- [ ] T036 [P] Create base UI components: Input in src/shared/ui/input.tsx
- [ ] T037 [P] Create base UI components: Modal in src/shared/ui/modal.tsx
- [ ] T038 [P] Create base UI components: Spinner in src/shared/ui/spinner.tsx

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - 이미지 포스트 생성 및 공유 (Priority: P1) 🎯 MVP

**Goal**: 사용자가 이미지를 선택하고 캡션을 작성하여 피드에 게시할 수 있습니다

**Independent Test**: 사용자가 이미지를 업로드하고 캡션을 입력한 후 "게시" 버튼을 클릭하면, 포스트가 성공적으로 생성되고 데이터베이스에 저장되는지 API나 데이터베이스를 직접 확인

### Authentication (Required for US1)

- [ ] T039 [P] [US1] Implement POST /api/auth/register route in src/app/api/auth/register/route.ts (per auth.openapi.yaml)
- [ ] T040 [P] [US1] Implement POST /api/auth/login route in src/app/api/auth/login/route.ts (per auth.openapi.yaml)
- [ ] T041 [P] [US1] Create login page UI in src/app/login/page.tsx
- [ ] T042 [P] [US1] Create register page UI in src/app/register/page.tsx

### Post Entity & Services

- [ ] T043 [P] [US1] Create Post entity types in src/entities/post/model/types.ts
- [ ] T044 [P] [US1] Create Post validation schema in src/entities/post/lib/validation.ts (Zod: imageUrl required, caption max 2000 chars)
- [ ] T045 [US1] Implement POST /api/posts route in src/app/api/posts/route.ts (create post, per posts.openapi.yaml)

### Post Creation UI

- [ ] T046 [US1] Create image upload component in src/features/post/ui/image-upload.tsx (Firebase Storage integration)
- [ ] T047 [US1] Create post form component in src/features/post/ui/post-form.tsx (image + caption input)
- [ ] T048 [US1] Create post creation mutation hook in src/features/post/model/use-post-mutations.ts (React Query useMutation)
- [ ] T049 [US1] Create post creation page in src/app/create/page.tsx (uses PostForm widget)
- [ ] T050 [US1] Add post creation navigation link to app layout

**Checkpoint**: User Story 1 완료 - 사용자가 포스트를 생성하고 데이터베이스에 저장할 수 있음

---

## Phase 4: User Story 2 - 피드 조회 (Priority: P2)

**Goal**: 사용자가 피드 화면에서 다른 사용자들이 게시한 이미지 포스트들을 시간순으로 조회합니다

**Independent Test**: 여러 포스트를 사전에 생성해두고(seed data 또는 US1 기능 사용), 사용자가 피드 화면을 열었을 때 모든 포스트가 최신순으로 표시되는지 확인

### Feed API

- [ ] T051 [US2] Implement GET /api/posts route in src/app/api/posts/route.ts (feed with pagination, per posts.openapi.yaml)
- [ ] T052 [US2] Implement GET /api/posts/[id] route in src/app/api/posts/[id]/route.ts (post detail, per posts.openapi.yaml)
- [ ] T053 [US2] Create feed query hook in src/features/feed/model/use-feed-query.ts (React Query useInfiniteQuery with cursor pagination)

### Feed UI

- [ ] T054 [P] [US2] Create feed item component in src/features/feed/ui/feed-item.tsx (displays post image, author, caption, timestamp, comment count)
- [ ] T055 [P] [US2] Create feed list component in src/features/feed/ui/feed-list.tsx (infinite scroll, pull-to-refresh)
- [ ] T056 [US2] Create feed page widget in src/widgets/feed-page/ui/feed-page.tsx (combines FeedList + post creation button)
- [ ] T057 [US2] Create home page in src/app/page.tsx (uses FeedPage widget)
- [ ] T058 [US2] Add "새 포스트 보기" refresh button to feed list (manual refetch)

### Post Detail

- [ ] T059 [US2] Create post detail page in src/app/post/[id]/page.tsx (displays single post with author info)

**Checkpoint**: User Story 2 완료 - 사용자가 피드에서 모든 포스트를 시간순으로 조회할 수 있음

---

## Phase 5: User Story 3 - 댓글 작성 및 조회 (Priority: P3)

**Goal**: 사용자가 포스트에 댓글을 작성하고, 다른 사용자들의 댓글을 조회하여 소통합니다

**Independent Test**: 특정 포스트에 대해 댓글을 작성하고, 해당 포스트의 댓글 목록을 조회하여 방금 작성한 댓글이 표시되는지 확인

### Comment Entity & API

- [ ] T060 [P] [US3] Create Comment entity types in src/entities/comment/model/types.ts
- [ ] T061 [P] [US3] Implement GET /api/posts/[postId]/comments route in src/app/api/posts/[postId]/comments/route.ts (per comments.openapi.yaml)
- [ ] T062 [P] [US3] Implement POST /api/posts/[postId]/comments route in src/app/api/posts/[postId]/comments/route.ts (create comment, per comments.openapi.yaml)
- [ ] T063 [P] [US3] Implement DELETE /api/comments/[id] route in src/app/api/comments/[id]/route.ts (delete comment, per comments.openapi.yaml)

### Comment Hooks

- [ ] T064 [P] [US3] Create comments query hook in src/features/comment/model/use-comments-query.ts (React Query useQuery)
- [ ] T065 [P] [US3] Create comment mutations hook in src/features/comment/model/use-comment-mutations.ts (React Query useMutation for create/delete)

### Comment UI

- [ ] T066 [P] [US3] Create comment list component in src/features/comment/ui/comment-list.tsx (displays all comments with author, content, timestamp)
- [ ] T067 [P] [US3] Create comment form component in src/features/comment/ui/comment-form.tsx (input + submit button, max 500 chars)
- [ ] T068 [US3] Create post detail page widget in src/widgets/post-detail-page/ui/post-detail-page.tsx (combines Post + CommentList + CommentForm)
- [ ] T069 [US3] Update post detail page in src/app/post/[id]/page.tsx to use PostDetailPage widget
- [ ] T070 [US3] Add comment count and "댓글 보기" link to feed item component in src/features/feed/ui/feed-item.tsx
- [ ] T071 [US3] Implement optimistic updates for comment creation in use-comment-mutations.ts

**Checkpoint**: User Story 3 완료 - 사용자가 포스트에 댓글을 작성하고 조회할 수 있음

---

## Phase 6: User Story 4 - 푸시 알림 수신 (Priority: P4)

**Goal**: 사용자는 자신의 포스트에 댓글이 달리거나 다른 상호작용이 발생하면 푸시 알림을 받습니다

**Independent Test**: 한 사용자가 다른 사용자의 포스트에 댓글을 작성하면, 포스트 작성자에게 푸시 알림이 전송되는지 확인

### Notification Entity & API

- [ ] T072 [P] [US4] Create Notification entity types in src/entities/notification/model/types.ts
- [ ] T073 [P] [US4] Implement GET /api/notifications route in src/app/api/notifications/route.ts (per notifications.openapi.yaml)
- [ ] T074 [P] [US4] Implement PATCH /api/notifications/[id]/read route in src/app/api/notifications/[id]/read/route.ts (per notifications.openapi.yaml)
- [ ] T075 [P] [US4] Implement PATCH /api/notifications/read-all route in src/app/api/notifications/read-all/route.ts (per notifications.openapi.yaml)

### FCM Setup

- [ ] T076 [US4] Create Firebase Cloud Messaging service worker in public/firebase-messaging-sw.js (per quickstart.md)
- [ ] T077 [US4] Implement FCM token registration in src/shared/lib/fcm.ts (request permission, save token to DB)
- [ ] T078 [US4] Create FCM notification sender utility in src/shared/lib/fcm-admin.ts (Firebase Admin SDK)
- [ ] T079 [US4] Add FCM token update to user profile (update User.fcmToken in database)

### Notification Business Logic

- [ ] T080 [US4] Add notification creation logic to POST /api/posts/[postId]/comments route (create COMMENT notification when comment is created)
- [ ] T081 [US4] Add FCM push notification trigger to POST /api/posts/[postId]/comments route (send push after notification creation)
- [ ] T082 [US4] Implement notification filtering logic (skip notification if user comments on own post)

### Notification UI

- [ ] T083 [P] [US4] Create notification list component in src/features/notification/ui/notification-list.tsx
- [ ] T084 [P] [US4] Create notifications query hook in src/features/notification/model/use-notifications.ts (React Query useQuery)
- [ ] T085 [US4] Add notification badge to app layout header (unread count)
- [ ] T086 [US4] Create notifications page in src/app/notifications/page.tsx
- [ ] T087 [US4] Implement notification click handler (navigate to related post)
- [ ] T088 [US4] Add foreground notification handler in src/shared/lib/fcm.ts (in-app notification banner)

**Checkpoint**: User Story 4 완료 - 사용자가 댓글 알림을 받고 확인할 수 있음

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories and production readiness

### Additional Features

- [ ] T089 [P] Implement DELETE /api/posts/[id] route in src/app/api/posts/[id]/route.ts (delete post with cascade, per posts.openapi.yaml)
- [ ] T090 [P] Add post deletion UI to feed item and post detail (delete button for own posts)
- [ ] T091 [P] Add comment deletion UI to comment list (delete button for own comments or post author)
- [ ] T092 Add loading states to all forms and data fetching components
- [ ] T093 Add error boundaries in src/app/error.tsx and src/app/global-error.tsx
- [ ] T094 Add 404 not found page in src/app/not-found.tsx

### Performance & Optimization

- [ ] T095 [P] Add Next.js Image optimization to all image displays (use next/image component)
- [ ] T096 [P] Add database indexes validation (ensure all indexes from data-model.md are created)
- [ ] T097 Implement React Query stale time configuration (5 minutes for feed)
- [ ] T098 Add lazy loading for heavy components using next/dynamic

### Developer Experience

- [ ] T099 [P] Create Docker setup per quickstart.md (docker-compose.yml, Dockerfile.dev)
- [ ] T100 [P] Create README.md with setup and run instructions
- [ ] T101 [P] Add TypeScript type check script in package.json (tsc --noEmit)
- [ ] T102 Run quickstart.md validation (verify all setup steps work)

### Deployment Preparation

- [ ] T103 [P] Create Vercel configuration in vercel.json
- [ ] T104 [P] Create GitHub Actions workflow in .github/workflows/deploy.yml (lint, type-check, deploy)
- [ ] T105 [P] Setup environment variables in Vercel dashboard
- [ ] T106 Verify all API routes have proper error handling and status codes

### Code Quality

- [ ] T107 Run ESLint and fix all warnings
- [ ] T108 Run Prettier and format all code
- [ ] T109 Review all components for accessibility (ARIA labels, keyboard navigation)
- [ ] T110 Add loading skeletons to feed and post detail pages

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational phase completion
- **User Story 2 (Phase 4)**: Depends on Foundational phase completion (NOT dependent on US1, but needs auth from US1 for full functionality)
- **User Story 3 (Phase 5)**: Depends on Foundational phase completion + US2 (needs posts to exist for comments)
- **User Story 4 (Phase 6)**: Depends on Foundational phase completion + US3 (needs comments for notifications)
- **Polish (Phase 7)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Needs auth from US1 for full testing but is independently testable
- **User Story 3 (P3)**: Depends on US2 (needs posts to comment on) - Should test with seed data or US1/US2 created posts
- **User Story 4 (P4)**: Depends on US3 (needs comment events to trigger notifications)

### Within Each User Story

**User Story 1**:
- Auth routes (T039, T040) before API routes (T045)
- Entity types (T043) before validation (T044) before API (T045)
- API (T045) before UI mutations (T048)
- All components before page integration (T049)

**User Story 2**:
- API routes (T051, T052) before query hooks (T053)
- Query hooks (T053) before UI components (T054, T055)
- Components before page widgets (T056, T057)

**User Story 3**:
- Entity types (T060) before API routes (T061-T063)
- API routes before hooks (T064, T065)
- Hooks before UI components (T066, T067)
- Components before widget (T068) and page integration (T069)

**User Story 4**:
- Entity types (T072) before API routes (T073-T075)
- FCM setup (T076-T079) in parallel with API
- Business logic (T080-T082) after API and FCM setup
- UI components (T083-T088) after business logic

### Parallel Opportunities

**Phase 1 (Setup)**:
- T002-T010 can all run in parallel (different config files)

**Phase 2 (Foundational)**:
- T014-T017 (Prisma models) can run in parallel
- T024-T026 (User entity) can run in parallel
- T027-T030 (Firebase) can run in parallel
- T035-T038 (UI components) can run in parallel

**User Story 1**:
- T039-T042 (Auth routes and pages) can run in parallel
- T043-T044 (Post entity) can run in parallel

**User Story 2**:
- T054-T055 (Feed components) can run in parallel

**User Story 3**:
- T061-T063 (Comment API routes) can run in parallel
- T064-T065 (Comment hooks) can run in parallel
- T066-T067 (Comment UI) can run in parallel

**User Story 4**:
- T072-T075 (Notification API) can run in parallel
- T083-T084 (Notification UI) can run in parallel

**Phase 7 (Polish)**:
- T089-T091 (Deletion features) can run in parallel
- T095-T096 (Optimizations) can run in parallel
- T099-T102 (Developer experience) can run in parallel
- T103-T106 (Deployment) can run in parallel

---

## Parallel Example: User Story 1

```bash
# After Foundational phase complete, launch US1 auth components in parallel:
Task: "Implement POST /api/auth/register route in src/app/api/auth/register/route.ts"
Task: "Implement POST /api/auth/login route in src/app/api/auth/login/route.ts"
Task: "Create login page UI in src/app/login/page.tsx"
Task: "Create register page UI in src/app/register/page.tsx"

# Launch US1 post entity tasks in parallel:
Task: "Create Post entity types in src/entities/post/model/types.ts"
Task: "Create Post validation schema in src/entities/post/lib/validation.ts"
```

---

## Implementation Strategy

### MVP First (User Story 1 + User Story 2)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Post Creation)
4. Complete Phase 4: User Story 2 (Feed View)
5. **STOP and VALIDATE**: Test that users can create posts and view them in feed
6. Deploy/demo if ready (minimal viable product!)

### Incremental Delivery

1. Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Users can create posts (but can't view feed yet)
3. Add User Story 2 → Test independently → Users can create and view posts (MVP!)
4. Add User Story 3 → Test independently → Users can create, view posts, and comment
5. Add User Story 4 → Test independently → Full social feed with notifications
6. Polish → Production ready

### Recommended Sequence

Given the dependencies:

1. **Phase 1-2**: Setup + Foundational (required for everything)
2. **Phase 3**: User Story 1 (P1) - Post creation
3. **Phase 4**: User Story 2 (P2) - Feed view → **First deployable MVP**
4. **Phase 5**: User Story 3 (P3) - Comments → **Enhanced social features**
5. **Phase 6**: User Story 4 (P4) - Notifications → **Complete feature set**
6. **Phase 7**: Polish → **Production ready**

### Parallel Team Strategy

With 2 developers after Foundational phase completes:

- **Sprint 1**:
  - Dev A: User Story 1 (Post creation)
  - Dev B: User Story 2 (Feed view, can use seed data initially)
- **Sprint 2**:
  - Dev A: User Story 3 (Comments)
  - Dev B: User Story 4 (Notifications)
- **Sprint 3**:
  - Both: Polish & deployment

---

## Task Summary

- **Total Tasks**: 110
- **Phase 1 (Setup)**: 12 tasks
- **Phase 2 (Foundational)**: 26 tasks (BLOCKING)
- **Phase 3 (US1 - Post Creation)**: 12 tasks
- **Phase 4 (US2 - Feed View)**: 9 tasks
- **Phase 5 (US3 - Comments)**: 12 tasks
- **Phase 6 (US4 - Notifications)**: 17 tasks
- **Phase 7 (Polish)**: 22 tasks

**Parallel Opportunities**: 45 tasks marked [P] can run in parallel with others in their phase

**MVP Scope**: Phase 1 + Phase 2 + Phase 3 + Phase 4 = 59 tasks (Setup + Foundational + US1 + US2)

---

## Notes

- All tasks follow strict checklist format: `- [ ] [ID] [P?] [Story?] Description with file path`
- [P] tasks = different files, no dependencies within same phase
- [Story] label (US1-US4) maps task to specific user story for traceability
- Each user story should be independently completable and testable
- File paths follow Next.js + FSD structure from plan.md
- No test tasks included (not requested in spec.md)
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Suggested MVP: Complete through Phase 4 (US1 + US2) for minimal deployable product
