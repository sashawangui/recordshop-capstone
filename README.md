# 🎵 RecordShop API

A REST API back-end for an e-commerce record store built with Java, Spring Boot, and MySQL.

**GitHub:** [sashawangui/recordshop-capstone](https://github.com/sashawangui/recordshop-capstone)

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [API Endpoints](#api-endpoints)
- [Interesting Code](#interesting-code)
- [Known Scope](#known-scope)
- [Author](#author)

---

## Overview

RecordShop is a fully functional e-commerce REST API that supports:

- JWT-based user authentication and role-based access control
- Full CRUD for product categories (admin only for writes)
- Product search with filters (category, price range, subcategory)
- A persistent shopping cart tied to each authenticated user
- User profile management

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot |
| Security | Spring Security + JWT |
| Database | MySQL |
| ORM | Spring Data JPA |
| Testing | Postman / Newman |
| IDE | IntelliJ IDEA |

---

## Architecture

The project follows a three-layer architecture:

```
Controller  →  receives HTTP requests, returns responses
Service     →  contains all business logic
Repository  →  talks to the database via JPA
```

Each feature has its own controller, service, and repository — keeping concerns cleanly separated.

---

## Getting Started

### Prerequisites

- Java 17+
- MySQL 8+
- IntelliJ IDEA
- Insomnia or Postman

### Setup

1. Clone the repo:
   ```
   git clone https://github.com/sashawangui/recordshop-capstone.git
   ```

2. Create the database — run the SQL script in MySQL Workbench:
   ```
   src/main/resources/create_database_recordshop.sql
   ```

3. Update `src/main/resources/application.properties` with your MySQL credentials:
   ```
   spring.datasource.url=jdbc:mysql://localhost:3306/recordshop
   spring.datasource.username=root
   spring.datasource.password=YOUR_PASSWORD
   spring.jpa.hibernate.ddl-auto=update
   ```

4. Run `RecordshopApplication.java` from IntelliJ. The server starts on `http://localhost:8080`.

### Default Users

| Username | Password | Role |
|---|---|---|
| user | password | ROLE_USER |
| admin | password | ROLE_ADMIN |

---

## API Endpoints

### Auth
| Method | Endpoint | Access | Description |
|---|---|---|---|
| POST | `/register` | Public | Register new user |
| POST | `/login` | Public | Login and receive JWT token |

### Categories
| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/categories` | Public | Get all categories |
| GET | `/categories/{id}` | Public | Get category by ID |
| GET | `/categories/{id}/products` | Public | Get products in a category |
| POST | `/categories` | Admin only | Create a category |
| PUT | `/categories/{id}` | Admin only | Update a category |
| DELETE | `/categories/{id}` | Admin only | Delete a category |

### Products
| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/products` | Public | Search/filter products |
| GET | `/products/{id}` | Public | Get product by ID |
| POST | `/products` | Admin only | Add a product |
| PUT | `/products/{id}` | Admin only | Update a product |
| DELETE | `/products/{id}` | Admin only | Delete a product |

**Product search query params:** `cat`, `minPrice`, `maxPrice`, `subCategory` — all optional and combinable.

Example: `GET /products?cat=1&minPrice=20&maxPrice=40`

### Shopping Cart
| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/cart` | User | View current cart |
| POST | `/cart/products/{productId}` | User | Add product to cart |
| PUT | `/cart/products/{productId}` | User | Update product quantity |
| DELETE | `/cart` | User | Clear the cart |

### Profile
| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/profile` | User | View own profile |
| PUT | `/profile` | User | Update own profile |

---

## Interesting Code

### Role-Based Access Control
Admin-only endpoints are protected with a single annotation:
```
@PreAuthorize("hasRole('ADMIN')")
```
Unauthenticated requests return **401 Unauthorized**. Authenticated non-admin requests return **403 Forbidden**.

### Cart Quantity Logic
Adding the same product twice increments the quantity instead of creating a duplicate row:
```
CartItem existing = shoppingCartRepository.findByUserIdAndProductId(userId, productId);

if (existing != null) {
    existing.setQuantity(existing.getQuantity() + 1);
    shoppingCartRepository.save(existing);
} else {
    CartItem cartItem = new CartItem();
    cartItem.setUserId(userId);
    cartItem.setProductId(productId);
    cartItem.setQuantity(1);
    shoppingCartRepository.save(cartItem);
}
```

### Product Filtering with Streams
All product filters are optional and applied in the service layer using Java Streams:
```
return products.stream()
    .filter(p -> minPrice == null || p.getPrice() >= minPrice)
    .filter(p -> maxPrice == null || p.getPrice() <= maxPrice)
    .filter(p -> subCategory == null || subCategory.equalsIgnoreCase(p.getSubCategory()))
    .toList();
```

---

## Known Scope

Phase 5 (Orders/Checkout) was intentionally scoped out. The database schema is fully in place — `orders` and `order_line_items` 
tables are created in the SQL script. The checkout layer would follow the same Controller → Service → Repository pattern and 
convert the cart into a persisted order on POST `/orders`.

---

## Author

**Sasha Iluku**
Year Up United — Puget Sound | Software Development Track
[LinkedIn](https://linkedin.com/in/sasha-iluku-2b77492a9) . 

[GitHub](https://github.com/sashawangui)
