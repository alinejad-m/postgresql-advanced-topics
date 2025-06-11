# Unique Indexes in PostgreSQL

## What is a Unique Index?

A unique index is a special type of index that enforces the uniqueness of values in one or more columns of a table. When you create a unique constraint or primary key constraint, PostgreSQL automatically creates a unique index on the specified column(s).

## When to Use Unique Indexes?

Unique indexes are ideal for:

1. **Enforcing Data Integrity**
   - Primary keys
   - Natural unique identifiers (email, username, etc.)
   - Business-specific unique constraints

2. **Data Validation**
   - Preventing duplicate records
   - Ensuring data consistency
   - Maintaining referential integrity

3. **Performance Optimization**
   - Fast lookups for unique values
   - Efficient JOIN operations
   - Index-only scans for covered queries

## Why Use Unique Indexes?

### Advantages:
- Guarantees data uniqueness
- Improves query performance
- Supports foreign key references
- Prevents data inconsistencies
- Automatically maintains data integrity
- Can span multiple columns

### Best For:
- Primary key columns
- Natural unique identifiers
- Business-specific unique constraints
- Columns requiring uniqueness validation

## How to Create Unique Indexes

### Basic Syntax:
```sql
CREATE UNIQUE INDEX index_name ON table_name (column_name);
```

### Examples:

1. **Single Column Unique Index:**
```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

2. **Multi-Column Unique Index:**
```sql
CREATE UNIQUE INDEX idx_order_items ON order_items(order_id, product_id);
```

3. **Partial Unique Index:**
```sql
CREATE UNIQUE INDEX idx_users_active_email 
ON users(email) WHERE active = true;
```

## Performance Considerations

1. **When Unique Indexes Help:**
   - Enforcing data integrity
   - Quick lookups of unique values
   - Join optimization
   - Index-only scans

2. **When to Be Cautious:**
   - High-concurrency INSERT operations
   - Bulk data loading
   - Tables with frequent updates to indexed columns

## Maintenance

1. **Monitoring Violations:**
```sql
-- Check for potential duplicates
SELECT column_name, COUNT(*)
FROM table_name
GROUP BY column_name
HAVING COUNT(*) > 1;
```

2. **Handling Existing Data:**
```sql
-- Remove duplicates before creating unique index
DELETE FROM table_name
WHERE id IN (
    SELECT id
    FROM (
        SELECT id,
        ROW_NUMBER() OVER (PARTITION BY column_name ORDER BY id) AS rnum
        FROM table_name
    ) t
    WHERE t.rnum > 1
);
```

## Best Practices

1. **Index Creation:**
   - Create unique indexes during table creation when possible
   - Use CONCURRENTLY for production environments
   - Consider partial unique indexes for specific use cases

2. **Error Handling:**
   - Implement proper error handling for uniqueness violations
   - Use ON CONFLICT clauses in UPSERT operations
   - Consider deferrable constraints when needed

3. **Design Considerations:**
   - Choose appropriate columns for uniqueness
   - Consider composite unique indexes
   - Plan for NULL values

## Common Use Cases

1. **Primary Keys**
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,  -- Creates unique index automatically
    sku VARCHAR(50) UNIQUE  -- Creates another unique index
);
```

2. **Natural Keys**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    CONSTRAINT users_email_key UNIQUE (email)
);
```

3. **Composite Unique Constraints**
```sql
CREATE TABLE reservations (
    room_id INTEGER,
    date_from DATE,
    date_to DATE,
    CONSTRAINT unique_room_dates UNIQUE (room_id, date_from, date_to)
);
```

## Limitations and Considerations

1. **Performance Impact:**
   - Additional overhead on INSERT/UPDATE operations
   - Lock contention in high-concurrency scenarios
   - Storage space requirements

2. **NULL Handling:**
   - Multiple NULL values are allowed (not considered equal)
   - Consider NOT NULL constraints when needed

3. **Concurrency:**
   - Lock contention during index creation
   - Consider using CREATE UNIQUE INDEX CONCURRENTLY
   - Plan for deadlock scenarios

## Advanced Features

1. **Partial Unique Indexes**
```sql
CREATE UNIQUE INDEX idx_active_users_email 
ON users(email) 
WHERE status = 'active';
```

2. **Deferrable Unique Constraints**
```sql
CREATE TABLE orders (
    id INTEGER,
    order_number VARCHAR(50),
    UNIQUE (order_number) DEFERRABLE INITIALLY DEFERRED
);
```

3. **UPSERT Operations**
```sql
INSERT INTO users (email, name)
VALUES ('user@example.com', 'John Doe')
ON CONFLICT (email) 
DO UPDATE SET name = EXCLUDED.name;
```
