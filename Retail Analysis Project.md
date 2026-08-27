# Store Performance, Forecasting & Labor Optimization
### A retail analytics case study covering diagnostics, demand forecasting, and labor efficiency

---

![Dashboard Screenshot](./images/retail-dashboard-screenshot.png)

## Executive Summary

Three metrics anchor this project, each tied directly to the business problem's core question: which stores are underperforming, why, and what is it worth?

| North Star Metric | Finding | Answers |
|---|---|---|
| **Conversion Rate** | Bottom-10 stores convert at 12-13%, well below a healthy specialty-apparel benchmark | Where's the problem? |
| **Forecast Bias %** | Chronic over-forecasting of 20%+ concentrated in January | When's the problem? |
| **Labor Gap $** | ~$48K per store lost to overstaffing relative to demand | How much is it worth? |

The headline finding is that the fleet is losing an estimated $48K per affected store to labor misallocation. Two independent causes are driving it: suburban-archetype stores converting at roughly half the rate of the fleet average, and a seasonal forecasting model that runs more than 20% hot every January, which leads to overstaffing right as demand drops off after the holidays.

---

## The Business Problem

Retail fleets often grow new stores faster than their planning processes mature. Labor tends to get scheduled off flat historical averages instead of a demand-driven forecast, so stores end up overstaffed during slow periods and understaffed during peak traffic at the same time. That second part matters more than it sounds, since it directly costs high-margin, full-price sales in categories where associate-assisted selling drives conversion. On top of that, leadership often lacks a repeatable process for spotting which stores are underperforming and why, so store-level coaching and resource allocation stays reactive instead of data-driven.

The core question: which stores are leaving revenue or margin on the table, is it a traffic problem, a conversion problem, or a basket problem, and how many payroll dollars and dollars of at-risk sales are tied to the gap between forecasted demand and actual staffing?

Why it matters to different stakeholders:
- **Fleet Leaders** need a ranked, explainable list of which stores need intervention and what kind, not a black-box model
- **Finance** needs the labor recommendation converted into real payroll dollars, tied to four-wall contribution
- **Real Estate / Planning** wants this diagnostic pattern applied to new-store ramp decisions as the fleet grows

---

## Methodology

**Data:** three tables, joined on StoreID (and Date where applicable)
- `Dim_Store`: StoreID, Market, Store Format, Square Footage, Open Date, Archetype
- `daily_sales`: StoreID, Date, Net Sales, Transactions, Units, Foot Traffic, Promo Flag, Holiday Flag, Return $, Return Units
- `daily_labor`: StoreID, Date, Scheduled Hours, Hourly Wage Rate, Worked Hours

**Data cleaning and validation:**
- Profiled row counts, date range coverage, and null rates before building any calculations
- Confirmed the StoreID join key matched cleanly across all three tables (250 distinct stores, no orphaned IDs)
- Standardized table naming inconsistencies discovered mid-project (an earlier load had used a different table name than the current source) to avoid querying stale data
- Documented known data limitations instead of working around them. There's no product/category dimension, so product-performance and pricing-strategy slicing is out of scope. No channel/BOPIS field, so omnichannel comp-sales framing is out of scope. No inventory/receipts data, so Sell-Through % and Fill Rate are out of scope

**Analytical approach:**
1. Fleet-wide baseline: monthly trend, promo/holiday-adjusted organic trend, day-of-week pattern
2. Store-level diagnostic: full decomposition (Net Sales = Foot Traffic x Conversion Rate x UPT x AUR), ranked and segmented by Store Format and Archetype
3. Demand forecast: trailing moving-average trend baseline combined with a day-of-week seasonal index, validated against a holdout period using WAPE and directional bias
4. Labor model: earned hours derived from the forecast and a fleet-wide SPLH benchmark, compared against scheduled hours to quantify the labor gap in dollars

---

## Skills

- **BigQuery / SQL**: CTEs, window functions (AVG() OVER, NTILE()), SAFE_DIVIDE for null-safe calculations, date functions, multi-table joins, materialized tables for reusable forecast output
- **Power BI**: dashboard build with DAX time intelligence, KPI cards, slicers by Store Format/Market/Archetype, Analytics pane forecasting

---

## Results & Business Recommendations

**Seasonality**
Average daily sales peak around November and December, with declines starting after January year over year and minimal growth the rest of the year. A summer promotional push could help spark demand during the off-season instead of relying so heavily on holiday volume.

**Promo vs. Holiday Strategy**
Stores generate meaningful organic growth on their own, but promos and holidays still matter as cyclical drivers. Holidays work as an intensity strategy: rare, but each one drives a large lift. Promos work as a frequency strategy: smaller lift per event, but frequent enough to move more total volume. It makes sense to use promo days to cycle staple or slow-moving inventory, and save holiday events for scarcity-driven, limited-item pushes that maximize basket size rather than just traffic.

**Day-of-Week Pattern**
Sales peak on Saturday, about 18% above average, with a low on Tuesday. Staffing should concentrate on peak shopping days to support closer customer attention and drive higher transaction values.

**Store Performance Diagnostic**
The bottom 10 stores by total sales skew heavily suburban, converting at 12-13% with an average dollar sale around $60, both below fleet average. This underperformance clusters in mall and strip-mall suburban locations specifically. Ad targeting should align to the suburban family demographic to match store archetype. For high-traffic urban underperformers, the fix looks different: driving foot traffic and conversion through pop-up events and visual merchandising, staffing to peak hours for closer attention, and stocking by regional customer taste (more loungewear for suburban stores, more athleisure and multifunctional pieces for high-traffic urban).

**Forecast Accuracy**
The stores with the worst WAPE don't overlap with the bottom-decile sales stores. Forecast accuracy and sales underperformance turn out to be two independent issues in the fleet, not one compounding problem.

**Forecast Bias**
Store-level bias is worst in January, with one outlier in October, showing over 20% systematic over-forecasting. The seasonal forecasting model needs adjusting specifically for the holiday-to-January transition, so labor isn't scheduled against demand that's already dropped off.

**Labor Efficiency**
Three stores carry payroll at 45-50% of sales with $40-50 SPLH, a clear overstaffing signal relative to return. Cross-referencing against the bottom-decile sales list confirms this is a separate problem, not the same stores. These stores need reduced scheduled hours and upsell/cross-sell training to raise productivity before any headcount gets added back.

**Labor Gap**
Stores are overstaffed relative to forecasted demand, at an estimated cost of about $48K per store in underutilized labor. Scheduled hours should come down to match earned demand, with the saved coaching time reinvested into training associates to convert traffic into higher-basket transactions. SPLH should improve before staff gets added back.

---

## Next Steps

- Add a Fact_ProductSales table (StoreID, Date, Category, Units, Net Sales) to unlock true product-performance and pricing-strategy analysis, currently out of scope
- Incorporate inventory/receipts data to calculate Sell-Through % and Fill Rate, closing the operational-efficiency gap in the current model
- Add a channel/BOPIS dimension to reflect an omnichannel comp-sales view rather than treating the fleet as pure brick-and-mortar
- Explicitly model promo lift as a third forecast factor (currently a documented simplification, since promo/holiday lift is a modest share of total trend)
- Pilot the labor and targeting recommendations in a small subset of flagged stores before rolling out fleet-wide, to validate the dollar impact against real outcomes
- Automate the pipeline through scheduled BigQuery view refreshes so the dashboard reflects new data without manual re-runs
