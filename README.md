
# Marketplace Platform

A full-stack e-commerce marketplace with a customer storefront and an admin dashboard.
Customers browse products, place orders and track them; admins manage the product catalog
and fulfil orders. Authentication is handled by **Supabase**, the storefront is built with
**Next.js**, and order/product business logic runs in a **Go** API backed by PostgreSQL.

Demo video can be seen here:
https://www.youtube.com/watch?v=zrlz6jEA5Nc

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS 4, lucide-react |
| Auth | Supabase Auth (`@supabase/ssr`, `@supabase/supabase-js`) |
| Backend | Go 1.25, Gin, GORM |
| Database | PostgreSQL (via Supabase) |

## Prerequisites

- **Node.js 20+**
- **Go 1.25+**
- A **Supabase project** (for auth and the PostgreSQL database)

## Setup

### 1. Backend (Go API)

```bash
cd backend
```

Create a `.env` file with your database connection string:

```env
DATABASE_URL=postgresql://<user>:<password>@<host>:5432/<db>
```

Install dependencies and run:

```bash
go mod download
go run main.go
```

The API starts on `http://localhost:8080`. CORS is configured to allow the frontend at
`http://localhost:3000`.

> The schema (`products`, `orders`, `order_items`, `profiles`) is expected to already exist
> in the database. The models in `backend/models/` map to these tables.

### 2. Frontend (Next.js)

```bash
cd frontend
```

Create `.env.local` with your Supabase project credentials:

```env
NEXT_PUBLIC_SUPABASE_URL=https://<your-project>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-anon-key>
```

Install dependencies and run the dev server:

```bash
npm install
npm run dev
```

The app is available at `http://localhost:3000`.

## API reference

Base URL: `http://localhost:8080`

### Orders

| Method | Path | Description |
|---|---|---|
| `POST` | `/create-order` | Place an order. Body: `{user_id, shipping_address, items: [{product_id, quantity}]}`. Validates stock, decrements it, and computes the total in a single transaction. |
| `GET` | `/orders/user/:user_id` | List a user's orders (newest first), including order items and product details. |

### Admin

| Method | Path | Description |
|---|---|---|
| `POST` | `/admin/products` | Create a product. |
| `GET` | `/admin/products` | List all products. |
| `GET` | `/admin/products/:id` | Get a single product. |
| `PUT` | `/admin/products/:id` | Update a product. |
| `DELETE` | `/admin/products/:id` | Delete a product. |
| `GET` | `/admin/orders` | List all orders. |
| `PATCH` | `/admin/orders/:id/status` | Update an order's status. |
| `GET` | `/admin/orders/stats` | Order statistics for the dashboard. |

## Data model

| Table | Key fields |
|---|---|
| `products` | `id`, `name`, `description`, `price`, `stock_quantity`, `image_url`, `is_active`, `created_at` |
| `orders` | `id`, `user_id`, `status`, `total_amount`, `shipping_address`, `created_at` |
| `order_items` | `id`, `order_id`, `product_id`, `quantity`, `price_at_purchase` |
| `profiles` | `id`, `email`, `first_name`, `last_name`, `role`, `created_at` |

## Project structure

```
.
├── backend/                 # Go + Gin API
│   ├── main.go              # Routes, CORS, server bootstrap
│   ├── config/db.go         # GORM PostgreSQL connection
│   ├── controllers/         # userController.go, adminController.go
│   └── models/              # order.go, product.go, profile.go
├── frontend/                # Next.js app
│   ├── app/                 # Routes: store, product, checkout, orders, admin, auth
│   │   └── components/      # Shared UI (navbars, cards, forms, layouts)
│   ├── lib/                 # Supabase client/server helpers
│   └── middleware.ts        # Supabase session sync
└── supabase/                # Supabase project config
```

## Environment variables

**Backend (`backend/.env`)**

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |

**Frontend (`frontend/.env.local`)**

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anonymous (public) key |
