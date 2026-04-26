# Trust-First Offline Car Rental Platform with AI Assistance

> A production-grade, full-stack car rental platform designed for trust, transparency, and offline-first operations.

## 🎯 Core Value Proposition

This platform eliminates rental disputes through:
- **Verifiable Evidence**: Timestamped, geo-tagged images
- **Offline Reliability**: Full functionality without internet
- **Trust-First Design**: AI-assisted transparency
- **Safety Focus**: Built-in features for vulnerable users

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   CLIENT LAYER (PWA)                      │
│  Next.js 15 + React + Tailwind CSS + IndexedDB             │
│  - Offline-first sync engine                               │
│  - Service Worker for background sync                      │
└──────────────────────────────┬──────────────────────────────┘
                   │ HTTPS/REST
┌──────────────────────────────┴──────────────────────────────┐
│                   API GATEWAY LAYER                         │
│  - Rate Limiting (Redis)                                   │
│  - Request Validation                                      │
│  - Error Handling                                          │
└──────────────────────────────┬──────────────────────────────┘
                   │
┌──────────────────────────────┴──────────────────────────────┐
│              APPLICATION LAYER (Express)                    │
│  ┌──────────────┬──────────────┬──────────────┐             │
│  │ Auth         │ Booking      │ Pricing      │             │
│  │ (JWT)        │ Service      │ Engine       │             │
│  └──────────────┴──────────────┴──────────────┘             │
│  ┌──────────────┬──────────────┬──────────────┐             │
│  │ AI Service   │ Sync Engine  │ Notification │             │
│  │ (Context     │ (Conflict    │ Service      │             │
│  │ Builder)     │ Resolution)  │ (BullMQ)     │             │
│  └──────────────┴──────────────┴──────────────┘             │
└──────────────────────────┬──────────────────────────────────┘
        ┌──────────────────┼──────────────────┬──────────────┐
        │          │          │          │
┌───────┴───┐ ┌────┴─────┐ ┌──┴──────┐ ┌──┴──────┐
│PostgreSQL │ │Redis     │ │BullMQ   │ │AWS S3   │
│(PostGIS)  │ │Cache     │ │Queue    │ │Images   │
└───────────┘ └──────────┘ └─────────┘ └─────────┘
```

### Key Architectural Decisions

1. **Offline-First with Sync**
   - IndexedDB for local persistence
   - Service Worker for background sync
   - Conflict resolution via version vectors
   - Queue-based retry logic

2. **AI as Context Provider (Not Decision Maker)**
   - LLM provides guidance, not decisions
   - All critical decisions remain with humans
   - Structured prompts for consistency
   - Confidence scoring for outputs

3. **Trust Through Immutability**
   - Damage reports are append-only
   - Image metadata stored separately
   - Audit trail for all bookings
   - Cryptographic signatures for disputes

4. **Scalability Patterns**
   - Database indexing on geolocation
   - Redis caching for pricing/availability
   - Queue-based processing (notifications)
   - CDN-ready image storage

---

## 📋 Feature Breakdown

### 1. Trust & Damage Accountability
- **Pre/Post Trip Images**: Geo-tagged, timestamped
- **Damage Timeline**: Chronological tracking
- **Dispute Detection**: Automatic flagging for missing images, time inconsistencies, GPS anomalies

### 2. Offline-First System
- **Local Storage**: IndexedDB for trips, bookings, images
- **Sync Engine**: Smart conflict resolution
- **Background Sync**: Service Worker integration

### 3. Safety Features
- **KYC Verification**: Email + ID simulation
- **Emergency Button**: In-trip alert mechanism
- **Live Trip Tracking**: Real-time location sharing
- **Safety Ratings**: User and car safety scores

### 4. Dynamic Pricing
- **Formula-Based**: Demand + time + location
- **Cron Updates**: Periodic recalculation
- **Smart Caching**: 15-min validity with Redis

### 5. AI Assistant
- **Damage Guidance**: Step-by-step image capture instructions
- **Dispute Explanation**: Neutral, data-driven explanations
- **Offline Reassurance**: Guides users about sync behavior

---

## 🚀 Getting Started

### Prerequisites
```bash
# Node.js 18+
# PostgreSQL 14+
# Redis 7+
# Git
```

### Quick Setup

```bash
# Clone repository
git clone https://github.com/aditeejha/car-rental-service.git
cd car-rental-service

# Install dependencies
npm install

# Setup environment variables
cp .env.example .env
# Edit .env with your configuration

# Setup database
npm run db:migrate
npm run db:seed

# Start development servers
npm run dev
```

Access:
- Frontend: http://localhost:3000
- Backend: http://localhost:5000
- API Docs: http://localhost:5000/api/docs

---

## 📊 Project Structure

```
car-rental-service/
├── backend/                    # Express.js server
│   ├── src/
│   │   ├── routes/            # API routes
│   │   ├── services/          # Business logic
│   │   ├── middleware/        # Express middleware
│   │   ├── database/          # DB pool, migrations
│   │   ├── queue/             # BullMQ workers
│   │   ├── cache/             # Redis helpers
│   │   ├── ai/                # AI integration
│   │   ├── utils/             # Helpers
│   │   └── index.ts           # Entry point
│   ├── package.json
│   └── tsconfig.json
│
├── frontend/                   # Next.js app
│   ├── app/
│   │   ├── (auth)/            # Auth pages
│   │   ├── (dashboard)/       # Protected pages
│   │   ├── (booking)/         # Booking flow
│   │   └── layout.tsx         # Root layout
│   ├── components/            # React components
│   ├── lib/                   # Utilities
│   ├── hooks/                 # Custom hooks
│   ├── public/                # Static assets
│   └── package.json
│
├── database/                   # SQL migrations
│   └── migrations/
│
├── docs/                       # Documentation
│   ├── ARCHITECTURE.md        # Deep architecture
│   ├── API.md                 # API reference
│   └── DEPLOYMENT.md          # Deployment guide
│
├── .env.example
├── package.json               # Root workspace
└── README.md
```

---

## 🔧 Development

### Running Tests
```bash
# Unit & integration tests
npm run test

# E2E tests
npm run test:e2e

# Watch mode
npm run test:watch
```

### Database Operations
```bash
# Create migration
npm run db:migrate:create -- --name add_user_table

# Run migrations
npm run db:migrate

# Rollback
npm run db:rollback

# Reset database
npm run db:reset

# Seed with test data
npm run db:seed
```

### Code Quality
```bash
# Lint code
npm run lint

# Format code
npm run format

# Type checking
npm run type-check
```

---

## 🗄️ Database Schema

Key tables:
- **users**: Auth & profiles
- **cars**: Rental inventory
- **bookings**: Rental transactions
- **trips**: Active/completed journeys
- **damage_reports**: Pre/post images
- **disputes**: Conflict tracking
- **wallets**: Credit management
- **notifications**: Event queue
- **audit_logs**: State changes

Full schema in `database/migrations/`

---

## 🤖 AI System

### Architecture
```
AI Request → Context Builder → Prompt Generator → LLM API → Response Formatter
```

### Use Cases
1. **Damage Guidance**: "Capture all 4 angles before departure"
2. **Dispute Explanation**: "Previous renter reported dent on left side"
3. **Offline Reassurance**: "Your images queued. Will sync when online"

### Sample API
```bash
POST /api/ai/assist
{
  "type": "damage_guidance",
  "trip_id": "...",
  "context": {...}
}

Response:
{
  "message": "Please capture images of all 4 sides...",
  "confidence": 0.95,
  "next_step": "upload_images"
}
```

---

## 🔐 Security

- **Authentication**: JWT with 24h expiry + refresh tokens
- **Authorization**: Role-Based Access Control (RBAC)
- **Data Protection**: TLS/HTTPS, encryption at rest
- **Rate Limiting**: 100 req/min per user
- **Image Validation**: EXIF verification, GPS checks

---

## 📈 Performance

### Caching Strategy
- **Pricing**: 15 minutes (Redis)
- **Car listings**: 5 minutes
- **User ratings**: 1 hour
- **Images**: 30 days (CloudFront CDN)

### Database Optimization
- Indexes on frequently queried columns
- Materialized views for analytics
- Connection pooling (50 connections)
- Query optimization with EXPLAIN ANALYZE

---

## 📱 PWA & Offline

### Service Worker
- Offline image capture (queued for sync)
- Cached API responses
- Push notifications
- Background sync

### IndexedDB Schema
```javascript
{
  trips: { keyPath: "id" },
  pending_images: { keyPath: "id" },
  sync_queue: { keyPath: "id" },
  local_cache: { keyPath: "url" }
}
```

---

## 🚢 Deployment

### Docker
```bash
docker build -t car-rental:latest .
docker run -p 5000:5000 --env-file .env car-rental:latest
```

### Environment
- **Frontend**: Vercel (automatic)
- **Backend**: Docker/Kubernetes
- **Database**: AWS RDS + PostGIS
- **Cache**: Redis Cloud
- **Images**: AWS S3 + CloudFront

---

## 📚 Documentation

- **ARCHITECTURE.md**: Deep system design
- **API.md**: Complete API reference
- **DEPLOYMENT.md**: Production deployment
- **CONTRIBUTING.md**: Development guidelines

---

## 📄 License

MIT

---

## 👨‍💻 Author

Built by [aditeejha](https://github.com/aditeejha)
