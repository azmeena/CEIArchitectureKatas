# MobilityCorp Use Cases

## A. Customer onboarding and account

### UCA1 Register and verify customer account
- **Actors**: Customer
- **Trigger**: New user installs app / opens web
- **Pre**: None
- **Main**: Sign up → verify phone/email → accept terms and privacy → add payment method → set communication preferences
- **Alt/Exceptions**: Payment verification fails; identity check required; underage block
- **Post**: Account active and eligible to book
- **Data**: Customer profile, KYC flags, payment token

### UCA2 Manage identity, devices, and consents
- **Actors**: Customer
- **Trigger**: User updates profile or consent
- **Main**: View and edit personal info → manage linked devices → manage marketing/telemetry/sharing consents
- **Post**: Audit trail updated; new consent version recorded
- **Data**: PII, consent ledger

---

## B. Discovery and booking

### UCB1 Discover available vehicles by map/list
- **Actors**: Customer
- **Trigger**: App home open or search
- **Pre**: Location services on; fleet data in sync
- **Main**: View nearby vehicles/bays with live status (SoC, range, price) → filter by vehicle type
- **Alt**: Location disabled; show citywide list and manual filters
- **Post**: Selected vehicle/bay context cached
- **Data**: Bay inventory, current positions (GPS), pricing rules

### UCB2 Book vehicle (cars/vans – scheduled window)
- **Actors**: Customer
- **Trigger**: Select car/van and time
- **Pre**: User verified; payment valid; future window ≤ 7 days; no overlapping booking per user/vehicle
- **Main**: Choose start/end; price estimate; confirm → booking holds inventory
- **Alt**: Late arrival grace period; auto-cancel if no show
- **Post**: Booking status = Confirmed (timeboxed)
- **Data**: Booking, reserved window, policy snapshot

### UCB3 Start openended ride (bikes/scooters – up to 30 min ahead)
- **Actors**: Customer
- **Trigger**: Tap “Start ride” now or within 30 minutes
- **Pre**: No active ride; vehicle free; battery above minimum; safety checklist
- **Main**: Reserve → approach → NFC/app unlock → ride starts; meter per minute
- **Alt**: Unlock fails → remote unlock fallback; vehicle fault prevents start
- **Post**: Trip status = Inprogress; billing clock running
- **Data**: Trip, meter, SoC at start

---

## C. Access and control
### UCC1 Unlock / lock vehicle via app (NFC / BLE / network)
- **Actors**: Customer
- **Trigger**: Start or pause/stop
- **Pre**: Active booking or ride; proximity checks
- **Main**: Send unlock → vehicle confirms; on stop, lock and collect final telemetry
- **Alt**: Connectivity loss; fall back to NFC/BLE token
- **Post**: State synced; security audit
- **Data**: Lock events, tokens

### UCC2 Remote disable (cars/vans) for safety/compliance
- **Actors**: Operations, Safety
- **Trigger**: Policy violation/theft/suspicious activity
- **Pre**: Rules matched; human authorization
- **Main**: Send disable; ensure safe stop envelope
- **Alt**: Abort if unsafe
- **Post**: Vehicle disabled; case opened
- **Data**: Disable command, case

---

## D. Riding and trip management

### UCD1 Active trip tracking and assistance
- **Actors**: Customer; Operations (read)
- **Trigger**: Trip start
- **Pre**: Telemetry streaming
- **Main**: Live timer, cost estimate, SoC, geofences; show nearby return bays
- **Alt**: Outofservice zone alert; low battery reroute
- **Post**: Trip continues or ends
- **Data**: GPS, SoC, geofence events

### UCD2 Midtrip incident reporting
- **Actors**: Customer
- **Trigger**: Report damage/hazard
- **Main**: Capture photos/notes; notify Ops; optionally pause trip
- **Post**: Case created; vehicle marked for inspection
- **Data**: Incident record

---

## E. Returns and compliance

### UCE1 Guided return to designated bay
- **Actors**: Customer
- **Trigger**: End ride
- **Pre**: Return bay availability; geofence; EV charger slot for cars/vans 
- **Main**: Navigate to designated bay → for cars/vans plug in charger → capture photos → submit feedback
- **Alt**: Bay full → alternate nearby bay with policy notice; charger fault → instruct alternate charger
- **Post**: Return request created
- **Data**: Return target, charger status 

### UCE2 AI vision return verification (automated)
- **Actors**: System; Human reviewer (exception)
- **Trigger**: Return request with photos
- **Pre**: Photo metadata includes booking, bay, GPS
- **Main**: **Vision Agent** checks bay correctness, plugin status (cars/vans), visible damage; confidence ≥ threshold → autoapprove; compute charges/fines; close trip
- **Alt**: Confidence < threshold → queue human review; potential partial credit
- **Post**: Trip closed; evidence stored; invoice updated
- **Data**: Evidence URIs, decision, confidence

### UCE3 Wronglocation or late return handling
- **Actors**: System; Customer Support (appeals)
- **Trigger**: Rule violation detected (geofence/time)
- **Main**: Assess fine per policy; inform customer with explanation and evidence; allow dispute window
- **Alt**: Waive/adjust based on support outcome
- **Post**: Invoice final; policy metrics updated
- **Data**: Policy snapshot, fine line items 

---

## F. Billing and payments

### UCF1 Realtime metering and pricing
- **Actors**: System
- **Trigger**: Trip events
- **Main**: Accrue perminute charges; apply plan/discounts; hold funds if needed
- **Post**: Running balance visible; final charge at close
- **Data**: Tariff rules, meter ticks 

### UCF2 Invoice and settlement
- **Actors**: System; Payment Provider; Customer
- **Trigger**: Trip closed
- **Main**: Generate invoice; capture payment; send receipt; update wallet/points
- **Alt**: Payment fail → retry/fallback; partial capture with dispute
- **Post**: Paid or outstanding
- **Data**: Invoice, payment token

### UCF3 Dispute and refund workflow
- **Actors**: Customer; Support
- **Trigger**: Customer disputes charge or fine
- **Main**: Case → review evidence and policy → decision and refund/adjust
- **Post**: Case closed; learning logged
- **Data**: Case notes, refund

---

## G. Operations: field service, batteries, rebalancing

### UCG1 Demand forecasting and rebalancing plan (agentic)
- **Actors**: **Demand & Rebalance Agent**, Planner
- **Trigger**: Scheduled run; event “demand spike”
- **Pre**: Data refresh (rides, bay occupancy, weather/events)
- **Main**: Forecast demand per bay/hour → compute target inventory and move list → publish plan and KPIs
- **Alt**: Data drift detected → fall back to baseline rules; flag to planner
- **Post**: Move tasks created; plan versioned

### UCG2 Dispatch route generation (agentic)
- **Actors**: **Dispatch Agent**, Crew Lead, Driver
- **Trigger**: Approved plan or ad hoc request
- **Pre**: Crew availability; traffic matrix
- **Main**: Cluster tasks → produce optimized multistop routes with SLAs and time windows → assign to crews
- **Alt**: Vehicle or bay becomes unavailable → reoptimize
- **Post**: Routes delivered to Crew App; live ETAs

### UCG3 Battery swap workflow (bikes/scooters)
- **Actors**: Crew
- **Trigger**: Route task “swap battery”
- **Main**: Navigate → swap pack → update SoC/photo → mark complete
- **Alt**: No charged packs; defer with reason
- **Post**: Pack inventory updated; SLA tracked
- **Data**: Swap event, SoC curve, pack IDs 

### UCG4 Charge session management (cars/vans)
- **Actors**: Crew; System
- **Trigger**: Return or dispatch instruction
- **Main**: Plug in; track kWh and occupancy; free spot when done
- **Alt**: Charger fault; reroute
- **Post**: Charge session record; utilization metrics
- **Data**: Session, bay charger status 

### UCG5 Battery and charger prioritization (agentic)
- **Actors**: **Charge & Battery Agent**
- **Trigger**: Forecasts + SoC telemetry
- **Main**: Predict timetoempty; predict charger occupancy; prioritize swaps/charges; emit tasks
- **Post**: Tasks queued; KPIs recorded (runouts avoided)

### UCG6 Live rebalancing during events/weather (agentic)
- **Actors**: **Demand & Rebalance Agent**, **Dispatch Agent**, Ops
- **Trigger**: Sudden demand deviation (concert, storm)
- **Main**: Whatif simulation; compare gains vs cost; if positive → issue highpriority moves; notify Ops
- **Post**: Event response plan executed

---

## H. Safety, compliance, and enforcement

### UCH1 Geofence policy enforcement
- **Actors**: System; Customer
- **Trigger**: Enter restricted or speedlimit zone
- **Main**: Notify; soft restrict speed; advise return paths
- **Post**: Compliance metrics updated
- **Data**: Zone catalog

### UCH2 Theft/loss detection and recovery
- **Actors**: System; Security; Law Enforcement liaison
- **Trigger**: Tamper patterns; nomovement after unlock; power removal
- **Main**: Flag; track; remote disable when safe; escalate
- **Post**: Case resolved
- **Data**: Anomaly signals

### UCH3 Incident triage and escalation
- **Actors**: Customer; Ops; Support
- **Trigger**: Crash or hazard report
- **Main**: Prioritize; notify emergency responders if thresholds; lock further bookings for the vehicle pending inspection
- **Post**: Resolution; learning captured

---

## I. Customer engagement and retention

### UCI1 Personalized commute plan and nudges (agentic)
- **Actors**: **Support Copilot**, **Policy & Pricing Agent**, Customer
- **Trigger**: Morning window; pattern detected
- **Main**: Recommend bay/vehicle and incentive; show ETA, cost saving vs alternatives; onetap reserve
- **Alt**: Supply low → suggest alternate mode/time
- **Post**: Plan accepted or dismissed; feedback loop updated
- **Data**: Segment, forecasts, promotions

### UCI2 Transparent policy and pricing explanations (agentic)
- **Actors**: Customer; **Policy & Pricing Agent**
- **Trigger**: User asks “why was I charged this?”
- **Main**: Retrieve policy snapshot and trip evidence; generate plainlanguage explanation with citations; offer dispute link
- **Post**: Customer informed; reduced support load

### UCI3 Loyalty, passes, and incentives
- **Actors**: Customer; Marketing
- **Trigger**: Thresholds met or targeted campaign
- **Main**: Offer ride passes or credit; apply automatically on eligible trips
- **Post**: Retention lift tracked

---

## J. Support and service

### UCJ1 Omnichannel support and case management
- **Actors**: Customer; Support
- **Trigger**: User opens chat/email/call
- **Main**: Identify user; surface recent trips, policies, and agent decisions; propose quick actions (refund, waive fine, credit)
- **Post**: Case closed; quality checks

### UCJ2 Human review queue for vision exceptions
- **Actors**: Reviewer
- **Trigger**: Low confidence return verification
- **Main**: Review photos and GPS; confirm/override; annotate cause (lighting, occlusion)
- **Post**: Model feedback captured; user notified

---

## K. Asset care and maintenance

### UCK1 Fault reporting and triage
- **Actors**: Customer; Crew; System
- **Trigger**: On return or inspection
- **Main**: Log fault; severity; take out of service; schedule repair
- **Post**: Work order created; status on Ops board

### UCK2 Preventive maintenance planning
- **Actors**: System; Maintenance Planner
- **Trigger**: Usage/age thresholds; anomaly signals
- **Main**: Suggest maintenance windows and parts; bundle with rebalancing routes where possible
- **Post**: Plan accepted; downtime minimized

---

## L. Data, analytics, and AI governance

### UCL1 Data product curation for analytics
- **Actors**: Data Engineering; Analytics
- **Trigger**: New source or model
- **Main**: Publish trip, booking, return, and telemetry data products; define SLAs and lineage; certify datasets
- **Post**: Analysts and agents consume governed data

### UCL2 Model lifecycle and evaluation
- **Actors**: MLOps; **Agent Orchestrator**
- **Trigger**: New model version or drift
- **Main**: Train → register → evaluate with offline metrics (WAPE/MASE, precision/recall); run A/B online; rollback on regressions
- **Post**: Model promoted or reverted

### UCL3 Policy, safety, and human-in-the-loop controls
- **Actors**: AI Governance; Ops Leads
- **Trigger**: Low confidence or high-risk actions
- **Main**: Route to human approval; enforce content safety; log prompts/policies; periodic review
- **Post**: Compliance satisfied; learnings fed back

---

## M. Finance and reporting

### UCM1 Revenue recognition and reconciliation
- **Actors**: Finance
- **Trigger**: End of day/month
- **Main**: Aggregate invoices, payments, refunds, fines; reconcile with payment provider and ledger
- **Post**: Financials posted; anomalies flagged

### UCM2 Operations KPIs and cost to serve
- **Actors**: Leadership; Ops
- **Trigger**: Scheduled dashboard refresh
- **Main**: Show utilization, runouts avoided, rebalancing uplift, charger occupancy, SLA attainment, cost/ride
- **Post**: Actions assigned

---

## N. Risk, security, and privacy

### UCN1 Access control and least privilege
- **Actors**: Security; IT
- **Trigger**: Role change or audit
- **Main**: Enforce RBAC, rotate secrets, review privileged actions; log all access
- **Post**: Attestation complete

### UCN2 Data minimization and retention
- **Actors**: Privacy; Data
- **Trigger**: Policy update or request
- **Main**: Minimize PII in analytics/features; mask evidence; enforce retention windows for telemetry and photos
- **Post**: Data retained only as required
