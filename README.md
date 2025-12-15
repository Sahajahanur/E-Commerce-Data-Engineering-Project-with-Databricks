# E-Commerce-Data-Engineering-Project-with-Databricks

## 📋 Project Overview

This project demonstrates a complete end-to-end data engineering pipeline for an e-commerce business using Databricks and the Medallion Architecture (Bronze, Silver, Gold). The project migrates from a legacy AWS-based architecture to a modern, unified Databricks lakehouse platform.

## 🎯 Business Problem

The legacy architecture had several challenges:

* Fragmented ETL - Logic scattered across multiple AWS services (Lambda, Glue, EC2)
* Maintenance Overhead - Complex setup and limited transparency in billing
* Limited Scalability - Inefficient resource usage and high costs
* Lack of Unified Platform - Separate tools for data engineering, analytics, and BI

## 🏗️ Architecture

### Legacy Architecture

    OLTP System → Python Script → S3 (Data Lake) → AWS Services (Lambda/Glue/EC2) → Redshift (Data Warehouse) → BI Tools

### New Databricks Architecture (Data Lakehouse)

    OLTP System → Python Script → S3 → Databricks (Unified Platform) → Databricks BI

<img width="1918" height="837" alt="image" src="https://github.com/user-attachments/assets/385afaff-548f-4b8b-9826-a4f3ff17818e" />
    

## Benefits:

* Single unified platform for data engineering, analytics, and AI
* Native Apache Spark integration for scalability
* Delta Lake format for ACID transactions and time travel
* Integrated BI and analytics capabilities

## 📊 Data Model

### Dimension Tables

* Brands - Brand information (brand_code, brand_name, category_code)
* Categories - Product categories (category_code, category_name)
* Products - Product details (product_id, SKU, color, size, material, weight, price)
* Customers - Customer information (customer_id, phone, country, state, region)
* Date - Calendar dimension (date, year, quarter, month, week, day, is_weekend)

### Fact Table

* Order Items - Transaction data (order_id, date, customer_id, product_id, quantity, unit_price, discount, tax, currency)  

## 🔄 Medallion Architecture

### Bronze Layer (Raw Data)

* Ingests raw CSV files from source
* Minimal transformations
* Stored in Delta format for reliability
* Metadata columns added (source_file, ingested_at)

### Silver Layer (Cleaned Data)

* Data quality improvements:
   * Trimming whitespace
   * Removing special characters
   * Standardizing codes and formats
   * Handling null values
   * Removing duplicates
   * Type conversions (string to date, timestamp, numeric)
* Business logic applied:
   * Normalized category/brand codes
   * Converted channel names (web → website, app → mobile app)
   * Applied currency formatting

## Gold Layer (Business-Ready Data)

### Dimension Tables:

* Products (enriched with category and brand names)
* Customers (enhanced with region mapping)
* Date (added is_weekend, month_name, date_id)

### Fact Table:

 * Calculated columns:

        gross_amount = quantity × unit_price
        discount_amount = gross_amount × discount_percentage
        sales_amount = gross_amount - discount_amount + tax_amount

* Currency normalization (all amounts converted to INR)
* Coupon flag indicator
* Date ID for easy joining

## 🛠️ Technologies Used

* Databricks - Unified data platform
* Apache Spark - Distributed data processing
* Delta Lake - Storage format with ACID transactions
* PySpark - Python API for Spark
* SQL - Data querying and transformations
* Unity Catalog - Data governance and catalog management
* Databricks Workflows - Job orchestration
* Databricks BI - Native dashboarding and visualization
* Genie (AI) - AI-powered data insights

## 📁 Project Structure

      project-ecommerce/
    │
    ├── setup/
    │   └── 01_catalog_setup.ipynb          # Catalog and schema creation
    │
    ├── medallion-processing-dim/
    │   ├── 01_dim_bronze.ipynb             # Dimension tables - Bronze layer
    │   ├── 02_dim_silver.ipynb             # Dimension tables - Silver layer
    │   └── 03_dim_gold.ipynb               # Dimension tables - Gold layer
    │
    ├── medallion-processing-fact/
    │   ├── 01_fact_bronze.ipynb            # Fact table - Bronze layer
    │   ├── 02_fact_silver.ipynb            # Fact table - Silver layer
    │   └── 03_fact_gold.ipynb              # Fact table - Gold layer
    │
    ├── data/
    │   ├── brands/
    │   ├── categories/
    │   ├── products/
    │   ├── customers/
    │   ├── dates/
    │   └── order_items/
    │       └── landing/                     # Daily transaction files
    │
    ├── sql/
    │   └── create_unified_view.sql          # Denormalized view for BI
    │
    └── dashboards/
        └── sales_insights_dashboard.json    # BI dashboard configuration

## 🚀 Setup Instructions

### Prerequisites

* Databricks workspace (Community or Enterprise)
* AWS S3 bucket (for data storage)
* Sample e-commerce dataset

### Step 1: Create Catalog and Schemas
    
      CREATE CATALOG IF NOT EXISTS ecommerce;
      USE CATALOG ecommerce;
    
    CREATE SCHEMA IF NOT EXISTS bronze;
    CREATE SCHEMA IF NOT EXISTS silver;
    CREATE SCHEMA IF NOT EXISTS gold;
    CREATE SCHEMA IF NOT EXISTS source_data;

 ## Step 2: Upload Data to Unity Catalog

1. Navigate to Catalog → Create Volume
2. Create a managed volume named raw under source_data schema
3. Upload CSV files to the volume   

## Step 3: Run Bronze Layer Processing

    # Run notebooks in sequence
    # 01_dim_bronze.ipynb - Process dimension tables
    # 01_fact_bronze.ipynb - Process fact table

## Step 4: Run Silver Layer Processing

        # Run notebooks in sequence
    # 02_dim_silver.ipynb - Clean dimension tables
    # 02_fact_silver.ipynb - Clean fact table
  
## Step 5: Run Gold Layer Processing

    # Run notebooks in sequence
    # 03_dim_gold.ipynb - Create business-ready dimensions
    # 03_fact_gold.ipynb - Create business-ready fact table

## Step 6: Create Unified View

    CREATE OR REPLACE VIEW ecommerce.gold.vw_sales_unified AS
    SELECT 
        f.*,
        d.month_name,
        d.quarter,
        d.is_weekend,
        p.category_name,
        p.brand_name,
        p.material,
        c.region
    FROM ecommerce.gold.fact_order_items f
    LEFT JOIN ecommerce.gold.dim_date d ON f.date_id = d.date_id
    LEFT JOIN ecommerce.gold.dim_products p ON f.product_id = p.product_id
    LEFT JOIN ecommerce.gold.dim_customers c ON f.customer_id = c.customer_id;

<img width="1917" height="847" alt="image" src="https://github.com/user-attachments/assets/39f9bf6c-5aeb-42b0-b2f4-8125b4ae7e50" />

    

## Step 7: Create BI Dashboard

1. Navigate to Dashboards → Create Dashboard
2. Connect to ecommerce.gold.vw_sales_unified
3. Build visualizations:
   
   * Monthly sales trend (line chart)
   * Sales by category (bar chart)
   * Sales heatmap by day/hour
   * Regional performance
   * Coupon usage analytics
  
<img width="1897" height="865" alt="image" src="https://github.com/user-attachments/assets/70eea0fb-9cf8-42f4-8fcd-e39705652346" />

   

## 🤖 AI-Powered Insights with Genie

Databricks Genie provides natural language querying capabilities:

Example Queries:

* "How many transactions were made in US currency?"
* "What is the total revenue in INR by month?"
* "What is the average revenue per region?"
* "Show me monthly sales trends with a chart"

<img width="1918" height="872" alt="image" src="https://github.com/user-attachments/assets/ccb64f14-c351-4f5e-bf59-c819206117da" />



## 📈 Key Transformations

### Data Quality Improvements

* Removed leading/trailing spaces
* Standardized text casing (uppercase/lowercase)
* Removed special characters from codes
* Corrected spelling mistakes in material names
* Converted negative values to positive (ratings)
* Handled null values appropriately
   
### Business Logic

* Currency conversion to INR using exchange rates
* Calculated metrics (gross amount, discount, net sales)
* Region mapping for customers
* Weekend/weekday flags
* Enhanced date attributes (month names, quarter labels)

### Performance Optimizations

* Delta Lake format for faster queries
* Partitioning by date
* Z-ordering for frequently queried columns
* Unified view for simplified BI access

## 🔁 Orchestration (Production Ready)

      # Create a job with tasks
    Task 1: dim_bronze_processing (Notebook)
    Task 2: dim_silver_processing (Depends on Task 1)
    Task 3: dim_gold_processing (Depends on Task 2)
    Task 4: fact_bronze_processing (Parallel with Task 1)
    Task 5: fact_silver_processing (Depends on Task 4)
    Task 6: fact_gold_processing (Depends on Task 5)
    
    # Schedule: Daily at 11:00 PM
    Trigger: Scheduled (Cron: 0 23 * * *)

## 📊 Dashboard Features

### The BI dashboard provides:

* Monthly Sales Trends - Line chart showing revenue over time
* Category Performance - Bar chart of sales by product category
* Sales Heatmap - Day-of-week and hour-of-day analysis
* Regional Insights - Performance by geographic region
* Coupon Analysis - Impact of promotional codes
* Interactive Filters - Category, date range, region selection

## 🎓 Learning Outcomes

### This project demonstrates:

✅ Medallion Architecture implementation

✅ Data quality and cleansing techniques

✅ PySpark transformations and optimizations

✅ Unity Catalog for data governance

✅ Delta Lake features (ACID, time travel)

✅ Databricks Workflows for orchestration

✅ BI dashboard creation

✅ AI-powered analytics with Genie    

<img width="1912" height="858" alt="image" src="https://github.com/user-attachments/assets/3d17629b-bfb0-4cd6-9f23-c118e13ab3c3" />



## 📝 Key Metrics

* Data Volume: 183,000+ transactions (3 months)
* Dimension Tables: 5 tables with 50,000+ products
* Processing Layers: 3 (Bronze → Silver → Gold)
* BI Visualizations: 5+ interactive charts
* Data Quality Rules: 20+ transformations applied

## 👤 Author

Codebasics Tutorial Project

Project completed as part of the Databricks data engineering course
Original tutorial by the Codebasics team  
     



