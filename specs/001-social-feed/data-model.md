# Data Model: 소셜 피드

**Feature**: 001-social-feed
**Date**: 2025-11-09
**Database**: PostgreSQL (Vercel Postgres)
**ORM**: Prisma

## Overview

이 문서는 CrossFeed Phase 1의 데이터 모델을 정의합니다. 모든 엔티티는 PostgreSQL 데이터베이스에 저장되며, Prisma ORM을 통해 관리됩니다.

---

## Entity Relationship Diagram

```text
┌─────────────┐
│    User     │
└──────┬──────┘
       │
       │ 1:N
       │
   ┌───▼────────┐
   │    Post    │
   └───┬────────┘
       │
       │ 1:N
       │
   ┌───▼─────────┐
   │   Comment   │
   └─────────────┘

User 1:N Notification (받는 사람)
User 1:N Notification (보낸 사람)
Post 1:N Notification (관련 포스트)
```

---

## Entities

### 1. User (사용자)

사용자 계정 정보를 저장합니다.

**Table**: `users`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PRIMARY KEY, DEFAULT uuid_generate_v4() | 사용자 고유 ID |
| `email` | VARCHAR(255) | UNIQUE, NOT NULL | 이메일 (로그인 ID) |
| `password_hash` | VARCHAR(255) | NOT NULL | 비밀번호 해시 (bcrypt) |
| `username` | VARCHAR(50) | UNIQUE, NOT NULL | 사용자명 (고유, 표시용) |
| `display_name` | VARCHAR(100) | NOT NULL | 표시 이름 |
| `profile_image_url` | TEXT | NULLABLE | 프로필 사진 URL (Firebase Storage) |
| `bio` | TEXT | NULLABLE | 자기소개 (최대 500자) |
| `fcm_token` | TEXT | NULLABLE | FCM 푸시 알림 토큰 |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성일시 |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | 수정일시 |

**Indexes**:
- `idx_users_email` ON `email` (로그인 조회 최적화)
- `idx_users_username` ON `username` (사용자명 검색)

**Validation Rules**:
- `email`: 이메일 형식 검증 (Zod schema)
- `password`: 최소 8자, 영문+숫자+특수문자 조합
- `username`: 3-50자, 영문+숫자+언더스코어만
- `bio`: 최대 500자

**Relationships**:
- `posts`: User → Post (1:N)
- `comments`: User → Comment (1:N)
- `notifications_received`: User → Notification (1:N) as recipient
- `notifications_sent`: User → Notification (1:N) as actor

---

### 2. Post (포스트)

사용자가 생성한 이미지 포스트를 저장합니다.

**Table**: `posts`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PRIMARY KEY, DEFAULT uuid_generate_v4() | 포스트 고유 ID |
| `author_id` | UUID | FOREIGN KEY → users(id), NOT NULL, ON DELETE CASCADE | 작성자 ID |
| `image_url` | TEXT | NOT NULL | 이미지 URL (Firebase Storage) |
| `caption` | TEXT | NULLABLE | 캡션 (최대 2000자) |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성일시 |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | 수정일시 |

**Indexes**:
- `idx_posts_created_at` ON `created_at DESC` (피드 정렬 최적화)
- `idx_posts_author_id` ON `author_id` (작성자별 포스트 조회)

**Validation Rules**:
- `image_url`: Firebase Storage URL 형식 (https://firebasestorage.googleapis.com/...)
- `caption`: 최대 2000자

**Relationships**:
- `author`: Post → User (N:1)
- `comments`: Post → Comment (1:N)
- `notifications`: Post → Notification (1:N)

**Cascade Delete**:
- 포스트 삭제 시 → 관련 comments, notifications 자동 삭제

---

### 3. Comment (댓글)

포스트에 달린 댓글을 저장합니다.

**Table**: `comments`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PRIMARY KEY, DEFAULT uuid_generate_v4() | 댓글 고유 ID |
| `post_id` | UUID | FOREIGN KEY → posts(id), NOT NULL, ON DELETE CASCADE | 포스트 ID |
| `author_id` | UUID | FOREIGN KEY → users(id), NOT NULL, ON DELETE CASCADE | 작성자 ID |
| `content` | TEXT | NOT NULL | 댓글 내용 (최대 500자) |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성일시 |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | 수정일시 |

**Indexes**:
- `idx_comments_post_id_created_at` ON `(post_id, created_at ASC)` (포스트별 댓글 조회)
- `idx_comments_author_id` ON `author_id` (작성자별 댓글 조회)

**Validation Rules**:
- `content`: 1-500자, NOT NULL

**Relationships**:
- `post`: Comment → Post (N:1)
- `author`: Comment → User (N:1)

**Cascade Delete**:
- 포스트 삭제 시 → 댓글 자동 삭제
- 사용자 삭제 시 → 댓글 자동 삭제 (또는 author_id NULL + "삭제된 사용자" 표시 - 선택)

---

### 4. Notification (알림)

사용자 활동에 대한 알림을 저장합니다.

**Table**: `notifications`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PRIMARY KEY, DEFAULT uuid_generate_v4() | 알림 고유 ID |
| `recipient_id` | UUID | FOREIGN KEY → users(id), NOT NULL, ON DELETE CASCADE | 수신자 ID |
| `actor_id` | UUID | FOREIGN KEY → users(id), NOT NULL, ON DELETE SET NULL | 행위자 ID (댓글 작성자) |
| `type` | ENUM | NOT NULL | 알림 유형: 'COMMENT' (Phase 1) |
| `post_id` | UUID | FOREIGN KEY → posts(id), NULLABLE, ON DELETE CASCADE | 관련 포스트 ID |
| `comment_id` | UUID | FOREIGN KEY → comments(id), NULLABLE, ON DELETE CASCADE | 관련 댓글 ID |
| `is_read` | BOOLEAN | NOT NULL, DEFAULT false | 읽음 여부 |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성일시 |

**Indexes**:
- `idx_notifications_recipient_read_created` ON `(recipient_id, is_read, created_at DESC)` (알림 목록 조회 최적화)

**Validation Rules**:
- `type`: 'COMMENT' (Phase 1), 향후 'LIKE', 'FOLLOW' 등 추가 가능

**Relationships**:
- `recipient`: Notification → User (N:1)
- `actor`: Notification → User (N:1)
- `post`: Notification → Post (N:1)
- `comment`: Notification → Comment (N:1)

**Business Logic**:
- 댓글 생성 시 → 포스트 작성자에게 'COMMENT' 알림 생성
- 자기 자신의 포스트에 댓글 작성 시 → 알림 생성하지 않음
- FCM 푸시: `is_read = false` 알림에 대해 FCM 메시지 전송

---

## Prisma Schema

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id               String   @id @default(uuid())
  email            String   @unique
  passwordHash     String   @map("password_hash")
  username         String   @unique
  displayName      String   @map("display_name")
  profileImageUrl  String?  @map("profile_image_url")
  bio              String?
  fcmToken         String?  @map("fcm_token")
  createdAt        DateTime @default(now()) @map("created_at")
  updatedAt        DateTime @updatedAt @map("updated_at")

  posts                  Post[]
  comments               Comment[]
  notificationsReceived  Notification[] @relation("NotificationRecipient")
  notificationsSent      Notification[] @relation("NotificationActor")

  @@index([email])
  @@index([username])
  @@map("users")
}

model Post {
  id        String   @id @default(uuid())
  authorId  String   @map("author_id")
  imageUrl  String   @map("image_url")
  caption   String?
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  author        User           @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments      Comment[]
  notifications Notification[]

  @@index([createdAt(sort: Desc)])
  @@index([authorId])
  @@map("posts")
}

model Comment {
  id        String   @id @default(uuid())
  postId    String   @map("post_id")
  authorId  String   @map("author_id")
  content   String
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  post          Post           @relation(fields: [postId], references: [id], onDelete: Cascade)
  author        User           @relation(fields: [authorId], references: [id], onDelete: Cascade)
  notifications Notification[]

  @@index([postId, createdAt])
  @@index([authorId])
  @@map("comments")
}

enum NotificationType {
  COMMENT
}

model Notification {
  id          String           @id @default(uuid())
  recipientId String           @map("recipient_id")
  actorId     String           @map("actor_id")
  type        NotificationType
  postId      String?          @map("post_id")
  commentId   String?          @map("comment_id")
  isRead      Boolean          @default(false) @map("is_read")
  createdAt   DateTime         @default(now()) @map("created_at")

  recipient User     @relation("NotificationRecipient", fields: [recipientId], references: [id], onDelete: Cascade)
  actor     User     @relation("NotificationActor", fields: [actorId], references: [id], onDelete: SetNull)
  post      Post?    @relation(fields: [postId], references: [id], onDelete: Cascade)
  comment   Comment? @relation(fields: [commentId], references: [id], onDelete: Cascade)

  @@index([recipientId, isRead, createdAt(sort: Desc)])
  @@map("notifications")
}
```

---

## Migrations

### Initial Migration

```sql
-- CreateExtension (if not exists)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- CreateEnum
CREATE TYPE "NotificationType" AS ENUM ('COMMENT');

-- CreateTable: users
CREATE TABLE "users" (
    "id" UUID NOT NULL DEFAULT uuid_generate_v4(),
    "email" VARCHAR(255) NOT NULL,
    "password_hash" VARCHAR(255) NOT NULL,
    "username" VARCHAR(50) NOT NULL,
    "display_name" VARCHAR(100) NOT NULL,
    "profile_image_url" TEXT,
    "bio" TEXT,
    "fcm_token" TEXT,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "updated_at" TIMESTAMP NOT NULL DEFAULT NOW(),

    CONSTRAINT "users_pkey" PRIMARY KEY ("id")
);

-- CreateTable: posts
CREATE TABLE "posts" (
    "id" UUID NOT NULL DEFAULT uuid_generate_v4(),
    "author_id" UUID NOT NULL,
    "image_url" TEXT NOT NULL,
    "caption" TEXT,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "updated_at" TIMESTAMP NOT NULL DEFAULT NOW(),

    CONSTRAINT "posts_pkey" PRIMARY KEY ("id")
);

-- CreateTable: comments
CREATE TABLE "comments" (
    "id" UUID NOT NULL DEFAULT uuid_generate_v4(),
    "post_id" UUID NOT NULL,
    "author_id" UUID NOT NULL,
    "content" TEXT NOT NULL,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "updated_at" TIMESTAMP NOT NULL DEFAULT NOW(),

    CONSTRAINT "comments_pkey" PRIMARY KEY ("id")
);

-- CreateTable: notifications
CREATE TABLE "notifications" (
    "id" UUID NOT NULL DEFAULT uuid_generate_v4(),
    "recipient_id" UUID NOT NULL,
    "actor_id" UUID NOT NULL,
    "type" "NotificationType" NOT NULL,
    "post_id" UUID,
    "comment_id" UUID,
    "is_read" BOOLEAN NOT NULL DEFAULT false,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),

    CONSTRAINT "notifications_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "users_email_key" ON "users"("email");
CREATE UNIQUE INDEX "users_username_key" ON "users"("username");
CREATE INDEX "idx_users_email" ON "users"("email");
CREATE INDEX "idx_users_username" ON "users"("username");

CREATE INDEX "idx_posts_created_at" ON "posts"("created_at" DESC);
CREATE INDEX "idx_posts_author_id" ON "posts"("author_id");

CREATE INDEX "idx_comments_post_id_created_at" ON "comments"("post_id", "created_at");
CREATE INDEX "idx_comments_author_id" ON "comments"("author_id");

CREATE INDEX "idx_notifications_recipient_read_created" ON "notifications"("recipient_id", "is_read", "created_at" DESC);

-- AddForeignKey
ALTER TABLE "posts" ADD CONSTRAINT "posts_author_id_fkey" FOREIGN KEY ("author_id") REFERENCES "users"("id") ON DELETE CASCADE ON UPDATE CASCADE;

ALTER TABLE "comments" ADD CONSTRAINT "comments_post_id_fkey" FOREIGN KEY ("post_id") REFERENCES "posts"("id") ON DELETE CASCADE ON UPDATE CASCADE;
ALTER TABLE "comments" ADD CONSTRAINT "comments_author_id_fkey" FOREIGN KEY ("author_id") REFERENCES "users"("id") ON DELETE CASCADE ON UPDATE CASCADE;

ALTER TABLE "notifications" ADD CONSTRAINT "notifications_recipient_id_fkey" FOREIGN KEY ("recipient_id") REFERENCES "users"("id") ON DELETE CASCADE ON UPDATE CASCADE;
ALTER TABLE "notifications" ADD CONSTRAINT "notifications_actor_id_fkey" FOREIGN KEY ("actor_id") REFERENCES "users"("id") ON DELETE SET NULL ON UPDATE CASCADE;
ALTER TABLE "notifications" ADD CONSTRAINT "notifications_post_id_fkey" FOREIGN KEY ("post_id") REFERENCES "posts"("id") ON DELETE CASCADE ON UPDATE CASCADE;
ALTER TABLE "notifications" ADD CONSTRAINT "notifications_comment_id_fkey" FOREIGN KEY ("comment_id") REFERENCES "comments"("id") ON DELETE CASCADE ON UPDATE CASCADE;
```

---

## Sample Data

### User

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "password_hash": "$2b$10$...",
  "username": "john_doe",
  "display_name": "John Doe",
  "profile_image_url": "https://firebasestorage.googleapis.com/.../profile.jpg",
  "bio": "사진 찍는 것을 좋아합니다 📸",
  "fcm_token": "fY3x...",
  "created_at": "2025-01-01T00:00:00Z",
  "updated_at": "2025-01-01T00:00:00Z"
}
```

### Post

```json
{
  "id": "660e8400-e29b-41d4-a716-446655440001",
  "author_id": "550e8400-e29b-41d4-a716-446655440000",
  "image_url": "https://firebasestorage.googleapis.com/.../image.jpg",
  "caption": "오늘의 일몰 🌅",
  "created_at": "2025-01-02T12:00:00Z",
  "updated_at": "2025-01-02T12:00:00Z"
}
```

### Comment

```json
{
  "id": "770e8400-e29b-41d4-a716-446655440002",
  "post_id": "660e8400-e29b-41d4-a716-446655440001",
  "author_id": "880e8400-e29b-41d4-a716-446655440003",
  "content": "정말 멋져요! 👍",
  "created_at": "2025-01-02T12:10:00Z",
  "updated_at": "2025-01-02T12:10:00Z"
}
```

### Notification

```json
{
  "id": "990e8400-e29b-41d4-a716-446655440004",
  "recipient_id": "550e8400-e29b-41d4-a716-446655440000",
  "actor_id": "880e8400-e29b-41d4-a716-446655440003",
  "type": "COMMENT",
  "post_id": "660e8400-e29b-41d4-a716-446655440001",
  "comment_id": "770e8400-e29b-41d4-a716-446655440002",
  "is_read": false,
  "created_at": "2025-01-02T12:10:01Z"
}
```

---

## Query Patterns

### 1. 피드 조회 (최신순, 페이지네이션)

```sql
SELECT
  p.id, p.image_url, p.caption, p.created_at,
  u.username, u.display_name, u.profile_image_url,
  COUNT(c.id) AS comment_count
FROM posts p
INNER JOIN users u ON p.author_id = u.id
LEFT JOIN comments c ON c.post_id = p.id
WHERE p.created_at < :cursor  -- cursor-based pagination
GROUP BY p.id, u.id
ORDER BY p.created_at DESC
LIMIT 20;
```

### 2. 포스트 상세 + 댓글

```sql
-- Post
SELECT
  p.id, p.image_url, p.caption, p.created_at,
  u.username, u.display_name, u.profile_image_url
FROM posts p
INNER JOIN users u ON p.author_id = u.id
WHERE p.id = :post_id;

-- Comments
SELECT
  c.id, c.content, c.created_at,
  u.username, u.display_name, u.profile_image_url
FROM comments c
INNER JOIN users u ON c.author_id = u.id
WHERE c.post_id = :post_id
ORDER BY c.created_at ASC;
```

### 3. 알림 목록 (읽지 않은 알림 우선)

```sql
SELECT
  n.id, n.type, n.is_read, n.created_at,
  actor.username AS actor_username,
  actor.display_name AS actor_display_name,
  p.id AS post_id, p.image_url AS post_image_url
FROM notifications n
INNER JOIN users actor ON n.actor_id = actor.id
LEFT JOIN posts p ON n.post_id = p.id
WHERE n.recipient_id = :user_id
ORDER BY n.is_read ASC, n.created_at DESC
LIMIT 50;
```

---

## Performance Considerations

1. **Indexes**: 모든 주요 쿼리에 대해 인덱스 생성 (피드 정렬, 댓글 조회, 알림 조회)
2. **Pagination**: Cursor-based pagination (Offset 대신 `created_at` 기반)
3. **Count Optimization**: 댓글 수는 JOIN + GROUP BY 대신 별도 count 쿼리 또는 캐싱 고려
4. **Connection Pooling**: Prisma connection pooling (Vercel Postgres 기본 지원)
5. **N+1 Problem**: Prisma `include` 사용 시 주의, 필요 시 raw query

---

## Next Steps

1. **Prisma Setup**: `prisma init`, schema 작성, migration 생성
2. **Seed Data**: 테스트용 샘플 데이터 생성 (`prisma/seed.ts`)
3. **Type Generation**: `prisma generate` → TypeScript 타입 자동 생성
4. **API Implementation**: data-model 기반 API Routes 구현 ([contracts/](./contracts/) 참조)
