# airbnb-clone-project
This is a repo for a clone of and airbnb app

# Team Roles
- Backend Developer: Responsible for implementing API endpoints, database schemas, and business logic.
- Database Administrator: Manages database design, indexing, and optimizations.
- DevOps Engineer: Handles deployment, monitoring, and scaling of the backend services.
- QA Engineer: Ensures the backend functionalities are thoroughly tested and meet quality standards.

# Technology Stack
- Django: A high-level Python web framework used for building the RESTful API.
- Django REST Framework: Provides tools for creating and managing RESTful APIs.
- PostgreSQL: A powerful relational database used for data storage.
- GraphQL: Allows for flexible and efficient querying of data.
- Celery: For handling asynchronous tasks such as sending notifications or processing payments.
- Redis: Used for caching and session management.
- Docker: Containerization tool for consistent development and deployment environments.
- CI/CD Pipelines: Automated pipelines for testing and deploying code changes.

# Database Design
---
## 1. Users Table

**Purpose:** Stores user information and authentication data.

| Field Name     | Data Type    | Constraints / Notes            |
| -------------- | ------------ | ------------------------------ |
| id             | UUID / PK    | Primary Key, unique identifier |
| username       | VARCHAR(50)  | Unique, not null               |
| email          | VARCHAR(100) | Unique, not null               |
| password\_hash | VARCHAR(255) | Hashed password, not null      |
| first\_name    | VARCHAR(50)  | Optional                       |
| last\_name     | VARCHAR(50)  | Optional                       |
| date\_joined   | TIMESTAMP    | Default: CURRENT\_TIMESTAMP    |
| is\_active     | BOOLEAN      | Default: true                  |
| is\_staff      | BOOLEAN      | Default: false                 |

**Indexes:**

* Unique index on `username` and `email` for quick lookups.
* Optional index on `is_active` for filtering active users.

---

## 2. Properties Table

**Purpose:** Stores information about listed properties.

| Field Name        | Data Type     | Constraints / Notes                   |
| ----------------- | ------------- | ------------------------------------- |
| id                | UUID / PK     | Primary Key                           |
| owner\_id         | UUID / FK     | References `users(id)`, not null      |
| title             | VARCHAR(255)  | Not null                              |
| description       | TEXT          | Optional                              |
| address           | VARCHAR(255)  | Optional                              |
| city              | VARCHAR(100)  | Optional                              |
| state             | VARCHAR(100)  | Optional                              |
| country           | VARCHAR(100)  | Optional                              |
| zip\_code         | VARCHAR(20)   | Optional                              |
| price\_per\_night | DECIMAL(10,2) | Not null                              |
| created\_at       | TIMESTAMP     | Default: CURRENT\_TIMESTAMP           |
| updated\_at       | TIMESTAMP     | Default: CURRENT\_TIMESTAMP on update |
| is\_active        | BOOLEAN       | Default: true                         |

**Indexes:**

* Index on `owner_id` for fetching user properties.
* Index on `city`, `state`, and `country` for search queries.
* Index on `price_per_night` if filtering by price.

---

## 3. Bookings Table

**Purpose:** Tracks property bookings by users.

| Field Name   | Data Type     | Constraints / Notes                   |
| ------------ | ------------- | ------------------------------------- |
| id           | UUID / PK     | Primary Key                           |
| user\_id     | UUID / FK     | References `users(id)`, not null      |
| property\_id | UUID / FK     | References `properties(id)`, not null |
| check\_in    | DATE          | Not null                              |
| check\_out   | DATE          | Not null                              |
| total\_price | DECIMAL(10,2) | Calculated from `price_per_night`     |
| status       | VARCHAR(20)   | Values: pending, confirmed, canceled  |
| created\_at  | TIMESTAMP     | Default: CURRENT\_TIMESTAMP           |
| updated\_at  | TIMESTAMP     | Default: CURRENT\_TIMESTAMP on update |

**Indexes:**

* Composite index on `(property_id, check_in, check_out)` for fast availability checks.
* Index on `user_id` for user-specific bookings.

---

## 4. Payments Table

**Purpose:** Tracks payments associated with bookings.

| Field Name      | Data Type     | Constraints / Notes                     |
| --------------- | ------------- | --------------------------------------- |
| id              | UUID / PK     | Primary Key                             |
| booking\_id     | UUID / FK     | References `bookings(id)`, not null     |
| amount          | DECIMAL(10,2) | Not null                                |
| payment\_method | VARCHAR(50)   | e.g., credit\_card, paypal              |
| status          | VARCHAR(20)   | Values: pending, completed, failed      |
| transaction\_id | VARCHAR(100)  | Optional, for external payment provider |
| paid\_at        | TIMESTAMP     | When payment was made                   |
| created\_at     | TIMESTAMP     | Default: CURRENT\_TIMESTAMP             |

**Indexes:**

* Index on `booking_id` to quickly find payments per booking.
* Index on `status` to track pending payments.

---

## 5. Reviews Table

**Purpose:** Stores user reviews for properties.

| Field Name   | Data Type | Constraints / Notes                   |
| ------------ | --------- | ------------------------------------- |
| id           | UUID / PK | Primary Key                           |
| user\_id     | UUID / FK | References `users(id)`, not null      |
| property\_id | UUID / FK | References `properties(id)`, not null |
| rating       | INT       | Not null, values 1-5                  |
| comment      | TEXT      | Optional                              |
| created\_at  | TIMESTAMP | Default: CURRENT\_TIMESTAMP           |
| updated\_at  | TIMESTAMP | Default: CURRENT\_TIMESTAMP on update |

**Indexes:**

* Composite index on `(property_id, user_id)` to prevent multiple reviews per property per user.
* Index on `rating` for analytics.

---

## 6. Database Relationships

* **Users → Properties**: One-to-Many (a user can own many properties).
* **Users → Bookings**: One-to-Many (a user can make many bookings).
* **Properties → Bookings**: One-to-Many (a property can have many bookings).
* **Bookings → Payments**: One-to-One / One-to-Many (a booking can have one or multiple payments).
* **Users → Reviews → Properties**: Many-to-Many via reviews (a user can review multiple properties; a property can have multiple reviews).

---

## 7. Indexing & Performance

* Use **composite indexes** where frequent filtering occurs (e.g., property availability queries).
* Index **foreign keys** for faster JOIN operations.
* Add **unique constraints** on username, email, and transaction\_id.
* Consider caching frequently accessed data like property listings or top-rated properties.
* Use database transactions for booking + payment operations to maintain consistency.

---

## 8. Optional Enhancements

* **Audit Logs**: Table to track changes to bookings or payments.
* **Images Table**: Store property images with references to `properties(id)`.
* **Tags Table**: For categorizing properties (e.g., beach, luxury, family-friendly).

