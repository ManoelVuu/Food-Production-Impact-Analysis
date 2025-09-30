# Food-Production-Impact-Analysis

## Table of Contents

- [Business Understanding](#business-understanding)
- [Data Understanding](#data-understanding)
- [Data Preparation](#data-preparation)
- [Analysis](#analysis)
- [Visualisation & Key Insights](#visualisation-&-key-insights)
- [Evaluation](#evaluation)
- [Deployment](#deployment)

# Business Understanding

## Objective
Quantify the environmental footprint of 43 food products across CO2 emissions, water use, land use, and eutrophication to derive sustainable recommendations for policymakers, producers, and consumers.

## Business Questions
1. Which food has the highest total CO2 footprint per kg?
2. How do CO2 emissions vary across production stages for the top 5 CO2-emitting foods?
3. Which foods consume the most freshwater withdrawals per kg?
4. How do plant-based vs. animal-based foods compare in land use per kg?
5. Which production stage contributes most to average CO2 emissions across all 43 foods?
6. Which foods have the highest eutrophying emissions per kg?
7. Is there a correlation between freshwater withdrawals and total CO2 emissions per kg?

# Data Understanding
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

# Data preparation
- **Preprocessing**
- **Null Handling**:
- **Per kg (12% empty)**:
  - **Action**: Replaced nulls with 0 for 5 foods.
  - **Why**: Small scope (12%)—low skew risk; grains (Bread, Meal, Beer) and Tofu likely have negligible unrecorded impact; Shrimps (farmed) conservatively set to 0, flagged for review.
  - **Formula**: Power Query UI step—Replace Values: null → 0 (applied to each column).
- **Per 1000kcal (23-30% empty)**:
  - **Action**: Left nulls in 10-13 foods.
  - **Why**: Low-calorie foods (e.g., Other Vegetables, Wine)—0 underestimates impact (e.g., wine needs water despite low kcal); awaits categorization for filtering or imputation.
- **Per 100g protein (37-40% empty)**:
  - **Action**: Left nulls in 16-17 foods.
  - **Why**: Zero/low-protein foods (e.g., Olive Oil, Cane Sugar)—0 misrepresents (implies no impact); excluded from protein analysis.
- **Data Cleaning**:
  - Ensured numeric columns (e.g., "GHG emissions per kg") set to Decimal Number.
  - Checked for duplicates—none found (43 distinct foods).
  - Flagged outliers (e.g., Scarcity-weighted water per kg max = 229889.8 L).

- **Feauture Engineering**:
- **Feauture: Total CO2 Column**
  - **Action**: Added as sum of 7 CO2 stages.
  - **Why**: Aggregates individual stages into a single metric to directly answer Q1 (highest CO2 food), Q2 (stage variation), Q5 (average stage contribution), and Q7 (correlation with water); simplifies analysis and visualization.
  - **Formula**: 
      ```
      Total CO2 = [Land use change] + [Animal Feed] + [Farm] + [Processing] + [Transport] + [Packaging] + [Retail]
      ```
- **Feauture: Category Column**
  - **Action**: Categorized 43 foods into 3 groups:
  - Animal (10): Beef (beef herd), Beef (dairy herd), Lamb & Mutton, Pig Meat, Poultry Meat, Milk, Cheese, Eggs, Fish (farmed), Shrimps (farmed).
  - Plant (26): Wheat & Rye (Bread), Maize (Meal), Barley (Beer), Oatmeal, Rice, etc.
  - Non-Protein (7): Cane Sugar, Beet Sugar, Soybean Oil, Palm Oil, Sunflower Oil, Rapeseed Oil, Olive Oil.
  - **Why**: Enables Q4 (plant vs. animal land use comparison); addresses "per 100g protein" nulls by excluding Non-Protein foods (e.g., oils, sugars) from protein-based metrics; supports filtering for "per 1000kcal" nulls (e.g., low-calorie Plants).
  - **Formula**: 
      ```
      if Text.Contains([Food], "Beef") or Text.Contains([Food], "Lamb") or Text.Contains([Food], "Pig") or Text.Contains([Food], "Poultry") or Text.Contains([Food], "Milk") or Text.Contains([Food], "Cheese") or Text.Contains([Food], "Eggs") or Text.Contains([Food], "Fish") or Text.Contains([Food], "Shrimps") then "Animal"
      else if Text.Contains([Food], "Oil") or Text.Contains([Food], "Sugar") then "Non-Protein"
      else "Plant"
      ```
- "Per 1000kcal" and "per 100g protein" nulls unchanged—handled via Category filters in analysis.

# Analysis
To answer the 7 business questions, I used Power BI’s Query Editor and DAX, leveraging the preprocessed Total CO₂ and Category (Animal, Plant, Non-Protein) columns.

**Q1: Which food has the highest total CO₂ footprint per kg?**
- **Method**: Sorted Total CO₂ descending, filtered to "per kg" metrics.
  
**Q2: How do CO₂ emissions vary across production stages for the top 5 CO₂-emitting foods?**
- **Method**: Filtered top 5 foods (Beef, Lamb, Cheese, etc.), analyzed CO₂ stages (Land use change, Farm, etc.).
  
**Q3: Which foods consume the most freshwater withdrawals per kg?**
- **Method**: Sorted Freshwater Withdrawals per kg descending.

**Q4: How do plant-based vs. animal-based foods compare in land use per kg?**
- **Method**: Grouped by Category, summed Land Use per kg.

**Q5: Which production stage contributes most to average CO₂ emissions across all foods?**
- **Method**: Used DAX to calculate average Total CO₂ per stage across 43 foods.
  
**Q6: Which foods have the highest eutrophying emissions per kg?**
- **Method**: Sorted Eutrophying emissions per kg descending.
  
**Q7: Is there a correlation between freshwater withdrawals and total CO₂ emissions per kg?**
- **Method**: Created scatter chart and gauge visuals between Total CO₂ and Freshwater Withdrawals per kg.

**Notes**:
- Filtered out nulls in "per 1000kcal" and "per 100g protein" analyses by using the Category field (excluded Non-Protein foods for protein-related metrics).
- Shrimps (farmed) nulls (set to 0 for "per kg") had minimal impact, as analysis focused on major contributors (e.g., Beef, Cheese).

# Visualization & Key Insights
The analysis is visualized in `Environmental Impact of Food Production Analysis.pbix` across 7 pages, each addressing a specific question or summary. Visuals are designed for clarity, including bar, pie, and scatter charts, with KPI cards for quick insights.

1. **Overview & Key Metrics** <img width="1296" height="714" alt="Overview   Key Metrics" src="https://github.com/user-attachments/assets/718cf241-485e-4351-ad7f-bf4ed76a94f9" />

   - **Insight**: On average, foods emit 5.97 kg CO₂, use 824 L of water, 25.9 m² of land, and cause 40.8 g eutrophication per kg underscoring their heavy environmental toll. Animal-based foods dominate with 72.5% of total CO₂, more than double plants. Strikingly, just five foods; beef, lamb, cheese, dairy beef, and dark chocolate drive nearly the entire emissions footprint, showing that tackling a handful of items can unlock major sustainability gains.
  
2. **Q1: Which food has the highest total CO2 footprint per kg?** <img width="600" height="645" alt="CO2 emission" src="https://github.com/user-attachments/assets/52c9346b-bf47-4755-95b1-e8c0d03cdccc" />

   - **Insight**: Animal-based foods produce over 2.5× more CO₂ per kg than plant-based, making them the biggest driver of emissions; reducing animal intake offers the largest sustainability gains.
     
3. **Q2: How do CO2 emissions vary across production stages for the top 5 CO2-emitting foods?** <img width="698" height="641" alt="Top 5 foods production stage emission" src="https://github.com/user-attachments/assets/e22deac4-a053-45fb-8f21-dbcb64fc3d27" />
  
   - **Insight**: Across the top 5 CO2-emitting foods, farming is the largest contributor (e.g., 39.4 kg CO2/kg for beef), except for dark chocolate where land-use change dominates; showing that tackling farm emissions in livestock and deforestation in cocoa are key levers for reduction.

4. **Q3: Which foods consume the most freshwater withdrawals per kg?** <img width="796" height="642" alt="Water use" src="https://github.com/user-attachments/assets/e1082f71-b0b3-4734-b3e9-da3ef074bcd4" />
  
   - **Insight**:Animal-based foods consume the most water (18.9k L/kg), nearly double plants (11.9k L/kg) and 4× non-protein (4.6k L/kg); showing that reducing animal products or improving irrigation in livestock farming offers the biggest water savings.

5. **Q4: How do plant-based vs. animal-based foods compare in land use per kg?** <img width="502" height="643" alt="Land use" src="https://github.com/user-attachments/assets/7313dd1f-c207-42ce-b0d4-b7822e87439f" />
 
   - **Insight**: Animal-based foods dominate land use (84.6%) compared to plants (15.4%), showing that shifting diets toward plant-based options can significantly free up land for forests, biodiversity, or alternative agriculture.

6. **Q5: Which production stage contributes most to average CO2 emissions across all 43 foods?** <img width="1293" height="639" alt="Production stage emission" src="https://github.com/user-attachments/assets/52f298e3-ee9f-4c66-84ad-c0aa2767e39d" />
  
   - **Insight**: The farm stage is the biggest driver of CO2 emissions (3.47 kg CO2/kg), over 25× higher than retail (0.07 kg CO2/kg), showing that improving farming practices offers the clearest path to large-scale emission reductions across foods.
     
7. **Q6: Which foods have the highest eutrophying emissions per kg?** <img width="1312" height="639" alt="Eutrophication emission" src="https://github.com/user-attachments/assets/dc734246-3c75-4a00-80e2-bf8e9ec9dde8" />
  
   - **Insight**: Animal-based foods generate the highest eutrophying emissions (1,255 g/kg), over 3× higher than plants (347 g/kg), making them the main source of water pollution; shifting diets toward plant-based foods can significantly cut nutrient runoff.

8. **Q7: Is there a correlation between freshwater withdrawals and total CO2 emissions per kg?** <img width="1299" height="646" alt="Correlation analysis" src="https://github.com/user-attachments/assets/d9944bf0-c304-4606-82ee-fb3d0c8cae57" />
  
   - **Insight**: CO2 emissions and freshwater use show a moderate correlation (r = 0.33), meaning high-emission foods also tend to overuse water; targeting these foods offers a double win for climate and water sustainability.

9. **Sustainability Summary & Recommendations** <img width="1289" height="730" alt="Sustaintainability summary" src="https://github.com/user-attachments/assets/3154820e-cf8f-4cc8-843e-fe91c72c436a" />
 
   - **Content**: Full text provided below.

## Sustainability Summary & Recommendations
The final page synthesizes findings into a concise, actionable narrative for policymakers, producers, and consumers.

**The Challenge (Why it matters)**  
Our food choices are reshaping the planet. Food production is one of the largest drivers of environmental change. Across 43 foods analyzed, the Animal-based category consistently dominates CO₂ emissions, land use, water consumption, and pollution with beef, cheese, and lamb standing out as major contributors. In contrast, plant-based foods like nuts and tofu leave a much smaller footprint. Understanding these category-level differences highlights where action can deliver the biggest sustainability gains.

**Key Insights (What we found)**  
- Animal-based foods dominate impact; they produce over 2.5× more CO₂ than plant-based, use 85% of land, consume the most water, and generate the highest pollution.
- Farming is the biggest hotspot; the farm stage drives the majority of CO₂ emissions across foods, making it the clearest target for reductions.
- Category trade-offs matter; while dark chocolate’s emissions are land-use driven, most livestock impact stem from farming, highlighting different levers (deforestation vs. feed efficiency).
- Environmental impacts are linked; foods high in CO₂ also tend to overuse water (moderate correlation, r = 0.33), offering “double-win” opportunities by targeting the same foods.

**So What (Why this matters to you)**  
- A small group of animal-based foods drives the majority of environmental impact across CO₂, land, water, and pollution.
- Targeting just these high-impact foods and farming practices can unlock outsized sustainability gains.
- Using data-driven choices in sourcing, production, and consumer engagement helps organizations accelerate ESG progress while creating real-world environmental benefits.
  
**What Now (Recommendations)**  
**1. Shift Toward Low-Impact Foods**
- Promote plant-based options (nuts, legumes, grains) to cut emissions and land use by up to 80% per kg.
- Run education campaigns to raise awareness of high-impact animal-based foods.

**2. Transform Farming and Livestock Practices**
- Focus on farm-stage CO₂ reduction (e.g., dietary changes for livestock, methane capture).
- Improve irrigation efficiency for water-intensive foods like cheese and rice.

**3. Support Food System Innovation**
- Invest in sustainable alternatives (plant-based proteins, cultured meat).
- Adopt precision agriculture to optimize inputs and reduce waste.

**4. Lead with Policy & Procurement**
- Introduce CO₂/eco labeling to inform consumers and drive demand shifts.
- Update procurement policies to prioritize low-impact suppliers and menu items.

**Conclusion**  
A small set of foods and production stages cause most environmental harm. By targeting animal-based products and improving farm practices, businesses can cut emissions, conserve water and land, and reduce pollution. Smarter food choices aren’t just better for the planet, they also deliver competitive advantage, consumer trust, and ESG progress.

# Evaluation
The analysis answered seven core business questions and synthesized findings in a final **Sustainability Summary** page. Key insights and their implications are shared above.

# Deployment
The project is deployed as a shareable, well-documented repository designed for stakeholder use.

## Components:

- **Power BI File:** `Environment Impact of Food Production Analysis.pbix` contains seven interactive pages (Overview, Q1–Q7, Sustainability Summary).
- **Visuals:** Nine static PNGs in the `/visuals` folder for quick reference and presentation use.
 
## Usage Instructions:

- **Interactive Exploration:** Open the `.pbix` file in Power BI Desktop to filter, drill down, and explore data insights.
- **Static Review:** Use the PNGs in the `/visuals` folder for quick insights or inclusion in presentations.
- **Recommendations:** Refer to the **Sustainability Summary** page for policy formulation, procurement guidance, or consumer education initiatives.

## Future Steps

- **Data Expansion:** Integrate more food categories (e.g., processed foods) or regional breakdowns to enhance localized insight.
- **Cost-Benefit Analysis:** Estimate the return on investment (ROI) for proposed interventions (e.g., CO₂ labeling costs vs. ESG benefits).
- **Real-Time Monitoring:** Develop dynamic dashboards to track supply chain sustainability using live data.
- **Validation:** Investigate missing values for farmed shrimp and address outliers in scarcity-weighted water usage (e.g., **229,889.8 L/kg**) with updated or verified data.

## Contact

For inquiries or collaboration, please reach out to: [manoelvuu@gmail.com](mailto:manoelvuu@gmail.com)
- **Medium**: Read the full story at [How Your Plate Shape the Planet](https://medium.com/@manoelvuu/how-your-plate-shapes-the-planet-a-data-dive-into-foods-hidden-costs-e2749444a4fc)

## Tools Used

- **Power BI:** Unified tool for data preparation, analysis, and interactive visualization, streamlining the CRISP-DM methodology.
- **Git/GitHub:** Version control and repository hosting for `.pbix` files, visuals, and supporting documentation.

**Date:** April 11, 2025

**Updated:** September 30, 2025
