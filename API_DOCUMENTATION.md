# E-Commerce API Documentation

**Version:** 1.0.0  
**Base URL:** `http://localhost:8000`  
**API prefix:** `/api/v1`

---

## Table of contents

1. [Overview](#overview)
2. [Database tables vs API](#database-tables-vs-api)
3. [Authentication](#authentication)
4. [Endpoints](#endpoints)
   - [Health](#health)
   - [Auth](#auth)
   - [Stores](#stores)
   - [Catalogs](#catalogs)
   - [Products](#products)
   - [Stories](#stories)
   - [Cart](#cart)
   - [Orders](#orders)
   - [Followers](#followers)
   - [Listerrboard](#listerrboard)
   - [Address](#address)
5. [Error responses](#error-responses)
6. [Interactive docs](#interactive-docs)

---

## Overview

- All v1 endpoints live under **`/api/v1`**.
- **Rate limiting** is applied per client IP (configurable).
- **CORS** is enabled; origins are configurable via `CORS_ORIGINS`.
- **JSON** is used for request and response bodies.

---

## Database tables vs API

There are **25 database tables**. The following mapping shows which tables are exposed via which API section, and which have no public CRUD endpoints yet.

| # | Database table | API section | Endpoints |
|---|----------------|-------------|-----------|
| 1 | `user` | Auth | Register, Login (read/write via auth only) |
| 2 | `user_sessions` | — | No public API (used internally for sessions) |
| 3 | `seller` | Stores | List, Get, Create, Update, Delete |
| 4 | `seller_integrations` | — | No public API |
| 5 | `seller_followers` | Followers | List, Get, Create (follow) |
| 6 | `connections` | — | No public API |
| 7 | `addresses` | Address | List, Get, Create, Update, Delete |
| 8 | `delivery_modes` | — | No public API |
| 9 | `delivery_zones` | — | No public API |
| 10 | `delivery_mode_zone_rates` | — | No public API |
| 11 | `checkout_points` | — | No public API |
| 12 | `tax_profiles` | — | No public API |
| 13 | `brands` | — | No public API |
| 14 | `story` | Stories | List, Get, Create, Update |
| 15 | `listerrboard` | Listerrboard | List, Get, Create, Update |
| 16 | `products` | Products | List, Get, Create, Update, Delete |
| 17 | `product_pricing` | — | No public API |
| 18 | `inventory` | — | No public API |
| 19 | `product_attributes` | — | No public API |
| 20 | `media_assets` | — | No public API |
| 21 | `product_media_map` | — | No public API |
| 22 | `story_media_map` | — | No public API |
| 23 | `catalogue` | Catalogs | List, Get, Create, Update |
| 24 | `cart` | Cart | List, Get, Create, Update |
| 25 | `orders` | Orders | List, Get, Create, Update |

**Summary:** 11 resource groups (Auth, Stores, Catalogs, Products, Stories, Cart, Orders, Followers, Listerrboard, Address) have public endpoints; 14 tables are not exposed via dedicated API yet (they can be added later as needed).

---

## Authentication

- **Login:** `POST /api/v1/auth/login` with `email` and `password` returns a JWT **access token**.
- **Protected routes:** Send the token in the header:
  ```http
  Authorization: Bearer <access_token>
  ```
- Token expiry is configurable (`ACCESS_TOKEN_EXPIRE_MINUTES`).

---

## Endpoints

### Health

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Root; returns service name and docs link. |
| `GET` | `/health` | Health check; returns service and database status. |

**Example response (success):**
```json
{
  "status": "ok",
  "database": "connected"
}
```

---

### Auth

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/auth/login` | Login with email and password. Returns JWT. |
| `POST` | `/api/v1/auth/register` | Register a new user. |

**POST /api/v1/auth/login**  
Request body:
```json
{
  "email": "user@example.com",
  "password": "your_password"
}
```
Response (200):
```json
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "expires_in": 1800
}
```

**POST /api/v1/auth/register**  
Request body:
```json
{
  "name": "John Doe",
  "email": "user@example.com",
  "password": "secure_password",
  "phone": "+919876543210"
}
```
Response (200): User object (e.g. `user_uuid`, `name`, `email`, `created_at`); no password.

---

### Stores

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/stores` | List stores (paginated). Query: `skip`, `limit`. |
| `GET` | `/api/v1/stores/{seller_uuid}` | Get one store by UUID. |
| `POST` | `/api/v1/stores` | Create a store. |
| `PATCH` | `/api/v1/stores/{seller_uuid}` | Update a store. |
| `DELETE` | `/api/v1/stores/{seller_uuid}` | Delete a store. |

**POST /api/v1/stores** body (minimal): `name`, `email`; optional: `phone_no`, `profilepicture`, `gst`, `status`.

---

### Catalogs

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/catalogs` | List catalogs. Query: `skip`, `limit`, `seller_uuid`. |
| `GET` | `/api/v1/catalogs/{catalog_uuid}` | Get one catalog. |
| `POST` | `/api/v1/catalogs` | Create a catalog. Body: `seller_uuid`, optional `slides`, `product_uuids`. |
| `PATCH` | `/api/v1/catalogs/{catalog_uuid}` | Update a catalog. |

---

### Products

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/products` | List products. Query: `skip`, `limit`, `seller_uuid`. |
| `GET` | `/api/v1/products/{product_uuid}` | Get one product. |
| `POST` | `/api/v1/products` | Create a product. Body: `seller_uuid`, `name`; optional slug, description, about, tags, category_uuid, sub_category_uuid, tax_slab_uuid, brand_uuid, status. |
| `PATCH` | `/api/v1/products/{product_uuid}` | Update a product. |
| `DELETE` | `/api/v1/products/{product_uuid}` | Delete a product. |

---

### Stories

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/stories` | List stories. Query: `skip`, `limit`, `seller_uuid`. |
| `GET` | `/api/v1/stories/{story_uuid}` | Get one story. |
| `POST` | `/api/v1/stories` | Create a story. Body: `seller_uuid`, `title`; optional description, urlslug, status, cover_info, story_type, lang, etc. |
| `PATCH` | `/api/v1/stories/{story_uuid}` | Update a story. |

---

### Cart

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/cart` | List carts. Query: `skip`, `limit`, `user_uuid`, `seller_uuid`. |
| `GET` | `/api/v1/cart/{cart_uuid}` | Get one cart. |
| `POST` | `/api/v1/cart` | Create a cart. Body: `user_uuid`, `seller_uuid`; optional checkout_uuid, delivery_type, subtotal, tax_total, other_charges, delivery_fee, total_price, status. |
| `PATCH` | `/api/v1/cart/{cart_uuid}` | Update a cart. |

---

### Orders

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/orders` | List order items. Query: `skip`, `limit`, `order_uuid`, `cart_uuid`. |
| `GET` | `/api/v1/orders/{order_item_uuid}` | Get one order item. |
| `POST` | `/api/v1/orders` | Create an order item. Body: `order_uuid`, `product_uuid`; optional user_uuid, seller_uuid, cart_uuid, checkout_uuid, shipping_address_uuid, billing_address_uuid, product_name_snapshot, delivery_type, qty, unit_price_snapshot, unit_sale_price_snapshot, line_total, order_status, payment_status, item_status. |
| `PATCH` | `/api/v1/orders/{order_item_uuid}` | Update an order item (e.g. qty, item_status). |

---

### Followers

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/followers` | List follower records. Query: `skip`, `limit`, `user_uuid`, `seller_uuid`. |
| `GET` | `/api/v1/followers/{follow_uuid}` | Get one follower record. |
| `POST` | `/api/v1/followers` | Follow a seller. Body: `user_uuid`, `seller_uuid`; optional status. |

---

### Listerrboard

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/listerrboard` | List boards. Query: `skip`, `limit`. |
| `GET` | `/api/v1/listerrboard/{board_uuid}` | Get one board. |
| `POST` | `/api/v1/listerrboard` | Create a board. Body: `story_uuid`; optional title, name, description, status. |
| `PATCH` | `/api/v1/listerrboard/{board_uuid}` | Update a board. |

---

### Address

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/address` | List addresses. Query: `skip`, `limit`, `owner_uuid`, `owner_type`. |
| `GET` | `/api/v1/address/{address_uuid}` | Get one address. |
| `POST` | `/api/v1/address` | Create an address. Body: `owner_uuid`; optional owner_type, type, address_line1, address_line2, location, lat, long, pincode, city, state, country, permanent_address, is_default, status. |
| `PATCH` | `/api/v1/address/{address_uuid}` | Update an address. |
| `DELETE` | `/api/v1/address/{address_uuid}` | Delete an address. |

---

## Error responses

- **400 Bad Request** – Invalid input (e.g. validation error, duplicate email).
- **401 Unauthorized** – Missing or invalid token.
- **404 Not Found** – Resource not found (e.g. wrong UUID).
- **422 Unprocessable Entity** – Request body validation failed.
- **429 Too Many Requests** – Rate limit exceeded.
- **500 Internal Server Error** – Server or database error.

Standard error body shape:
```json
{
  "detail": "Human-readable message",
  "code": "optional_error_code"
}
```

---

## Interactive docs

- **Swagger UI:** [http://localhost:8000/docs](http://localhost:8000/docs) – try endpoints from the browser.
- **ReDoc:** [http://localhost:8000/redoc](http://localhost:8000/redoc) – readable API reference.
- **OpenAPI JSON:** [http://localhost:8000/openapi.json](http://localhost:8000/openapi.json) – machine-readable schema.


