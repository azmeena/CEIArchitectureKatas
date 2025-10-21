# CEIArchitectureKatas


# MobilityCorp Architecture Kata Submission

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

## AI-Driven Solution Overview


<img width="1680" height="2512" alt="High Level Architecture" src="https://github.com/user-attachments/assets/46108984-b66b-472e-a8ae-35cc6dd8c0fb" />


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


