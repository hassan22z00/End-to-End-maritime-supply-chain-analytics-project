# End to End maritime supply chain analytics project
An end-to-end data analytics and engineering project simulating the real-world operational and financial challenges faced by global shipping leaders like CMA CGM. This project models global seaport congestion, calculates delay-associated risks, and predicts vessel dwell times using a Modern Data Stack.

1. Business Problem & Project Objectives

Global maritime supply chains are highly vulnerable to geopolitical, weather, and operational disruptions. When container vessels face unexpected terminal bottlenecks, shipping lines suffer massive financial penalties (demurrage, detention, and fuel waste).
This project addresses these critical challenges by answering three key business questions for executive stakeholders:
1. Financial Exposure: What is the estimated financial loss caused by current terminal delays across our shipping lines?
2. Operational Bottlenecks: Which global maritime zones (zone_code) and specific seaports are experiencing critical congestion?
3. Proactive Planning: Can we leverage daily operational metrics and historical lag indicators to predict terminal dwell times before our vessels drop anchor?

2. Tech Stack & Architecture

The data pipeline is designed using an In-Database Processing architecture to enforce a strict separation of concerns, decoupling heavy transformations from the visualization and predictive layers.
[Raw Data: IMF PortWatch & Upply] 
       ➔ [PostgreSQL Database] 
       ➔ [dbt Transformation Layer] 
       ➔ [Python Machine Learning] 
       ➔ [Power BI Executive Dashboard]
• Data Engineering & Governance: dbt (Data Build Tool) & SQL
• Data Warehouse / Relational Database: PostgreSQL
• Predictive Analytics: Python (Pandas, Scikit-Learn, SQLAlchemy) — Restricted strictly to the predictive modeling stage.
• Business Intelligence: Power BI / Advanced Excel

🔄 3. Data Transformation & Modeling (dbt Layer)

We ingested two distinct real-world public data sources: IMF PortWatch Daily Ports Data (vessel activity and dwell metrics) and Upply Global Seaport Open Data (standardized UN LOCODE registries and precise geographical coordinates).

A. Staging Layer (models/staging/)

Raw data is cast, normalized, and modularized into lightweight, atomic views:
• stg_ports: Cleanses seaport dimensional data, enforces proper string casing (INITCAP), and standardizes country_code and zone_code.
• stg_activity: Parses historical reporting dates into proper DATE formats and extracts operational integers for ship arrivals and cargo weights.

B. Core Marts Layer (models/marts/)

We structured a production-ready Star Schema to power upstream analytics:
• dim_ports: Houses immutable dimensional properties and geographic coordinates required for BI mapping.
• fct_port_congestion: Merges activity with port dimensions. It calculates Net Delay Days (Actual Dwell Days minus Planned Dwell Days) and applies a business logic metric: Estimated Delay Cost (assuming a industry-standard penalty of $20,000/day for every net delayed vessel).

C. Data Governance & Testing

Using dbt's native schema testing (models/marts/marts.yml), we implemented constraints to enforce data integrity before any information reaches the dashboard or ML models:
• unique & not_null: Enforced on primary keys (un_locode and congestion_key) to eliminate data duplication and structural gaps.

🔮 4. Predictive Analytics Stage (Python ML)

Once dbt materializes the clean, trusted analytics mart in PostgreSQL, Python is utilized exclusively to load the modeled data and build an early-warning system.
• Methodology: A Random Forest Regressor model was trained using dbt-engineered features: daily vessel calls, cargo volume, and a custom statistical lag feature representing the previous day's delay.
• Result: The model inputs real-time port load metrics to output expected vessel delays with an excellent R² Score, providing the operations team with a proactive re-routing window.
• Warehouse Integration: The final predictions are pushed directly back into a dedicated table (marts.predicted_port_delays) within the database.

📊 5. Business Impact & Dashboard Preview

The finalized pipeline feeds a high-performance executive dashboard. Instead of showing raw data, it tells a clear financial story:
• Executive Cards: Instantly highlight Total Financial Loss and Average Delay Days globally.
• Geographical Risk Map: Maps port coordinates (latitude/longitude) where bubble sizes dynamically scale according to the financial delay cost, allowing the operations team in France to pinpoint European bottlenecks (e.g., Marseille, Le Havre) immediately.
• Scatter Visuals: Plots Vessel Calls against Delay Days to visually reveal the exact operational tipping point where a port moves from "busy" to "critically congested."
Developed by Hassan Mohamed — Financial Modeling & Data Analytics Portfolio Project.
