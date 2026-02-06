![Logo](https://github.com/user-attachments/assets/055ef9ab-58d6-4d7a-8419-0c53365c6a73)  

[Matthew Walters LinkedIn](https://www.linkedin.com/in/matthew-walters-a66a4322)

# Candy Production and Shipping Report

## Problem Statement  
### Stakeholders lacked efficient and consistent visibility into financial and operational performance and were not able to identify sales opportunities by product or geography.  

## Solution  
### Developed an interactive dashboard that provides:  
- Visibility into sales opportunities by geography (national, state, county, or zip) and by product (division, factory, or product)  
- Standardized, efficient, and near real-time reporting tailored for each functional area  
### Semantic Model also includes:  
> Not available in the publish to web report  
- Work In Progress (WIP) capabilites for improved managerial accounting  
- Flexible Time and Variance Intelligence capabilities for all measures  


## [Live Link](https://app.powerbi.com/view?r=eyJrIjoiOGYyYzEzOWYtNTdiOS00YzgyLTkwYTEtOTQyMzUxOTk4ZDcxIiwidCI6ImNlMTg3NGVjLWMwMjktNGY3YS1iMDMyLTkwNGJhMDU4YWE2ZCIsImMiOjN9)  

![Financial Summary](https://mwalters-data-storage.atl1.cdn.digitaloceanspaces.com/US_Candy_Distributor/FinancialSummary.png)  

## Business Insight and Impacts  
- Discovered that Gross Margin and EBIT% were both down ~3% between 2023 and 2025, but Gross Profit and EBIT dollars increased ~50% and ~45% respectively over the same time period. Primary driver was disproportionate increases in shipping expenses relative to sales.  
- Identified the highest concentration of sales opportunities to be in the Northeast, Great Lakes, Florida, Texas, and California.  

![Geographical Sales Opportunities](https://mwalters-data-storage.atl1.cdn.digitaloceanspaces.com/US_Candy_Distributor/Region_Sales_Opportunities.png)  

## Interactivity  
### Financial Summary Page  
Select the metric that will be calculated in matrix.  

![Financial Metric Selection](https://mwalters-data-storage.atl1.cdn.digitaloceanspaces.com/US_Candy_Distributor/metric_selection.gif)  

Hover over the KPI to see a monthly breakdown of performance status and trend for the metric. Status represents favorable or unfavorable YoY variance while trend represents the same for MoM.  

![Financial Metric Hover](https://mwalters-data-storage.atl1.cdn.digitaloceanspaces.com/US_Candy_Distributor/metric_hover.gif)  


Hover over the waterfall chart to see a regional breakdown of the metric.  

![breakdown_hover](https://mwalters-data-storage.atl1.cdn.digitaloceanspaces.com/US_Candy_Distributor/breaddown_hover.gif)  


## Technical  
### Semantic Model Relationships
![Relationships](https://mwalters-data-storage.atl1.cdn.digitaloceanspaces.com/US_Candy_Distributor/Semantic_Model_Relationships.png)  
The active relationship between Calendar and FCT_Sales is through the invoice date, making it the default.

### User-Defined Functions (UDF)
Tested the preview feature for user-defined functions by implementing them with the various date relationships. These are used in both the _Select_Period calculation group and measures that required specific date relationships.  

Example UDF  

    DeliverDate = 
    (
        _measure : EXPR
    ) =>
    CALCULATE (
        _measure,
        USERELATIONSHIP ( FCT_Sales[Arrival Date], 'Calendar'[adjusted_date] )
    )
 Semantic Model has the following UDF's:
 - **Haversine**: given two sets of latitudes and longitudes, calculates the distance along the Earth's surface
 - **DeliverDate**: uses the relationship from the delivery date to the calendar table
 - **InvoiceDate**: uses the relationship from the invoice date to the calendar table
 - **OrderDate**: uses the relationship from the order date to the calendar table
 - **ShipDate**: uses the relationship from the ship date to the calendar table
 - **WIP**: calculates value in a specific stage in the order lifecyle
    - *Production* -> Ordered but not shipped
    - *Shipping* -> Shipped but not delivered
    - *Total* -> Ordered but not delivered
    - *Financial* -> Delivered but not invoiced

### Metrics Table  

The metric table is a disconnected table that allows the user to filter or slice to a measure name and the value in a visual is updated based on the selection.

The metric table is used in conjunction with the KPI_measure. The measure is also dynamically typed, using the stored format string in the table to render the results in the appropriate format.

    KPI Measure = 
    SWITCH (
        SELECTEDVALUE ( _Metrics[Metric] ),
        "REVENUE", InvoiceDate ( [Total Sales] ) + [Revenue Accrued],
        "Sales", InvoiceDate ( [Total Sales] ),
        "Rev Accrual", [Revenue Accrued],
        "COGS", ShipDate ( [Production Costs] ) + DeliverDate ( [Shipping Costs] ),
        "Production Costs", ShipDate ( [Production Costs] ),
        "Shipping Costs", DeliverDate ( [Shipping Costs] ),
        "GROSS PROFIT",
            InvoiceDate ( [Total Sales] ) + [Revenue Accrued]
                - ShipDate ( [Production Costs] )
                - DeliverDate ( [Shipping Costs] ),
        "Gross Margin", [Gross Margin],
        "OPEX",
            [SG&A] + [Marketing Costs] + [Depreciation],
        "SG&A", [SG&A],
        "Marketing", [Marketing Costs],
        "Depreciation", [Depreciation],
        "OPERATING PROFIT", [EBIT],
        "EBIT%", [EBIT %]
    )

## Tools
| Phase | Tool / Technique |
|-------|------------------|
| Data Sourcing | Object Storage (Digital Ocean) |
| ETL | Power Query / M Code / Custom Functions |
| Data Modeling | Star Schema, 1"* Relationships |
| Calculations / Measures | DAX Studio / Power BI Desktop |
| KPI / Visual Icons | Tabular Editor |
| Source Control | Visual Studio Code / GitHub |
| Ad-Hoc Analysis | Excel (live-connected to semantic model)|