<p align="center">
  <img src="public/manofcave.png" alt="Man of Cave logo" width="160" />
</p>

<h1 align="center">Man of Cave</h1>

<p align="center">
  A multi-branch barbershop and grooming management platform with a public booking website,
  a customer portal, branch admin dashboards and a super-admin control centre.
</p>

<p align="center">
  Next.js 16 · React 19 · TypeScript · Firebase · Tailwind CSS 4 · Zustand
</p>

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Firebase Data Model](#firebase-data-model)
- [Available Scripts](#available-scripts)
- [Building for Production](#building-for-production)
- [Deployment](#deployment)
- [Project Status](#project-status)

## Overview

**Man of Cave** is a web application for running a grooming business that operates across several branches. It brings the customer-facing website and the internal back office into a single Next.js codebase:

- **Customers** browse services, products and branch locations, book appointments, place product orders, and manage their bookings, loyalty points, wallet and profile from a personal portal.
- **Branch admins** run day-to-day operations for their own branch — appointments, the booking calendar, clients, staff, orders, expenses, invoices and customer chat.
- **Super admins** oversee the whole business across all branches, including branch management, users and permissions, financial reporting, staff attendance, memberships and blog content.

Firebase provides authentication, the Firestore database and file storage, so the application runs without a separate custom backend.

## Key Features

### Public Website
- Landing page, service catalogue, product catalogue and a menu page
- Branch directory with individual branch detail pages (`/branches/[id]`)
- Blog listing and article pages (`/blog/[slug]`)
- Appointment booking flow with branch, service, staff, date and time selection
- Shopping cart and checkout with Cash on Delivery, digital wallet and mixed payment options (wallet and mixed payment require a signed-in account)

### Customer Portal (`/customer/portal`)
- Email/password registration and sign-in via Firebase Authentication
- Booking history, order history and transaction history
- Loyalty points and wallet balance
- Profile management
- Live chat with the branch team (`/customer/chat`)

### Branch Admin Dashboard (`/admin`)
- Appointment management and an interactive booking calendar with PDF export
- Client, staff, service, category and product management
- Order management and membership management
- Expense tracking with charts
- Custom invoice generation with PDF export
- Customer chats, internal messages and customer feedback
- Audible notifications for incoming messages
- Branch-level analytics and settings, including display currency selection (USD, INR, PKR, AED)

### Super Admin Control Centre (`/super-admin`)
- Management of all branches and branch configuration
- User management with per-user page access (`allowedPages`) and role-based access
- Cross-branch appointments, booking calendar, clients and staff
- Staff attendance tracking
- Financial management, booking (sales) reports and order finance reports
- Analytics with Excel (`.xlsx`) export
- Expense tracking, custom invoices, memberships, orders and products
- Blog management, customer chat, messages and feedback
- Global settings, including file uploads to Firebase Storage

## Technology Stack

| Area | Technologies |
| --- | --- |
| Framework | [Next.js 16](https://nextjs.org/) (App Router), [React 19](https://react.dev/) |
| Language | [TypeScript 5](https://www.typescriptlang.org/) (strict mode) |
| Backend services | [Firebase](https://firebase.google.com/) — Authentication, Cloud Firestore, Cloud Storage, Analytics |
| Styling | [Tailwind CSS 4](https://tailwindcss.com/), `tw-animate-css` |
| UI components | [shadcn/ui](https://ui.shadcn.com/) (New York style) built on [Radix UI](https://www.radix-ui.com/), [Lucide](https://lucide.dev/) icons |
| State management | [Zustand](https://zustand.docs.pmnd.rs/) |
| Forms & validation | [React Hook Form](https://react-hook-form.com/), [Zod](https://zod.dev/) |
| Dates & calendars | `date-fns`, `react-day-picker`, `react-datepicker` |
| Charts | [Recharts](https://recharts.org/) |
| Documents & exports | `jspdf`, `jspdf-autotable` (PDF), `xlsx` (Excel) |
| Carousels & feedback | Embla Carousel, Sonner toasts |
| Tooling | ESLint 9 with `eslint-config-next` |

## Architecture

```text
┌──────────────────────────────────────────────────────────────────┐
│                       Next.js App Router (app/)                  │
│                                                                  │
│  Public site     Customer portal     /admin         /super-admin │
│  /, /services,   /customer/*         Branch admin   Business-wide│
│  /booking, ...                       dashboard      control      │
└───────────────┬──────────────────────────────────────────────────┘
                │ React components (src/components)
                │ Contexts: Auth, Branch, Language (src/contexts)
                │ Zustand stores (src/stores)
                ▼
┌──────────────────────────────────────────────────────────────────┐
│                Firebase client SDK (src/lib/firebase.ts)         │
│        Authentication · Cloud Firestore · Cloud Storage          │
└──────────────────────────────────────────────────────────────────┘
```

- **Client-side data access** — pages and Zustand stores read and write Firestore directly through the Firebase client SDK; there is no separate API server.
- **Authentication and roles** — `AuthContext` listens to Firebase Auth state and loads the user profile from the `users` collection. Each profile carries a `role` (`super_admin`, `admin` or `customer`), an optional `branchId`, and an optional `allowedPages` list used to control dashboard access.
- **Branch context** — `BranchContext` and the branch stores scope data to the selected branch.
- **Internationalisation** — `LanguageContext` provides an English/Arabic translation dictionary and switches the document direction to right-to-left for Arabic.
- **API route** — `app/api/booking/route.ts` exposes a `POST` endpoint that validates booking payloads and returns a generated booking ID. It does not persist data; bookings in the application are stored in Firestore.

## Repository Structure

```text
manofcave/
├── app/                      # Next.js App Router pages and layouts
│   ├── page.tsx              # Landing page
│   ├── layout.tsx            # Root layout and providers
│   ├── globals.css           # Tailwind CSS and theme variables
│   ├── admin/                # Branch admin dashboard
│   ├── super-admin/          # Super admin control centre
│   ├── customer/             # Customer login, chat and portal
│   ├── api/booking/          # Booking validation API route
│   ├── blog/                 # Blog listing and [slug] article pages
│   ├── booking/              # Appointment booking flow
│   ├── branches/             # Branch directory and [id] detail pages
│   ├── checkout/             # Cart checkout
│   ├── cms/                  # Landing page CMS screen
│   ├── login/                # Staff (admin / super admin) sign-in
│   ├── menu/                 # Services menu
│   ├── mobile-app/           # Mobile app management screen
│   ├── products/             # Product catalogue
│   └── services/             # Service catalogue
├── src/
│   ├── components/
│   │   ├── ui/               # shadcn/ui primitives and custom calendar
│   │   ├── admin/            # Admin dashboard components
│   │   ├── booking/          # Booking components
│   │   ├── shared/           # Header, Footer and shared layout pieces
│   │   ├── invoice-templates.tsx
│   │   ├── AppointmentDetailsModal.tsx
│   │   ├── ProtectedRoute.tsx
│   │   └── providers.tsx
│   ├── contexts/             # Auth, Branch, CustomerAuth and Language contexts
│   ├── hooks/                # Message notifications and toast hooks
│   ├── lib/                  # Firebase setup, messaging helpers, invoice utilities
│   └── stores/               # Zustand stores (bookings, branches, services, staff, ...)
├── types/                    # Shared TypeScript types
├── public/                   # Static assets (logos, notification sound, icons)
├── components.json           # shadcn/ui configuration
├── next.config.ts            # Next.js configuration (remote image domains)
├── eslint.config.mjs         # ESLint configuration
├── postcss.config.mjs        # PostCSS / Tailwind CSS configuration
├── tsconfig.json             # TypeScript configuration (`@/*` → `src/*`)
└── package.json
```

## Getting Started

### Prerequisites

- **Node.js 20.9 or later** (required by Next.js 16)
- **npm** (a `package-lock.json` is committed)
- A **Firebase project** with Authentication (Email/Password provider), Cloud Firestore and Cloud Storage enabled

### Installation

```bash
git clone https://github.com/faiza-ijaz0/manofcave.git
cd manofcave
npm install
```

### Run the Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

| Area | Local URL |
| --- | --- |
| Public website | http://localhost:3000 |
| Customer sign-in / registration | http://localhost:3000/customer/login |
| Staff sign-in | http://localhost:3000/login |
| Branch admin dashboard | http://localhost:3000/admin |
| Super admin control centre | http://localhost:3000/super-admin |

## Configuration

### Firebase

The Firebase client configuration is defined in [`src/lib/firebase.ts`](src/lib/firebase.ts), which initialises and exports `db` (Firestore), `auth` (Authentication), `storage` (Cloud Storage) and `analytics` (browser only).

To run the project against your own Firebase project, replace the values in the `firebaseConfig` object with the web app configuration from **Firebase Console → Project settings → Your apps**.

> The project does not currently read environment variables, and no `.env` file is required. `.env*` files are excluded by `.gitignore` if you choose to move the configuration into environment variables.

### Remote Images

[`next.config.ts`](next.config.ts) allows `next/image` to load images from `images.unsplash.com`, `images.pexels.com` and `encrypted-tbn0.gstatic.com`. Add further hosts to `images.remotePatterns` if needed.

### Path Alias

`@/*` resolves to `src/*` (see [`tsconfig.json`](tsconfig.json)), for example `import { db } from '@/lib/firebase'`.

## Firebase Data Model

The application uses Cloud Firestore; collections are created on first write, so no migration step is required. The main collections referenced in the code are:

| Domain | Collections |
| --- | --- |
| Users & access | `users`, `customers` |
| Branches & catalogue | `branches`, `services`, `categories`, `products`, `staff` |
| Bookings & sales | `bookings`, `orders`, `cart`, `transactions`, `customInvoices` |
| Loyalty & payments | `loyaltyPoints`, `loyalty_settings`, `wallets`, `walletTransactions`, `cashbackPrograms`, `memberships`, `offers`, `promoCodes` |
| Messaging | `conversations`, `customerMessages`, `customerReplies`, `adminMessages`, `branchMessages` |
| Operations & content | `attendance`, `manualExpenses`, `feedbacks`, `blogs` |

### Creating the First Staff Account

Staff roles are read from the `users` collection. To create an admin:

1. Create the user in **Firebase Console → Authentication**.
2. Add a document to `users` whose ID is that user's UID, with at least:
   - `role`: `"super_admin"` or `"admin"`
   - `name`, and for branch admins `branchId` / `branchName`
   - optionally `allowedPages`: an array of permitted dashboard pages
3. Sign in at `/login`.

Customers can register themselves at `/customer/login`.

## Available Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Starts the Next.js development server on port 3000 |
| `npm run build` | Creates an optimised production build |
| `npm run start` | Serves the production build (run `npm run build` first) |
| `npm run lint` | Runs ESLint across the project |

> No automated test suite is configured in this repository.

## Building for Production

```bash
npm run build
npm run start
```

## Deployment

Man of Cave is a standard Next.js application and can be deployed to any platform that supports Next.js, such as [Vercel](https://vercel.com/) or a Node.js server.

**Vercel**

1. Import the GitHub repository into Vercel.
2. Keep the detected defaults (framework **Next.js**, build command `npm run build`).
3. Deploy.

**Node.js server**

```bash
npm ci
npm run build
npm run start        # serves on port 3000 by default; set PORT to change it
```

After deploying, add the production domain to **Firebase Console → Authentication → Settings → Authorized domains** so that sign-in works on the live site.

## Project Status

The application is under active development. Some screens, such as the landing page CMS (`/cms`), currently use placeholder content, and the `/api/booking` route validates requests without persisting them.
