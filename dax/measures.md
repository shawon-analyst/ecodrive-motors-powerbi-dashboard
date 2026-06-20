# 🧮 DAX Measure Catalogue

Full reference of every DAX measure used in the **Eco Drive Motors** Power BI model, grouped by table and purpose. 80 logic-bearing measures (out of 89 total) are documented here — the remaining 9 are static HTML/SVG layout blobs used purely for cover/section design and are not included for brevity.

> 💡 For a curated walkthrough of the *most advanced* patterns (dynamic Top-N ranking, what-if scenario modelling, DAX-generated SVG icons, auto-written narrative insights, the disconnected-table waterfall pattern), see the **[Advanced DAX Highlights](../README.md#-advanced-dax-highlights)** section in the main README.

---

## KPI 1 & KPI 2 — Sales, Revenue & Profit
*Table: `All_Measure`*

### Core KPI Measures

<details>
<summary><code>Total Revenue</code></summary>

```dax
VAR Revenue_Value = 
    SUMX(
        Fact_Sales, 
        Fact_Sales[Selling Price Per Unit] * Fact_Sales[Units Sold]
    )
    
RETURN
"£" & FORMAT(Revenue_Value / 1000000, "#,##0.0") & "M"
```

</details>

<details>
<summary><code>Total Units Sold</code></summary>

```dax
SUM(Fact_Sales[Units Sold])
```

</details>

<details>
<summary><code>EV Market Share %</code></summary>

```dax
VAR EVUnits = 
    CALCULATE(
        [Total Units Sold], 
        Dim_Vehicle[Vehicle Type] = "Electric"
    )
VAR TotalUnits = [Total Units Sold]

RETURN
DIVIDE(EVUnits, TotalUnits, 0)
```

</details>

<details>
<summary><code>Net Profit Margin %</code></summary>

```dax
VAR Revenue_Value = 
    SUMX(
        Fact_Sales, 
        Fact_Sales[Selling Price Per Unit] * Fact_Sales[Units Sold]
    )

VAR Total_Cost = 
    SUMX(
        Fact_Sales, 
        (Fact_Sales[Aftersales Cost per Unit] + 
         Fact_Sales[Financing Cost per Unit] + 
         Fact_Sales[Labour Cost per Unit] + 
         Fact_Sales[Materials Cost per Unit] + 
         Fact_Sales[SGA Cost per Unit] + 
         Fact_Sales[Taxes Cost per Unit]) * Fact_Sales[Units Sold]
    )

VAR Net_Profit = Revenue_Value - Total_Cost

RETURN
DIVIDE(Net_Profit, Revenue_Value, 0)
```

</details>

<details>
<summary><code>Main_Top_Region_Name</code></summary>

```dax
VAR SummaryTable = 
    CALCULATETABLE(
        SUMMARIZE(
            'Fact_Sales', 
            'Fact_Sales'[Region], 
            "RevenueValue", SUMX('Fact_Sales', 'Fact_Sales'[Units Sold] * 'Fact_Sales'[Selling Price Per Unit])
        ),
        ALL('Fact_Sales'[Region]), 
        KEEPFILTERS(VALUES('Dim_Date'[month_slicer])), 
        KEEPFILTERS(VALUES('Fact_Sales'[Vehicle Type])) 
    )

VAR RankedTable = 
    ADDCOLUMNS(
        SummaryTable,
        "Rank", RANKX(SummaryTable, [RevenueValue], , DESC, Dense)
    )

RETURN 
MAXX(FILTER(RankedTable, [Rank] = 1), 'Fact_Sales'[Region])
```

</details>

<details>
<summary><code>Main_Top_Sales_Rep</code></summary>

```dax
VAR SummaryTable = 
    CALCULATETABLE(
        SUMMARIZE(
            'Fact_Sales', 
            'Fact_Sales'[Sale Representative], 
            "RevenueValue", SUMX('Fact_Sales', 'Fact_Sales'[Units Sold] * 'Fact_Sales'[Selling Price Per Unit])
        ),
        ALL('Fact_Sales'[Sale Representative]), 
        ALL('Fact_Sales'[Region]),             
        KEEPFILTERS(VALUES('Dim_Date'[month_slicer])), 
        KEEPFILTERS(VALUES('Fact_Sales'[Vehicle Type])) 
    )

VAR TopRepTable = TOPN(1, SummaryTable, [RevenueValue], DESC)

RETURN 
MAXX(TopRepTable, 'Fact_Sales'[Sale Representative])
```

</details>

<details>
<summary><code>Total_Revenue_Numeric</code></summary>

```dax
SUMX(
    'Fact_Sales', 
    'Fact_Sales'[Selling Price Per Unit] * 'Fact_Sales'[Units Sold]
)
```

</details>

<details>
<summary><code>Max_Revenue_Value</code></summary>

```dax
MAXX(
    ALLSELECTED('Fact_Sales'[Vehicle Type]), 
    [Total_Revenue_Numeric]
)
```

</details>

<details>
<summary><code>Background_Filler</code></summary>

```dax
[Max_Revenue_Value] - [Total_Revenue_Numeric]
```

</details>

<details>
<summary><code>Custom_Legend_Text</code></summary>

```dax
SELECTEDVALUE('Fact_Sales'[Vehicle Type]) & " — " & 
"£" & FORMAT([Total_Revenue_Numeric] / 1000000, "#,##0.0") & "M" & " (" & 
FORMAT([Net Profit Margin %], "0.0%") & ")"
```

</details>

<details>
<summary><code>Rep_Rank</code></summary>

```dax
IF(
    ISINSCOPE(Dim_Region_Rep[Sale Representative]),
    VAR SalesRank = RANKX(ALLSELECTED(Dim_Region_Rep), [Total_Revenue_Numeric], , DESC)
    RETURN
    IF(SalesRank <= 5, SalesRank, BLANK())
)
```

</details>

<details>
<summary><code>#</code></summary>

```dax
IF(
    ISINSCOPE(Dim_Region_Rep[Sale Representative]), 
    VAR RepRank = 
        RANKX(
            ALL(Dim_Region_Rep), 
            [Total_Revenue_Numeric], 
            , 
            DESC, 
            Dense
        )
    RETURN
    IF(RepRank <= 5, RepRank, BLANK())
)
```

</details>

<details>
<summary><code>Total_COGS</code></summary>

```dax
SUMX(
    'Fact_Sales', 
    ('Fact_Sales'[Labour Cost per Unit] + 'Fact_Sales'[Materials Cost per Unit]) * 'Fact_Sales'[Units Sold]
)
```

</details>

<details>
<summary><code>Gross_Profit</code></summary>

```dax
[Total_Revenue_Numeric] - [Total_COGS]
```

</details>

<details>
<summary><code>Total_Opex</code></summary>

```dax
SUMX(
    'Fact_Sales', 
    ('Fact_Sales'[SGA Cost per Unit] + 
     'Fact_Sales'[Taxes Cost per Unit] + 
     'Fact_Sales'[Financing Cost per Unit] + 
     'Fact_Sales'[Aftersales Cost per Unit]) * 'Fact_Sales'[Units Sold]
)
```

</details>

<details>
<summary><code>Net_Profit</code></summary>

```dax
[Gross_Profit] - [Total_Opex]
```

</details>

<details>
<summary><code>Highest_Margin_Vehicle</code></summary>

```dax
VAR TopVehicle = TOPN(1, ALL('Fact_Sales'[Vehicle Type]), [Net Profit Margin %], DESC)
RETURN
TopVehicle
```

</details>

<details>
<summary><code>AVG MARGIN</code></summary>

```dax
AVERAGEX(
    KEEPFILTERS(VALUES('Fact_Sales'[Model Name])), 
    [Net Profit Margin %]
)
```

</details>

### Subtitle / Context Measures

<details>
<summary><code>Revenue_Subtitle</code></summary>

```dax
FORMAT(MIN(Fact_Sales[Date]), "MMM") & " – " & 
FORMAT(MAX(Fact_Sales[Date]), "MMM yyyy")
```

</details>

<details>
<summary><code>Units_Subtitle</code></summary>

```dax
IF(
    ISFILTERED(Dim_Vehicle[Vehicle Type]), 
    SELECTEDVALUE(Dim_Vehicle[Vehicle Type]), 
    "All vehicle types"
)
```

</details>

<details>
<summary><code>EV_Subtitle</code></summary>

```dax
VAR EV_Count = 
    CALCULATE(
        SUM(Fact_Sales[Units Sold]), 
        Fact_Sales[Vehicle Type] = "Electric"
    )
RETURN
FORMAT(EV_Count, "#,##0") & " EV units"
```

</details>

<details>
<summary><code>Profit_Subtitle</code></summary>

```dax
VAR Revenue = 
    SUMX(
        Fact_Sales, 
        Fact_Sales[Selling Price Per Unit] * Fact_Sales[Units Sold]
    )

VAR Total_Cost = 
    SUMX(
        Fact_Sales, 
        (Fact_Sales[Aftersales Cost per Unit] + 
         Fact_Sales[Financing Cost per Unit] + 
         Fact_Sales[Labour Cost per Unit] + 
         Fact_Sales[Materials Cost per Unit] + 
         Fact_Sales[SGA Cost per Unit] + 
         Fact_Sales[Taxes Cost per Unit]) * Fact_Sales[Units Sold]
    )

VAR Net_Profit_Val = Revenue - Total_Cost

RETURN
"£" & FORMAT(Net_Profit_Val / 1000000, "0.0") & "M net profit"
```

</details>

<details>
<summary><code>Subtitle_Total_Revenue</code></summary>

```dax
VAR SummaryTable = 
    SUMMARIZE(
        ALLSELECTED('Fact_Sales'), 
        'Fact_Sales'[Vehicle Type], 
        "RevValue", [Total Revenue]
    )

VAR TopCatName = MAXX(TOPN(1, SummaryTable, [RevValue], DESC), 'Fact_Sales'[Vehicle Type])
VAR MaxRev = MAXX(TOPN(1, SummaryTable, [RevValue], DESC), [RevValue])

RETURN
"▲ Top: " & TopCatName & " (" & FORMAT(MaxRev, "£#0.0M") & ")"
```

</details>

<details>
<summary><code>Subtitle_Units_Sold_Smart</code></summary>

```dax
VAR IsVehicleFiltered = ISFILTERED('Fact_Sales'[Vehicle Type])
VAR IsRegionFiltered = ISFILTERED('Fact_Sales'[Region])
VAR SelectedVehicle = SELECTEDVALUE('Fact_Sales'[Vehicle Type], "All Types")
VAR SelectedRegion = SELECTEDVALUE('Fact_Sales'[Region], "All Regions")

RETURN
IF(
    IsVehicleFiltered || IsRegionFiltered,
    "Filtered: " & SelectedVehicle & " in " & SelectedRegion,
    "Across all types & regions"
)
```

</details>

<details>
<summary><code>Subtitle_Top_Region</code></summary>

```dax
VAR SummaryTable = 
    CALCULATETABLE(
        SUMMARIZE(
            'Fact_Sales', 
            'Fact_Sales'[Region], 
            "Rev", SUMX('Fact_Sales', 'Fact_Sales'[Units Sold] * 'Fact_Sales'[Selling Price Per Unit])
        ),
        ALL('Fact_Sales'[Region]),
        KEEPFILTERS(VALUES('Dim_Date'[month_slicer])),
        KEEPFILTERS(VALUES('Fact_Sales'[Vehicle Type]))
    )

VAR MaxRev = MAXX(SummaryTable, [Rev])

RETURN
FORMAT(MaxRev / 1000000, "£#0.0") & "M revenue"
```

</details>

<details>
<summary><code>Subtitle_Top_Sales_Rep</code></summary>

```dax
VAR SummaryTable = 
    CALCULATETABLE(
        SUMMARIZE(
            'Fact_Sales', 
            'Fact_Sales'[Sale Representative], 
            'Fact_Sales'[Region],            
            "RevenueValue", SUMX('Fact_Sales', 'Fact_Sales'[Units Sold] * 'Fact_Sales'[Selling Price Per Unit])
        ),
        ALL('Fact_Sales'[Sale Representative]), 
        ALL('Fact_Sales'[Region]),              
        KEEPFILTERS(VALUES('Dim_Date'[month_slicer])), 
        KEEPFILTERS(VALUES('Fact_Sales'[Vehicle Type])) 
    )

VAR MaxRevValue = MAXX(SummaryTable, [RevenueValue])

VAR TopRepTable = TOPN(1, SummaryTable, [RevenueValue], DESC)
VAR TopRegionName = MAXX(TopRepTable, [Region])

RETURN
FORMAT(MaxRevValue / 1000000, "£#0.0") & "M · " & TopRegionName
```

</details>

<details>
<summary><code>Dynamic_Gap_Analysis</code></summary>

```dax
VAR Bullet = "🟡" 
VAR ManchesterRev = CALCULATE([Total_Revenue_Numeric], 'Fact_Sales'[Region] = "Manchester")
VAR BirminghamRev = CALCULATE([Total_Revenue_Numeric], 'Fact_Sales'[Region] = "Birmingham")
VAR GapAmt = (ManchesterRev - BirminghamRev) / 1000000
VAR GapPct = DIVIDE((ManchesterRev - BirminghamRev), BirminghamRev)

RETURN
Bullet & "  Gap Analysis: Manchester outperforms Birmingham by £" & 
FORMAT(GapAmt, "#,##0.1") & "M (" & 
FORMAT(GapPct, "0.0%") & "). " & 
"Regional differentiation is relatively low, indicating balanced market penetration."
```

</details>

<details>
<summary><code>Dynamic_Trend_Insight</code></summary>

```dax
VAR ResultText = 
    "All three vehicle types maintained a strong market presence across Jan-Mar 2023. While revenue experienced fluctuations, EV revenue continues to show the most promising long-term growth rate."

RETURN
ResultText
```

</details>

<details>
<summary><code>GP_Margin_%</code></summary>

```dax
DIVIDE([Gross_Profit],[Total_Revenue_Numeric])
```

</details>

<details>
<summary><code>GP_Subtitle</code></summary>

```dax
VAR Margin = [GP_Margin_%]
RETURN
FORMAT(Margin, "0.0%") & " of total revenue"
```

</details>

<details>
<summary><code>fake_kpi</code></summary>

```dax
""
```

</details>

<details>
<summary><code>NP_Subtitle</code></summary>

```dax
VAR NP_Margin = DIVIDE([Net_Profit], [Total_Revenue_Numeric]) 
RETURN
FORMAT(NP_Margin, "0.0%") & " overall margin"
```

</details>

<details>
<summary><code>Highest_Margin_Subtitle</code></summary>

```dax
VAR MaxMargin = MAXX(TOPN(1, ALL('Fact_Sales'[Vehicle Type]), [Net Profit Margin %], DESC), [Net Profit Margin %])
RETURN
FORMAT(MaxMargin, "0.0%") & " net margin"
```

</details>

### Conditional Formatting (Color Logic)

<details>
<summary><code>Subtitle_Color</code></summary>

```dax
VAR SummaryTable = 
    SUMMARIZE(
        ALLSELECTED('Fact_Sales'), 
        'Fact_Sales'[Vehicle Type], 
        "RevValue", [Total Revenue]
    )

VAR MaxRevText = MAXX(TOPN(1, SummaryTable, [RevValue], DESC), [RevValue])

VAR CleanNumber = 
    VALUE(
        SUBSTITUTE(
            SUBSTITUTE(
                SUBSTITUTE(MaxRevText, "£", ""), 
            "M", ""), 
        ",", "")
    )

RETURN 
IF(CleanNumber >= 0, "#228B22", "#FF0000")
```

</details>

<details>
<summary><code>Revenue_Color</code></summary>

```dax
VAR SalesRank = 
    RANKX(
        ALL(Dim_Region_Rep), 
        [Total_Revenue_Numeric], 
        , 
        DESC, 
        Dense
    )

RETURN
IF(
    ISINSCOPE(Dim_Region_Rep[Sale Representative]),
    SWITCH( TRUE(),
        SalesRank = 1, "#228B22", 
        SalesRank = 5, "#B22222",
        "#000000"                 
    ),
    BLANK()
)
```

</details>

<details>
<summary><code>GapValue_Measure</code></summary>

```dax
VAR TopRevenue = MAXX(TOPN(1, ALL(Dim_Region_Rep), [Total_Revenue_Numeric], DESC), [Total_Revenue_Numeric])
VAR BottomRevenue = MINX(TOPN(5, ALL(Dim_Region_Rep), [Total_Revenue_Numeric], DESC), [Total_Revenue_Numeric])
VAR GapValue = (TopRevenue - BottomRevenue) / 1000000 

RETURN
FORMAT(GapValue, "£#0.0") & "M"
```

</details>

<details>
<summary><code>GapPercent_Measure</code></summary>

```dax
VAR TopRevenue = MAXX(TOPN(1, ALL(Dim_Region_Rep), [Total_Revenue_Numeric], DESC), [Total_Revenue_Numeric])
VAR BottomRevenue = MINX(TOPN(5, ALL(Dim_Region_Rep), [Total_Revenue_Numeric], DESC), [Total_Revenue_Numeric])

VAR GapValue = TopRevenue - BottomRevenue

VAR GapPercent = DIVIDE(GapValue, BottomRevenue) 

RETURN
"(" & FORMAT(GapPercent, "0.0%") & ")"
```

</details>

<details>
<summary><code>Gross_Profit_Color</code></summary>

```dax
VAR GP = [Gross_Profit]
RETURN
IF(
    GP >= 0, 
    "#228B22", 
    "#FF0000"  
)
```

</details>

<details>
<summary><code>Net_Profit_Color</code></summary>

```dax
VAR NP = [Net_Profit]
RETURN
IF(
    NP >= 0, 
    "#228B22",
    "#FF0000"  
)
```

</details>

<details>
<summary><code>Matrix_BG_Color</code></summary>

```dax
VAR AvgMargin = [Net Profit Margin %]
RETURN
    SWITCH( TRUE(),
        AvgMargin >= 0.24, "#E2EFDA", 
        AvgMargin >= 0.20, "#FFF2CC", 
        "#FBE5D6"                     
    )
```

</details>

<details>
<summary><code>Net_Profit_Margin_Color</code></summary>

```dax
VAR NPM = [Net Profit Margin %]
RETURN
IF(
    ISBLANK(NPM), 
    "#808080",
    IF(
        NPM >= 0, 
        "#006400", 
        "#FF0000"
    )
)
```

</details>

<details>
<summary><code>Margin_Icon_Logic</code></summary>

```dax
VAR SummaryTable = 
    ALLSELECTED('Fact_Sales'[Vehicle Type], 'Fact_Sales'[Model_name_upper])

VAR RankHighest = 
    RANKX(SummaryTable, [Net Profit Margin %], , DESC, Dense)

VAR RankLowest = 
    RANKX(SummaryTable, [Net Profit Margin %], , ASC, Dense)


VAR StarIcon = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='%23FFD700' d='M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z'/></svg>"

VAR BoldArrow = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='%23FF0000' d='M20 7H4l8 10z'/></svg>"

RETURN
IF(
    ISBLANK([Net Profit Margin %]), 
    BLANK(),
    SWITCH(TRUE(),
        RankHighest = 1, StarIcon,
        RankLowest = 1, BoldArrow,
        BLANK()
    )
)
```

</details>

### Dynamic Narrative / Insight Text

<details>
<summary><code>Text_Part_Highest</code></summary>

```dax
VAR SummaryTable = ALLSELECTED('Fact_Sales'[Vehicle Type], 'Fact_Sales'[Model_name_upper])
VAR Max_Margin = MAXX(SummaryTable, [Net Profit Margin %])
VAR MaxName = MAXX(FILTER(SummaryTable, [Net Profit Margin %] = Max_Margin), 'Fact_Sales'[Vehicle Type] & " " & 'Fact_Sales'[Model_name_upper])

RETURN
"Highest — " & MaxName & " (" & FORMAT(Max_Margin, "0.0%") & ")" & " |"
```

</details>

<details>
<summary><code>Text_Part_Lowest</code></summary>

```dax
VAR SummaryTable = ALLSELECTED('Fact_Sales'[Vehicle Type], 'Fact_Sales'[Model_name_upper])
VAR MinMargin = MINX(SummaryTable, [Net Profit Margin %])
VAR MinName = MAXX(FILTER(SummaryTable, [Net Profit Margin %] = MinMargin), 'Fact_Sales'[Vehicle Type] & " " & 'Fact_Sales'[Model_name_upper])

RETURN
" Lowest — " & MinName & " (" & FORMAT(MinMargin, "0.0%") & ")" & " | "
```

</details>

<details>
<summary><code>Text_Part_Gap</code></summary>

```dax
VAR SummaryTable = ALLSELECTED('Fact_Sales'[Vehicle Type], 'Fact_Sales'[Model_name_upper])


VAR MaxMargin = MAXX(SummaryTable, [Net Profit Margin %])
VAR MinMargin = MINX(SummaryTable, [Net Profit Margin %])


VAR Gap = (MaxMargin - MinMargin) * 100

RETURN
"EV margin gap vs Diesel: " & FORMAT(Gap, "0.0") & " percentage points — driven by higher battery & materials cost"
```

</details>

---

## Profit Waterfall Chart
*Table: `Waterfall_Data`*

### Other / Helper Measures

<details>
<summary><code>Waterfall_Value</code></summary>

```dax
VAR vCOGS = [Total_COGS]
VAR vOpex = [Total_Opex]

VAR Selection = SELECTEDVALUE(Waterfall_Data[Category])

RETURN
    SWITCH(
        Selection,
        "Revenue", [Total_Revenue_Numeric],
        
       
        "COGS", vCOGS,  
        "Opex", vOpex,
        
        "Gross Profit", [Gross_Profit],
        "Net Profit", [Net_Profit]
    )
```

</details>

<details>
<summary><code>Label_Position</code></summary>

```dax
-- এখানে তোর রেভিনিউ ভ্যালুর চেয়ে ২০-৩০% বেশি একটা ফিক্সড ভ্যালু দে
-- যাতে লেবেলগুলো বারের মাথায় না লেগে একটু উপরে থাকে
MAXX(ALL(Waterfall_Data), [Total_Revenue_Numeric]) * 1.2
```

</details>

<details>
<summary><code>Column_Color</code></summary>

```dax
VAR Selection = SELECTEDVALUE(Waterfall_Data[Category])
RETURN
    SWITCH(
        Selection,
        "Revenue", "#0078D4",
        "COGS", "#D13438",
        "Gross Profit", "#498205",
        "Opex", "#FF6B6B",
        "Net Profit", "#00b294",
        "#A19F9D"
    )
```

</details>

<details>
<summary><code>Label_Color_Simple</code></summary>

```dax
VAR Selection = SELECTEDVALUE(Waterfall_Data[Category])
RETURN
    IF(
        Selection = "COGS" || Selection = "Opex",
        "#D13438", 
        "#107C10"  
    )
```

</details>

<details>
<summary><code>MODEL_NAME_UPPER_MEASURE</code></summary>

```dax
UPPER(SELECTEDVALUE('Fact_Sales'[Model Name]))
```

</details>

---

## KPI 3 — Sustainability
*Table: `KPI_3_Measure`*

### Core KPI Measures (Sustainability)

<details>
<summary><code>EV_Units_Sold</code></summary>

```dax
CALCULATE(
    SUM('Fact_Sales'[Units Sold]), 
    'Dim_Vehicle'[Vehicle Type] = "Electric"
)
```

</details>

<details>
<summary><code>EV_Avg_Emission_Value</code></summary>

```dax
CALCULATE(
    AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 
    'Fact_Sales'[Vehicle Type] = "Electric"
)
```

</details>

<details>
<summary><code>EV_Avg_Emission_Display</code></summary>

```dax
FORMAT([EV_Avg_Emission_Value], "0.0") & " g/km"
```

</details>

<details>
<summary><code>Est_CO2_Avoided_Value</code></summary>

```dax
VAR FossilAvg = [FossilAvg_Dynamic]
VAR EVAvg = 
    CALCULATE(
        AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 
        'Fact_Sales'[Vehicle Type] = "Electric"
    )
VAR Units = [EV_Units_Sold]
VAR AnnualKM = 15000

RETURN
DIVIDE((FossilAvg - EVAvg) * Units * AnnualKM, 1000000, 0)
```

</details>

<details>
<summary><code>Est_CO2_Avoided_Display</code></summary>

```dax
VAR Result = [Est_CO2_Avoided_Value]
RETURN
FORMAT(Result, "#,##0") & " t"
```

</details>

<details>
<summary><code>EV_Fuel_Efficiency_Display</code></summary>

```dax
VAR AvgEfficiency = 
    CALCULATE(
        AVERAGE('Fact_Sales'[Fuel Efficiency]), 
        'Fact_Sales'[Vehicle Type] = "Electric"
    )
RETURN
FORMAT(AvgEfficiency, "0.0") & " km/kWh"
```

</details>

<details>
<summary><code>EV_Avg_Emission_Value_for_chart</code></summary>

```dax
CALCULATE(
    AVERAGE('Fact_Sales'[Carbon Emission (g per km)])
)
```

</details>

<details>
<summary><code>EV_Fuel_Efficiency_Numeric</code></summary>

```dax
AVERAGE('Fact_Sales'[Fuel Efficiency])
```

</details>

<details>
<summary><code>Vehicle_Unit_Only</code></summary>

```dax
VAR CurrentType = SELECTEDVALUE('Fact_Sales'[Vehicle Type])
RETURN 
IF(
    CurrentType = "Electric", 
    "km / kWh", 
    "km / litre"
)
```

</details>

<details>
<summary><code>Relative_Perf_Label</code></summary>

```dax
VAR CurrentType = SELECTEDVALUE('Fact_Sales'[Vehicle Type])

RETURN
    SWITCH(
        CurrentType,
        "Diesel", "Moderate",
        "Petrol", "Lowest",
        "Electric", "Zero tailpipe emissions",
        "Unknown"
    )
```

</details>

<details>
<summary><code>FossilAvg_Dynamic</code></summary>

```dax
AVERAGEX(
        FILTER(
            VALUES('Fact_Sales'[Vehicle Type]), 
            'Fact_Sales'[Vehicle Type] <> "Electric"
        ),
        CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]))
    )
```

</details>

<details>
<summary><code>CURRENT_EV_SHARE</code></summary>

```dax
VAR EVUnits = 
    CALCULATE(
        [Total Units Sold], 
        Dim_Vehicle[Vehicle Type] = "Electric"
    )
VAR TotalUnits = [Total Units Sold]

RETURN
DIVIDE(EVUnits, TotalUnits, 0)
```

</details>

<details>
<summary><code>Target</code></summary>

```dax
1
```

</details>

<details>
<summary><code>Scenario_Avg_Emission</code></summary>

```dax
VAR SelectedType = SELECTEDVALUE('Fact_Sales'[Vehicle_Type_Display])
RETURN
    SWITCH( SelectedType,
        "Current (EV)", 
            CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Electric"),
        "If Petrol", 
            CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Petrol"),
        "If Diesel", 
            CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Diesel")
    )
```

</details>

<details>
<summary><code>Scenario_Annual_CO2</code></summary>

```dax
VAR Total_Units = 
    CALCULATE(
        SUM('Fact_Sales'[Units Sold]), 
        ALLSELECTED('Fact_Sales'),
        'Fact_Sales'[Vehicle Type] = "Electric"
    )

VAR Avg_Em = [Scenario_Avg_Emission]

RETURN
    DIVIDE(Avg_Em * Total_Units * 15000, 1000000)
```

</details>

<details>
<summary><code>Scenario_Difference</code></summary>

```dax
VAR Current_Annual_CO2 = [Scenario_Annual_CO2]

VAR EV_Annual_CO2 = 
    CALCULATE(
        [Scenario_Annual_CO2], 
        'Fact_Sales'[Vehicle Type] = "Electric",
        ALLSELECTED('Fact_Sales')
    )

RETURN
    IF(
        SELECTEDVALUE('Fact_Sales'[Vehicle_Type_Display]) = "Current (EV)", 
        "Baseline", 
        VAR Diff = Current_Annual_CO2 - EV_Annual_CO2
        RETURN
            IF(Diff > 0, "+", "") & FORMAT(Diff, "#,##0") & " t"
    )
```

</details>

### Reference / Comparison Measures

<details>
<summary><code>Ref_EV_Units_Sold</code></summary>

```dax
VAR EVUnits = CALCULATE(SUM('Fact_Sales'[Units Sold]), 'Fact_Sales'[Vehicle Type] = "Electric")
VAR TotalUnits = SUM('Fact_Sales'[Units Sold])
VAR Share = DIVIDE(EVUnits, TotalUnits, 0)
RETURN
FORMAT(Share, "0.0%") & " of all sales"
```

</details>

<details>
<summary><code>Ref_EV_Emission</code></summary>

```dax
VAR FossilAvg = [FossilAvg_Dynamic]
VAR EVAvg = [EV_Avg_Emission_Value]
VAR Savings = DIVIDE(FossilAvg - EVAvg, FossilAvg, 0)
RETURN
"▼ " & FORMAT(Savings, "0%") & " lower than fossil avg (" & FORMAT(FossilAvg, "0.0") & " g/km)"
```

</details>

<details>
<summary><code>Ref_CO2_Avoided</code></summary>

```dax
"Tonnes per year (15k km avg)"
```

</details>

<details>
<summary><code>Ref_Fuel_Efficiency</code></summary>

```dax
VAR DieselAvg = 
    CALCULATE(
        AVERAGE('Fact_Sales'[Fuel Efficiency]), 
        'Fact_Sales'[Vehicle Type] = "Diesel"
    )

VAR PetrolAvg = 
    CALCULATE(
        AVERAGE('Fact_Sales'[Fuel Efficiency]), 
        'Fact_Sales'[Vehicle Type] = "Petrol"
    )

RETURN
"vs Diesel " & FORMAT(DieselAvg, "0.0") & " km/L, Petrol " & FORMAT(PetrolAvg, "0.0") & " km/L"
```

</details>

<details>
<summary><code>Relative_Perf_Icon</code></summary>

```dax
VAR CurrentType = SELECTEDVALUE('Fact_Sales'[Vehicle Type])

VAR CircleColor = 
    SWITCH(
        CurrentType,
        "Diesel", "%23FFD700",    
        "Petrol", "%23FF4C4C",   
        "Electric", "%232ECC71", 
        "%23CCCCCC"               
    )

RETURN
"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'>
    <circle cx='12' cy='12' r='7' fill='" & CircleColor & "'/>
</svg>"
```

</details>

### Dynamic Narrative / Insight Text

<details>
<summary><code>Value_1_EV_Emit</code></summary>

```dax
CALCULATE(
    AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 
    'Fact_Sales'[Vehicle Type] = "Electric"
)
```

</details>

<details>
<summary><code>Val_Reduction</code></summary>

```dax
VAR EV_Avg = CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Electric")
VAR Fossil_Base = [FossilAvg_Dynamic] 
RETURN FORMAT(Fossil_Base - EV_Avg, "0.0")
```

</details>

<details>
<summary><code>Val_Percent</code></summary>

```dax
VAR EV_Avg = CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Electric")
VAR Fossil_Base = [FossilAvg_Dynamic]
VAR Diff = Fossil_Base - EV_Avg
VAR Perc = DIVIDE(Diff, Fossil_Base, 0)
RETURN FORMAT(Perc, "0.0%")
```

</details>

<details>
<summary><code>Full_Narrative_Text</code></summary>

```dax
VAR EV_Val = FORMAT(
    CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Electric"), 
    "0.0"
)
VAR Red_Val = FORMAT(
    109.96 - CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Electric"), 
    "0.0"
)
VAR Perc_Val = FORMAT(
    DIVIDE(109.96 - CALCULATE(AVERAGE('Fact_Sales'[Carbon Emission (g per km)]), 'Fact_Sales'[Vehicle Type] = "Electric"), 109.96, 0), 
    "0.0%"
)

RETURN
"Electric vehicles emit " & EV_Val & " g/km, representing a reduction of " & 
Red_Val & " g/km (" & Perc_Val & ") compared to the fossil fuel average of 109.96 g/km."
```

</details>

<details>
<summary><code>EV_Gap_Numeric</code></summary>

```dax
VAR CurrentShare = [CURRENT_EV_SHARE] * 100
RETURN 100 - CurrentShare
```

</details>

### Conditional Formatting (Color Logic)

<details>
<summary><code>Scenario_Color_Logic</code></summary>

```dax
VAR SelectedType = SELECTEDVALUE('Fact_Sales'[Vehicle_Type_Display])
RETURN
    SWITCH( SelectedType,
        "Current (EV)", "#00b294",  
        "If Diesel", "#2B65EC",     
        "If Petrol", "#E5A017",     
        "#000000"                   
    )
```

</details>

<details>
<summary><code>Difference_Color_Logic</code></summary>

```dax
VAR Current_Scenario = SELECTEDVALUE('Fact_Sales'[Vehicle_Type_Display])
VAR Current_Annual_CO2 = [Scenario_Annual_CO2]
VAR EV_CO2 = 
    CALCULATE(
        [Scenario_Annual_CO2], 
        'Fact_Sales'[Vehicle Type] = "Electric",
        ALLSELECTED('Fact_Sales')
    )
RETURN
    SWITCH( TRUE(),
        Current_Scenario = "Current (EV)", "#006400", 
        Current_Annual_CO2 > EV_CO2, "#D22B2B",      
        Current_Annual_CO2 < EV_CO2, "#006400",      
        "#000000"                                    
    )
```

</details>

<details>
<summary><code>Vechicle_Color_Logic</code></summary>

```dax
VAR SelectedType = SELECTEDVALUE('Fact_Sales'[Vehicle Type])
RETURN
    SWITCH( SelectedType,
        "Electric", "#00b294",
        "Diesel", "#2B65EC",
        "Petrol", "#E5A017",
        "#000000"
    )
```

</details>

---

## All_SVG_Measure
*Table: `All_SVG_Measure`*

### Other / Helper Measures

<details>
<summary><code>Circle</code></summary>

```dax
VAR SalesRank = 
    RANKX(
        ALL(Dim_Region_Rep), 
        [Total_Revenue_Numeric], 
        , 
        DESC, 
        Dense
    )

VAR CircleColor = 
    SWITCH(SalesRank,
        1, "FFD100", 
        2, "BFC5C9", 
        3, "C3803F", 
        4, "F0F0F0", 
        5, "FCE4E4", 
        "FFFFFF"
    )

VAR TextColor = 
    SWITCH(SalesRank,
        5, "A52A2A", 
        "333333"
    )

VAR SVG_Circle = 
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'>" &
    "<circle cx='50' cy='50' r='48' fill='%23" & CircleColor & "' />" &
    "<text x='50' y='58' text-anchor='middle' alignment-baseline='middle' " & 
    "font-size='48' font-family='Segoe UI' font-weight='bold' fill='%23" & TextColor & "'>" & 
    SalesRank & "</text></svg>"

RETURN
IF(
    ISINSCOPE(Dim_Region_Rep[Sale Representative]) && SalesRank <= 5, 
    SVG_Circle, 
    BLANK()
)
```

</details>

---

## 🎨 Static HTML / SVG Design Measures

In addition to the logic measures above, the model uses **9 measures that return raw HTML or SVG markup**, rendered through the HTML Content custom visual. These power the title slide, KPI framework summary cards, company background panel, dataset overview card, and the numbered recommendation badges on the Conclusion page — a technique used to achieve fully custom, pixel-controlled layouts beyond native Power BI visual formatting.

- `EcoDrive_4K_Professional_Layout_Light` — All_SVG_Measure
- `Analytical_Framework_Final` — All_SVG_Measure
- `Company_Background_HTML` — All_SVG_Measure
- `Dataset_Overview_Compact` — All_SVG_Measure
- `Data_Note_HTML` — All_SVG_Measure
- `SVG_Rec_1` — Conclusion & Recommendations_svg_code
- `SVG_Rec_2` — Conclusion & Recommendations_svg_code
- `SVG_Rec_3` — Conclusion & Recommendations_svg_code
- `SVG_Rec_4` — Conclusion & Recommendations_svg_code
