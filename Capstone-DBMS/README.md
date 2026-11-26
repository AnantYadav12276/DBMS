# Inventory Management System

The frontend was started from a friend's project and redesigned; the inventory database and backend were built separately.

A full-stack inventory management tool featuring live stock tracking, automated low-stock threshold detection, atomic stock movements, and an audit trail.

---

## Features

- **Dashboard & Analytics:** Overview metrics tracking total products, total stock units, low-stock alerts count, and 30-day inbound/outbound movement volumes.
- **Stock Movement Timeline:** Visualizes 30-day stock inflow (bars) vs. outflow (trend line) aggregated from actual movement history.
- **Low-Stock Detection:** Real-time table identifying products at or below configured reorder levels with automatic shortfall computation.
- **Product Catalog Management:** Full CRUD operations (Add, Edit, Delete with confirmation) with automatic SKU generation and price tracking.
- **Atomic Stock In / Out Movements:** Stock replenishment and dispatch with transaction boundaries (`BEGIN` / `COMMIT` / `ROLLBACK`) preventing negative balances.
- **Movement History:** Complete audit trail of all historical stock movements with timestamps and product associations.

---

## Tech Stack

- **Frontend:** React 19, Vite, Tailwind CSS v4, Recharts
- **Backend:** Node.js, Express, `pg` (node-postgres), CORS, dotenv
- **Database:** PostgreSQL (`inventory_db`)

---

## Database Tables

The database schema (`inventory-backend/schema.sql`) implements:

- **`products`**: Product catalog (`product_id`, `name`, `sku`, `price`, `reorder_threshold`).
- **`warehouses`**: Storage facilities (`warehouse_id`, `name`, `location`).
- **`bins`**: Warehouse storage bins (`bin_id`, `warehouse_id`, `aisle`, `bin_code`, `capacity`).
- **`inventory`**: Product-by-bin balance tracking (`product_id`, `bin_id`, `quantity`, `updated_at`). Enforces non-negative stock.
- **`orders`**: Customer orders (`order_id`, `customer_name`, `status`, `created_at`, `fulfilled_at`).
- **`order_items`**: Order line items (`order_item_id`, `order_id`, `product_id`, `quantity`).
- **`inventory_transactions`**: Append-only transactional audit ledger (`transaction_id`, `product_id`, `bin_id`, `order_id`, `txn_type`, `quantity_change`, `quantity_after`, `created_at`). Protected by an append-only trigger.
- **`restock_alerts`**: Low-stock alert table automatically updated by the `trg_inventory_reorder_check` trigger when stock drops below threshold.
- **`daily_stock_trends`**: Materialized view calculating rolling 90-day stock depletion, restock, and transaction counts.

---

## API Routes

| Method | Path | Description |
| :--- | :--- | :--- |
| `GET` | `/` | API discovery and endpoint listing |
| `GET` | `/health` | Service health check & DB connection verification |
| `GET` | `/products` | List all products with aggregated stock quantities |
| `POST` | `/products` | Add a new product (`name`, `price`, `reorder_level`, `sku`) |
| `PUT` | `/products/:id` | Update product details |
| `DELETE`| `/products/:id` | Remove a product (if no ledger history exists) |
| `POST` | `/stock` | Record stock movement (`product_id`, `type`, `quantity`) |
| `GET` | `/products/low-stock` | List products with quantity $\le$ reorder threshold |
| `GET` | `/history` | Chronological stock movement log |
| `GET` | `/warehouses` | List warehouses |
| `GET` | `/bins` | List storage bins |
| `GET` | `/alerts` | List active restock alerts |
| `GET` | `/trends` | View daily stock trends from materialized view |

---

## Setup & Installation

### 1. Initialize PostgreSQL Database
Ensure PostgreSQL is running locally, then create the database and execute the schema:
```bash
createdb inventory_db
psql -d inventory_db -f inventory-backend/schema.sql
```

### 2. Configure Backend Environment
Create `inventory-backend/.env` with your database credentials:
```env
PORT=4000
DATABASE_URL=postgres://localhost:5432/inventory_db
```

### 3. Start Backend Server
```bash
cd inventory-backend
node server.js
```
The server will start listening on port 4000.

### 4. Start Frontend
In a separate terminal:
```bash
cd warehouse-ui
npm install
npm run dev
```
Open `http://localhost:5173` in your browser.
