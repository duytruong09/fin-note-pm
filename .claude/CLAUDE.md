# FIN-NOTE: Project Conventions & Guidelines

## Project Overview

Fin-Note is a voice-enabled expense tracking application. Users can speak naturally ("Hôm nay ăn cơm 10 nghìn") and AI automatically creates structured transactions.

**Tech Stack:**

- Backend: NestJS + TypeScript + PostgreSQL + Prisma ORM
- Frontend: React Native + Expo + TypeScript
- AI: OpenAI Whisper (speech-to-text) + GPT-4o-mini (parsing)

---

## Code Conventions

### TypeScript

- **Always use TypeScript** with strict mode enabled
- Prefer `interface` over `type` for object shapes
- Use explicit return types for all functions
- No `any` types - use `unknown` and type guards instead

### Naming Conventions

**Files:**

- Components: PascalCase (e.g., `VoiceRecorder.tsx`)
- Services: kebab-case with suffix (e.g., `whisper.service.ts`)
- DTOs: kebab-case with suffix (e.g., `create-transaction.dto.ts`)
- Entities: kebab-case with suffix (e.g., `transaction.entity.ts`)
- Hooks: camelCase with `use` prefix (e.g., `useVoiceRecorder.ts`)

**Code:**

- Variables/Functions: camelCase (e.g., `parseExpense`, `audioFile`)
- Classes/Components: PascalCase (e.g., `WhisperService`, `VoiceRecorder`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_AUDIO_DURATION`)
- Enums: PascalCase (e.g., `TransactionType`)

### Backend (NestJS)

**Project Structure:**

```
src/
├── main.ts                       # Application entry point
├── app.module.ts                 # Root module
│
├── config/                       # Configuration files
│   ├── configuration.ts          # App configuration
│   └── env.validation.ts         # Environment validation schema
│
├── common/                       # Shared utilities
│   ├── decorators/               # Custom decorators (@CurrentUser, etc.)
│   ├── guards/                   # Auth guards, role guards
│   ├── filters/                  # Exception filters
│   ├── interceptors/             # Logging, transform interceptors
│   ├── pipes/                    # Validation pipes
│   └── constants/                # App-wide constants
│
├── infrastructure/               # Infrastructure layer
│   ├── database/
│   │   ├── prisma.service.ts     # Prisma service
│   │   ├── prisma.module.ts      # Prisma module
│   │   └── schema.prisma         # Prisma schema (or in root)
│   ├── ai/
│   │   ├── ai.service.ts         # OpenAI client wrapper
│   │   ├── ai.module.ts
│   │   └── prompts/              # AI prompt templates
│   └── cache/
│       ├── redis.service.ts
│       └── cache.module.ts
│
├── modules/                      # Feature modules
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.module.ts
│   │   ├── dto/
│   │   └── strategies/           # JWT, Refresh token strategies
│   ├── users/
│   ├── transactions/
│   ├── categories/
│   ├── budgets/
│   └── voice/                    # Voice processing module
│       ├── voice.controller.ts
│       ├── voice.service.ts
│       ├── voice.module.ts
│       ├── dto/
│       └── services/
│           ├── whisper.service.ts
│           ├── gpt-parser.service.ts
│           └── voice-storage.service.ts
│
└── shared/                       # Shared code
    ├── types/                    # TypeScript types/interfaces
    ├── utils/                    # Utility functions
    └── helpers/                  # Helper functions
```

**Module Structure (inside modules/):**

```
<feature>/
├── <feature>.controller.ts       # HTTP endpoints
├── <feature>.service.ts          # Business logic
├── <feature>.module.ts           # Module definition
├── dto/                          # Data Transfer Objects
│   ├── create-<feature>.dto.ts
│   ├── update-<feature>.dto.ts
│   └── <feature>-response.dto.ts
└── services/                     # Additional services (optional)
    └── <specialized>.service.ts
```

**Best Practices:**

- Use dependency injection for all services
- Validate all inputs with `class-validator` DTOs
- Use proper HTTP status codes and exceptions
- Implement proper logging with `@nestjs/logger`
- Always use transactions for multi-step database operations
- Keep controllers thin - business logic goes in services

**Error Handling:**

```typescript
// Use NestJS built-in exceptions
throw new BadRequestException("Invalid audio format");
throw new UnauthorizedException("Invalid token");
throw new NotFoundException("Transaction not found");

// Custom exceptions for specific cases
throw new VoiceProcessingException("Whisper API failed", originalError);
```

### Frontend (React Native)

**Component Structure:**

```typescript
// Use functional components with hooks
interface VoiceRecorderProps {
  onRecordingComplete: (uri: string) => void;
  language: 'vi' | 'en';
}

export const VoiceRecorder: React.FC<VoiceRecorderProps> = ({
  onRecordingComplete,
  language,
}) => {
  // Hooks first
  const { isRecording, startRecording, stopRecording } = useVoiceRecorder();
  const [error, setError] = useState<string | null>(null);

  // Event handlers
  const handlePress = async () => {
    try {
      const uri = await startRecording();
      onRecordingComplete(uri);
    } catch (err) {
      setError(err.message);
    }
  };

  // Render
  return (
    <View>
      {/* UI */}
    </View>
  );
};
```

**State Management (Zustand):**

```typescript
// Use slices for different domains
interface VoiceState {
  isProcessing: boolean;
  parsedExpense: ParsedExpense | null;
  error: string | null;

  // Actions
  processVoice: (audioUri: string) => Promise<void>;
  clearError: () => void;
}

export const useVoiceStore = create<VoiceState>((set, get) => ({
  // ... implementation
}));
```

**Hooks:**

- Extract reusable logic into custom hooks
- Prefix all hooks with `use`
- Keep hooks focused on single responsibility

### Database (PostgreSQL + Prisma)

**Prisma Schema Conventions:**

- Use `snake_case` for database column names
- Use `camelCase` for Prisma model field names with `@map()`
- Always include `createdAt` and `updatedAt` timestamps
- Use soft deletes with `deletedAt` for important data
- Define proper indexes with `@@index()`
- Use enums for fixed value sets

**Example Model (prisma/schema.prisma):**

```prisma
model Transaction {
  id          String   @id @default(uuid())
  userId      String   @map("user_id")
  categoryId  String?  @map("category_id")

  amount      Decimal  @db.Decimal(15, 2)
  currency    String   @default("VND") @db.VarChar(3)
  type        TransactionType @default(EXPENSE)

  description String?  @db.Text
  transactionDate DateTime @default(now()) @map("transaction_date")

  // Voice-specific fields
  isVoiceCreated Boolean @default(false) @map("is_voice_created")
  voiceAudioUrl  String? @map("voice_audio_url")
  voiceTranscript String? @map("voice_transcript") @db.Text
  voiceParsingConfidence Decimal? @map("voice_parsing_confidence") @db.Decimal(3, 2)

  // Timestamps
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  deletedAt DateTime? @map("deleted_at")

  // Relations
  user     User      @relation(fields: [userId], references: [id])
  category Category? @relation(fields: [categoryId], references: [id])

  @@index([userId])
  @@index([transactionDate])
  @@index([categoryId])
  @@index([isVoiceCreated])
  @@map("transactions")
}

enum TransactionType {
  INCOME
  EXPENSE
}
```

**Migrations:**

```bash
# Create migration
npx prisma migrate dev --name create_transactions_table

# Apply migrations
npx prisma migrate deploy

# Generate Prisma Client
npx prisma generate

# Reset database (dev only)
npx prisma migrate reset
```

**Using Prisma in Services:**

```typescript
import { PrismaService } from "@/infrastructure/database/prisma.service";

@Injectable()
export class TransactionsService {
  constructor(private prisma: PrismaService) {}

  async create(userId: string, dto: CreateTransactionDto) {
    return this.prisma.transaction.create({
      data: {
        ...dto,
        userId,
      },
      include: {
        category: true,
      },
    });
  }

  async findAll(userId: string, filters: TransactionFilters) {
    return this.prisma.transaction.findMany({
      where: {
        userId,
        deletedAt: null,
        ...filters,
      },
      include: {
        category: true,
      },
      orderBy: {
        transactionDate: "desc",
      },
    });
  }
}
```

---

## API Design

### RESTful Conventions

- Use plural nouns for resources: `/api/v1/transactions`, `/api/v1/categories`
- Use proper HTTP verbs: GET (read), POST (create), PATCH (partial update), DELETE (delete)
- Version your API: `/api/v1/`
- Use nested routes for relationships: `/api/v1/budgets/:id/status`

### Response Format

**Success:**

```json
{
  "data": { ... },
  "meta": {
    "timestamp": "2026-02-13T10:00:00Z"
  }
}
```

**Pagination:**

```json
{
  "data": [...],
  "meta": {
    "page": 1,
    "perPage": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

**Error:**

```json
{
  "error": {
    "code": "INVALID_AUDIO_FORMAT",
    "message": "Audio file must be in m4a or wav format",
    "details": {
      "receivedFormat": "mp3",
      "allowedFormats": ["m4a", "wav"]
    }
  },
  "meta": {
    "timestamp": "2026-02-13T10:00:00Z"
  }
}
```

---

## Voice Processing

### Prompt Management

- All AI prompts stored in `.prompt/` directory
- Use separate files for Vietnamese and English
- Version prompts: `expense-parser-vi-v1.txt`, `expense-parser-vi-v2.txt`
- Document prompt changes in `.prompt/CHANGELOG.md`
- Include few-shot examples in prompts

### Error Handling

**Voice Pipeline Stages:**

1. Audio upload validation (size, format)
2. Whisper transcription (catch API errors)
3. GPT parsing (validate structured output)
4. User review (allow corrections)
5. Transaction creation

**Fallback Strategy:**

- If Whisper fails: Ask user to try again or type manually
- If GPT parsing has low confidence (<0.7): Show warning, allow editing
- If parsing fails completely: Fall back to manual entry with pre-filled transcript

### Logging

Log all voice processing attempts to `voice_processing_logs` table for:

- Debugging failed parses
- Improving prompts with real data
- Analytics on accuracy
- Cost tracking

---

## Security

### Authentication

- Use JWT with access tokens (15min) + refresh tokens (7 days)
- Store tokens securely (iOS Keychain / Android Keystore)
- Implement token rotation
- Never log tokens or sensitive data

### Data Protection

- Hash passwords with bcrypt (min 10 rounds)
- Validate all user inputs
- Sanitize outputs to prevent XSS
- Use parameterized queries (TypeORM handles this)
- Implement rate limiting on sensitive endpoints
- Delete audio files after processing (unless user opts to keep)

### API Keys

- Store OpenAI API key in environment variables
- Never commit `.env` files
- Use different keys for dev/staging/prod
- Monitor API usage to detect abuse

---

## Testing

### Backend Testing

**Unit Tests:**

- Test all services in isolation
- Mock external dependencies (Whisper, GPT, database)
- Aim for 80%+ coverage
- Run on every commit

**Integration Tests:**

- Test API endpoints with real database (test DB)
- Test voice pipeline with mocked OpenAI APIs
- Test authentication flow end-to-end

### Frontend Testing

**Component Tests:**

- Test rendering with different props
- Test user interactions
- Mock API calls

**E2E Tests:**

- Test critical flows (voice input → transaction creation)
- Use Detox or similar framework
- Run before releases

---

## Git Workflow

### Branch Strategy

- `main` - production-ready code
- `develop` - integration branch
- `feature/<feature-name>` - feature branches
- `fix/<bug-name>` - bug fixes
- `hotfix/<critical-fix>` - production hotfixes

### Commit Messages

Follow conventional commits:

```
feat: Add Whisper transcription service
fix: Handle empty audio files in voice controller
refactor: Extract GPT parsing logic to separate service
docs: Update API documentation for voice endpoints
test: Add unit tests for expense parser
chore: Update dependencies
```

### Pull Requests

- Use PR template (create `.github/PULL_REQUEST_TEMPLATE.md`)
- Require code review before merge
- Run CI/CD checks (tests, linting)
- Squash commits when merging to main

---

## Performance

### Backend

- Use database indexes on frequently queried fields
- Implement caching for categories, user settings
- Use connection pooling for database
- Optimize N+1 queries (use eager loading or DataLoader)
- Monitor query performance with logging

### Frontend

- Use React.memo for expensive components
- Implement virtual scrolling for long lists
- Lazy load images
- Cache API responses with TanStack Query
- Minimize re-renders with proper dependency arrays

### Voice Processing

- Compress audio before upload (reduce file size)
- Stream audio processing if possible
- Show progress indicators during processing
- Implement request timeout (30s max)

---

## Environment Variables

### Backend (.env)

```env
# Application
NODE_ENV=development
PORT=3000
API_VERSION=v1

# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=finnote_dev
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres

# JWT
JWT_ACCESS_SECRET=your-access-secret-here
JWT_REFRESH_SECRET=your-refresh-secret-here
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=7d

# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_MODEL_WHISPER=whisper-1
OPENAI_MODEL_GPT=gpt-4o-mini

# Storage
STORAGE_TYPE=local # or 's3'
STORAGE_PATH=./uploads
# S3_BUCKET=fin-note-audio
# S3_REGION=us-east-1

# Logging
LOG_LEVEL=debug
```

### Frontend (.env)

```env
EXPO_PUBLIC_API_URL=http://localhost:3000/api/v1
EXPO_PUBLIC_API_TIMEOUT=30000
EXPO_PUBLIC_SENTRY_DSN=
```

---

## Project-Specific Notes

### Voice Accuracy Optimization

1. **Collect failed parses** - Log every parse to `voice_processing_logs`
2. **Analyze patterns** - Weekly review of low-confidence parses
3. **Update prompts** - Add failing examples as few-shot training
4. **A/B test prompts** - Split traffic between prompt versions
5. **User feedback loop** - Track user corrections to learn patterns

### Vietnamese Language Support

- Whisper supports Vietnamese well
- GPT-4 handles Vietnamese but needs good prompts
- Include common Vietnamese slang in prompts
- Handle numbers with "nghìn" (thousand), "triệu" (million)
- Support both formal and informal language

### Cost Optimization

**Current OpenAI Pricing (as of 2026):**

- Whisper: ~$0.006 per minute
- GPT-4o-mini: ~$0.15 per 1M input tokens, ~$0.60 per 1M output tokens

**Strategies:**

- Keep audio recordings short (15s max recommended)
- Use gpt-4o-mini instead of gpt-4
- Cache common parsing results
- Implement free tier limits (e.g., 100 voice transactions/month)

---

## Monitoring & Observability

### Metrics to Track

**Backend:**

- API response times (p50, p95, p99)
- Voice processing success rate
- Whisper API latency
- GPT parsing accuracy
- Database query performance

**Frontend:**

- App crash rate
- Voice feature adoption rate
- Time to create transaction (voice vs manual)
- User session duration

### Logging Strategy

- Use structured logging (JSON format)
- Include correlation IDs for request tracing
- Log levels: ERROR (user-facing issues), WARN (degraded performance), INFO (important events), DEBUG (detailed flow)
- Never log sensitive data (passwords, tokens, full audio transcripts in production)

---

## Deployment

### Backend Deployment

- Use Docker for consistent environments
- Deploy to cloud (AWS, GCP, or Azure)
- Use managed PostgreSQL (RDS, Cloud SQL)
- Configure auto-scaling for API servers
- Set up health checks and monitoring

### Frontend Deployment

- Submit to App Store and Google Play
- Use EAS (Expo Application Services) for builds
- Implement OTA updates for bug fixes
- Use staged rollouts (5% → 25% → 50% → 100%)

---

## Resources

- [NestJS Documentation](https://docs.nestjs.com)
- [React Native Documentation](https://reactnative.dev)
- [Expo Documentation](https://docs.expo.dev)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [TypeORM Documentation](https://typeorm.io)

---

## Quick Start Commands

```bash
# Backend
cd fin-note-be
npm install                      # Install dependencies
docker-compose up -d             # Start PostgreSQL
npx prisma generate              # Generate Prisma Client
npx prisma migrate dev           # Run migrations
npx prisma db seed               # Seed database
npm run start:dev                # Start dev server

# Prisma Studio (Database GUI)
npx prisma studio                # Open at http://localhost:5555

# Frontend
cd fin-note-app
npm install                      # Install dependencies
npx expo start                   # Start Expo dev server
npx expo start --ios             # Run on iOS simulator
npx expo start --android         # Run on Android emulator
npx expo start --web             # Run in web browser
```

---

_Last updated: 2026-02-13_
