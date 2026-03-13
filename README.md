# Supply Chain Logistics Cost & Efficiency Analysis
### Power Query · Excel · Power BI

**By Jasmine Unochi** · [LinkedIn](https://www.linkedin.com/in/jasmine-unochi-4613a3169) · [GitHub](https://github.com/unochifarah)

---

## Overview

This project analyzes supply chain logistics data from the Brunel University London dataset, a real-world multi-table dataset of 7 relational files covering warehouse operations, freight carriers, shipping lanes, and order routing. Using Power Query, Excel, and Power BI, I built a full transformation pipeline across 209,402 order lines to identify freight cost inefficiencies, warehouse bottlenecks, and carrier performance gaps.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Power Query | 7-table transformation pipeline, data merging, calculated columns |
| Excel | Pivot table analysis, carrier, warehouse, and route breakdowns |
| Power BI | Interactive 4-page dashboard with DAX measures |

---

## Business Questions Answered

- Which carrier has the best cost-to-speed ratio?
- Which warehouse is closest to hitting max capacity?
- Which routes are consistently over budget?
- Where should new orders be routed given current warehouse loads?
- How should a discontinued carrier (V44_3) be handled in the data pipeline?

---

## Key Findings

- **V444_0 has a Cost-to-Speed Score of $3.97 vs V444_1's $23.04** — V444_1 costs 6x more per shipment with only a 2.5 percentage point improvement in on-time rate
- **94% of all orders (196,953) flow through a single warehouse — PLANT03** — creating a critical single point of failure in the supply chain
- **PLANT08 has an 82.19% on-time rate**, the lowest of all active plants, against a network average of 98.08%
- **PORT09 → PORT09 averages $1,713 per shipment** — 156x more expensive than the main route, flagged as a data anomaly or special handling scenario
- **Carrier V44_3 has 854 orders with no freight rate data** — a real data integrity issue documented and resolved in the pipeline
- **99.9% of shipments run through a single route: PORT04 → PORT09** — the operation is effectively a single-lane network

---

## Carrier Performance

Compared carriers on cost per shipment, on-time delivery rate, and a combined Cost-to-Speed Score — calculated as `Avg Cost × (1 + (1 − On-Time Rate))` to penalise carriers that are both expensive and unreliable.

| Carrier | Avg Freight Cost | On-Time Rate | Total Orders | Cost-to-Speed Score |
|---|---|---|---|---|
| V444_0 | $3.86 | 97.06% | 124,668 | $3.97 |
| V444_1 | $22.94 | 99.57% | 83,880 | $23.04 |
| V44_3 | — | — (discontinued) | 854 | — |

---

## Warehouse Analysis

| Plant | Total Units | Total Freight Cost | On-Time Rate | Daily Capacity |
|---|---|---|---|---|
| PLANT03 | 545,188,213 | $2,146,605.42 | 98.14% | 1,013 |
| PLANT08 | 5,526,024 | $5,016.88 | 82.19% | 14 |
| PLANT12 | 3,099,780 | $128,780.01 | 100.00% | 209 |
| PLANT16 | 257,585 | $116,508.91 | 100.00% | 457 |
| PLANT13 | 867,580 | $7,998.55 | 100.00% | 490 |
| PLANT09 | 4,476,600 | $761.51 | 100.00% | 11 |
| PLANT04 | 348 | — | 100.00% | 554 |

> **Note:** WhCapacities and OrderList appear to use different unit scales (cases vs. individual units). Direct utilisation % comparison is unreliable without a confirmed conversion factor, flagged as a data limitation.

---

## Data Quality Issues & Resolutions

### Issue 1 — Carrier V44_3 (Discontinued)
V44_3 appears in 854 order lines but has no matching rows in FreightRates — freight cost cannot be calculated for these orders.

- **Quantification:** 854 orders, 0.41% of total volume, 35,102 units affected
- **Resolution:** Left Outer Join retained all V44_3 rows. A `Carrier_Status` column flags these as `"Discontinued"`. All downstream analysis filters to Active carriers by default. V44_3 on-time rate shows 100% but is treated as unreliable due to missing rate data.

### Issue 2 — PORT09 → PORT09 Route Anomaly
207 orders show identical origin and destination port with an average cost of $1,713.37 — 156x more expensive than the main route. Likely internal transfers, special handling fees, or data entry errors. Not excluded, flagged for investigation.

### Issue 3 — Warehouse Capacity Unit Scale Mismatch
Daily Capacity values (max 1,013) cannot be directly compared to Unit Quantity totals (PLANT03: 545M units). The columns appear to use different scales. Utilisation % is presented as-is with this caveat noted.

---

## Business Recommendations

| Priority | Area | Action |
|---|---|---|
| 1 — Immediate | Carrier routing | Route all new orders through V444_0 — 6x better Cost-to-Speed Score |
| 2 — Urgent | PLANT08 | Investigate 82% on-time rate — lowest capacity and worst performance in the network |
| 3 — Investigate | Route anomaly | Review 207 PORT09 → PORT09 orders at $1,713 avg cost before budgeting |
| 4 — Resolve | V44_3 history | Confirm whether 854 discontinued carrier orders were fulfilled and at what actual cost |
| 5 — Risk mitigation | PLANT03 dependency | Redistribute volume to PLANT12 or PLANT16 — both have spare capacity and 100% on-time rates |

---

## Power Query Pipeline

7-table transformation pipeline built in Power Query:

- `OrderList` → `FreightRates` — Left Outer Join on Carrier + Origin Port + Destination Port + Service Level
- `OrderList` → `PlantPorts` — Left Outer Join on Plant Code
- `OrderList` → `WarehouseData` (pre-merged WhCosts + WhCapacities) — Left Outer Join on Plant Code
- `OrderList` → `ProductsPerPlant` — Left Outer Join on Plant Code + Product ID
- `OrderList` → `VmiCustomers` — Left Outer Join on Customer

### Calculated Columns Added in Power Query

- `Delivery_Status` — `"Late"` if Ship Late Day Count > 0, else `"On Time"`
- `Freight_Cost` — `List.Max({[minimum cost], [Weight] * [rate]})` — replicates real carrier billing logic with minimum cost floor
- `Route_Key` — Origin Port & `" → "` & Destination Port
- `Carrier_Status` — `"Discontinued"` where Freight_Cost is null, else `"Active"`

---

## Power BI Dashboard

4-page interactive dashboard:
1. **Executive Overview** — total freight cost, on-time rate, total orders, avg cost per shipment KPI cards; freight cost by carrier and delivery status; on-time vs late donut chart
2. **Carrier Performance Analysis** — avg cost per shipment bar chart, carrier scorecard table, Cost-to-Speed Score comparison
3. **Warehouse Analysis** — order volume by plant bar chart, warehouse scorecard with on-time rates and capacity
4. **Route Analysis** — avg freight cost by route bar chart, full route breakdown table with PORT09 anomaly visible
> ![Executive Overview](Dashboard/Supply_Chain_Logistics_Cost_&_Efficiency_Analysis-0001.jpg) 
> ![Carrier Performance Analysis](Dashboard/Supply_Chain_Logistics_Cost_&_Efficiency_Analysis-0002.jpg) 
> ![Warehouse Analysis](Dashboard/Supply_Chain_Logistics_Cost_&_Efficiency_Analysis-0003.jpg)
> ![Route Analysis](Dashboard/Supply_Chain_Logistics_Cost_&_Efficiency_Analysis-0004.jpg) 

---

## Power Query Concepts Showcased

- Multi-table `Merge Queries` — 5-step join sequence across 7 source files
- `Left Outer Join` — preserves all fact table rows including unmatched (V44_3) records
- `List.Max{}` — minimum cost floor logic replicating real carrier billing
- Custom Columns with conditional logic — `if/then/else` for derived fields
- Data type enforcement — manual typing on all join keys to prevent silent coercion errors
- Whitespace trimming on join keys — prevents invisible character mismatches

---

## Repository Structure

```
supply-chain-logistics-analysis/
  README.md
  data/
    OrderList.csv
    FreightRates.csv
    PlantPorts.csv
    ProductsPerPlant.csv
    VmiCustomers.csv
    WhCapacities.csv
    WhCosts.csv
  Supply_Chain_Analysis.xlsx
  Supply_Chain_Dashboard.pbix
```

---

## Dataset

**Source:** [Supply Chain Logistics Problem](https://brunel.figshare.com/articles/dataset/Supply_Chain_Logistics_Problem_Dataset/7558679) — Brunel University London  
**Records:** 209,402 order lines across 7 relational tables  
**Fields:** Orders, Freight Rates, Plant-Port Mappings, Products, Warehouse Capacities, Warehouse Costs, VMI Customers
