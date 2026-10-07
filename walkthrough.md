# Phase 23 Walkthrough — Admin Operations Center
# Platform Management + Moderation + Monitoring

## Overview
**Phase 23** establishes the dedicated **Admin Operations Center** for Dead2Deal, providing complete platform oversight, content moderation, negotiation monitoring, user lifecycle management, and immutable audit logging:
- Preserved strict **Admin Authentication Isolation** (`admin@dead2deal.com` rejected on regular `/login`, authenticated exclusively via `/admin-login` with `portal: "admin"`).
- Extended the database schema with the **AuditLog** model and indexes in both Prisma and the in-memory fallback.
- Implemented complete backend operations controllers and protected routes (`/api/admin/*`) requiring authenticated `ADMIN` role.
- Enforced complete privacy protection: zero password/token/secret leakage, zero seller-private AI intelligence leakage to buyers, and zero letter-grade (Grade A/B/C/D) terminology.
- Created the frontend admin service layer (`src/services/adminService.js`) and dedicated operations pages (`AdminBuyersPage`, `AdminSellersPage`, `AdminOffersPage`, `AdminAuditLogsPage`).
- Implemented and passed all 33 scenarios in the comprehensive **Phase 23 Test Suite** (`scratch/test_phase23_admin.mjs`).

---

## 🏗️ 1. Architecture & Security Implementation

### A. Admin Authentication & Role Isolation
- **Regular Login (`POST /api/auth/login`)**: Strictly rejects `ADMIN` accounts with **HTTP 403 Forbidden** and a message directing to `/admin-login`.
- **Admin Portal (`POST /api/auth/login` with `portal: "admin"`)**: Authenticates `ADMIN` accounts and generates JWT tokens; rejects BUYER and SELLER accounts with **HTTP 403**.
- **Backend Route Protection (`requireAuth` + `requireRole('ADMIN')`)**:
  - Missing token: **HTTP 401 Unauthorized**.
  - BUYER or SELLER token accessing `/api/admin/*`: **HTTP 403 Forbidden**.
  - ADMIN token: **HTTP 200 OK**.

### B. AuditLog Database Schema & Persistence
Added the `AuditLog` model to `server/prisma/schema.prisma` with relation to `User`:
```prisma
model AuditLog {
  id          String   @id @default(cuid())
  adminUserId String
  adminUser   User     @relation("AdminAuditLogs", fields: [adminUserId], references: [id], onDelete: Cascade)
  action      String
  entityType  String
  entityId    String
  metadata    Json?
  createdAt   DateTime @default(now())

  @@index([adminUserId])
  @@index([action])
  @@index([entityType])
  @@index([entityId])
  @@index([createdAt])
}
```
Dual-mode support verified:
- Prisma client generated and validated (`npx prisma validate` & `prisma generate`).
- Dual-mode in-memory store in `server/src/config/db.js` implements `this.auditLogs` and `get auditLog()`.

### C. Admin Operations Endpoints (`/api/admin/*`)
| Area | Endpoint | Method | Functionality |
|---|---|---|---|
| **Dashboard** | `/api/admin/dashboard` | `GET` | Aggregated operational KPIs, active listings, GMV, and recent events |
| **Users** | `/api/admin/users` | `GET` | Search, role & status filtering, pagination (20-100 items) |
| **Users** | `/api/admin/users/:id` | `GET` | Single user detail, company, and activity metrics |
| **Users** | `/api/admin/users/:id/status` | `PATCH` | Suspend or reactivate user; logs `USER_SUSPENDED` / `USER_REACTIVATED` |
| **Buyers** | `/api/admin/buyers` | `GET` | Buyer company, RFQ submission count, and order volumes |
| **Sellers** | `/api/admin/sellers` | `GET` | Seller company, GST verification status, catalog item count, and activity |
| **Sellers** | `/api/admin/sellers/:id` | `GET` | Single seller profile and catalog summary |
| **Sellers** | `/api/admin/sellers/:id/status` | `PATCH` | Suspend or reactivate seller; logs `SELLER_SUSPENDED` / `SELLER_REACTIVATED` |
| **Inventory** | `/api/admin/inventory` | `GET` | Platform inventory moderation: search, category, condition, seller |
| **Inventory** | `/api/admin/inventory/:id` | `GET` | Full listing inspection with seller credentials |
| **Inventory** | `/api/admin/inventory/:id/status` | `PATCH` | Change status (`ACTIVE`, `DRAFT`, `AT_RISK`, `SOLD`); logs moderation event |
| **RFQs** | `/api/admin/rfqs` | `GET` | Buyer quotation monitoring, requested prices, and terms |
| **RFQs** | `/api/admin/rfqs/:id` | `GET` | Detailed RFQ inspection with buyer, seller, and product details |
| **Offers** | `/api/admin/offers` | `GET` | Negotiation monitoring, counter-offers, and deal status |
| **Offers** | `/api/admin/offers/:id` | `GET` | Single offer inspection with timeline history |
| **Orders** | `/api/admin/orders` | `GET` | Order supervision, payment amount, and escrow state |
| **Orders** | `/api/admin/orders/:id` | `GET` | Single order inspection with shipping/tracking details |
| **Orders** | `/api/admin/orders/:id/status` | `PATCH` | Status update; logs `ORDER_STATUS_CHANGED` |
| **Notifications** | `/api/admin/notifications` | `GET` | Platform notification dispatch monitoring |
| **Audit Logs** | `/api/admin/audit-logs` | `GET` | Filter by action, entityType, adminUserId, date; paginated |

---

## 🎨 2. Frontend Admin Operations Center

### A. Admin Service Layer (`src/services/adminService.js`)
All admin network calls are centralized through `adminService` using `apiClient.js` (no raw `fetch` calls in React components). Automatic JWT token injection, token refresh handling, and friendly error normalization.

### B. Admin Navigation & Layout
Updated `AdminSidebar.jsx` and `App.jsx` with dedicated routes protected by `<ProtectedRoute requiredRole="admin">`:
- `/admin` → `AdminOverviewPage`
- `/admin/users` → `AdminUsersPage`
- `/admin/buyers` → `AdminBuyersPage`
- `/admin/sellers` → `AdminSellersPage`
- `/admin/inventory` → `AdminInventoryPage`
- `/admin/rfqs` → `AdminRFQsPage`
- `/admin/offers` → `AdminOffersPage`
- `/admin/orders` → `AdminOrdersPage`
- `/admin/notifications` → `AdminNotificationsPage`
- `/admin/audit-logs` → `AdminAuditLogsPage`
- `/admin/settings` → `AdminSettingsPage`

---

## 🧪 3. Verification & Test Suite Summary

### Phase 23 Test Suite (`scratch/test_phase23_admin.mjs`)
Executed all 33 required verification scenarios:

```
================================================================
PHASE 23 — ADMIN OPERATIONS CENTER SECURITY & FUNCTIONAL TEST SUITE
================================================================

  ✅ [PASS] 1. Admin login via /admin-login portal succeeds with JWT
  ✅ [PASS] 2. Normal login strictly rejects admin account (HTTP 403)
  ✅ [PASS] 3. Buyer cannot access admin dashboard API (HTTP 403)
  ✅ [PASS] 4. Seller cannot access admin dashboard API (HTTP 403)
  ✅ [PASS] 5. Admin dashboard API returns comprehensive platform metrics
  ✅ [PASS] 6. Admin users list returns paginated users with total count
  ✅ [PASS] 7. Admin user detail endpoint returns single user profile
  ✅ [PASS] 8. Admin can suspend a user account (status -> SUSPENDED)
  ✅ [PASS] 9. Admin can reactivate a suspended user account (status -> ACTIVE)
  ✅ [PASS] 10. Seller management endpoint returns sellers with catalog and operational counts
  ✅ [PASS] 11. Seller suspend and reactivate actions persist successfully
  ✅ [PASS] 12. Inventory management: listing retrieval and status moderation (DRAFT <-> ACTIVE)
  ✅ [PASS] 13. RFQ monitoring endpoint returns all buyer quote requests for supervision
  ✅ [PASS] 14. Offer monitoring endpoint returns negotiation counters and history
  ✅ [PASS] 15. Order monitoring endpoint returns transaction records and escrow states
  ✅ [PASS] 16. Notification monitoring endpoint returns operational dispatch records
  ✅ [PASS] 17. Audit log creation: admin operations automatically create immutable log records
  ✅ [PASS] 18. Audit log retrieval returns structured items with administrator identity
  ✅ [PASS] 19. Admin list pagination: supports page, limit, and totalPages calculation
  ✅ [PASS] 20. Unauthorized request without JWT returns HTTP 401
  ✅ [PASS] 21. Buyer token accessing /api/admin/users returns HTTP 403 Forbidden
  ✅ [PASS] 22. Seller token accessing /api/admin/inventory returns HTTP 403 Forbidden
  ✅ [PASS] 23. Sensitive data protection: password and passwordHash are NEVER leaked in admin APIs
  ✅ [PASS] 24. Token confidentiality: refreshTokens and tokenHashes are NEVER exposed in admin responses
  ✅ [PASS] 25. Financial privacy: no private bank credentials or payout secrets leaked
  ✅ [PASS] 26. AI Privacy Audit: ZERO seller-private AI intelligence leaked to public buyer APIs
  ✅ [PASS] 27. Condition Terminology Audit: ZERO letter-grade terminology (NEW_UNUSED, OPEN_BOX, USED, REFURBISHED verified)
  ✅ [PASS] 28. Existing backend test suite passes (Phase 18 authentication & foundation)
  ✅ [PASS] 29. Existing integration test suite passes (Phase 19 API transport & auth flow)
  ✅ [PASS] 30. Existing marketplace test suite passes (Phase 20 inventory, RFQs, offers, orders)
  ✅ [PASS] 31. Phase 22 persistence suite passes (PostgreSQL readiness & verified fallback)
  ✅ [PASS] 32. Route smoke test passes (All 33 frontend routes return HTTP 200 OK)
  ✅ [PASS] 33. Production build passes with 0 errors (Vite production bundle)

================================================================
PHASE 23 TEST RESULTS: 33 PASSED, 0 FAILED (TOTAL: 33/33)
================================================================
```

### Full Regression Suite Results
| Test Suite | Command | Assertions | Result |
|---|---|---|---|
| **Phase 23 Admin Center** | `npm run test:phase23` | **33 passed, 0 failed** | ✅ PASS (100%) |
| **Phase 22 Persistence** | `npm run test:phase22` | **52 passed, 0 failed, 2 not run** | ✅ PASS |
| **Phase 20 Marketplace** | `npm run test:marketplace` | **48 passed, 0 failed** | ✅ PASS |
| **Phase 19 Integration** | `npm run test:integration` | **38 passed, 0 failed** | ✅ PASS |
| **Phase 18 Backend** | `npm run test:backend` | **35 passed, 0 failed** | ✅ PASS |
| **Route Smoke Test** | `node scratch/test_routes.mjs` | **33 / 33 routes return 200 OK** | ✅ PASS |
| **Production Build** | `npm run build` | **0 errors (1.16s)** | ✅ PASS |

> [!NOTE]
> In accordance with instructions, PostgreSQL tests are clearly marked NOT RUN (not fabricated) when the PostgreSQL daemon is offline, and all operations function seamlessly on the verified in-memory fallback.
