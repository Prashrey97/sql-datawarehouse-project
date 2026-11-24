# Data Warehouse and Analytics Project
Building a modern data warehouse with SQL Server, including ETL processes, data modelling (Medallion Architecture), and analytics
# Introduction 
Taking raw files from two systems, a CRM and an ERP, and building one data warehouse in SQL Server. At the end, the data will be clean and ready for analysis.
# 🏗️ Data Architecture
The data architecture for this project follows Medallion Architecture Bronze, Silver, and Gold layers:
<img width="1167" height="705" alt="image" src="https://github.com/user-attachments/assets/38044c08-ed81-43a7-aad9-b1665502a4d8" />
- Bronze Layer: Stores raw data as-is from the source systems. Data is ingested from CSV Files into SQL Server Database.
- Silver Layer: This layer includes data cleaning, standardization, and normalization processes to prepare data for analysis.
- Gold Layer: Houses business-ready data modeled into a star schema required for reporting and analytics.
# 📖 Project Overview
The flow of the project is as follows:-
- Designing the architecture: a modern data warehouse with three layers, bronze, silver and gold.
- Building the ETL pipelines: load the source files and move the data through the layers.
- Modelling the data: facts and dimensions, ready for reports.
- Analyzing it: answer business questions with SQL.
# 🚀 Project Requirements
## Building the Data Warehouse (Data Engineering)
1. Objective - Develop a modern data warehouse using SQL Server to consolidate sales data, enabling analytical reporting and informed decision-making.

2. Specifications -
+ Data Sources: Import data from two source systems (ERP and CRM) provided as CSV files.
+ Data Quality: Cleanse and resolve data quality issues prior to analysis.
+ Integration: Combine both sources into a single, user-friendly data model designed for analytical queries.
+ Scope: Focus on the latest dataset only; historization of data is not required.
+ Documentation: Provide clear documentation of the data model to support both business stakeholders and analytics teams.

## BI: Analytics & Reporting (Data Analysis)
1. Objective - Develop SQL-based analytics to deliver detailed insights into:
    - Customer Behavior
    - Product Performance
    - Sales Trends
  
These insights empower stakeholders with key business metrics, enabling strategic decision-making.
