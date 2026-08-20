# Cart API 🛒

> A RESTful e-commerce cart API built with Ruby on Rails, PostgreSQL, Redis, and Sidekiq.

Cart API is a backend application that provides the core functionality of an e-commerce shopping cart.

The project was developed as a technical challenge and focuses on building a maintainable REST API while implementing real-world business rules around products, cart items, cart states, totals, inactivity, and abandoned carts.

The application also demonstrates asynchronous background processing with **Sidekiq**, scheduled jobs with **Sidekiq Cron**, database persistence with **PostgreSQL**, and automated testing with **RSpec** and **FactoryBot**.

## ✨ Features

* 🛒 Add products to a shopping cart
* 📋 Retrieve the current cart and its items
* 🔢 Update product quantities
* 🗑️ Remove products from the cart
* 🧹 Clear the cart
* 💰 Calculate cart totals and item subtotals
* 📦 Persist products, carts, and cart items in PostgreSQL
* 🔄 Automatically update cart interaction timestamps
* 💤 Detect inactive carts
* 🚨 Mark inactive carts as abandoned
* 🧹 Remove carts that have remained abandoned for a defined period
* ⚙️ Background processing with Sidekiq
* ⏰ Scheduled jobs with Sidekiq Cron
* 🧪 Automated tests with RSpec
* 🏭 Test data generation with FactoryBot and Faker
* 🐳 Docker support

## 🛠️ Tech Stack

### Backend

* **Ruby 3.3.1**
* **Ruby on Rails 7.1.3.2**
* **Rails REST API**
* **Active Record**
* **Puma**

### Database

* **PostgreSQL 16**

### Background Processing

* **Redis 7**
* **Sidekiq 7**
* **Sidekiq Cron**

### Testing

* **RSpec Rails**
* **FactoryBot Rails**
* **Faker**
* **Shoulda Matchers**
* **Database Cleaner**

### Infrastructure

* **Docker**
* **Docker Compose**

The project's dependency configuration explicitly includes PostgreSQL, Redis, Sidekiq, Sidekiq Cron, RSpec, FactoryBot, Faker, Shoulda Matchers, and Database Cleaner.

## 🏗️ Architecture

Cart API follows a conventional Ruby on Rails backend architecture.

The API receives HTTP requests, applies the application's business rules, persists data through Active Record, and returns the resulting cart information.

Background processing is separated from the synchronous HTTP request cycle using Sidekiq and Redis.

```text
┌─────────────────────────┐
│                         │
│       API Client        │
│                         │
│  Postman / Frontend /   │
│       HTTP Client       │
│                         │
└────────────┬────────────┘
             │
             │ HTTP / JSON
             ▼
┌─────────────────────────┐
│                         │
│      Rails API          │
│                         │
│ Controllers             │
│ Models                  │
│ Business Rules          │
│                         │
└────────────┬────────────┘
             │
             │ Active Record
             ▼
┌─────────────────────────┐
│                         │
│      PostgreSQL         │
│                         │
│ Products                │
│ Carts                   │
│ Cart Items              │
│                         │
└─────────────────────────┘

             ▲
             │
             │
┌────────────┴────────────┐
│                         │
│       Sidekiq           │
│                         │
│ Background Jobs         │
│                         │
└────────────┬────────────┘
             │
             │
             ▼
┌─────────────────────────┐
│                         │
│         Redis           │
│                         │
│ Job Queue / Scheduling  │
│                         │
└─────────────────────────┘
```

This architecture keeps user-facing API requests separate from operations that can be processed asynchronously.

## 🧩 Domain Model

The application is centered around three main entities:

```text
Product
   │
   │
   ▼
CartItem
   │
   │
   ▼
Cart
```

### Product

Represents a product available in the e-commerce catalog.

### Cart

Represents a customer's shopping cart and maintains its current state.

A cart can have one of the following statuses:

* `active`
* `abandoned`
* `completed`

The `Cart` model also tracks the last interaction with the cart and provides domain methods for adding, removing, updating, and clearing products.

### CartItem

Represents a product added to a cart.

Each cart item contains:

* Product reference
* Cart reference
* Quantity
* Calculated subtotal

The application prevents the same product from being duplicated within a cart through a scoped uniqueness validation. Quantity is also validated as a positive integer with an upper limit.

## 🔌 API Endpoints

The API exposes endpoints for the main shopping cart operations.

### Add a product to the cart

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

Adds the specified quantity of a product to the cart.

### Get the current cart

```http
GET /cart
```

Returns the cart, its items, quantities, subtotals, and total value.

### Update product quantity

```http
POST /cart/add_item
Content-Type: application/json
```

```json
{
  "product_id": 1,
  "quantity": 3
}
```

Updates the quantity of a product already associated with the cart.

### Remove a product

```http
DELETE /cart/1
```

Removes the specified cart item.

The endpoint definitions and current request examples are documented in the project's existing README.

## 💰 Cart Calculation

The cart is responsible for calculating its total from the persisted cart items and their associated product prices.

```text
Product price × Item quantity
                ↓
          Item subtotal
                ↓
       Sum of cart items
                ↓
           Cart total
```

For example:

```text
Product A
R$ 10.00 × 2 = R$ 20.00

Product B
R$ 25.50 × 1 = R$ 25.50

-----------------------
Cart total = R$ 45.50
```

The `Cart` model exposes methods for calculating the total, counting items, checking whether the cart is empty, and generating a detailed summary.

## 🔄 Cart Lifecycle

A cart progresses through different states during its lifecycle.

```text
                 ┌──────────────┐
                 │    ACTIVE    │
                 └──────┬───────┘
                        │
                        │ Inactive
                        ▼
                 ┌──────────────┐
                 │  ABANDONED   │
                 └──────┬───────┘
                        │
                        │ Retention period
                        ▼
                 ┌──────────────┐
                 │   REMOVED    │
                 └──────────────┘

ACTIVE
  │
  │ Checkout completed
  ▼
COMPLETED
```

The cart model defines explicit statuses and scopes for active, abandoned, and completed carts. It also tracks interaction timestamps and provides methods for transitioning the cart to abandoned or completed states.

## ⚙️ Background Jobs

One of the main technical aspects of this project is the use of asynchronous background processing.

The application uses **Sidekiq** backed by **Redis** to process jobs outside the HTTP request lifecycle.

Scheduled processing is configured with **Sidekiq Cron**.

The current business rules include:

* Detect carts that have been inactive for more than 3 hours
* Mark inactive carts as abandoned
* Remove carts that have remained abandoned for more than 7 days

This allows maintenance operations to run automatically without blocking API requests.

### Why background jobs?

Operations such as cart cleanup do not need to happen while a user is waiting for an API response.

Instead:

```text
Scheduled task
      ↓
Sidekiq
      ↓
Background job
      ↓
Find eligible carts
      ↓
Update / remove records
```

This is a practical example of separating **user-facing operations** from **maintenance and asynchronous processing**.

## 🧠 Business Rules

The project implements several domain-level rules directly in the application.

### Cart status

Only the following statuses are accepted:

```text
active
abandoned
completed
```

### Product quantity

Cart item quantities must:

* Be present
* Be integers
* Be greater than zero
* Be less than or equal to 999

### Duplicate products

A product cannot have multiple cart item records within the same cart.

When a product is added again, the existing cart item quantity is increased instead of creating another cart item.

### Cart interaction

Cart interactions update the `last_interaction_at` timestamp, allowing the application to identify stale carts.

These rules are implemented through Active Record validations, associations, scopes, callbacks, and domain methods.

## 📁 Project Structure

```text
cart-api/
├── app/
│   ├── controllers/
│   │   └── ...              # HTTP/API controllers
│   │
│   ├── jobs/
│   │   └── ...              # Background jobs
│   │
│   ├── models/
│   │   ├── cart.rb
│   │   ├── cart_item.rb
│   │   └── product.rb
│   │
│   └── views/
│
├── config/
│   ├── routes.rb             # API routes
│   └── ...
│
├── db/
│   ├── migrate/              # Database migrations
│   └── ...
│
├── spec/
│   ├── factories/             # FactoryBot factories
│   ├── models/                # Model specifications
│   ├── support/               # Test support configuration
│   ├── rails_helper.rb
│   └── spec_helper.rb
│
├── Dockerfile
├── Gemfile
├── Gemfile.lock
├── .ruby-version
├── CHANGELOG.md
└── README.md
```

The repository includes dedicated `app`, `db`, `spec`, Docker, and configuration directories following the standard Rails project structure.

## 🧪 Testing

The project uses **RSpec** as its testing framework, together with **FactoryBot**, **Shoulda Matchers**, and **Database Cleaner**.

Run the complete test suite:

```bash
bundle exec rspec
```

Run tests with detailed output:

```bash
bundle exec rspec --format documentation
```

Generate an HTML test report:

```bash
bundle exec rspec --format html --out coverage/index.html
```

The test setup is organized under the `spec/` directory with factories, model specifications, and shared support configuration.

## 🐳 Getting Started

### Prerequisites

For the Docker-based workflow, install:

* Docker
* Docker Compose

For local development without Docker:

* Ruby 3.3.1
* Rails 7.1.3.2
* PostgreSQL 16
* Redis 7
* Bundler

### 1. Clone the repository

```bash
git clone https://github.com/digidweb/cart-api.git
cd cart-api
```

### 2. Run with Docker

Build and start the application:

```bash
docker-compose up --build
```

In another terminal, create the database and run the migrations:

```bash
docker-compose exec web rails db:create db:migrate
```

The project currently recommends Docker and Docker Compose as the preferred development environment.

### 3. Run locally without Docker

Install the Ruby dependencies:

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

In another terminal, start Sidekiq:

```bash
bundle exec sidekiq
```

Then start Rails:

```bash
bundle exec rails server
```

The Rails application will be available at:

```text
http://localhost:3000
```

## 🧪 Development Workflow

A typical development workflow is:

```text
Change application code
        ↓
Run RSpec
        ↓
Verify business rules
        ↓
Run Rails application
        ↓
Exercise API endpoints
        ↓
Verify background jobs
        ↓
Review database state
```

This workflow provides practical experience with the complete backend development cycle, from implementing domain logic to testing API behavior and asynchronous processing.

## 💡 Technical Takeaways

The main goal of Cart API was to practice building a backend application around a realistic e-commerce domain.

The project provides hands-on experience with:

* Designing RESTful API endpoints
* Modeling relationships with Active Record
* Implementing business rules inside domain models
* Working with PostgreSQL
* Validating API-related data
* Managing cart state and lifecycle
* Calculating totals from related records
* Using scopes to query domain states
* Using callbacks to maintain timestamps
* Processing background jobs with Sidekiq
* Scheduling recurring jobs with Sidekiq Cron
* Using Redis as a job-processing backend
* Writing automated tests with RSpec
* Creating test data with FactoryBot and Faker
* Running the application with Docker

## 🎯 Project Goals

The project was created as a technical challenge focused on backend development and e-commerce domain modeling.

The main objectives were to demonstrate the ability to:

1. Build a RESTful API with Ruby on Rails
2. Model a shopping cart domain using relational data
3. Implement business rules and validations
4. Persist data with PostgreSQL
5. Process asynchronous tasks with Sidekiq
6. Schedule recurring maintenance tasks
7. Write automated tests with RSpec
8. Provide a reproducible development environment with Docker

## 🔮 Potential Improvements

The current implementation provides the core shopping cart functionality, while several improvements could make the API closer to a production-ready e-commerce service.

Possible next steps include:

* Add request/integration specs for all API endpoints
* Add API authentication and authorization
* Associate carts with users
* Add product stock validation
* Prevent adding quantities beyond available inventory
* Add price snapshotting to cart items
* Add standardized API error responses
* Add API versioning
* Add pagination where applicable
* Add API documentation with OpenAPI/Swagger
* Add request rate limiting
* Add structured application logging
* Add GitHub Actions for automated testing
* Add RuboCop to the CI pipeline
* Add test coverage reporting
* Add production deployment configuration
* Add monitoring for Sidekiq jobs

## 📌 Project Context

Cart API is a personal backend project focused on **Ruby on Rails, REST API development, relational data modeling, automated testing, and asynchronous processing**.

The project intentionally uses a realistic e-commerce scenario to demonstrate how a Rails application can combine synchronous API operations with background processing.

It complements projects in this portfolio that explore different aspects of software development, including full-stack React/Rails integration, domain-driven business rules, testing, and application architecture.

## 🚀 What This Project Demonstrates

From a backend engineering perspective, Cart API demonstrates experience with:

**Ruby on Rails · REST APIs · PostgreSQL · Active Record · Redis · Sidekiq · Sidekiq Cron · RSpec · FactoryBot · Docker**

More importantly, it demonstrates the ability to translate business requirements into application behavior:

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
