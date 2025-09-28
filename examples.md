# DATABASE LANGUAGES - CODE EXAMPLES AND DIAGRAMS

_Technical Study Report Examples - DIVYANSH JHA (2024SL70022)_

---

## TABLE OF CONTENTS

1. [Three-Schema Architecture Diagram](#1-three-schema-architecture-diagram)
2. [Section A - Database Languages Overview](#2-section-a---database-languages-overview)
3. [Section B - Architecture Integration](#3-section-b---architecture-integration)
4. [Section C - Applications and Case Studies](#4-section-c---applications-and-case-studies)
5. [Section D - Challenges and Solutions](#5-section-d---challenges-and-solutions)
6. [Performance Comparison Tables](#6-performance-comparison-tables)

---

## 1. THREE-SCHEMA ARCHITECTURE DIAGRAM

**📍 Location: Section 2.4 Three-Schema Architecture Foundation**

```
┌─────────────────────────────────────────────────────────┐
│                    EXTERNAL LEVEL                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────┐   │
│  │ User View 1 │ │ User View 2 │ │ Application API │   │
│  │    (VDL)    │ │    (VDL)    │ │    Interface    │   │
│  └─────────────┘ └─────────────┘ └─────────────────┘   │
│                           │                            │
│                    Logical Mapping                     │
└─────────────────────────────────────────────────────────┘
                             │
┌─────────────────────────────────────────────────────────┐
│                   CONCEPTUAL LEVEL                     │
│  ┌─────────────────────────────────────────────────────┐ │
│  │        Complete Logical Database Schema            │ │
│  │               (DDL + DML)                          │ │
│  │  • Entity Relationships  • Business Rules         │ │
│  │  • Integrity Constraints • Data Types             │ │
│  │  • Security Policies     • Transaction Logic      │ │
│  └─────────────────────────────────────────────────────┘ │
│                           │                            │
│                   Physical Mapping                     │
└─────────────────────────────────────────────────────────┘
                             │
┌─────────────────────────────────────────────────────────┐
│                    INTERNAL LEVEL                      │
│  ┌─────────────────────────────────────────────────────┐ │
│  │           Physical Storage Implementation          │ │
│  │                    (SDL)                           │ │
│  │  • File Organization    • Access Methods           │ │
│  │  • Indexing Strategies  • Storage Allocation       │ │
│  │  • Compression          • Performance Tuning       │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## 2. SECTION A - DATABASE LANGUAGES OVERVIEW

### A.1 Storage Definition Language (SDL)

**📍 Location: SECTION A - A.1 Storage Definition Language (SDL)**

#### Oracle SDL Example - Tablespace and Storage Configuration

```sql
-- Creating tablespace with specific storage parameters
CREATE TABLESPACE sales_data
DATAFILE '/opt/oracle/oradata/salesdb01.dbf' SIZE 1G
AUTOEXTEND ON NEXT 100M MAXSIZE 10G
BLOCKSIZE 8K
EXTENT MANAGEMENT LOCAL AUTOALLOCATE
SEGMENT SPACE MANAGEMENT AUTO;

-- Creating table with storage specifications
CREATE TABLE sales_transactions (
    transaction_id NUMBER(10) PRIMARY KEY,
    customer_id NUMBER(10),
    product_id NUMBER(10),
    sale_date DATE,
    amount NUMBER(10,2)
)
TABLESPACE sales_data
STORAGE (
    INITIAL 10M
    NEXT 5M
    PCTINCREASE 10
    MAXEXTENTS 100
)
PCTFREE 10 PCTUSED 80;

-- Creating index with storage parameters
CREATE INDEX idx_sales_date ON sales_transactions(sale_date)
TABLESPACE sales_data
STORAGE (INITIAL 2M NEXT 1M)
PCTFREE 20;
```

### A.2 Data Definition Language (DDL)

**📍 Location: SECTION A - A.2 Data Definition Language (DDL)**

#### Comprehensive Database Schema Creation

```sql
-- Creating comprehensive database schema
CREATE SCHEMA ecommerce_db;

-- Creating tables with complete constraint definitions
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    registration_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    credit_limit DECIMAL(10,2) DEFAULT 1000.00,

    -- Check constraints for business rules
    CONSTRAINT chk_email_format CHECK (email REGEXP '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT chk_credit_limit CHECK (credit_limit >= 0)
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(255) NOT NULL,
    category_id INT,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock_quantity INT DEFAULT 0,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT chk_price_positive CHECK (price > 0),
    CONSTRAINT chk_stock_non_negative CHECK (stock_quantity >= 0),

    FOREIGN KEY (category_id) REFERENCES categories(category_id)
        ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    total_amount DECIMAL(12,2) NOT NULL,
    shipping_address TEXT NOT NULL,
    payment_method ENUM('credit_card', 'debit_card', 'paypal', 'cash_on_delivery'),

    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE RESTRICT ON UPDATE CASCADE,

    CONSTRAINT chk_total_positive CHECK (total_amount > 0)
);
```

#### Advanced DDL Features - Triggers and Procedures

```sql
-- Creating triggers for business logic
DELIMITER //
CREATE TRIGGER update_stock_after_order
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock_quantity = stock_quantity - NEW.quantity,
        last_updated = CURRENT_TIMESTAMP
    WHERE product_id = NEW.product_id;

    -- Log inventory change
    INSERT INTO inventory_log (product_id, change_type, quantity_changed, timestamp)
    VALUES (NEW.product_id, 'sale', -NEW.quantity, NOW());
END//
DELIMITER ;

-- Creating stored procedures for complex operations
CREATE PROCEDURE process_bulk_order(
    IN p_customer_id INT,
    IN p_product_list JSON,
    OUT p_order_id INT,
    OUT p_status VARCHAR(50)
)
BEGIN
    DECLARE v_total_amount DECIMAL(12,2) DEFAULT 0;
    DECLARE v_stock_available BOOLEAN DEFAULT TRUE;
    DECLARE done INT DEFAULT FALSE;

    -- Transaction management
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_status = 'ERROR: Transaction failed';
    END;

    START TRANSACTION;

    -- Validate stock availability
    -- Process order creation
    -- Update inventory
    -- Calculate totals

    COMMIT;
    SET p_status = 'SUCCESS';
END;

-- Creating views for data abstraction
CREATE VIEW customer_order_summary AS
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_spent,
    MAX(o.order_date) as last_order_date,
    AVG(o.total_amount) as avg_order_value
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.status = 'active'
GROUP BY c.customer_id, c.first_name, c.last_name, c.email;
```

#### Schema Evolution and Maintenance

```sql
-- Adding new columns with default values
ALTER TABLE customers
ADD COLUMN loyalty_points INT DEFAULT 0,
ADD COLUMN preferred_contact ENUM('email', 'sms', 'phone') DEFAULT 'email';

-- Modifying existing constraints
ALTER TABLE orders
MODIFY COLUMN status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'returned') DEFAULT 'pending';

-- Creating composite indexes for performance
CREATE INDEX idx_order_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_product_category_price ON products(category_id, price);

-- Adding partitioning for large tables
ALTER TABLE orders
PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION future VALUES LESS THAN MAXVALUE
);
```

### A.3 Data Manipulation Language (DML)

**📍 Location: SECTION A - A.3 Data Manipulation Language (DML)**

#### Basic Data Retrieval and Complex Joins

```sql
-- Basic data retrieval
SELECT customer_id, first_name, last_name, email
FROM customers
WHERE status = 'active'
ORDER BY last_name, first_name;

-- Complex joins with multiple tables
SELECT
    c.first_name + ' ' + c.last_name as customer_name,
    c.email,
    o.order_id,
    o.order_date,
    o.status as order_status,
    o.total_amount,
    p.product_name,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) as line_total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= '2024-01-01'
    AND o.status IN ('shipped', 'delivered')
ORDER BY o.order_date DESC, c.last_name;
```

#### Advanced Analytical Queries with Window Functions

```sql
-- Advanced analytical queries
SELECT
    EXTRACT(YEAR FROM order_date) as order_year,
    EXTRACT(MONTH FROM order_date) as order_month,
    COUNT(*) as total_orders,
    SUM(total_amount) as revenue,
    AVG(total_amount) as avg_order_value,
    COUNT(DISTINCT customer_id) as unique_customers,

    -- Window functions for advanced analytics
    SUM(total_amount) OVER (
        PARTITION BY EXTRACT(YEAR FROM order_date)
        ORDER BY EXTRACT(MONTH FROM order_date)
        ROWS UNBOUNDED PRECEDING
    ) as running_yearly_total,

    LAG(SUM(total_amount), 1) OVER (
        ORDER BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)
    ) as previous_month_revenue,

    RANK() OVER (
        ORDER BY SUM(total_amount) DESC
    ) as revenue_rank
FROM orders
WHERE status = 'delivered'
GROUP BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)
ORDER BY order_year DESC, order_month DESC;
```

#### Data Modification Operations

```sql
-- Bulk data insertion
INSERT INTO products (product_name, category_id, description, price, stock_quantity)
VALUES
    ('Laptop Pro 15"', 1, 'High-performance laptop with 16GB RAM', 1299.99, 25),
    ('Wireless Mouse', 2, 'Ergonomic wireless mouse with USB receiver', 29.99, 150),
    ('USB-C Hub', 2, '7-in-1 USB-C hub with HDMI and Ethernet', 79.99, 75),
    ('Monitor 27"', 1, '4K IPS monitor with USB-C connectivity', 399.99, 40);

-- Conditional updates with business logic
UPDATE customers
SET loyalty_points = loyalty_points + FLOOR(
    (SELECT COALESCE(SUM(total_amount), 0) FROM orders
     WHERE customer_id = customers.customer_id
       AND status = 'delivered'
       AND order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
    ) / 10
)
WHERE status = 'active';

-- Complex update with joins
UPDATE products p
INNER JOIN (
    SELECT
        oi.product_id,
        SUM(oi.quantity) as total_sold
    FROM order_items oi
    INNER JOIN orders o ON oi.order_id = o.order_id
    WHERE o.status = 'delivered'
      AND o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 1 MONTH)
    GROUP BY oi.product_id
) sales_data ON p.product_id = sales_data.product_id
SET p.stock_quantity = GREATEST(0, p.stock_quantity - sales_data.total_sold);

-- Conditional deletions with safety checks
DELETE FROM order_items
WHERE order_id IN (
    SELECT order_id FROM orders
    WHERE status = 'cancelled'
      AND order_date < DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH)
);
```

#### Complex Transaction Management

```sql
-- Complex transaction with error handling
START TRANSACTION;

-- Create order
INSERT INTO orders (customer_id, order_date, status, total_amount, shipping_address, payment_method)
VALUES (12345, NOW(), 'pending', 0, '123 Main St, City, State', 'credit_card');

SET @order_id = LAST_INSERT_ID();

-- Add order items and calculate total
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
SELECT @order_id, p.product_id, cart.quantity, p.price
FROM temp_cart cart
INNER JOIN products p ON cart.product_id = p.product_id
WHERE cart.session_id = 'user_session_123';

-- Update order total
UPDATE orders
SET total_amount = (
    SELECT SUM(quantity * unit_price)
    FROM order_items
    WHERE order_id = @order_id
)
WHERE order_id = @order_id;

-- Validate inventory
SELECT @stock_issue := COUNT(*)
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.product_id
WHERE oi.order_id = @order_id
  AND p.stock_quantity < oi.quantity;

IF @stock_issue > 0 THEN
    ROLLBACK;
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Insufficient inventory for order';
ELSE
    -- Update inventory
    UPDATE products p
    INNER JOIN order_items oi ON p.product_id = oi.product_id
    SET p.stock_quantity = p.stock_quantity - oi.quantity
    WHERE oi.order_id = @order_id;

    -- Clear cart
    DELETE FROM temp_cart WHERE session_id = 'user_session_123';

    COMMIT;
END IF;
```

#### Common Table Expressions (CTEs)

```sql
-- Common Table Expressions (CTEs) for complex logic
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) as sales_month,
        SUM(total_amount) as monthly_revenue,
        COUNT(*) as monthly_orders
    FROM orders
    WHERE status = 'delivered'
    GROUP BY DATE_TRUNC('month', order_date)
),
sales_growth AS (
    SELECT
        sales_month,
        monthly_revenue,
        LAG(monthly_revenue) OVER (ORDER BY sales_month) as previous_month,
        CASE
            WHEN LAG(monthly_revenue) OVER (ORDER BY sales_month) > 0
            THEN ROUND(
                ((monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY sales_month)) * 100.0 /
                 LAG(monthly_revenue) OVER (ORDER BY sales_month)), 2
            )
            ELSE 0
        END as growth_percentage
    FROM monthly_sales
)
SELECT * FROM sales_growth
WHERE sales_month >= '2024-01-01'
ORDER BY sales_month;
```

### A.4 View Definition Language (VDL)

**📍 Location: SECTION A - A.4 View Definition Language (VDL)**

#### Security and Access Control Views

```sql
-- Customer service representative view (limited customer data)
CREATE VIEW customer_service_view AS
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email,
    c.phone,
    c.registration_date,
    c.status,
    c.loyalty_points,
    COUNT(o.order_id) as total_orders,
    COALESCE(SUM(o.total_amount), 0) as lifetime_value,
    MAX(o.order_date) as last_order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.email,
         c.phone, c.registration_date, c.status, c.loyalty_points
WITH CHECK OPTION;

-- Executive dashboard view (aggregated business metrics)
CREATE VIEW executive_dashboard AS
SELECT
    'Today' as period_name,
    CURRENT_DATE as period_date,
    COUNT(DISTINCT o.customer_id) as active_customers,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_revenue,
    AVG(o.total_amount) as avg_order_value,
    (SELECT COUNT(*) FROM customers WHERE status = 'active') as total_active_customers,
    (SELECT COUNT(*) FROM products WHERE stock_quantity > 0) as products_in_stock
FROM orders o
WHERE DATE(o.order_date) = CURRENT_DATE
  AND o.status IN ('delivered', 'shipped')

UNION ALL

SELECT
    'This Month' as period_name,
    DATE_TRUNC('month', CURRENT_DATE) as period_date,
    COUNT(DISTINCT o.customer_id) as active_customers,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_revenue,
    AVG(o.total_amount) as avg_order_value,
    (SELECT COUNT(*) FROM customers WHERE status = 'active') as total_active_customers,
    (SELECT COUNT(*) FROM products WHERE stock_quantity > 0) as products_in_stock
FROM orders o
WHERE DATE_TRUNC('month', o.order_date) = DATE_TRUNC('month', CURRENT_DATE)
  AND o.status IN ('delivered', 'shipped');
```

#### Inventory Management and Analytics Views

```sql
-- Inventory management view for warehouse staff
CREATE VIEW inventory_management AS
SELECT
    p.product_id,
    p.product_name,
    c.category_name,
    p.stock_quantity,
    p.price,

    -- Calculate reorder point based on sales velocity
    COALESCE(sales_30days.avg_daily_sales * 14, 0) as suggested_reorder_point,

    -- Stock status classification
    CASE
        WHEN p.stock_quantity = 0 THEN 'OUT_OF_STOCK'
        WHEN p.stock_quantity <= COALESCE(sales_30days.avg_daily_sales * 7, 5) THEN 'LOW_STOCK'
        WHEN p.stock_quantity <= COALESCE(sales_30days.avg_daily_sales * 14, 10) THEN 'MEDIUM_STOCK'
        ELSE 'HIGH_STOCK'
    END as stock_status,

    -- Financial metrics
    p.stock_quantity * p.price as inventory_value,
    COALESCE(sales_30days.total_sold, 0) as units_sold_30days,
    COALESCE(sales_30days.revenue_30days, 0) as revenue_30days,

    -- Performance indicators
    CASE
        WHEN sales_30days.total_sold > 0
        THEN p.stock_quantity / (sales_30days.total_sold / 30.0)
        ELSE NULL
    END as days_of_inventory_remaining

FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
LEFT JOIN (
    SELECT
        oi.product_id,
        SUM(oi.quantity) as total_sold,
        AVG(oi.quantity) as avg_daily_sales,
        SUM(oi.quantity * oi.unit_price) as revenue_30days
    FROM order_items oi
    INNER JOIN orders o ON oi.order_id = o.order_id
    WHERE o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
      AND o.status = 'delivered'
    GROUP BY oi.product_id
) sales_30days ON p.product_id = sales_30days.product_id;
```

#### Materialized Views for Performance

```sql
-- Materialized view for customer analytics (refreshed daily)
CREATE MATERIALIZED VIEW customer_analytics_mv AS
SELECT
    c.customer_id,
    c.first_name + ' ' + c.last_name as customer_name,
    c.email,
    c.registration_date,
    c.status,

    -- Order statistics
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as lifetime_value,
    AVG(o.total_amount) as avg_order_value,
    MIN(o.order_date) as first_order_date,
    MAX(o.order_date) as last_order_date,

    -- Customer segmentation
    CASE
        WHEN SUM(o.total_amount) >= 10000 THEN 'VIP'
        WHEN SUM(o.total_amount) >= 5000 THEN 'PREMIUM'
        WHEN SUM(o.total_amount) >= 1000 THEN 'REGULAR'
        WHEN COUNT(o.order_id) > 0 THEN 'NEW'
        ELSE 'PROSPECT'
    END as customer_segment,

    -- Behavioral analysis
    DATEDIFF(CURRENT_DATE, MAX(o.order_date)) as days_since_last_order,
    COUNT(DISTINCT DATE_TRUNC('month', o.order_date)) as active_months,

    -- Purchase patterns
    (SELECT category_name FROM categories WHERE category_id =
        (SELECT category_id FROM products WHERE product_id =
            (SELECT product_id FROM order_items WHERE order_id IN
                (SELECT order_id FROM orders WHERE customer_id = c.customer_id)
             GROUP BY product_id ORDER BY SUM(quantity) DESC LIMIT 1)
        )
    ) as favorite_category

FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id AND o.status = 'delivered'
GROUP BY c.customer_id, c.first_name, c.last_name, c.email, c.registration_date, c.status;

-- Create refresh schedule for materialized view
CREATE EVENT refresh_customer_analytics
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY + INTERVAL 2 HOUR
DO REFRESH MATERIALIZED VIEW customer_analytics_mv;
```

### A.5 SQL as Unified Database Language

**📍 Location: SECTION A - A.5 SQL as Unified Database Language**

#### Complete Database Solution Implementation

```sql
-- DDL: Create comprehensive database structure
CREATE DATABASE ecommerce_platform;
USE ecommerce_platform;

-- Create enum types for data consistency
CREATE TYPE order_status_enum AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'returned');
CREATE TYPE payment_method_enum AS ENUM ('credit_card', 'debit_card', 'paypal', 'bank_transfer', 'cash_on_delivery');

-- DDL + SDL: Create optimized table structures
CREATE TABLE IF NOT EXISTS customers (
    customer_id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    date_of_birth DATE,
    registration_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active',
    loyalty_points INTEGER DEFAULT 0,
    credit_limit DECIMAL(10,2) DEFAULT 1000.00,
    preferred_language VARCHAR(5) DEFAULT 'en-US',
    marketing_consent BOOLEAN DEFAULT FALSE,

    -- Comprehensive constraints
    CONSTRAINT chk_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT chk_credit_limit CHECK (credit_limit >= 0),
    CONSTRAINT chk_loyalty_points CHECK (loyalty_points >= 0)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- DML: Populate with sample data
INSERT INTO customers (email, password_hash, first_name, last_name, phone, date_of_birth) VALUES
('john.doe@email.com', SHA2('password123', 256), 'John', 'Doe', '+1-555-0101', '1985-06-15'),
('jane.smith@email.com', SHA2('securepass', 256), 'Jane', 'Smith', '+1-555-0102', '1990-03-22'),
('bob.wilson@email.com', SHA2('mypassword', 256), 'Bob', 'Wilson', '+1-555-0103', '1988-11-08');

-- VDL: Create customer service view
CREATE VIEW customer_service_portal AS
SELECT
    customer_id,
    first_name + ' ' + last_name as full_name,
    email,
    phone,
    status,
    loyalty_points,
    registration_date,
    DATEDIFF(CURRENT_DATE, registration_date) AS days_as_customer
FROM customers
WHERE status != 'suspended';
```

### A.5 Procedural vs Declarative Languages

**📍 Location: SECTION A - A.6 Declarative vs Procedural Language Paradigms**

#### Declarative Example - Advanced Analytics

```sql
-- Complex analytical query - Declarative approach
-- System optimizes execution automatically
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_spent,
    AVG(o.total_amount) as avg_order_value,

    -- Window functions for advanced analytics
    RANK() OVER (ORDER BY SUM(o.total_amount) DESC) as spending_rank,
    PERCENT_RANK() OVER (ORDER BY SUM(o.total_amount)) as spending_percentile,

    -- Moving averages
    AVG(o.total_amount) OVER (
        PARTITION BY c.customer_id
        ORDER BY o.order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_order_value,

    -- Lead/Lag functions for trend analysis
    LAG(o.total_amount, 1) OVER (
        PARTITION BY c.customer_id
        ORDER BY o.order_date
    ) as previous_order_amount,

    -- Conditional aggregation
    SUM(CASE WHEN o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
             THEN o.total_amount ELSE 0 END) as spending_last_year,

    -- Complex case statements
    CASE
        WHEN COUNT(o.order_id) >= 50 AND SUM(o.total_amount) >= 10000 THEN 'VIP Customer'
        WHEN COUNT(o.order_id) >= 20 AND SUM(o.total_amount) >= 5000 THEN 'Premium Customer'
        WHEN COUNT(o.order_id) >= 10 THEN 'Regular Customer'
        WHEN COUNT(o.order_id) >= 1 THEN 'New Customer'
        ELSE 'Prospect'
    END as customer_classification

FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id AND o.status = 'delivered'
WHERE c.status = 'active'
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING SUM(o.total_amount) > 100  -- Only customers with meaningful spending
ORDER BY total_spent DESC, total_orders DESC;
```

#### Procedural Example - Complex Business Logic

```sql
-- Complex business logic with procedural control
DELIMITER //

CREATE PROCEDURE calculate_customer_rewards(
    IN p_customer_id INT,
    IN p_calculation_date DATE,
    OUT p_points_earned INT,
    OUT p_tier_status VARCHAR(20),
    OUT p_processing_status VARCHAR(100)
)
BEGIN
    -- Variable declarations
    DECLARE v_total_spent DECIMAL(12,2) DEFAULT 0;
    DECLARE v_order_count INT DEFAULT 0;
    DECLARE v_last_order_date DATE;
    DECLARE v_months_active INT DEFAULT 0;
    DECLARE v_base_points INT DEFAULT 0;
    DECLARE v_bonus_multiplier DECIMAL(3,2) DEFAULT 1.0;
    DECLARE v_tier_bonus INT DEFAULT 0;
    DECLARE v_error_count INT DEFAULT 0;

    -- Exception handling
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        GET DIAGNOSTICS CONDITION 1
            @sqlstate = RETURNED_SQLSTATE,
            @errno = MYSQL_ERRNO,
            @text = MESSAGE_TEXT;
        SET v_error_count = v_error_count + 1;
        SET p_processing_status = CONCAT('ERROR: ', @errno, ' - ', @text);
    END;

    -- Initialize output parameters
    SET p_points_earned = 0;
    SET p_tier_status = 'Bronze';
    SET p_processing_status = 'Processing...';

    -- Start transaction for data consistency
    START TRANSACTION;

    -- Step 1: Gather customer data
    SELECT
        COALESCE(SUM(total_amount), 0),
        COUNT(*),
        MAX(order_date),
        PERIOD_DIFF(
            DATE_FORMAT(p_calculation_date, '%Y%m'),
            DATE_FORMAT(MIN(order_date), '%Y%m')
        )
    INTO v_total_spent, v_order_count, v_last_order_date, v_months_active
    FROM orders
    WHERE customer_id = p_customer_id
      AND status = 'delivered'
      AND order_date <= p_calculation_date;

    -- Step 2: Calculate base points (1 point per $10 spent)
    SET v_base_points = FLOOR(v_total_spent / 10);

    -- Step 3: Apply tier-based multipliers
    IF v_total_spent >= 10000 AND v_order_count >= 25 THEN
        SET p_tier_status = 'VIP';
        SET v_bonus_multiplier = 2.5;
        SET v_tier_bonus = 1000;
    ELSEIF v_total_spent >= 5000 AND v_order_count >= 15 THEN
        SET p_tier_status = 'Gold';
        SET v_bonus_multiplier = 2.0;
        SET v_tier_bonus = 500;
    ELSEIF v_total_spent >= 1000 AND v_order_count >= 5 THEN
        SET p_tier_status = 'Silver';
        SET v_bonus_multiplier = 1.5;
        SET v_tier_bonus = 100;
    ELSE
        SET p_tier_status = 'Bronze';
        SET v_bonus_multiplier = 1.0;
        SET v_tier_bonus = 0;
    END IF;

    -- Step 4: Apply activity bonuses
    IF v_months_active >= 12 THEN
        SET v_bonus_multiplier = v_bonus_multiplier + 0.2;
    END IF;

    -- Step 5: Check for recent activity bonus
    IF DATEDIFF(p_calculation_date, v_last_order_date) <= 30 THEN
        SET v_tier_bonus = v_tier_bonus + 50;
    END IF;

    -- Step 6: Calculate final points
    SET p_points_earned = ROUND(v_base_points * v_bonus_multiplier) + v_tier_bonus;

    -- Step 7: Update customer record
    UPDATE customers
    SET
        loyalty_points = loyalty_points + p_points_earned,
        last_updated = p_calculation_date
    WHERE customer_id = p_customer_id;

    -- Step 8: Log the transaction
    INSERT INTO loyalty_transactions (
        customer_id,
        transaction_date,
        points_earned,
        tier_status,
        calculation_basis,
        created_at
    ) VALUES (
        p_customer_id,
        p_calculation_date,
        p_points_earned,
        p_tier_status,
        CONCAT('Spent: $', v_total_spent, ', Orders: ', v_order_count),
        NOW()
    );

    -- Commit transaction if no errors
    IF v_error_count = 0 THEN
        COMMIT;
        SET p_processing_status = CONCAT('SUCCESS: Awarded ', p_points_earned, ' points, Tier: ', p_tier_status);
    ELSE
        ROLLBACK;
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        SET p_processing_status = CONCAT('CRITICAL ERROR: ', SQLERRM);
        SET p_points_earned = 0;
        SET p_tier_status = 'ERROR';

END//

DELIMITER ;

-- Usage example
CALL calculate_customer_rewards(12345, CURRENT_DATE, @points, @tier, @status);
SELECT @points as points_earned, @tier as new_tier, @status as processing_result;
```

#### Hybrid Approach - Best of Both Worlds

```sql
-- Stored function combining declarative and procedural approaches
CREATE FUNCTION get_customer_lifetime_metrics(p_customer_id INT)
RETURNS JSON
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE result JSON DEFAULT JSON_OBJECT();
    DECLARE v_customer_data JSON;
    DECLARE v_order_stats JSON;
    DECLARE v_product_preferences JSON;

    -- Declarative query for customer data
    SELECT JSON_OBJECT(
        'customer_id', customer_id,
        'name', CONCAT(first_name, ' ', last_name),
        'email', email,
        'registration_date', registration_date,
        'status', status,
        'current_loyalty_points', loyalty_points
    ) INTO v_customer_data
    FROM customers
    WHERE customer_id = p_customer_id;

    -- Declarative query for order statistics
    SELECT JSON_OBJECT(
        'total_orders', COUNT(*),
        'total_spent', COALESCE(SUM(total_amount), 0),
        'avg_order_value', COALESCE(AVG(total_amount), 0),
        'first_order_date', MIN(order_date),
        'last_order_date', MAX(order_date),
        'favorite_payment_method', (
            SELECT payment_method
            FROM orders
            WHERE customer_id = p_customer_id
            GROUP BY payment_method
            ORDER BY COUNT(*) DESC
            LIMIT 1
        )
    ) INTO v_order_stats
    FROM orders
    WHERE customer_id = p_customer_id AND status = 'delivered';

    -- Combine results using procedural logic
    SET result = JSON_MERGE_PRESERVE(
        JSON_OBJECT('customer_info', v_customer_data),
        JSON_OBJECT('order_statistics', v_order_stats),
        JSON_OBJECT('analysis_date', NOW())
    );

    RETURN result;
END;

-- Usage
SELECT get_customer_lifetime_metrics(12345) as customer_analysis;
```

---

## 3. SECTION B - ARCHITECTURE INTEGRATION

### B.1 Advanced SDL Implementation

**📍 Location: SECTION B - B.1 Detailed Architecture Analysis**

#### PostgreSQL Storage Configuration

```sql
-- PostgreSQL Storage Configuration
CREATE TABLESPACE fast_storage LOCATION '/mnt/ssd/postgres_data';
CREATE TABLESPACE archive_storage LOCATION '/mnt/hdd/postgres_archive';

-- Table with specific storage parameters
CREATE TABLE high_frequency_transactions (
    transaction_id BIGSERIAL PRIMARY KEY,
    account_id BIGINT NOT NULL,
    transaction_date TIMESTAMP DEFAULT NOW(),
    amount DECIMAL(15,2),
    transaction_type VARCHAR(20),
    description TEXT
) TABLESPACE fast_storage
WITH (
    fillfactor = 90,          -- Leave 10% free space for updates
    parallel_workers = 4,     -- Enable parallel operations
    autovacuum_enabled = true,
    autovacuum_vacuum_scale_factor = 0.1
);

-- Partition tables for better performance (SDL aspect)
CREATE TABLE transaction_archive (
    transaction_id BIGINT,
    account_id BIGINT,
    transaction_date TIMESTAMP,
    amount DECIMAL(15,2),
    transaction_type VARCHAR(20),
    description TEXT
) PARTITION BY RANGE (transaction_date);

-- Create monthly partitions
CREATE TABLE transaction_archive_2024_01 PARTITION OF transaction_archive
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01')
    TABLESPACE archive_storage;

CREATE TABLE transaction_archive_2024_02 PARTITION OF transaction_archive
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01')
    TABLESPACE archive_storage;

-- Advanced indexing strategies (SDL)
CREATE INDEX CONCURRENTLY idx_transaction_account_date
ON high_frequency_transactions (account_id, transaction_date DESC)
INCLUDE (amount, transaction_type)
WITH (fillfactor = 90);

-- Partial indexes for specific query patterns
CREATE INDEX idx_large_transactions
ON high_frequency_transactions (transaction_date, amount)
WHERE amount > 10000;

-- Expression indexes for computed values
CREATE INDEX idx_transaction_month
ON high_frequency_transactions (date_trunc('month', transaction_date));
```

#### Bank Teller View Interface

```sql
-- Bank teller view (operational data access)
CREATE VIEW teller_transaction_interface AS
SELECT
    c.customer_number,
    c.first_name || ' ' || c.last_name as customer_name,
    c.phone,
    a.account_number,
    a.account_name,
    at.type_name as account_type,
    a.current_balance,
    a.available_balance,
    a.overdraft_limit,

    -- Transaction limits and controls
    dtl.transactions_today,
    dtl.amount_today,
    at.transaction_limit_daily,
    (at.transaction_limit_daily - COALESCE(dtl.amount_today, 0)) as remaining_daily_limit,

    -- Account restrictions
    CASE
        WHEN a.status = 'frozen' THEN 'Account frozen - manager approval required'
        WHEN c.risk_profile = 'high' THEN 'High risk customer - verify identity'
        WHEN (at.transaction_limit_daily - COALESCE(dtl.amount_today, 0)) < 100 THEN 'Near daily limit'
        ELSE 'Normal operations'
    END as transaction_notes,

    -- Recent transaction history
    (SELECT json_agg(json_build_object(
        'date', t.transaction_date,
        'type', t.transaction_type,
        'amount', t.amount,
        'description', t.description
    ) ORDER BY t.transaction_date DESC)
     FROM transactions t
     WHERE t.account_id = a.account_id
       AND t.transaction_date >= CURRENT_DATE - INTERVAL '7 days'
     LIMIT 10
    ) as recent_transactions

FROM customers c
INNER JOIN accounts a ON c.customer_id = a.customer_id
INNER JOIN account_types at ON a.account_type_id = at.type_id
LEFT JOIN daily_transaction_limits dtl ON a.account_id = dtl.account_id
    AND dtl.limit_date = CURRENT_DATE
WHERE c.status != 'closed'
  AND a.status != 'closed';

-- Management reporting view (analytical perspective)
CREATE VIEW management_portfolio_analysis AS
SELECT
    at.type_name as account_type,
    COUNT(DISTINCT a.account_id) as total_accounts,
    COUNT(DISTINCT c.customer_id) as unique_customers,
    SUM(a.current_balance) as total_balance,
    AVG(a.current_balance) as average_balance,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY a.current_balance) as median_balance,

    -- Risk analysis
    COUNT(CASE WHEN c.risk_profile = 'high' THEN 1 END) as high_risk_customers,
    SUM(CASE WHEN a.current_balance < 0 THEN a.current_balance ELSE 0 END) as total_overdraft,

    -- Activity analysis
    SUM(CASE WHEN a.last_transaction_date >= CURRENT_DATE - INTERVAL '30 days'
             THEN 1 ELSE 0 END) as active_accounts_30_days,

    -- Performance metrics
    SUM(a.interest_accrued) as total_interest_accrued,
    SUM(COALESCE(at.maintenance_fee, 0)) as potential_fee_revenue,

    -- Growth trends
    COUNT(CASE WHEN a.opened_date >= CURRENT_DATE - INTERVAL '1 year'
               THEN 1 END) as new_accounts_this_year,
    COUNT(CASE WHEN a.closed_date >= CURRENT_DATE - INTERVAL '1 year'
               THEN 1 END) as closed_accounts_this_year

FROM account_types at
LEFT JOIN accounts a ON at.type_id = a.account_type_id
LEFT JOIN customers c ON a.customer_id = c.customer_id
WHERE at.is_active = true
GROUP BY at.type_id, at.type_name
ORDER BY total_balance DESC;
```

### B.2 Data Independence Examples

**📍 Location: SECTION B - B.3 Data Independence Demonstration**

#### Physical Data Independence

```sql
-- Initial table structure
CREATE TABLE customer_orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    total_amount DECIMAL(10,2)
);

-- Application code using the table
SELECT order_id, customer_id, order_date, total_amount
FROM customer_orders
WHERE customer_id = 12345;

-- Physical storage changes (SDL modifications) - Application code unchanged
-- 1. Add partitioning
CREATE TABLE customer_orders_new (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    total_amount DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

-- 2. Change indexing strategy
DROP INDEX IF EXISTS idx_customer_orders_customer_id;
CREATE INDEX idx_customer_orders_customer_date ON customer_orders (customer_id, order_date);

-- 3. Move to different tablespace
ALTER TABLE customer_orders SET TABLESPACE fast_storage;

-- Application queries remain identical - Physical Data Independence achieved
```

#### Logical Data Independence

```sql
-- Original conceptual schema
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100),
    full_name VARCHAR(200)
);

-- User view remains stable
CREATE VIEW user_profile AS
SELECT user_id, username, email, full_name
FROM users;

-- Schema evolution - split name fields (DDL changes)
ALTER TABLE users
ADD COLUMN first_name VARCHAR(100),
ADD COLUMN last_name VARCHAR(100);

UPDATE users
SET first_name = SPLIT_PART(full_name, ' ', 1),
    last_name = SPLIT_PART(full_name, ' ', 2);

ALTER TABLE users DROP COLUMN full_name;

-- Update view to maintain compatibility (Logical Data Independence)
CREATE OR REPLACE VIEW user_profile AS
SELECT
    user_id,
    username,
    email,
    CONCAT(first_name, ' ', last_name) as full_name
FROM users;

-- Applications using the view continue to work unchanged
```

---

## 4. SECTION C - APPLICATIONS AND CASE STUDIES

### C.1 E-commerce Platform Case Study

**📍 Location: SECTION C - C.1 Enterprise E-commerce Platform Case Study**

#### Complete E-commerce Schema

```sql
-- Core business entities with comprehensive constraints
CREATE SCHEMA ecommerce_global;
SET search_path TO ecommerce_global;

-- Customer management with international support
CREATE TABLE customers (
    customer_id BIGSERIAL PRIMARY KEY,
    customer_uuid UUID DEFAULT gen_random_uuid() UNIQUE,
    email VARCHAR(320) NOT NULL, -- RFC 5321 compliant
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    date_of_birth DATE,
    registration_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login TIMESTAMP WITH TIME ZONE,
    email_verified BOOLEAN DEFAULT FALSE,
    phone_verified BOOLEAN DEFAULT FALSE,
    status customer_status_enum DEFAULT 'active',
    preferred_language VARCHAR(5) DEFAULT 'en-US',
    timezone VARCHAR(50) DEFAULT 'UTC',
    marketing_consent BOOLEAN DEFAULT FALSE,
    gdpr_consent_date TIMESTAMP WITH TIME ZONE,
    loyalty_tier loyalty_tier_enum DEFAULT 'bronze',
    loyalty_points INTEGER DEFAULT 0,
    lifetime_value DECIMAL(15,2) DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Comprehensive constraints
    CONSTRAINT chk_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT chk_birth_date CHECK (date_of_birth IS NULL OR date_of_birth <= CURRENT_DATE - INTERVAL '13 years'),
    CONSTRAINT chk_loyalty_points CHECK (loyalty_points >= 0),
    CONSTRAINT chk_lifetime_value CHECK (lifetime_value >= 0)
);
```

#### Customer Segmentation Analysis

```sql
-- Customer segmentation analysis with advanced window functions
WITH customer_metrics AS (
    SELECT
        c.customer_id,
        c.first_name || ' ' || c.last_name as customer_name,
        c.registration_date,
        c.loyalty_tier,
        c.lifetime_value,

        -- Order metrics
        COUNT(o.order_id) as total_orders,
        COALESCE(SUM(o.total_amount), 0) as total_spent,
        COALESCE(AVG(o.total_amount), 0) as avg_order_value,
        MAX(o.order_date) as last_order_date,
        MIN(o.order_date) as first_order_date,

        -- Behavioral analysis
        EXTRACT(DAYS FROM NOW() - MAX(o.order_date)) as days_since_last_order,
        COUNT(DISTINCT DATE_TRUNC('month', o.order_date)) as active_months,

        -- Seasonal analysis
        COUNT(CASE WHEN EXTRACT(QUARTER FROM o.order_date) = 4 THEN 1 END) as q4_orders,
        COUNT(CASE WHEN EXTRACT(QUARTER FROM o.order_date) = 1 THEN 1 END) as q1_orders,

        -- Product diversity
        COUNT(DISTINCT oi.product_id) as unique_products_purchased,

        -- Payment behavior
        COUNT(CASE WHEN o.payment_status = 'failed' THEN 1 END) as failed_payments

    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
        AND o.status IN ('completed', 'shipped', 'delivered')
    LEFT JOIN order_items oi ON o.order_id = oi.order_id
    WHERE c.status = 'active'
    GROUP BY c.customer_id, c.first_name, c.last_name, c.registration_date,
             c.loyalty_tier, c.lifetime_value
),
customer_segments AS (
    SELECT
        *,
        -- RFM Analysis (Recency, Frequency, Monetary)
        NTILE(5) OVER (ORDER BY days_since_last_order DESC NULLS LAST) as recency_score,
        NTILE(5) OVER (ORDER BY total_orders) as frequency_score,
        NTILE(5) OVER (ORDER BY total_spent) as monetary_score,

        -- Customer lifecycle stage
        CASE
            WHEN first_order_date IS NULL THEN 'Prospect'
            WHEN total_orders = 1 AND days_since_last_order <= 90 THEN 'New Customer'
            WHEN total_orders >= 2 AND days_since_last_order <= 90 THEN 'Regular Customer'
            WHEN total_orders >= 5 AND total_spent >= 1000 THEN 'VIP Customer'
            WHEN days_since_last_order > 365 THEN 'Inactive'
            WHEN days_since_last_order > 180 THEN 'At Risk'
            ELSE 'Active'
        END as lifecycle_stage,

        -- Predictive churn risk
        CASE
            WHEN days_since_last_order > 365 OR failed_payments >= 3 THEN 'High'
            WHEN days_since_last_order > 180 OR (avg_order_value < 50 AND total_orders < 3) THEN 'Medium'
            ELSE 'Low'
        END as churn_risk

    FROM customer_metrics
)
SELECT
    lifecycle_stage,
    churn_risk,
    COUNT(*) as customer_count,
    ROUND(AVG(total_spent), 2) as avg_lifetime_value,
    ROUND(AVG(avg_order_value), 2) as avg_order_value,
    ROUND(AVG(days_since_last_order), 0) as avg_days_since_last_order,
    SUM(total_spent) as total_segment_value,

    -- Segment distribution
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) as percentage_of_customers

FROM customer_segments
GROUP BY lifecycle_stage, churn_risk
ORDER BY total_segment_value DESC;
```

#### Product Performance Analysis

```sql
-- Product performance analysis with inventory optimization
WITH product_performance AS (
    SELECT
        p.product_id,
        p.product_sku,
        p.product_name,
        c.category_name,
        b.brand_name,
        p.base_price,
        p.stock_quantity,
        p.low_stock_threshold,

        -- Sales metrics (last 90 days)
        COUNT(oi.order_item_id) as total_orders,
        SUM(oi.quantity) as units_sold,
        SUM(oi.quantity * oi.unit_price) as gross_revenue,
        SUM(oi.quantity * (oi.unit_price - p.cost_price)) as gross_profit,
        AVG(oi.unit_price) as avg_selling_price,

        -- Performance calculations
        CASE
            WHEN SUM(oi.quantity) > 0
            THEN p.stock_quantity / (SUM(oi.quantity) / 90.0)
            ELSE NULL
        END as days_of_inventory,

        -- Velocity classification
        CASE
            WHEN SUM(oi.quantity) >= 100 THEN 'Fast Moving'
            WHEN SUM(oi.quantity) >= 20 THEN 'Regular Moving'
            WHEN SUM(oi.quantity) >= 1 THEN 'Slow Moving'
            ELSE 'Dead Stock'
        END as velocity_category,

        -- Stock status
        CASE
            WHEN p.stock_quantity = 0 THEN 'Out of Stock'
            WHEN p.stock_quantity <= p.low_stock_threshold THEN 'Low Stock'
            WHEN p.stock_quantity > p.low_stock_threshold * 5 THEN 'Overstock'
            ELSE 'Normal'
        END as stock_status

    FROM products p
    INNER JOIN categories c ON p.category_id = c.category_id
    LEFT JOIN brands b ON p.brand_id = b.brand_id
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    LEFT JOIN orders o ON oi.order_id = o.order_id
        AND o.order_date >= CURRENT_DATE - INTERVAL '90 days'
        AND o.status IN ('completed', 'shipped', 'delivered')
    WHERE p.status = 'active'
    GROUP BY p.product_id, p.product_sku, p.product_name, c.category_name,
             b.brand_name, p.base_price, p.stock_quantity, p.low_stock_threshold
)
SELECT
    velocity_category,
    stock_status,
    COUNT(*) as product_count,
    SUM(gross_revenue) as total_revenue,
    SUM(gross_profit) as total_profit,
    ROUND(AVG(days_of_inventory), 1) as avg_days_inventory,
    SUM(stock_quantity * base_price) as inventory_value,

    -- Recommendations
    SUM(CASE WHEN stock_status = 'Out of Stock' AND velocity_category = 'Fast Moving'
             THEN 1 ELSE 0 END) as urgent_restock_needed,
    SUM(CASE WHEN stock_status = 'Overstock' AND velocity_category = 'Slow Moving'
             THEN 1 ELSE 0 END) as clearance_candidates

FROM product_performance
GROUP BY velocity_category, stock_status
ORDER BY total_revenue DESC;
```

#### Order Processing Workflow

```sql
-- Comprehensive order processing stored procedure
CREATE OR REPLACE FUNCTION process_customer_order(
    p_customer_id BIGINT,
    p_cart_items JSON,
    p_shipping_address JSON,
    p_billing_address JSON,
    p_payment_method VARCHAR(50),
    p_coupon_code VARCHAR(50) DEFAULT NULL
) RETURNS JSON AS $$
DECLARE
    v_order_id BIGINT;
    v_order_number VARCHAR(50);
    v_subtotal DECIMAL(15,2) := 0;
    v_tax_amount DECIMAL(15,2) := 0;
    v_shipping_amount DECIMAL(15,2) := 0;
    v_discount_amount DECIMAL(15,2) := 0;
    v_total_amount DECIMAL(15,2);
    v_customer_tier loyalty_tier_enum;
    v_stock_issues JSON := '[]';
    v_processing_errors JSON := '[]';
    v_result JSON;

    cart_item JSON;
    v_product_id BIGINT;
    v_quantity INTEGER;
    v_unit_price DECIMAL(10,2);
    v_available_stock INTEGER;
    v_item_total DECIMAL(10,2);

BEGIN
    -- Input validation
    IF p_customer_id IS NULL OR p_cart_items IS NULL THEN
        RETURN json_build_object(
            'success', false,
            'error', 'Invalid input parameters',
            'order_id', null
        );
    END IF;

    -- Get customer information
    SELECT loyalty_tier INTO v_customer_tier
    FROM customers
    WHERE customer_id = p_customer_id AND status = 'active';

    IF v_customer_tier IS NULL THEN
        RETURN json_build_object(
            'success', false,
            'error', 'Customer not found or inactive',
            'order_id', null
        );
    END IF;

    -- Start transaction
    BEGIN
        -- Generate order number
        SELECT 'ORD-' || TO_CHAR(NOW(), 'YYYYMMDD') || '-' ||
               LPAD(NEXTVAL('order_number_seq')::TEXT, 6, '0') INTO v_order_number;

        -- Validate cart items and check stock
        FOR cart_item IN SELECT * FROM json_array_elements(p_cart_items)
        LOOP
            v_product_id := (cart_item->>'product_id')::BIGINT;
            v_quantity := (cart_item->>'quantity')::INTEGER;

            -- Check product availability and stock
            SELECT base_price, stock_quantity
            INTO v_unit_price, v_available_stock
            FROM products
            WHERE product_id = v_product_id AND status = 'active';

            IF v_unit_price IS NULL THEN
                v_processing_errors := v_processing_errors ||
                    json_build_object('error', 'Product not found', 'product_id', v_product_id);
                CONTINUE;
            END IF;

            IF v_available_stock < v_quantity THEN
                v_stock_issues := v_stock_issues ||
                    json_build_object(
                        'product_id', v_product_id,
                        'requested', v_quantity,
                        'available', v_available_stock
                    );
                CONTINUE;
            END IF;

            -- Calculate item total
            v_item_total := v_unit_price * v_quantity;
            v_subtotal := v_subtotal + v_item_total;
        END LOOP;

        -- Check for errors
        IF json_array_length(v_processing_errors) > 0 OR json_array_length(v_stock_issues) > 0 THEN
            RETURN json_build_object(
                'success', false,
                'error', 'Order validation failed',
                'stock_issues', v_stock_issues,
                'processing_errors', v_processing_errors
            );
        END IF;

        -- Apply customer tier discounts
        CASE v_customer_tier
            WHEN 'platinum' THEN v_discount_amount := v_subtotal * 0.15;
            WHEN 'gold' THEN v_discount_amount := v_subtotal * 0.10;
            WHEN 'silver' THEN v_discount_amount := v_subtotal * 0.05;
            ELSE v_discount_amount := 0;
        END CASE;

        -- Calculate tax (8.5% rate example)
        v_tax_amount := (v_subtotal - v_discount_amount) * 0.085;

        -- Calculate shipping
        IF (v_subtotal - v_discount_amount) >= 100 THEN
            v_shipping_amount := 0;
        ELSE
            v_shipping_amount := 15.99;
        END IF;

        -- Calculate total
        v_total_amount := v_subtotal - v_discount_amount + v_tax_amount + v_shipping_amount;

        -- Create order record
        INSERT INTO orders (
            order_number, customer_id, status, subtotal, tax_amount,
            shipping_amount, discount_amount, total_amount, payment_method,
            billing_first_name, billing_last_name, billing_address_1,
            shipping_first_name, shipping_last_name, shipping_address_1
        ) VALUES (
            v_order_number, p_customer_id, 'pending', v_subtotal, v_tax_amount,
            v_shipping_amount, v_discount_amount, v_total_amount, p_payment_method,
            p_billing_address->>'first_name', p_billing_address->>'last_name',
            p_billing_address->>'address_1',
            p_shipping_address->>'first_name', p_shipping_address->>'last_name',
            p_shipping_address->>'address_1'
        ) RETURNING order_id INTO v_order_id;

        -- Create order items and update inventory
        FOR cart_item IN SELECT * FROM json_array_elements(p_cart_items)
        LOOP
            v_product_id := (cart_item->>'product_id')::BIGINT;
            v_quantity := (cart_item->>'quantity')::INTEGER;

            SELECT base_price INTO v_unit_price
            FROM products WHERE product_id = v_product_id;

            -- Insert order item
            INSERT INTO order_items (order_id, product_id, quantity, unit_price)
            VALUES (v_order_id, v_product_id, v_quantity, v_unit_price);

            -- Update inventory
            UPDATE products
            SET stock_quantity = stock_quantity - v_quantity,
                updated_at = NOW()
            WHERE product_id = v_product_id;
        END LOOP;

        -- Update customer lifetime value
        UPDATE customers
        SET lifetime_value = lifetime_value + v_total_amount,
            updated_at = NOW()
        WHERE customer_id = p_customer_id;

        -- Success response
        v_result := json_build_object(
            'success', true,
            'order_id', v_order_id,
            'order_number', v_order_number,
            'total_amount', v_total_amount,
            'message', 'Order processed successfully'
        );

    EXCEPTION
        WHEN OTHERS THEN
            v_result := json_build_object(
                'success', false,
                'error', 'Database error during order processing',
                'details', SQLERRM
            );
    END;

    RETURN v_result;
END;
$$ LANGUAGE plpgsql;

-- Usage example
SELECT process_customer_order(
    12345,
    '[{"product_id": 101, "quantity": 2}, {"product_id": 102, "quantity": 1}]',
    '{"first_name": "John", "last_name": "Doe", "address_1": "123 Main St"}',
    '{"first_name": "John", "last_name": "Doe", "address_1": "123 Main St"}',
    'credit_card',
    'SAVE10'
) as order_result;
```

### C.2 Banking System Case Study

**📍 Location: SECTION C - C.2 Financial Services Banking System Case Study**

#### Banking Schema with Business Rules

```sql
-- Complete conceptual schema for banking system
CREATE SCHEMA banking_core;
SET search_path TO banking_core;

-- Core entity definitions with business rules
CREATE TABLE account_types (
    type_id SERIAL PRIMARY KEY,
    type_name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    min_balance DECIMAL(15,2) DEFAULT 0,
    interest_rate DECIMAL(5,4) DEFAULT 0,
    transaction_limit_daily DECIMAL(15,2),
    maintenance_fee DECIMAL(10,2) DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE customers (
    customer_id BIGSERIAL PRIMARY KEY,
    customer_number VARCHAR(20) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    date_of_birth DATE NOT NULL,
    ssn_hash VARCHAR(64) UNIQUE,  -- Hashed for privacy
    address_line1 VARCHAR(255),
    address_line2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    postal_code VARCHAR(20),
    country VARCHAR(50) DEFAULT 'USA',
    customer_since DATE DEFAULT CURRENT_DATE,
    status VARCHAR(20) DEFAULT 'active',
    risk_profile VARCHAR(20) DEFAULT 'medium',
    last_updated TIMESTAMP DEFAULT NOW(),

    -- Business rule constraints
    CONSTRAINT chk_customer_status CHECK (status IN ('active', 'inactive', 'suspended', 'closed')),
    CONSTRAINT chk_risk_profile CHECK (risk_profile IN ('low', 'medium', 'high')),
    CONSTRAINT chk_birth_date CHECK (date_of_birth <= CURRENT_DATE - INTERVAL '18 years'),
    CONSTRAINT chk_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

CREATE TABLE accounts (
    account_id BIGSERIAL PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    customer_id BIGINT NOT NULL,
    account_type_id INTEGER NOT NULL,
    account_name VARCHAR(100) NOT NULL,
    current_balance DECIMAL(15,2) DEFAULT 0,
    available_balance DECIMAL(15,2) DEFAULT 0,
    opened_date DATE DEFAULT CURRENT_DATE,
    closed_date DATE,
    status VARCHAR(20) DEFAULT 'active',
    overdraft_limit DECIMAL(15,2) DEFAULT 0,
    last_transaction_date TIMESTAMP,
    interest_accrued DECIMAL(15,2) DEFAULT 0,

    -- Foreign key relationships
    CONSTRAINT fk_account_customer FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id) ON DELETE RESTRICT,
    CONSTRAINT fk_account_type FOREIGN KEY (account_type_id)
        REFERENCES account_types(type_id) ON DELETE RESTRICT,

    -- Business rule constraints
    CONSTRAINT chk_account_status CHECK (status IN ('active', 'inactive', 'frozen', 'closed')),
    CONSTRAINT chk_balance_positive CHECK (current_balance >= -overdraft_limit),
    CONSTRAINT chk_closed_date CHECK (closed_date IS NULL OR closed_date >= opened_date)
);

-- Triggers for business logic enforcement
CREATE OR REPLACE FUNCTION update_account_balance()
RETURNS TRIGGER AS $$
BEGIN
    -- Update current and available balances
    IF TG_OP = 'INSERT' THEN
        UPDATE accounts
        SET current_balance = current_balance + NEW.amount,
            available_balance = CASE
                WHEN NEW.amount > 0 THEN available_balance + NEW.amount
                ELSE GREATEST(available_balance + NEW.amount, -overdraft_limit)
            END,
            last_transaction_date = NEW.transaction_date
        WHERE account_id = NEW.account_id;

        -- Update daily limits
        INSERT INTO daily_transaction_limits (account_id, limit_date, transactions_today, amount_today)
        VALUES (NEW.account_id, NEW.transaction_date::DATE, 1, ABS(NEW.amount))
        ON CONFLICT (account_id, limit_date) DO UPDATE
        SET transactions_today = daily_transaction_limits.transactions_today + 1,
            amount_today = daily_transaction_limits.amount_today + ABS(NEW.amount);

        RETURN NEW;
    END IF;

    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_update_balance_after_transaction
    AFTER INSERT ON transactions
    FOR EACH ROW EXECUTE FUNCTION update_account_balance();
```

#### HIPAA-Compliant Banking Views

```sql
-- Role-based access control through sophisticated views
CREATE SCHEMA banking_secure;
SET search_path TO banking_secure;

-- Customer service representative view - limited PII access
CREATE VIEW csr_customer_interface AS
SELECT
    c.customer_id,
    CASE
        WHEN CURRENT_USER IN (SELECT user_name FROM authorized_csr_users)
        THEN c.first_name
        ELSE '***'
    END as first_name,
    CASE
        WHEN CURRENT_USER IN (SELECT user_name FROM authorized_csr_users)
        THEN c.last_name
        ELSE '***'
    END as last_name,

    -- Masked sensitive data
    LEFT(c.email, 3) || '***@' || SPLIT_PART(c.email, '@', 2) as masked_email,
    'XXX-XXX-' || RIGHT(c.phone, 4) as masked_phone,

    -- Account summary (non-sensitive)
    COUNT(a.account_id) as total_accounts,
    SUM(CASE WHEN a.status = 'active' THEN 1 ELSE 0 END) as active_accounts,
    c.customer_since,
    c.status as customer_status,

    -- Risk indicators
    CASE
        WHEN c.risk_score > 80 THEN 'HIGH'
        WHEN c.risk_score > 50 THEN 'MEDIUM'
        ELSE 'LOW'
    END as risk_level,

    -- Recent activity flags
    (SELECT COUNT(*) FROM transactions t
     INNER JOIN accounts acc ON t.account_id = acc.account_id
     WHERE acc.customer_id = c.customer_id
       AND t.transaction_date >= CURRENT_DATE - INTERVAL '24 hours'
       AND t.suspicious_activity_flag = true
    ) as recent_suspicious_transactions

FROM customers c
LEFT JOIN accounts a ON c.customer_id = a.customer_id
WHERE c.status IN ('active', 'restricted')
GROUP BY c.customer_id, c.first_name, c.last_name, c.email, c.phone,
         c.customer_since, c.status, c.risk_score
WITH CHECK OPTION;

-- Compliance officer view - full audit trail access
CREATE VIEW compliance_audit_trail AS
SELECT
    t.transaction_id,
    t.account_id,
    a.account_number,
    c.customer_id,
    -- Full customer details for compliance
    c.first_name,
    c.last_name,
    c.ssn_hash,

    -- Transaction details
    t.transaction_date,
    t.transaction_type,
    t.amount,
    t.description,
    t.source_account,
    t.destination_account,

    -- Risk and compliance flags
    t.suspicious_activity_flag,
    t.large_cash_transaction_flag,
    t.cross_border_flag,
    t.aml_risk_score,

    -- Regulatory reporting fields
    CASE
        WHEN t.amount >= 10000 THEN 'CTR_REQUIRED'  -- Currency Transaction Report
        WHEN t.suspicious_activity_flag THEN 'SAR_REQUIRED'  -- Suspicious Activity Report
        WHEN t.cross_border_flag AND t.amount >= 3000 THEN 'FBAR_RELEVANT'
        ELSE 'STANDARD'
    END as regulatory_requirement,

    -- Geographic risk assessment
    CASE
        WHEN t.originating_country IN (SELECT country_code FROM high_risk_countries)
        THEN 'HIGH_RISK_GEOGRAPHY'
        ELSE 'STANDARD_GEOGRAPHY'
    END as geographic_risk,

    -- Processing metadata
    t.processed_by_user,
    t.approval_status,
    t.approval_date,
    t.created_at,
    t.updated_at

FROM transactions t
INNER JOIN accounts a ON t.account_id = a.account_id
INNER JOIN customers c ON a.customer_id = c.customer_id
WHERE
    -- Compliance officer access control
    CURRENT_USER IN (SELECT user_name FROM compliance_officers)
    AND (
        t.amount >= 3000  -- Focus on significant transactions
        OR t.suspicious_activity_flag = true
        OR t.cross_border_flag = true
    )
WITH CHECK OPTION;
```

#### Real-time Fraud Detection Function

```sql
-- Real-time transaction fraud scoring
CREATE OR REPLACE FUNCTION evaluate_transaction_risk(
    p_account_id BIGINT,
    p_transaction_type VARCHAR(50),
    p_amount DECIMAL(15,2),
    p_merchant_category VARCHAR(100),
    p_location_data JSON
) RETURNS JSON AS $$
DECLARE
    v_risk_score INTEGER := 0;
    v_risk_factors JSON := '[]';
    v_customer_profile JSON;
    v_account_history JSON;
    v_velocity_check JSON;
    v_geographic_risk INTEGER := 0;
    v_behavioral_risk INTEGER := 0;
    v_amount_risk INTEGER := 0;
    v_final_decision VARCHAR(20);

BEGIN
    -- Get customer baseline profile
    SELECT json_build_object(
        'customer_id', c.customer_id,
        'risk_profile', c.risk_profile,
        'account_age_days', EXTRACT(DAYS FROM NOW() - a.opened_date),
        'typical_balance', AVG(dh.end_of_day_balance),
        'typical_transaction_amount', AVG(t.amount),
        'typical_locations', array_agg(DISTINCT t.location_city)
    ) INTO v_customer_profile
    FROM accounts a
    INNER JOIN customers c ON a.customer_id = c.customer_id
    LEFT JOIN daily_balance_history dh ON a.account_id = dh.account_id
        AND dh.balance_date >= CURRENT_DATE - INTERVAL '90 days'
    LEFT JOIN transactions t ON a.account_id = t.account_id
        AND t.transaction_date >= CURRENT_DATE - INTERVAL '90 days'
        AND t.status = 'completed'
    WHERE a.account_id = p_account_id
    GROUP BY c.customer_id, c.risk_profile, a.opened_date;

    -- Velocity-based risk assessment
    WITH velocity_analysis AS (
        SELECT
            COUNT(*) as transactions_today,
            SUM(amount) as amount_today,
            COUNT(CASE WHEN amount > 1000 THEN 1 END) as large_transactions_today,
            COUNT(DISTINCT merchant_category) as unique_merchants_today,
            COUNT(DISTINCT location_city) as unique_locations_today
        FROM transactions
        WHERE account_id = p_account_id
          AND DATE(transaction_date) = CURRENT_DATE
          AND status IN ('completed', 'pending')
    )
    SELECT json_build_object(
        'transactions_today', transactions_today,
        'amount_today', amount_today,
        'large_transactions_today', large_transactions_today,
        'unique_merchants_today', unique_merchants_today,
        'unique_locations_today', unique_locations_today
    ) INTO v_velocity_check
    FROM velocity_analysis;

    -- Geographic risk assessment
    IF p_location_data->>'country' != 'USA' THEN
        v_geographic_risk := v_geographic_risk + 25;
        v_risk_factors := v_risk_factors || json_build_object(
            'factor', 'international_transaction',
            'score', 25,
            'details', 'Transaction from ' || (p_location_data->>'country')
        );
    END IF;

    -- Check for high-risk geographic locations
    IF EXISTS (
        SELECT 1 FROM high_risk_countries
        WHERE country_code = p_location_data->>'country'
    ) THEN
        v_geographic_risk := v_geographic_risk + 40;
        v_risk_factors := v_risk_factors || json_build_object(
            'factor', 'high_risk_country',
            'score', 40,
            'details', 'Transaction from high-risk jurisdiction'
        );
    END IF;

    -- Behavioral risk assessment
    IF p_amount > (v_customer_profile->>'typical_transaction_amount')::DECIMAL * 5 THEN
        v_behavioral_risk := v_behavioral_risk + 30;
        v_risk_factors := v_risk_factors || json_build_object(
            'factor', 'unusual_amount',
            'score', 30,
            'details', 'Transaction amount significantly higher than typical'
        );
    END IF;

    -- Velocity risk
    IF (v_velocity_check->>'transactions_today')::INTEGER > 10 THEN
        v_behavioral_risk := v_behavioral_risk + 20;
        v_risk_factors := v_risk_factors || json_build_object(
            'factor', 'high_velocity',
            'score', 20,
            'details', 'Unusually high transaction frequency'
        );
    END IF;

    -- Time-based risk (late night transactions)
    IF EXTRACT(HOUR FROM NOW()) BETWEEN 2 AND 5 THEN
        v_behavioral_risk := v_behavioral_risk + 15;
        v_risk_factors := v_risk_factors || json_build_object(
            'factor', 'unusual_time',
            'score', 15,
            'details', 'Transaction during unusual hours'
        );
    END IF;

    -- Amount-based risk
    CASE
        WHEN p_amount >= 50000 THEN v_amount_risk := 35;
        WHEN p_amount >= 10000 THEN v_amount_risk := 25;
        WHEN p_amount >= 5000 THEN v_amount_risk := 15;
        WHEN p_amount >= 1000 THEN v_amount_risk := 5;
        ELSE v_amount_risk := 0;
    END CASE;

    -- Calculate total risk score
    v_risk_score := v_geographic_risk + v_behavioral_risk + v_amount_risk;

    -- Add customer risk profile adjustment
    CASE v_customer_profile->>'risk_profile'
        WHEN 'high' THEN v_risk_score := v_risk_score + 20;
        WHEN 'medium' THEN v_risk_score := v_risk_score + 10;
        ELSE v_risk_score := v_risk_score + 0;
    END CASE;

    -- Determine final decision
    CASE
        WHEN v_risk_score >= 80 THEN v_final_decision := 'BLOCK';
        WHEN v_risk_score >= 60 THEN v_final_decision := 'REVIEW';
        WHEN v_risk_score >= 40 THEN v_final_decision := 'MONITOR';
        ELSE v_final_decision := 'APPROVE';
    END CASE;

    -- Log risk assessment
    INSERT INTO transaction_risk_assessments (
        account_id, risk_score, risk_factors, decision, assessment_date
    ) VALUES (
        p_account_id, v_risk_score, v_risk_factors, v_final_decision, NOW()
    );

    RETURN json_build_object(
        'risk_score', v_risk_score,
        'decision', v_final_decision,
        'risk_factors', v_risk_factors,
        'customer_profile', v_customer_profile,
        'velocity_check', v_velocity_check,
        'assessment_timestamp', NOW()
    );

END;
$$ LANGUAGE plpgsql;
```

### C.3 Healthcare Information System Case Study

**📍 Location: SECTION C - C.3 Healthcare Information Management System Case Study**

#### HIPAA-Compliant Patient Data Access

```sql
-- Patient data access with comprehensive privacy controls
CREATE SCHEMA healthcare_secure;
SET search_path TO healthcare_secure;

-- Physician view - treatment-focused access
CREATE VIEW physician_patient_care AS
SELECT
    p.patient_id,
    p.medical_record_number,

    -- Patient demographics (limited based on treatment relationship)
    CASE
        WHEN EXISTS (
            SELECT 1 FROM physician_patient_assignments ppa
            WHERE ppa.patient_id = p.patient_id
              AND ppa.physician_id = get_current_physician_id()
              AND ppa.status = 'active'
        ) THEN p.first_name
        ELSE 'RESTRICTED'
    END as first_name,

    CASE
        WHEN EXISTS (
            SELECT 1 FROM physician_patient_assignments ppa
            WHERE ppa.patient_id = p.patient_id
              AND ppa.physician_id = get_current_physician_id()
              AND ppa.status = 'active'
        ) THEN p.last_name
        ELSE 'RESTRICTED'
    END as last_name,

    p.date_of_birth,
    p.gender,
    p.blood_type,

    -- Clinical information
    p.allergies,
    p.chronic_conditions,
    p.current_medications,
    p.emergency_contact,

    -- Recent encounters
    (SELECT json_agg(json_build_object(
        'encounter_id', e.encounter_id,
        'encounter_date', e.encounter_date,
        'encounter_type', e.encounter_type,
        'chief_complaint', e.chief_complaint,
        'diagnosis_codes', e.diagnosis_codes,
        'treatment_notes', CASE
            WHEN e.attending_physician_id = get_current_physician_id()
            THEN e.treatment_notes
            ELSE 'Access restricted to attending physician'
        END
    ) ORDER BY e.encounter_date DESC)
     FROM patient_encounters e
     WHERE e.patient_id = p.patient_id
       AND e.encounter_date >= CURRENT_DATE - INTERVAL '1 year'
     LIMIT 20
    ) as recent_encounters,

    -- Lab results (time-restricted)
    (SELECT json_agg(json_build_object(
        'test_date', lr.test_date,
        'test_type', lr.test_type,
        'result_value', lr.result_value,
        'reference_range', lr.reference_range,
        'abnormal_flag', lr.abnormal_flag
    ) ORDER BY lr.test_date DESC)
     FROM lab_results lr
     WHERE lr.patient_id = p.patient_id
       AND lr.test_date >= CURRENT_DATE - INTERVAL '2 years'
       AND lr.result_status = 'final'
    ) as lab_results,

    -- Medication history
    (SELECT json_agg(json_build_object(
        'medication_name', m.medication_name,
        'dosage', m.dosage,
        'frequency', m.frequency,
        'start_date', m.start_date,
        'end_date', m.end_date,
        'prescribing_physician', ph.first_name || ' ' || ph.last_name
    ) ORDER BY m.start_date DESC)
     FROM patient_medications m
     INNER JOIN physicians ph ON m.prescribing_physician_id = ph.physician_id
     WHERE m.patient_id = p.patient_id
       AND (m.end_date IS NULL OR m.end_date >= CURRENT_DATE - INTERVAL '1 year')
    ) as medication_history

FROM patients p
WHERE p.status = 'active'
  AND EXISTS (
      SELECT 1 FROM physician_patient_assignments ppa
      WHERE ppa.patient_id = p.patient_id
        AND ppa.physician_id = get_current_physician_id()
        AND ppa.status = 'active'
  )
WITH CHECK OPTION;

-- Research data view - de-identified for studies
CREATE VIEW research_patient_cohort AS
SELECT
    -- De-identified patient identifier
    SHA256(CONCAT(patient_id::text, 'research_salt_2024'))::varchar as research_id,

    -- Demographic data (aggregated/de-identified)
    CASE
        WHEN EXTRACT(YEAR FROM age(date_of_birth)) BETWEEN 0 AND 17 THEN '0-17'
        WHEN EXTRACT(YEAR FROM age(date_of_birth)) BETWEEN 18 AND 34 THEN '18-34'
        WHEN EXTRACT(YEAR FROM age(date_of_birth)) BETWEEN 35 AND 54 THEN '35-54'
        WHEN EXTRACT(YEAR FROM age(date_of_birth)) BETWEEN 55 AND 74 THEN '55-74'
        ELSE '75+'
    END as age_group,

    gender,
    LEFT(postal_code, 3) as postal_area,  -- Geographic region only

    -- Clinical indicators (for research)
    chronic_conditions,
    primary_diagnosis_category,

    -- Aggregated utilization metrics
    (SELECT COUNT(*) FROM patient_encounters
     WHERE patient_id = p.patient_id
       AND encounter_date >= CURRENT_DATE - INTERVAL '1 year'
    ) as encounters_last_year,

    (SELECT COUNT(DISTINCT diagnosis_primary) FROM patient_encounters
     WHERE patient_id = p.patient_id
       AND encounter_date >= CURRENT_DATE - INTERVAL '2 years'
    ) as unique_diagnoses_2_years,

    -- Outcome measures (de-identified)
    CASE
        WHEN EXISTS (
            SELECT 1 FROM patient_encounters
            WHERE patient_id = p.patient_id
              AND encounter_type = 'emergency'
              AND encounter_date >= CURRENT_DATE - INTERVAL '30 days'
        ) THEN 'Y'
        ELSE 'N'
    END as recent_emergency_visit,

    -- Remove all direct identifiers
    NULL as first_name,
    NULL as last_name,
    NULL as ssn,
    NULL as phone,
    NULL as email,
    NULL as address

FROM patients p
WHERE p.consent_for_research = TRUE
  AND p.status = 'active'
  AND CURRENT_USER IN (SELECT username FROM approved_researchers);
```

---

## 5. SECTION D - CHALLENGES AND SOLUTIONS

### D.1 Performance Optimization

**📍 Location: SECTION D - D.1 Current Industry Challenges**

#### Advanced Query Optimization

```sql
-- Using query hints for complex analytical queries
SELECT /*+ USE_HASH(c,o) PARALLEL(4) */
    c.customer_segment,
    COUNT(DISTINCT c.customer_id) as unique_customers,
    SUM(o.total_amount) as total_revenue,
    AVG(o.total_amount) as avg_order_value
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= ADD_MONTHS(SYSDATE, -12)
GROUP BY c.customer_segment
ORDER BY total_revenue DESC;

-- Partition-wise joins for better performance
CREATE TABLE sales_data_2024 (
    sale_id BIGINT,
    customer_id BIGINT,
    product_id BIGINT,
    sale_date DATE,
    amount DECIMAL(12,2)
) PARTITION BY RANGE (sale_date) (
    PARTITION sales_q1_2024 VALUES LESS THAN (TO_DATE('2024-04-01', 'YYYY-MM-DD')),
    PARTITION sales_q2_2024 VALUES LESS THAN (TO_DATE('2024-07-01', 'YYYY-MM-DD')),
    PARTITION sales_q3_2024 VALUES LESS THAN (TO_DATE('2024-10-01', 'YYYY-MM-DD')),
    PARTITION sales_q4_2024 VALUES LESS THAN (TO_DATE('2025-01-01', 'YYYY-MM-DD'))
);

-- Materialized view for complex aggregations
CREATE MATERIALIZED VIEW monthly_sales_summary AS
SELECT
    DATE_TRUNC('month', sale_date) as sales_month,
    product_category,
    SUM(amount) as total_sales,
    COUNT(*) as transaction_count,
    AVG(amount) as avg_transaction_value
FROM sales_data_2024 s
INNER JOIN products p ON s.product_id = p.product_id
GROUP BY DATE_TRUNC('month', sale_date), product_category;

-- Automatic refresh schedule
CREATE OR REPLACE PROCEDURE refresh_sales_summary
AS
BEGIN
    EXECUTE IMMEDIATE 'REFRESH MATERIALIZED VIEW monthly_sales_summary';
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
```

### D.2 Security Implementation

**📍 Location: SECTION D - D.1 Current Industry Challenges - Security Solutions**

#### Row-Level Security and Encryption

```sql
-- Row-level security implementation
CREATE POLICY customer_data_access ON customers
    FOR ALL TO application_users
    USING (
        customer_id IN (
            SELECT customer_id FROM user_customer_access
            WHERE user_id = current_user_id()
              AND access_type IN ('read', 'write')
              AND expiration_date > CURRENT_DATE
        )
    );

-- Encrypted sensitive data storage
CREATE TABLE customer_pii (
    customer_id BIGINT PRIMARY KEY,
    encrypted_ssn BYTEA,  -- AES-256 encrypted
    encrypted_credit_card BYTEA,
    encryption_key_id INTEGER,
    created_at TIMESTAMP DEFAULT NOW(),

    CONSTRAINT fk_encryption_key FOREIGN KEY (encryption_key_id)
        REFERENCES encryption_keys(key_id)
);

-- Audit trail for all data modifications
CREATE OR REPLACE FUNCTION audit_data_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO data_audit_log (
        table_name,
        operation_type,
        record_id,
        old_values,
        new_values,
        user_id,
        session_id,
        ip_address,
        timestamp
    ) VALUES (
        TG_TABLE_NAME,
        TG_OP,
        COALESCE(NEW.id, OLD.id),
        CASE WHEN TG_OP != 'INSERT' THEN row_to_json(OLD) ELSE NULL END,
        CASE WHEN TG_OP != 'DELETE' THEN row_to_json(NEW) ELSE NULL END,
        current_user_id(),
        current_session_id(),
        inet_client_addr(),
        NOW()
    );

    RETURN CASE WHEN TG_OP = 'DELETE' THEN OLD ELSE NEW END;
END;
$$ LANGUAGE plpgsql;

-- Apply audit trigger to sensitive tables
CREATE TRIGGER audit_customers
    AFTER INSERT OR UPDATE OR DELETE ON customers
    FOR EACH ROW EXECUTE FUNCTION audit_data_changes();
```

### D.3 AI/ML Integration with Database Languages

**📍 Location: SECTION D - D.2 Emerging Technology Integration**

#### In-Database Machine Learning

```sql
-- In-database machine learning (PostgreSQL with ML extensions)
-- Customer churn prediction model
CREATE MODEL customer_churn_model
USING logistic_regression
AS SELECT
    customer_age,
    total_orders,
    avg_order_value,
    days_since_last_order,
    customer_service_contacts,
    (CASE WHEN last_order_date < CURRENT_DATE - INTERVAL '6 months'
          THEN 1 ELSE 0 END) as churned
FROM customer_analytics_mv
WHERE registration_date < CURRENT_DATE - INTERVAL '1 year';

-- Apply ML model in real-time queries
SELECT
    customer_id,
    customer_name,
    PREDICT(customer_churn_model,
        customer_age, total_orders, avg_order_value,
        days_since_last_order, customer_service_contacts
    ) as churn_probability
FROM customer_current_metrics
WHERE churn_probability > 0.7
ORDER BY churn_probability DESC;
```

#### Multi-Model Database Support

```sql
-- Multi-model database support example (PostgreSQL)
-- JSON document storage with relational integration
SELECT
    customer_id,
    customer_data->>'name' as customer_name,
    (customer_data->'preferences'->>'categories')::text[] as preferred_categories,

    -- Graph traversal for customer network
    WITH RECURSIVE customer_network AS (
        SELECT customer_id, referrer_id, 1 as level
        FROM customer_referrals
        WHERE customer_id = 12345

        UNION ALL

        SELECT cr.customer_id, cr.referrer_id, cn.level + 1
        FROM customer_referrals cr
        INNER JOIN customer_network cn ON cr.referrer_id = cn.customer_id
        WHERE cn.level < 5
    )
    SELECT COUNT(*) FROM customer_network

FROM customers
WHERE customer_data @> '{"status": "active"}'
  AND customer_data->'location'->>'country' = 'USA';
```

#### Cloud-Native Database Patterns

```sql
-- Multi-region data distribution
CREATE TABLE global_customers (
    customer_id BIGINT PRIMARY KEY,
    region VARCHAR(20) NOT NULL,
    customer_data JSONB
) PARTITION BY LIST (region);

-- Regional partitions for data locality
CREATE TABLE customers_us PARTITION OF global_customers
    FOR VALUES IN ('us-east', 'us-west')
    TABLESPACE us_storage;

CREATE TABLE customers_eu PARTITION OF global_customers
    FOR VALUES IN ('eu-west', 'eu-central')
    TABLESPACE eu_storage;

-- Cross-region replication setup
CREATE PUBLICATION global_customer_updates FOR TABLE global_customers;
-- Subscription setup on replica regions
CREATE SUBSCRIPTION eu_replica
    CONNECTION 'host=us-primary.db port=5432 dbname=production'
    PUBLICATION global_customer_updates;
```

#### Quantum-Safe Security Implementation

```sql
-- Future quantum-safe encryption implementation
CREATE TABLE sensitive_customer_data (
    customer_id BIGINT PRIMARY KEY,
    encrypted_pii BYTEA,
    quantum_safe_hash VARCHAR(512),
    encryption_algorithm VARCHAR(50) DEFAULT 'CRYSTALS-KYBER-1024',
    key_rotation_date TIMESTAMP DEFAULT NOW(),

    -- Post-quantum digital signature
    quantum_signature BYTEA,
    signature_algorithm VARCHAR(50) DEFAULT 'DILITHIUM-3'
);

-- Function for quantum-safe data encryption
CREATE OR REPLACE FUNCTION encrypt_with_pqc(
    p_data TEXT,
    p_algorithm VARCHAR(50) DEFAULT 'CRYSTALS-KYBER-1024'
) RETURNS BYTEA AS $$
BEGIN
    -- Implementation would use post-quantum cryptography library
    -- This is a conceptual example
    RETURN pgp_sym_encrypt(p_data, current_setting('app.quantum_key'), p_algorithm);
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## 6. ADVANCED EXAMPLES AND EDGE CASES

### 6.1 Complex Transaction Management

**📍 Location: SECTION A - A.3 Data Manipulation Language (DML) - Advanced Examples**

#### Multi-Step Transaction with Savepoints

```sql
-- Complex multi-step transaction with savepoints
BEGIN TRANSACTION;

-- Savepoint for customer creation
SAVEPOINT customer_creation;

-- Step 1: Create customer
INSERT INTO customers (first_name, last_name, email, phone)
VALUES ('John', 'Doe', 'john.doe@email.com', '+1-555-0123');

SET @customer_id = LAST_INSERT_ID();

-- Step 2: Create primary account
SAVEPOINT account_creation;

INSERT INTO accounts (customer_id, account_type_id, account_name, initial_deposit)
VALUES (@customer_id, 1, 'Primary Checking', 1000.00);

SET @account_id = LAST_INSERT_ID();

-- Step 3: Setup automatic transfers (could fail)
SAVEPOINT auto_transfer_setup;

BEGIN
    INSERT INTO recurring_transfers (
        from_account_id, to_account_id, amount, frequency, start_date
    ) VALUES (
        @account_id,
        (SELECT account_id FROM system_accounts WHERE account_type = 'savings_auto'),
        100.00,
        'monthly',
        DATE_ADD(CURRENT_DATE, INTERVAL 1 MONTH)
    );
EXCEPTION
    WHEN OTHERS THEN
        -- If auto-transfer fails, rollback just this part
        ROLLBACK TO auto_transfer_setup;

        INSERT INTO customer_notes (customer_id, note_type, note_text, created_at)
        VALUES (@customer_id, 'setup_warning', 'Auto-transfer setup failed - manual setup required', NOW());
END;

-- Step 4: Send welcome email (external system call)
-- If this fails, we don't want to rollback the entire account creation
SAVEPOINT welcome_email;

BEGIN
    CALL send_welcome_email(@customer_id);
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK TO welcome_email;

        INSERT INTO pending_communications (customer_id, communication_type, priority, created_at)
        VALUES (@customer_id, 'welcome_email', 'high', NOW());
END;

-- Final validation before commit
IF (SELECT COUNT(*) FROM customers WHERE customer_id = @customer_id) = 1
   AND (SELECT COUNT(*) FROM accounts WHERE customer_id = @customer_id) >= 1 THEN
    COMMIT;
    SELECT @customer_id as new_customer_id, 'Account created successfully' as status;
ELSE
    ROLLBACK;
    SELECT NULL as new_customer_id, 'Account creation failed' as status;
END IF;
```

#### Batch Processing with Error Handling

```sql
-- Batch processing with comprehensive error handling
DELIMITER //

CREATE PROCEDURE process_monthly_statements(
    IN p_process_date DATE,
    OUT p_success_count INT,
    OUT p_error_count INT,
    OUT p_error_details TEXT
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_customer_id BIGINT;
    DECLARE v_account_id BIGINT;
    DECLARE v_error_msg TEXT DEFAULT '';
    DECLARE v_current_balance DECIMAL(15,2);

    -- Cursor for active accounts
    DECLARE account_cursor CURSOR FOR
        SELECT c.customer_id, a.account_id, a.current_balance
        FROM customers c
        INNER JOIN accounts a ON c.customer_id = a.customer_id
        WHERE c.status = 'active'
          AND a.status = 'active'
          AND a.account_type_id IN (1, 2, 3); -- Checking, Savings, Money Market

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- Initialize counters
    SET p_success_count = 0;
    SET p_error_count = 0;
    SET p_error_details = '';

    -- Create temporary table for batch processing results
    CREATE TEMPORARY TABLE temp_statement_results (
        customer_id BIGINT,
        account_id BIGINT,
        status VARCHAR(50),
        error_message TEXT,
        processing_time TIMESTAMP DEFAULT NOW()
    );

    -- Open cursor and process each account
    OPEN account_cursor;

    read_loop: LOOP
        FETCH account_cursor INTO v_customer_id, v_account_id, v_current_balance;

        IF done THEN
            LEAVE read_loop;
        END IF;

        BEGIN
            -- Nested transaction for each statement
            DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
            BEGIN
                GET DIAGNOSTICS CONDITION 1 v_error_msg = MESSAGE_TEXT;

                INSERT INTO temp_statement_results
                VALUES (v_customer_id, v_account_id, 'ERROR', v_error_msg, NOW());

                SET p_error_count = p_error_count + 1;
                SET p_error_details = CONCAT(p_error_details,
                    'Account ', v_account_id, ': ', v_error_msg, '; ');

                -- Log error but continue processing
                INSERT INTO processing_errors (
                    process_type, reference_id, error_message, error_date
                ) VALUES (
                    'monthly_statement', v_account_id, v_error_msg, NOW()
                );
            END;

            -- Generate statement for this account
            INSERT INTO monthly_statements (
                customer_id,
                account_id,
                statement_date,
                opening_balance,
                closing_balance,
                total_debits,
                total_credits,
                interest_earned,
                fees_charged,
                statement_period_start,
                statement_period_end
            )
            SELECT
                v_customer_id,
                v_account_id,
                p_process_date,

                -- Opening balance (last day of previous month)
                COALESCE((
                    SELECT balance
                    FROM daily_balances
                    WHERE account_id = v_account_id
                      AND balance_date = LAST_DAY(DATE_SUB(p_process_date, INTERVAL 1 MONTH))
                ), 0),

                v_current_balance, -- Closing balance

                -- Total debits for the month
                COALESCE((
                    SELECT SUM(ABS(amount))
                    FROM transactions
                    WHERE account_id = v_account_id
                      AND amount < 0
                      AND DATE(transaction_date) BETWEEN
                          DATE_FORMAT(p_process_date, '%Y-%m-01')
                          AND LAST_DAY(p_process_date)
                ), 0),

                -- Total credits for the month
                COALESCE((
                    SELECT SUM(amount)
                    FROM transactions
                    WHERE account_id = v_account_id
                      AND amount > 0
                      AND DATE(transaction_date) BETWEEN
                          DATE_FORMAT(p_process_date, '%Y-%m-01')
                          AND LAST_DAY(p_process_date)
                ), 0),

                -- Interest earned (calculated)
                COALESCE((
                    SELECT SUM(interest_amount)
                    FROM interest_calculations
                    WHERE account_id = v_account_id
                      AND calculation_date BETWEEN
                          DATE_FORMAT(p_process_date, '%Y-%m-01')
                          AND LAST_DAY(p_process_date)
                ), 0),

                -- Fees charged
                COALESCE((
                    SELECT SUM(fee_amount)
                    FROM account_fees
                    WHERE account_id = v_account_id
                      AND fee_date BETWEEN
                          DATE_FORMAT(p_process_date, '%Y-%m-01')
                          AND LAST_DAY(p_process_date)
                ), 0),

                DATE_FORMAT(p_process_date, '%Y-%m-01'), -- Period start
                LAST_DAY(p_process_date) -- Period end
            ;

            -- Mark as successful
            INSERT INTO temp_statement_results
            VALUES (v_customer_id, v_account_id, 'SUCCESS', NULL, NOW());

            SET p_success_count = p_success_count + 1;

        END;

    END LOOP;

    CLOSE account_cursor;

    -- Generate summary report
    INSERT INTO batch_processing_summary (
        process_type,
        process_date,
        total_records,
        success_count,
        error_count,
        processing_duration,
        created_at
    ) VALUES (
        'monthly_statements',
        p_process_date,
        p_success_count + p_error_count,
        p_success_count,
        p_error_count,
        TIMESTAMPDIFF(SECOND,
            (SELECT MIN(processing_time) FROM temp_statement_results),
            (SELECT MAX(processing_time) FROM temp_statement_results)
        ),
        NOW()
    );

    -- Cleanup
    DROP TEMPORARY TABLE temp_statement_results;

END//

DELIMITER ;

-- Usage example
CALL process_monthly_statements('2024-01-31', @success, @errors, @error_details);
SELECT @success as successful_statements, @errors as failed_statements, @error_details as error_summary;
```

---

## 6. PERFORMANCE COMPARISON TABLES

### Database Language Performance Comparison

**📍 Location: Section 9.1 Database Language Selection Framework**

| Requirement Type         | SDL | DDL | DML | VDL | Procedural SQL |
| ------------------------ | --- | --- | --- | --- | -------------- |
| Performance Optimization | ★★★ | ★★☆ | ★★★ | ★☆☆ | ★★★            |
| Security Implementation  | ★☆☆ | ★★☆ | ★☆☆ | ★★★ | ★★★            |
| Business Logic           | ☆☆☆ | ★☆☆ | ★★☆ | ☆☆☆ | ★★★            |
| Data Integration         | ★☆☆ | ★★★ | ★★★ | ★★☆ | ★★☆            |
| Compliance Requirements  | ★☆☆ | ★★☆ | ★☆☆ | ★★★ | ★★☆            |

### Performance Impact Metrics

**📍 Location: Section A.1 Storage Definition Language (SDL)**

| Optimization Type         | Performance Improvement | Storage Reduction | Scalability Impact |
| ------------------------- | ----------------------- | ----------------- | ------------------ |
| Proper SDL Implementation | 300-500%                | 20-40%            | High               |
| Index Optimization        | 200-400%                | N/A               | Medium             |
| Partitioning Strategy     | 150-300%                | 10-20%            | Very High          |
| Query Optimization        | 100-250%                | N/A               | Medium             |
| Materialized Views        | 500-1000%               | -10-20%           | High               |

---

## 📋 USAGE INSTRUCTIONS

### How to Use This File:

1. **Copy the entire content** from this markdown file
2. **Create a new repository** on GitHub
3. **Paste the content** into a README.md or examples.md file
4. **Take screenshots** of the formatted code blocks from GitHub
5. **Insert screenshots** into your Word document at the specified locations

### Location Guide:

Each example is marked with **📍 Location** indicating exactly where it should be placed in your Word document. Simply search for the heading text in your Word file and insert the screenshot there.

### GitHub Formatting Benefits:

- **Syntax highlighting** for SQL code
- **Professional table formatting**
- **Clean markdown rendering**
- **Easy copy-paste for future use**

This approach will make your technical report look incredibly professional with properly formatted code examples! 🚀
