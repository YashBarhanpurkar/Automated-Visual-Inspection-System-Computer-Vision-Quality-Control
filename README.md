# Manufacturing KPI Dashboard — Power BI Analytics Solution

> A production-grade Power BI business intelligence solution demonstrating advanced data modelling, DAX engineering, and operational KPI dashboard design — built on the AdventureWorks manufacturing dataset covering $24.9M in revenue across 6 global markets.

---

## Project Overview

This project builds a complete, multi-page Power BI analytics solution for **AdventureWorks** — a global manufacturing company producing cycling equipment. The dataset spans January 2020 to June 2022 across Australia, Canada, France, Germany, UK, and USA.

The focus is on the **engineering quality** of the solution: a clean star-schema data model, robust DAX measure design, production-grade ETL logic in Power Query (M), and a polished multi-page dashboard — the same skills required to build OEE, maintenance, and supply chain dashboards in a real manufacturing environment.

**Data source:** Maven Analytics / Microsoft AdventureWorks sample databases

---

## Dashboard Pages

| Page | Purpose | Key Metrics |
|---|---|---|
| **Executive Summary** | Cross-functional business health at a glance | Revenue, Profit, Orders, Return Rate with drill-through |
| **Map View** | Geospatial market share analysis | Total orders by country — 6 global territories |
| **Product Detail** | SKU-level performance vs. targets | Revenue per product, return %, What-If price adjustment |
| **Customer Detail** | High-value customer identification | Per-customer revenue, order frequency, top customer segmentation |

### Key Business Findings

- **Revenue:** $24.9M total / $10.5M profit over 2.5 years
- **Seasonal peak:** December 2021 reached $1.64M — highest single month, indicating effective seasonal promotions
- **Product mix:** Tires & Tubes are highest volume; Clothing & Accessories deliver highest margin
- **Market efficiency:** USA leads by volume ($7.94M) but Australia leads by revenue-per-customer ($2,131)

---

## Technical Highlights

### Data Model — Star Schema Architecture

A clean star schema with the `Fact_Sales` table at centre and 6 dimension tables:

```
Dim_Calendar ──┐
Dim_Customer ──┤
Dim_Product ───┼──► Fact_Sales ◄──── Fact_Returns
Dim_Category ──┤
Dim_SubCat ────┤
Dim_Territory ─┘
```

**Cyclic reference resolution:** The product hierarchy (Category → Subcategory → Product) created a cyclic dependency that Power BI disables by default due to ambiguous filter propagation. This was resolved by restructuring the relationship chain — demonstrating awareness of how model errors cause incorrect aggregations in production reports.

### Power Query (M) — ETL Engineering

- **Folder Path Parameter:** The entire report reconnects to local data sources on any machine by updating a single parameter — no manual table re-pointing. This mirrors production deployment practice where dashboards connect to network drives or SharePoint.
- **Multi-file append:** Fixed `Table to Binary` conversion errors during the file combination process to successfully stack 3 years of sales CSVs (2020, 2021, 2022) into a single unified fact table.
- **Rolling calendar:** Built a continuous date dimension in M-code to support all time-intelligence DAX functions.

### DAX Measure Engineering

```dax
-- Rolling 90-day revenue
Revenue_90Day = 
CALCULATE([Total Revenue], DATESINPERIOD('Calendar'[Date], LASTDATE('Calendar'[Date]), -90, DAY))

-- YTD with prior year comparison
Revenue_PY = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Calendar'[Date]))

-- Return rate
Return_Rate = DIVIDE([Total Returns], [Total Orders], 0)

-- Adjusted profit (What-If parameter)
Adjusted_Profit = [Total Profit] * (1 + 'Price Adjustment'[Price Adjustment Value])
```

---

## Technical Stack

| Tool | Usage |
|---|---|
| Power BI Desktop | Data modelling, DAX, dashboard design, custom navigation UI |
| Power Query (M) | ETL transformation, file append, parameter-driven paths |
| DAX | KPI measures, time-intelligence, What-If analysis |
| AdventureWorks Dataset | Source data (Microsoft benchmark, via Maven Analytics) |

---

## Repository Structure

```
Adventure_Works_BI_Analytics/
│
├── Report/
│   └── AdventureWorks_Final.pbix          # Full Power BI report file
│
├── AdventureWorks Raw Data/
│   ├── Sales Data/                         # Annual sales CSVs (2020, 2021, 2022)
│   ├── AdventureWorks Customer Lookup.csv
│   ├── AdventureWorks Product Lookup.csv
│   ├── AdventureWorks Calendar Lookup.csv
│   ├── AdventureWorks Territory Lookup.csv
│   ├── AdventureWorks Returns Data.csv
│   └── ...
│
├── Images/                                 # Custom navigation icons for dashboard UI
├── Documentation/
│   └── AdventureWorks_Final.pdf           # PDF export of full report
└── README.md
```

---

## Manufacturing Relevance

The data model architecture and DAX patterns in this solution are **directly transferable** to industrial KPI dashboards:

| AdventureWorks Pattern | Manufacturing Equivalent |
|---|---|
| Rolling 90-day revenue | Rolling OEE trend (7-day, 30-day shopfloor window) |
| Return rate by SKU | Defect / scrap rate by part number or machine |
| YTD orders vs. target | YTD production output vs. capacity plan |
| Territory drill-through | Production line / shift / machine drill-through |
| What-If price adjustment | What-If parameter for OEE target thresholds |
| Seasonal revenue peaks | Seasonal demand planning for production scheduling |

A factory OEE tracker, downtime dashboard, or supply chain KPI report uses the **identical star schema, DAX calculation patterns, and drill-through UX** as this solution — with production telemetry replacing sales transactions.

---

## Future Roadmap

- [ ] Rebuild with a manufacturing-specific dataset (OEE, downtime logs, scrap records) to demonstrate direct shopfloor applicability
- [ ] Add Row-Level Security (RLS) for shift-manager vs. plant-manager access separation
- [ ] Connect to a live SQL database backend (simulating MES data export) for scheduled refresh

---

## Setup

1. Clone the repository
2. Open `Report/AdventureWorks_Final.pbix` in Power BI Desktop
3. Update the `FolderPath` parameter to point to your local `AdventureWorks Raw Data/` directory
4. Refresh data — all tables will reconnect automatically

---

## Author

**Yash Barhanpurkar**  
M.Sc. Global Production Engineering — Technische Universität Berlin  
[LinkedIn](https://www.linkedin.com/in/yash-barhanpurkar/) · [GitHub](https://github.com/YashBarhanpurkar)

---

*Topics: `power-bi` `dax` `data-modelling` `star-schema` `manufacturing-analytics` `business-intelligence` `kpi-dashboard` `power-query` `oee` `adventureworks`*