# WenSync (WHMS) – Wenlock Hospital Management System

> A full‑stack hospital management application designed to streamline and digitize hospital operations for improved efficiency and patient care. WHMS integrates real‑time communication, role‑based access control (RBAC), and intelligent scheduling to address bottlenecks in hospital workflows.

<p align="center">
  <img alt="WenSync" src="./docs/cover.png" width="720" />
</p>

<p align="center">
  <a href="#tech-stack">Tech Stack</a> •
  <a href="#features">Features</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#quickstart">Quickstart</a> •
  <a href="#configuration">Configuration</a> •
  <a href="#core-domain-models">Domain Models</a> •
  <a href="#rbac--security">RBAC & Security</a> •
  <a href="#scheduling-engine">Scheduling Engine</a> •
  <a href="#api-surface">API Surface</a> •
  <a href="#testing--quality">Testing</a> •
  <a href="#deployment">Deployment</a> •
  <a href="#screenshots">Screenshots</a> •
  <a href="#roadmap">Roadmap</a> •
  <a href="#license">License</a>
</p>

---

## Why WenSync?

Hospitals struggle with siloed systems, manual coordination, and delays that affect patient outcomes and staff productivity. **WenSync (WHMS)** focuses on:

* **Real‑time coordination** between reception, nurses, doctors, lab, pharmacy, and billing.
* **Least‑friction scheduling** for doctors, operating rooms, and diagnostics.
* **Granular RBAC** powered by Firebase Auth & Firestore rules to protect PHI and enforce compliance‑aligned access.
* **Operational visibility** (queues, bed occupancy, critical alerts, turnaround times).

> ⚠️ **Disclaimer:** WenSync is a hackathon prototype focused on workflows and engineering rigor. It is **not** a medical device and is **not** HIPAA/NDHM certified. Do not use for real patient care without required audits and approvals.

---

## Tech Stack

* **Frontend & Fullstack:** Next.js (App Router) + TypeScript, Tailwind CSS, React Query (data), Zustand (state).
* **Backend APIs:** Next.js API routes, Socket.IO server for real‑time communication.
* **Database:** MongoDB (Atlas/local) for clinical & operational data.
* **Auth & RBAC:** Firebase Authentication + Firestore Security Rules.
* **DevOps:** Docker & Docker Compose, GitHub Actions (CI/CD), ESLint/Prettier, Jest, Playwright.

---

## Features

### 1) Patient & Visit Management

* Patient registration (MRN), demographics, contact & emergency details.
* OPD/IPD workflows, admissions, transfers, and discharge summaries.
* Clinical notes, vitals snapshots, diagnosis, prescriptions & medication history.

### 2) Appointment & Intelligent Scheduling

* Doctor rosters, clinic sessions, leave/holiday calendars.
* Slot generation with buffers, double‑booking guardrails, and **priority scoring** (triage level, age, comorbidities).
* OR/OT scheduling with conflict detection.

### 3) Real‑time Coordination

* **Live queues** for triage, pharmacy, lab, billing using Socket.IO rooms.
* **Secure messaging** (department channels + patient‑context threads).
* **Alerts** for emergencies, lab readiness, low inventory, bed availability changes.

### 4) Lab, Pharmacy, Billing

* Lab orders, status tracking, result upload.
* Pharmacy inventory (batch, expiry), e‑prescriptions, dispense logs.
* Billing items, insurance plans, invoices & payments.

### 5) RBAC, Audit & Compliance

* Roles: **Admin, Doctor, Nurse, Receptionist, LabTech, Pharmacist, Billing, Guest**.
* Firebase Auth → Role claims mapped to Firestore rules.
* Audit logs stored in MongoDB.

### 6) Dashboards & Metrics

* Bed occupancy, waiting times, clinic load, OR utilization.
* Export reports in CSV/Excel.

---

## Architecture

```
┌───────────────┐     HTTPS/WSS      ┌───────────────────────────┐
│   Next.js UI  │ ◀────────────────▶ │ Next.js API + Socket.IO   │
│ (App Router)  │                   │  (RBAC + Scheduling)       │
└──────┬────────┘                   └──────────┬────────────────┘
       │                                     │
       │                                     ▼
       │                              ┌───────────────┐
       │                              │   MongoDB     │
       │                              └───────────────┘
       │                                     │
       ▼                                     ▼
┌───────────────────────┐          ┌───────────────────────┐
│ Firebase Auth & Rules │          │   Firestore (RBAC)    │
└───────────────────────┘          └───────────────────────┘
```

* **API style:** REST (Next.js API routes) + WebSocket events.
* **Validation:** Zod schemas shared across client and server.

---

## Quickstart

### 1) Prerequisites

* Node.js ≥ 18
* Docker Desktop (optional)
* MongoDB Atlas (recommended)
* Firebase Project (Auth + Firestore enabled)

### 2) Clone & Install

```bash
git clone https://github.com/<you>/wensync-whms.git
cd wensync-whms
npm install
```

### 3) Environment Variables

`.env.local`

```env
NEXT_PUBLIC_API_URL=http://localhost:3000
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster0.mongodb.net/wensync
FIREBASE_API_KEY=your_api_key
FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
FIREBASE_PROJECT_ID=your_project_id
FIREBASE_STORAGE_BUCKET=your_project.appspot.com
FIREBASE_MESSAGING_SENDER_ID=...
FIREBASE_APP_ID=...
JWT_SECRET=supersecret_change_me
```

### 4) Run

```bash
npm run dev
```

Frontend & API: [http://localhost:3000](http://localhost:3000)

### 5) Seed Demo Data

```bash
npm run seed
```

Demo accounts will be auto‑created in Firebase Auth.

---

## Configuration

* **RBAC:** Firebase role claims (`customClaims`) configured in `auth.ts`.
* **Scheduling:** adjustable in `lib/scheduling.ts`.
* **Alerts:** thresholds in `lib/alerts.ts`.

---

## Core Domain Models

(Simplified)

**User**

```ts
{ uid, name, email, role, departmentId?, phone, active }
```

**Patient**

```ts
{ _id, mrn, name, dob, gender, contact, emergencyContact, allergies }
```

**Appointment**

```ts
{ _id, patientId, doctorId, start, end, status, triageLevel }
```

**Admission**

```ts
{ _id, patientId, ward, bedNo, status, admittedAt, dischargedAt }
```

**LabOrder, Prescription, Invoice** follow similar patterns stored in MongoDB.

---

## RBAC & Security

* Firebase Auth (email/password, Google, etc.).
* Role claims assigned via Firebase Admin SDK.
* Firestore security rules + server‑side checks.
* JWT secret for session validation.
* Audit logs persisted in MongoDB.

---

## Scheduling Engine

* Implements **priority‑based slot assignment**.
* Inputs: doctor availability, patient triage, room constraints.
* Heuristics: priority scoring + conflict detection.
* Runs as serverless functions triggered on appointment creation/update.

---

## API Surface (Next.js API Routes)

* `POST /api/auth/login`
* `GET /api/patients`
* `POST /api/patients`
* `GET /api/appointments`
* `POST /api/appointments`
* `PATCH /api/appointments/:id`
* `GET /api/billing/invoices`

**Socket.IO Events**

* `queue:update`
* `appointments:update`
* `alerts:new`
* `message:new`

---

## Testing & Quality

* **Unit/API:** Jest (`npm run test`)
* **E2E:** Playwright (`npm run test:e2e`)
* **Lint/Format:** ESLint + Prettier
* **CI/CD:** GitHub Actions workflow for lint → test → build → deploy.

---

## Deployment

**Option A – Vercel (recommended)**

* Next.js frontend & API auto‑deploy.
* Env vars set in Vercel dashboard.

**Option B – Docker Compose**

```bash
docker compose up -d
```

* Frontend/API: [http://localhost:3000](http://localhost:3000)
* MongoDB: docker volume

**Firebase**: RBAC & auth remain managed by Firebase project.

---

## Screenshots

* Dashboard (queues, occupancy)
* Patient profile & timeline
* Scheduling board
* Lab order lifecycle

```md
![Dashboard](./docs/dashboard.png)
![Scheduling](./docs/scheduling.png)
```

---

## Roadmap

* [ ] NDHM/ABDM compliant identifiers
* [ ] FHIR resource mapping
* [ ] MFA & SSO (OIDC)
* [ ] Offline‑first triage tablets
* [ ] HL7/HIS integrations
* [ ] Predictive wait‑time analytics

---

## Contributing

1. Fork repo & create branch: `feat/<short-name>`
2. Write tests, follow commit style `type(scope): subject`
3. Open PR with context/screenshots

---

## License

MIT © 2025 WenSync Team
