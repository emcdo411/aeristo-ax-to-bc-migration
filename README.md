Aeristo Data Migration Plan: Microsoft Dynamics AX to Dynamics 365 Business Central
📌 Purpose
This migration plan, designed by an Associate Solutions Architect, outlines a step-by-step, end-to-end ETL and data pipeline solution to transfer ~500,000 records from Microsoft Dynamics AX (pre-2023 legacy data) to Microsoft Dynamics 365 Business Central (post-2023 live data). It ensures accurate data extraction, transformation, and loading (ETL) to support real-time analytics via R Shiny applications in RStudio, as well as Excel and Power BI reporting. The plan uses proven SQL commands, OData V4 API calls, Azure Data Factory (ADF) configurations, and R scripts from successful Dynamics migrations. It addresses real-time data updates from Business Central, API reliability, and RStudio integration for charts, visualizations, reports, and applications, aligning with the Aeristo Data Integration Discovery Document.
📚 Table of Contents

Overview and Assumptions
Migration Process Overview
Step-by-Step Migration Process
Step 1: Preparation and Setup
Step 2: Data Extraction from Dynamics AX
Step 3: Staging in Azure SQL Database
Step 4: Data Transformation
Step 5: Loading to Business Central
Step 6: Real-Time Data Updates from Business Central
Step 7: RStudio Integration for Reporting and Applications
Step 8: Validation and Testing
Step 9: Monitoring and Maintenance


Error Handling and Troubleshooting
Tech Stacks
Conclusion

🧭 Overview and Assumptions
Objective: Migrate ~500,000 records from Dynamics AX (Customer, Sales, Inventory, Vendor, Purchasing, General Ledger, Manufacturing) to Business Central, enabling real-time analytics in RStudio via R Shiny.
Assumptions:

Business Central is cloud-based (Azure SQL Database), with REST/OData V4 APIs for data access (on-premises SQL Server as contingency).
Dynamics AX uses SQL Server (on-premises/hosted).
Azure SQL Database is used as a staging layer.
Aeristo IT provides schema documentation, credentials, and API keys.
RStudio is hosted on Azure or internal servers for R Shiny deployment.
Migration timeline: 4-6 weeks (Phase 1 of Aeristo case study).

Scope:

Full historical data migration (~500,000 records).
Incremental daily updates (~1K-5K records).
Real-time API connectivity for RStudio reporting.

🚀 Migration Process Overview
The migration follows a phased ETL pipeline:

Preparation: Set up Azure resources, validate access, and map schemas.
Extraction: Pull data from AX using SQL or DMF.
Staging: Store raw data in Azure SQL.
Transformation: Align AX data to BC schema using ADF and R.
Loading: Push transformed data to BC via OData APIs.
Real-Time Updates: Configure incremental API syncs.
RStudio Integration: Build R Shiny apps for real-time reporting.
Validation: Test data integrity and performance.
Monitoring: Ensure pipeline reliability.

🛠️ Step-by-Step Migration Process
Step 1: Preparation and Setup
Objective: Configure infrastructure and validate access.
Tasks:

Engage Stakeholders:
Meet Aeristo IT to confirm BC deployment (cloud/on-premises), AX SQL credentials, and OData API keys.


Provision Azure Resources:
Create Azure SQL Database (100 DTUs, ~500K records).
Set up ADF instance in Azure portal.


Validate Access:
Test AX SQL Server connection.
Test BC OData API with OAuth 2.0.


Map Schemas:
Use AX AOT and BC metadata (via API: https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/$metadata) to finalize field mappings (e.g., CustTable.AccountNum → Customer.No).


Create Lookup Tables:
In Azure SQL, create tables for code mappings.

CREATE TABLE CodeMapping (
    AX_Code NVARCHAR(50),
    BC_Code NVARCHAR(50),
    Module NVARCHAR(50)
);
INSERT INTO CodeMapping (AX_Code, BC_Code, Module)
VALUES ('CG001', 'RETAIL', 'Customer');



Tools:

Queries AX SQL Server for schema validation.
Provisions Azure SQL and ADF instances.
Tests BC OData API connectivity.

Commands:

Test AX SQL connection:SELECT @@VERSION; -- Verify SQL Server version
SELECT TOP 10 AccountNum FROM CustTable; -- Sample data


Test BC OData API (via Postman):GET https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/customers
Authorization: Bearer {access_token}



Duration: 1 week.
Step 2: Data Extraction from Dynamics AX
Objective: Extract ~500,000 records from AX.
Tasks:

SQL Extraction (preferred):
Use SSMS to query key tables.

SELECT AccountNum, Name, Address, Phone, CustGroup, Email, CreditLimit, PaymentTerms
FROM CustTable;
SELECT SalesId, CustAccount, SalesQty, InvoiceDate, AmountCur, SalesStatus, DeliveryDate
FROM SalesTable;
SELECT ItemId, Name, Qty, InventLocationId, UnitId, ItemGroupId, TransDate
FROM InventTable
WHERE TransDate IS NOT NULL;


DMF Export (if SQL restricted):
In AX, use Data Management Framework to export entities to CSV.
Steps: Navigate to Data Management > Export > Select entity (e.g., “Customers”) > Export to CSV.


Copy to Azure Blob Storage:
Upload SQL results or CSV files to Azure Blob Storage.

azcopy copy "C:\Exports\Customers.csv" "https://{storage_account}.blob.core.windows.net/ax-data/?{sas_token}"



Tools:

Executes SQL queries for AX data extraction.
Exports entities to CSV if SQL access is restricted.
Transfers CSV files to Azure Blob Storage.

Commands:

Export to CSV via SQL:BCP "SELECT * FROM CustTable" queryout "C:\Exports\Customers.csv" -c -T -S {server} -d {database}



Duration: 1 week.
Step 3: Staging in Azure SQL Database
Objective: Store raw AX data in Azure SQL.
Tasks:

Create Staging Tables:
Mirror AX schema in Azure SQL.

CREATE TABLE Staging_CustTable (
    AccountNum NVARCHAR(50),
    Name NVARCHAR(100),
    Address NVARCHAR(255),
    Phone NVARCHAR(20),
    CustGroup NVARCHAR(20),
    Email NVARCHAR(100),
    CreditLimit DECIMAL(18,2),
    PaymentTerms NVARCHAR(20),
    Source NVARCHAR(10) DEFAULT 'AX'
);
CREATE TABLE Staging_SalesTable (
    SalesId NVARCHAR(50),
    CustAccount NVARCHAR(50),
    SalesQty DECIMAL(18,2),
    InvoiceDate DATE,
    AmountCur DECIMAL(18,2),
    SalesStatus NVARCHAR(20),
    DeliveryDate DATE,
    Source NVARCHAR(10) DEFAULT 'AX'
);


Load Data via ADF:
Create ADF pipeline to ingest data from Blob Storage to Azure SQL.
Pipeline JSON (simplified):

{
    "name": "AX_to_AzureSQL",
    "activities": [
        {
            "name": "CopyAXCustomers",
            "type": "Copy",
            "inputs": [{"name": "BlobCustomers"}],
            "outputs": [{"name": "AzureSQLCustomers"}],
            "typeProperties": {
                "source": {"type": "BlobSource"},
                "sink": {"type": "SqlSink"}
            }
        }
    ]
}



Tools:

Stores staged data for transformation.
Orchestrates data loading from Blob Storage.
Holds raw AX data files.

Commands:

Verify data load:SELECT COUNT(*) FROM Staging_CustTable; -- Should be ~500K



Duration: 1 week.
Step 4: Data Transformation
Objective: Align AX data to BC schema.
Tasks:

Create Transformation Pipeline in ADF:
Use ADF Data Flows to map fields, reformat dates, and standardize IDs.
Example mappings:
AccountNum → No (prefix with “AX_”).
InvoiceDate (MM/DD/YYYY) → DocumentDate (YYYY-MM-DD).
Split Address into Address, City, PostCode.


ADF Expression for date:toDate(InvoiceDate, 'MM/dd/yyyy')




R Script for Complex Transformations:
Run in Azure Synapse or local RStudio for address parsing, deduplication.

library(dplyr)
library(stringr)
# Read staged data (via DBI/RODBC)
data <- read.csv("Staging_CustTable.csv") # Placeholder for SQL connection
transformed <- data %>%
  mutate(No = paste0("AX_", AccountNum)) %>%
  mutate(DocumentDate = format(as.Date(InvoiceDate, "%m/%d/%Y"), "%Y-%m-%d")) %>%
  mutate(Address = str_extract(Address, "^[^,]+"),
         City = str_extract(Address, "[A-Za-z]+(?=,\\s\\d{5})"),
         PostCode = str_extract(Address, "\\d{5}")) %>%
  distinct(No, .keep_all = TRUE)
write.csv(transformed, "Transformed_CustTable.csv")


Load Transformed Data to New Azure SQL Tables:CREATE TABLE Transformed_CustTable (
    No NVARCHAR(50),
    Name NVARCHAR(100),
    Address NVARCHAR(100),
    City NVARCHAR(50),
    PostCode NVARCHAR(10),
    PhoneNo NVARCHAR(20),
    CustomerPostingGroup NVARCHAR(20),
    Email NVARCHAR(100),
    CreditLimit DECIMAL(18,2),
    PaymentTermsCode NVARCHAR(20)
);



Tools:

Executes primary transformations and mappings.
Handles complex transformations (e.g., address parsing).
Optional for large-scale transformations.
Stores transformed data.

Duration: 1-2 weeks.
Step 5: Loading to Business Central
Objective: Push transformed data to BC via OData APIs.
Tasks:

Configure BC API Access:
Obtain OAuth 2.0 token via Azure AD.

POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded
grant_type=client_credentials
client_id={client_id}
client_secret={client_secret}
scope=https://api.businesscentral.dynamics.com/.default


Create ADF Pipeline for Loading:
Use OData sink to push data to BC.
Example API call for Customers:

POST https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/customers
Authorization: Bearer {access_token}
Content-Type: application/json
{
    "number": "AX_CUST001",
    "displayName": "Customer Name",
    "addressLine1": "123 Main St",
    "city": "Dallas",
    "postalCode": "75201",
    "phoneNumber": "+1-214-555-1234",
    "email": "info@customer.com",
    "customerFinancialDetails": {
        "creditLimit": 10000.00
    },
    "paymentTermsCode": "NET30"
}


Handle Incremental Loads:
Use ADF to filter changed records.

LastModifiedDateTime >= addDays(utcNow(), -1)



Tools:

Loads data to BC via OData APIs.
Tests API connectivity and payloads.

Commands:

Verify load in BC:GET https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/customers?$filter=number eq 'AX_CUST001'
Authorization: Bearer {access_token}



Duration: 1 week.
Step 6: Real-Time Data Updates from Business Central
Objective: Enable incremental syncs for real-time data.
Tasks:

Configure ADF for Incremental Sync:
Create pipeline to pull daily updates from BC APIs.
Example OData query:

GET https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/salesOrders?$filter=lastModifiedDateTime ge 2025-06-20T00:00:00Z
Authorization: Bearer {access_token}


Stage Updates in Azure SQL:
Update or insert records.

MERGE INTO Transformed_SalesTable AS target
USING Staging_SalesUpdates AS source
ON target.No = source.No
WHEN MATCHED THEN
    UPDATE SET Amount = source.Amount, DocumentDate = source.DocumentDate
WHEN NOT MATCHED THEN
    INSERT (No, Amount, DocumentDate)
    VALUES (source.No, source.Amount, source.DocumentDate);



Tools:

Manages incremental API syncs.
Stores updated records.

Duration: 3-5 days.
Step 7: RStudio Integration for Reporting and Applications
Objective: Build R Shiny apps to pull live BC data for visualizations.
Tasks:

Configure RStudio:
Deploy RStudio on Azure App Service or internal server.
Install packages:

install.packages(c("shiny", "httr", "jsonlite", "dplyr", "ggplot2"))


Connect to BC APIs:
Use httr to authenticate and pull data.

library(httr)
library(jsonlite)
# Get OAuth token
token_response <- POST(
  "https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token",
  body = list(
    grant_type = "client_credentials",
    client_id = "{client_id}",
    client_secret = "{client_secret}",
    scope = "https://api.businesscentral.dynamics.com/.default"
  ),
  encode = "form"
)
access_token <- content(token_response)$access_token
# Pull sales data
sales_response <- GET(
  "https://api.businesscentral.dynamics.com/v2.0/{tenant}/api/v2.0/companies({companyId})/salesOrders",
  add_headers(Authorization = paste("Bearer", access_token))
)
sales_data <- fromJSON(content(sales_response, "text"))$value


Build R Shiny App:
Create dashboard for sales trends.

library(shiny)
library(ggplot2)
library(dplyr)
ui <- fluidPage(
  titlePanel("Aeristo Sales Dashboard"),
  sidebarLayout(
    sidebarPanel(
      dateRangeInput("date_range", "Select Date Range",
                     start = Sys.Date() - 30, end = Sys.Date())
    ),
    mainPanel(
      plotOutput("sales_plot")
    )
  )
)
server <- function(input, output) {
  output$sales_plot <- renderPlot({
    sales_data %>%
      filter(documentDate >= input$date_range[1] & documentDate <= input$date_range[2]) %>%
      summarise(total_amount = sum(amount, na.rm = TRUE), .by = documentDate) %>%
      ggplot(aes(x = as.Date(documentDate), y = total_amount)) +
      geom_line() +
      labs(title = "Daily Sales Trends", x = "Date", y = "Total Amount (USD)")
  })
}
shinyApp(ui = ui, server = server)


Deploy Shiny App:
Host on Azure App Service or Shiny Server.

rsconnect::deployApp(appDir = "path/to/shiny/app", account = "azure_account")



Tools:

Develops and runs R Shiny apps.
Hosts R Shiny applications.
Alternative hosting for Shiny apps.

Duration: 1-2 weeks.
Step 8: Validation and Testing
Objective: Ensure data integrity and performance.
Tasks:

Schema Validation:
Compare AX and BC data:

SELECT COUNT(*) FROM Staging_CustTable; -- ~500K
SELECT COUNT(*) FROM Transformed_CustTable; -- Match staging


Data Integrity:
Sample check via R:

library(DBI)
con <- dbConnect(odbc::odbc(), Driver = "SQL Server", Server = "{azure_sql_server}", Database = "{database}")
sample <- dbGetQuery(con, "SELECT TOP 100 No, Name FROM Transformed_CustTable")
summary(sample)


Performance:
Target <2 hours for daily loads (monitor ADF logs).


Reporting:
Verify R Shiny app pulls live data correctly.



Tools:

Validates staged data.
Runs data integrity checks.
Monitors pipeline performance.
Visualizes data for validation.

Duration: 3-5 days.
Step 9: Monitoring and Maintenance
Objective: Ensure pipeline reliability.
Tasks:

Set Up ADF Monitoring:
Configure alerts for pipeline failures.


Log Errors:
Create Azure SQL error table:

CREATE TABLE ErrorLog (
    ErrorID INT IDENTITY(1,1),
    ErrorMessage NVARCHAR(1000),
    RecordID NVARCHAR(50),
    Timestamp DATETIME DEFAULT GETDATE()
);


Schedule Maintenance:
Rotate API tokens monthly.
Optimize SQL indexes:

CREATE INDEX IX_CustTable_No ON Transformed_CustTable(No);



Tools:

Monitors pipeline execution.
Stores error logs.
Sends alerts for failures.

Duration: Ongoing.
⚠️ Error Handling and Troubleshooting



Error
Cause
Fix



“Cannot convert String to Integer”
Type mismatch (e.g., CreditLimit)
ADF: to_decimal(CreditLimit)


“Customer does not exist”
Missing foreign key
Load Customer before SalesOrders


“Invalid date format”
AX date format
ADF: toDate(InvoiceDate, 'MM/dd/yyyy')


“Record already exists”
Duplicate keys
Prefix IDs (e.g., “AX_”)


“Length exceeds 20”
Field too long
ADF: left(Name, 20)


“Field cannot be empty”
Missing data
ADF: iifNull(Name, 'Unknown')


Process: Log errors in ErrorLog table, review via ADF Monitor, test fixes in pilot loads.
💻 Tech Stacks
  

Orchestrates ETL pipelines.

  

Staging and error logging.

  

AX data extraction.

  

Transformations, Shiny apps.

  

Optional for complex transformations.

  

CSV exports if SQL restricted.

  

Validation via reports.

📩 Conclusion
This migration plan provides a robust, end-to-end ETL solution to transfer Dynamics AX data to Business Central, enabling real-time analytics in RStudio via R Shiny. Using proven SQL, OData, ADF, and R commands, with tools presented in colorful banners, it ensures accurate data migration, reliable API connectivity, and scalable reporting for Aeristo’s 2025 goals.


