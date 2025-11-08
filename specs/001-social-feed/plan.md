# Implementation Plan: 소셜 피드 (Social Feed)

**Branch**: `001-social-feed` | **Date**: 2025-11-09 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-social-feed/spec.md`

**Note**: This plan follows the constitution requirements for Next.js + Feature-Sliced Design architecture.

## Summary

CrossFeed Phase 1은 인스타그램 스타일의 소셜 피드 애플리케이션으로, 사용자가 이미지를 공유하고 댓글을 통해 소통하며 푸시 알림을 받습니다. 웹 우선(web-first) 접근 방식으로 개발하며, 4개의 User Story를 점진적으로 구현합니다: (P1) 포스트 생성, (P2) 피드 조회, (P3) 댓글, (P4) 푸시 알림.

기술 스택은 Next.js App Router + TypeScript 기반이며, Feature-Sliced Design 아키텍처를 따릅니다. 서버 상태는 React Query로 관리하고, Firebase를 스토리지 및 알림 인프라로 사용합니다.

## Technical Context

**Language/Version**: TypeScript 5.x (Next.js 14+ App Router 지원)
**Primary Dependencies**: Next.js 14+, React 18+, React Query (TanStack Query), Zustand, Firebase SDK (Storage + FCM)
**UI Library**: Tailwind CSS + shadcn/ui (Radix UI 기반 접근성 우수 컴포넌트)
**Storage**: Firebase Storage (이미지 파일), PostgreSQL (구조화된 데이터 - Vercel Postgres)
**Testing**: Jest + React Testing Library (단위/통합), Playwright (E2E - 선택)
**Target Platform**: Web 브라우저 (Phase 1), Capacitor 모바일 앱 (Phase 2 이후)
**Project Type**: Web Application (frontend + backend API)
**Performance Goals**: 피드 로딩 <2초 (20 포스트), 댓글 반영 <1초, 푸시 알림 전송 <5초, 1000 동시 사용자 지원
**Constraints**: 이미지 최대 10MB, 캡션 최대 2000자, 댓글 최대 500자, 한국어 전용 (Phase 1)
**Scale/Scope**: 초기 목표 사용자 1000명, 일 포스트 생성 ~100건, 확장성 고려한 설계

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### ✅ Architecture - Feature-Sliced Design

- **Requirement**: All features follow Next.js + FSD structure
- **Status**: PASS
- **Evidence**: 프로젝트 구조가 `app/`, `entities/`, `features/`, `widgets/`, `shared/` 계층으로 설계됨
- **Notes**:
  - `entities/user`, `entities/post`, `entities/comment` - 도메인 엔티티
  - `features/post`, `features/feed`, `features/comment`, `features/notification` - 엔티티 기반 기능 단위
  - `pages/` 대신 Next.js App Router의 `app/` 디렉터리 사용
  - 상향 의존성 없음을 코드 리뷰에서 검증

### ✅ Code Standards

- **Requirement**: TypeScript, PascalCase/camelCase/kebab-case 명명 규칙, ESLint + Prettier
- **Status**: PASS
- **Evidence**:
  - 모든 코드 TypeScript로 작성
  - 컴포넌트: `FeedList.tsx`, 함수: `useFeedQuery`, 파일: `feed-list.tsx`
  - `.eslintrc.js`, `.prettierrc` 설정 파일 포함
  - 한국어 주석 및 문서화

### ✅ API Conventions

- **Requirement**: RESTful API, JSON, 표준 HTTP 메서드 및 상태 코드
- **Status**: PASS
- **Evidence**:
  - API Routes: `GET /api/posts`, `POST /api/posts`, `POST /api/comments`, etc.
  - 모든 응답 `application/json`
  - Bearer 토큰 인증 (`Authorization: Bearer <token>`)
  - 표준 HTTP 상태 코드 사용 (200, 201, 400, 401, 404, 500)

### ✅ State Management

- **Requirement**: React Query (server state), Zustand (local state), state minimization
- **Status**: PASS
- **Evidence**:
  - React Query: `useFeedQuery`, `usePostMutation`, `useCommentQuery`
  - Zustand: UI 상태 (모달, 필터) 최소한으로만 사용
  - 컴포넌트 로컬 상태 우선 (`useState`)

### ✅ Platform & Infrastructure

- **Requirement**: FCM (push), Firebase Storage (files), Capacitor (mobile), Vercel (deploy), GitHub Actions (CI/CD)
- **Status**: PASS (Phase 1은 웹 우선, Capacitor는 Phase 2)
- **Evidence**:
  - Firebase Storage: 이미지 업로드 및 URL 관리
  - FCM: 웹 푸시 알림 (Service Worker)
  - Vercel: Next.js 배포 최적화
  - GitHub Actions: 자동 빌드 및 배포
  - **Phase 1 Scope Clarification**: Capacitor 네이티브 앱 빌드는 Phase 2로 연기, Phase 1은 반응형 웹 앱에 집중

### ✅ Release & Deployment Policy

- **Requirement**: Feature branches, PR-based merge, main 항상 배포 가능
- **Status**: PASS
- **Evidence**:
  - Feature branch: `001-social-feed`
  - PR 필수, spec 링크 포함
  - GitHub Actions workflow: `main` 브랜치 병합 시 자동 배포

### Summary

**All gates passed.** No constitutional violations. 프로젝트는 모든 아키텍처 원칙을 준수합니다.

## Project Structure

### Documentation (this feature)

```text
specs/001-social-feed/
├── spec.md              # Feature specification (completed)
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (technology decisions)
├── data-model.md        # Phase 1 output (entities and relationships)
├── quickstart.md        # Phase 1 output (setup and run instructions)
├── contracts/           # Phase 1 output (API contracts)
│   ├── auth.openapi.yaml
│   ├── posts.openapi.yaml
│   ├── comments.openapi.yaml
│   └── notifications.openapi.yaml
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

프로젝트는 **Next.js + Feature-Sliced Design** 구조를 따릅니다:

```text
feed-app/
├── src/
│   ├── app/                    # Next.js App Router (페이지 라우팅)
│   │   ├── layout.tsx          # 루트 레이아웃
│   │   ├── page.tsx            # 홈 (피드)
│   │   ├── login/
│   │   │   └── page.tsx        # 로그인
│   │   ├── register/
│   │   │   └── page.tsx        # 회원가입
│   │   ├── post/
│   │   │   └── [id]/
│   │   │       └── page.tsx    # 포스트 상세
│   │   └── api/                # API Routes (백엔드)
│   │       ├── auth/
│   │       │   ├── register/route.ts
│   │       │   └── login/route.ts
│   │       ├── posts/
│   │       │   ├── route.ts         # GET (목록), POST (생성)
│   │       │   └── [id]/
│   │       │       └── route.ts     # GET (상세), DELETE
│   │       ├── comments/
│   │       │   └── route.ts         # POST (생성), DELETE
│   │       └── notifications/
│   │           └── route.ts         # GET (목록), PATCH (읽음 처리)
│   │
│   ├── entities/               # FSD: 도메인 엔티티 (재사용 가능한 비즈니스 로직)
│   │   ├── user/
│   │   │   ├── model/
│   │   │   │   ├── types.ts         # User 타입 정의
│   │   │   │   └── schema.ts        # Validation schema
│   │   │   └── lib/
│   │   │       └── auth.ts          # 인증 헬퍼
│   │   ├── post/
│   │   │   ├── model/
│   │   │   │   └── types.ts         # Post 타입 정의
│   │   │   └── lib/
│   │   │       └── validation.ts    # 포스트 유효성 검사
│   │   ├── comment/
│   │   │   └── model/
│   │   │       └── types.ts         # Comment 타입 정의
│   │   └── notification/
│   │       └── model/
│   │           └── types.ts         # Notification 타입 정의
│   │
│   ├── features/               # FSD v2.1: 사용자 상호작용 기능 (Pages First)
│   │   ├── post/               # Post 엔티티 CRUD (create, edit, delete)
│   │   │   ├── ui/
│   │   │   │   ├── post-form.tsx
│   │   │   │   └── image-upload.tsx
│   │   │   └── model/
│   │   │       └── use-post-mutations.ts  # React Query mutations
│   │   ├── feed/               # 피드 조회 및 무한 스크롤 (여러 post 집합 뷰)
│   │   │   ├── ui/
│   │   │   │   ├── feed-list.tsx
│   │   │   │   └── feed-item.tsx
│   │   │   └── model/
│   │   │       └── use-feed-query.ts      # React Query query
│   │   ├── comment/            # Comment 엔티티 CRUD (create, delete)
│   │   │   ├── ui/
│   │   │   │   ├── comment-list.tsx
│   │   │   │   └── comment-form.tsx
│   │   │   └── model/
│   │   │       ├── use-comments-query.ts
│   │   │       └── use-comment-mutations.ts
│   │   └── notification/       # Notification 엔티티 조회 및 읽음 처리
│   │       ├── ui/
│   │       │   └── notification-list.tsx
│   │       └── model/
│   │           └── use-notifications.ts
│   │
│   ├── widgets/                # FSD: 독립적인 UI 블록 (여러 features 조합)
│   │   ├── feed-page/
│   │   │   └── ui/
│   │   │       └── feed-page.tsx         # Feed + Post
│   │   └── post-detail-page/
│   │       └── ui/
│   │           └── post-detail-page.tsx  # Post + Comment
│   │
│   └── shared/                 # FSD: 공통 유틸리티 및 UI 컴포넌트
│       ├── ui/
│       │   ├── button.tsx
│       │   ├── input.tsx
│       │   ├── modal.tsx
│       │   └── spinner.tsx
│       ├── lib/
│       │   ├── firebase.ts              # Firebase 초기화
│       │   ├── query-client.ts          # React Query 설정
│       │   └── utils.ts                 # 일반 유틸리티
│       └── api/
│           └── client.ts                # API 클라이언트 (fetch wrapper)
│
├── public/
│   ├── firebase-messaging-sw.js         # FCM Service Worker
│   └── icons/
│
├── tests/                      # 테스트 (별도 디렉터리)
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD workflow
│
├── next.config.js
├── tsconfig.json
├── .eslintrc.js
├── .prettierrc
├── package.json
└── README.md
```

**Structure Decision**:

Next.js App Router 기반 웹 애플리케이션으로, **Feature-Sliced Design v2.1 (Pages First)** 아키텍처를 적용합니다.

- **`app/`**: Next.js App Router - 페이지 라우팅 및 API Routes (서버 로직)
- **`entities/`**: 도메인 비즈니스 로직 (user, post, comment, notification)
- **`features/`**: 사용자 상호작용 기능 (post, feed, comment, notification)
  - **명명 규칙**: 엔티티 기반 단수형 (Entity-based naming)
  - DB 테이블(`posts`, `comments`, `notifications`)과 1:1 매핑
  - 각 feature는 하나의 엔티티 생명주기(CRUD) 관리
  - Phase 2 확장 시 `post-like/`, `post-report/`, `follow/` 등 추가 가능
  - 불필요한 접미사 제거 (`-creation`, `-view`, `-section`, `-center` 등)
- **`widgets/`**: 여러 features를 조합한 페이지 수준 컴포넌트
- **`shared/`**: 공통 UI 컴포넌트, 유틸리티, API 클라이언트

**FSD v2.1 "Pages First" 원칙**:
- 계층 간 단방향 의존성: `app → widgets → features → entities → shared`
- 페이지(`app/`)가 widgets를 조합하고, widgets가 features를 조합
- 각 User Story는 독립적으로 개발 및 테스트 가능

## Complexity Tracking

N/A - No constitutional violations. 모든 원칙을 준수합니다.
