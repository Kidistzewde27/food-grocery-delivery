# Food & Grocery Delivery Platform

## Database Design Document

**Project:** Food & Grocery Delivery Platform (PWA)

**Team:** Team 3 – 2026 Summer Internship

---

## 1. Introduction

This document describes the database design for the Food & Grocery Delivery Platform. It serves as the foundation for backend development and provides a shared understanding of the database structure before implementation.

The design is based on the approved project requirements and is intended to support all core user roles:

- Customer
- Restaurant Manager
- Driver
- Administrator

The goal of this document is to define the database tables, relationships, and business rules that will be implemented using Laravel migrations and MySQL.

## 2. Database Design Goals

The database is designed to:

- Support all four user roles using a single users table.
- Maintain data integrity through primary and foreign keys.
- Prevent duplicate and inconsistent data.
- Store historical order information accurately.
- Support role-based access control.
- Allow future system expansion without major redesign.
- Provide a reliable foundation for the Laravel REST API.

## 3. Database Tables

The system consists of the following application tables:

1. users
2. restaurants
3. categories
4. menu_items
5. cart_items
6. orders
7. order_items
8. driver_profiles

Laravel will also generate several system tables, including:

- personal_access_tokens
- password_reset_tokens
- failed_jobs
- cache (optional)

### 3.1 users

**Purpose**

Stores all registered users of the system. A single table is used for Customers, Restaurant Managers, Drivers, and Administrators. User roles determine each user's permissions within the platform.

| Column Name | Data Type    | Constraints                                 | Description                                     |
| ----------- | ------------ | ------------------------------------------- | ----------------------------------------------- |
| id          | BIGINT       | Primary Key, Auto Increment                 | Unique identifier for each user.                |
| name        | VARCHAR(255) | NOT NULL                                    | Full name of the user.                          |
| email       | VARCHAR(255) | NOT NULL, UNIQUE                            | User's email address used for login.            |
| phone       | VARCHAR(20)  | UNIQUE                                      | User's contact phone number.                    |
| password    | VARCHAR(255) | NOT NULL                                    | Encrypted (hashed) password.                    |
| role        | ENUM         | customer, restaurant_manager, driver, admin | Defines the user's role in the system.          |
| status      | ENUM         | active, inactive, suspended                 | Indicates whether the user's account is active. |
| created_at  | TIMESTAMP    | Auto Generated                              | Record creation timestamp.                      |
| updated_at  | TIMESTAMP    | Auto Generated                              | Record update timestamp.                        |

**Relationships**

- One Customer can place many orders.
- One Customer can have many cart items.
- One Restaurant Manager manages one restaurant.
- One Driver has one driver profile.
- One Driver can deliver many orders.

### 3.2 restaurants

**Purpose**

Stores information about restaurants and grocery stores registered on the platform. Each restaurant is managed by one Restaurant Manager and must be approved by an Administrator before customers can place orders.

| Column Name     | Data Type    | Constraints                      | Description                                         |
| --------------- | ------------ | -------------------------------- | --------------------------------------------------- |
| id              | BIGINT       | Primary Key, Auto Increment      | Unique identifier for each restaurant.              |
| manager_id      | BIGINT       | NOT NULL, Foreign Key → users.id | Restaurant Manager responsible for this restaurant. |
| name            | VARCHAR(255) | NOT NULL                         | Restaurant or store name.                           |
| description     | TEXT         | NULL                             | Description of the restaurant.                      |
| address         | TEXT         | NOT NULL                         | Restaurant address.                                 |
| phone           | VARCHAR(20)  | NOT NULL                         | Restaurant contact number.                          |
| logo            | VARCHAR(255) | NULL                             | Restaurant logo image path.                         |
| approval_status | ENUM         | pending, approved, rejected      | Approval decision by the Administrator.             |
| status          | ENUM         | active, inactive, suspended      | Operational status after approval.                  |
| created_at      | TIMESTAMP    | Auto Generated                   | Record creation timestamp.                          |
| updated_at      | TIMESTAMP    | Auto Generated                   | Record update timestamp.                            |

**Relationships**

- One Restaurant Manager manages one restaurant.
- One Restaurant has many menu items.
- One Restaurant receives many orders.

### 3.3 categories

**Purpose**

Stores the categories used to organize menu items.

| Column Name | Data Type    | Constraints                 | Description                          |
| ----------- | ------------ | --------------------------- | ------------------------------------ |
| id          | BIGINT       | Primary Key, Auto Increment | Unique identifier for each category. |
| name        | VARCHAR(100) | NOT NULL, UNIQUE            | Category name.                       |
| description | TEXT         | NULL                        | Optional category description.       |
| created_at  | TIMESTAMP    | Auto Generated              | Record creation timestamp.           |
| updated_at  | TIMESTAMP    | Auto Generated              | Record update timestamp.             |

**Relationships**

- One Category contains many menu items.

### 3.4 menu_items

**Purpose**

Stores all food and grocery items offered by restaurants.

| Column Name        | Data Type     | Constraints                            | Description                           |
| ------------------ | ------------- | -------------------------------------- | ------------------------------------- |
| id                 | BIGINT        | Primary Key, Auto Increment            | Unique identifier for each menu item. |
| restaurant_id      | BIGINT        | NOT NULL, Foreign Key → restaurants.id | Restaurant that owns the item.        |
| category_id        | BIGINT        | NOT NULL, Foreign Key → categories.id  | Category of the menu item.            |
| name               | VARCHAR(255)  | NOT NULL                               | Menu item name.                       |
| description        | TEXT          | NULL                                   | Item description.                     |
| price              | DECIMAL(10,2) | NOT NULL                               | Current selling price.                |
| quantity_available | INT           | DEFAULT 0                              | Number of available units.            |
| stock_status       | ENUM          | in_stock, out_of_stock                 | Current availability status.          |
| image              | VARCHAR(255)  | NULL                                   | Item image path.                      |
| created_at         | TIMESTAMP     | Auto Generated                         | Record creation timestamp.            |
| updated_at         | TIMESTAMP     | Auto Generated                         | Record update timestamp.              |

**Relationships**

- One Restaurant has many menu items.
- One Category contains many menu items.
- One Menu Item can appear in many cart items.
- One Menu Item can appear in many order items.

### 3.5 cart_items

**Purpose**

Stores the current shopping cart for customers before checkout.

| Column Name  | Data Type | Constraints                           | Description                           |
| ------------ | --------- | ------------------------------------- | ------------------------------------- |
| id           | BIGINT    | Primary Key, Auto Increment           | Unique identifier for each cart item. |
| customer_id  | BIGINT    | NOT NULL, Foreign Key → users.id      | Customer who owns the cart item.      |
| menu_item_id | BIGINT    | NOT NULL, Foreign Key → menu_items.id | Selected menu item.                   |
| quantity     | INT       | NOT NULL, DEFAULT 1                   | Quantity selected by the customer.    |
| created_at   | TIMESTAMP | Auto Generated                        | Record creation timestamp.            |
| updated_at   | TIMESTAMP | Auto Generated                        | Record update timestamp.              |

**Relationships**

- One Customer has many cart items.
- One Menu Item appears in many cart items.

### 3.6 orders

**Purpose**

Stores customer orders from placement until delivery.

| Column Name      | Data Type     | Constraints                                                           | Description                           |
| ---------------- | ------------- | --------------------------------------------------------------------- | ------------------------------------- |
| id               | BIGINT        | Primary Key, Auto Increment                                           | Unique identifier for each order.     |
| customer_id      | BIGINT        | NOT NULL, Foreign Key → users.id                                      | Customer who placed the order.        |
| restaurant_id    | BIGINT        | NOT NULL, Foreign Key → restaurants.id                                | Restaurant receiving the order.       |
| driver_id        | BIGINT        | NULL, Foreign Key → users.id                                          | Assigned driver.                      |
| subtotal         | DECIMAL(10,2) | NOT NULL                                                              | Total cost of ordered items.          |
| delivery_fee     | DECIMAL(10,2) | DEFAULT 0.00                                                          | Delivery charge.                      |
| total_amount     | DECIMAL(10,2) | NOT NULL                                                              | Final amount to be paid.              |
| delivery_address | TEXT          | NOT NULL                                                              | Delivery destination.                 |
| delivery_notes   | TEXT          | NULL                                                                  | Optional delivery instructions.       |
| phone            | VARCHAR(20)   | NOT NULL                                                              | Customer contact number for delivery. |
| status           | ENUM          | pending, preparing, ready_for_pickup, in_transit, delivered, rejected | Current order status.                 |
| delivered_at     | TIMESTAMP     | NULL                                                                  | Time when the order was delivered.    |
| created_at       | TIMESTAMP     | Auto Generated                                                        | Record creation timestamp.            |
| updated_at       | TIMESTAMP     | Auto Generated                                                        | Record update timestamp.              |

**Relationships**

- One Customer places many orders.
- One Restaurant receives many orders.
- One Driver delivers many orders.
- One Order contains many order items.

### 3.7 order_items

**Purpose**

Stores each individual item included in an order. Item prices are copied from the menu at checkout to preserve the original receipt.

| Column Name  | Data Type     | Constraints                           | Description                             |
| ------------ | ------------- | ------------------------------------- | --------------------------------------- |
| id           | BIGINT        | Primary Key, Auto Increment           | Unique identifier for each order item.  |
| order_id     | BIGINT        | NOT NULL, Foreign Key → orders.id     | Order that contains this item.          |
| menu_item_id | BIGINT        | NOT NULL, Foreign Key → menu_items.id | Purchased menu item.                    |
| quantity     | INT           | NOT NULL                              | Quantity ordered.                       |
| unit_price   | DECIMAL(10,2) | NOT NULL                              | Price at the time the order was placed. |
| subtotal     | DECIMAL(10,2) | NOT NULL                              | Quantity × unit price.                  |
| created_at   | TIMESTAMP     | Auto Generated                        | Record creation timestamp.              |
| updated_at   | TIMESTAMP     | Auto Generated                        | Record update timestamp.                |

**Relationships**

- One Order contains many order items.
- One Menu Item appears in many order items.

### 3.8 driver_profiles

**Purpose**

Stores additional information specific to drivers.

| Column Name    | Data Type    | Constraints                              | Description                                                     |
| -------------- | ------------ | ---------------------------------------- | --------------------------------------------------------------- |
| id             | BIGINT       | Primary Key, Auto Increment              | Unique identifier for each driver profile.                      |
| user_id        | BIGINT       | NOT NULL, Foreign Key → users.id, UNIQUE | Driver associated with this profile.                            |
| vehicle_type   | VARCHAR(100) | NOT NULL                                 | Type of vehicle used for deliveries.                            |
| license_number | VARCHAR(100) | NULL                                     | Driver's license number (optional).                             |
| is_online      | BOOLEAN      | DEFAULT FALSE                            | Indicates whether the driver is available to accept deliveries. |
| created_at     | TIMESTAMP    | Auto Generated                           | Record creation timestamp.                                      |
| updated_at     | TIMESTAMP    | Auto Generated                           | Record update timestamp.                                        |

**Relationships**

- One Driver has one driver profile.
