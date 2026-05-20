# AWS Infrastructure — เดิม (Reference)

> **Source:** AWS Pricing Calculator export dated 2024-01-23  
> **Region:** Asia Pacific (Singapore)  
> **ใช้เป็น:** Production infrastructure ของ PetPaw เดิมก่อน restructure

---

## Cost Summary

| | USD | บาท (~35฿/$) |
|--|----:|-------------:|
| Monthly | $854.53 | ~29,909 |
| 12 months | $10,254.36 | ~358,903 |

---

## Breakdown

| Service | Monthly (USD) | หน้าที่ |
|---------|-------------:|--------|
| Amazon EC2 (32 × t2.micro, On-Demand) | $408.26 | App servers หลัก |
| Amazon RDS for MySQL (db.m1.large, 100 GB, Single-AZ) | $264.80 | Database หลัก |
| Amazon MQ / RabbitMQ — 2 brokers (mq.t3.micro) | $98.11 | Message queue (production) |
| Amazon MQ / RabbitMQ — 1 broker (mq.t3.micro) | $49.05 | Message queue (staging/dev) |
| Amazon CloudWatch (24 metrics, 4 GB logs) | $10.02 | Monitoring & logging |
| Amazon MemoryDB for Redis (db.t4g.small × 2) | $10.59 | Caching |
| Amazon RDS for MySQL (db.m1.small, 30 GB, 10% util) | $9.25 | Database รอง / dev |
| Amazon Elastic Container Registry (20 GB) | $2.00 | Container image registry |
| Amazon API Gateway (REST) | $1.25 | API routing |
| AWS Fargate (32 tasks/day, 2 GB RAM) | $1.20 | Containerized background tasks |
| **Total** | **$854.53** | |

---

## รวม Infra ทั้งหมด (เดิม)

| Service | บาท/เดือน |
|---------|----------:|
| AWS (production) | ~29,909 |
| Digital Ocean | 7,000 |
| Microsoft Azure | 20,000 |
| Google Cloud | 1,000 |
| Vultr | 200 |
| **รวม** | **~58,109** |

เทียบกับใหม่ (Vercel + Neon): ~1,365 บาท/เดือน → **ประหยัด ~56,744 บาท/เดือน**
