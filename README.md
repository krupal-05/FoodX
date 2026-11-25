# FoodX — Hyperlocal Food Discovery, Ordering & Restaurant Management Platform

![Node.js](https://img.shields.io/badge/Backend-Node.js%20%2B%20Express%205-green.svg)
![MongoDB](https://img.shields.io/badge/Database-MongoDB%20%2F%20Mongoose%209-emerald.svg)
![React](https://img.shields.io/badge/Frontend-React%2019%20%2B%20Vite%208-blue.svg)
![Tailwind CSS](https://img.shields.io/badge/Styling-Tailwind%20CSS%20v4-38bdf8.svg)
![License](https://img.shields.io/badge/License-ISC-orange.svg)

**FoodX** is a full-stack hyperlocal food discovery, ordering, and multi-portal restaurant management platform. It consists of a decoupled **Node.js + Express 5 + MongoDB** REST API (`backend/`) and a **React 19 + Vite 8 + Tailwind CSS v4** single-page application (`frontend/`).

The application serves three primary role-based portals—**Customer**, **Restaurant Owner**, and **Super Admin**—backed by JWT access/refresh token authentication, server-persisted carts and orders, Cloudinary image uploads, and Mongoose data modeling.

---

## Table of Contents

- [Project Description & Purpose](#project-description--purpose)
- [Implemented Features](#implemented-features)
- [Technologies & Dependencies](#technologies--dependencies)
- [Project Folder Structure](#project-folder-structure)
- [Frontend Architecture](#frontend-architecture)
- [Backend Architecture](#backend-architecture)
- [Database Schema & Relationships](#database-schema--relationships)
- [Authentication & Authorization Flow](#authentication--authorization-flow)
- [How the Complete Application Works](#how-the-complete-application-works)
- [REST API Documentation](#rest-api-documentation)
- [Environment Variables](#environment-variables)
- [Local Installation & Running Commands](#local-installation--running-commands)
- [Deployment Instructions](#deployment-instructions)
- [Known Limitations & Future Improvements](#known-limitations--future-improvements)
- [License](#license)

---

## Project Description & Purpose

FoodX is designed to connect local customers with neighborhood restaurants through a unified web platform:

1. **Customers** browse restaurants and dishes, filter menus by category and dietary preference (`veg`, `non-veg`, `vegan`), manage a database-persisted shopping cart, place orders with delivery address validation, cancel pending orders, update their profile/password, and leave 1–5 star restaurant reviews.
2. **Restaurant Owners (`restaurantOwner`)** manage one or more restaurant outlets, toggle live open/closed availability, upload restaurant banners and dish photos to Cloudinary, maintain menu items (CRUD + stock availability), and fulfill incoming orders through a Kitchen Display System (KDS) queue.
3. **Administrators (`admin`)** monitor platform-wide accounts, restaurants, global food listings, and order volume (GMV), with direct moderation capabilities (restaurant/food deletion and order status overrides).

---

## Implemented Features

### 1. Customer Portal (`/`)
- **Home & Discovery (`/`):** Hero showcase, dynamically computed food categories from catalog items (with fallback defaults), top 8 recommended dishes, verified restaurant cards, and partner onboarding CTA.
- **Smart Menu & Filtering (`/menu`):** Debounced keyword search (`350ms` via `useDebounce`), URL-synced category pills, dietary type filters (`all`, `veg`, `non-veg`, `vegan`), and client-side sorting (recommended, price low-to-high, price high-to-low, highest rating).
- **Restaurant Storefront & Reviews (`/restaurant/:id`):** Restaurant banner, cuisine badge, delivery time, minimum order amount, average rating, category-filtered menu, customer review cards, and a modal for authenticated customers to submit 1–5 star reviews.
- **Cart & Checkout (`/cart`):** Database-synchronized cart (`CartContext`) with inline quantity controls, item removal, clear cart, delivery instructions, saved-address pre-fill, 6-digit pincode validation (`/^\d{6}$/`), minimum order threshold check, payment method selection (`cod` or `online`), and `canvas-confetti` feedback on order placement.
- **Order History & Cancellation (`/orders`):** Displays customer orders with status badges (`pending`, `confirmed`, `preparing`, `ready`, `out_for_delivery`, `delivered`, `cancelled`), itemized totals, delivery address, and an inline two-step confirmation to cancel `pending` orders.
- **Profile & Security (`/profile`):** Update `firstName`, `lastName`, `phone`, and default delivery `address` (`street`, `city`, `pincode`), plus password rotation via current password verification.
- **About & Contact (`/about`):** Brand overview, headquarters/contact info, and a local UI-only feedback form.

### 2. Restaurant Owner Portal (`/owner/*`)
- **Owner Dashboard (`/owner/dashboard`):** Displays primary outlet open/closed toggle (`isOpen`), KPI cards (pending tickets, in-kitchen prep count, active menu items, fulfilled revenue), quick navigation cards, and the latest 5 live kitchen tickets with one-click status advancement.
- **Outlet Management (`/owner/restaurants`):** Create, edit, and delete owned restaurants with `multipart/form-data` image upload to Cloudinary, address fields, delivery time, minimum order, and open/closed status.
- **Food Catalog Management (`/owner/foods`):** Switch across owned outlets, search dishes, create/edit/delete dishes with dietary tags (`veg`, `non-veg`, `vegan`) and image uploads, and toggle `isAvailable` (`In Stock` / `Sold Out`) inline.
- **Kitchen Order Fulfillment (`/owner/orders`):** Filter incoming orders by status tabs (`all`, `pending`, `preparing`, `ready`, `out_for_delivery`, `delivered`, `cancelled`), search by order ID or customer name, view customer contact/instructions, and advance or decline orders.

### 3. Super Admin Portal (`/admin/*`)
- **Isolated Admin Login (`/admin/login`):** Dedicated dark-themed login screen that verifies `user.role === 'admin'` client-side after login (automatically signing out non-admin accounts).
- **Platform Metrics (`/admin/dashboard`):** Aggregates total accounts (customers vs merchants), total/open outlets, catalog item count, fulfilled GMV (from `delivered` orders), and the latest 6 platform orders.
- **User Directory (`/admin/users`):** Lists all accounts with role summary cards, role tabs (`all`, `customer`, `owner`, `admin`, `deliveryagent`), and search by name, email, or phone.
- **Restaurant Governance (`/admin/restaurants`):** Lists all outlets with owner contact details, ratings, open status, direct links to public menus, and cascade delete (`deleteRestaurant` deletes associated `Food` documents).
- **Global Food Catalog (`/admin/foods`):** Cross-restaurant food moderation table with category pills, search, and delete capability.
- **Global Order Ledger (`/admin/orders`):** Filter all platform orders by any of the 7 status stages, search by order/customer/restaurant, and override order status via an inline `<select>` dropdown.

---

## Technologies & Dependencies

### Backend (`backend/package.json`)
- **Runtime:** Node.js (ES Modules, `"type": "module"`)
- **Framework:** `express` (`^5.2.1`)
- **Database ODM:** `mongoose` (`^9.9.4`)
- **Authentication & Security:** `jsonwebtoken` (`^9.0.3`), `bcrypt` (`^6.0.0`), `cookie-parser` (`^1.4.7`)
- **File Upload & Media:** `multer` (`^2.3.0`), `cloudinary` (`^2.11.0`)
- **Middleware & Configuration:** `cors` (`^2.8.6`), `dotenv` (`^17.4.2`)
- **Dev Tooling:** `nodemon` (`^3.1.14`)

### Frontend (`frontend/package.json`)
- **UI Library:** `react` (`^19.2.8`), `react-dom` (`^19.2.8`)
- **Build Tool:** `vite` (`^8.3.0`), `@vitejs/plugin-react` (`^6.1.1`)
- **Routing:** `react-router-dom` (`^7.18.4`)
- **Styling:** `tailwindcss` (`^4.3.3`), `@tailwindcss/vite` (`^4.3.3`)
- **HTTP Client:** `axios` (`^1.20.0`)
- **Icons & Effects:** `lucide-react` (`^1.52.0`), `canvas-confetti` (`^1.9.4`)
- **Linting & Type Stubs:** `oxlint` (`^1.81.0`), `@types/react` (`^19.2.18`), `@types/react-dom` (`^19.2.7`)

---

## Project Folder Structure

```text
Food Web/
├── .env.example                             # Legacy root env template (see Environment Variables section)
├── .gitignore                               # Ignores node_modules, .env*, uploads, dist/build, logs
├── README.md                                # Root project documentation
├── backend/
│   ├── index.js                             # Loads dotenv, connects MongoDB, starts HTTP server
│   ├── package.json                         # Backend dependencies and scripts (start, dev, test)
│   ├── seed.js                              # Seeds 15 sample Food items when Food collection is empty
│   ├── test-backend.js                      # Smoke script verifying all backend module & route imports
│   ├── public/
│   │   └── temp/                            # Multer temporary upload directory (auto-created if missing)
│   ├── uploads/                             # Static directory mounted at /uploads
│   └── src/
│       ├── app.js                           # Express app, CORS, JSON/URL/cookie parsers, route mounts, 404 & error handler
│       ├── config/
│       │   ├── clouldinary.js               # Cloudinary v2 config & uploadCloud() helper with temp cleanup
│       │   └── db.js                        # Mongoose connection helper (MONGODB_URL)
│       ├── controllers/
│       │   ├── Cart.controller.js           # Cart CRUD & total calculations
│       │   ├── Order.controller.js          # Order placement, user/owner/admin queries, status & cancellation
│       │   ├── Restaurant.controller.js     # Restaurant CRUD, search/sort filters, Cloudinary image handling
│       │   ├── User.controller.js           # Auth (register, login, logout, refresh), profile, password, admin user list
│       │   ├── food.controller.js           # Food CRUD, search/filter queries, image updates
│       │   └── review.controller.js         # Create/list/delete reviews & recalculate restaurant rating
│       ├── middleware/
│       │   ├── auth.middleware.js           # authorizeRoles, verifyAdmin, verifyOwner, verifyOwnerOrAdmin
│       │   ├── multer.midelware.js          # Multer diskStorage (5MB limit, image/* filter -> ./public/temp)
│       │   └── varify.middleware.js         # verifyJwt / varifyjwt (cookie or Bearer token verification)
│       ├── models/
│       │   ├── Cart.model.js                # User cart schema with foodId & quantity items
│       │   ├── Order.model.js               # Order schema with items, deliveryAddress, status, payment, assignedAgent
│       │   ├── Restaurant.model.js          # Restaurant schema with address, owner ref, rating, virtual ownerId
│       │   ├── User.model.js                # User schema, bcrypt pre-save hook, JWT generators
│       │   ├── food.model.js                # Food schema with foodType, restaurant ref, virtual restaurantId
│       │   └── review.model.js              # Review schema (user, restaurant, rating 1-5, comment)
│       ├── routers/
│       │   ├── cart.router.js               # /api/v1/cart routes
│       │   ├── food.router.js               # /api/v1/foods routes
│       │   ├── order.router.js              # /api/v1/orders routes
│       │   ├── restaurant.router.js         # /api/v1/restaurants routes
│       │   ├── review.router.js             # /api/v1/reviews routes
│       │   └── user.router.js               # /api/v1/users routes
│       └── utils/
│           ├── ApiError.js                  # Standardized operational Error subclass
│           ├── Apiresponse.js               # Standardized JSON response wrapper { statusCode, data, message, success }
│           └── asyncHandler.js              # Express async route wrapper forwarding errors to next()
└── frontend/
    ├── .gitignore                           # Frontend gitignore rules
    ├── .oxlintrc.json                       # Oxlint configuration (react & oxc plugins)
    ├── README.md                            # Frontend-specific notes
    ├── index.html                           # HTML entrypoint (loads Google Fonts Inter & Outfit)
    ├── package.json                         # Frontend dependencies and Vite/oxlint scripts
    ├── vite.config.js                       # Vite 8 config with @vitejs/plugin-react & @tailwindcss/vite
    ├── public/
    │   ├── favicon.svg
    │   └── icons.svg
    └── src/
        ├── App.jsx                          # Root providers: ErrorBoundary, BrowserRouter, Toast, Auth, Cart, AppRoutes
        ├── index.css                        # Tailwind CSS v4 import, base typography, custom scrollbar & fadeIn
        ├── main.jsx                         # ReactDOM.createRoot mount with StrictMode
        ├── assets/                          # hero.png, react.svg, vite.svg
        ├── components/
        │   ├── common/
        │   │   ├── Badge.jsx                # Status/tag pill badge with optional indicator dot
        │   │   ├── Button.jsx               # Multi-variant button with loading spinner & Lucide icon
        │   │   ├── EmptyState.jsx           # Reusable empty state card with CTA
        │   │   ├── ErrorBoundary.jsx        # Class-based React error boundary with page reload recovery
        │   │   ├── Input.jsx                # Form input/textarea with icon, error/helper text & password toggle
        │   │   ├── Modal.jsx                # Accessible modal dialog (Escape key, backdrop click, scroll lock)
        │   │   └── Skeleton.jsx             # Skeleton, FoodCardSkeleton, and TableSkeleton loaders
        │   ├── food/
        │   │   ├── CategoryCard.jsx         # Category tile linking to /menu?category=...
        │   │   ├── FoodCard.jsx             # Dish card with dietary badge, rating, and live cart +/- controls
        │   │   └── RestaurantCard.jsx       # Restaurant card with open/closed overlay, rating, delivery time
        │   └── layout/
        │       ├── Footer.jsx               # Multi-column dark footer with explore, partner, and contact links
        │       └── Navbar.jsx               # Sticky header with role-aware profile dropdown, cart badge, mobile drawer
        ├── constants/
        │   ├── api.constants.js             # API_BASE_URL and ENDPOINTS map
        │   └── order.constants.js           # ORDER_STATUSES, ORDER_STATUS_CONFIG, PAYMENT_METHODS, USER_ROLES
        ├── context/
        │   ├── AuthContext.jsx              # Session persistence, login/register/logout/profile sync, role booleans
        │   ├── CartContext.jsx              # Server-backed cart state, add/update/remove/clear actions
        │   └── ToastContext.jsx             # Floating toast notification stack (success, error, info)
        ├── hooks/
        │   ├── useAuth.js                   # Re-exports AuthContext hook
        │   ├── useCart.js                   # Re-exports CartContext hook
        │   ├── useDebounce.js               # Generic debounce hook (default 400ms)
        │   └── useToast.js                  # Re-exports ToastContext hook
        ├── layouts/
        │   ├── AdminLayout.jsx              # Dark sidebar shell for /admin/*
        │   ├── CustomerLayout.jsx           # Navbar + Outlet + Footer shell for customer/public routes
        │   └── OwnerLayout.jsx              # Dark MerchantSuite sidebar shell for /owner/*
        ├── pages/
        │   ├── NotFoundPage.jsx             # 404 fallback page
        │   ├── admin/                       # AdminDashboard, AdminFoods, AdminOrders, AdminRestaurants, UserManagement
        │   ├── auth/                        # AdminLoginPage, LoginPage, RegisterPage
        │   ├── customer/                    # AboutPage, CartPage, HomePage, MenuPage, OrdersPage, ProfilePage, RestaurantDetailPage
        │   └── restaurant/                  # FoodManagement, OwnerDashboard, OwnerOrders, RestaurantManagement
        ├── routes/
        │   ├── AppRoutes.jsx                # Lazy-loaded route table with Suspense and RBAC wrappers
        │   ├── ProtectedRoute.jsx           # Redirects unauthenticated users to /login
        │   └── RoleRoute.jsx                # Enforces allowedRoles or renders Access Restricted view
        └── services/
            ├── api.js                       # Axios instance, Bearer injection, 401 refresh queue, error extraction
            ├── auth.service.js              # User auth, profile, password, and admin user list calls
            ├── cart.service.js              # Cart API calls
            ├── food.service.js              # Food catalog & multipart upload calls
            ├── order.service.js             # Order placement, history, owner/admin queries, status updates
            ├── restaurant.service.js        # Restaurant discovery & owner/admin management calls
            └── review.service.js            # Restaurant review fetch, submit, delete calls
```

---

## Frontend Architecture

### Component & Provider Hierarchy
In `frontend/src/App.jsx`, the application wraps `AppRoutes` in the following provider tree:

```text
ErrorBoundary
 └── BrowserRouter
      └── ToastProvider
           └── AuthProvider
                └── CartProvider
                     └── AppRoutes (Suspense + lazy-loaded Layouts & Pages)
```

### Routing & RBAC Matrix (`frontend/src/routes/AppRoutes.jsx`)
All page components are code-split with `React.lazy()` and rendered inside `<Suspense fallback={<PageLoader />}>`:

| Route | Layout | Access Guard | Allowed Roles | Page Component |
| :--- | :--- | :--- | :--- | :--- |
| `/` | `CustomerLayout` | Public | All | `HomePage` |
| `/menu` | `CustomerLayout` | Public | All | `MenuPage` |
| `/restaurant/:id` | `CustomerLayout` | Public | All | `RestaurantDetailPage` |
| `/about` | `CustomerLayout` | Public | All | `AboutPage` |
| `/login` | Standalone | Public | All | `LoginPage` |
| `/register` | Standalone | Public | All | `RegisterPage` |
| `/admin/login` | Standalone | Public | All | `AdminLoginPage` |
| `/cart` | `CustomerLayout` | `ProtectedRoute` + `RoleRoute` | `customer` | `CartPage` |
| `/orders` | `CustomerLayout` | `ProtectedRoute` | Any authenticated user | `OrdersPage` |
| `/profile` | `CustomerLayout` | `ProtectedRoute` | Any authenticated user | `ProfilePage` |
| `/owner/dashboard` | `OwnerLayout` | `ProtectedRoute` + `RoleRoute` | `restaurantOwner`, `owner`, `admin` | `OwnerDashboard` |
| `/owner/restaurants` | `OwnerLayout` | `ProtectedRoute` + `RoleRoute` | `restaurantOwner`, `owner`, `admin` | `RestaurantManagement` |
| `/owner/foods` | `OwnerLayout` | `ProtectedRoute` + `RoleRoute` | `restaurantOwner`, `owner`, `admin` | `FoodManagement` |
| `/owner/orders` | `OwnerLayout` | `ProtectedRoute` + `RoleRoute` | `restaurantOwner`, `owner`, `admin` | `OwnerOrders` |
| `/admin/dashboard` | `AdminLayout` | `ProtectedRoute` + `RoleRoute` | `admin` | `AdminDashboard` |
| `/admin/users` | `AdminLayout` | `ProtectedRoute` + `RoleRoute` | `admin` | `UserManagement` |
| `/admin/restaurants` | `AdminLayout` | `ProtectedRoute` + `RoleRoute` | `admin` | `AdminRestaurants` |
| `/admin/foods` | `AdminLayout` | `ProtectedRoute` + `RoleRoute` | `admin` | `AdminFoods` |
| `/admin/orders` | `AdminLayout` | `ProtectedRoute` + `RoleRoute` | `admin` | `AdminOrders` |
| `*` | `CustomerLayout` | Public | All | `NotFoundPage` |

### State Management & API Layer
- **`AuthContext`:** Initializes `user` from `localStorage` (`food_user`), verifies active sessions on mount via `GET /api/v1/users/user` when `food_access_token` is present, and exposes normalized role flags (`isCustomer`, `isOwner`, `isAdmin`, `isAuthenticated`).
- **`CartContext`:** Automatically fetches `/api/v1/cart` when `isAuthenticated && isCustomer`, and exposes `addToCart`, `updateQuantity`, `removeItem`, `clearCart`, and `getItemQuantity`.
- **`ToastContext`:** Manages auto-dismissing notifications (`3500ms` default) for `success`, `error`, and `info` feedback.
- **`api.js` (Axios Instance):** Configured with `baseURL: API_BASE_URL` and `withCredentials: true`. Unwraps standardized `ApiResponse` envelopes (`response.data?.data !== undefined ? response.data : response`) and attaches `error.extractedMessage` on failure.

### Design System & Responsive Layout
- **Styling Engine:** Tailwind CSS v4 loaded via `@tailwindcss/vite` in `frontend/vite.config.js` and `@import "tailwindcss";` in `frontend/src/index.css`.
- **Typography:** `Inter` for body text and `Outfit` for headings (`h1`–`h6`), loaded via Google Fonts in `frontend/index.html`.
- **Visual Language:** Light slate canvas (`#f8fafc`) with orange-to-amber gradients (`from-orange-500 to-amber-500`) for Customer and Owner views, and a dark slate/purple palette (`bg-slate-950`, `purple-600`) for Super Admin views.
- **Responsiveness:** Sticky navbar with mobile drawer in `CustomerLayout`, slide-over sidebar drawers in `OwnerLayout` and `AdminLayout`, and responsive CSS grids across catalogs and dashboards.

---

## Backend Architecture

- **Entrypoint (`backend/index.js`):** Loads `.env` via `dotenv.config()`, establishes the MongoDB connection (`dbConnection()`), and starts the Express HTTP server on `process.env.PORT || 5000`.
- **Express Setup (`backend/src/app.js`):**
  - Configures `cors({ origin: process.env.CORS_ORIGIN || true, credentials: true })`.
  - Parses JSON and URL-encoded bodies up to `16kb`, and parses cookies with `cookie-parser`.
  - Serves static directories at `/uploads` and `/public`.
  - Mounts primary REST routers under `/api/v1/{users,restaurants,foods,cart,orders,reviews}` plus legacy compatibility mounts directly under `/api/v1`.
  - Provides JSON 404 handling and a centralized error handler that formats Mongoose `ValidationError` and duplicate-key (`11000`) errors, returning stack traces only when `NODE_ENV === "development"`.
- **Middleware Pipeline:**
  - `varifyjwt` / `verifyJwt` (`backend/src/middleware/varify.middleware.js`): Extracts JWT from `req.cookies.accessToken` or `Authorization: Bearer <token>`, verifies it, and attaches `req.user` (excluding `password` and `reFreshToken`).
  - `auth.middleware.js`: Provides `authorizeRoles(...roles)`, `verifyAdmin`, and `verifyOwner` / `verifyOwnerOrAdmin`.
  - `multer.midelware.js`: Stores single uploaded `image` files in `./public/temp` (5 MB max, `image/*` MIME filter) before `uploadCloud()` in `clouldinary.js` uploads to Cloudinary (`folder: "food_delivery"`) and deletes the local temp file.

---

## Database Schema & Relationships

The backend uses **MongoDB** via **Mongoose 9** with six collections:

```text
User (1) ──────< (N) Restaurant (owner)
User (1) ────── (1) Cart (userId) ──────< (N) CartItem ──> Food (foodId)
User (1) ──────< (N) Order (userId / assignedAgent)
Restaurant (1) ─< (N) Food (restaurant)
Restaurant (1) ─< (N) Order (restaurantId) ──< (N) OrderItem ──> Food (foodId)
Restaurant (1) ─< (N) Review (restaurant) <── (N) User (user)
```

| Model | File | Key Fields & Constraints |
| :--- | :--- | :--- |
| **User** | `backend/src/models/User.model.js` | `firstName`, `lastName`, `email` (unique, lowercase), `password` (bcrypt hashed), `phone`, `role` (`customer`, `restaurantOwner`, `admin`, `deliveryAgent`), `address` (`street`, `city`, `pincode`), `avatar`, `reFreshToken` |
| **Restaurant** | `backend/src/models/Restaurant.model.js` | `name`, `cuisine`, `description`, `address` (`street`, `city`, `pincode` required), `phone`, `image`, `imageUrl`, `rating` (0–5, default 4.5), `totalReviews`, `isOpen` (default `true`), `deliveryTime` (default `30`), `minOrder` (default `100`), `owner` (`ObjectId -> User`), virtual `ownerId` |
| **Food** | `backend/src/models/food.model.js` | `name`, `description`, `price` (>= 0), `image`, `category`, `foodType` (`veg`, `non-veg`, `vegan`), `isAvailable` (default `true`), `rating` (0–5, default 4.5), `restaurant` (`ObjectId -> Restaurant`), virtual `restaurantId` |
| **Cart** | `backend/src/models/Cart.model.js` | `userId` (`ObjectId -> User`, unique), `items[]` (`foodId -> Food`, `quantity` >= 1) |
| **Order** | `backend/src/models/Order.model.js` | `userId` (`ObjectId -> User`), `restaurantId` (`ObjectId -> Restaurant`), `items[]` (`foodId`, `name`, `quantity`, `price`), `deliveryAddress` (`street`, `city`, `pincode`), `status` (`pending`, `confirmed`, `preparing`, `ready`, `out_for_delivery`, `delivered`, `cancelled`), `totalAmount`, `paymentStatus` (`pending`, `paid`, `failed`, `refunded`), `paymentMethod` (`cod`, `online`, `card`, `upi`), `instructions`, `assignedAgent` (`ObjectId -> User`), virtuals `user` & `restaurant` |
| **Review** | `backend/src/models/review.model.js` | `user` (`ObjectId -> User`), `restaurant` (`ObjectId -> Restaurant`), `rating` (1–5), `comment` |

---

## Authentication & Authorization Flow

1. **Registration / Login (`POST /api/v1/users/register` or `/login`):**
   - Backend validates credentials (`bcrypt.compare`), generates a JWT Access Token (`1d` default) and JWT Refresh Token (`7d` default), persists `reFreshToken` on the `User` document, sets `accessToken` and `refreshToken` as `HttpOnly` cookies (`sameSite: "lax"`, `secure` in production), and returns `{ user, accessToken, reFreshToken }`.
2. **Client Persistence:**
   - `AuthContext.jsx` stores `food_user`, `food_access_token`, and `food_refresh_token` in `localStorage`.
   - `api.js` injects `Authorization: Bearer <food_access_token>` on outgoing requests and sends cookies (`withCredentials: true`).
3. **Automatic Token Rotation (`401` Interceptor in `frontend/src/services/api.js`):**
   - On `401` (excluding `/login` and `/refresh-token`), concurrent failed requests are queued (`failedQueue`) while a single request calls `POST /api/v1/users/refresh-token` with `{ refreshToken }` and cookies.
   - Backend validates the refresh token against `user.reFreshToken`, rotates both tokens, and returns `{ accessToken, reFreshToken }`.
   - If refresh fails, `localStorage` session keys are cleared and `window.dispatchEvent(new Event('auth:unauthorized'))` logs the user out cleanly.
4. **Logout (`POST /api/v1/users/logout`):**
   - Clears `reFreshToken` in MongoDB, clears `accessToken`/`refreshToken` cookies, and wipes `localStorage` on the frontend.

---

## How the Complete Application Works

1. **Discovery & Browsing:** Visitors open `/` or `/menu`, fetching public restaurants (`GET /api/v1/restaurants`) and dishes (`GET /api/v1/foods`). Selecting a restaurant navigates to `/restaurant/:id`, loading the outlet, its menu (`foods`), and reviews (`GET /api/v1/reviews/restaurant/:id`).
2. **Cart Synchronization:** Logged-in customers click **Add** on `FoodCard`, triggering `POST /api/v1/cart/items`. Cart items and totals are persisted in MongoDB and reflected in the `Navbar` badge and `FoodCard` quantity steppers.
3. **Checkout & Order Placement:** On `/cart`, the customer enters/verifies a 6-digit delivery address, chooses `cod` or `online`, and clicks **Place Order**. `POST /api/v1/orders` verifies every `foodId` and its price in MongoDB, checks `restaurant.minOrder`, creates the `Order`, and clears the user's `Cart`.
4. **Kitchen Fulfillment:** The restaurant owner views the new `pending` ticket on `/owner/dashboard` or `/owner/orders`, accepts/prepares/marks ready/dispatches/completes it via `PUT /api/v1/orders/:orderId/status`. When a `cod` order reaches `delivered`, the backend automatically updates `paymentStatus` to `paid`.
5. **Order Tracking & Cancellation:** The customer refreshes `/orders` to view status updates or cancels an order while it is still `pending` (`PUT /api/v1/orders/:orderId/cancel`).
6. **Platform Administration:** Super admins sign in at `/admin/login` to monitor GMV, filter users, audit/delete restaurants and dishes, and override any order status at `/admin/*`.

---

## REST API Documentation

Base URL: `http://localhost:5000/api/v1`

**Health Checks:**
- `GET /` — Returns `{ status: "success", message, timestamp }`
- `GET /api/v1/health` — Returns `{ status: "ok", message: "API is healthy" }`

### 1. Users & Authentication (`/api/v1/users`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/register` | Public | Register user (`firstName`, `lastName`, `email`, `password`, optional `phone`, `role`, `address`) |
| `POST` | `/login` | Public | Login with `email` and `password`; returns user + tokens |
| `POST` | `/refresh-token` | Public | Rotate access & refresh tokens via cookie or `{ refreshToken }` |
| `POST` | `/logout` | Authenticated | Clear DB refresh token and auth cookies (`DELETE /user` also maps here) |
| `GET` | `/profile` \| `/user` | Authenticated | Get current authenticated user profile |
| `PUT` | `/profile` \| `/user` | Authenticated | Update `firstName`, `lastName`, `phone`, `address`, `avatar` |
| `PUT` | `/change-password` | Authenticated | Update password (`oldPassword`, `newPassword`) |
| `GET` | `/users` | Admin | List all users (excluding password and refresh token) |

### 2. Restaurants (`/api/v1/restaurants`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/` \| `/all` \| `/All-restaurants` | Public | List restaurants; supports `?search=`, `?cuisine=`, `?isOpen=`, `?sort=rating\|deliveryTime\|minOrder` |
| `GET` | `/:restaurantId` | Public | Get restaurant details + its `foods` array |
| `GET` | `/owner/my-restaurants` | Owner / Admin | List restaurants owned by `req.user._id` |
| `POST` | `/register` | Owner / Admin | Create restaurant (`multipart/form-data` optional `image`, 6-digit `address.pincode` required) |
| `PUT` | `/:restaurantId` | Owner (of outlet) / Admin | Update restaurant fields and optional `image` |
| `DELETE` | `/:restaurantId` | Owner (of outlet) / Admin | Delete restaurant and cascade delete its `Food` items |

### 3. Foods (`/api/v1/foods`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/` \| `/all` | Public | List foods; supports `?search=`, `?category=`, `?foodType=`, `?restaurantId=`, `?minPrice=`, `?maxPrice=` |
| `GET` | `/:id` | Public | Get single food item populated with restaurant summary |
| `GET` | `/restaurant/:restaurantId` | Public | Get all food items belonging to a restaurant |
| `POST` | `/` | Owner (of outlet) / Admin | Create food item (`multipart/form-data` optional `image`) |
| `PUT` | `/:id` | Owner (of outlet) / Admin | Update food item fields and optional `image` |
| `PATCH` | `/:id/image` | Owner (of outlet) / Admin | Replace food item `image` (`multipart/form-data` `image` required) |
| `DELETE` | `/:id` | Owner (of outlet) / Admin | Delete food item |

### 4. Cart (`/api/v1/cart`) — *All routes require `verifyJwt`*

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/` | Get populated cart (`items`, `totalAmount`, `totalCount`); auto-creates empty cart if none exists |
| `POST` | `/items` \| `/add` | Add `{ foodId, quantity }` to cart (validates `food.isAvailable`) |
| `PATCH` \| `PUT` | `/items` (or `PUT /update`) | Set item `quantity`; removes item if `quantity <= 0` |
| `DELETE` | `/items/:foodId` \| `/items` | Remove a specific `foodId` from cart |
| `DELETE` | `/clear` | Empty all items from user's cart |

### 5. Orders (`/api/v1/orders`) — *All routes require `verifyJwt`*

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/` \| `/create` | Authenticated | Place order from `{ restaurantId, items, deliveryAddress, paymentMethod, instructions }` or active Cart; verifies DB prices & `minOrder`, clears Cart |
| `GET` | `/my-orders` \| `/user` | Authenticated | Fetch logged-in user's order history |
| `GET` | `/:orderId` | Order Customer / Outlet Owner / Admin | Fetch single order details |
| `PUT` \| `PATCH` | `/:orderId/cancel` | Order Customer / Admin | Cancel order (only permitted while `status === 'pending'`) |
| `GET` | `/owner/restaurant-orders` | Owner / Admin | Fetch all orders for restaurants owned by `req.user._id` |
| `PUT` \| `PATCH` | `/:orderId/status` | Outlet Owner / Admin / `deliveryAgent` | Update `status` and/or `paymentStatus` |
| `GET` | `/admin/all` | Admin | Fetch all orders across the platform |

### 6. Reviews (`/api/v1/reviews`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/restaurant/:restaurantId` | Public | Fetch all reviews for a restaurant |
| `POST` | `/` | Authenticated | Submit review (`restaurantId`, `rating` 1–5, `comment`) and recalculate restaurant `rating` & `totalReviews` |
| `DELETE` | `/:reviewId` | Review Author / Admin | Delete review and recalculate restaurant rating |

---

## Environment Variables

> **Note:** The root `.env.example` file contains outdated variable names (`MONGO_URI`, `JWT_SECRET`, `JWT_EXPIRES_IN`). Configure the actual environment files inside `backend/.env` and `frontend/.env` using the keys below.

### Backend (`backend/.env`)

```env
PORT=5000
MONGODB_URL=mongodb://localhost:27017/food_delivery_db
CORS_ORIGIN=http://localhost:5173
NODE_ENV=development

JWT_ACCESS_SECRET=replace_with_a_strong_random_access_secret
JWT_ACCESS_EXPIRES=1d
JWT_REFRESH_SECRET=replace_with_a_strong_random_refresh_secret
JWT_REFRESH_EXPIRES=7d

CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
```

*(Note: `backend/src/config/clouldinary.js` also falls back to legacy env keys `CLOULDINARY_NAME`, `CLOULDINARY_API_KEY`, and `CLOUDINARY_SECRATE` if the standard `CLOUDINARY_*` keys are unset.)*

### Frontend (`frontend/.env`)

```env
VITE_API_URL=http://localhost:5000/api/v1
```

If omitted, `frontend/src/constants/api.constants.js` defaults `API_BASE_URL` to `http://localhost:5000/api/v1`.

---

## Local Installation & Running Commands

### Prerequisites
- **Node.js** (v18+ or v20+ recommended for Express 5 & Vite 8)
- **npm**
- **MongoDB** (local service or MongoDB Atlas URI)
- **Cloudinary account** (optional for local development; required for image uploads)

### 1. Backend Setup & Run
```bash
cd backend
npm install

# Verify all backend module & router imports
npm test

# Start development server with nodemon
npm run dev

# Or start production server
npm start
```
The API server listens on `http://localhost:5000` (or `PORT`).

#### Optional: Seed Sample Food Items
If your MongoDB database already contains at least one `Restaurant` document and the `Food` collection is empty (`count === 0`), you can insert 15 sample food items:
```bash
cd backend
node seed.js
```

### 2. Frontend Setup & Run
Open a new terminal from the repository root:
```bash
cd frontend
npm install

# Start Vite development server
npm run dev
```
The frontend dev server launches at `http://localhost:5173`.

### 3. Frontend Linting & Production Build
```bash
cd frontend
npm run lint
npm run build
npm run preview
```
`npm run build` outputs the production bundle to `frontend/dist/`.

---

## Deployment Instructions

Because `backend/` and `frontend/` are separate Node/Vite projects, deploy them as two independent services:

### 1. Database & Media
- Provision a managed MongoDB cluster (e.g., MongoDB Atlas) and obtain the connection string for `MONGODB_URL`.
- Create a Cloudinary account and note `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, and `CLOUDINARY_API_SECRET`.

### 2. Deploy Backend (`backend/`)
- Deploy `backend/` to a Node.js host (Render, Railway, Fly.io, AWS EC2, etc.).
- Set `NODE_ENV=production`, `PORT`, `MONGODB_URL`, strong unique values for `JWT_ACCESS_SECRET` and `JWT_REFRESH_SECRET`, `CLOUDINARY_*` keys, and `CORS_ORIGIN` set to your deployed frontend origin (e.g., `https://your-app.vercel.app`).
- Install & start command:
  ```bash
  npm install && npm start
  ```
- Verify deployment via `GET https://<your-backend-domain>/api/v1/health`.

### 3. Deploy Frontend (`frontend/`)
- Deploy `frontend/` to a static/SPA host (Vercel, Netlify, Cloudflare Pages, etc.) with root directory `frontend`.
- Set build environment variable:
  ```env
  VITE_API_URL=https://<your-backend-domain>/api/v1
  ```
- Build command: `npm run build` | Output directory: `dist`.
- **Important SPA Routing Note:** Configure your host to rewrite all non-file routes to `/index.html` so direct visits to `/menu`, `/owner/dashboard`, or `/admin/login` resolve via React Router instead of returning a server 404.
- **Cross-Origin Cookie Note:** When frontend and backend run on different domains over HTTPS, note that `User.controller.js` currently sets `sameSite: "lax"` on auth cookies; the frontend still authenticates reliably via `Authorization: Bearer` tokens stored in `localStorage`.

---

## Known Limitations & Future Improvements

Based on inspection of the current codebase, the following limitations and gaps exist:

1. **No Real Payment Gateway Integration:** Selecting `"online"` during checkout (`CartPage.jsx` / `Order.controller.js`) immediately sets `paymentStatus: "paid"` without integrating Razorpay, Stripe, or another payment processor.
2. **Checkout Fee Calculation Mismatch:** `CartPage.jsx` displays a delivery fee (`₹40`, free over `₹299`) and a platform fee (`₹15`) in the Bill Summary, whereas `Order.controller.js` (`addOrder`) calculates `Order.totalAmount` strictly from the sum of `food.price * quantity` in MongoDB without adding delivery or platform fees.
3. **No Single-Restaurant Cart Restriction:** The backend `Cart` model and `addCart` controller allow adding dishes from multiple different restaurants into one cart, while `CartPage.jsx` submits `restaurantId` from the first item (`items[0]`). Enforcing a single-restaurant cart guard (or splitting multi-restaurant checkouts) is recommended.
4. **Delivery Agent Role Incomplete:** `deliveryAgent` is defined in `User.model.js`, `Order.model.js` (`assignedAgent`), `Order.controller.js` (`updateOrderStatus`), and `UserManagement.jsx`, but no delivery agent portal, routes, or order-assignment API endpoints are implemented.
5. **Unrestricted Role Registration & Fallback Secrets:** `POST /api/v1/users/register` accepts `"admin"` in `allowedRoles` if called directly via API (though the UI hides it), and `User.model.js` / `varify.middleware.js` contain hardcoded fallback JWT secrets if environment variables are missing. Production hardening should restrict self-registration to `customer`/`restaurantOwner` and require env secrets.
6. **Local-Only Feedback Form & Unused Review Delete UI:** The contact/feedback form on `AboutPage.jsx` only updates local component state and does not persist to the backend. Additionally, `DELETE /api/v1/reviews/:reviewId` exists in the backend and `review.service.js` but is not wired into `RestaurantDetailPage.jsx`.
7. **Seed Script Prerequisite:** `backend/seed.js` seeds `Food` documents only and assumes `Restaurant` documents already exist in MongoDB (no seed script exists for initial `User` or `Restaurant` records).
8. **Real-Time Order Updates:** Order dashboards rely on manual **Refresh** / **Sync** actions rather than WebSockets or Server-Sent Events (SSE).

---

## License

This project is licensed under the **ISC License** (as specified in `backend/package.json`).
