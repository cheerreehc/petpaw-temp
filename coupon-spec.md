# Coupon / Discount — Concept Spec

> **Status:** approved
> **Phase:** 1
> **Last updated:** 2026-05-18

เอกสารนี้ว่าด้วย **คอนเซ็ปต์** ของระบบ coupon/discount สำหรับ multi-seller marketplace
เน้นความหมาย พฤติกรรม และกฎเกณฑ์ทางธุรกิจ — **ไม่อิงภาษา/framework/database** ใดทั้งสิ้น
ใช้เป็น reference เพื่อ implement บน tech stack ใดก็ได้ภายหลัง

---

## 1. Context

ระบบ e-commerce ที่ออกแบบเป็น marketplace มีหลาย store อยู่บน platform เดียวกัน (รูปแบบ Shopee/Lazada)
ส่วนลดที่ user ได้รับมาได้จากหลายแหล่งพร้อมกัน:

- Platform เป็นเจ้าของ campaign
- Store/seller ออกเอง
- Brand ออกเอง
- ส่วนลดค่าจัดส่ง (อาจมาจาก platform หรือ store)
- ราคา flash sale ของสินค้า (เป็นการ “ปรับราคา” ไม่ใช่ส่วนลด)

ความท้าทาย:

- ส่วนลดบางตัว **ใช้ร่วมกันไม่ได้** (เช่น store + platform บางตัว)
- ส่วนลดทุกตัวมี **เงื่อนไข** เฉพาะ (ขั้นต่ำราคา, วันที่, ประเภทสินค้า, ลูกค้าใหม่ ฯลฯ)
- การคำนวณต้อง **โปร่งใส** — ลูกค้าและ accounting ต้องตรวจที่มาของแต่ละบาทได้

---

## 2. Mental Model — Pricing Layers

หัวใจของระบบคือมอง “ราคาสุดท้าย” เป็นผลของการ apply discount เป็น **ชั้นๆ ตามลำดับ** ไม่ย้อนกลับ:

```text
Layer 0: List Price           ราคาที่ seller ตั้งไว้
Layer 1: Flash Sale override  ราคาพิเศษช่วงเวลา → “current item price”
Layer 2: Brand discount       ลดเฉพาะ line ของ brand นั้น
Layer 3: Store discount       ลด store subtotal (1 ใบ/store)
Layer 4: Platform discount    ลด cart subtotal — กระจาย pro-rata ลงทุก store/line
Layer 5: Shipping fee + Shipping discount
─────────────────────────────────────────
         Final Total
```

ทำไมต้องแบ่งเป็น layer:

- ทำให้ลำดับการคำนวณคาดเดาได้ ไม่มีรอบย้อน
- รองรับ partial refund (รู้ได้ว่าถ้าคืน item นี้ ส่วนลดไหนจะหายไปบ้าง)
- รายงาน accounting แยกผู้รับภาระค่าส่วนลด (platform vs store vs brand)

---

## 3. Domain Entities (เชิงคอนเซ็ปต์)

### 3.1 Catalog

- **Product** — มี: id, ชื่อ, ราคา list, อยู่ที่ store ใด, แบรนด์ใด, หมวดใด
- **Store** — ผู้ขายบน platform; อาจครอบ brand หลายตัว
- **Brand** — แบรนด์ของสินค้า
- **Category** — หมวดสินค้า

### 3.2 Flash Sale

- ผูกกับ **product** เป็นรายชิ้น
- มีราคาพิเศษ (`salePrice`) + ช่วงเวลา (`startsAt`, `endsAt`) + (optional) จำกัด stock
- **ไม่ใช่ coupon** — ไม่ต้อง claim ไม่ต้อง apply ลูกค้าได้ราคานี้อัตโนมัติเมื่ออยู่ในช่วง

### 3.3 Coupon Definition (template)

“ใบเทมเพลต” ที่ admin/seller สร้างไว้ มีคุณสมบัติเชิงคอนเซ็ปต์ดังนี้:

| คุณสมบัติ | ความหมาย |
|----------|---------|
| **Code / Name / Description** | ระบุ + อธิบายให้ลูกค้าอ่าน |
| **Issuer** | ใครเป็นคนออก: `platform` / `store` / `brand` / `shipping` |
| **Scope** | ไป apply กับชั้นไหน: `line` / `store_subtotal` / `cart_subtotal` / `shipping` |
| **Discount value** | `percentage` (มี max cap ได้) / `fixed` / `free_shipping` |
| **Conditions** | เงื่อนไขที่ต้องผ่านก่อนใช้ได้ (ดูข้อ 5) |
| **Stacking policy** | ใช้ร่วมกับ coupon อื่นได้แบบไหน (ดูข้อ 6) |
| **Funded by** | ใครรับภาระค่าส่วนลด: `platform` / `store` / `brand` |
| **Active window** | ช่วงเวลาที่ coupon ใช้ได้ |
| **Quotas** | จำนวนรวม (`totalQuota`) + ต่อคน (`perUserQuota`) |
| **Owner** | ถ้า issuer=store ผูกกับ store ใด; ถ้า brand ผูกกับ brand ใด |

### 3.4 User Coupon (claim record)

- ลูกค้า **claim** coupon ใบหนึ่งเข้ามาไว้กับตัวเอง
- บันทึก: ลูกค้าคนนี้, coupon ใบนี้, เวลา claim, จำนวนครั้งที่ใช้แล้ว
- ใช้ตรวจ `perUserQuota` และเป็นกรรมสิทธิ์ของลูกค้า (coupon ที่ยังไม่ claim จะ apply ไม่ได้)

### 3.5 Cart

- รายการ **line** = product + qty
- รายการ coupon ที่ลูกค้า **apply** ใน checkout ปัจจุบัน

### 3.6 Pricing Context

ตัวแปรแวดล้อมที่จำเป็นต่อการประเมิน eligibility:

- เวลาปัจจุบัน (`now`) — สำหรับ active window / date predicates
- ผู้ใช้ — userId
- วิธีชำระเงิน — เผื่อ coupon บางใบจำกัด
- เป็นลูกค้าใหม่หรือไม่ — สำหรับ first-order coupon
- ประวัติการใช้ coupon ของลูกค้านี้ — สำหรับ `perUserQuota`

---

## 4. Discount Dimensions

### 4.1 Issuer (ใครออก)

| Issuer | ลักษณะ |
|--------|-------|
| `platform` | platform/marketplace ออกเอง — ใช้ได้ทุก store ทุก brand |
| `store` | ร้านค้าออก — apply เฉพาะสินค้าของร้านนั้น |
| `brand` | แบรนด์ออก — apply เฉพาะสินค้าของ brand นั้น (ข้ามร้าน) |
| `shipping` | ส่วนลดค่าจัดส่ง (ใครออกก็ได้ แต่แยกประเภทเพราะ apply กับ shipping fee) |

### 4.2 Scope (ลดที่ชั้นไหน)

| Scope | คำอธิบาย | จำนวนสูงสุดต่อ cart |
|-------|---------|---------------------|
| `line` | ลดต่อ line item (เช่น brand discount) | apply ได้หลายใบ ถ้าตรง entitlement ของแต่ละ line |
| `store_subtotal` | ลดบนยอดรวมของ 1 store | **1 ใบ/store** |
| `cart_subtotal` | ลดบนยอดรวมทั้ง cart (หลัง store discount) | **1 ใบ/cart** |
| `shipping` | ลดค่าส่ง | **1 ใบ/cart** |

### 4.3 Discount value (ค่าส่วนลด)

| Type | ความหมาย |
|------|---------|
| `percentage` | ลด % โดยมี `maxCap` (เพดานบาท) เป็น optional |
| `fixed` | ลดเป็นจำนวนเงินคงที่ (clamp ไม่เกินยอดฐาน) |
| `free_shipping` | ลดค่าส่ง 100% (ใช้กับ scope=shipping) |

### 4.4 Funded by (ใครจ่าย)

สำคัญสำหรับ **settlement กับ seller**:

- `platform` — platform หักจากกระเป๋าตัวเอง, seller ได้ยอดเต็ม
- `store` — หักจาก seller payout
- `brand` — หักจาก brand (มัก reimburse กลับ)

ใน UI/breakdown ควรแสดงแยกสีตาม funder เพื่อโปร่งใสกับลูกค้า

---

## 5. Eligibility (เงื่อนไขการใช้)

โครงสร้าง: **list of predicates แบบ AND** (ทุก predicate ต้องผ่าน coupon ถึงจะ apply ได้)

### 5.1 Predicate types

| Predicate | ความหมาย |
|-----------|---------|
| `minSubtotal` (line/store/cart) | ยอดขั้นต่ำของ scope ที่ระบุ ≥ value |
| `dateWindow` | เวลาปัจจุบันต้องอยู่ใน [from, to] |
| `specificDates` | วันที่ปัจจุบันต้องตรงกับ list เช่น Double Day (5/5, 10/10) |
| `includeStores` / `excludeStores` | จำกัด store ที่ entitled |
| `includeBrands` | จำกัด brand |
| `includeCategories` | จำกัดหมวด |
| `excludeFlashSaleItems` | ตัด item ที่อยู่ใน flash sale ออกจากฐานการลด |
| `firstOrderOnly` | เฉพาะลูกค้าที่ยังไม่เคยสั่งซื้อ |
| `paymentMethod` | จำกัดวิธีชำระเงิน |
| `perUserUsageLimit` | ใช้ได้กี่ครั้ง/คน |

### 5.2 Cross-cutting (ตรวจอัตโนมัติทุกใบ)

นอกเหนือจาก predicate list ระบบต้องตรวจเพิ่ม:

- เวลาปัจจุบันอยู่ใน [activeFrom, activeTo] ของ coupon
- `totalQuota` ยังไม่หมด
- `perUserQuota` ของลูกค้านี้ยังไม่ครบ

### 5.3 ทำไมใช้ predicate list (ไม่ hard-code)

- เพิ่ม condition type ใหม่ได้โดยไม่ต้องแก้ schema (extensible)
- Admin UI render form ได้ตาม type (dynamic)
- สามารถ serialize ลง DB ในรูปแบบ JSON / structured

---

## 6. Stacking Rules (กฎการรวมส่วนลด)

### 6.1 หลักการ: tag-based compatibility

แต่ละ coupon มี 3 ฟิลด์เกี่ยวกับ stacking:

- `stackingTags` — ป้ายของตัวเอง (เช่น `[shipping]`, `[platform, doubleday]`)
- `allowStackWith` — รายการ tag ที่ยอม stack ด้วย
- `denyStackWith` — รายการ tag ที่ห้าม stack ด้วย

### 6.2 Validation: pairwise + bidirectional

เมื่อลูกค้า apply coupon หลายใบ ระบบตรวจทุกคู่ตามกฎต่อไปนี้ — **ทั้ง 2 ฝั่งต้องไม่ขัด**:

```text
สำหรับคู่ (A, B):
  1. ถ้า A.denyStackWith ∩ B.stackingTags ≠ ∅  →  ห้าม
  2. ถ้า B.denyStackWith ∩ A.stackingTags ≠ ∅  →  ห้าม
  3. ทุก tag ของ B ต้อง ∈ A.allowStackWith  (หรือ tag เดียวกันกับ A เอง)
  4. ทุก tag ของ A ต้อง ∈ B.allowStackWith  (หรือ tag เดียวกันกับ B เอง)
ถ้า fail ข้อใดข้อหนึ่ง → ห้ามใช้คู่นี้
```

### 6.3 ทำไม bidirectional

- ป้องกัน config ผิดพลาด (A allow B แต่ B ไม่ allow A ก็ถือว่าไม่ใช้ร่วม)
- ทำให้ Admin คิดสองทาง: ถ้าใส่ tag ใหม่ ต้องคิดทั้งสองด้าน

### 6.4 Same-tag implicit allow

ถ้า A และ B มี tag เดียวกัน (เช่นทั้งคู่เป็น `platform`) → implicit allow โดยไม่ต้องระบุใน `allowStackWith` (แต่ scope ระดับเดียวกันจะถูกจำกัดอีกชั้นในข้อ 4.2 — เช่น cart_subtotal ใช้ได้แค่ 1 ใบ)

### 6.5 ตัวอย่าง

| Coupon | tags | allow | deny |
|--------|------|-------|------|
| Free Shipping | `[shipping]` | `[platform, store, brand]` | — |
| Platform 10% | `[platform]` | `[shipping, brand]` | `[store, doubleday]` |
| Store 20% | `[store]` | `[brand, shipping]` | `[platform, doubleday]` |
| Brand 15% | `[brand]` | `[platform, store, shipping]` | — |
| Double Day ฿100 | `[platform, doubleday]` | `[shipping]` | `[store]` |

จากตารางนี้:

- Shipping + Platform :white_check_mark: ใช้ร่วมกันได้
- Platform + Store :x: ทั้งคู่ deny ซึ่งกันและกัน
- Brand + ทุกใบ :white_check_mark: stack ได้กับทุกอัน
- Double Day + Platform 10% :x: doubleday tag ถูก deny โดย Platform 10%

---

## 7. Calculation Flow

### 7.1 ลำดับขั้นตอน

1. **โหลด coupon ที่ apply** จาก cart
2. **Validate stacking** ทุกคู่ — ถ้ามี violation → คืน error, ไม่คำนวณต่อ
3. **Resolve item price** — apply flash sale override ทุก line → ได้ `current_price`
4. **Apply line-level (brand) discount** ต่อ line ที่ entitled → ได้ `line_after_discount`
5. **Group by store** + คำนวณ `store_subtotal`
6. **Apply store-level discount** (max 1/store) → ได้ `store_after_discount`
7. **คำนวณ cart_subtotal** = Σ store_after_discount
8. **Apply platform-level discount** บน cart_subtotal:
   - ถ้ามี `excludeFlashSaleItems` → ฐาน = subtotal เฉพาะ line ไม่ flash
   - Clamp ไม่เกิน cart_subtotal
   - **กระจาย pro-rata** ลงทุก store ตามสัดส่วน store_after_discount
   - กระจายต่อลง line ตามสัดส่วน line_after_discount
9. **Apply shipping discount** ต่อ shipping fee ของแต่ละ store
10. **Roll up**:
    - store_total = store_after_discount − platform_share + shipping_after
    - grand_total = Σ store_total

### 7.2 Output (Breakdown)

ผลลัพธ์ที่ engine ต้องคืน เพื่อให้ UI/accounting/audit ทำงานได้ครบ:

- **steps[]** — รายการแต่ละขั้นพร้อม label + math + amount (ใช้ render breakdown แบบโปร่งใส)
- **lines[]** — สถานะของแต่ละ line item หลังลด (รวม pro-rata share)
- **stores[]** — สถานะของแต่ละ store (subtotal, store discount, platform share, shipping)
- **errors[]** — coupon ที่ apply ไม่ได้ + เหตุผล
- **warnings[]** — coupon eligible แต่ไม่มี item ที่ apply ได้ ฯลฯ
- **finalTotal, totalDiscount, itemsSubtotal, shippingTotal**

### 7.3 Engine ต้องเป็น Pure Function

- รับ input snapshot → return result
- **ไม่แตะ DB** ไม่มี side effect
- เรียกซ้ำได้ทั้งใน cart preview, checkout, audit, replay
- ทำให้ test ง่ายและ deterministic

---
[5:08 PM]## 8. Pro-rata Distribution

### 8.1 ทำไมต้อง pro-rata

ถ้า platform ลด ฿100 บน cart 2 store (subtotal 600 + 400 = 1000):

- ลูกค้าจ่ายจริง 900
- **ต้องรู้ว่าแต่ละ store ได้ลด “เท่าไร“** เพื่อ:
  - ถ้าลูกค้าคืน store A หมด → คืนเงิน 600 − 60 (ส่วนแบ่ง pro-rata) = 540
  - Settlement: platform จ่ายให้ store A 540, store B 360 (ลูกค้าจ่าย platform 900 รวม)

### 8.2 สูตร

```text
store_share = total_discount × (store_subtotal / cart_subtotal)
line_share  = store_share   × (line_subtotal  / store_subtotal)
```

ปัดเป็นจำนวนเต็มทุกตัว → อาจมี rounding drift

### 8.3 Rounding drift

ผลรวม share หลังปัดอาจไม่ตรง total discount เป๊ะ ๆ → drift ที่เหลือใส่ลง **store ใหญ่สุด** (ค่าเสียหายน้อยที่สุดเมื่อคิดเป็น %)

---

## 9. Business Decisions & Trade-offs

| ประเด็น | เลือก | เหตุผล |
|--------|-------|--------|
| **Flash sale** | price override (ไม่ใช่ coupon) | ลูกค้าไม่ต้อง claim, ราคาแสดงตอนเลือกซื้อเลย |
| **Store coupon ต่อ store** | สูงสุด 1 ใบ | UX ชัด, ลด edge case |
| **Platform coupon ต่อ cart** | สูงสุด 1 ใบ | เหตุผลเดียวกัน |
| **Stacking direction** | bidirectional | กัน config ผิด |
| **Discount distribution** | pro-rata ลงทุก line | จำเป็นสำหรับ partial refund + settlement |
| **Best combo auto-pick** | ไม่ทำใน v1 | ปัญหา NP-hard, UX ยุ่ง; ให้ user เลือกเอง + แสดง hint |
| **Predicate model** | JSON predicate list | extensible, ไม่ต้อง redeploy เมื่อเพิ่ม condition |
| **Funded-by tracking** | บังคับทุก coupon | ใช้ใน settlement + UI transparency |
| **Conditions logic** | AND ทั้งหมด | ง่าย เข้าใจง่าย; ถ้าต้อง OR ให้สร้าง coupon คนละใบ |

---

## 10. Reference Scenarios

scenario เหล่านี้ใช้ทดสอบความถูกต้องของ engine + เป็นตัวอย่างที่ครอบคลุมการใช้งานหลัก

### A. Cart < min subtotal

- Cart รวม 290 บาท
- Apply: Platform 10% (min 500) + Free Shipping
- คาดหวัง: Platform fail (ไม่ถึงขั้นต่ำ) → error / Free Shipping ใช้ได้

### B. Stack: Platform + Shipping

- Cart รวม 557
- Apply: Platform 10% cap 50 + Free Shipping
- คาดหวัง: ลดได้ 50 (cap) + ฟรีค่าส่ง

### C. Stacking conflict

- Apply: Store 20% + Platform 10% (deny ซึ่งกันและกัน)
- คาดหวัง: error banner, ไม่คำนวณต่อ

### D. Multi-store pro-rata

- Cart: store A 198 + store B 359 (total 557)
- Apply: Platform 10% cap 50
- คาดหวัง: ลด 50, กระจาย A≈18, B≈32 (ตามสัดส่วน)

### E. Exclude flash sale

- Cart: item flash (99×2=198) + item ปกติ (50)
- Apply: 10% coupon ที่ `excludeFlashSaleItems`
- คาดหวัง: คิด 10% จากฐาน 50 = ลด 5

### F. Double Day specific date

- Apply DoubleDay ฿100 (เฉพาะ 5/5, 10/10)
- ถ้าวันที่ไม่ตรง → fail
- ถ้าตรง → ลด 100

### G. Brand + Store + Shipping triple stack

- Cart มี item ของ brand X ใน store Y
- Apply: Brand 15% (line) + Store 20% (store) + Free Shipping
- คาดหวัง: ลด 3 ชั้นซ้อน, ไม่ชน

### H. First order

- Apply NEWBIE coupon (firstOrderOnly)
- ถ้า user เคยสั่งแล้ว → fail
- ถ้ายัง → ลดได้ + เพิ่ม usage count

---

## 11. Edge Cases

| Case | พฤติกรรมที่คาด |
|------|---------------|
| Empty cart + apply coupon | total = 0, มี warning “ไม่มีสินค้า” |
| Coupon หมดอายุ | eligibility fail + reason ระบุ |
| Quota หมด (total / per-user) | claim ไม่ได้; ถ้า apply ค้างมา → fail ตอน checkout |
| Flash sale หมดเวลา | ราคาเด้งกลับ list price อัตโนมัติ |
| Apply 2 store coupons ที่ store เดียว | apply ใบแรก + warning ใบที่สอง |
| Platform discount > cart subtotal | clamp = cart subtotal |
| Pro-rata มี rounding drift | drift ใส่ store ที่ใหญ่ที่สุด |
| ลบ item จน eligibility fail | coupon โดน drop + warning, total ยังคำนวณได้ |
| Admin ลบ coupon ที่ user มี applied | unclaim อัตโนมัติ + warning |
| `excludeFlashSaleItems` แต่ทุก item เป็น flash | ฐาน = 0 → ส่วนลด 0 + warning |

---

## 12. Out of Scope

หัวข้อต่อไปนี้ **ไม่ครอบ** ในเอกสารนี้ (ต้องออกแบบเพิ่มเติมตอน implement):

- Refund / partial return flow (การ reverse discount เมื่อคืนของ)
- Loyalty points / cashback
- Bundle promotion “buy X get Y free”
- Auto-optimizer (เลือก combo coupon ที่คุ้มสุดให้ลูกค้า)
- Payment gateway integration
- Authentication / authorization
- Coupon distribution campaigns (mass issuance, referral code)
- Admin UI / role-based permissions
- Logging / observability
- A/B test framework

---

## 13. Glossary

| คำ | ความหมาย |
|----|---------|
| **Coupon Definition** | ใบเทมเพลตที่ admin/seller สร้าง — มีกฎทุกอย่าง |
| **User Coupon** | บันทึกที่ลูกค้าเก็บ coupon ใบหนึ่งไว้กับตัวเอง |
| **Claim** | การที่ลูกค้ากดเก็บ coupon (ก่อน apply) |
| **Apply** | การที่ลูกค้าเลือกใช้ coupon ใน cart ปัจจุบัน |
| **Issuer** | คนที่ออก coupon (platform/store/brand/shipping) |
| **Funded by** | คนที่รับภาระค่าส่วนลด |
| **Scope** | ชั้นที่ coupon ไป apply (line/store/cart/shipping) |
| **Stacking** | การใช้ coupon มากกว่า 1 ใบพร้อมกัน |
| **Predicate** | เงื่อนไข 1 ข้อใน eligibility (AND กันทั้งหมด) |
| **Pro-rata** | การกระจายส่วนลดตามสัดส่วน |
| **Pricing Layer** | ชั้นการคำนวณแต่ละขั้น (0-5) |
| **Pricing Context** | ข้อมูลแวดล้อม (เวลา, user, payment) ที่ใช้ประเมิน eligibility |
| **Flash Sale** | การปรับราคาพิเศษบนสินค้าตามช่วงเวลา (ไม่ใช่ coupon) |

---

## 14. Implementation Notes (สำหรับใช้ตอน build จริง)

> เอกสารนี้ไม่ระบุ tech แต่เมื่อจะ implement ควรเก็บหลักการต่อไปนี้:

1. **Engine ต้องเป็น pure function** — ไม่แตะ DB ไม่มี side effect รับ input → return output
2. **แยก domain layer ออกจาก framework** — domain code ไม่ควร import จาก HTTP/ORM/UI
3. **Test bottom-up** — เริ่มจาก primitive helpers → discount math → predicate → coupon eligibility → stacking → pricing engine (integration)
4. **Persist เป็น JSON** — predicates + stacking lists เก็บเป็น JSON ใน DB เพื่อ flexibility
5. **Audit trail** — เก็บ coupon usage log แยกตาราง (immutable) ใช้ใน accounting/refund
6. **Settlement** — รายงานต้องแยก funded-by ได้ทุกบาท (สำคัญสำหรับ multi-seller)

---

## 15. Related

- [flash-sale](./flash-sale.md) — price override (ไม่ใช่ coupon) ที่กระทบฐานราคาก่อน apply discount
- [cart](./cart.md) — pricing context (item lines, qty) ที่ engine ใช้
- [checkout](./checkout.md) — เป็น caller หลักที่ apply coupon
- [order](./order.md) — เก็บ usage log + breakdown ตอน confirm
- [settlement-payout](./settlement-payout.md) — `Funded by` ส่งผลต่อ payout ของแต่ละ store
- [transaction-ledger](./transaction-ledger.md) — coupon usage = financial event ที่ต้อง log
- [catalog](./catalog.md) — product/brand/category ที่ใช้ตรวจ entitlement