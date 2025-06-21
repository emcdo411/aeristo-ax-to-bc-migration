## Aeristo Data Migration Plan: Microsoft Dynamics AX to Dynamics 365 Business Central

### 📌 Purpose

This migration plan, designed by an Associate Solutions Architect, outlines a step-by-step, end-to-end ETL and data pipeline solution to transfer \~500,000 records from Microsoft Dynamics AX (pre-2023 legacy data) to Microsoft Dynamics 365 Business Central (post-2023 live data). It ensures accurate data extraction, transformation, and loading (ETL) to support real-time analytics via R Shiny applications in RStudio, as well as Excel and Power BI reporting.

> **Note on Acronyms:**
>
> * **ADF (Azure Data Factory)**: A cloud-based data integration tool from Microsoft that allows us to move and transform data at scale.
> * **OData (Open Data Protocol)**: A standard protocol used to retrieve and update data via RESTful APIs.
> * **ETL (Extract, Transform, Load)**: The process of collecting data, converting it to fit new systems, and uploading it into the target platform.
> * **DMF (Data Management Framework)**: A Microsoft Dynamics AX utility to export data into flat files when direct SQL access is not available.
> * **SQL (Structured Query Language)**: A standard language for managing and querying data stored in relational databases.
> * **API (Application Programming Interface)**: A set of rules and protocols for connecting software systems and transferring data.
> * **R Shiny**: An R package used to build interactive web apps for data visualization directly from R.
> * **Azure SQL**: A fully managed cloud database provided by Microsoft Azure, used for storing and processing data securely.
> * **OAuth 2.0**: A secure authorization framework that allows applications to obtain limited access to user accounts on an HTTP service.

---

### 📚 Table of Contents

1. [Overview and Assumptions](#overview-and-assumptions)
2. [Migration Process Overview](#migration-process-overview)
3. [Step-by-Step Migration Process with Code Examples](#step-by-step-migration-process-with-code-examples)
4. [Error Handling and Troubleshooting](#error-handling-and-troubleshooting)
5. [Tech Stacks](#tech-stacks)
6. [Conclusion](#conclusion)

---

### 🛝 Overview and Assumptions

**Objective**: Migrate \~500,000 records from Dynamics AX (Customer, Sales, Inventory, Vendor, Purchasing, General Ledger, Manufacturing) to Business Central, enabling real-time analytics in RStudio via R Shiny.

**Assumptions:**

* Business Central is cloud-based (Azure SQL Database), with REST/OData V4 APIs for data access.
* Dynamics AX uses SQL Server (on-premises/hosted).
* Azure SQL Database is used as a staging layer.
* Aeristo IT provides schema documentation, credentials, and API keys.
* RStudio is hosted on Azure or internal servers for R Shiny deployment.
* Migration timeline: 4-6 weeks.

**Scope:**

* Full historical data migration (\~500,000 records).
* Incremental daily updates (\~1K–5K records).
* Real-time API connectivity for RStudio reporting.

---

### 🚀 Migration Process Overview

The migration follows a phased ETL pipeline:

1. **Preparation** – Set up Azure resources, validate access, map schemas.
2. **Extraction** – Pull data from AX using SQL or DMF.
3. **Staging** – Store raw data in Azure SQL.
4. **Transformation** – Align AX data to BC schema using ADF and R.
5. **Loading** – Push transformed data to BC via OData APIs.
6. **Real-Time Updates** – Configure incremental API syncs.
7. **RStudio Integration** – Build R Shiny apps for real-time reporting.
8. **Validation** – Test data integrity and performance.
9. **Monitoring** – Ensure pipeline reliability.

---

### 🧪 Step-by-Step Migration Process with Code Examples

Each step is detailed with code snippets, tools, and explanation.

#### Step 1: Preparation and Setup

```sql
-- SQL Server Version
SELECT @@VERSION;
-- Sample AX Customer Data
SELECT TOP 10 AccountNum FROM CustTable;
```

```http
-- Test OData API
GET https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/customers
Authorization: Bearer {access_token}
```

**Tools:**
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure) ![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure) ![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman)

---

#### Step 2: Data Extraction from Dynamics AX

```sql
SELECT * FROM CustTable;
SELECT * FROM SalesTable;
```

```bash
azcopy copy "C:\Exports\Customers.csv" "https://yourblob.blob.core.windows.net/ax-data/?{sas_token}"
```

**Tools:**
![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?logo=microsoftsqlserver) ![Azure Blob](https://img.shields.io/badge/Azure_Blob_Storage-0089D6?logo=microsoftazure) ![DMF](https://img.shields.io/badge/Dynamics_AX_DMF-003B57?logo=microsoftdynamics)

---

#### Step 3: Staging in Azure SQL Database

```sql
CREATE TABLE Staging_CustTable (...);
SELECT COUNT(*) FROM Staging_CustTable;
```

**Tools:**
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure) ![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)

---

#### Step 4: Data Transformation

```r
transformed <- data %>%
  mutate(No = paste0("AX_", AccountNum)) %>%
  mutate(DocumentDate = format(as.Date(InvoiceDate, "%m/%d/%Y"), "%Y-%m-%d"))
```

**Tools:**
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure) ![R](https://img.shields.io/badge/R-276DC3?logo=r) ![RStudio](https://img.shields.io/badge/RStudio-75AADB?logo=rstudio)

---

#### Step 5: Loading to Business Central

```http
POST https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/customers
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Tools:**
![OData](https://img.shields.io/badge/OData_V4-9A4993?logo=odata) ![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)

---

#### Step 6: Real-Time Data Updates from BC

```sql
MERGE INTO Transformed_SalesTable AS target
USING Staging_SalesUpdates AS source
ON target.No = source.No
WHEN MATCHED THEN UPDATE ...
WHEN NOT MATCHED THEN INSERT ...;
```

**Tools:**
![OData](https://img.shields.io/badge/OData_V4-9A4993?logo=odata) ![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)

---

#### Step 7: RStudio Integration

```r
library(httr)
token <- POST(...)
sales_data <- GET(...)
```

**Tools:**
![RStudio](https://img.shields.io/badge/RStudio-75AADB?logo=rstudio) ![Shiny](https://img.shields.io/badge/Shiny-17becf?logo=rstudio) ![Azure](https://img.shields.io/badge/Azure_App_Service-0078D4?logo=microsoftazure)

---

#### Step 8: Validation and Testing

```r
summary(sample)
dbGetQuery(con, "SELECT COUNT(*) FROM Transformed_CustTable")
```

**Tools:**
![DBI](https://img.shields.io/badge/R_DBI-276DC3?logo=r) ![R](https://img.shields.io/badge/R-276DC3?logo=r)

---

#### Step 9: Monitoring and Maintenance

```sql
CREATE TABLE ErrorLog (...);
CREATE INDEX IX_CustTable_No ON Transformed_CustTable(No);
```

**Tools:**
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure) ![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)

---

### ⚠️ Error Handling and Troubleshooting

| Error                            | Cause                             | Fix                                      |
| -------------------------------- | --------------------------------- | ---------------------------------------- |
| Cannot convert String to Integer | Type mismatch (e.g., CreditLimit) | ADF: `to_decimal(CreditLimit)`           |
| Customer does not exist          | Missing foreign key               | Load Customer before SalesOrders         |
| Invalid date format              | AX date format                    | ADF: `toDate(InvoiceDate, 'MM/dd/yyyy')` |
| Record already exists            | Duplicate keys                    | Prefix IDs (e.g., `AX_`)                 |
| Length exceeds 20                | Field too long                    | ADF: `left(Name, 20)`                    |
| Field cannot be empty            | Missing data                      | ADF: `iifNull(Name, 'Unknown')`          |

---

### 💻 Tech Stacks

![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?logo=microsoftsqlserver)
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)
![RStudio](https://img.shields.io/badge/RStudio-75AADB?logo=rstudio)
![Shiny](https://img.shields.io/badge/Shiny-17becf?logo=rstudio)
![OData](https://img.shields.io/badge/OData_V4-9A4993?logo=odata)
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)
![Power BI](https://img.shields.io/badge/PowerBI-F2C811?logo=powerbi)
![Excel](https://img.shields.io/badge/Excel-217346?logo=microsoftexcel)

---

### 📩 Conclusion

This migration plan delivers a full-spectrum ETL solution to transition Dynamics AX data to Business Central, enabling robust, real-time R Shiny analytics in RStudio. It ensures reliable pipeline execution using well-documented tools and API integrations to meet Aeristo’s strategic reporting and business intelligence goals.

> With detailed definitions, preserved code, section links, and badges, this is now a fully GitHub-compatible, production-grade documentation file.

