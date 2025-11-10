## Cost Analysis

- A hybrid AI + human-in-the-loop approach is cost-effective versus today’s manual operations.
- With the target Azure architecture, cloud run-rate is low on a per-trip basis, while benefits scale with utilization and reliability improvements.
- Even at 5x–10x scaling, Azure costs grow roughly linearly and remain small relative to recovered revenue and OpEx savings.

### Assumptions (Pilot City, “Base”)

- Vehicles: 1,000; Completed trips/day: 6,000; Days/month: 30
- Price/trip: $6.00; Variable cost/trip: $2.00
- Lost-demand: 8% of attempted trips
- Disputes/chargebacks: 1.2% of trips at $12/refund
- Battery ops: 220 swaps/day; 18 min/swap; fully-loaded labor $35/hr
- Vision: 3 photos/return; 35% human review rate
- AI usage: ~300 tokens/trip (OpenAI); ~1 IoT msg/vehicle/min
- Azure run-rate (Base): ~$4,700/mo

### Current Monthly Costs (Pilot City)

- Trips/month: 180,000
- Revenue: $1,080,000
- Variable cost: $720,000
- Gross margin: $360,000
- Refunds from disputes: 6,000 trips/mo × 1.2% × $12 × 30 = $25,920
- Battery swap labor: 220 swaps/day × 18 min × $35/hr × 30 = $69,300

| Metric | Value |
|---|---:|
| Completed trips/month | 180,000 |
| Revenue ($) | 1,080,000 |
| Variable costs ($) | 720,000 |
| Gross margin ($) | 360,000 |
| Refunds/chargebacks ($/mo) | 25,920 |
| Battery swap labor ($/mo) | 69,300 |

### Azure Run-Rate (Target Architecture)

| Service | Monthly ($) |
|---|---:|
| OpenAI (≈54M tokens) | 540 |
| Vision (≈540k images) | 540 |
| Cognitive Search | 200 |
| IoT Hub (≈43M msgs) | 500 |
| Event Hubs/Service Bus | 100 |
| Cosmos DB | 600 |
| Azure SQL | 300 |
| Storage/Data Lake | 30 |
| Azure Maps | 150 |
| Functions/Durable | 40 |
| API/App/APIM | 400 |
| Redis | 60 |
| Key Vault | 10 |
| Monitor/Log Analytics | 350 |
| Azure ML | 100 |
| Subtotal | 3,920 |
| Buffer (20%) | 780 |
| Total | 4,700 |

Cost per trip from Azure: ~$0.026

### Benefits with AI-Assisted Operations

- Demand Prediction (reduce 8% stockouts by 30%):
  - Lost trips/day ≈ 6,000 ÷ (1 − 0.08) − 6,000 ≈ 522
  - Regained trips/day (30%): ≈ 157 → monthly margin: 157 × $4 × 30 ≈ $18,780
- Personalization Copilot (+5% trips): 300/day × $4 × 30 ≈ $36,000
- Battery Optimization (–20% time/route):
  - Daily swap labor = $2,310; savings/day = $462 → monthly ≈ $13,860
- Vision Return Verification (–40% disputes/refunds): 40% × $25,920 ≈ $10,368

| Capability | Monthly benefit ($) |
|---|---:|
| Demand Prediction | 18,780 |
| Personalization Copilot | 36,000 |
| Battery Optimization | 13,860 |
| Vision Verification | 10,368 |
| Subtotal benefits | 79,008 |
| Less: Azure run-rate | (4,700) |
| Net monthly uplift | 74,308 |

### Unit Economics (Base)

- Azure cost per trip: $4,700 / 180,000 ≈ $0.026
- Battery labor saved per trip: $13,860 / 180,000 ≈ $0.077
- Refunds avoided per trip: $10,368 / 180,000 ≈ $0.058
- Net per-trip improvement on existing trips: ≈ +$0.109
- Incremental trips (regained + personalization): ≈ 457/day → +$54.8k margin/mo

### Scenarios (Conservative / Base / Aggressive)

| Driver | Conservative | Base | Aggressive |
|---|:--:|:--:|:--:|
| Demand regain from stockouts | 15% of lost | 30% of lost | 45% of lost |
| Personalization lift | +3% | +5% | +8% |
| Battery labor savings | 10% | 20% | 30% |
| Dispute reduction | 20% | 40% | 60% |
| Azure run-rate | $4.7k | $4.7k | $5.5k |

| Scenario | Monthly benefits ($) | Net uplift ($/mo) |
|---|---:|---:|
| Conservative | ~43,100 | ~38,400 |
| Base | ~79,000 | ~74,300 |
| Aggressive | ~122,900 | ~117,400 |

Notes:

- Benefits scale primarily with completed trips and operational volumes; Azure costs scale roughly linearly with usage and stay small per trip.

### Scaling (1×, 5×, 10× Cities, holding per city mix)

| Scaling factor | Cities | Azure run-rate ($/mo) | Net uplift ($/mo) | Azure cost/trip |
|---:|---:|---:|---:|---:|
| 1× | 1 | 4,700 | 74,300 | ~$0.026 |
| 5× | 5 | 23,500 | 371,500 | ~$0.026 |
| 10× | 10 | 47,000 | 743,000 | ~$0.026 |

### ROI and Payback (Base)

- One-time costs (build + rollout): $180k (conservative range: $120k–$250k)
- Annualized net uplift (Base): ~$891.6k
- ROI = (Annual uplift − One-time) / One-time ≈ 396%
- Payback ≈ 2.4 months

### Guardrails (Cost Discipline)

- Autoscale; reserved capacity for steady workloads; spot/preemptible for batch
- Retrieval-first, small models, token budgets, response caching
- Log Analytics daily cap and sampling; storage lifecycle to cool/archive
- Per-feature budgets, alerts, and kill switches (FinOps)

### Business outcomes

- Productivity – Do more with the same team (≈2× now; up to 5× in manual-heavy areas)
  - With fewer manual photo checks and shorter battery-swap routes, staff can manage more trips and exceptions efficiently.
  - Day to day: Fewer back and forths, less time on routine reviews, focus on true issues.
  - We cut routine ops time per trip and reduce the number of manual reviews; the same team covers more vehicles and trips.
- Cost – Lower weekly operating costs (≈20–30% in our base; up to 60–80% where work is mostly manual)
  - We spend less on routine labor and refunds, while Azure run rate stays small per trip.
  - Day to day: Fewer dispute payouts and fewer long battery runs; cloud spend is predictable and modest.
  - In our base pilot, weekly ops + refunds drop meaningfully; if today’s process is largely manual review, reductions can be much bigger.
- Efficiency – More trips per staff hour (≈1.8× now; up to 4× on verification workflow)
  - Every hour of staff time turns into more completed trips and faster turnarounds.
  - Day to day: Operations feel lighter—less waiting on reviews, fewer “empty” miles for swap crews.
