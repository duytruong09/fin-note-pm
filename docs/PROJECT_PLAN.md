Dưới đây là nội dung file **`PROJECT_PLAN.md`** đặt ở root project `fin-note/`.
Mục tiêu: để Claude hiểu đầy đủ kiến trúc, định hướng dài hạn, tiêu chuẩn code và cách tổ chức project.

---

# fin-note

Personal Finance Note App (Voice-powered Expense Tracking)

---

# 1. Vision

**fin-note** là ứng dụng ghi chép chi tiêu cá nhân:

- Ghi nhanh bằng text
- Ghi bằng giọng nói → AI phân tích → lưu structured data
- Thống kê, báo cáo, phân loại tự động
- Thiết kế cloud-native, scale được

Tech stack:

- Backend: NestJS + PostgreSQL
- Mobile App: React Native
- AI/NLP: LLM API (Claude/OpenAI) + optional local rules
- Infra: Docker + Kubernetes-ready

---

# 2. High-Level Architecture

```
React Native App
        |
        v
NestJS API (REST)
        |
        +-- PostgreSQL
        +-- Redis (cache, rate limit)
        +-- Object Storage (voice file nếu cần)
        +-- AI Service (LLM API)
```

---

# 3. Core Features

## 3.1 Authentication

- Email/password
- JWT access + refresh token
- Optional OAuth (Google/Apple)

## 3.2 Expense Management

- Create expense
- Update expense
- Delete expense
- List expense (pagination + filter)
- Category management
- Monthly report
- Statistics

## 3.3 Voice Input (Special Feature)

User nói:

> "Hôm nay ăn cơm 10 nghìn"

Flow:

1. FE record audio
2. Upload to BE
3. BE:
   - Speech-to-text (STT)
   - Send text to LLM
   - Parse structured output

4. Validate data
5. Persist DB

Expected AI Output:

```json
{
  "amount": 10000,
  "currency": "VND",
  "category": "food",
  "description": "Ăn cơm",
  "date": "2026-02-13"
}
```

---

# 4. Backend Structure (fin-note-be)

Path:

```
/Users/piggi/Documents/LEARNING/fin-note/fin-note-be
```

## 4.1 Folder Structure

```
fin-note-be/
│
├── src/
│   ├── modules/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── expenses/
│   │   ├── categories/
│   │   ├── voice/
│   │   ├── reports/
│   │
│   ├── common/
│   │   ├── guards/
│   │   ├── filters/
│   │   ├── interceptors/
│   │   ├── decorators/
│   │
│   ├── config/
│   ├── database/
│   └── main.ts
│
├── prisma or typeorm/
├── test/
├── .claude/
├── .prompt/
└── CLAUDE.md
```

---

# 5. Database Design (PostgreSQL)

## users

```
id (uuid)
email
password_hash
created_at
updated_at
```

## categories

```
id
user_id (nullable for system default)
name
type (income | expense)
```

## expenses

```
id
user_id
category_id
amount (numeric)
currency
description
expense_date
source (manual | voice)
created_at
```

## voice_logs

```
id
user_id
raw_text
ai_result (jsonb)
status (parsed | failed)
created_at
```

Indexes:

- idx_expenses_user_date
- idx_expenses_category
- idx_voice_user

---

# 6. Voice Processing Design

## 6.1 STT Strategy

Option A:

- Client STT (cheaper)
- Send text only

Option B:

- Server STT (centralized control)

Recommended: Client STT → send text

## 6.2 LLM Prompt Design

LLM must return STRICT JSON.

System prompt example:

```
You are a financial parser.
Extract structured expense data from Vietnamese sentence.
Return JSON only.
```

Add validation layer:

- amount > 0
- category fallback = "other"
- date default = today

---

# 7. Frontend Structure (fin-note-app)

Path:

```
/Users/piggi/Documents/LEARNING/fin-note/fin-note-app
```

## Folder Structure

```
fin-note-app/
│
├── src/
│   ├── screens/
│   │   ├── Home
│   │   ├── AddExpense
│   │   ├── Reports
│   │   ├── Settings
│   │
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── store/
│   └── navigation/
│
├── .claude/
├── .prompt/
└── CLAUDE.md
```

State Management:

- Zustand hoặc Redux Toolkit

Networking:

- Axios + interceptor JWT refresh

---

# 8. .claude Directory Strategy

Root:

```
fin-note/
│
├── .claude/
│   ├── roles/
│   │   ├── backend-architect.md
│   │   ├── mobile-architect.md
│   │   ├── db-expert.md
│   │   ├── security-reviewer.md
│   │   └── performance-reviewer.md
│   │
│   ├── standards/
│   │   ├── coding-guidelines.md
│   │   ├── api-design.md
│   │   ├── error-handling.md
│   │   └── testing-strategy.md
│   │
│   └── roadmap.md
```

Purpose:

- Mỗi role xử lý task chuyên biệt
- Claude có thể assume role khi generate code

---

# 9. .prompt Directory Strategy

```
.prompt/
│
├── voice-parser.prompt.md
├── expense-crud.prompt.md
├── report-generation.prompt.md
├── migration.prompt.md
└── validation.prompt.md
```

Each prompt file:

- Context
- Expected input
- Output format
- Constraints
- Edge cases

---

# 10. CLAUDE.md (Root Instruction File)

Chứa:

- Tech stack
- Coding standards
- Folder rules
- Naming convention
- DB rules
- Security rules
- Performance rules

---

# 11. Long-Term Roadmap

## Phase 1 – MVP

- Auth
- CRUD expense
- Basic voice parsing
- Monthly report

## Phase 2 – AI Smart Features

- Auto category suggestion
- Spending pattern detection
- Weekly insight summary

## Phase 3 – Scale

- Multi-device sync
- Multi-currency
- Shared wallet
- Export CSV
- Background processing queue

## Phase 4 – Enterprise Ready

- Event-driven architecture
- CQRS for reporting
- Read replica DB
- Caching strategy
- Rate limit

---

# 12. Security Requirements

- bcrypt password hash
- JWT short-lived
- Refresh token rotation
- Input validation (class-validator)
- Rate limit voice endpoint
- Prevent prompt injection (strip instructions)

---

# 13. Performance Strategy

- Pagination always
- Index on user_id + date
- JSONB for AI result
- Avoid N+1 query
- Use batch insert if import

---

# 14. Testing Strategy

- Unit test service layer
- E2E test auth + expense
- Mock AI layer
- Seed test DB

---

# 15. Future Improvements

- Offline mode
- Background sync
- ML model fine-tuning
- On-device parsing fallback
- Analytics dashboard
