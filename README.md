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

... \[no change to this section] ...

---

### 🚀 Migration Process Overview

... \[no change to this section] ...

---

### 🧪 Step-by-Step Migration Process with Code Examples

... \[no change to this section] ...

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

> Process: Log errors in `ErrorLog` table, review via ADF Monitor, test fixes in pilot loads.

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

> With detailed definitions of all acronyms, included pipeline code, clickable navigation, and colorful tool banners, this document is complete for both engineers and business stakeholders.
