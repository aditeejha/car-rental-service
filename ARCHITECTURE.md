# System Architecture Deep Dive

## 1. Frontend Architecture (Next.js + PWA)

### Directory Structure
```
frontend/
├── app/
│   ├── (auth)/                 # Auth pages (login, register, reset)
│   ├── (dashboard)/            # Protected user dashboard
│   ├── (booking)/              # Booking flow (search, select, confirm)
│   ├── (trip)/                 # Trip management (active, history)
│   ├── (admin)/                # Admin dashboard
│   ├── api/                    # API routes (optional)
│   └── layout.tsx              # Root layout
├── components/
│   ├── auth/                   # Auth components
│   ├── booking/                # Booking flow steps
│   ├── car/                    # Car cards, details
│   ├── trip/                   # Trip tracking, damage report
│   ├── ai/                     # AI assistant panel
│   └── common/                 # Reusable components
├── lib/
│   ├── api.ts                  # API client with offline fallback
│   ├── sync/                   # Offline sync engine
│   ├── indexeddb/              # IndexedDB wrapper
│   └── utils/                  # Helper functions
├── hooks/
│   ├── useAuth.ts              # Auth context
│   ├── useOfflineSync.ts       # Sync engine hook
│   └── useTrip.ts              # Trip state
├── public/
│   └── sw.js                   # Service Worker
└── styles/
    └── globals.css             # Tailwind + custom styles
```

### Offline Sync Flow
```
User Action (Online)
  ↓
Optimistic Update (IndexedDB)
  ↓
API Call (immediate)
  ↓
Server Response
  ↓
Update IndexedDB + UI

User Action (Offline)
  ↓
Optimistic Update (IndexedDB)
  ↓
Queue in sync_queue (IDB)
  ↓
Service Worker monitors connection
  ↓
When online: Push queued changes
  ↓
Conflict resolution if needed
  ↓
Sync complete
```

---

## 2. Backend Architecture (Express + Modular)

### Directory Structure
```
backend/
├── src/
│   ├── routes/                 # API route handlers
│   │   ├── auth.ts             # POST /login, /register
│   │   ├── bookings.ts         # Booking CRUD
│   │   ├── cars.ts             # Car search/filter
│   │   ├── trips.ts            # Trip management
│   │   ├── ai.ts               # AI assistance
│   │   ├── sync.ts             # Offline sync
│   │   ├── pricing.ts          # Dynamic pricing
│   │   └── admin.ts            # Admin operations
│   ├── services/               # Business logic
│   │   ├── AuthService.ts
│   │   ├── BookingService.ts
│   │   ├── AIContextBuilder.ts
│   │   ├── PricingEngine.ts
│   │   ├── SyncEngine.ts
│   │   └── NotificationService.ts
│   ├── middleware/             # Express middleware
│   ├── database/               # DB pool, migrations
│   ├── queue/                  # BullMQ workers
│   ├── cache/                  # Redis helpers
│   ├── ai/                     # AI integration
│   ├── utils/                  # Helpers
│   └── index.ts                # Entry point
```

### Service Layer Pattern
```typescript
class BookingService {
  async createBooking(userId, carId, dates) {
    // 1. Validate inputs
    // 2. Acquire distributed lock
    // 3. Check availability (atomic)
    // 4. Calculate price
    // 5. Create booking (transaction)
    // 6. Publish event
  }
}
```

---

## 3. AI System Architecture

### Context Builder Pattern
```
AI Request
  ↓
Context Builder
  ├─ Trip data
  ├─ User history
  ├─ Image metadata
  └─ Safety ratings
  ↓
Prompt Generator
  ├─ System role
  ├─ Context injection
  └─ Task specification
  ↓
LLM API (OpenAI)
  ↓
Response Formatter
  ├─ Extract message
  ├─ Calculate confidence
  └─ Suggest action
```

### AI Use Cases
1. **Damage Guidance**: Step-by-step image instructions
2. **Dispute Explanation**: Data-driven explanations
3. **Offline Reassurance**: Queue status updates

---

## 4. Database Schema

### Core Tables
```sql
users              -- Auth + profiles + KYC
cars               -- Inventory
bookings           -- Rental transactions
trips              -- Active/completed journeys
damage_reports     -- Pre/post images
images             -- Image metadata + EXIF
disputes           -- Conflict tracking
wallets            -- Credit management
notifications      -- Event queue
audit_logs         -- All state changes
```

### Key Indexes
- `bookings(user_id, status, start_date)`
- `cars(location)` using PostGIS
- `trips(user_id, status, created_at)`
- `damage_reports(trip_id, report_type)`
- `images(damage_report_id, captured_at)`

---

## 5. Sync Engine

### Conflict Resolution
```typescript
conflictResolution(local, server) {
  if (local.version > server.version) return 'local';
  if (server.version > local.version) return 'server';
  if (local.hash === server.hash) return 'skip';
  return 'manual_review';
}
```

### Sync Queue
```sql
CREATE TABLE sync_queue (
  id UUID PRIMARY KEY,
  operation_type ENUM('create', 'update', 'delete'),
  resource_type VARCHAR(100),
  resource_id UUID,
  payload JSONB,
  status ENUM('pending', 'processing', 'completed', 'failed'),
  retry_count INTEGER DEFAULT 0,
  created_at TIMESTAMP
);
```

---

## 6. Pricing Engine

### Formula
```
Base Price = baseRate × duration
Demand Multiplier = 0.8 + (occupancy × 0.7)  // 0.8 to 1.5
Time Multiplier = 1.0 × (weekend: 1.3) × (peak: 1.2)
Seasonal Multiplier = varies by date
Loyalty Discount = 0 to 20%

Final Price = max(min(base × demand × time × seasonal × (1 - loyalty), max), min)
```

---

## 7. Notification Architecture

### BullMQ Queues
- **notifications**: Email, SMS, push
- **images**: Damage detection processing
- **pricing**: Dynamic pricing updates

### Retry Strategy
```
Attempt 1: Immediate
Attempt 2: +5 seconds (exponential backoff)
Attempt 3: +10 seconds
Fails after 3 attempts
```

---

## 8. Security Architecture

### JWT Strategy
```typescript
Access Token: 24 hours, stored in memory
Refresh Token: 7 days, stored in secure HTTP-only cookie
Token Rotation: Automatic on each refresh
```

### Rate Limiting
```
Window: 15 minutes
Max Requests: 100 per user
Key: user_id or IP
```

### Image Validation
- MIME type checking (JPEG, PNG only)
- File size limit (5MB)
- EXIF timestamp validation
- GPS coordinate validation
- Perceptual hash duplicate detection

---

## 9. Monitoring & Observability

### Structured Logging
```typescript
logger.info('booking_created', {
  booking_id: '123',
  user_id: '456',
  duration_ms: 234,
  final_price: 99.99
});
```

### Metrics
- Request count by endpoint
- Error rate by type
- Booking price distribution
- Active trips gauge
- Sync conflict rate

---

## 10. Deployment

### Local Development
```bash
npm install
npm run db:migrate
npm run dev
```

### Docker
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
```

### Production (AWS)
- **Frontend**: Vercel (auto-deploy)
- **Backend**: ECS + ALB
- **Database**: RDS (PostgreSQL + PostGIS)
- **Cache**: ElastiCache (Redis)
- **Storage**: S3 + CloudFront
- **Queue**: SQS (alternative to BullMQ)

---

This architecture scales from MVP to 100k+ daily rentals while maintaining trust and reliability.
