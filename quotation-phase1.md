# เอกสารเสนอราคา — PetPaw E-Commerce Platform

| | |
|-|-|
| **โครงการ** | PetPaw E-Commerce Platform — Phase 1 |
| **วันที่** | 21 พฤษภาคม 2026 |
| **ระยะเวลา** | พฤษภาคม – กรกฎาคม 2026 (2 เดือน) |
| **เวอร์ชัน** | v1.0 |

---

## ภาพรวมโครงการ

พัฒนาระบบ E-Commerce สำหรับ PetPaw ประกอบด้วย Backoffice สำหรับจัดการสินค้า/คำสั่งซื้อ และ Mobile Application สำหรับลูกค้า (iOS & Android) พร้อมเชื่อมต่อระบบบริหารจัดการและ Payment

---

## ขอบเขตงาน (Scope of Work)

### Phase 1A — Planning & Setup (พ.ค. 2026)

| งาน | รายละเอียด |
|-----|-----------|
| Infrastructure & DevOps | ตั้งค่า Cloud Infrastructure, CI/CD Pipeline, Monitoring |
| Database Schema | ออกแบบโครงสร้างฐานข้อมูลรองรับ Multi-seller ในอนาคต |
| Authentication System | ระบบ Login ด้วยเบอร์โทร + OTP (ลูกค้า) / Email (Admin) |
| UI/UX Backoffice | ออกแบบ MVP Backoffice |
| 3rd Party Setup | เตรียม credentials และทดสอบ API ทุก service |

### Phase 1B — Development Month 1 (มิ.ย. 2026)

**Backoffice System**

| งาน | รายละเอียด |
|-----|-----------|
| Product Management | จัดการสินค้า, หมวดหมู่, Brand, รูปภาพ |
| Order Management | ดู/จัดการคำสั่งซื้อ, อัปเดตสถานะ |
| Inventory Management | ติดตาม Stock |
| Banner & Promotion | จัดการแบนเนอร์หน้าแรก |

**Mobile Application (iOS & Android)**

| งาน | รายละเอียด |
|-----|-----------|
| Register / Login | สมัครและ Login ด้วยเบอร์โทร + OTP |
| Home Page | หน้าแรก Banner, หมวดหมู่, สินค้าแนะนำ |
| Product Listing | แสดงสินค้า, กรองตามหมวดหมู่ |
| Search | ค้นหาสินค้า, แสดงผลการค้นหา |
| Product Detail | หน้ารายละเอียดสินค้า |
| Add to Cart | เพิ่มสินค้าเข้าตะกร้า |
| Promotion / Promo Code | กรอก Promo Code ส่วนลด |

**Integration**

| งาน | รายละเอียด |
|-----|-----------|
| Payment Gateway | เชื่อมต่อ Flash Pay (รับชำระเงิน) |
| Shipping | เชื่อมต่อ Flash Express (คำนวณค่าส่ง, จอง) |
| OMS | เชื่อมต่อ Zort (Push Order, Sync Stock) |
| Search Engine | ติดตั้งและตั้งค่าระบบค้นหา |
| Push Notification | ตั้งค่า OneSignal (แจ้งเตือน Mobile) |

### Phase 1C — Development Month 2 (ก.ค. 2026)

**Mobile Application**

| งาน | รายละเอียด |
|-----|-----------|
| Checkout Flow | กระบวนการสั่งซื้อ, เลือกที่อยู่, ยืนยัน |
| Address Management | จัดการที่อยู่จัดส่ง |
| Order Tracking & History | ติดตามสถานะพัสดุ, ประวัติคำสั่งซื้อ |
| Customer Profile | ข้อมูลส่วนตัว, Settings, Logout |

**Integration**

| งาน | รายละเอียด |
|-----|-----------|
| Email Notification | ส่งอีเมลยืนยันคำสั่งซื้อ (Mailgun) |

**Quality Assurance**

| งาน | รายละเอียด |
|-----|-----------|
| UAT | ทดสอบระบบร่วมกับทีม PetPaw |
| Bug Fix | แก้ไขข้อผิดพลาดที่พบใน UAT |
| Deployment | นำขึ้น Production (Web + App Store / Play Store) |

---

## ราคาและเงื่อนไขการชำระเงิน

### ราคารวม

> **380,000 บาท** (ไม่รวม VAT 7%)

### Milestone การชำระเงิน

| Milestone | เงื่อนไข | จำนวนเงิน |
|-----------|---------|----------:|
| 1 — เริ่มงาน | ลงนามสัญญาและชำระ Deposit | **152,000 บาท** (40%) |
| 2 — ส่ง UAT | ส่งมอบระบบให้ทีม PetPaw ทดสอบ | **114,000 บาท** (30%) |
| 3 — Launch | Go-live บน Production สำเร็จ | **114,000 บาท** (30%) |

---

## สิ่งที่ไม่รวมในขอบเขต (Out of Scope)

- ค่าบริการ 3rd party รายเดือน (Vercel, Neon, Flash Pay, Mailgun, Apitel, OneSignal ฯลฯ) — PetPaw ชำระโดยตรง
- ค่า Apple Developer Account / Google Play Console
- Feature ใน Phase 2 (Seller Portal, Full Coupon System, Dashboard, Chat ฯลฯ)
- Post-launch Support / Maintenance — คิดเป็น Retainer แยกต่างหาก
- UI/UX Design ต้นฉบับ (กรณีไม่มี Figma ส่งมอบจาก PetPaw)

---

## เงื่อนไขและข้อตกลง

1. **Design ต้อง Approve ก่อน Dev เริ่ม** — การเปลี่ยน Design หลังจาก Dev เริ่มแล้วคิดเป็น Change Request
2. **3rd Party Credentials** — PetPaw ต้องส่งมอบ API Key / Merchant Account ทุก service ก่อนสิ้นสุด Planning Phase
3. **PetPaw Dev** — Developer ฝั่ง PetPaw ต้อง Available full-time ตลอด Project
4. **Change Request** — งานที่ไม่อยู่ใน Scope จะประเมินราคาและ Timeline แยก
5. **UAT Period** — PetPaw มีเวลา UAT 1 สัปดาห์ หากเกินอาจกระทบ Timeline
6. **รับประกัน** — Bug ที่เกิดจากการ Development รับประกัน 30 วันหลัง Launch

---

## ติดต่อ

| | |
|-|-|
| **Email** | o.muangmoon@gmail.com |
| **โครงการ** | PetPaw Phase 1 |

---

*เอกสารนี้มีอายุ 30 วันนับจากวันที่ออก*
