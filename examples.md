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
```

### A.3 Data Manipulation Language (DML)

**📍 Location: SECTION A - A.3 Data Manipulation Language (DML)**

#### Advanced Analytical Queries with Window Functions

```sql
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

### A.5 Procedural vs Declarative Languages

**📍 Location: SECTION A - A.6 Declarative vs Procedural Language Paradigms**

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

    -- Step 4: Calculate final points
    SET p_points_earned = ROUND(v_base_points * v_bonus_multiplier) + v_tier_bonus;

    -- Step 5: Update customer record
    UPDATE customers
    SET
        loyalty_points = loyalty_points + p_points_earned,
        last_updated = p_calculation_date
    WHERE customer_id = p_customer_id;

    -- Commit transaction if no errors
    IF v_error_count = 0 THEN
        COMMIT;
        SET p_processing_status = CONCAT('SUCCESS: Awarded ', p_points_earned, ' points, Tier: ', p_tier_status);
    ELSE
        ROLLBACK;
    END IF;

END//

DELIMITER ;

-- Usage example
CALL calculate_customer_rewards(12345, CURRENT_DATE, @points, @tier, @status);
SELECT @points as points_earned, @tier as new_tier, @status as processing_result;
```

---

## 3. SECTION B - ARCHITECTURE INTEGRATION

### B.1 Data Independence Examples

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

### C.2 Banking System Case Study

**📍 Location: SECTION C - C.2 Financial Services Banking System Case Study**

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
    END as risk_level

FROM customers c
LEFT JOIN accounts a ON c.customer_id = a.customer_id
WHERE c.status IN ('active', 'restricted')
GROUP BY c.customer_id, c.first_name, c.last_name, c.email, c.phone,
         c.customer_since, c.status, c.risk_score
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
    v_velocity_check JSON;
    v_geographic_risk INTEGER := 0;
    v_behavioral_risk INTEGER := 0;
    v_amount_risk INTEGER := 0;
    v_final_decision VARCHAR(20);

BEGIN
    -- Geographic risk assessment
    IF p_location_data->>'country' != 'USA' THEN
        v_geographic_risk := v_geographic_risk + 25;
        v_risk_factors := v_risk_factors || json_build_object(
            'factor', 'international_transaction',
            'score', 25,
            'details', 'Transaction from ' || (p_location_data->>'country')
        );
    END IF;

    -- Behavioral risk assessment
    IF p_amount > 5000 THEN
        v_behavioral_risk := v_behavioral_risk + 30;
        v_risk_factors := v_risk_factors || json_build_object(
            'factor', 'unusual_amount',
            'score', 30,
            'details', 'Transaction amount significantly higher than typical'
        );
    END IF;

    -- Calculate total risk score
    v_risk_score := v_geographic_risk + v_behavioral_risk + v_amount_risk;

    -- Determine final decision
    CASE
        WHEN v_risk_score >= 80 THEN v_final_decision := 'BLOCK';
        WHEN v_risk_score >= 60 THEN v_final_decision := 'REVIEW';
        WHEN v_risk_score >= 40 THEN v_final_decision := 'MONITOR';
        ELSE v_final_decision := 'APPROVE';
    END CASE;

    RETURN json_build_object(
        'risk_score', v_risk_score,
        'decision', v_final_decision,
        'risk_factors', v_risk_factors,
        'assessment_timestamp', NOW()
    );

END;
$$ LANGUAGE plpgsql;
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
