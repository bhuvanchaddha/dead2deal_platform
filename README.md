# Dead2Deal — Surplus & Dead Inventory B2B Marketplace

Dead2Deal is an enterprise B2B marketplace platform for industrial and business surplus, overstock, and dead inventory recovery. It features a React + Vite frontend and an enterprise-ready Node.js + Express + Prisma (PostgreSQL) backend with resilient in-memory fallback.

---

## 🏛️ Architecture Overview

```
dead2deal-platform/
├── server/                         # Backend API & Database Service (Phase 21)
│   ├── prisma/
│   │   ├── schema.prisma           # Prisma PostgreSQL schema (8 indexed models)
│   │   └── seed.js                 # Idempotent seed script with bcrypt hashed accounts
│   ├── src/
│   │   ├── config/
│   │   │   ├── env.js              # Zod environment variable validation
│   │   │   └── db.js               # Dual-mode database layer (PostgreSQL + resilient fallback)
│   │   ├── controllers/            # Thin API controllers
│   │   │   ├── authController.js
│   │   │   ├── healthController.js
│   │   │   ├── inventoryController.js
│   │   │   ├── productController.js
│   │   │   ├── rfqController.js
│   │   │   ├── offerController.js
│   │   │   ├── orderController.js
│   │   │   └── notificationController.js
│   │   ├── repositories/           # Unified Data Access Layer (PostgreSQL & Fallback)
│   │   │   ├── userRepository.js
│   │   │   ├── inventoryRepository.js
│   │   │   ├── rfqRepository.js
│   │   │   ├── offerRepository.js
│   │   │   ├── orderRepository.js
│   │   │   └── notificationRepository.js
│   │   ├── middleware/
│   │   │   ├── authMiddleware.js   # requireAuth & requireRole ('BUYER' | 'SELLER' | 'ADMIN')
│   │   │   └── errorMiddleware.js  # Centralized error handling
│   │   ├── routes/                 # Express API routes
│   │   ├── utils/                  # JWT and bcrypt utilities
│   │   ├── validators/             # Zod input validation schemas
│   │   ├── app.js                  # Express app configuration
│   │   └── server.js               # HTTP server entrypoint
│   ├── .env.example                # Safe server environment template (no credentials)
│   └── package.json
├── src/                            # React + Vite Frontend
│   ├── components/                 # UI design system components
│   ├── pages/                      # Buyer, Seller, Admin, and Public pages
│   ├── services/                   # Service layer interacting with backend APIs
│   └── ...
├── scratch/                        # Verification Test Suites
│   ├── test_phase18_backend.mjs    # Phase 18 auth verification (35 tests)
│   ├── test_phase19_integration.mjs# Phase 19 frontend ↔ backend integration (38 tests)
│   ├── test_phase20_marketplace.mjs# Phase 20 marketplace core APIs (48 tests)
│   └── test_phase21_postgres.mjs   # Phase 21 PostgreSQL & Prisma persistence (50 tests)
└── package.json                    # Root scripts
```

---

## 🐘 PostgreSQL Production Setup & Configuration

Dead2Deal is engineered to use **PostgreSQL** in production with Prisma ORM, while maintaining a transparent in-memory fallback for local development environments where PostgreSQL is not yet installed.

### 1. PostgreSQL Installation Requirement
To connect PostgreSQL, install PostgreSQL 14+ on your system:
- **Windows**: Download the installer from [postgresql.org](https://www.postgresql.org/download/windows/) or install via Scoop/Winget:
  ```powershell
  winget install PostgreSQL.PostgreSQL
  ```
- **Docker**: Alternatively, run a PostgreSQL container:
  ```bash
  docker run --name dead2deal-postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=dead2deal -p 5432:5432 -d postgres:16
  ```

### 2. Configure DATABASE_URL
Create `server/.env` (modeled after `server/.env.example`) with your connection string:
```ini
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173

# PostgreSQL Connection String Format:
# postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public
DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/dead2deal?schema=public"

JWT_SECRET="dead2deal_dev_jwt_access_secret_super_secure_key_2026"
JWT_EXPIRES_IN="15m"
JWT_REFRESH_SECRET="dead2deal_dev_jwt_refresh_secret_super_secure_key_2026"
JWT_REFRESH_EXPIRES_IN="7d"
```

### 3. Run Prisma Migration
Once PostgreSQL is running and `DATABASE_URL` is configured:
```bash
cd server
npx prisma migrate dev --name init_dead2deal
```
This generates the PostgreSQL database tables, relations, unique constraints, and indexes.

### 4. Run Prisma Seed
To seed the database with initial industrial surplus data, companies, and test accounts:
```bash
cd server
node prisma/seed.js
```

### 5. Resilient Fallback Behavior
If PostgreSQL is offline or unreachable:
- The backend automatically switches to the **in-memory fallback** data store.
- No crashes or unhandled rejections occur.
- The API contract and data responses remain **100% identical**.
- Developers can build and test immediately without configuring a database daemon.

### 6. Verifying Database Connectivity
Check the health check endpoint:
```bash
curl http://localhost:5000/api/health
```
- When PostgreSQL is active:
  ```json
  {
    "status": "ok",
    "database": "postgresql",
    "service": "Dead2Deal Core API"
  }
  ```
- When operating in fallback mode:
  ```json
  {
    "status": "ok",
    "database": "memory-fallback",
    "service": "Dead2Deal Core API"
  }
  ```

### 7. Database Backup & Restore (Production Basics)
- **Backup Database**:
  ```bash
  pg_dump -U postgres -h localhost -p 5432 -d dead2deal -F c -b -v -f "dead2deal_backup_$(date +%Y%m%d).dump"
  ```
- **Restore Database**:
  ```bash
  pg_restore -U postgres -h localhost -p 5432 -d dead2deal -v "dead2deal_backup.dump"
  ```

---

## 🚀 Running the Platform Locally

### 1. Starting the Backend Server
```bash
# From workspace root
npm run server

# Or with live auto-reload:
npm run server:dev
# Backend runs at: http://localhost:5000
# Health check: http://localhost:5000/api/health
```

### 2. Starting the Frontend
```bash
# From workspace root
npm run client
# Or: npm run dev
# Frontend runs at: http://localhost:5173
```

---

## 🧪 Automated Verification Test Suites

| Command | Focus Area | Assertions |
|---|---|---|
| `npm run test:phase21` | PostgreSQL & Prisma persistence, transactions, AI shielding, condition scan | **50 / 50 PASSED** |
| `npm run test:marketplace` | Phase 20 Inventory, Catalog, RFQ, Offer, Order lifecycle | **48 / 48 PASSED** |
| `npm run test:integration` | Phase 19 Frontend ↔ Backend integration, JWT refresh, session rehydration | **38 / 38 PASSED** |
| `npm run test:backend` | Phase 18 Core Express API, role gates, Zod validation | **35 / 35 PASSED** |
| `npm run build` | Vite production bundle compilation | **0 errors (1.51s)** |

---

## 🔐 Authentication & Roles

Dead2Deal enforces three distinct roles:
1. **`BUYER`**: Marketplace catalog, RFQ creation, quote negotiation, order tracking.
2. **`SELLER`**: Surplus inventory management, RFQ review, counter-offers, fulfillment dispatch, seller intelligence.
3. **`ADMIN`**: Platform operations role isolated to dedicated portal (`/admin`, `/admin-login`).

### Development Test Credentials
> [!CAUTION]
> **PRODUCTION SECURITY WARNING**:
> The credentials below are provided STRICTLY for local development and integration testing.
> NEVER deploy these accounts or secrets to any staging or production environment.

| Role | Email | Password | Organization |
|---|---|---|---|
| **SELLER** | `raj@ksbindia.com` | `seller123` | KSB Pumps & Surplus Industrial Ltd |
| **SELLER 2** | `anand@tatasteel.com` | `seller123` | Tata Industrial Metals Ltd |
| **BUYER** | `priya@buildright.in` | `buyer123` | BuildRight Construction & Engineering |
| **BUYER 2** | `vikram@infraprojects.in` | `buyer123` | InfraProjects Apex Corp |
| **ADMIN** | `admin@dead2deal.com` | `admin123` | Platform Operations Admin |

---

## 📦 Database Schema & Indexes (Prisma)

Core models in `server/prisma/schema.prisma`:
- **`User`**: `@@index([email])`, `@@index([role])`
- **`Company`**: `@@index([companyName])`, `@@index([gstin])`
- **`RefreshToken`**: `@@index([userId])`, `@unique([token])`
- **`InventoryItem`**: `@@index([sellerId])`, `@@index([companyId])`, `@@index([status])`, `@@index([category])`, `@@index([condition])`, `@@index([createdAt])`
- **`RFQ`**: `@@index([buyerId])`, `@@index([sellerId])`, `@@index([inventoryItemId])`, `@@index([status])`
- **`Offer`**: `@@index([rfqId])`, `@@index([senderId])`, `@@index([createdAt])`
- **`Order`**: `@@index([buyerId])`, `@@index([sellerId])`, `@@index([status])`, `offerId @unique`
- **`Notification`**: `@@index([userId])`, `@@index([readAt])`

### Standard Condition Terminology
Allowed conditions strictly follow:
`NEW_UNUSED` | `OPEN_BOX` | `USED` | `REFURBISHED`
*(Zero letter-grade Grade A/B/C/D terminology).*

---

## 🛡️ Security & Git Safety
- `.env` and `.env.*` are excluded via `.gitignore`.
- Passwords hashed with `bcrypt` (10 rounds).
- Access tokens signed with HMAC SHA-256 (`env.JWT_SECRET`, 15m expiration).
- Refresh tokens signed with HMAC SHA-256 (`env.JWT_REFRESH_SECRET`, 7d expiration).
- Helmet enabled for security headers.
- Cross-seller and cross-buyer data isolation enforced via verified JWT claims.
- Public product API strictly strips private seller decision intelligence.
