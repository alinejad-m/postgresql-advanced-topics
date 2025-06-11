# Hash Indexes in PostgreSQL

## What is a Hash Index?

A hash index is a specialized index structure that stores a hash value of indexed data rather than the actual data. It uses a hash function to convert indexed values into bucket numbers, allowing for extremely fast exact-match queries but not supporting range queries or ordering operations.

## When to Use Hash Indexes?

Hash indexes are ideal for:

1. **Equality Operations**
   - Exact matches only (`column = value`)
   - Simple key-value lookups
   - High-performance point queries

2. **Memory-Resident Data**
   - Fast in-memory operations
   - Frequently accessed lookup tables
   - Session-level temporary tables

3. **Simple Comparisons**
   - Single column equality checks
   - IS NULL / IS NOT NULL conditions
   - Cache-like query patterns

## Why Use Hash Indexes?

### Advantages:
- Faster than B-tree for equality comparisons
- Smaller size than equivalent B-tree
- Better memory utilization
- Excellent for exact matches
- No maintenance required for index balance
- Good for high-cardinality columns

### Best For:
- Columns used only in equality conditions
- High-performance lookup tables
- Temporary tables with frequent exact matches
- Memory-resident datasets
- Large tables with selective equality queries

## How to Create Hash Indexes

### Basic Syntax:
```sql
CREATE INDEX index_name ON table_name USING HASH (column_name);
```

### Examples:

1. **Simple Hash Index:**
```sql
CREATE INDEX idx_users_id_hash ON users USING HASH (id);
```

2. **Hash Index on Text Column:**
```sql
CREATE INDEX idx_sessions_token_hash ON sessions USING HASH (token);
```

3. **Hash Index with Custom Settings:**
```sql
CREATE INDEX idx_cache_key_hash ON cache_table USING HASH (cache_key)
WITH (fillfactor = 75);
```

## Performance Considerations

1. **When Hash Indexes Help:**
   - Equality-only queries
   - High-cardinality columns
   - Frequent point lookups
   - Memory-resident data

2. **When Hash Indexes Don't Help:**
   - Range queries
   - Pattern matching
   - Sorting operations
   - Partial matches

## Maintenance

1. **Monitoring Hash Index Usage:**
```sql
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE indexname LIKE '%hash%';
```

2. **Rebuilding Hash Indexes:**
```sql
REINDEX INDEX index_name;
```

## Best Practices

1. **Index Creation:**
   - Use for equality-only operations
   - Consider table size and memory
   - Monitor performance compared to B-tree
   - Create during low-traffic periods

2. **Usage Guidelines:**
   - Prefer for exact matches only
   - Consider memory requirements
   - Monitor index size growth
   - Compare with B-tree performance

3. **Design Considerations:**
   - Single column only
   - High-cardinality data
   - Equality predicates
   - Memory availability

## Common Use Cases

1. **Session Management**
```sql
CREATE TABLE sessions (
    session_id VARCHAR(64),
    user_data JSONB,
    created_at TIMESTAMP
);

CREATE INDEX idx_session_lookup ON sessions USING HASH (session_id);
```

2. **Cache Tables**
```sql
CREATE TABLE cache (
    cache_key VARCHAR(255),
    cache_value TEXT,
    expires_at TIMESTAMP
);

CREATE INDEX idx_cache_lookup ON cache USING HASH (cache_key);
```

3. **User Authentication**
```sql
CREATE TABLE auth_tokens (
    token VARCHAR(128),
    user_id INTEGER,
    expires_at TIMESTAMP
);

CREATE INDEX idx_token_lookup ON auth_tokens USING HASH (token);
```

## Limitations

1. **Functionality:**
   - No range query support
   - No ordering support
   - Single column only
   - No partial indexes

2. **Performance:**
   - Not suitable for range scans
   - May be slower for small tables
   - Memory intensive
   - No index-only scans

3. **Maintenance:**
   - Cannot be used as unique constraints
   - No support for clustering
   - Limited to equality operations

## Comparing with B-tree

1. **Use Hash When:**
```sql
-- Good for hash indexes
SELECT * FROM users WHERE id = 1234;
SELECT * FROM sessions WHERE token = 'abc123';
```

2. **Use B-tree When:**
```sql
-- Better for B-tree indexes
SELECT * FROM users WHERE id > 1000;
SELECT * FROM users ORDER BY username;
```

## Advanced Topics

1. **Hash Index Internals**
   - Uses a hash function to distribute values
   - Creates fixed-size buckets
   - Handles collisions with bucket chains
   - Automatically expands as needed

2. **Memory Considerations**
```sql
-- Check index size
SELECT pg_size_pretty(pg_relation_size('index_name'));
```

3. **Performance Monitoring**
```sql
-- Monitor hash index effectiveness
SELECT relname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
JOIN pg_stat_user_tables ON idx_rel_parent = relid
WHERE indexrelname LIKE '%hash%';
```

## Tips and Tricks

1. **Converting Existing Indexes**
```sql
DROP INDEX IF EXISTS existing_btree_index;
CREATE INDEX new_hash_index ON table_name USING HASH (column_name);
```

2. **Testing Performance**
```sql
EXPLAIN ANALYZE
SELECT * FROM large_table
WHERE exact_match_column = 'specific_value';
```

3. **Monitoring Hash Collisions**
```sql
SELECT * FROM pg_stat_user_indexes
WHERE indexrelname = 'your_hash_index'
AND idx_tup_read > idx_tup_fetch;
```
