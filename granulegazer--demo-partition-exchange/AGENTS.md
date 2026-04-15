# GitHub Copilot Instructions

## Project Context

This is an Oracle Database 19c demonstration project showcasing partition exchange methodology for efficient data archival. The project uses interval partitioning, compression, and metadata-only partition exchange operations.

## Key Technical Requirements

### Oracle Version
- Target: Oracle Database 19.26
- Compatibility: Oracle 11g and higher
- Edition: Works with Enterprise editions

### Core Technologies
- **Partitioning**: Range partitioning with INTERVAL clause
- **Exchange Method**: `ALTER TABLE ... EXCHANGE PARTITION` with staging tables
- **Compression**: `COMPRESS BASIC` (available in all editions)
- **Indexes**: LOCAL indexes only (partition-aligned)

## Code Generation Guidelines

### 1. Table Creation

Always use `FOR EXCHANGE WITH TABLE` syntax for archive and staging tables:

```sql
-- CORRECT
CREATE TABLE sales_archive
FOR EXCHANGE WITH TABLE sales
PARTITION BY RANGE (sale_date)
INTERVAL (NUMTODSINTERVAL(1, 'DAY'))
(
    PARTITION sales_old VALUES LESS THAN (DATE '2000-01-01')
)
COMPRESS BASIC;

-- AVOID: Manual column definition
CREATE TABLE sales_archive (
    sale_id NUMBER,
    sale_date DATE,
    -- ... manual columns
);
```

### 2. Partition Naming

- Initial partition: `sales_old` (not `p_initial` or `p_archive_initial`)
- Boundary date: `DATE '2000-01-01'` (use DATE literal, not TO_DATE)
- Auto-created partitions: Oracle generates names automatically (e.g., `SYS_P12345`)

### 3. Index Definitions

All indexes must be LOCAL for partitioned tables:

```sql
-- CORRECT
CREATE INDEX idx_sales_date ON sales(sale_date) LOCAL;
CREATE INDEX idx_sales_customer ON sales(customer_id) LOCAL;
CREATE INDEX idx_sales_region ON sales(region) LOCAL;

-- AVOID: Global indexes
CREATE INDEX idx_sales_date ON sales(sale_date);  -- Missing LOCAL
```

### 4. Partition Exchange Pattern

Always use the two-step exchange with staging table:

```sql
-- Step 1: Move partition from source to staging
ALTER TABLE sales EXCHANGE PARTITION partition_name
WITH TABLE sales_staging_temp
INCLUDING INDEXES WITHOUT VALIDATION;

-- Step 2: Move from staging to archive
ALTER TABLE sales_archive EXCHANGE PARTITION partition_name
WITH TABLE sales_staging_temp
INCLUDING INDEXES WITHOUT VALIDATION;

-- Step 3: Drop empty partition from source
ALTER TABLE sales DROP PARTITION partition_name;
```

### 5. Error Handling and Logging

Use the autonomous logging procedure for tracking steps and errors:

```sql
-- Log informational message
prc_log_error_autonomous(
    p_table_name   => 'ARCHIVE_PROCEDURE',
    p_error_type   => 'I',
    p_trace_code   => step_number,
    p_sql_error_num => NULL,
    p_sql_error_msg => NULL,
    p_error_info1  => 'Step description',
    p_error_info2  => additional_info,
    p_user_id      => USER
);

-- Log error
prc_log_error_autonomous(
    p_table_name   => 'ARCHIVE_PROCEDURE',
    p_error_type   => 'E',
    p_trace_code   => step_number,
    p_sql_error_num => SQLCODE,
    p_sql_error_msg => SQLERRM,
    p_error_info1  => 'Error description',
    p_error_info2  => NULL,
    p_user_id      => USER
);
```

Include proper cleanup in exception handlers:

```sql
EXCEPTION
    WHEN OTHERS THEN
        -- Log the error
        prc_log_error_autonomous(
            p_table_name   => 'PROCEDURE_NAME',
            p_error_type   => 'E',
            p_trace_code   => v_step,
            p_sql_error_num => SQLCODE,
            p_sql_error_msg => SQLERRM,
            p_error_info1  => 'Error in procedure',
            p_error_info2  => NULL,
            p_user_id      => USER
        );
        
        -- Clean up staging table if exists
        BEGIN
            EXECUTE IMMEDIATE 'DROP TABLE sales_staging_temp';
        EXCEPTION WHEN OTHERS THEN NULL;
        END;
        ROLLBACK;
        RAISE;
END;
```

### 6. Logging Procedure Specification

**Procedure**: `prc_log_error_autonomous`

**Purpose**: Logs error and informational messages to ERROR_LOG_TABLE using an autonomous transaction.

**Parameters**:
- `p_table_name` - Name of the table or procedure related to the log entry
- `p_error_type` - Type of log entry ('I' for info, 'E' for error)
- `p_trace_code` - Trace code or step number for tracking
- `p_sql_error_num` - SQL error number, if applicable
- `p_sql_error_msg` - SQL error message, if applicable
- `p_error_info1` - Additional information (step, message, details)
- `p_error_info2` - Additional information (optional)
- `p_user_id` - User ID or context for the log entry

**Usage Pattern**:
```sql
-- Start of procedure
v_step := 1;
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 'Starting archival', NULL, USER);

-- Before each major step
v_step := v_step + 1;
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 
    'Exchanging partition', partition_name, USER);

-- After successful step
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 
    'Exchange completed', v_count || ' records', USER);

-- In exception handler
prc_log_error_autonomous('ARCHIVE_PROC', 'E', v_step, SQLCODE, SQLERRM, 
    'Step failed', additional_context, USER);
```

### 7. Table Statistics Function

**Function**: `f_degrag_get_table_size_stats_util`

**Purpose**: Returns partition table size statistics for monitoring and debugging.

**Usage Pattern**:
```sql
-- Log table stats before operation
v_stats := f_degrag_get_table_size_stats_util(p_table_name);
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 
    'Table stats before exchange', v_stats, USER);

-- Log stats after operation
v_stats := f_degrag_get_table_size_stats_util(p_table_name);
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 
    'Table stats after exchange', v_stats, USER);

-- Display stats in output
DBMS_OUTPUT.PUT_LINE('Source table stats: ' || f_degrag_get_table_size_stats_util('SALES'));
DBMS_OUTPUT.PUT_LINE('Archive table stats: ' || f_degrag_get_table_size_stats_util('SALES_ARCHIVE'));
```

**Best Practice**: Call this function before and after partition exchange operations to track:
- Partition count changes
- Space utilization
- Table size metrics
- Performance monitoring data

## Architectural Patterns

### Data Flow
```
Sales Table → Staging Table → Archive Table
  (instant)      (instant)      (compressed)
```

### Table Structure
- **sales**: Active data, interval partitioned, no compression
- **sales_archive**: Historical data, interval partitioned, COMPRESS BASIC
- **sales_staging_template**: Template for creating staging tables
- **sales_unified_view**: UNION ALL view of sales + archive

### Column Exclusions
Do NOT include these columns (removed from design):
- ❌ `archive_date`
- ❌ `archived_by`

## File Organization

```
working/
├── 00_run_all.sql              # Master orchestration script
├── 01_setup.sql                # Tables, types, indexes
├── 02_data_generator.sql       # Test data generation
├── 03_archive_procedure.sql    # Core archival logic
├── 04_test_scenarios.sql       # Test cases
├── 05_monitoring_queries.sql   # Diagnostic queries
├── 06_unified_view.sql         # Unified access view
├── 07_helper_functions.sql     # Utility functions
└── 99_cleanup.sql              # Teardown script
```

## Common Patterns

### Date Array Type Usage
```sql
DECLARE
    v_dates date_array_type;
BEGIN
    v_dates := date_array_type(
        DATE '2024-01-15',
        DATE '2024-01-16'
    );
    archive_partitions_by_dates(v_dates);
END;
/
```

### Partition Date Extraction
```sql
TO_DATE(
    TRIM(BOTH '''' FROM REGEXP_SUBSTR(p.high_value, '''[^'']+''')),
    'YYYY-MM-DD'
)
```

### Dynamic SQL with Partitions
```sql
v_sql := 'SELECT COUNT(*) FROM sales PARTITION (' || partition_name || ')';
EXECUTE IMMEDIATE v_sql INTO v_count;
```

## Anti-Patterns to Avoid

### ❌ Don't Use INSERT for Archival
```sql
-- WRONG: Slow, generates undo, requires twice the space
INSERT INTO sales_archive
SELECT * FROM sales WHERE sale_date < archive_cutoff;

DELETE FROM sales WHERE sale_date < archive_cutoff;
```

### ❌ Don't Use CTAS Without FOR EXCHANGE
```sql
-- WRONG: Structure might not match exactly
CREATE TABLE sales_archive AS SELECT * FROM sales WHERE 1=0;

-- RIGHT: Guaranteed compatibility
CREATE TABLE sales_archive FOR EXCHANGE WITH TABLE sales;
```

### ❌ Don't Mix Global and Local Indexes
```sql
-- WRONG: Can't exchange partitions with global indexes
CREATE INDEX idx_sales_date ON sales(sale_date);  -- Global
CREATE INDEX idx_archive_date ON sales_archive(sale_date) LOCAL;

-- RIGHT: Both must be LOCAL
CREATE INDEX idx_sales_date ON sales(sale_date) LOCAL;
CREATE INDEX idx_archive_date ON sales_archive(sale_date) LOCAL;
```

### ❌ Don't Skip Staging Table
```sql
-- WRONG: Direct exchange would swap data bidirectionally
ALTER TABLE sales EXCHANGE PARTITION p1
WITH TABLE sales_archive PARTITION p1;

-- RIGHT: Use staging as intermediary
CREATE TABLE staging_temp FOR EXCHANGE WITH TABLE sales;
-- Then do two-step exchange
```

## Performance Considerations

### Time Complexity
- Partition Exchange: O(1) - constant time regardless of data volume
- Traditional INSERT/DELETE: O(n) - linear with data volume

### Space Requirements
- Exchange: No temporary space needed
- INSERT/DELETE: 2x data size (undo + redo)

### Locking
- Exchange: Brief metadata lock only
- INSERT/DELETE: Row-level locks for duration of operation

## Suggested Improvements

When suggesting code improvements, consider:
1. Adding error logging to a dedicated table
2. Implementing notification system for archival completion
3. Adding statistics gathering after exchange
4. Creating archival schedules based on retention policies
5. Implementing purge mechanisms for old archived data

## Testing Approach

Always verify:
1. Partition structure matches between tables
2. Indexes are LOCAL and match between tables
3. Data counts before and after exchange
4. Partition can be queried after exchange
5. Unified view returns complete results

## References

- Oracle 19c Partitioning Guide
- `FOR EXCHANGE WITH TABLE` syntax (Oracle 12.2+)
- Interval partitioning behavior
- Partition exchange validation rules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/granulegazer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
