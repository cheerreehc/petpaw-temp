# PetPaw — Project Overview

> **Status:** Early stage · Restructure from scratch  
> **Phase 1 timeline:** มิถุนายน – กรกฎาคม 2026 (1 มิ.ย. – 26 ก.ค. 2026)  
> **Team:** 2–3 devs  
> **Last updated:** 2026-05-20

---

## 1. What is PetPaw

Multi-seller marketplace สำหรับสินค้าสัตว์เลี้ยง (Shopee/Lazada style)

- **Phase 1** — PetPaw เป็นผู้ขายรายเดียว fulfil ด้วย flow เดิม (Zort OMS + Flash Express)  
- **Phase 2+** — เปิดให้ seller ภายนอก listing สินค้าบน platform ได้

ทำไมต้อง restructure:
- ของเก่ามี CRM แยก + backoffice แยก → รวมให้อยู่ที่เดียว
- 3rd party ใหม่หมด: low cost, easy maintain สำหรับทีมเล็ก
- ออกแบบตั้งแต่แรกให้รองรับ multi-seller ใน schema/domain

---

## 2. Architecture

### 2.1 Monorepo Structure

```
petpaw/
├── apps/
│   ├── web/          ← Next.js 15  (backoffice + customer web)
│   └── mobile/       ← React Native / Expo  (iOS + Android)
└── packages/
    ├── db/           ← Prisma schema + Neon client
    ├── domain/       ← Business logic pure functions (coupon engine, pricing)
    └── types/        ← Shared TypeScript types
```

Toolchain: **Turborepo** — shared build cache, task pipeline

### 2.2 Tech Stack

| Layer | Tool | เหตุผล |
|-------|------|--------|
| Web / Backoffice | **Next.js 15** (App Router, TypeScript) | Fullstack ในที่เดียว, team เล็ก MA ง่าย |
| Mobile | **React Native + Expo** | Cross-platform iOS/Android, share types กับ web |
| Database | **PostgreSQL via Neon** | Serverless Postgres, Vercel Marketplace, scale ได้ |
| ORM | **Prisma** | Type-safe, migration ง่าย |
| Auth | **Better Auth** | TypeScript-native, roles/sessions, ไม่มี vendor lock |
| Hosting | **Vercel** | Zero-config, Next.js native, preview URL ทุก PR |
| Image Storage | **Vercel Blob** | แทน Cloudinary — ถูกกว่า, integrate กับ Vercel โดยตรง |
| Search | **Meilisearch** | แทน Algolia — self-host (Railway), open source, MA ง่าย |
| Push Notifications | **OneSignal** | Free tier เพียงพอ Phase 1 |
| Mobile Build | **Expo EAS** | OTA update, build cloud |

### 2.3 3rd Party Integrations

| Service | Role | Decision |
|---------|------|----------|
| **Zort OMS** | Order management / fulfillment (Phase 1) | ต่อ API เดิม |
| **Flash Express** | Shipping | ต่อ API เดิม |
| **Flash Pay** | Payment gateway | มี merchant account แล้ว — ใช้เดิม |
| **Mailgun** | Transactional email (order confirm, reset) | ใช้เดิม |
| **Apitel** | SMS OTP | ใช้เดิม |
| **OneSignal** | Push notification (mobile) | ใช้เดิม |
| Cloudinary | Image CDN | **ยกเลิก** → Vercel Blob |
| Algolia | Search | **ยกเลิก** → Meilisearch |

### 2.4 Auth Strategy

Customer login: **เบอร์โทรศัพท์ + OTP** (ผ่าน Apitel SMS)  
Admin/Staff login: Email + Password  
ไม่ใช้ social login ใน Phase 1

---

## 3. Domain Modules

ทุก module อยู่ใน backoffice เดียวกัน (ไม่แยก CRM อีกต่อไป)

| Module | ประกอบด้วย |
|--------|-----------|
| **Catalog** | Product, Category, Brand, Variant, Image |
| **Inventory** | Stock, Warehouse (Phase 1: single location + Zort sync) |
| **Order** | Cart, Checkout, Order lifecycle, Zort push |
| **Customer** | Account, Address, Order history |
| **Coupon / Pricing** | ตาม [coupon-spec.md](./coupon-spec.md) — pure function engine |
| **Shipping** | Flash Express rate calculation, label |
| **Payment** | Gateway webhook, payment status |
| **Seller** | (Phase 2) Seller onboarding, listing, payout |
| **Settlement** | (Phase 2) funded-by tracking, payout calculation |
| **Notification** | Push (OneSignal), email (Mailgun), SMS (Apitel) |
| **In-app Chat** | (Phase 2) Customer ↔ Seller chat — Sendbird |

---

## 4. Database Design Principles

- **Relational** (PostgreSQL) — เหมาะกับ multi-seller, settlement, partial refund
- Schema ออกแบบรองรับ multi-seller ตั้งแต่แรก (`seller_id` / `store_id` ทุก table ที่เกี่ยวข้อง) แม้ Phase 1 จะมีแค่ร้านเดียว
- Coupon predicates + stacking เก็บเป็น JSONB (flexible, ไม่ต้อง migrate เมื่อเพิ่ม condition)
- Audit trail แยก table (immutable log) สำหรับ order events + coupon usage
- Soft delete สำหรับ product / coupon (ไม่ลบจริง เพื่อ order history)

---

## 5. Roles & Auth

| Role | สิทธิ์ |
|------|-------|
| `platform_admin` | ทุกอย่าง |
| `platform_staff` | จัดการ order, catalog, ไม่แตะ config |
| `seller_admin` | (Phase 2) จัดการ store ของตัวเอง |
| `customer` | browse, order, manage own account |

---

## 6. Phase 1 — Scope (2 months)

**IN:**
- Backoffice: product, category, brand, inventory, order management
- Customer web (Next.js): browse, search, detail page
- Mobile app (RN/Expo): browse, search, cart, checkout, order tracking, push notification
- Auth: admin/staff login (backoffice), customer login (web + mobile)
- Image upload → Vercel Blob
- Search → Meilisearch (product text search)
- Zort OMS integration (push order เมื่อ checkout สำเร็จ)
- Flash Express shipping rate
- Payment gateway (integrate ทันที payment เลือกได้)
- Coupon — simplified version (platform + store coupon พื้นฐาน)

**OUT (Phase 2+):**
- Multi-seller onboarding / seller portal
- Settlement / payout report
- Advanced coupon (stacking, brand, double day)
- Auto-optimizer coupon
- Loyalty / cashback
- Bundle promotions
- Review / rating system
- In-app Chat (Sendbird) — customer ↔ seller communication

---

## 7. Project Timeline

### Week 1 — Foundation
- [ ] Monorepo setup (Turborepo + pnpm workspaces)
- [ ] Next.js 15 project + TypeScript config
- [ ] React Native / Expo project
- [ ] Neon PostgreSQL + Prisma setup
- [ ] Vercel project link + env vars pipeline
- [ ] Better Auth setup (admin/staff/customer roles)
- [ ] Core DB schema (user, store, product, order skeleton)

### Week 2 — Catalog Backoffice
- [ ] Product CRUD (backoffice)
- [ ] Category + Brand management
- [ ] Image upload → Vercel Blob
- [ ] Meilisearch setup + product indexing
- [ ] Product variant / stock schema

### Week 3 — Inventory + Zort
- [ ] Inventory management (backoffice)
- [ ] Zort OMS API integration (push order, sync stock)
- [ ] Stock alert / low stock warning

### Week 4 — Mobile: Browse + Search
- [ ] Product listing screen (RN)
- [ ] Category browsing
- [ ] Search screen (Meilisearch API)
- [ ] Product detail screen
- [ ] Bottom navigation shell

### Week 5 — Cart + Checkout
- [ ] Cart system (mobile)
- [ ] Address management
- [ ] Flash Express shipping rate call
- [ ] Basic coupon (platform fixed/percentage — Phase 1 simplified)
- [ ] Checkout summary screen
- [ ] Payment gateway integration

### Week 6 — Order + Fulfillment
- [ ] Order creation flow (checkout → Zort push)
- [ ] Order management backoffice (list, detail, status update)
- [ ] Order status lifecycle
- [ ] Expo Push notification setup
- [ ] Order tracking screen (mobile)

### Week 7 — Customer Account + Polish
- [ ] Customer auth (mobile: register/login)
- [ ] Profile + address management (mobile)
- [ ] Order history screen (mobile)
- [ ] Customer web (Next.js): basic storefront pages
- [ ] Payment webhook handler + order confirmation

### Week 8 — QA + Launch
- [ ] Bug fix sprint
- [ ] E2E testing critical paths (checkout flow)
- [ ] Performance review (image CDN, query optimize)
- [ ] Expo EAS build + App Store / Play Store submission
- [ ] Vercel production deploy
- [ ] Monitoring setup (Vercel Analytics / Sentry)
- [ ] Handoff docs

---

## 8. Key Files

| File | ความหมาย |
|------|---------|
| [coupon-spec.md](./coupon-spec.md) | Coupon/discount concept spec (approved) |
| `packages/domain/pricing.ts` | Pricing engine — pure function |
| `apps/web/` | Next.js backoffice + customer web |
| `apps/mobile/` | React Native app |
| `packages/db/schema.prisma` | Database schema (source of truth) |

---

## 9. Open Decisions

| ประเด็น | สถานะ |
|--------|-------|
| Payment gateway | **Flash Pay** — มี merchant account แล้ว |
| Search v2 | Meilisearch สำหรับ Phase 1; อนาคตพิจารณา pgvector AI search |
| Seller portal UX | Phase 2 — ยังไม่ออกแบบ |
| Settlement logic | Phase 2 — ใช้ coupon-spec funded-by เป็น base |
