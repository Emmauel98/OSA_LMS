
# OSA LMS — Backend API

**Oluwatosin Success Academy Learning Management System**
A production-grade REST API built with Node.js, Express, and TypeScript.

---

## Table of Contents

1. [Tech Stack](#tech-stack)
2. [Project Structure](#project-structure)
3. [Setup & Installation](#setup--installation)
4. [Environment Variables](#environment-variables)
5. [Running the Server](#running-the-server)
6. [Database Seeding](#database-seeding)
7. [Running Tests](#running-tests)
8. [API Overview](#api-overview)
9. [Authentication Flow](#authentication-flow)
10. [Role Permissions Matrix](#role-permissions-matrix)
11. [Default Passwords](#default-passwords)
12. [Payment Flow](#payment-flow)
13. [Excel Upload Format](#excel-upload-format)
14. [Security Features](#security-features)
15. [Deployment Notes](#deployment-notes)

---

## Tech Stack

| Layer         | Technology                                        |
|---------------|---------------------------------------------------|
| Runtime       | Node.js 18+                                       |
| Framework     | Express.js 4                                      |
| Language      | TypeScript 5                                      |
| Database      | MongoDB + Mongoose 8                              |
| Auth          | JWT (access + refresh token pattern)              |
| Payments      | Paystack (initialize + webhook verification)      |
| File Upload   | Multer (images + Excel)                           |
| Excel         | ExcelJS (read/write .xlsx)                        |
| PDF Receipts  | PDFKit                                            |
| Validation    | express-validator                                 |
| Security      | Helmet, express-rate-limit, express-mongo-sanitize, CORS |
| Logging       | Winston                                           |
| Testing       | Jest + ts-jest + Supertest + mongodb-memory-server |
| API Docs      | Swagger UI (swagger-jsdoc + swagger-ui-express)   |

---

## Project Structure

```
osa-lms/
├── src/
│   ├── config/
│   │   ├── database.ts        MongoDB connection
│   │   ├── migrate.ts         Index sync migration script
│   │   └── seed.ts            Creates default admin account
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── user.controller.ts
│   │   ├── result.controller.ts
│   │   ├── payment.controller.ts
│   │   ├── attendance.controller.ts
│   │   └── comment.controller.ts
│   ├── middleware/
│   │   ├── auth.ts            protect() + authorize() guards
│   │   ├── error.ts           Global error handler + AppError class
│   │   ├── upload.ts          Multer (pictures + Excel files)
│   │   └── validate.ts        express-validator runner
│   ├── models/
│   │   ├── User.ts            All roles in one collection
│   │   ├── Class.ts
│   │   ├── Result.ts
│   │   ├── Payment.ts
│   │   ├── Attendance.ts
│   │   └── Misc.ts            FeeConfig, Comment, SchoolCalendar
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── user.routes.ts
│   │   └── index.ts           Results, Payments, Attendance, Comments
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── user.service.ts
│   │   ├── result.service.ts
│   │   ├── payment.service.ts
│   │   └── attendance.service.ts
│   ├── types/index.ts         All interfaces, enums, Express augmentation
│   ├── utils/
│   │   ├── jwt.ts
│   │   ├── logger.ts
│   │   ├── password.ts
│   │   └── response.ts
│   ├── validators/index.ts    All express-validator chains
│   ├── app.ts                 Express app (middleware + routes)
│   └── server.ts              Entry point + graceful shutdown
├── tests/
│   ├── helpers.ts             In-memory DB setup + factory helpers
│   ├── unit/
│   │   ├── password.test.ts
│   │   ├── jwt.test.ts
│   │   ├── user.model.test.ts
│   │   └── result.model.test.ts
│   └── integration/
│       ├── auth.test.ts
│       ├── users.test.ts
│       ├── results.test.ts
│       └── attendance-comments.test.ts
├── uploads/                   Runtime upload dir (gitignored)
├── logs/                      Winston log files (gitignored)
├── .env                       Local env vars (gitignored)
├── .env.example               Safe template to commit
├── .eslintrc.json
├── .gitignore
├── jest.config.ts
├── package.json
└── tsconfig.json
```

---

## Setup & Installation

### Prerequisites

- Node.js 18+
- MongoDB 6+ (local or Atlas)
- npm 9+

### 1. Clone and install

```bash
git clone <your-repo-url>
cd osa-lms
npm install
```

### 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env` and update at minimum:
- `MONGODB_URI`
- `JWT_SECRET` and `JWT_REFRESH_SECRET` (use long random strings)
- `PAYSTACK_SECRET_KEY` and `PAYSTACK_WEBHOOK_SECRET`

### 3. Seed database

```bash
npm run seed
```

Creates the default admin using `ADMIN_EMAIL` / `ADMIN_PASSWORD` from `.env`.
**Change the password immediately after first login.**

### 4. Sync indexes

```bash
npm run migrate
```

---

## Environment Variables

| Variable                 | Required | Description                                        |
|--------------------------|----------|----------------------------------------------------|
| `NODE_ENV`               | Yes      | development / production / test                    |
| `PORT`                   | Yes      | HTTP port (default 5000)                           |
| `MONGODB_URI`            | Yes      | MongoDB connection string                          |
| `MONGODB_URI_TEST`       | Yes      | Separate DB used during tests                      |
| `JWT_SECRET`             | Yes      | Access token signing secret (min 32 chars)         |
| `JWT_EXPIRES_IN`         | Yes      | Access token TTL (e.g. 7d)                         |
| `JWT_REFRESH_SECRET`     | Yes      | Refresh token signing secret (min 32 chars)        |
| `JWT_REFRESH_EXPIRES_IN` | Yes      | Refresh token TTL (e.g. 30d)                       |
| `BCRYPT_ROUNDS`          | Yes      | bcrypt cost factor (12 for production)             |
| `PAYSTACK_SECRET_KEY`    | Yes      | sk_live_... or sk_test_...                         |
| `PAYSTACK_WEBHOOK_SECRET`| Yes      | From Paystack dashboard webhook settings           |
| `MAX_FILE_SIZE`          | No       | Upload limit in bytes (default 10485760 = 10MB)    |
| `RATE_LIMIT_WINDOW_MS`   | No       | Rate limit window ms (default 900000 = 15 min)     |
| `RATE_LIMIT_MAX`         | No       | Max requests per window (default 100)              |
| `FRONTEND_URL`           | No       | CORS allowed origin (default *)                    |
| `LOG_LEVEL`              | No       | Winston log level (default info)                   |
| `ADMIN_EMAIL`            | Seed     | Used by seed script only                           |
| `ADMIN_PASSWORD`         | Seed     | Used by seed script only                           |

---

## Running the Server

```bash
# Development with hot reload
npm run dev

# Production
npm run build
npm start
```

- Server: `http://localhost:5000`
- Swagger UI: `http://localhost:5000/api/docs`
- Health check: `http://localhost:5000/api/health`

---

## Database Seeding

```bash
npm run seed
```

Creates one admin account from `.env` values. Safe to run multiple times — skips if an admin already exists.

---

## Running Tests

```bash
# All tests
npm test

# With coverage report
npm run test:coverage

# Unit tests only
npm run test:unit

# Integration tests only
npm run test:integration
```

Uses `mongodb-memory-server` — no external MongoDB connection required for tests.

---

## API Overview

Base URL: `http://localhost:5000/api/v1`

### Auth

| Method | Endpoint      | Access | Description                          |
|--------|---------------|--------|--------------------------------------|
| POST   | /auth/login   | Public | Login — returns access + refresh tokens |
| POST   | /auth/refresh | Public | Exchange refresh token for new access token |
| GET    | /auth/me      | All    | Get own profile                      |

### Users

| Method | Endpoint                           | Access        | Description                     |
|--------|------------------------------------|---------------|---------------------------------|
| POST   | /users/students                    | Admin         | Create student account          |
| POST   | /users/parents                     | Admin         | Create parent account           |
| POST   | /users/teachers                    | Admin         | Create teacher account          |
| POST   | /users/admins                      | Admin         | Create admin account            |
| GET    | /users/role/:role                  | Admin         | List users by role (paginated)  |
| GET    | /users/:id                         | Admin         | Get user by ID                  |
| PATCH  | /users/:id/status                  | Admin         | Activate / deactivate / lock    |
| POST   | /users/link                        | Admin         | Link student to parent          |
| PATCH  | /users/:id/picture                 | Admin/Teacher | Upload picture for any user     |
| PATCH  | /users/me/picture                  | All           | Upload own picture              |
| GET    | /users/me/children                 | Parent        | Get own linked children         |
| GET    | /users/parents/:parentId/children  | Admin         | Get children of any parent      |

### Classes

| Method | Endpoint                          | Access        | Description          |
|--------|-----------------------------------|---------------|----------------------|
| POST   | /users/classes                    | Admin         | Create class         |
| GET    | /users/classes/all                | Admin/Teacher | List all classes     |
| PATCH  | /users/classes/:id                | Admin         | Update class         |
| GET    | /users/classes/:classId/students  | Admin/Teacher | Students in a class  |

### Results

| Method | Endpoint                  | Access         | Description                           |
|--------|---------------------------|----------------|---------------------------------------|
| POST   | /results                  | Admin/Teacher  | Create single result                  |
| POST   | /results/upload           | Admin/Teacher  | Bulk upload from Excel                |
| GET    | /results/template         | Admin/Teacher  | Download Excel template               |
| GET    | /results/export           | Admin/Teacher  | Export class results as Excel         |
| GET    | /results/student/:id      | All            | Get results (filtered by role)        |
| PATCH  | /results/verify           | Admin          | Verify results (bulk)                 |
| PATCH  | /results/archive          | Admin          | Archive results (bulk)                |
| PATCH  | /results/:id              | Admin/Teacher  | Update a result                       |
| DELETE | /results/:id              | Admin          | Delete a result                       |

### Payments

| Method | Endpoint                  | Access              | Description                        |
|--------|---------------------------|---------------------|------------------------------------|
| POST   | /payments/initialize      | Student/Parent      | Initialize Paystack transaction    |
| GET    | /payments/verify/:ref     | All                 | Verify payment by reference        |
| POST   | /payments/webhook         | Paystack (unsigned) | Paystack event receiver            |
| PATCH  | /payments/:id/verify      | Admin               | Manually verify payment            |
| GET    | /payments/my              | Student/Parent/Admin| Payment history + summary          |
| GET    | /payments/by-class        | Admin               | Payments grouped by class          |
| GET    | /payments/:id/receipt     | All                 | Download PDF receipt               |
| POST   | /payments/fee-config      | Admin               | Set semester fee amount            |
| GET    | /payments/fee-config      | All                 | Get fee config                     |

### Attendance

| Method | Endpoint                      | Access              | Description                     |
|--------|-------------------------------|---------------------|---------------------------------|
| POST   | /attendance                   | Teacher             | Mark attendance for class       |
| GET    | /attendance                   | Admin/Teacher       | Get attendance by class + date  |
| GET    | /attendance/student/:id       | All                 | Student attendance summary      |
| POST   | /attendance/calendar          | Admin               | Create/update school calendar   |
| GET    | /attendance/calendar          | All                 | Get calendars                   |

### Comments

| Method | Endpoint              | Access       | Description                  |
|--------|-----------------------|--------------|------------------------------|
| POST   | /comments             | Parent       | Send message to admin        |
| GET    | /comments             | Admin/Parent | Get comments (role-filtered) |
| PATCH  | /comments/:id/reply   | Admin        | Reply to a parent message    |

---

## Authentication Flow

```
1.  POST /auth/login  →  { accessToken (7d), refreshToken (30d) }
2.  All protected requests: Authorization: Bearer <accessToken>
3.  On 401: POST /auth/refresh { refreshToken }  →  { accessToken }
4.  On refresh 401: user must log in again
```

---

## Role Permissions Matrix

| Feature                         | Admin | Teacher | Student | Parent |
|---------------------------------|-------|---------|---------|--------|
| Create user accounts            | ✅    | ❌      | ❌      | ❌     |
| Deactivate / lock accounts      | ✅    | ❌      | ❌      | ❌     |
| Create / modify results         | ✅    | ✅      | ❌      | ❌     |
| Delete results                  | ✅    | ❌      | ❌      | ❌     |
| Verify results                  | ✅    | ❌      | ❌      | ❌     |
| Archive results                 | ✅    | ❌      | ❌      | ❌     |
| View student results            | ✅    | ✅      | ✅ *    | ✅ **  |
| Make payments                   | ❌    | ❌      | ✅      | ✅     |
| View payment history            | ✅    | ❌      | ✅      | ✅     |
| Set semester fees               | ✅    | ❌      | ❌      | ❌     |
| Manually verify payments        | ✅    | ❌      | ❌      | ❌     |
| Mark attendance                 | ❌    | ✅      | ❌      | ❌     |
| View attendance                 | ✅    | ✅      | ✅      | ✅     |
| Create school calendar          | ✅    | ❌      | ❌      | ❌     |
| Message admin                   | ❌    | ❌      | ❌      | ✅     |
| Reply to parent messages        | ✅    | ❌      | ❌      | ❌     |
| Upload/download Excel results   | ✅    | ✅      | ❌      | ❌     |

\* Student: only if fees are fully paid (verified payments > 0)
\** Parent: only admin-verified results

---

## Default Passwords

| Role    | Formula                        | Example for "Emeka Obi"       |
|---------|--------------------------------|-------------------------------|
| Student | Surname (first word)           | `Emeka`                       |
| Parent  | Surname + year of registration | `Ngozi2024`                   |
| Teacher | `OSA` + Surname                | `OSAIbrahim`                  |
| Admin   | Set explicitly at creation     | As provided                   |

Students and parents cannot change their passwords (by design per spec). The `defaultPassword` is returned in the creation response so admin can communicate it to the user securely.

---

## Payment Flow

```
Student/Parent                 OSA Backend                   Paystack
     │                              │                              │
     │  POST /payments/initialize   │                              │
     │─────────────────────────────▶│                              │
     │                              │  POST /transaction/initialize│
     │                              │─────────────────────────────▶│
     │                              │◀─────────────────────────────│
     │◀─────────────────────────────│  { authorizationUrl, ref }   │
     │                              │                              │
     │  Redirect to authorizationUrl│                              │
     │────────────────────────────────────────────────────────────▶│
     │              User completes payment on Paystack              │
     │◀────────────────────────────────────────────────────────────│
     │                              │                              │
     │                              │  POST /payments/webhook      │
     │                              │◀─────────────────────────────│
     │                              │  Verify HMAC-SHA512          │
     │                              │  Update status → verified    │
     │                              │  Generate PDF receipt        │
     │                              │                              │
     │  GET /payments/verify/:ref   │                              │
     │─────────────────────────────▶│                              │
     │◀─────────────────────────────│                              │
     │  { payment, receiptUrl }     │                              │
```

---

## Excel Upload Format

**Endpoint:** `POST /results/upload`
**Download template:** `GET /results/template`

| Column | Field     | Type   | Required | Notes                          |
|--------|-----------|--------|----------|--------------------------------|
| A      | StudentID | string | Yes      | MongoDB ObjectId of student    |
| B      | Subject   | string | Yes      | e.g. Mathematics               |
| C      | Grade     | number | Yes      | 0–100                          |
| D      | Term      | string | Yes      | first / second / third         |
| E      | ClassID   | string | Yes      | MongoDB ObjectId of class      |

Row 1 is the header row and is skipped. Invalid rows are collected in the `errors` array in the response. Valid rows are inserted in bulk. `session` must also be passed as a body field.

---

## Security Features

| Feature                   | Implementation                                        |
|---------------------------|-------------------------------------------------------|
| Password hashing          | bcrypt (configurable rounds, default 12)              |
| Auth tokens               | JWT access (7d) + refresh (30d) token pattern         |
| HTTP security headers     | Helmet (HSTS, CSP, X-Frame-Options, etc.)             |
| NoSQL injection           | express-mongo-sanitize strips `$` and `.` operators   |
| Rate limiting             | Global: 100 req/15min; Auth endpoint: 20 req/15min    |
| CORS                      | Configurable allowed origin via `FRONTEND_URL`        |
| Webhook verification      | HMAC-SHA512 signature validation on Paystack webhooks |
| Input validation          | express-validator chains on all mutating endpoints    |
| Account lockout           | Admin locks accounts; locked users receive 403        |
| File type enforcement     | Multer fileFilter: images only / .xlsx only           |
| Error sanitization        | Stack traces only in `NODE_ENV=development`           |
| Request payload limit     | 10MB on JSON body and file uploads                    |

---

## Deployment Notes

### Production environment
- Set `NODE_ENV=production`
- Use strong random strings for `JWT_SECRET` and `JWT_REFRESH_SECRET` (64+ chars)
- Use MongoDB Atlas or a properly secured self-hosted instance
- Set `FRONTEND_URL` to your exact frontend origin

### Paystack webhook
Register this URL in your Paystack dashboard settings:
```
https://your-domain.com/api/v1/payments/webhook
```
Copy the webhook secret into `PAYSTACK_WEBHOOK_SECRET`.

### File storage
`uploads/` is local disk by default. For scalable production deployments, replace Multer's `diskStorage` in `src/middleware/upload.ts` with an S3 or Cloudinary adapter.

### Process management
```bash
npm run build
npm install -g pm2
pm2 start dist/server.js --name osa-lms
pm2 save
pm2 startup
```

### Logging
Logs are written to `logs/combined.log` and `logs/error.log`. Point `LOG_DIR` to a persistent volume in containerized environments.
#
