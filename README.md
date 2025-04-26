# Food-Production-Impact-Analysis

- # Environmental Impact of Food Production Analysis

## Objective
Quantify the environmental footprint of 43 food products across CO2 emissions, water use, land use, and eutrophication to derive sustainable recommendations for policymakers, producers, and consumers.

## Tools
- **Power BI**: Integrated tool for data prep, analysis, and interactive visualization, streamlining CRISP-DM.
- **Git/GitHub**: Version control and storage for `.pbix`, visuals, and documentation.

## Dataset 
- **Source**: 
- **Rows**: 43 foods
- **Columns**: 23 (7 CO2 stages, eutrophying emissions, freshwater withdrawals, land use, scarcity-weighted water; per kg, 1000kcal, 100g protein)
- **Data Validity and Nulls**:
  - **Per kg Metrics (88% valid, 12% empty, 5 foods)**:
    - Columns: Eutrophying emissions, Freshwater withdrawals, Land use, Scarcity-weighted water.
    - Foods: Wheat & Rye (Bread), Maize (Meal), Barley (Beer), Tofu, Shrimps (farmed).
  - **Per 1000kcal Metrics (70-77% valid, 23-30% empty, 10-13 foods)**:
    - Columns: Eutrophying emissions (23%), Freshwater withdrawals (30%), GHG emissions (23%), Land use (23%), Scarcity-weighted water (30%).
    - Foods: Core 5 + Soymilk, Soybean Oil, Other Vegetables, Wine, Other Fruit, + Cassava, Peas, Other Pulses (varies).
  - **Per 100g protein Metrics (60-63% valid, 37-40% empty, 16-17 foods)**:
    - Columns: Eutrophying emissions (37%), Freshwater withdrawals (40%), GHG emissions (37%), Land use (37%), Scarcity-weighted water (40%).
    - Foods: All above + Cane Sugar, Beet Sugar, Palm Oil, Sunflower Oil, Rapeseed Oil, Olive Oil.
- **Observations**:
  - Nulls tied to nutritional scaling: "Per 100g protein" nulls in zero/low-protein foods (oils, sugars); "Per 1000kcal" in low-calorie items (vegetables, wine).
  - Core "per kg" data more complete; high-impact foods (e.g., beef, dairy) unaffected.
  - Shrimps (farmed) nulls across all scales—flagged for validation.

## Methodology
- **Initial Preprocessing**:
  - **Null Handling**:
    - **Per kg (12% empty)**:
      - *Action*: Replaced nulls with 0 for 5 foods.
      - *Why*: Small scope (12%)—low skew risk; grains (Bread, Meal, Beer) and Tofu likely have negligible unrecorded impact; Shrimps (farmed) conservatively set to 0, flagged for review.
      - *Formula*: Power Query UI step—Replace Values: null → 0 (applied to each column).
    - **Per 1000kcal (23-30% empty)**:
      - *Action*: Left nulls in 10-13 foods.
      - *Why*: Low-calorie foods (e.g., Other Vegetables, Wine)—0 underestimates impact (e.g., wine needs water despite low kcal); awaits categorization for filtering or imputation.
    - **Per 100g protein (37-40% empty)**:
      - *Action*: Left nulls in 16-17 foods.
      - *Why*: Zero/low-protein foods (e.g., Olive Oil, Cane Sugar)—0 misrepresents (implies no impact); excluded from protein analysis via Day 2 categorization.
  - **Cleaning**:
    - Ensured numeric columns (e.g., "GHG emissions per kg") set to Decimal Number.
    - Checked for duplicates—none found (43 distinct foods).
    - Flagged outliers (e.g., Scarcity-weighted water per kg max = 229889.8 L) for Day 2 review.

## Progress
- Repo initialized, dataset loaded into Power BI, nulls analyzed by food, "per kg" nulls replaced with 0, "per nutrient" nulls left for categorization, README updated with stats and rationale.

## Business Questions
1. Which food has the highest total CO2 footprint per kg?
2. How do CO2 emissions vary across production stages for the top 5 CO2-emitting foods?
3. Which foods consume the most freshwater withdrawals per kg?
4. How do plant-based vs. animal-based foods compare in land use per kg?
5. Which production stage contributes most to average CO2 emissions across all 43 foods?
6. Which foods have the highest eutrophying emissions per kg?
7. Is there a correlation between freshwater withdrawals and total CO2 emissions per kg?

- **Preprocessing**:
  - **Total CO2 Column**:
    - *Action*: Added as sum of 7 CO2 stages.
    - *Why*: Aggregates individual stages into a single metric to directly answer Q1 (highest CO2 food), Q2 (stage variation), Q5 (average stage contribution), and Q7 (correlation with water); simplifies analysis and visualization.
    - *Formula*: 
      ```
      Total CO2 = [Land use change] + [Animal Feed] + [Farm] + [Processing] + [Transport] + [Packaging] + [Retail]
      ```
  - **Category Column**:
    - *Action*: Categorized 43 foods into 3 groups:
      - Animal (10): Beef (beef herd), Beef (dairy herd), Lamb & Mutton, Pig Meat, Poultry Meat, Milk, Cheese, Eggs, Fish (farmed), Shrimps (farmed).
      - Plant (26): Wheat & Rye (Bread), Maize (Meal), Barley (Beer), Oatmeal, Rice, etc.
      - Non-Protein (7): Cane Sugar, Beet Sugar, Soybean Oil, Palm Oil, Sunflower Oil, Rapeseed Oil, Olive Oil.
    - *Why*: Enables Q4 (plant vs. animal land use comparison); addresses "per 100g protein" nulls by excluding Non-Protein foods (e.g., oils, sugars) from protein-based metrics; supports filtering for "per 1000kcal" nulls (e.g., low-calorie Plants) in Day 3 visuals.
    - *Formula*: 
      ```
      if Text.Contains([Food], "Beef") or Text.Contains([Food], "Lamb") or Text.Contains([Food], "Pig") or Text.Contains([Food], "Poultry") or Text.Contains([Food], "Milk") or Text.Contains([Food], "Cheese") or Text.Contains([Food], "Eggs") or Text.Contains([Food], "Fish") or Text.Contains([Food], "Shrimps") then "Animal"
      else if Text.Contains([Food], "Oil") or Text.Contains([Food], "Sugar") then "Non-Protein"
      else "Plant"
      ```
  - "Per 1000kcal" and "per 100g protein" nulls unchanged—handled via Category filters in analysis.
 
- ## Progress
- Repo created, raw dataset and Power BI file uploaded, "per kg" nulls replaced with 0, README updated.
- Defined 7 business questions, added Total CO2 and Category columns in Power BI, updated README with preprocessing details, rationale, and formulas, committed changes.

- ## Progress (Continued)
- **Analysis Completed**: Answered 7 business questions (Q1-Q7) using Power BI queries, filters, and DAX calculations on the preprocessed dataset (`Total CO2`, `Category` columns).
- **Visualization Built**: Created 8 Power BI pages in `FoodImpact.pbix`: Overview (KPI cards, Top Emitters, Plant vs. Animal land), Q1-Q7 (bar, pie, scatter charts), and Sustainability Summary (text).
- **Evaluation Finalized**: Derived key insights from Q1-Q7 and synthesized findings into a Sustainability Summary & Recommendations page, providing actionable steps for policymakers, producers, and consumers.
- **Deployment**: Committed final `Environment Impact of Food Analysis.pbix`, exported visuals to `/visuals` (9 PNGs), and updated README with full project documentation, including insights, visuals, and usage instructions.

## Analysis
To answer the 7 business questions, we used Power BI’s Query Editor and DAX to process the dataset, leveraging the preprocessed `Total CO2` (sum of 7 CO2 stages) and `Category` (Animal, Plant, Non-Protein) columns. Key steps:

- **Q1: Highest CO2 Footprint per kg**  
  - Sorted `Total CO2` descending, filtered to “per kg” metrics.  
  - Result: Beef (beef herd) = 59.60 kg CO₂/kg, ~300x nuts’ 0.20 kg.

- **Q2: CO2 Emissions by Stage for Top 5 Emitters**  
  - Filtered top 5 foods by `Total CO2` (Beef, Lamb, Cheese, etc.), extracted CO2 stages (Land use change, Farm, etc.).  
  - Result: Farm stage contributes ~27% for Beef, highest among stages.

- **Q3: Highest Freshwater Withdrawals per kg**  
  - Sorted `Freshwater withdrawals per kg` descending.  
  - Result: Cheese = 5,605 L/kg, ~16% of total water use.

- **Q4: Plant vs. Animal Land Use per kg**  
  - Grouped by `Category` (Animal, Plant), summed `Land use per kg`.  
  - Result: Animal foods = 93% of total land; Plants = 7%.

- **Q5: Stage with Highest Average CO2**  
  - Calculated average CO2 per stage across 43 foods using DAX
  - Result: Farm stage = 3.47 kg CO₂/kg, ~50x Retail’s 0.07 kg.

- **Q6: Highest Eutrophying Emissions per kg**  
  - Sorted `Eutrophying emissions per kg` descending.  
  - Result: Beef (dairy herd) = 365 g PO₄eq/kg, ~21% of total.

- **Q7: Correlation Between CO2 and Water Use**  
  - Used scatter chart and gauge visual `Total CO2` and `Freshwater withdrawals per kg`
  - Result: Moderate correlation = 0.33.

**Notes**:
- Nulls in “per 1000kcal” and “per 100g protein” metrics were filtered out using `Category` (excluded Non-Protein foods for protein analysis).
- Shrimps (farmed) nulls (set to 0 for “per kg”) had minimal impact, as focus was on high-impact foods (Beef, Cheese).

## Visualization
The analysis is visualized in `Environment Impact of Food Production Analysis.pbix` across 8 pages, each addressing a specific question or summary. Visuals are designed for clarity, using bar, pie, and scatter charts, with KPI cards for quick insights.

1. **Overview & Key Metrics** ![Overview   Key Metrics](https://github.com/user-attachments/assets/3e2da351-b32a-4cce-a997-1c6e2f97b8f7)
 
   - **Visuals**:  
     - KPI cards: Avg. CO₂ (3.47 kg/kg), Water (1,242 L/kg), Land (5.58 m²/kg), Eutrophication (29 g PO₄eq/kg).  
     - Multi-row Card: Top 5 CO₂ Emitters (Beef, Lamb, Cheese, etc.).  
     - Pie Chart: Plant (7%) vs. Animal (93%) land use.  
   - **Purpose**: Quick snapshot of dataset averages and key culprits.

2. **Q1: CO2 Emissions by Food** ![CO2 Emission by Food](https://github.com/user-attachments/assets/b6703bf1-7a3c-4131-a99f-04324f43d7ec)
 
   - **Visual**: Bar chart (x: Food, y: Total CO2 per kg).  
   - **Insight**: Beef (beef herd) = 59.60 kg CO₂/kg, ~300x nuts’ 0.20 kg.

3. **Q2: CO2 by Production Stage for Top 5 Food** ![CO2 Emission by Production Stage for top 5 food](https://github.com/user-attachments/assets/a5017e63-7f87-46a1-bab7-0b3f21c531cb)

   - **Visual**: Stacked bar chart (x: Top 5 Foods, y: CO2 by stage).  
   - **Insight**: Farm stage ~27% for Beef, highest contributor.

4. **Q3: Water Usage** (`visuals/Q3_Water.png`)  ![Water Usage](https://github.com/user-attachments/assets/ac49ea0b-acd4-467b-80eb-9d4e7a76fc94)

   - **Visual**: Bar chart (x: Food, y: Freshwater withdrawals per kg).  
   - **Insight**: Cheese = 5,605 L/kg, ~16% of total.

5. **Q4: Land Use - Plant vs. Animal** ![Land Use - Plant Vs Animal](https://github.com/user-attachments/assets/abea72fe-0166-48a7-8f61-8ae2b749a308)

   - **Visual**: Pie chart (Animal: 93%, Plant: 7%).  
   - **Insight**: Animal foods dominate land use.

6. **Q5: Average CO2 by Stage** ![CO2 by Production Stage](https://github.com/user-attachments/assets/e4771d7b-f285-4863-a488-4b58c9dc58ab)

   - **Visual**: Bar chart (x: Stage, y: Avg. CO2 per kg).  
   - **Insight**: Farm stage = 3.47 kg CO₂/kg, ~50x Retail.

7. **Q6: Eutrophying Emissions** ![Eutrophying Emissions](https://github.com/user-attachments/assets/fbecb8bc-3e7c-4749-9aba-b85e2be8eaad)
 
   - **Visual**: Bar chart (x: Food, y: Eutrophying emissions per kg).  
   - **Insight**: Beef (dairy herd) = 365 g PO₄eq/kg, ~21% of total.

8. **Q7: CO2 vs. Water Correlation** ![CO2 Vs Water Correlation](https://github.com/user-attachments/assets/458adec7-2152-48db-a167-155cc22527fd)

   - **Visual**: Scatter plot (x: Total CO2, y: Freshwater withdrawals).  
   - **Insight**: Moderate correlation = 0.33.

9. **Sustainability Summary & Recommendations** ![Sustainability Summary and Recommendation](https://github.com/user-attachments/assets/0b02a98b-7c4d-4f67-b149-ee15a0f56da1)
 
   - **Visual**: Text (Challenge, Insights, So What, What Now, Conclusion).  
   - **Content**: See below for full text.

## Evaluation
The analysis answered all 7 business questions, revealing critical environmental impacts and actionable solutions. Key findings and their implications:

- **Q1**: Beef’s 59.60 kg CO₂/kg (300x nuts) makes it the top target for emission cuts—shifting to nuts could reduce CO₂ by orders of magnitude.
- **Q2**: Farm stage’s ~27% CO₂ share for Beef highlights the need for methane capture and feed improvements.
- **Q3**: Cheese’s 5,605 L/kg water (16% of total) underscores irrigation upgrades as a priority.
- **Q4**: Animal foods’ 93% land use vs. Plants’ 7% shows Plant-based diets as a high-impact solution.
- **Q5**: Farm stage’s 3.47 kg CO₂/kg average (50x Retail) reinforces farming as the key intervention point.
- **Q6**: Beef (dairy herd)’s 365 g PO₄eq/kg (21% of pollution) links high-CO₂ foods to water pollution.
- **Q7**: CO₂-water correlation (0.33) confirms that high-impact foods (e.g., Beef, Cheese) harm across multiple metrics.

### Sustainability Summary & Recommendations
The final page synthesizes findings into a concise, actionable narrative for policymakers, producers, and consumers.

**The Challenge**  
Food choices shape our planet. Beef and Cheese use massive land, water, and energy, fueling climate change. Nuts and Tofu use far less. We studied 43 foods to find what harms most and how to fix it.

**Key Insights**  
- Beef emits 59.60 kg CO₂/kg—300x Nuts.
- Animal foods take 93% of land; Plants 7%.
- Cheese uses 5,605 L/kg water—tops all.
- Farm stage drives ~30% of CO₂, led by Beef.

**So What**  
A few foods cause most harm. Targeting Beef, Cheese, and Farms can cut emissions, land, and water fast—big wins for planet and business.

**What Now**  
1. **Pick Plants**: Choose Nuts, Tofu—cuts 80% impact.  
2. **Greener Farms**: Cut Beef’s CO₂, Cheese’s water.  
3. **Label CO₂**: Tag foods’ impact—guide smart buys.

**Final Word**  
Smarter foods save planet and billions. Start with Plants—act now!

**So What? Why?**  
- **Why 59.60?**: Beef’s CO₂ drives climate crisis—must target.
- **Why 93%?**: Animal foods waste land—Plants fix it.
- **Why Act?**: 80% cut saves planet, profits—urgent.

## Deployment
The project is deployed as a shareable, documented repository for stakeholders:

- **Power BI File**: `FoodImpact.pbix` contains 8 interactive pages (Overview, Q1-Q7, Sustainability Summary).
- **Visuals**: 9 PNGs in `/visuals` capture each page for static viewing:
  - `Overview_Metrics.png`
  - `Q1_CO2.png`, `Q2_Stages.png`, `Q3_Water.png`, `Q4_Land.png`, `Q5_Avg_CO2.png`, `Q6_Eutrophying.png`, `Q7_Correlation.png`
  - `Sustainability_Summary.png`
- **README**: This file details objective, methodology, insights, visuals, and usage.

**Usage Instructions**:
1. **Interactive Exploration**: Open `FoodImpact.pbix` in Power BI Desktop to filter, drill down, and explore visuals.
2. **Static Review**: View `/visuals` PNGs for quick insights or presentations.
3. **Recommendations**: Use Sustainability Summary for policy, procurement, or consumer education strategies.

## Future Steps
- **Data Expansion**: Include more foods (e.g., processed items) or regional data for localized insights.
- **Cost-Benefit Analysis**: Quantify ROI of recommendations (e.g., CO₂ labeling costs vs. ESG gains).
- **Real-Time Monitoring**: Build dashboards for supply chain tracking using live data.
- **Validation**: Revisit Shrimps (farmed) nulls and scarcity-weighted water outliers (e.g., 229889.8 L) with updated data.

## Contact
For questions, reach out to [manoelvuu@gmail.com].

---
**Built with**: Power BI  
**Date**: April 26, 2025  
**License**: MIT
