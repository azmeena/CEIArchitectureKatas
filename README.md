

# MobilityCorp AI Enhanced Last Mile Transportation Platform

## Problem Statement
MobilityCorp provides short-term rentals for last-mile transport, including electric scooters, eBikes, and electric cars/vans. Operating in both urban and suburban areas, the company faces challenges in vehicle availability, battery management, and customer engagement. Customers book vehicles via a mobile app, and vehicles must be returned to designated spots with proof of return and feedback.

## Business Constraints
Vehicles must be returned to designated parking spots.
Cars and vans must be plugged into EV chargers upon return.
Bikes and scooters require manual battery swaps by staff.
Bookings vary by vehicle type: cars/vans (up to 7 days), bikes/scooters (up to 30 minutes in advance).
Customers pay per minute; fines apply for late or incorrect returns.
All vehicles are GPS-enabled and remotely unlockable.

## Key Challenges
Vehicle Availability: Customers often find vehicles unavailable at desired locations.
Battery Management: Electric vehicles frequently run out of charge, impacting service reliability.
Customer Retention: Most users rely on MobilityCorp for ad-hoc trips; regular usage is low.
Operational Efficiency: Staff must manually redistribute vehicles and swap batteries.

## Key Objectives
Optimize Fleet Distribution: Ensure vehicles are available where and when needed.
Enhance Battery Logistics: Prioritize battery swaps and charging based on predicted demand.
Improve Customer Engagement: Encourage regular usage through personalized experiences.
Automate Operations: Use AI to validate returns, detect damage, and streamline staff routing.

## Business Constraints
Booking rules: cars/vans bookable up to 7 days (bounded-duration); bikes/scooters up to 30 minutes with open-ended rentals (up to 12 hours). 
Payment per minute; fines for late/wrong returns. 
All vehicles have GPS; cars/vans may be remotely disabled; vehicles unlock via NFC-capable smartphone app. 
Return rules: must be returned to designated spots, photographic proof required; cars/vans must be plugged to charger on return. 
Operational reality: staff swap bike/scooter battery packs and redistribute vehicles via vans.
Budget and margins: solutions must be cost-effective (cloud + API usage costs, operations).
Regulatory & safety: remote disable and customer safety features must comply with local laws and insurance requirements.

## Reference Architecture
<img width="1680" height="2256" alt="ReferenceArchitecture" src="https://github.com/user-attachments/assets/b8aab02f-f48f-4120-bcd7-698901c905ec" />



## Cloud Architecture

<img width="2528" height="4690" alt="Azure Arch" src="https://github.com/user-attachments/assets/bd36b6ee-4de6-4a01-995c-9524b7db5ab9" />


## AI Usecase 1: Demand Prediction & Rebalance
### Demand Prediction & Rebalance

- **Purpose**: Forecast bay/zone demand and generate rebalancing targets.
- **Data**: Fabric OneLake Gold (rides, bay occupancy, weather/events), Cosmos DB current state (last seen), SQL bookings, feature snapshots cached in Redis.
- **Train**: Fabric pipeline → Azure ML (AutoML TS / custom LightGBM); register in AML; lineage in Purview.
- **Serve**
  - **Batch**: AML pipeline writes next 12–24h `demand_by_bay_hour` to SQL and hot ranks to Redis.
  - **Online what‑if**: Agent Orchestrator queries AML Endpoint for scenario runs.
- **Orchestration**: Agent Orchestrator (Azure AI Foundry) triggers on schedule or “demand spike” event; emits Service Bus message for Dispatch.
- **Validate**: WAPE/MASE, hotspot hit rate, utilization uplift; drift alerts on feature distributions.

<img width="4566" height="2772" alt="Demand Forecasting and Rebalancing Workflow" src="https://github.com/user-attachments/assets/25a1dca2-0ae8-4846-83d7-95fd3b4d0ff6" />

## AI Usecase 2: Battery & Charging Optimization

- **Purpose**: Predict time-to-empty and charger occupancy; create swap/charge routes.
- **Data**: Fabric Gold (SOC curves, temp/elevation), telemetry stream-derived features, charger events; Azure Maps traffic matrix.
- **Train**: Azure ML models for SOC depletion and occupancy; re‑trained daily; metrics logged and versioned.
- **Serve**: Agent Orchestrator fetches predictions → Optimization (OR-Tools or Azure Quantum) with Azure Maps → writes crew tasks/routes to SQL and task state to Cosmos DB; caches “next actions” in Redis.
- **Validate**: Run-outs avoided, task SLA, route time/distance, charger-occupancy error.
- 
<img width="5124" height="3998" alt="Battery and Charging Optimization" src="https://github.com/user-attachments/assets/78af0f28-c27a-4156-96d8-bf693a5844f3" />

## AI Usecase 3: Vision Return Verification

- **Purpose**: Verify correct bay, EV plugged, and visible damage; route low confidence to human review.
- **Data**: Photos in OneLake; metadata (bookingId, bayId, GPS, ts) in SQL; policies in Cognitive Search.
- **Models**: Azure AI Vision or Azure OpenAI Vision; thresholding; optional RAG explanation on policy.
- **Flow**: Returns API starts Durable Function → Agent Orchestrator calls Vision → high confidence: lock via IoT Hub, compute charges/fines in SQL, store evidence URIs; low confidence: enqueue review and hold charges.
- **Validate**: Precision/recall, dispute/chargeback rate, average review time, latency; Content Safety applied to uploads/prompts.

<img width="3958" height="3060" alt="Vision Return Verification" src="https://github.com/user-attachments/assets/3a1360dd-91ea-4c5f-b326-632afb03f9f0" />

## AI Usecase 4: Personalization, Policy & Pricing Copilot

- **Purpose**: Next‑best‑action (commute plan, bay suggestion, incentive) and natural‑language explanations of charges/fines with citations.
- **Data**: SQL (history, segments, prices), Redis (recent features), forecasts from SQL, Cognitive Search over policies/FAQs; PII minimized.
- **Models**: AML propensity/ranking + Azure OpenAI with RAG; Content Safety filters; bandit learning on outcomes.
- **Serve**: API calls Agent Orchestrator → score propensity → retrieve grounding → generate message/plan; write decisions to SQL; cache quick tips in Redis.
- **Validate**: Uplift (CTR/retention), CSAT, safety flags, latency/cost; A/B via APIM revisions.
<img width="5256" height="3253" alt="Personalization Engine" src="https://github.com/user-attachments/assets/5cf11d44-dabb-4d17-bfac-c525886604bf" />


Data collection is the bottleneck: telemetry completeness, time synchronization, and photo quality determine model accuracy. 

q4architecturekatamobilitycorp1…

Human-in-loop remains critical for edge cases; fully autonomous control is risky and legally sensitive.

RAG (retrieve + generate) can make the LLM's outputs more grounded — but retrieval quality and indexing matter.

Small, interpretable models for operational decisions + LLMs for natural language tasks is an effective hybrid.

Governance and observability pay off: tracking prompt versions and model outputs drastically reduces incident time-to-resolve.
