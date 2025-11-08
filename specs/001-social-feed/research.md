# Research: 소셜 피드 기술 결정

**Feature**: 001-social-feed
**Date**: 2025-11-09
**Purpose**: Phase 0 - 기술 스택 및 아키텍처 패턴 연구 및 결정

## Overview

이 문서는 CrossFeed Phase 1 구현을 위한 주요 기술 결정사항을 기록합니다. 각 결정은 프로젝트 요구사항, constitution 원칙, 성능 목표를 기반으로 합니다.

---

## 1. 데이터베이스 선택: PostgreSQL vs Firebase Firestore

### Decision: **PostgreSQL (Vercel Postgres or Supabase)**

### Rationale

**PostgreSQL 선택 이유**:
- ✅ **관계형 데이터 모델 적합**: User-Post-Comment-Notification 간 명확한 관계
- ✅ **트랜잭션 지원**: 포스트 삭제 시 관련 댓글 cascade delete 보장
- ✅ **쿼리 최적화**: 인덱스 기반 피드 조회, 댓글 페이지네이션 효율적
- ✅ **Vercel 통합**: Vercel Postgres를 통한 seamless deployment
- ✅ **확장성**: 1000명 이상으로 확장 시 sharding, replication 가능
- ✅ **비용 효율**: Firestore보다 읽기/쓰기 비용 예측 가능

**Firestore 거부 이유**:
- ❌ 복잡한 관계 쿼리 제한 (JOIN 불가)
- ❌ 읽기/쓰기 비용 높음 (피드 조회 시 매번 과금)
- ❌ 트랜잭션 제약
- ❌ 인덱스 관리 복잡성

### Alternatives Considered

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Firestore | Firebase 생태계 통합, Real-time 기본 지원 | 비용, 쿼리 제한, 관계 모델링 어려움 | ❌ Rejected |
| Supabase (PostgreSQL) | PostgreSQL + Real-time, Auth 내장 | Vercel보다 덜 통합됨 | ✅ Alternative |
| Vercel Postgres | Vercel 최적화, serverless, 자동 확장 | Firebase보다 Real-time 어려움 | ✅ **Selected** |
| PlanetScale (MySQL) | Serverless MySQL, branching DB | PostgreSQL 생태계 아님 | ⚠️ Considered |

### Implementation Notes

- **Connection**: `@vercel/postgres` SDK 사용
- **ORM**: Prisma (타입 안전, 마이그레이션 관리, TypeScript 통합)
- **Schema**: 정규화된 관계형 스키마 (User, Post, Comment, Notification 테이블)
- **Indexes**:
  - `posts.created_at` (피드 정렬)
  - `comments.post_id` (댓글 조회)
  - `notifications.recipient_id, read` (알림 조회)

---

## 2. 인증 전략: NextAuth.js vs Firebase Auth

### Decision: **NextAuth.js (Auth.js v5)**

### Rationale

**NextAuth.js 선택 이유**:
- ✅ **Next.js 네이티브 통합**: App Router, Server Components, Middleware 지원
- ✅ **PostgreSQL 직접 연동**: User 테이블과 Session 테이블 통합
- ✅ **유연한 Provider 지원**: 향후 OAuth (Google, Kakao) 확장 용이
- ✅ **JWT + Session**: Bearer token 생성 및 검증 built-in
- ✅ **무료**: 비용 없음

**Firebase Auth 거부 이유**:
- ❌ 별도 User 관리 시스템 (DB와 분리)
- ❌ Next.js와 통합 복잡 (Server Components에서 제한)
- ❌ PostgreSQL과 동기화 오버헤드

### Implementation Notes

- **Provider**: Credentials (이메일/비밀번호) - Phase 1
- **Session Strategy**: JWT (stateless, Bearer token으로 사용)
- **Password Hashing**: bcrypt
- **Token Validation**: Middleware에서 `/api/*` 보호
- **Future**: Phase 2에서 OAuth providers 추가 (Google, Kakao)

---

## 3. 이미지 업로드 및 최적화

### Decision: **Firebase Storage + Next.js Image Optimization**

### Rationale

**Firebase Storage 사용** (constitution 요구사항):
- ✅ 이미지 파일 저장소로 적합
- ✅ CDN 자동 제공 (빠른 전송)
- ✅ 액세스 제어 규칙 (보안)
- ✅ 무료 tier 충분 (5GB storage, 1GB/day download)

**Next.js Image Component 사용**:
- ✅ 자동 이미지 최적화 (WebP, AVIF 변환)
- ✅ Lazy loading (viewport 진입 시 로드)
- ✅ Responsive images (srcset 자동 생성)
- ✅ Blur placeholder (UX 개선)

### Implementation Notes

1. **Upload Flow**:
   - Client → Firebase Storage (직접 업로드, signed URL 또는 Firebase SDK)
   - Storage → 업로드 완료 후 public URL 반환
   - Client → API에 URL + 메타데이터 POST (`/api/posts`)
   - Server → PostgreSQL에 post 저장

2. **Optimization**:
   - Firebase Storage Rules: 파일 크기 10MB 제한, 이미지 포맷 제한 (JPEG, PNG, WebP)
   - Next.js `<Image>`: `priority` (첫 화면), `loading="lazy"` (스크롤 피드)
   - 썸네일 생성: Firebase Cloud Functions (선택, Phase 2) 또는 Next.js Image API

3. **Security**:
   - Firebase Storage Rules: 인증된 사용자만 업로드 가능
   - Read public (CDN 캐싱 활용)

---

## 4. 푸시 알림: FCM Web Push

### Decision: **Firebase Cloud Messaging (FCM) for Web**

### Rationale

- ✅ Constitution 요구사항 충족
- ✅ 웹 푸시 알림 표준 지원 (Service Worker)
- ✅ 백그라운드 및 포그라운드 알림 모두 지원
- ✅ 무료 (무제한 메시지)

### Implementation Notes

1. **Setup**:
   - FCM SDK 초기화 (`firebase-messaging-sw.js` Service Worker)
   - 사용자 알림 권한 요청 (`Notification.requestPermission()`)
   - FCM Token 생성 및 DB 저장 (User 테이블에 `fcm_token` 컬럼)

2. **Notification Trigger**:
   - Server (API Route): 댓글 생성 시 → 포스트 작성자의 `fcm_token` 조회
   - FCM Admin SDK: 서버에서 푸시 알림 전송
   - Client: Service Worker에서 알림 수신 및 표시

3. **Deep Linking**:
   - 알림 클릭 시 → 해당 포스트 상세 페이지로 이동 (`/post/[id]`)
   - Notification payload: `{ click_action: "/post/123" }`

4. **Fallback**:
   - 웹 푸시 미지원 브라우저 (iOS Safari): 인앱 알림 배너로 대체

---

## 5. 실시간 업데이트 전략

### Decision: **Polling (Phase 1) → WebSocket/SSE (Phase 2)**

### Rationale

**Phase 1: Polling**:
- ✅ 구현 단순 (React Query `refetchInterval`)
- ✅ 서버 부하 낮음 (30초 간격)
- ✅ Stateless (serverless 친화적)

**Phase 2: WebSocket or Server-Sent Events (SSE)**:
- Real-time 요구사항 높아질 경우 고려 (채팅 기능)
- Next.js에서 WebSocket 제한적 → Ably, Pusher 같은 third-party 또는 별도 WebSocket 서버

### Implementation Notes (Phase 1)

- React Query `refetchInterval: 30000` (30초)
- "새 포스트 보기" 버튼: `refetch()` 수동 호출
- Optimistic Updates: 댓글/포스트 생성 시 즉시 UI 반영 (`useMutation` onMutate)

---

## 6. API 설계 패턴

### Decision: **RESTful API via Next.js App Router API Routes**

### Rationale

- ✅ Constitution 요구사항 (REST)
- ✅ Next.js App Router `route.ts` 네이티브 지원
- ✅ Serverless functions (Vercel Edge Functions)
- ✅ TypeScript end-to-end

### API Endpoints

| Endpoint | Method | Description | Auth |
|----------|--------|-------------|------|
| `/api/auth/register` | POST | 회원가입 | Public |
| `/api/auth/login` | POST | 로그인 (JWT 발급) | Public |
| `/api/posts` | GET | 피드 조회 (페이지네이션) | Required |
| `/api/posts` | POST | 포스트 생성 | Required |
| `/api/posts/[id]` | GET | 포스트 상세 | Required |
| `/api/posts/[id]` | DELETE | 포스트 삭제 | Required (작성자만) |
| `/api/posts/[id]/comments` | GET | 댓글 목록 | Required |
| `/api/posts/[id]/comments` | POST | 댓글 생성 | Required |
| `/api/comments/[id]` | DELETE | 댓글 삭제 | Required (작성자 or 포스트 작성자) |
| `/api/notifications` | GET | 알림 목록 | Required |
| `/api/notifications/[id]/read` | PATCH | 알림 읽음 처리 | Required |

### Implementation Notes

- **Pagination**: Cursor-based (`?cursor=<post_id>&limit=20`)
- **Error Handling**: 표준 JSON error response `{ error: string, code: number }`
- **Validation**: Zod schemas for request bodies
- **Rate Limiting**: Vercel Edge Config (향후 추가)

---

## 7. 상태 관리 아키텍처

### Decision: **React Query + Zustand (Minimal)**

### Rationale

**React Query (TanStack Query)**:
- ✅ 서버 상태 (posts, comments, notifications) 전용
- ✅ Caching, refetching, optimistic updates built-in
- ✅ TypeScript 지원 우수

**Zustand**:
- ✅ UI 상태만 (모달 open/close, 이미지 미리보기)
- ✅ 최소한으로만 사용 (constitution state minimization 원칙)

**useState (React)**:
- ✅ 컴포넌트 로컬 상태 우선 (form inputs, loading states)

### Implementation Notes

- Query Keys: `['posts']`, `['post', id]`, `['comments', postId]`
- Mutations: `useCreatePost`, `useCreateComment`, `useDeletePost`
- Cache Invalidation: `queryClient.invalidateQueries(['posts'])` on mutation success
- Zustand Store: `useModalStore` (모달 상태), `useImagePreviewStore` (이미지 프리뷰)

---

## 8. 테스트 전략

### Decision: **Jest + React Testing Library (Unit/Integration), Playwright (E2E 선택)**

### Rationale

- ✅ Next.js 공식 권장
- ✅ TypeScript 지원
- ✅ React Testing Library: 사용자 중심 테스트
- ✅ Playwright: E2E 자동화 (선택, 리소스 있을 경우)

### Testing Scope (Phase 1)

1. **Unit Tests**:
   - Utility functions (`shared/lib/utils.ts`)
   - Validation schemas (`entities/*/model/schema.ts`)

2. **Integration Tests**:
   - React Query hooks (`features/*/model/use-*.ts`)
   - API Routes (`app/api/*/route.ts`)

3. **Component Tests**:
   - UI Components (`features/*/ui/*.tsx`)
   - User interactions (button clicks, form submissions)

4. **E2E Tests (Optional)**:
   - 핵심 사용자 플로우 (회원가입 → 로그인 → 포스트 생성 → 피드 조회)

### Implementation Notes

- Test DB: SQLite in-memory (Prisma supports)
- Mocking: MSW (Mock Service Worker) for API requests
- Coverage Goal: >70% (Phase 1)

---

## 9. 성능 최적화 전략

### Decision: **Next.js SSR + React Query Prefetching + CDN**

### Rationale

**Success Criteria 달성**:
- SC-002: 피드 로딩 <2초 (20 포스트)
- SC-005: 1000 동시 사용자 지원, 응답 <3초

**최적화 전략**:
1. **Server-Side Rendering (SSR)**:
   - 첫 페이지 로드: 서버에서 초기 피드 데이터 prefetch
   - `getServerSideProps` 대신 App Router `fetch` with cache

2. **React Query Prefetching**:
   - 서버에서 초기 데이터 hydration
   - 클라이언트에서 stale-while-revalidate

3. **Image Optimization**:
   - Next.js `<Image>` 자동 최적화
   - Firebase Storage CDN (global distribution)

4. **Code Splitting**:
   - Dynamic imports for heavy components (`next/dynamic`)
   - Route-based splitting (Next.js 기본)

5. **Database Indexes**:
   - `posts.created_at DESC` (피드 정렬)
   - `comments.post_id` (댓글 조회)

6. **Caching**:
   - React Query: 5분 stale time (피드)
   - CDN: Firebase Storage 이미지 edge caching
   - Vercel Edge Functions: Static Generation where possible

### Performance Monitoring

- Vercel Analytics: Web Vitals (LCP, FID, CLS)
- React Query Devtools: Cache 상태 모니터링

---

## 10. 배포 및 CI/CD

### Decision: **Vercel + GitHub Actions**

### Rationale

- ✅ Constitution 요구사항
- ✅ Next.js 최적 플랫폼 (zero-config)
- ✅ Preview deployments (PR마다 자동)
- ✅ 자동 HTTPS, CDN
- ✅ Serverless functions

### CI/CD Pipeline

1. **GitHub Actions Workflow** (`.github/workflows/deploy.yml`):
   ```yaml
   name: Deploy
   on:
     push:
       branches: [main]
     pull_request:
       branches: [main]
   jobs:
     test:
       - Lint (ESLint, Prettier)
       - Type check (tsc --noEmit)
       - Unit tests (Jest)
     deploy:
       - Vercel deploy (main → production, PR → preview)
   ```

2. **Deployment Flow**:
   - Feature branch → PR → Preview deployment
   - Code review + manual testing
   - Merge to `main` → Production deployment
   - Rollback: Vercel dashboard one-click

---

## Summary

모든 기술 결정은 constitution 원칙과 성능 목표를 충족합니다:

| Decision | Rationale | Constitution Principle |
|----------|-----------|------------------------|
| PostgreSQL | 관계형 데이터 적합, 비용 효율 | - |
| NextAuth.js | Next.js 네이티브, PostgreSQL 통합 | II. Code Standards (TypeScript) |
| Firebase Storage + FCM | Constitution 요구, CDN, 푸시 알림 | V. Platform & Infrastructure |
| React Query + Zustand | 서버/로컬 상태 분리, 최소화 | IV. State Management |
| Polling (Phase 1) | 단순, serverless 친화 | - |
| REST API (App Router) | Constitution 요구, Next.js 네이티브 | III. API Conventions |
| Jest + RTL | Next.js 권장, TypeScript 지원 | - |
| Vercel + GitHub Actions | Constitution 요구, Next.js 최적 | V. Platform & Infrastructure, VI. Release Policy |

**Next Steps**: Phase 1 - Data Model 및 API Contracts 작성
