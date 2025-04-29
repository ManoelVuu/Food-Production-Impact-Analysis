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
- **Result**: Beef (beef herd) = 59.60 kg CO₂/kg, nearly 300× that of Nuts (0.20 kg CO₂/kg).

**Q2: How do CO₂ emissions vary across production stages for the top 5 CO₂-emitting foods?**
- **Method**: Filtered top 5 foods (Beef, Lamb, Cheese, etc.), analyzed CO₂ stages (Land use change, Farm, etc.).
- **Result**: Farm stage contributes 27% of Beef’s total emissions — the highest among all stages.

**Q3: Which foods consume the most freshwater withdrawals per kg?**
- **Method**: Sorted Freshwater Withdrawals per kg descending.
- **Result**: Cheese = 5,605 L/kg, accounting for 16% of total freshwater usage.

**Q4: How do plant-based vs. animal-based foods compare in land use per kg?**
- **Method**: Grouped by Category, summed Land Use per kg.
- **Result**: Animal foods = 93% of total land use; Plants = 7%.

**Q5: Which production stage contributes most to average CO₂ emissions across all foods?**
- **Method**: Used DAX to calculate average Total CO₂ per stage across 43 foods.
- **Result**: Farm stage = 3.47 kg CO₂/kg, approximately 50× higher than Retail’s 0.07 kg.

**Q6: Which foods have the highest eutrophying emissions per kg?**
- **Method**: Sorted Eutrophying emissions per kg descending.
- **Result**: Beef (dairy herd) = 365 g PO₄eq/kg, representing 21% of total eutrophication impact.

**Q7: Is there a correlation between freshwater withdrawals and total CO₂ emissions per kg?**
- **Method**: Created scatter chart and gauge visuals between Total CO₂ and Freshwater Withdrawals per kg.
- **Result**: Moderate positive correlation = 0.33.

**Notes**:
- Filtered out nulls in "per 1000kcal" and "per 100g protein" analyses by using the Category field (excluded Non-Protein foods for protein-related metrics).
- Shrimps (farmed) nulls (set to 0 for "per kg") had minimal impact, as analysis focused on major contributors (e.g., Beef, Cheese).

# Visualization & Key Insights
The analysis is visualized in `Environment Impact of Food Production Analysis.pbix` across 8 pages, each addressing a specific question or summary. Visuals are designed for clarity, including bar, pie, and scatter charts, with KPI cards for quick insights.

1. **Overview & Key Metrics** ![Overview   Key Metrics](https://github.com/user-attachments/assets/3e2da351-b32a-4cce-a997-1c6e2f97b8f7)
 
   - **Visuals**:  
     - KPI cards: Avg. CO₂ (3.47 kg/kg), Water (1,242 L/kg), Land (5.58 m²/kg), Eutrophication (29 g PO₄eq/kg).  
     - Multi-row Card: Top 5 CO₂ Emitters (Beef, Lamb, Cheese, etc.).  
     - Pie Chart: Plant (7%) vs. Animal (93%) land use.  
   - **Purpose**: Quick snapshot of dataset averages and key culprits.

2. **Q1: Which food has the highest total CO2 footprint per kg?** ![CO2 Emission per kg by Food](https://github.com/user-attachments/assets/df0a2cd0-a918-4d93-a59e-1e6751b85fa3)

   - **Visual**: Bar chart (x: Food, y: Total CO2 per kg).  
   - **Insight**: Beef (beef herd) = 59.60 kg CO₂/kg, ~300x nuts’ 0.20 kg.

3. **Q2: How do CO2 emissions vary across production stages for the top 5 CO2-emitting foods?** ![CO2 Emission by Production Stage for Top 5 Foods](https://github.com/user-attachments/assets/f7ce4d64-6869-47db-8a7c-5b442e254885)

   - **Visual**: Stacked bar chart (x: Top 5 Foods, y: CO2 by stage).  
   - **Insight**: Farm stage ~27% for Beef, highest contributor.

4. **Q3: Which foods consume the most freshwater withdrawals per kg?** ![Freshwater Withdrawal per kg by Food](https://github.com/user-attachments/assets/e10e060f-0cb2-4e57-845a-a5c4a373943c)

   - **Visual**: Bar chart (x: Food, y: Freshwater withdrawals per kg).  
   - **Insight**: Cheese = 5,605 L/kg, ~16% of total.

5. **Q4: How do plant-based vs. animal-based foods compare in land use per kg?** ![Average kg per Land Use - Animal Vs Plant](https://github.com/user-attachments/assets/1f25b3f6-044b-478f-9c01-39b76d9a1f81)

   - **Visual**: Pie chart (Animal: 93%, Plant: 7%).  
   - **Insight**: Animal foods dominate land use.

6. **Q5: Which production stage contributes most to average CO2 emissions across all 43 foods?** ![CO2 by Production Stage](https://github.com/user-attachments/assets/ca1e365b-d254-496b-98cf-7b4aefd53a29)

   - **Visual**: Bar chart (x: Stage, y: Avg. CO2 per kg).  
   - **Insight**: Farm stage = 3.47 kg CO₂/kg, ~50x Retail.

7. **Q6: Which foods have the highest eutrophying emissions per kg?** ![Eutrophying Emissions](https://github.com/user-attachments/assets/0370f257-f143-48bd-b530-914d405586f4)

   - **Visual**: Bar chart (x: Food, y: Eutrophying emissions per kg).  
   - **Insight**: Beef (dairy herd) = 365 g PO₄eq/kg, ~21% of total.

8. **Q7: Is there a correlation between freshwater withdrawals and total CO2 emissions per kg?** ![CO2 Vs Water Correlation](https://github.com/user-attachments/assets/20a0a3ff-b1bc-441f-9cdf-e9779ea2b9fd)

   - **Visual**: Scatter plot and Gauge Visual (x: Total CO2, y: Freshwater withdrawals).  
   - **Insight**: Moderate correlation = 0.33.

9. **Sustainability Summary & Recommendations** ![Sustainability Summary and Recommendation](https://github.com/user-attachments/assets/0b02a98b-7c4d-4f67-b149-ee15a0f56da1)
 
   - **Visual**: Text (Challenge, Insights, So What, What Now, Conclusion).  
   - **Content**: Full text provided below.

## Sustainability Summary & Recommendations
The final page synthesizes findings into a concise, actionable narrative for policymakers, producers, and consumers.

**The Challenge (Why it matters)**  
Our food choices are reshaping the planet. Some foods, especially beef and cheese consume excessive land, water, and energy, accelerating climate change and environmental degradation. Others, like nuts and tofu, have a much smaller footprint. We analyzed 43 common foods to identify the biggest environmental contributors and spotlight smarter alternatives.

**Key Insights (What we found)**  
- Beef leads emissions: Beef (beef herd) emits 59.60 kg CO₂ per kg, 300x more than nuts (0.20 kg CO₂/kg).
- Animal products dominate impact: Beef, lamb, and cheese account for 93% of land use, 80% of CO₂ emissions, and 80% of pollution across all foods.
- Cheese drains water: Cheese uses 5,605 liters of water per kg, about 16% of total water use in our analysis.
- Farm stage is the biggest CO₂ source: Farming contributes, 27–50% of food-related CO₂, especially for animal products.
- Impacts are linked: Food’s high in CO₂ emissions tend to also use more water (correlation: 0.33), amplifying their environmental cost.

**So What (Why this matters to you)**  
- A small group of animal-based foods is responsible for a majority of environmental damage.
- Addressing just a few high-impact foods or stages can yield significant sustainability gains across land, water, emissions, and pollution.
- Data-backed decisions around food sourcing, production methods, and consumer engagement can help companies hit ESG goals faster with real-world impact.
  
**What Now (Recommendations)**  
**1. Prioritize Low-Impact Foods**
- Promote plant-based options like nuts, legumes, and grains to reduce emissions and land use by up to 80% per kg.
- Support internal and public education campaigns on high-impact foods.

**2. Transform Agricultural and Livestock Practices**
- Focus on carbon dioxide (CO2) reduction strategies (e.g., improve feeding through dietary changes) to lower Farm stage emissions.
- Improve irrigation for high-water-use foods like cheese and rice.

**3. Support Food Innovation**
- Invest in sustainable meat alternatives (plant-based, cultured meat).
- Adopt precision agriculture to optimize inputs and cut waste.

**4. Lead with Policy & Procurement**
- Implement CO₂ labeling or eco scorecards for food products.
- Review procurement policies to favor low-impact suppliers and menu items.

**Conclusion**  
A handful of foods cause the majority of environmental harm, and now we have the data to act. By targeting the most impactful items and improving key production stages, we can reduce emissions, conserve resources, and drive real progress toward a more sustainable food system.

Smarter food choices aren’t just better for the planet, they’re better for business.

# Evaluation
The analysis answered seven core business questions and synthesized findings in a final **Sustainability Summary** page. Key insights and their implications include:

- **Q1:** Beef produces **59.60 kg CO₂/kg**, nearly **300 times** more than nuts, making it the top target for emission cuts. Shifting to plant-based options like nuts can drastically reduce CO₂ emissions.
- **Q2:** The **farm stage accounts for 27%** of beef’s CO₂ emissions, highlighting the importance of targeted carbon reduction strategies at this stage.
- **Q3:** Cheese uses **5,605 L/kg of water**, contributing **16%** of total water use. This indicates that upgrading irrigation systems should be a priority.
- **Q4:** Animal-based foods consume **93%** of total land use, compared to **7%** for plant-based foods. Promoting plant-based diets offers a high-impact solution.
- **Q5:** The farm stage emits an average of **3.47 kg CO₂/kg**, nearly **50 times** more than the retail stage. This reinforces farming as the most effective intervention point.
- **Q6:** Beef from dairy herds produces **365 g PO₄eq/kg**, accounting for **21%** of eutrophying pollution, linking high-CO₂ foods with water pollution concerns.
- **Q7:** A **CO₂-water use correlation of 0.33** shows that high-impact foods (e.g., beef, cheese) negatively affect multiple environmental metrics simultaneously.

# Deployment
The project is deployed as a shareable, well-documented repository designed for stakeholder use.

## Components:

- **Power BI File:** `Environment Impact of Food Production Analysis.pbix` contains eight interactive pages (Overview, Q1–Q7, Sustainability Summary).
- **Visuals:** Nine static PNGs in the `/visuals` folder for quick reference and presentation use:
  - `Overview_Metrics.png`
  - `Q1_CO2.png`, `Q2_Stages.png`, `Q3_Water.png`, `Q4_Land.png`, `Q5_Avg_CO2.png`, `Q6_Eutrophying.png`, `Q7_Correlation.png`
  - `Sustainability_Summary.png`

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

## Tools Used

- **Power BI:** Unified tool for data preparation, analysis, and interactive visualization, streamlining the CRISP-DM methodology.
- **Git/GitHub:** Version control and repository hosting for `.pbix` files, visuals, and supporting documentation.

**Date:** April 11, 2025
