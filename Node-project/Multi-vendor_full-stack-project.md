<div align="center">

# 🛒 Multi-Vendor E-Commerce Platform

**A full-scale, production-grade multi-vendor e-commerce ecosystem**
built for a confidential client under a company NDA.

<br/>

![Node.js](https://img.shields.io/badge/Node.js-20-339933?style=for-the-badge&logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express.js-4.x-000000?style=for-the-badge&logo=express&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-14-000000?style=for-the-badge&logo=next.js&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![MongoDB](https://img.shields.io/badge/MongoDB-8-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/TailwindCSS-3.4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![AWS S3](https://img.shields.io/badge/AWS_S3-Storage-FF9900?style=for-the-badge&logo=amazon-s3&logoColor=white)

<br/>

> ⚠️ **Confidential Project** — Source code, project name, client details, and internal data are protected under a company NDA and are not publicly disclosed. This document exists solely to demonstrate professional implementation skills and technical experience.

</div>

---

## 📋 Table of Contents

- [📌 Project Overview](#-project-overview)
- [🏗️ System Architecture](#️-system-architecture)
- [📦 Application Modules](#-application-modules)
  - [1. 🔵 Backend REST API](#1--backend-rest-api)
  - [2. 🟢 Customer Web App](#2--customer-web-app)
  - [3. 🟠 Seller Portal](#3--seller-portal)
  - [4. 🔴 Admin Dashboard](#4--admin-dashboard)
- [🔧 DevOps & Infrastructure](#-devops--infrastructure-maintain-by-devops-team)
  - [CI/CD Pipeline — Jenkins](#cicd-pipeline--jenkins)
  - [Server Infrastructure](#server-infrastructure-maintain-by-devops-team)
- [🛠️ Full Technology Stack](#️-full-technology-stack)
  - [Backend](#backend)
  - [Frontend](#frontend-all-three-apps)
  - [DevOps & Tooling](#devops--tooling)
- [🚀 Key Engineering Highlights](#-key-engineering-highlights)
- [👨‍💻 My Role & Contribution](#-my-role--contribution)
- [📄 Confidentiality Notice](#-confidentiality-notice)

---

## 📌 Project Overview

This is a **production-deployed, full-stack multi-vendor e-commerce platform** developed as part of a commercial product for a technology company. The system comprises **four independent applications** — a RESTful backend API, a customer-facing storefront, a seller management portal, and an admin control panel — all communicating through a centralized API and deployed to a cloud VPS with automated CI/CD pipelines.

The platform enables multiple independent sellers to list and sell products through a unified marketplace, while customers can browse, purchase, and track orders seamlessly.

---

## 🏗️ System Architecture

The platform is built on a **microservices-inspired monorepo architecture** with four independently deployed applications:

```
┌────────────────────────────────────────────────────────┐
│                   PRODUCTION SERVERS                   │
│               (VPS · Nginx · PM2 · Jenkins)            │
└────────────┬──────────────┬────────────┬───────────────┘
             │              │            │
    ┌────────▼──┐   ┌───────▼──┐  ┌─────▼──────┐
    │  Customer │   │  Seller  │  │   Admin    │
    │  Web App  │   │  Portal  │  │  Dashboard │
    │ (Next.js) │   │(Next.js) │  │ (Next.js)  │
    └─────┬─────┘   └────┬─────┘  └─────┬──────┘
          │              │              │
          └──────────────▼──────────────┘
                   ┌─────────────┐
                   │  REST API   │
                   │  (Express)  │
                   └──────┬──────┘
              ┌───────────┼───────────┐
         ┌────▼────┐ ┌────▼────┐ ┌───▼────┐
         │ MongoDB │ │  Redis  │ │ AWS S3 │
         └─────────┘ └─────────┘ └────────┘
```

---

## 📦 Application Modules

### 1. 🔵 Backend REST API
> **Technology**: Node.js · Express.js · MongoDB (Mongoose) · Redis · AWS S3

The central API server that powers all three frontend applications with a well-structured RESTful architecture.

**Key Implementations:**
- **JWT-based Authentication** with `jose` and `jsonwebtoken` — stateless auth for customers, sellers, and admins
- **Role-Based Access Control (RBAC)** — separate auth flows and middleware guards per user role
- **Rate Limiting** using `rate-limiter-flexible` backed by Redis to prevent API abuse
- **File Uploads** via `multer` + `multer-s3` directly streaming to **AWS S3** buckets
- **Redis Caching** with `ioredis` for performance-sensitive endpoints
- **Push Notifications** via **Firebase Admin SDK** (FCM) for real-time alerts
- **Email Service** using `nodemailer` for transactional emails (order confirmation, OTP, etc.)
- **Swagger API Documentation** with auto-generation using `swagger-autogen` + `swagger-ui-express`
- **Performance Monitoring** via **New Relic APM** integration
- **Structured Logging** with `winston` (log rotation, error tracking)
- **Courier Integration** — third-party courier API integration for delivery tracking
- **Payment Gateway** integration for processing online payments

---

### 2. 🟢 Customer Web App
> **Technology**: Next.js 14 · React 18 · TailwindCSS · Redux Toolkit · React Query · NextAuth

The customer-facing storefront where users can browse products, manage their cart, and place orders.

**Key Features:**
- 🛍️ **Product Discovery** — multi-level category browsing, search with debounce, and advanced filters
- 🖼️ **Image Zoom & Gallery** — product image magnification with `react-image-magnify`
- 🛒 **Cart & Wishlist** — persisted state using Redux + `redux-persist`
- 💳 **Checkout Flow** — multi-step checkout with address management and payment integration
- 📦 **Order Tracking** — real-time order status with courier tracking integration
- 🎟️ **Coupon & Deals** — promo code application and campaign-based discount rendering
- 📲 **Push Notifications** — Firebase Cloud Messaging (FCM) for order updates
- 🔐 **Authentication** — NextAuth with JWT sessions and OTP-based flows
- 📊 **Analytics Charts** — ApexCharts for customer order history visualization
- 🔄 **Server-State Management** — TanStack React Query for efficient API caching and revalidation
- 📝 **Rich Text Reviews** — React Quill-based product review editor
- 🎠 **Promotional Banners** — Swiper.js for dynamic homepage carousels
- ⚡ **Performance** — Turbopack dev server, Next.js Image optimization with `sharp`

---

### 3. 🟠 Seller Portal
> **Technology**: Next.js 14 · React 18 · TailwindCSS · Redux Toolkit · React Query · Socket.io · Google Maps API

A full-featured dashboard for registered sellers to manage their store, products, and orders.

**Key Features:**
- 🏪 **Store Management** — seller profile, branding, and store settings
- 📦 **Product Management** — create/edit/delete products with multi-image upload, variants, rich description editor
- 🖼️ **Image Cropping** — built-in image editor using `react-easy-crop` before upload
- 📋 **Order Management** — accept/reject/fulfill orders with courier dispatch workflow
- 📊 **Analytics Dashboard** — sales charts, revenue reports, and performance metrics via ApexCharts
- 💰 **Cash Management** — earnings, withdrawal requests, and transaction history
- 🔔 **Real-Time Notifications** — Socket.io client for instant order alerts and live updates
- 📍 **Location Services** — Google Maps API integration for delivery zone and address picking
- ⏱️ **Countdown Timers** — `react-countdown` for flash sales and limited-time deals
- 🔐 **Authentication** — NextAuth with secure session management
- ✅ **Form Validation** — React Hook Form + Zod schema validation
- 🎨 **Color Picker** — `react-colorful` for product variant customization

---

### 4. 🔴 Admin Dashboard
> **Technology**: Next.js 14 · React 18 · TailwindCSS · Redux Toolkit · React Query · Framer Motion

A comprehensive admin control panel for platform administrators with full oversight across the marketplace.

**Key Features:**
- 👥 **User Management** — manage customers and sellers (ban, verify, view activity)
- ✅ **Seller Onboarding & Verification** — review and approve/reject seller registration requests
- 📦 **Product Oversight** — review, approve or remove any product across the marketplace
- 🗂️ **Category Management** — hierarchical parent/sub/category management with icon uploads
- 🎟️ **Coupon & Campaign Control** — create platform-wide coupons, manage time-limited campaigns
- 📰 **Homepage Section Builder** — configure dynamic homepage sections and featured products
- 💸 **Financial Management** — seller payout approvals and cash management oversight
- 📊 **Business Analytics** — platform-wide revenue, orders, and seller performance dashboards
- 🔔 **Notification Broadcasts** — send targeted push notifications to customers/sellers
- 🎞️ **Framer Motion Animations** — smooth page transitions and interactive UI animations
- 🧩 **Data Tables** — `react-data-table-component` + MUI DataGrid for large dataset management

---

## 🔧 DevOps & Infrastructure (Maintain by DevOps Team)

### CI/CD Pipeline — Jenkins
Each application has its own **Jenkins pipeline** (`Jenkinsfile`) with:
- ✅ Automated dependency installation with `pnpm`
- ✅ Build artifact creation and packaging
- ✅ VPS environment provisioning (Node.js, PM2, Nginx, Redis, Certbot — auto-detected & installed if missing)
- ✅ PM2 process restart with zero-downtime deployment strategy
- ✅ Post-deployment health check with retry logic
- ✅ Pipeline workspace cleanup on completion

### Server Infrastructure (Maintain by DevOps Team)

| Component | Technology |
|-----------|-----------|
| **Process Manager** | PM2 with `ecosystem.config.js` |
| **Reverse Proxy** | Nginx |
| **SSL** | Let's Encrypt via Certbot |
| **Cloud VPS** | Hostinger VPS |
| **Object Storage** | AWS S3 (via `@aws-sdk/client-s3` + `multer-s3`) |
| **Database** | MongoDB Atlas / Self-hosted |
| **Caching** | Redis (self-hosted on VPS) |
| **Monitoring** | New Relic APM |
| **Package Manager** | pnpm (all apps) |

---

## 🛠️ Full Technology Stack

### Backend

| Category | Technology |
|----------|-----------|
| Runtime | Node.js v20 |
| Framework | Express.js 4.x |
| Database | MongoDB 8 + Mongoose ODM |
| Cache | Redis + ioredis |
| Authentication | JWT · jose · bcryptjs · express-session |
| File Storage | AWS S3 (multer-s3) |
| Email | Nodemailer |
| Push Notifications | Firebase Admin SDK (FCM) |
| API Docs | Swagger (swagger-autogen + swagger-ui-express) |
| Monitoring | New Relic APM |
| Logging | Winston |
| Rate Limiting | rate-limiter-flexible |
| Template Engine | EJS (for email templates) |

### Frontend (All Three Apps)

| Category | Technology |
|----------|-----------|
| Framework | Next.js 14.2 (App Router) |
| UI Library | React 18 |
| Styling | TailwindCSS 3.4 |
| State Management | Redux Toolkit + redux-persist |
| Server State | TanStack React Query v5 |
| Authentication | NextAuth v4 |
| Forms | React Hook Form + Zod |
| HTTP Client | Axios |
| Charts | ApexCharts + react-apexcharts |
| Animations | Framer Motion |
| Push Notifications | Firebase Cloud Messaging (FCM) |
| Real-Time | Socket.io Client |
| Maps | Google Maps API (@react-google-maps/api) |
| Rich Text | React Quill |
| Image Tools | react-easy-crop · react-image-magnify |
| Tables | MUI DataGrid · react-data-table-component |
| Carousels | Swiper.js |
| Alerts | SweetAlert2 · react-toastify |
| Icons | Lucide React |
| Date Utils | date-fns · dayjs |

### DevOps & Tooling

| Category | Technology |
|----------|-----------|
| CI/CD | Jenkins (Declarative Pipeline) |
| Process Manager | PM2 |
| Reverse Proxy | Nginx |
| SSL | Let's Encrypt (Certbot) |
| VPS | Hostinger |
| Storage | AWS S3 |
| Monitoring | New Relic APM |
| Package Manager | pnpm |
| Linting | ESLint |

---

## 🚀 Key Engineering Highlights

| Skill Area | What Was Implemented |
|------------|---------------------|
| **Full-Stack Architecture** | Designed and implemented a 4-app system with a shared REST API, consistent auth strategy, and role-based access |
| **Authentication & Security** | JWT + NextAuth sessions, role-based middleware guards, rate limiting, bcrypt password hashing, API access key protection |
| **Real-Time Features** | Socket.io for live order notifications on the seller dashboard |
| **Cloud & Storage** | AWS S3 streaming uploads via multer-s3, Cloudflare CDN URL integration |
| **Push Notifications** | Firebase Cloud Messaging (FCM) across all three frontend apps + Firebase Admin SDK on backend |
| **CI/CD Automation** | Full Jenkins pipeline per app — build → package → SSH deploy → health check |
| **Performance** | Redis caching, TanStack React Query for client-side caching, Next.js Image optimization, Turbopack dev |
| **Monitoring** | New Relic APM integrated on the production API |
| **API Documentation** | Swagger auto-generated docs from Express routes |
| **3rd Party Integrations** | Courier APIs, Payment Gateway, Firebase, New Relic APM |

---

## 👨‍💻 My Role & Contribution

This project was developed as part of a **professional employment** engagement at a technology company. My contributions included:

- 🔨 Full-stack feature development across all four applications
- 🧠 Architecture decisions for API design, data modeling, and state management
- 🔐 Implementing security layers — authentication, authorization, and rate limiting
- 📦 Building the product, order, seller, and coupon management flows end-to-end
- 📊 Integrating third-party services (AWS S3, Firebase, Courier APIs, Payment Gateway)

---

## 📄 Confidentiality Notice

> 🔒 **This project is proprietary and confidential.**
>
> The source code, client identity, database schemas, API keys, environment configurations, and all internal business logic are protected under a company Non-Disclosure Agreement (NDA). No source code or sensitive data is disclosed in this document.
>
> This README is intended **solely as a professional portfolio reference** to demonstrate implementation skills and technical experience to potential employers.

---

<div align="center">

**Built with passion for scalable, production-grade Junior software engineering.**

</div>
