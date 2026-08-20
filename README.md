# README

# Shopping Cart API

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

This project was developed as part of a technical challenge.
