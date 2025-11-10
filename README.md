# Complete Guide to Databricks Unity Catalog

Based on this comprehensive video transcript, here's everything explained in detail:

---

## 🎯 **Overview & Importance**

### Why Unity Catalog Matters
- **Critical Statement**: "If you understand Unity Catalog, you can work with Databricks"
- Unity Catalog is the **backbone of modern Databricks architecture**
- Without understanding Unity Catalog, you cannot build modern Databricks solutions
- Companies are specifically looking for Unity Catalog expertise in job descriptions

### Prerequisites
- **No prior experience required** - completely beginner-friendly
- Suitable for:
  - Data Engineers (new to Databricks)
  - Data Analysts
  - Data Scientists
  - Machine Learning Engineers
  - Anyone working with data

---

## 📚 **What is Databricks?**

### Core Concept
Databricks is an **end-to-end platform** for:
- Data Analytics
- Data Engineering
- Data Warehousing
- Machine Learning
- Orchestration
- Data Storage (managed)

### Evolution
- **Originally**: A managed layer on top of Apache Spark
- **Now**: Leading AI and data platform with comprehensive features
- Handles cluster management automatically
- No need to manage underlying VMs or infrastructure

---

## 🗂️ **What is Unity Catalog?**

### Definition
Unity Catalog is a **centralized data catalog** that provides:

1. **Access Control** - Manage who sees what data
2. **Auditing** - Track all queries and user activities
3. **Lineage** - Track data flow from source to destination
4. **Quality Monitoring** - Monitor query performance and data quality
5. **Data Discovery** - Explore table metadata and properties

### Key Concept
Unity Catalog provides a **governing layer** on top of your cloud data storage, allowing you to:
- Create databases (catalogs)
- Create schemas
- Create tables
- Create views
- Create volumes
- Create functions

---

## 🏗️ **Unity Catalog Architecture**

### Four-Level Hierarchy

```
1. Metastore (Level 0)
   └── 2. Catalog (Level 1)
       └── 3. Schema (Level 2)
           └── 4. Objects (Level 3)
               ├── Tables
               ├── Views
               ├── Volumes
               └── Functions
```

### 1. **Metastore** (Top Level)
- **Repository for metadata** about data and AI assets
- One metastore per region
- Can be attached to multiple workspaces
- Stores information about catalogs, schemas, tables, columns, etc.

### 2. **Catalog** (Level 1)
- Equivalent to a **database** in traditional SQL
- Contains schemas
- Can be managed or external
- Three-level naming convention: `catalog.schema.table`

### 3. **Schema** (Level 2)
- Also known as **databases** in some systems
- Contains tables, views, volumes, and functions
- Organizational layer within catalogs
- Common use: bronze, silver, gold schemas (medallion architecture)

### 4. **Objects** (Level 3)
Four types:
- **Tables**: Structured data
- **Views**: Virtual tables based on queries
- **Volumes**: For non-tabular data (files, PDFs, images)
- **Functions**: Custom logic for data masking, transformations

---

## 🔧 **Setting Up Unity Catalog**

### Prerequisites Setup

#### 1. **Azure Account Creation**
- Create free Azure account (30 days free trial)
- Access portal at portal.azure.com
- Create Resource Group named "databricks-unity"

#### 2. **Create Data Lake Storage**
```
Storage Account Name: databricksunity
Type: Azure Blob Storage Gen 2
Workload: Big Data Analytics
Redundancy: LRS (cheapest)
Enable: Hierarchical namespace ✓
```

#### 3. **Create Databricks Workspace**
```
Workspace Name: databricks-unity-workspace
Region: East US
Pricing Tier: Premium (or Trial for 14 days free)
Managed Resource Group: databricks-unity-managed
```

### Access Connector Setup

**Why needed?** 
- Acts as an "ID card" for Databricks to access Azure storage
- Databricks cannot directly access Azure storage without authentication

**Steps:**
1. Create Access Connector for Azure Databricks
2. Grant "Storage Blob Data Contributor" role to connector
3. Use connector in metastore configuration

---

## 🗄️ **Creating Unity Metastore**

### Manual Creation Process

1. **Access Account Console**
   - Use your @*.onmicrosoft.com account (not regular Gmail)
   - Navigate to accounts.azuredatabricks.net
   - Set up admin permissions

2. **Create Metastore**
```sql
Name: metastore-us
Region: East US
ADLS Path: abss://metastore@databricksunity.dfs.core.windows.net/
Access Connector ID: [Resource ID from Azure]
```

3. **Assign Workspace**
   - Attach databricks workspace to metastore
   - Enable Unity Catalog

4. **Set Permissions**
   - Create "admins" group
   - Add yourself to admins group
   - Grant metastore admin rights to group

---

## 📦 **Storage Levels in Unity Catalog**

### Three Storage Level Scenarios

### **Level 1: Metastore-Level Storage**

**Configuration:**
- Metastore ✓ attached to storage
- Catalog: managed (no storage)
- Schema: managed (no storage)
- Tables: managed (no storage)

**Result:** All data stored in metastore location

```sql
-- Create managed catalog
CREATE CATALOG IF NOT EXISTS managed_catalog;

-- Create managed schema
CREATE SCHEMA IF NOT EXISTS managed_catalog.managed_schema;

-- Create managed table
CREATE TABLE IF NOT EXISTS managed_catalog.managed_schema.managed_table (
    id INT,
    name STRING
);

-- Insert data
INSERT INTO managed_catalog.managed_schema.managed_table 
VALUES (1, 'John'), (2, 'Jane');
```

**Data Location:** 
`metastore/tables/[table_identifier]/`

---

### **Level 2: Catalog-Level Storage**

**Configuration:**
- Metastore ✓ attached to storage
- Catalog ✓ attached to storage
- Schema: managed (no storage)
- Tables: managed (no storage)

**Result:** Data stored in catalog location (catalog takes precedence)

```sql
-- Create external catalog with location
CREATE CATALOG IF NOT EXISTS external_catalog
MANAGED LOCATION 'abss://metastore@databricksunity.dfs.core.windows.net/catalog';

-- Create managed schema
CREATE SCHEMA IF NOT EXISTS external_catalog.managed_schema;

-- Create table
CREATE TABLE IF NOT EXISTS external_catalog.managed_schema.employee (
    id INT,
    name STRING,
    salary INT
);
```

**Data Location:** 
`metastore/catalog/tables/[table_identifier]/`

---

### **Level 3: Schema-Level Storage**

**Configuration:**
- Metastore ✓ attached to storage
- Catalog ✓ attached to storage
- Schema ✓ attached to storage
- Tables: managed (no storage)

**Result:** Data stored in schema location (closest to object = highest priority)

```sql
-- Create external schema with location
CREATE SCHEMA IF NOT EXISTS external_catalog.external_schema
MANAGED LOCATION 'abss://metastore@databricksunity.dfs.core.windows.net/schema';

-- Create table
CREATE TABLE IF NOT EXISTS external_catalog.external_schema.external_table (
    id INT,
    name STRING
);
```

**Priority Rule:** 
> The layer **closest to the object** gets priority for data storage

---

## 🔐 **External Locations & Credentials**

### External Data Setup

#### 1. **Create Credentials**
```sql
-- Register access connector as credential
Credential Name: access_connector
Access Connector ID: [Azure Resource ID]
```

#### 2. **Create External Locations**
```sql
-- For raw container
Name: raw
URL: abss://raw@databricksunity.dfs.core.windows.net/
Credential: access_connector

-- For enriched container
Name: enriched
URL: abss://enriched@databricksunity.dfs.core.windows.net/
Credential: access_connector
```

**Purpose:** 
- Credentials = "ID card" for Databricks
- External Locations = Registered storage paths
- One external location per container (best practice)

---

## 📊 **Managed vs External Tables**

### Managed Tables

**Characteristics:**
- ✓ Metadata stored in metastore
- ✓ Data stored in managed location
- ✓ Databricks manages optimization
- ✓ Automatic performance tuning
- ✓ Undrop command available (7 days)

**Behavior on DROP:**
- Metadata: ❌ Deleted
- Data files: ❌ Deleted (but recoverable for 7 days)

```sql
-- Create managed table
CREATE TABLE managed_catalog.managed_schema.managed_table (
    id INT,
    name STRING
);

-- Drop table
DROP TABLE managed_catalog.managed_schema.managed_table;

-- Undrop table (within 7 days)
UNDROP TABLE managed_catalog.managed_schema.managed_table;
```

**Advantages:**
- Databricks handles all optimization
- Automatic liquid clustering
- Best performance without manual tuning
- Data recovery option

---

### External Tables

**Characteristics:**
- ✓ Metadata stored in metastore
- ✓ Data stored in YOUR chosen location
- ✓ You own and control the data
- ✗ Manual optimization required
- ✗ No automatic performance tuning

**Behavior on DROP:**
- Metadata: ❌ Deleted
- Data files: ✓ Remain intact (you own them)

```sql
-- Create external table
CREATE EXTERNAL TABLE bronze.external_schema.external_table (
    id INT,
    name STRING,
    email STRING
)
LOCATION 'abss://raw@databricksunity.dfs.core.windows.net/external_table';

-- Drop table
DROP TABLE bronze.external_schema.external_table;
-- Data still exists in raw container!
```

**Advantages:**
- Complete data ownership
- Data persists after table deletion
- Can share data across systems
- Independent of Databricks

---

### Comparison Table

| Feature | Managed Tables | External Tables |
|---------|---------------|-----------------|
| Data Storage | Databricks-managed | User-managed |
| Optimization | Automatic | Manual |
| DROP Behavior | Data deleted (7-day recovery) | Data preserved |
| Performance | Best (auto-tuned) | Requires tuning |
| Use Case | Production tables | Shared/legacy data |
| Control | Databricks | User |

---

## 📁 **Unity Catalog Volumes**

### What are Volumes?

**Concept:**
- Unity Catalog objects for **non-tabular data**
- Store ANY file type: PDFs, images, CSV, JSON, videos, etc.
- Provides governance for unstructured data
- Works like a folder within your catalog/schema

### Why Volumes?

**Problem:** 
- Tables = structured data only
- Need to govern files, images, documents

**Solution:**
- Volumes = governance layer for files
- Access files using Unity Catalog paths
- Apply permissions and access control

---

### Creating Volumes

#### **Managed Volume** (Databricks-managed storage)

```sql
-- Via UI: Catalog → Schema → Create Volume
Name: managed_volume
Type: Managed
-- Data stored in managed location automatically
```

**Upload files:**
- Click volume → Upload to volume
- Files stored in schema's managed location

---

#### **External Volume** (Your storage)

```sql
-- Create external volume pointing to existing data
CREATE EXTERNAL VOLUME external_catalog.external_schema.external_volume
LOCATION 'abss://raw@databricksunity.dfs.core.windows.net/data';
```

**Accessing Volume Data:**

```sql
-- Read CSV from volume
SELECT * FROM csv.`/Volumes/external_catalog/external_schema/external_volume/dim_airline.csv`;

-- Shortcut: Click volume → Double arrow → Auto-insert path
```

**Key Benefits:**
1. **Governance**: Files tracked in Unity Catalog
2. **Access Control**: Apply permissions like tables
3. **Discoverability**: Browse files in catalog explorer
4. **Unified Access**: Use catalog paths instead of cloud URLs

---

### Volume Use Cases

- **Data Science**: Store models, notebooks, datasets
- **Document Management**: PDFs, Word docs, reports
- **Media Storage**: Images, videos, audio files
- **Raw Data**: JSON, XML, Parquet files before table creation
- **Logs & Archives**: Application logs, audit files

---

## 🔍 **Five Pillars of Data Governance**

### 1. **Access Control**

**Scenario:**
- 100 tables in workspace
- 90 tables → accessible to all users
- 10 tables → only senior engineers/analysts

**Implementation:**
```sql
-- Grant permissions
GRANT SELECT ON TABLE sensitive_table TO `senior_engineers`;
GRANT USE CATALOG ON CATALOG bronze TO `all_users`;
GRANT CREATE SCHEMA ON CATALOG bronze TO `data_engineers`;
```

**Granular Control:**
- Catalog level
- Schema level
- Table level
- Column level
- Row level (using functions)

---

### 2. **Auditing**

**Capabilities:**
- Track all query executions
- See which users run which queries
- Monitor query performance
- Identify resource-intensive queries
- View query history and patterns

**Access:**
- Settings → Audit Logs
- Query History tab
- User activity monitoring
- Performance metrics

---

### 3. **Data Lineage**

**Problem Scenario:**
```
Bronze Layer → Silver Layer → Gold Layer
                                ├── Table 1
                                ├── Table 2
                                └── Table 3 (❌ showing wrong numbers)
```

**Without Lineage:**
- Spend days finding root cause
- Trace back through multiple tables manually

**With Lineage:**
```
Gold Table 3 → uses Silver Table X → uses Bronze Table Y → uses Source A
                                                              └── ⚠️ Issue found!
```

**Features:**
- Visual graph of data dependencies
- Upstream/downstream tracking
- Source identification
- Impact analysis

**Access:**
```
Catalog → Table → Lineage Tab → See Lineage Graph
```

---

### 4. **Quality Monitoring**

**Capabilities:**
- Create monitoring dashboards
- Track query performance metrics
- Monitor data quality over time
- Set up alerts for anomalies

**Setup:**
```
Table → Quality Tab → Enable Data Quality Monitoring
```

**Metrics Tracked:**
- Query execution time
- Data freshness
- Row counts
- Schema changes
- Compute usage

---

### 5. **Data Discovery**

**Information Available:**

**Table Level:**
- Table type (managed/external)
- Storage location
- Data source format (Delta, Parquet, etc.)
- Owner information
- Creation/modification dates
- Size and row count

**Column Level:**
- Data types
- Nullability
- Statistics
- Sample values

**Access:**
```
Catalog Explorer → Select Table → Details Tab
```

**Features:**
- AI-generated descriptions
- Sample data preview
- Column statistics
- Table properties
- Usage insights

---

## 🔒 **Data Security & Masking**

### Problem Statement

**Scenario:**
```
Employee Table:
- ID, Name, Email, Phone ✓ (accessible to all)
- Salary ❌ (confidential - only HR should see)
```

**Requirements:**
- Keep salary column in table
- Most users see: `*******`
- HR team sees: actual salary values

---

### Solution: Data Masking with Functions

### Step 1: Create User Groups

```sql
-- In Account Console → Identity & Access → Groups
Group Name: HR_Team
Members: [Add HR users]
```

---

### Step 2: Create Masking Function

```sql
-- Create masking function
CREATE FUNCTION external_catalog.external_schema.mask_salary(salary INT)
RETURNS INT
RETURN CASE
    WHEN is_account_group_member('HR_Team') THEN salary
    ELSE -1  -- or display '*******'
END;
```

**Function Logic:**
- Check if current user is in `HR_Team`
- If YES → return actual salary
- If NO → return masked value

---

### Step 3: Apply Function to Column

```sql
-- Apply masking to salary column
ALTER TABLE external_catalog.external_schema.employee
ALTER COLUMN salary
SET MASK external_catalog.external_schema.mask_salary;
```

---

### Step 4: Test Results

**As HR Team Member:**
```sql
SELECT * FROM employee;
-- Result: Real salary values visible
```

**As Regular User:**
```sql
SELECT * FROM employee;
-- Result: Salary shows -1 or *******
```

---

### Advanced Masking Options

#### **Email Masking**
```sql
CREATE FUNCTION mask_email(email STRING)
RETURNS STRING
RETURN CASE
    WHEN is_account_group_member('admins') THEN email
    ELSE CONCAT(SUBSTRING(email, 1, 3), '***@***.com')
END;
```

#### **Phone Masking**
```sql
CREATE FUNCTION mask_phone(phone STRING)
RETURNS STRING
RETURN CASE
    WHEN is_account_group_member('customer_service') THEN phone
    ELSE CONCAT('***-***-', SUBSTRING(phone, -4))
END;
```

#### **Conditional Masking**
```sql
CREATE FUNCTION mask_by_department(salary INT, department STRING)
RETURNS INT
RETURN CASE
    WHEN is_account_group_member('executives') THEN salary
    WHEN is_account_group_member('managers') AND department = 'Sales' THEN salary
    ELSE -1
END;
```

---

## 🎓 **Key Concepts Summary**

### Architecture Benefits

**Before Unity Catalog:**
- ❌ One metastore per workspace
- ❌ Multiple storage accounts to manage
- ❌ No cross-workspace sharing
- ❌ Complex governance

**With Unity Catalog:**
- ✅ Single metastore for all workspaces
- ✅ Centralized storage management
- ✅ Easy cross-workspace sharing
- ✅ Unified governance

---

### Best Practices

1. **Naming Convention**
   ```
   Always use: catalog.schema.object
   Example: bronze.raw_data.customers
   ```

2. **Storage Strategy**
   - Metastore level: Base managed location
   - Catalog level: Department-specific storage
   - Schema level: Project-specific storage
   - Table level: External sources only

3. **Security**
   - Use groups, not individual users
   - Apply least privilege principle
   - Mask sensitive columns
   - Audit regularly

4. **Organization**
   ```
   bronze_catalog
   ├── raw_schema (external sources)
   silver_catalog
   ├── curated_schema (cleaned data)
   gold_catalog
   ├── analytics_schema (business metrics)
   ```

---

## 🚀 **Next Steps**

### 1. **Practice Setup**
- Create new Databricks workspace
- Set up Unity Catalog from scratch
- Create all four storage level examples
- Test managed vs external tables

### 2. **Build Projects**
- Implement medallion architecture (Bronze → Silver → Gold)
- Create end-to-end data pipelines
- Apply data governance features
- Set up monitoring and lineage

### 3. **Advanced Topics**
- Delta Lake optimization
- Liquid clustering
- Time travel queries
- Data sharing with Delta Sharing
- ML feature stores

---

## 📝 **Quick Reference Commands**

### Catalog Management
```sql
-- Create catalog
CREATE CATALOG bronze;
CREATE CATALOG silver MANAGED LOCATION 'abss://...';

-- Use catalog
USE CATALOG bronze;

-- Show catalogs
SHOW CATALOGS;

-- Drop catalog
DROP CATALOG bronze CASCADE;
```

### Schema Management
```sql
-- Create schema
CREATE SCHEMA raw_data;
CREATE SCHEMA curated MANAGED LOCATION 'abss://...';

-- Use schema
USE SCHEMA raw_data;

-- Show schemas
SHOW SCHEMAS IN bronze;
```

### Table Operations
```sql
-- Managed table
CREATE TABLE employees (id INT, name STRING);

-- External table
CREATE EXTERNAL TABLE customers (...)
LOCATION 'abss://...';

-- Drop and undrop
DROP TABLE employees;
UNDROP TABLE employees;

-- Describe table
DESCRIBE TABLE EXTENDED employees;
```

### Volume Operations
```sql
-- Create managed volume
CREATE VOLUME my_volume;

-- Create external volume
CREATE EXTERNAL VOLUME raw_files
LOCATION 'abss://raw@storage.dfs.core.windows.net/data';

-- List volumes
SHOW VOLUMES IN schema_name;

-- Read from volume
SELECT * FROM csv.`/Volumes/catalog/schema/volume/file.csv`;
```

### Security & Functions
```sql
-- Create masking function
CREATE FUNCTION mask_data(input INT)
RETURNS INT
RETURN CASE
    WHEN is_account_group_member('admins') THEN input
    ELSE -1
END;

-- Apply mask
ALTER TABLE table_name
ALTER COLUMN column_name
SET MASK function_name;

-- Grant permissions
GRANT SELECT ON TABLE table_name TO `group_name`;
GRANT USE SCHEMA ON SCHEMA schema_name TO `group_name`;
```

---

## 💡 **Key Takeaways**

1. **Unity Catalog is Essential**: Cannot work with modern Databricks without it
2. **Four-Level Hierarchy**: Metastore → Catalog → Schema → Objects
3. **Storage Flexibility**: Choose where data lives at each level
4. **Governance Built-in**: Access control, lineage, auditing included
5. **Managed Tables Preferred**: Better performance, auto-optimization
6. **Volumes for Files**: Govern any file type, not just tables
7. **Data Masking**: Protect sensitive data with functions
8. **Practice Required**: Set up from scratch to truly understand
