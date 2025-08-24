# DAX Measures | Cereal Market Evolution

## Purpose

In the Cereal Market Evolution dashboard, custom DAX measures were created to power key performance indicators (KPIs) and strategic insights across pricing behavior, sales trends, brand loyalty, consumer flexibility, and promotional response. These measures were used to:

- Track **price per ounce** across brands, retailers, and time  
- Understand **revenue, volume, and margin trends**  
- Measure **perceived vs. actual price deltas**  
- Analyze **customer switching behavior** and **loyalty retention**  
- Evaluate **price volatility and retailer stability**  
- Identify **promotion lift** and **consumer price sensitivity**

The following list documents the key DAX measures developed in support of these insights.

---

### Price per Ounce  
**What it is:** Calculates price per unit of weight (e.g., $/oz)  
**Why it was created:** Enables normalized price comparison across package sizes  
**Business Question:** How do prices compare when adjusted for quantity?

```DAX
Price per Ounce :=
DIVIDE(
    SELECTEDVALUE(Fact_Cereal[Price]),
    SELECTEDVALUE(Fact_Cereal[Ounces])
)
```

---

### Weighted Average Price per Ounce  
**What it is:** Average $/oz weighted by units sold  
**Why it was created:** Avoids skew from low-volume SKUs  
**Business Question:** What was the average unit cost consumers actually paid?

```DAX
Weighted Avg Price per Ounce :=
VAR _total = SUMX(Fact_Cereal, [Price per Ounce] * Fact_Cereal[Units])
VAR _units = SUM(Fact_Cereal[Units])
RETURN DIVIDE(_total, _units)
```

---

### Average Brand Price  
**What it is:** Unit-weighted average price at the brand level  
**Why it was created:** Compares average brand pricing across retailers and time  
**Business Question:** Which brands are priced higher on average, and where?

```DAX
Avg Brand Price :=
VAR _total = SUMX(Fact_Cereal, Fact_Cereal[Price] * Fact_Cereal[Units])
VAR _units = SUM(Fact_Cereal[Units])
RETURN DIVIDE(_total, _units)
```

---

### Perceived vs. Actual Price Delta ($)  
**What it is:** Measures the gap between what consumers expect to pay vs. actual price  
**Why it was created:** Used for analyzing anchoring bias and price perception  
**Business Question:** Are certain brands or retailers perceived as overpriced?

```DAX
Perception Delta ($) :=
AVERAGE(Dim_PricePerception[Perceived_Price]) - [Avg Brand Price]
```

---

### Perceived vs. Actual Price Delta (%)  
**What it is:** Percentage difference between perceived and actual price  
**Why it was created:** Normalize perception gap to any brand/retailer  
**Business Question:** How much more or less do consumers think they’re paying?

```DAX
Perception Delta % :=
DIVIDE(
    AVERAGE(Dim_PricePerception[Perceived_Price]) - [Avg Brand Price],
    [Avg Brand Price]
)
```

---

### Price Volatility (Standard Deviation)  
**What it is:** Measures variability in price per ounce over time  
**Why it was created:** Identifies unstable pricing environments  
**Business Question:** Which retailers or brands have the most price volatility?

```DAX
Price Volatility :=
STDEVX.P(
    SUMMARIZE(
        VALUES(Dim_Calendar[Date]),
        Dim_Calendar[Date],
        "@d", [Weighted Avg Price per Ounce]
    ),
    [@d]
) 
```
🔗 **View KPI in Action:** [Price Volatility Visual & Insight](https://github.com/NicoleBIA/Cereal-Sales-Performance-Inflation-Pricing-and-Consumer-Loyalty-Insights/blob/main/Insights/insights/price_volatility_insights.md)

---

### Sales Volume (Units Sold)  
**What it is:** Total number of units sold  
**Why it was created:** Drives revenue and performance metrics  
**Business Question:** What’s the actual movement of cereal across retailers?

```DAX
Sales Volume :=
SUM(Fact_Cereal[Units])
```

---

### Total Revenue  
**What it is:** Revenue calculated as price × units sold  
**Why it was created:** Tracks revenue contribution by retailer, brand, and promotion  
**Business Question:** What’s the actual dollar value generated?

```DAX
Total Revenue :=
SUMX(Fact_Cereal, Fact_Cereal[Price] * Fact_Cereal[Units])
```

---

### Repeat Purchase Rate  
**What it is:** % of purchases from returning customers  
**Why it was created:** Measures customer loyalty  
**Business Question:** Which brands have the strongest repeat behavior?

```DAX
Repeat Purchase Rate :=
AVERAGE(Dim_Loyalty[Is_Repeat])
```

---

### Switch Frequency Score  
**What it is:** % of customers who switched to a new brand  
**Why it was created:** Measures customer flexibility and brand competition  
**Business Question:** Where is brand loyalty weak or competitive pressure high?

```DAX
Switch Frequency Score :=
AVERAGE(Dim_Loyalty[Switched_Brand])
```

---

### Brand Revenue Share %  
**What it is:** Proportion of revenue held by each brand within a group  
**Why it was created:** For brand comparison and portfolio analysis  
**Business Question:** Which brands dominate revenue in each retail setting?

```DAX
Brand Revenue Share % :=
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALL(Fact_Cereal[Brand]))
)
```

---

### Promotion Lift (%)  
**What it is:** Revenue gain from promotions compared to non-promo periods  
**Why it was created:** Assesses effectiveness of pricing strategy  
**Business Question:** Do promotions actually boost sales meaningfully?

```DAX
Promotion Lift % :=
VAR _promo = CALCULATE([Total Revenue], Fact_Cereal[Promotion] <> "No Promotion")
VAR _noPromo = CALCULATE([Total Revenue], Fact_Cereal[Promotion] = "No Promotion")
RETURN DIVIDE(_promo - _noPromo, _noPromo)
```

---

### YoY Revenue Growth %  
**What it is:** Year-over-year revenue increase  
**Why it was created:** Measures long-term sales trends  
**Business Question:** How is revenue evolving over time?

```DAX
Revenue YoY % :=
VAR _current = [Total Revenue]
VAR _prev = CALCULATE([Total Revenue], DATEADD(Dim_Calendar[Date], -1, YEAR))
RETURN DIVIDE(_current - _prev, _prev)
```

---

### Final Notes

- All DAX measures were designed to work with dynamic slicers for **Year**, **Retailer**, **Brand**, **Promotion Type**, and **Product**
- Key insights powered by these metrics are documented in the `Insights/` folder of this repository
- These measures can be extended or modified for use in future CPG dashboards or price sensitivity research
