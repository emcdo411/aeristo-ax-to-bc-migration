## Aeristo Data Migration Plan: Microsoft Dynamics AX to Dynamics 365 Business Central

### 📌 Purpose

This migration plan, designed by an Associate Solutions Architect, outlines a step-by-step, end-to-end ETL and data pipeline solution to transfer \~500,000 records from Microsoft Dynamics AX (pre-2023 legacy data) to Microsoft Dynamics 365 Business Central (post-2023 live data). It ensures accurate data extraction, transformation, and loading (ETL) to support real-time analytics via R Shiny applications in RStudio, as well as Excel and Power BI reporting.

---

### 🎓 Table of Contents

1. Overview and Assumptions
2. Migration Process Overview
3. Step-by-Step Migration Process
4. Error Handling and Troubleshooting
5. Tech Stacks
6. Conclusion

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
* Incremental daily updates (\~1K-5K records).
* Real-time API connectivity for RStudio reporting.

---

### 🚀 Migration Process Overview

The migration follows a phased ETL pipeline:

1. **Preparation**: Set up Azure resources, validate access, map schemas.
2. **Extraction**: Pull data from AX using SQL or DMF.
3. **Staging**: Store raw data in Azure SQL.
4. **Transformation**: Align AX data to BC schema using ADF and R.
5. **Loading**: Push transformed data to BC via OData APIs.
6. **Real-Time Updates**: Configure incremental API syncs.
7. **RStudio Integration**: Build R Shiny apps for real-time reporting.
8. **Validation**: Test data integrity and performance.
9. **Monitoring**: Ensure pipeline reliability.

---

### 🛠️ Step-by-Step Migration Process

Each step includes **tools** (with banners) and **commands**. Tools are labeled in colorful tech stack style banners.

#### Step 1: Preparation and Setup

**Tasks:**

* Engage Aeristo IT to confirm deployments and credentials.
* Provision Azure SQL and ADF.
* Validate AX SQL & OData API access.
* Map fields via AX AOT and BC metadata.
* Create lookup tables for code mappings.

**Commands:**

```sql
-- Test AX SQL connection
SELECT @@VERSION;
SELECT TOP 10 AccountNum FROM CustTable;
```

```http
-- Test BC OData API
GET https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/customers
Authorization: Bearer {access_token}
```

**📉 Tools:**
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)
![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman)

---

#### Step 2: Data Extraction from Dynamics AX

**Tasks:**

* Query AX SQL via SSMS.
* Use DMF for CSV exports if restricted.
* Upload to Azure Blob Storage via AzCopy.

**Commands:**

```bash
azcopy copy "C:\Exports\Customers.csv" "https://{storage_account}.blob.core.windows.net/ax-data/?{sas_token}"
```

**📉 Tools:**
![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?logo=microsoftsqlserver)
![DMF](https://img.shields.io/badge/Dynamics_AX_DMF-003B57?logo=microsoftdynamics)
![Azure Blob](https://img.shields.io/badge/Azure_Blob_Storage-0089D6?logo=microsoftazure)

---

#### Step 3: Staging in Azure SQL Database

**Tasks:**

* Create raw staging tables in Azure SQL.
* Load using ADF pipeline.

**Commands:**

```sql
SELECT COUNT(*) FROM Staging_CustTable; -- Validate ~500K
```

**📉 Tools:**
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)

---

#### Step 4: Data Transformation

**Tasks:**

* Transform using ADF Data Flows.
* Perform complex parsing in R (RStudio).

**R Code Snippet:**

```r
transformed <- data %>%
  mutate(No = paste0("AX_", AccountNum)) %>%
  mutate(DocumentDate = format(as.Date(InvoiceDate, "%m/%d/%Y"), "%Y-%m-%d"))
```

**📉 Tools:**
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)
![R](https://img.shields.io/badge/R-276DC3?logo=r)
![RStudio](https://img.shields.io/badge/RStudio-75AADB?logo=rstudio)

---

#### Step 5: Loading to Business Central

**Tasks:**

* Authenticate with Azure AD.
* Load customers via OData API.
* Filter for incremental loads.

**Commands:**

```http
POST https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/customers
Authorization: Bearer {access_token}
```

**📉 Tools:**
![OData](https://img.shields.io/badge/OData_V4-9A4993?logo=odata)
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)

---

#### Step 6: Real-Time Data Updates from BC

**Tasks:**

* Create daily sync pipelines.
* Stage updates via MERGE statements.

**📉 Tools:**
![OData](https://img.shields.io/badge/OData_V4-9A4993?logo=odata)
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)

---

#### Step 7: RStudio Integration for Reporting and Applications

**Tasks:**

* Pull API data with `httr` and `jsonlite`.
* Build R Shiny app to display live data.
* Host via Azure or Shiny Server.

**📉 Tools:**
![RStudio](https://img.shields.io/badge/RStudio-75AADB?logo=rstudio)
![Shiny](https://img.shields.io/badge/Shiny-17becf?logo=rstudio)
![Azure](https://img.shields.io/badge/Azure_App_Service-0078D4?logo=microsoftazure)

---

#### Step 8: Validation and Testing

**Tasks:**

* Run count comparisons.
* Sample integrity tests via R DBI.

**📉 Tools:**
![DBI](https://img.shields.io/badge/R_DBI-276DC3?logo=r)
![R](https://img.shields.io/badge/R-276DC3?logo=r)

---

#### Step 9: Monitoring and Maintenance

**Tasks:**

* Monitor ADF with alerts.
* Log issues to SQL table.
* Schedule token rotation and indexing.

**📉 Tools:**
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)

---

### ⚠️ Error Handling and Troubleshooting

| Error                              | Cause         | Fix                     |
| ---------------------------------- | ------------- | ----------------------- |
| "Cannot convert String to Integer" | Type mismatch | `to_decimal()`          |
| "Customer does not exist"          | FK missing    | Load dependencies first |
| "Invalid date format"              | AX formatting | `toDate()` in ADF       |
| "Record already exists"            | Duplicate     | Prefix IDs              |
| "Length exceeds 20"                | Truncation    | Use `left()`            |
| "Field cannot be empty"            | NULLs         | Use `iifNull()`         |

---

### 💻 Tech Stacks

![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?logo=microsoftsqlserver)
![ADF](https://img.shields.io/badge/Azure_Data_Factory-0062AD?logo=microsoftazure)
![RStudio](https://img.shields.io/badge/RStudio-75AADB?logo=rstudio)
![Shiny](https://img.shields.io/badge/Shiny-17becf?logo=rstudio)
![OData](https://img.shields.io/badge/OData_V4-9A4993?logo=odata)
![Azure SQL](https://img.shields.io/badge/Azure_SQL-0078D4?logo=microsoftazure)

---

### 📩 Conclusion

This migration plan delivers a full-spectrum ETL solution to transition Dynamics AX data to Business Central, enabling robust, real-time R Shiny analytics in RStudio. It ensures reliable pipeline execution using well-documented tools and API integrations to meet Aeristo’s strategic reporting and business intelligence goals.
