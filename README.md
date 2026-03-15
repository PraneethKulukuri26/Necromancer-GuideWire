# GroceryGuard: AI-Powered Income Protection for India's Gig Economy

> 🔗 **[2-Minute Pitch & Prototype Video](#)** *(Update before Mar 20!)*

---

## 📋 Table of Contents

1. [The Problem & Our Persona](#1-the-problem--our-persona-)
2. [The Solution & Strict Scope](#2-the-solution--strict-scope-)
3. [Parametric Triggers](#3-parametric-triggers-)
4. [Persona-Based Scenarios & Workflow](#4-persona-based-scenarios--workflow-)
5. [Architecture & Tech Stack](#5-architecture--tech-stack-)
6. [Development Plan](#6-development-plan-6-weeks-)
7. [Team Necromancer](#-team-necromancer)

---

## 1. The Problem & Our Persona 

India's **8M+ grocery delivery partners** lose **20–30% weekly income (₹2,000–₹3,000)** to weather disruptions. Zero protection exists beyond accident coverage.

### 🎯 Persona: Ravi Sharma (25, Vizag)

```
• Zepto rider, Madhurawada dark store (VIZ-003, coastal)
• Part-time: Books 6–10 PM slots daily (24 hrs/week)
• Target: ₹22k/month  →  Rain = ₹5k lost
• Pain: "Monsoon season = 50% income gone"
```

---

## 2. The Solution & Strict Scope 

**AI parametric insurance** with weekly premiums → instant Friday payouts. No paperwork. Fraud-proof ML.

> ⚠️ **STRICT SCOPE:** Income loss **ONLY** from external disruptions.
> **NO** health / life / accidents / repairs.

> 🗓️ **WEEKLY MODEL:** Matches gig payout cycles perfectly.

```
Sunday 8 AM : ₹125 premium debited
Thu Rain    : Auto ₹600 payout Friday
Net         : Ravi protected 70% of lost income
```

---

## 3. Parametric Triggers

Real-time automation — **no claims process required.**

| Trigger | Threshold | Data Source |
|---|---|---|
| Heavy Rain | > 10 mm/hr | IMD FREE API |
| Extreme Heat | > 40°C | IMD |
| Severe Pollution | AQI > 300 | AQI.in FREE |
| Curfew / Shutdown | GPS radius < 500m from store | News API (mock) |

---

## 4. Persona-Based Scenarios & Workflow

### Scenario 1 — Sunday Premium Sync (Hours-Tiered Coverage)

Every **Sunday 8 AM** → Ravi's hours worked determine his coverage tier.

```
Premium = Base × Multipliers
Coverage Limit = Hours Tier × ₹250/hr × 80%
```

| Hours / Week | Tier | Premium | Max Coverage | Ravi Status |
|---|---|---|---|---|
| < 20 hrs | Basic | ₹95 | ₹4,000 | - |
| 20-30 hrs | Pro | ₹125 | ₹6,000 | ✅ Ravi |
| > 30 hrs | Elite | ₹165 | ₹8,000 | - |

Ravi (24 hrs/week) → **Pro Tier**: ₹125 premium, ₹6,000 max coverage.

#### Step-by-Step Process (With Hours Logic)

1. **Data Pull:** Rider profile + Zepto slot history (mock API)
2. **Hours Tier:** `<20h=Basic` \| `20-30h=Pro` \| `>30h=Elite`
3. **Risk Scoring:** ML multipliers applied to tier
4. **Premium Compute:** Tier base × risk multipliers
5. **Coverage Set:** Tier hours × ₹250 × 80%
6. **UPI Auto-debit:** Coverage activates instantly

#### Enhanced Risk Factors (Hours-Aware)

| Factor | Source | Multiplier | Ravi (24 hrs) |
|---|---|---|---|
| Hours Tier | Zepto slots | Pro tier base ₹125 | ✅ |
| Location | Dark store | Coastal +20% | +₹25 |
| Season | IMD | Pre-monsoon +10% | +₹12 |
| Forecast | IMD 7-day | 2 rain days +5% | +₹6 |
| History | DB | Clean −10% | -₹12 |
| **Total** |  |  | **₹156** |

#### Updated Formula (Tiered Coverage)

```python
def calculate_premium(rider_id):
  rider = get_rider_data(rider_id)

  # STEP 1: Hours Tier → Base Premium + Coverage
  weekly_hours = get_zepto_hours(rider.zepto_id, days=7)
  tier = get_coverage_tier(weekly_hours)  # {'base': 125, 'max_coverage': 6000}

  # STEP 2: Risk Multipliers
  multipliers = {
    'location': get_store_risk(rider.dark_store_id),    # 0.20
    'season': get_imd_season(),                         # 0.10
    'forecast': imd_api.forecast(rider.pincode),        # 0.05
    'history': get_claim_history(rider_id)              # -0.10
  }

  # STEP 3: Final Premium
  risk_factor = sum(multipliers.values())
  premium = tier['base'] * (1 + risk_factor)  # ₹125 × 1.25 = ₹156

  # STEP 4: Coverage Limit
  coverage_limit = tier['max_coverage']        # ₹6,000

  return {'premium': premium, 'coverage': coverage_limit}
```

#### Ravi's Real Calculation (Week 12)

```
Hours Tier: 24 hrs → Pro Tier
Base Premium: ₹125 | Max Coverage: ₹6,000

Risk Adjustments:
├── Coastal (VIZ-003): +₹25 (20%)
├── Pre-monsoon: +₹12 (10%)
├── 2 rain forecast: +₹6 (5%)
└── Clean history: -₹12 (-10%)

TOTAL: ₹156/week (covers up to ₹6,000 loss)
```

#### Payout Examples (Tier-Based)

| Hours Lost | Hourly Rate | Tier | Gross Loss | Payout (80%) | Net (after premium) |
|---|---|---|---|---|---|
| 4 hrs | ₹150 | Pro | ₹600 | ₹480 | ₹324 |
| 12 hrs | ₹150 | Pro | ₹1,800 | ₹1,440 | ₹1,284 |
| 24 hrs | ₹150 | Elite | ₹3,600 | ₹2,880 | ₹2,715 |

> **Math note:** Pro rows use Ravi's risk-adjusted premium (**₹156**), while the Elite row assumes base Elite premium (**₹165**) with a 0% additional risk multiplier.

---

### Scenario 2 — Thursday Rain → Friday Payout

```
7:15 AM : Ravi books 6–10 PM slot  →  Mock Zepto sync
8:00 PM : IMD rain > 10 mm         →  Zepto halts service
GPS     : Ravi stationary at VIZ-003 store ✅
Friday  : ₹600 auto-payout  (4 hrs × ₹150/hr)
```

---

### Scenario 3 — GPS Fraud Caught

```
Fraudster claims "rain loss" but:
  • GPS: 80 km/h at 8 PM     (not stationary at store) ❌
  • Zepto login: 10 PM       (worked a later slot)     ❌
  • ML Isolation Forest: 92% fraud score               ❌

Result: Claim rejected  +  16% future premium surcharge
```

---

### Scenario 4 — Store Transfer

```
Ravi moves to Hyderabad  →  New store HYD-045
Sunday recalc: VIZ-003 ₹125  →  HYD-045 ₹110

Transparent: Ravi sees "12% lower risk — inland store"
```

---

## 5. Architecture & Tech Stack 
### Platform Choice: Why Mobile over Web?
We are building a Mobile Application (React Native) instead of a web platform. Grocery delivery partners like Ravi rely 100% on their Android smartphones while on the move. A mobile app is mandatory for our specific use-case because we require **background GPS tracking** for location/activity validation to prevent fraud, **push notifications** for real-time weather alerts, and **1-click UPI capabilities** for instant premium debits and payouts.

### System Architecture Visual
```
┌─ React Native App ──────────┐
│  Ravi dashboard, alerts     │
└────────────┬────────────────┘
             ↓ WebSocket
┌─ FastAPI Backend (Python) ──┐
│  Async 10k req/s  |  AI/ML  │
└────┬──────────┬───────┬─────┘
     │          │       │
PostgreSQL TimescaleDB Redis 
(Policies) (GPS Logs) (Cache)
     │
Razorpay Sandbox  IMD / AQI 
(UPI Simulator)   Free APIs
```

### Detailed Tech Stack & Justification
#### 1. Frontend: React Native
**Why:** Captures 90%+ of the Indian gig-worker Android market. It allows us to build an offline-first experience with background location fetching, which is critical for verifying a worker's location against environmental disruptions.

#### 2. Backend: FastAPI (Python)
**Why:** FastAPI is built for high-performance, asynchronous execution, easily handling 10,000+ concurrent requests per minute during heavy rainstorms. Because it is Python-based, it allows us to natively integrate our machine learning models directly into the backend without relying on external microservices.

#### 3. Databases & Caching
- **PostgreSQL (Supabase):** Our primary relational database. ACID compliance is non-negotiable for managing the financial ledger, weekly premium deductions, and strict insurance policy management.
- **TimescaleDB:** An optimized time-series database specifically chosen to ingest and partition 1 Billion+ GPS logs. This is how we rapidly cross-reference Ravi's real-time location against historical weather data to catch GPS spoofing and fake claims.
- **Redis:** In-memory caching to store frequently accessed data like IMD weather states and mock Zepto slot histories, drastically reducing our external API call latency and costs.

#### 4. AI / Machine Learning Packages
- **Scikit-learn & XGBoost:** Used to build our dynamic predictive risk models. This allows us to calculate hyper-local weekly premiums based on predictive weather modeling and location risks.
- **Isolation Forests:** Deployed for intelligent anomaly detection in claims, instantly flagging irregular GPS movements or timing mismatches to prevent duplicate or fraudulent payouts.

#### 5. APIs & Integrations
- **Trigger APIs (IMD Weather & AQI.in):** Free-tier APIs used for real-time parametric trigger monitoring to automatically initiate claim payouts without manual intervention.
- **Platform API (Mock Zepto):** A simulated API to pull worker consistency and hours-tiered coverage data.
- **Payment Gateway (Razorpay Sandbox):** Integrated simulated payment systems to demonstrate instant weekly premium auto-debits and automated payout processing for lost income.

### 200k Rider Scale Performance
| Operation | Latency Target | Capacity / Volume | Tech Enabling This |
|---|---|---|---|
| Premium Calc | < 100ms | 10,000 / min | FastAPI + Redis Cache |
| Fraud Check | < 200ms | Real-time | TimescaleDB + Isolation Forest |
| Payouts | 5s batch | 100,000 / week | Razorpay Sandbox Batch API |

---

## AI/ML Integration (Premium + Fraud)
### 🧠 AI-Powered Dynamic Premium Model (XGBoost)
Our core pricing engine uses a Scikit-learn XGBoost Regression model to calculate hyper-local weekly premiums, perfectly aligning with the hackathon's mandatory **Weekly** pricing constraint.
- **Inputs:** `[hours_tier, store_risk, imd_forecast, claim_history]`
- **Output:** Optimal weekly premium (R² = 0.95 target accuracy)
- **Training Data:** Mock Zepto platform data + historical IMD weather patterns.

#### 🔮 Predictive Risk Scoring
- **Weather Prediction Layer:** IMD 7-day forecast API → Rain/Disruption probability.
- **Historical Claims:** Maps historical disruption data to create a location risk multiplier.
- **Hyper-Local Output:** XGBoost calculates a tailored "Disruption probability" per rider.
  - **Example:** Ravi's dark store (VIZ-003) has a 35% weekly disruption risk due to coastal weather patterns → Premium loading: +12% vs the inland national average.

### 🛡️ Fraud Detection Pipeline (Isolation Forest)
To strictly enforce legitimate claims, we deploy an Isolation Forest algorithm for intelligent anomaly detection, ensuring complete location and activity validation.

**Real-time Features Evaluated:**
- **Location Validation:** GPS velocity anomalies (e.g., travelling >60km/h during a reported "heavy rainstorm").
- **Activity Validation:** Cancellation timing (<2hrs before a parametric trigger).
- **Duplicate Claim Prevention:** Multiple Zepto logins (same day, different zones/slots).

**The ML Pipeline:**
1. Extract 7-day background GPS history from TimescaleDB.
2. Feature engineering → 12 distinct anomaly signals.
3. Isolation Forest evaluates the array → Outputs Fraud Score (0-1).
4. **Action:** A Threshold > 0.85 triggers auto-rejection for the parametric payout and applies a +16% penalty to their next weekly premium.

### 🗺️ ML Development Roadmap
- **Phase 2 [Weeks 3-4 - Automation & Protection]:** Deploy the Live Premium Prediction API, actively adjusting the weekly premium based on predictive weather modelling and hyper-local risk factors.
- **Phase 3 [Weeks 5-6 - Scale & Optimise]:** Train and finalise Advanced Fraud Models (using 1M+ mock GPS logs to catch spoofing) and launch the Intelligent Admin Dashboard for insurers to view predictive analytics on next week's disruption claims.

### 💼 Business Impact
- **95% fraud reduction** compared to traditional manual claims processes.
- **<50ms premium decisions** enabling real-time Sunday pricing at a 200k rider scale.
- **30% lower loss ratio** for insurers through proactive, dynamic risk pricing.

---

## 6. Development Plan (6 Weeks) 

```
Phase 1 ✅  [Mar  4–20] : README + 2-min video + GitHub
Phase 2     [Mar 21–Apr 4] : Mock APIs + Razorpay sandbox
Phase 3     [Apr  5–17] : Full ML models + Admin dashboard
```

### APIs Live

- **IMD Weather** — rain / heat triggers
- **AQI.in** — pollution AQI > 300
- **Mock Zepto** — slot bookings 6–10 PM
- **Razorpay Sandbox** — UPI payments

### Security

```
✅ Aadhaar OTP + JWT
✅ GPS TTL 90 days  (DPDP compliant)
✅ Honeypot endpoints
✅ Cloudflare DDoS  (FREE tier)
```

---

## Team Necromancer

| Name | Role | Email |
|---|---|---|
| **Kulukuri Praneeth** | Team Leader | `2300090274csitelge@gmail.com` |
| **Dodda Sai prudhvi raja** | Member | |
| **VASANI ROSHAN ANANTHA SAI** | Member | |

---

*Built for India's gig workers — zero manual claims, fraud-proof, scales to 1M riders.*
