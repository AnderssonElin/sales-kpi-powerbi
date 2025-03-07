# Sales KPIs Dashboard

## Overview
This Power BI report is designed to track and visualize sales Key Performance Indicators (KPIs) for different business lines and products. The dashboard focuses on monitoring revenue streams from various sources including EaaS (Energy as a Service), PPA (Power Purchase Agreement), and Carbon Credit products across different vertical markets.

## Data Sources
The report connects to Dynamics 365 CRM through a direct query connection, pulling data from the following entities:
- Accounts
- Opportunities
- Opportunity Revenue
- Products

## Connection Setup
**IMPORTANT**: This report uses DirectQuery mode to connect to Dynamics 365 CRM. Before you can use the report, you must configure the connection:

1. Open the report in Power BI Desktop
2. Go to "Transform data" > "Data source settings"
3. Find and edit the "Environment" parameter
4. Enter your Dynamics 365 CRM URL (e.g., "yourdynamicsurl.crm4.dynamics.com")
5. Apply the changes and refresh the data

Without proper configuration, the report will not be able to retrieve any data from your Dynamics 365 environment.

## Model Structure
The semantic model consists of several tables:
- **Account**: Contains customer information with vertical categorization
- **Opportunity**: Tracks sales opportunities with status, probability, and value
- **OpportunityRevenue**: Details revenue streams associated with opportunities
- **Product**: Lists available products with associated goals
- **DateTable**: A calculated date table for time intelligence
- **Measure**: Contains all the DAX measures for KPI calculations
- **Toggle**: Provides functionality to switch between weighted and unweighted pipeline views

### Data Model Diagram
```
                  ┌───────────┐
                  │           │
                  │ DateTable │
                  │           │
                  └───────────┘
                        │ 1
                        │ 
                        │ *                    
┌───────────┐     ┌─────▼─────┐     ┌───────────┐
│           │     │           │     │           │
│  Account  ◄────► Opportunity│     │  Product  │
│           │ 1  *│           │     │           │
└───────────┘     └─────▲─────┘     └───────────┘
                        │ 1              │ 1
                        │                │
                        │ *              │
                  ┌─────▼─────┐          │
                  │           │          │
                  │Opportunity◄──────────┘
                  │  Revenue  │    *
                  │           │
                  └───────────┘
```

### Relationship Details
- **OpportunityRevenue.Key_Opportunity** (*) to **Opportunity.Key_Opportunity** (1): Many-to-one relationship with bi-directional filtering (◄──►)
- **Opportunity.Date** (*) to **DateTable.Date** (1): Many-to-one relationship, single direction filtering (→)
- **Opportunity.Key_Account** (*) to **Account.Key_Account** (1): Many-to-one relationship with bi-directional filtering (◄──►)
- **OpportunityRevenue.Product_ID** (*) to **Product.Product_ID** (1): Many-to-one relationship, single direction filtering (→)

## Key Features
- Tracking of sales goals vs. actual performance
- Pipeline analysis with weighted and unweighted views
- Vertical market segmentation (Renewable Energy, E-Commerce, Automotive Machinery, etc.)
- Product-specific performance metrics
- Remaining amount calculations to track progress toward goals
- Annual revenue distribution for multi-year opportunities
  - The model automatically calculates and distributes revenue across years for opportunities that span multiple years
  - Revenue is split based on start and end dates defined in the opportunity
  - This enables accurate year-by-year revenue forecasting and performance tracking
  - Particularly valuable for long-term contracts like PPA and Carbon Credit agreements as used in the default case

## Usage
1. Configure the DirectQuery connection as described in the "Connection Setup" section
2. Filter by vertical market to see specific goals and performance
3. Use the toggle to switch between weighted and unweighted pipeline views
4. Monitor goal achievement with visual indicators
5. Track remaining amounts needed to reach targets

## Performance Considerations
Since this report uses DirectQuery mode, performance depends on:
- The speed of your connection to Dynamics 365
- The volume of data in your CRM system
- The complexity of the queries being executed

For optimal performance, consider applying appropriate filters when analyzing data.

## Documentation
Additional documentation about the measures used in this report can be found in the "Measures" folder. 