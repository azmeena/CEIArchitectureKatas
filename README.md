

# MobilityCorp AI Enhanced Last Mile Transportation Platform

## Problem Statement
MobilityCorp provides short-term rentals for last-mile transport, including electric scooters, eBikes, and electric cars/vans. Operating in both urban and suburban areas, the company faces challenges in vehicle availability, battery management, and customer engagement. Customers book vehicles via a mobile app, and vehicles must be returned to designated spots with proof of return and feedback.

## Key Challenges
- Vehicle Availability: Customers often find vehicles unavailable at desired locations.
- Battery Management: Electric vehicles frequently run out of charge, impacting service reliability.
- Customer Retention: Most users rely on MobilityCorp for ad-hoc trips; regular usage is low.
- Operational Efficiency: Staff must manually redistribute vehicles and swap batteries.

## Key Objectives
- Optimize Fleet Distribution: Ensure vehicles are available where and when needed.
- Enhance Battery Logistics: Prioritize battery swaps and charging based on predicted demand.
- Improve Customer Engagement: Encourage regular usage through personalized experiences.
- Automate Operations: Use AI to validate returns, detect damage, and streamline staff routing.

## Business Constraints
- Vehicles must be returned to designated parking spots.
- Cars and vans must be plugged into EV chargers upon return.
- Bikes and scooters require manual battery swaps by staff.
- Bookings vary by vehicle type: cars/vans (up to 7 days), bikes/scooters (up to 30 minutes in advance).
- Customers pay per minute; fines apply for late or incorrect returns.
- All vehicles are GPS-enabled and remotely unlockable.


## Our AI-Powered Solution

Our solution primarily leverages the Microsoft Azure ecosystem, given its comprehensive suite of AI, infrastructure, and supporting technologies required to deliver this solution effectively. One key consideration during our design deliberations was AI platform stability and potential vendor lock-in. However, we determined that Microsoft’s position as a leading and mature player in the enterprise AI space, combined with its proven product stability, mitigates much of this risk. Furthermore, Azure’s AI Foundry architecture provides flexibility to switch or integrate new underlying AI models as the LLM landscape evolves, without disrupting the overall solution stack. This ensures long-term adaptability and resilience against shifts in the rapidly changing AI ecosystem.

Additionally, we selectively incorporated non-Microsoft products in areas where alternative solutions demonstrated clear superiority over their Microsoft counterparts. For instance, we integrated Google Maps for location services due to its richer mapping data and advanced APIs, and adopted open-source frameworks such as Prophet for demand forecasting, given their proven accuracy and flexibility. This hybrid approach ensures we leverage the best-in-class technologies across the ecosystem while maintaining seamless integration within the Azure-based architecture.The decisions for these choices are documented in ADR's



Below describes our AI solution approach 

**1. **Demand Prediction Engine: Putting Vehicles Where They're Needed****

The Challenge: How do we know when and where people will want vehicles? Can we anticipate customer needs?

Our AI Solution: We deployed a sophisticated machine learning pipeline that analyzes historical usage patterns, real-time events, and weather conditions to forecast demand across all MobilityCorp locations.
How It Works:

- Time-series forecasting models analyze historical booking data to identify temporal patterns (rush hour, weekends, seasonal trends)
- Location-based demand heatmaps predict hot spots using GPS tracking data combined with local events (concerts, sports games, festivals)
- Weather integration adjusts predictions based on conditions—rain increases car demand while reducing scooter usage
- Real-time recalibration continuously updates predictions as actual bookings occur

Business Impact: Staff can proactively reposition vehicles before demand spikes, dramatically reducing "vehicle not available" complaints and maximizing fleet utilization.

**2. Battery Optimization AI: Intelligent Resource Allocation**

The Challenge: Can we work out how to prioritise which vehicles to switch out batteries? (for bikes and scooters)

Our AI Solution: A multi-objective optimization engine that prioritizes battery swap routes based on predicted demand, current battery levels, and staff routing efficiency.
How It Works:

- ML prioritization algorithms score each vehicle based on:

	- Current battery level and predicted remaining usage time
	- Expected demand at the vehicle's location (from Demand Prediction Engine)
	- Historical usage patterns for that specific parking bay


- Route optimization generates efficient staff routes that maximize battery swaps per trip
- Battery life prediction models forecast degradation patterns to schedule preventive maintenance

Business Impact: Staff spend less time driving and more time swapping batteries where it matters most. Vehicles are charged and ready during peak demand periods, reducing customer frustration and maximizing revenue.

**3. Personalization Engine: Building Customer Loyalty Through Intelligence**

The Challenge: Right now, most of our customers just use them on an ad-hoc basis -we’d like more of them to rely on our fleet for regular trips like daily commutes

Our AI Solution: An LLM-powered personalization engine that understands individual user patterns and proactively encourages routine usage through smart recommendations and incentives.
How It Works:

- Usage pattern analysis identifies potential commuter routes based on repeated bookings
- LLM-powered recommendations generate natural language suggestions: "We noticed you often travel from Downtown to Tech Park on weekday mornings. Want to set up a recurring booking?"
- Smart notifications send timely alerts when vehicles are available at frequently-used locations
- Personalized incentives offer targeted promotions based on individual behavior patterns

Business Impact: Increased customer lifetime value through higher booking frequency. Ad-hoc users transition to daily commuters, creating predictable revenue streams.

**4. Vision AI: Automating Compliance and Quality Control**

The Challenge: Manual verification of vehicle returns creates operational overhead and potential disputes.

Our AI Solution: Computer vision models that automatically verify parking compliance, detect vehicle damage, and validate charging connections.
How It Works:

- Photo verification analyzes customer-submitted photos to confirm vehicles are in designated spots
- Damage detection identifies scratches, dents, or missing components, triggering maintenance workflows
- Charging compliance validation verifies that cars and vans are properly plugged into EV chargers
- Automated fine assessment reduces disputes by providing visual evidence

Business Impact: Reduces customer service workload, speeds up return processing, and ensures fair fine assessment with objective evidence.


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

Human-in-loop remains critical for edge cases; fully autonomous control is risky and legally sensitive.

RAG (retrieve + generate) can make the LLM's outputs more grounded — but retrieval quality and indexing matter.

Small, interpretable models for operational decisions + LLMs for natural language tasks is an effective hybrid.

Governance and observability pay off: tracking prompt versions and model outputs drastically reduces incident time-to-resolve.

## Limitations with adoption of Gen AI and how we solved them
- Non-determinism: LLM responses are probabilistic and unsafe for unverified control commands.
  - Mitigations: Hard-coded safety checks; human-in-the-loop for sensitive actions; degrade to deterministic templates/rules or “human review required”; circuit breakers, timeouts, idempotency.
- Data drift & retraining: Usage patterns change by season/city; models must be monitored and retrained.
  - Mitigations: Retrieval grounding with “no answer” on low confidence; quality telemetry and feature flags; regression runbook to freeze/revert; scheduled monitoring and retraining.
- Privacy & compliance: Location, payment, and photos are sensitive; avoid sending PII to third-party LLMs without guarantees.
  - Mitigations: Provider allowlist by data class; regional routing; PII minimization/redaction; full audit logs of AI decisions.
- Explainability: Summaries/recommendations need traceability for audits and disputes.
  - Mitigations: Retrieval-grounded answers with citations; audit trails; “no answer” on low confidence.
- Cost & latency: High-volume/real-time inference can be costly and latency-sensitive.
  - Mitigations: Per-feature budgets, alerts, caps, kill switches; tiered defaults prefer smaller/faster models on critical paths; prompt compression/truncation; retrieval-first; caching (Redis) and safe batching; policy router with latency SLOs and max cost/request.
- Vendor lock-in & disruption risk: Pricing/SLA changes or outages from a single provider.
  - Mitigations: Capability interfaces (`LLMProvider`, `VisionProvider`, `EmbeddingsProvider`) with per-vendor adapters; multi-provider policy router with health/cost-aware scoring and sticky routing; hot/warm secondary providers; feature flags for instant cutover.
- Edge limitations: LLMs may be unsuitable on-device for constrained scenarios.
  - Mitigations: Use smaller models or embeddings-only on device; offload heavy inference to cloud; degrade to cached/deterministic flows when constrained.

	## Architecture hooks
	- All AI calls traverse an Agent Orchestrator behind capability interfaces.
	- Retrieval grounded via Cognitive Search; fall back to “no answer” on low confidence.
	- Cost/quality telemetry per request; policies centrally managed (APIM/Config).
	

## Architecture Principles Achieved

1. Separation of Concerns

- Clear layer separation: Client Layer -> API Gateway -> Services -> Data -> IoT layers are distinctly separated
- Domain-driven service boundaries: Booking, Payment, and Fleet services handle specific business capabilities independently
- AI services isolated: AI-powered services are grouped separately, preventing AI concerns from bleeding into core business logic

2. Scalability

- Microservices architecture: Core services (Booking, Payment, Fleet) can scale independently based on demand
- Stateless API Gateway: Enables horizontal scaling of the entry point
- Separate data stores: Operational DB, Time-Series DB, Data Lake, and Feature Store can scale based on their specific workload patterns
- AI services decoupled: Demand Prediction, Battery Optimization, Personalization, and Vision AI can scale independently

3. Modularity & Loose Coupling

- Service-oriented design: Each service has well-defined responsibilities
- API Gateway pattern: Decouples clients from service implementations
- External integrations abstracted: Payment Gateway, Weather API, Events API, and LLM Provider are treated as external dependencies that can be swapped

4. Maintainability

- Single responsibility per service: Each service focuses on one domain (e.g., Booking Service only handles reservations)
- Clear interfaces: API Gateway provides a stable contract for clients
- Modular AI components: AI capabilities can be updated or replaced without affecting core business services

5. Reliability & Availability

- Fault isolation: Failure in one service (e.g., Personalization Engine) won't bring down core functions (Booking, Payment)
- Critical path protection: Core business services are separated from experimental AI features
- IoT layer independence: Vehicle IoT devices operate independently with their own communication layer

6. Performance Optimization

- Time-Series DB: Optimized for GPS tracking and battery data with high write throughput
- Feature Store: Pre-computed ML features reduce inference latency
- Data Lake separation: Historical data and ML training don't impact operational database performance
- Caching potential: API Gateway can implement caching strategies

7. Security

- Centralized authentication: API Gateway & Auth layer provides single point for security enforcement
- Layer-based access control: Different layers can have different security policies
- External integrations isolated: Payment processing handled by external gateway reduces PCI compliance scope
- IoT security: Separate device layer with GPS, NFC, and remote lock capabilities

8. Extensibility

- Plugin architecture for AI: New AI services can be added without modifying core services
- External API abstraction: New external data sources (Weather, Events) can be integrated easily
- LLM Provider abstraction: Can swap AI/ML service providers without changing application logic

9. AI-Specific Principles
- AI Uncertainty Handling

	- Provider abstraction: LLM Provider is a separate integration point, allowing provider switching
	- Fallback capability: Core services don't depend on AI services - business can operate if AI fails
	- Model versioning support: Feature Store enables A/B testing and gradual rollouts

- AI Validation & Verification

	- Data Lake for monitoring: Historical data enables validation of AI predictions vs. actual outcomes
	- Feature Store: Allows tracking of ML features and model performance over time
	- Vision AI verification: Photo verification results can be audited and retrained

10. Cost Optimization

- Right-sized data stores: Operational DB vs. Time-Series DB vs. Data Lake optimized for their use cases
- Selective AI usage: AI services target specific high-value problems (demand prediction, battery optimization)
- External API efficiency: Weather and Events APIs reduce need for internal data collection infrastructure

11. Interoperability

- Standard API patterns: RESTful API Gateway enables integration with various clients
- IoT standards: NFC, GPS, remote lock/disable use standard protocols
- Multiple client support: Mobile app and Staff Portal served through same API layer

12. Data-Driven Decision Making

- Comprehensive data collection: Operational DB + Time-Series DB + Data Lake capture all necessary data
- ML training pipeline: Data Lake feeds Feature Store for continuous model improvement
- Real-time and batch processing: Time-Series DB for real-time tracking, Data Lake for batch analytics


## Our Strategy for Verifying GenAI Works in Production (Non‑determinism)

### Pre‑production evaluation
- Golden datasets per use case (Demand, Battery, Vision, Personalization) including edge cases.
- Offline eval harness with task‑specific metrics; CI gates require threshold wins vs control.
- Version all prompts/chains/models; semantic versions with model cards and rollout notes.

### Safe rollouts
- Shadow traffic → canary (1–5%) → progressive rollout; auto‑rollback on SLO or safety breach.
- A/B tests and bandits for personalization within policy and safety bounds.

### Observability & drift
- Log: inputs/features, prompt template+version, provider+model+version, output, latency, cost, safety flags, retrieval coverage.
- Drift monitors: input distribution shift, hallucination/refusal rate, toxicity flags, grounding coverage/freshness.
- Weekly regression jobs on fresh samples; scorecards by city/segment.

### Guardrails & human‑in‑loop
- Hard checks for critical actions (e.g., fines/locks require deterministic validation).
- Confidence thresholds route low‑confidence cases to human review; Content Safety pre/post filters.
- RAG rules: require citations; degrade to “no answer” if retrieval confidence is low.

### Execution checklist
- Golden datasets + eval harness wired into CI.
- Shadow/canary rollout with auto‑rollback enabled.
- Dashboards and alerts for quality, cost, latency, safety, and drift.
- Human‑review workflow active; RAG guardrails enforced.

## Business Outcomes
Our AI-enhanced architecture directly addresses MobilityCorp's stated challenges:
- Right vehicles, right places: Demand prediction reduces "vehicle unavailable" complaints by anticipating needs
- Optimized battery management: Intelligent prioritization ensures high-demand vehicles are always charged
- Increased customer loyalty: Personalization converts occasional users into regular commuters
- Operational efficiency: Vision AI and route optimization reduce staff workload and operational costs
- Scalable foundation: Architecture supports expansion to new cities without redesigning core systems

## Conclusion
By strategically deploying AI at four critical decision points, demand forecasting, battery optimization, customer personalization, and compliance automation, we've transformed MobilityCorp from a reactive rental service into a predictive mobility platform. The architecture balances innovation with pragmatism, leveraging cutting-edge AI capabilities while maintaining operational resilience and preparing for the uncertain future of AI technology.

