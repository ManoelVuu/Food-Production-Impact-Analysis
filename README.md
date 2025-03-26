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
- **Day 1**: Repo initialized, dataset loaded into Power BI, nulls analyzed by food, "per kg" nulls replaced with 0, "per nutrient" nulls left for categorization, README updated with stats and rationale.

## Business Questions
1. Which food has the highest total CO2 footprint per kg?
2. How do CO2 emissions vary across production stages for top-emitting foods?
3. Which foods consume the most freshwater per kg?
4. How do plant-based vs. animal-based foods compare in land use per kg?
5. Which production stage contributes most to average CO2 emissions across all foods?
6. Which foods have the highest eutrophying emissions per kg?
7. Is there a correlation between freshwater withdrawals and CO2 emissions per kg?
