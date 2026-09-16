<div align="center">

<img src="https://img.shields.io/badge/Next.js-16-000000?style=for-the-badge&logo=nextdotjs&logoColor=white" />
<img src="https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
<img src="https://img.shields.io/badge/TypeScript-5-3178C6?style=for-the-badge&logo=typescript&logoColor=white" />
<img src="https://img.shields.io/badge/TailwindCSS-4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" />
<img src="https://img.shields.io/badge/AWS-S3-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" />
<img src="https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?style=for-the-badge&logo=jenkins&logoColor=white" />
<img src="https://img.shields.io/badge/PM2-Process%20Manager-2B037A?style=for-the-badge&logo=pm2&logoColor=white" />

# Competition Platform — Student Web Application

**A production-grade student-facing web application for a national academic competition — built for reliability, offline resilience, and a seamless exam experience under high-concurrency conditions.**

</div>

---

## Table of Contents

- [Project Overview](#project-overview)
- [Core Features](#core-features)
- [Technology Stack](#technology-stack)
- [Architecture Design](#architecture-design)
- [Offline Resilience & Exam Integrity](#offline-resilience--exam-integrity)
- [Authentication Flow](#authentication-flow)
- [File Upload Strategy](#file-upload-strategy)
- [Performance Optimizations](#performance-optimizations)
- [CI/CD Pipeline (Handled by DevOps Team)](#cicd-pipeline)
- [Project Structure](#project-structure)

---

## Project Overview

Production-grade student web application — engineering portfolio case study

This is a portfolio case study documenting the architecture, engineering decisions, and frontend patterns used to build a high-quality student-facing web application for a national academic competition platform.

The application was designed to handle student self-registration, OTP-based verification, admit card generation, multi-format live exam delivery, automated result display, and notice board access — all within a polished, mobile-friendly interface.

The production implementation is proprietary. This documentation presents the technical concepts and architectural patterns used in the project without exposing company-confidential source code or infrastructure details.

---

## Core Features

### 📝 Multi-Step Student Registration
- Guided multi-step registration flow with per-step form validation using **Zod schemas** and **React Hook Form**
- Live document upload directly to **AWS S3** via presigned URLs — student photo, birth certificate, and ID card
- Camera capture modal for in-browser photo capture without a native file picker
- Address selection hierarchy (Division → District → Upazila → Village) using Bangladesh address data
- Registration state persisted across page reloads using **Zustand + localStorage**, so partial progress is never lost on accidental navigation

### 🔐 Secure Authentication
- Mobile number and password login with **HTTP-only JWT cookie** management (no tokens in JavaScript memory)
- Automatic **silent token refresh** — a centralized API client intercepts 401 responses, refreshes the access token transparently, and replays the failed request; concurrent requests during refresh are queued and resolved together
- Forgot password flow with OTP delivery, verification, and password reset across a 3-step modal
- Persistent login state with hydration guard preventing redirect flicker on page reload

### 🪪 Digital Admit Card
- Auto-generated digital admit card with student details, venue information, and registration number
- **QR code** embedded in the admit card using `qrcode.react`
- Admit card exportable as a **PDF** via `jspdf` + `html-to-image` client-side rendering
- QR scan verification flow for administrators to validate student identity at the venue

### 📋 Notice Board
- Live notice board displaying competition announcements from the admin dashboard
- Real-time content updates with clean card-based layout

### 🧠 Multi-Format Practice Quiz (Demo Quiz)
- Students can attempt practice quizzes covering all six question formats before the live exam:
  - **MCQ** — tap to select one of four options
  - **Single Text** — free-text input with case-insensitive evaluation
  - **Single Letter** — single character input for fill-in-the-blank
  - **True/False Conditional** — True/False toggle with mandatory correction word when False
  - **Paragraph Correction** — interactive paragraph with highlighted error slots to fill
  - **Image Identification** — image displayed; student types the answer
- Per-section countdown timer with automatic section submission on expiry
- Instant local grading with correct/incorrect feedback shown after submission
- Encoded correct answer embedded in the quiz bundle (Base64) — grading happens entirely client-side with zero latency

### 🏆 Live Competition Exam
- Loads the complete exam bundle in a **single API request** (all sections + questions at once)
- Full exam state — current section, all answers, section timer state — persisted in **IndexedDB** via `idb`, surviving page refreshes and accidental tab closures without any data loss
- Section-level countdown timers that continue ticking accurately even after a page reload (timer end epoch is stored, not a running counter)
- One-time-only submission enforced both client-side and server-side
- Answer payload uploaded to **AWS S3** as a JSON audit record before submission to the backend
- Submission dispatched to the backend which queues it via AWS SQS for high-throughput processing
- Result screen with section-by-section breakdown of scores, correct/incorrect counts, and unattempted questions

### 👤 Student Profile
- View and edit profile details with optimistic UI update (local state updated immediately, synced to backend)
- Profile photo upload with S3 presigned PUT workflow

---

## Technology Stack

| Category | Technology | Purpose |
|---|---|---|
| **Framework** | Next.js 16 (App Router) | SSR/SSG, file-based routing, server components |
| **Language** | TypeScript 5 | End-to-end type safety across components and API contracts |
| **UI Library** | React 19 | Component model with concurrent rendering |
| **Styling** | Tailwind CSS 4 | Utility-first responsive design system |
| **Animation** | Motion (Framer Motion) | Page transitions, micro-interactions, exam UI animations |
| **State Management** | Zustand 5 | Lightweight global state with persistence middleware |
| **Form Handling** | React Hook Form + Zod | Performant forms with schema-driven validation |
| **Offline Storage** | IndexedDB (`idb`) | Durable exam session persistence across reloads |
| **QR Code** | `qrcode.react` | Admit card QR generation client-side |
| **PDF Export** | `jspdf` + `html-to-image` | Client-side admit card PDF generation |
| **Address Data** | `@bangladeshi/bangladesh-address` | Bangladesh divisional address hierarchy |
| **Icons** | Lucide React | Consistent icon system |
| **API Client** | Custom `fetch` wrapper | Typed requests, auto token refresh, S3 upload helpers |
| **File Upload** | AWS S3 Presigned PUT | Direct browser-to-S3 upload without backend proxy |
| **CI/CD** | Jenkins + PM2 + SSH | Automated build and zero-downtime deploy pipeline |
| **Process Manager** | PM2 | Node.js process management with auto-restart |

---

## Architecture Design

The application is built on the **Next.js App Router** with a clear separation between server-rendered public pages and client-side authenticated dashboard pages.

### Layered Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Routing Layer (App Router)                   │
│  Public Routes        │  Protected Dashboard Routes              │
│  /login  /register    │  /home  /quiz  /admit-card  /profile     │
│  /forgot-password     │  /notice-board  /scan-qr  /demo-quiz    │
└──────────────────────────────────────────────────────────────────┘
                                   │
┌──────────────────────────────────▼───────────────────────────────┐
│                       Component Layer                              │
│  Page Components  →  Feature Components  →  Shared UI Components  │
└──────────────────────────────────────────────────────────────────┘
                                   │
┌──────────────────────────────────▼───────────────────────────────┐
│                      State Management Layer                        │
│   Zustand Auth Store (persisted)  │  Transient File Store         │
│   Login session · Student profile │  Raw File refs (non-persisted)│
│   Registration flow · OTP state   │                               │
└──────────────────────────────────────────────────────────────────┘
                                   │
┌──────────────────────────────────▼───────────────────────────────┐
│                        Service / Utility Layer                     │
│  API Client (auto refresh)   │  S3 Upload Queue Service           │
│  Exam Quiz DB (IndexedDB)    │  Admit Card Generator (QR + PDF)  │
│  Form Schemas (Zod)          │  Bangladesh Address Helpers        │
└──────────────────────────────────────────────────────────────────┘
                                   │
                     Backend REST API  +  AWS S3
```

### Route Organization

| Route Group | Access | Description |
|---|---|---|
| `/` (root) | Public | Landing redirect |
| `/login` | Public | Student login page |
| `/register` | Public | Multi-step registration |
| `/forgot-password` | Public | OTP-based password reset |
| `/terms` | Public | Terms and conditions |
| `/welcome` | Public | Welcome/onboarding screen |
| `/(dashboard)/home` | Protected | Student home with active quiz and notices |
| `/(dashboard)/quiz` | Protected | Live exam entry and play flow |
| `/(dashboard)/demo-quiz` | Protected | Practice quiz flow |
| `/(dashboard)/admit-card` | Protected | Digital admit card with QR + PDF export |
| `/(dashboard)/profile` | Protected | View and edit student profile |
| `/(dashboard)/notice-board` | Protected | Competition announcements |
| `/(dashboard)/instruction` | Protected | Exam instructions screen |
| `/(dashboard)/scan-qr` | Protected | QR scanner for venue verification |

---

## Offline Resilience & Exam Integrity

This is the most critical engineering concern for a live exam application. A student losing their answers due to a page refresh or network drop during a 45-minute exam is unacceptable.

### IndexedDB-Based Exam State Persistence

The entire exam session is written to **IndexedDB** — the browser's client-side structured database — immediately and continuously. The key benefits:

- **Survives page refresh**: On reload, the app reads the existing exam state from IndexedDB and resumes exactly where the student left off — same section, same answers, same remaining timer
- **Timer accuracy**: Section countdown timers are stored as absolute epoch timestamps (`timerEndsAt = Date.now() + durationSeconds * 1000`), not relative countdowns. Even after a 2-minute reload, the timer picks up with the correct remaining time
- **Atomic answer saves**: Every answer interaction immediately writes the updated state to IndexedDB. No answer is ever held only in React component state
- **Section status tracking**: Each section transitions through `pending → active → submitted` states, preventing a student from going back to a submitted section

### Exam Data Flow

```
Bundle Load (single API call)
        │
        ▼
   Save to IndexedDB ◀──── Resume here on reload
        │
        ▼
Student answers questions
        │
   Each answer saved to IndexedDB immediately
        │
        ▼
Section timer expires OR student submits section
        │
   Section marked "submitted" in IndexedDB
        │
        ▼
All sections completed
        │
   Collect all answers from IndexedDB
        │
        ├──▶ Upload full answer JSON to AWS S3 (audit record)
        │
        └──▶ POST submission to backend (→ SQS → Database)
                │
           Clear IndexedDB exam state
                │
           Display result screen
```

---

## Authentication Flow

### Silent Token Refresh

The API client implements a robust token refresh strategy so students are never suddenly logged out mid-exam:

```
Any API request
      │
      ▼
  Response 401?
      │
   YES (and not already retrying)
      │
      ▼
  Is refresh already in progress?
   ├── YES → Queue this request; wait for refresh to complete → retry
   └── NO  → Start refresh
                │
          POST /auth/refresh (cookies sent automatically)
                │
         ┌──────┴──────┐
         │             │
       Success       Failure
         │             │
    Process queue   Redirect to /login
    Retry request   Clear auth state
```

No token is ever stored in JavaScript memory or localStorage — only HTTP-only cookies set by the server — making the auth system resilient to XSS attacks.

---

## File Upload Strategy

All file uploads (student photo, birth certificate, ID card, exam answer JSON) bypass the application server entirely using the **S3 Presigned PUT URL** pattern:

```
1. Browser → API Server:  Request presigned URL (filename + content-type)
2. API Server → Browser:  Return short-lived presigned PUT URL + permanent CDN URL
3. Browser → AWS S3:      PUT file binary directly (no backend involvement)
4. Browser → API Server:  Send permanent S3 URL as part of form/submission data
```

**Benefits:**
- Application server is never a bottleneck for file I/O
- Files go directly to S3 at full network speed
- Server memory is never consumed by file streams
- Works at any scale without infrastructure changes

For registration, an **S3 Upload Queue Service** manages sequential uploads and tracks completion state, ensuring all required files are uploaded before the registration request is sent.

---

## Performance Optimizations

| Optimization | Implementation | Benefit |
|---|---|---|
| **Single bundle fetch** | Entire quiz (all sections + questions) loaded in one HTTP call | Eliminates per-section network latency during exam |
| **Client-side grading** | Correct answers embedded in bundle as Base64-encoded JSON | Instant result display for practice quizzes, zero RTT |
| **Persistent login state** | Zustand with localStorage persistence + hydration guard | No auth flicker on reload; no unnecessary re-login |
| **IndexedDB for exam state** | Structured client-side DB vs. sessionStorage | Survives reload, larger quota, structured queries |
| **Form state isolation** | React Hook Form with Zod schema validation | Prevents unnecessary re-renders on field updates |
| **Token queue deduplication** | Concurrent 401 requests share a single refresh call | Never fires duplicate refresh requests |
| **Direct S3 upload** | Presigned PUT URLs bypass backend | Server stays stateless; upload speed is network-bound |
| **Motion animations** | Framer Motion with hardware-accelerated CSS transforms | Smooth 60fps transitions without layout thrash |

---

## CI/CD Pipeline (Handled by DevOps Team)

The project uses a **Jenkins declarative pipeline** deploying to a Linux VPS managed by **PM2**.

### Pipeline Stages (Handled by DevOps Team)

| Stage | Action |
|---|---|
| **Checkout** | Pull source from SCM |
| **Install Dependencies** | `npm ci` — clean, reproducible install from lockfile |
| **Build** | Fetch `.env.production` 
and run `next build` |
| **Provision Server** | Create deploy directory; set ownership on VPS |
| **Deploy Files** | `scp` `.next/`, `public/`, and config files to VPS; install production dependencies |
| **Restart PM2** | `pm2 restart` existing process or `pm2 start` if first deploy |
| **Health Check** | Poll the application URL up to 8 times (80s); fail build if unreachable |

## Project Structure

```
competition-frontend/
│
├── src/
│   ├── app/                        # Next.js App Router
│   │   ├── (dashboard)/            # Protected route group
│   │   │   └── [feature pages]/    # Home, quiz, admit card, profile, etc.
│   │   ├── [public pages]/         # Login, register, forgot password, terms
│   │   ├── layout.tsx              # Root layout with font loading
│   │   └── globals.css             # Global Tailwind base styles
│   │
│   ├── components/
│   │   ├── pages/                  # Full page-level feature components
│   │   │   ├── [Feature]/          # Scoped component folders per feature
│   │   │   └── ...
│   │   ├── common/                 # Shared layout and navigation components
│   │   └── ui/                     # Reusable primitive UI components
│   │                               # (Button, FormInput, FormSelect, FormUpload, etc.)
│   │
│   ├── services/                   # Typed API call functions per domain
│   │
│   ├── store/                      # Zustand global state stores
│   │                               # (Auth store with persistence middleware)
│   │
│   ├── schemas/                    # Zod validation schemas for all forms
│   │
│   ├── types/                      # Shared TypeScript type definitions
│   │
│   ├── config/                     # Environment variable configuration
│   │
│   └── utils/
│       ├── api.[ts]                # Central fetch wrapper with token refresh logic
│       ├── examQuizDb.[ts]         # IndexedDB interface for exam session persistence
│       ├── S3UploadQueueService    # Sequential S3 upload manager
│       ├── admitCardQr             # QR + PDF generation helpers
│       └── [data helpers]          # Address data, static content, formatters
│
├── public/                         # Static assets served at root
├── next.config.ts                  # Next.js configuration
├── tsconfig.json                   # TypeScript compiler options
├── postcss.config.mjs              # PostCSS / Tailwind pipeline
└── Jenkinsfile                     # Jenkins CI/CD pipeline definition
```

---

<div align="center">

*Engineered for reliability · Designed for students · Built for scale*

</div>
