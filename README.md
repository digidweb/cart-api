# Cart API 🛒

> A RESTful e-commerce cart API built with Ruby on Rails, PostgreSQL, Redis, and Sidekiq.

Cart API is a backend application that implements the core functionality of an e-commerce shopping cart.

The project focuses on **REST API design, relational data modeling, business rules, automated testing, and asynchronous background processing** using a realistic shopping cart domain.

## ✨ Features

* 🛒 Add, update, remove, and clear cart items
* 💰 Calculate item subtotals and cart totals
* 📦 Product, Cart, and CartItem domain modeling
* ✅ Business validations and domain rules
* 🔄 Cart activity and lifecycle management
* 💤 Detect and process inactive carts
* ⚙️ Background jobs with Sidekiq and Redis
* ⏰ Scheduled jobs with Sidekiq Cron
* 🧪 Automated tests with RSpec and FactoryBot
* 🐳 Docker development environment

## 🛠️ Tech Stack

**Backend**

* Ruby 3.3.1
* Ruby on Rails 7.1.3.2
* Active Record
* REST API

**Database & Background Processing**

* PostgreSQL 16
* Redis 7
* Sidekiq 7
* Sidekiq Cron

**Testing & Quality**

* RSpec Rails
* FactoryBot
* Faker
* Shoulda Matchers
* Database Cleaner

**Infrastructure**

* Docker
* Docker Compose

## 🏗️ Architecture

The application follows a conventional Rails API architecture, with business rules handled by the domain models and asynchronous operations delegated to background jobs.

```text
                   HTTP / JSON
                        │
                        ▼
              ┌─────────────────┐
              │    Rails API    │
              │                 │
              │ Controllers     │
              │ Domain Models   │
              └────────┬────────┘
                       │
                 Active Record
                       │
                       ▼
              ┌─────────────────┐
              │   PostgreSQL    │
              └─────────────────┘

                       ▲
                       │
              ┌────────┴────────┐
              │     Sidekiq     │
              │                 │
              │ Background Jobs │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │      Redis      │
              └─────────────────┘
```

This separation keeps synchronous API requests independent from background maintenance tasks.

## 🧩 Domain & Business Rules

The application is centered around three main entities:

```text
Product
   │
   ▼
CartItem
   │
   ▼
Cart
```

### Cart

A cart can be in one of three states:

* `active`
* `abandoned`
* `completed`

The application tracks cart activity and uses this information to manage inactive carts.

### CartItem

Cart items contain a product, quantity, and calculated subtotal.

Business rules include:

* Quantity must be a positive integer
* Quantity cannot exceed the defined limit
* A product cannot be duplicated within the same cart
* Adding an existing product increases its quantity
* Cart totals are calculated from persisted cart items

These rules are implemented through Active Record validations, associations, scopes, callbacks, and domain methods.

## ⚙️ Background Processing

Inactive cart management is handled asynchronously with **Sidekiq** and **Redis**.

Scheduled jobs can:

```text
Detect inactive carts
        ↓
Mark carts as abandoned
        ↓
Remove carts after the retention period
```

The current business rules identify carts inactive for more than **3 hours** and remove carts that have remained abandoned for more than **7 days**.

Using background jobs keeps maintenance operations outside the HTTP request lifecycle.

## 🔌 API

The API exposes the main shopping cart operations:

| Method   | Endpoint         | Purpose                   |
| -------- | ---------------- | ------------------------- |
| `POST`   | `/cart`          | Add a product to the cart |
| `GET`    | `/cart`          | Retrieve the current cart |
| `POST`   | `/cart/add_item` | Update an item's quantity |
| `DELETE` | `/cart/:id`      | Remove a cart item        |

Example:

```http
POST /cart
Content-Type: application/json
```

```json
{
  "product_id": 1,
  "quantity": 2
}
```

The API returns JSON responses and uses HTTP status codes to communicate successful operations and validation failures.

## 🧪 Testing

The project uses **RSpec** with FactoryBot, Shoulda Matchers, Faker, and Database Cleaner.

Run the test suite:

```bash
bundle exec rspec
```

For detailed output:

```bash
bundle exec rspec --format documentation
```

Tests focus on the application's domain behavior, validations, associations, and business rules.

## 🚀 Getting Started

### Prerequisites

* Ruby 3.3.1
* PostgreSQL 16
* Redis 7
* Bundler

Or:

* Docker
* Docker Compose

### Clone

```bash
git clone https://github.com/digidweb/cart-api.git
cd cart-api
```

### Local setup

Install dependencies:

```bash
bundle install
```

Create and migrate the database:

```bash
rails db:create
rails db:migrate
```

Start Redis:

```bash
redis-server
```

Start Sidekiq in another terminal:

```bash
bundle exec sidekiq
```

Start Rails:

```bash
bundle exec rails server
```

The API will be available at:

```text
http://localhost:3000
```

### Docker

Alternatively:

```bash
docker-compose up --build
```

## 💡 What This Project Demonstrates

Cart API demonstrates practical experience with:

**Ruby on Rails · REST APIs · PostgreSQL · Active Record · Redis · Sidekiq · RSpec · FactoryBot · Docker**

More importantly, the project demonstrates the ability to translate business requirements into backend behavior through:

```text
Business requirement
        ↓
Domain model
        ↓
Validation / business rule
        ↓
Persistence
        ↓
API response
        ↓
Automated test
```

## 🔮 Potential Improvements

Possible next steps include:

* Add request/integration tests for all API endpoints
* Add authentication and authorization
* Add product stock validation
* Add standardized API error responses
* Add OpenAPI/Swagger documentation
* Add GitHub Actions for CI
* Add test coverage reporting
* Add production deployment

---

## 📌 Portfolio Context

Cart API is part of a portfolio focused on **Ruby on Rails backend and full-stack development**.

The project complements other applications demonstrating Rails application architecture, React/Rails integration, automated testing, and domain-specific business logic.


```text
Business requirement
        ↓
Domain model
        ↓
Validation / business rule
        ↓
Persistence
        ↓
API response
        ↓
Automated test
```

<!--# Shopping Cart API

REST API for managing e-commerce shopping carts.

## 🛠 Technologies

* Ruby 3.3.1
* Rails 7.1.3.2
* PostgreSQL 16
* Redis 7.0.15
* Sidekiq
* RSpec

## 📋 Prerequisites

* Docker and Docker Compose (recommended)
* Or: Ruby, Rails, PostgreSQL, and Redis installed locally

## 🚀 Getting Started

### With Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/digidweb/cart_api
cd cart-api

# Start the containers
docker-compose up --build

# In another terminal, create the database and run migrations
docker-compose exec web rails db:create db:migrate

# Create some products for testing
docker-compose exec web rails console
Product.create(name: "Product 1", price: 10.99)
Product.create(name: "Product 2", price: 25.50)
```

### Without Docker

```bash
# Install dependencies
bundle install

# Set up the database
rails db:create db:migrate

# Start Redis (in one terminal)
redis-server

# Start Sidekiq (in another terminal)
bundle exec sidekiq

# Start Rails (in another terminal)
bundle exec rails server
```

## 📝 Endpoints

### 1. Add a product to the cart

```http
POST /cart
Content-Type: application/json

{
  "product_id": 1,
  "quantity": 2
}
```

### 2. Get the current cart

```http
GET /cart
```

### 3. Update product quantity

```http
POST /cart/add_item
Content-Type: application/json

{
  "product_id": 1,
  "quantity": 3
}
```

### 4. Remove a product

```http
DELETE /cart/1
```

## 🧪 Tests

```bash
# Run all tests
bundle exec rspec

# Run with detailed output
bundle exec rspec --format documentation

# Generate coverage report
bundle exec rspec --format html --out coverage/index.html
```

## 📦 Jobs

The system includes a background job that runs automatically:

* **Every hour:** marks inactive carts (3+ hours) as abandoned
* **Every hour:** removes carts that have been abandoned for more than 7 days

## 📄 License

This project was developed as part of a technical challenge.-->
