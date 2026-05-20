# PetPaw Phase 1 — Pricing (Internal)

> ⚠️ เอกสารนี้ใช้ภายในบริษัทเท่านั้น — ไม่แชร์ให้ PetPaw
> **Last updated:** 2026-05-20

---

## 1. โครงสร้างราคา

| รายการ | คำนวณ | ราคา (บาท) |
|--------|-------|----------:|
| Dev 2 คน (2 เดือน) | 2 × 50,000 | 100,000 |
| PM / Account Management | lump sum | 50,000 |
| **Base Total** | | **150,000** |
| Risk Buffer (20%) | 150,000 × 20% | 30,000 |
| Margin (~57%) | | 170,000 |
| **Recommended Quote** | | **350,000** |

> **Risk buffer 20%** จำเป็นสำหรับ fixed price — เหตุผล:
> - Integration 6 เจ้า (Flash Pay, Flash Express, Zort, Mailgun, Apitel, OneSignal)  
> - API 3rd party มักมี doc ไม่ครบ ต้อง trial-error
> - PetPaw dev ที่ต้อง onboard และจัดการด้วย
> - UAT มักเจอ bug เกิน estimate

---

## 2. สิ่งที่รวมอยู่ในราคา (Scope)

**ทีม:**
- Dev 2 คน จากบริษัท (full-time 2 เดือน)
- Dev 1 คน จาก PetPaw (บริษัทเป็นคน manage)
- PM / Account Management ดูแลทั้ง project

**Deliverables:**
- Backoffice (Next.js): product, inventory, order, banner, staff management
- Mobile App (React Native/Expo): browse, search, cart, checkout, order tracking, account
- Auth: Phone OTP (customer) + Email/Password (admin)
- Integration ครบ: Flash Pay, Flash Express, Zort OMS, Mailgun, Apitel, OneSignal, Meilisearch
- Hardcode discount (basic coupon code)
- UAT + Bug fix sprint
- Deploy: Vercel (web) + Expo EAS (mobile)

**ไม่รวม (Out of Scope):**
- 3rd party subscription fees (Vercel, Neon, Meilisearch, OneSignal ฯลฯ) → PetPaw จ่ายเอง
- Apple Developer / Google Play account → PetPaw จ่ายเอง
- Design (UI/UX mockup) → ถ้า PetPaw ไม่มี design ให้ คิดเพิ่ม
- Post-launch support → คิดเป็น retainer แยก
- Phase 2 features ทั้งหมด

---

## 3. วิเคราะห์ Margin (ภายใน)

| | บาท |
|-|----:|
| Revenue (recommended quote) | 350,000 |
| Dev cost (2 คน × 2 เดือน) | 100,000 |
| PM / Account | 50,000 |
| **Gross margin** | **~200,000 (~57%)** |

> ⚠️ **Margin บางมาก** — project นี้คุ้มค่าถ้ามองเป็น **strategic** (ต่อ relationship กับ PetPaw, ได้ codebase ที่เป็น portfolio) มากกว่ากำไรระยะสั้น
>
> ถ้าจะ protect margin แนะนำ quote ที่ **200,000 บาท** แทน (33% buffer)

---

## 4. ตัวเลือก Quote

| ตัวเลือก | ราคา | หมายเหตุ |
|---------|-----:|---------|
| | **350,000** | margin ~57%, สมเหตุสมผลกับ scope และตลาด TH |

---

## 5. เงื่อนไขสำคัญที่ต้องระบุใน contract

1. **Payment milestone** — แนะนำ 50% เริ่มงาน / 30% ส่งมอบ UAT / 20% launch
2. **PetPaw dev ต้อง available** full-time — ถ้าไม่ได้จะกระทบ timeline
3. **Design / Figma ต้อง approved ก่อน dev** — change หลัง dev เริ่มแล้วคิดเพิ่ม
4. **3rd party API access** — PetPaw ต้องให้ credentials ครบก่อน W1 สิ้นสุด
5. **Scope freeze** — feature ที่ไม่ได้อยู่ใน planning board = คิดเพิ่ม
6. **UAT period** — PetPaw มีเวลา UAT 1 สัปดาห์ ถ้าเกินคิด extra

---

## 6. 3rd Party ที่ PetPaw ต้องเตรียม (ก่อนเริ่ม)

| Service | สิ่งที่ต้องเตรียม |
|---------|----------------|
| Flash Pay | API Key + Webhook URL setup |
| Flash Express | API Key + Merchant account |
| Zort OMS | API Key + Order schema |
| Apitel | API Key + Sender name |
| Mailgun | API Key + Domain verify |
| OneSignal | App ID + API Key |
| Neon (PostgreSQL) | สร้าง project ใหม่ |
| Vercel | สร้าง team account |
