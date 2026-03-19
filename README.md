# GroceryGuard: AI-Powered Income Protection for India's Gig Economy

> 🔗 **[Live Prototype Demo](https://nicromancer-demo.vercel.app/)** | **[2-Minute Pitch Video](https://youtu.be/9xus4h3XnXw)** 

---

## 📋 Table of Contents

1. [The Problem & Our Persona](#1-the-problem--our-persona-)
2. [The Solution & Strict Scope](#2-the-solution--strict-scope-)
3. [Parametric Triggers](#3-parametric-triggers-)
4. [Persona-Based Scenarios & Workflow](#4-persona-based-scenarios--workflow-)
5. [Architecture & Tech Stack](#5-architecture--tech-stack-)
6. [AI/ML Integration (Premium + Fraud)](#6-aiml-integration-premium--fraud)
7. [Adversarial Defense & Anti-Spoofing Strategy](#7-adversarial-defense--anti-spoofing-strategy)
8. [Financial Model & Sustainability](#8-financial-model--sustainability-)
9. [Development Plan](#9-development-plan-6-weeks-)
10. [Team Necromancer](#10-team-necromancer)

---

## 1. The Problem & Our Persona 

India's **grocery delivery partners** lose **20–30% weekly income (₹2,000–₹3,000)** to weather disruptions. Zero protection exists beyond accident coverage.

### 🎯 Persona A: Ravi Sharma (25, Vizag) — Primary

```
• Zepto rider, Madhurawada dark store (VIZ-003, coastal zone)
• Part-time: Books 6–10 PM slots daily (24 hrs/week)
• Target: ₹22k/month  →  Rain = ₹5k lost per monsoon week
• Pain: "Monsoon season = 50% income gone, no safety net"
• Risk Profile: HIGH — coastal, pre-monsoon zone, consistent hours
```

### 🎯 Persona B: Priya Menon (29, Hyderabad) — Secondary

```
• Zepto rider, Kondapur dark store (HYD-045, inland zone)
• Full-time: Books 10 AM–10 PM split shifts (48 hrs/week)
• Target: ₹32k/month  →  Extreme heat/AQI disruptions in peak summer
• Pain: "40°C afternoons — Zepto halts, I lose 4 hours a day"
• Risk Profile: MEDIUM — inland, heat/pollution risk, high hours
```

> Comparing Ravi vs Priya drives our tiered premium model: coastal riders carry higher weather-loading; inland riders carry higher heat/AQI loading. Same platform, same weekly model, hyper-local pricing.

---

## 2. The Solution & Strict Scope 

**AI parametric insurance** → **INSTANT payouts on disruption day**. No paperwork. Fraud-proof ML.

> ⚠️ **STRICT SCOPE:** Income loss **ONLY** from external disruptions.
> **NO** health / life / accidents / vehicle repairs.

> 🗓️ **WEEKLY MODEL:** Sunday premium auto-debit → **Mon–Sun coverage window**.

```
Sunday 8 AM: ₹156 premium auto-debited (Ravi, Pro Tier)
Wednesday 8 PM: IMD rain >10mm → Zepto halts service at VIZ-003
Wednesday 8:20 PM: ₹600 INSTANT UPI payout (4 hrs × ₹150/hr × 80%)
Net result: Ravi protected the SAME night — no waiting, no forms
```

---

## 3. Parametric Triggers

Real-time automation — **no manual claims process required.**

| # | Trigger | Threshold | Data Source | Payout Basis |
|---|---|---|---|---|
| T1 | Heavy Rain | > 10 mm/hr | IMD FREE API | Hours halted × hourly rate × 80% |
| T2 | Extreme Heat | > 40°C | IMD FREE API | Hours halted × hourly rate × 80% |
| T3 | Severe Pollution | AQI > 200 (Very Unhealthy) | AQI.in FREE | Hours halted × hourly rate × 80% |
| T4 | Curfew / Zone Shutdown | Official shutdown alert + rider GPS movement < 200m/30min | News API (mock) + GPS signal | Full booked shift loss × 80% |
| T5 | Platform API Downtime | Zepto API health-check timeout > 15 min | Mock Zepto API ping | Hours of confirmed downtime × 80% |

> **T3 Note:** Threshold updated from AQI > 300 to AQI > 200 ("Very Unhealthy" per WHO/CPCB classification) — stronger scientific backing and captures more real disruption events, especially relevant for Priya in Hyderabad summers.

> **T4 Note:** Curfew detection uses two independent signals — a verified news/gazette API alert AND observed GPS movement freeze — to prevent false triggers from routine low-movement periods.

> **T5 Note:** Platform API downtime (Zepto app crashes) is explicitly listed in the DEVTrails problem statement as a valid income disruption. This trigger directly addresses that scenario.

---

## 4. Persona-Based Scenarios & Workflow

### Scenario 1 — Sunday Premium Sync (Hours-Tiered Coverage)

Every **Sunday 8 AM** → Ravi's past 7 days of hours worked determine his coverage tier for the coming week.

```
Premium = Base Tier × Risk Multipliers
Coverage Limit = Hours Tier × ₹250/hr × 80%
```

| Hours / Week | Tier | Base Premium | Max Coverage | Ravi | Priya |
|---|---|---|---|---|---|
| < 20 hrs | Basic | ₹95 | ₹4,000 | - | - |
| 20–30 hrs | Pro | ₹125 | ₹6,000 | ✅ (24 hrs) | - |
| > 30 hrs | Elite | ₹165 | ₹8,000 | - | ✅ (48 hrs) |

#### Step-by-Step Premium Calculation

1. **Data Pull:** Rider profile + Mock Zepto slot history API
2. **Hours Tier:** `<20h = Basic` | `20–30h = Pro` | `>30h = Elite`
3. **Risk Scoring:** XGBoost ML multipliers applied to tier base
4. **Premium Compute:** `Tier base × (1 + sum of risk multipliers)`
5. **Coverage Set:** `Tier max_coverage` (capped per tier)
6. **UPI Auto-debit:** Sunday 8 AM → Coverage active Mon–Sun

#### Risk Multiplier Table

| Factor | Source | Ravi (VIZ-003 Coastal) | Priya (HYD-045 Inland) |
|---|---|---|---|
| Hours Tier | Mock Zepto API | Pro base ₹125 | Elite base ₹165 |
| Location Risk | Dark store zone | Coastal +20% (+₹25) | Inland +5% (+₹8) |
| Season | IMD seasonal | Pre-monsoon +10% (+₹12) | Summer heat +8% (+₹13) |
| 7-Day Forecast | IMD forecast API | 2 rain days +5% (+₹6) | 3 heat days +6% (+₹10) |
| Claim History | PostgreSQL DB | Clean −10% (−₹12) | Clean −10% (−₹16) |
| **Final Premium** | | **₹156/week** | **₹180/week** |

#### Premium Formula (Python)

```python
def calculate_premium(rider_id):
    rider = get_rider_data(rider_id)

    # STEP 1: Hours Tier → Base Premium + Coverage Cap
    weekly_hours = mock_zepto_api.get_hours(rider.zepto_id, days=7)
    tier = get_coverage_tier(weekly_hours)
    # Returns: {'base': 125, 'max_coverage': 6000} for Pro Tier

    # STEP 2: XGBoost Risk Multipliers (hyper-local)
    features = [
        get_store_zone_risk(rider.dark_store_id),   # coastal/inland/flood-prone
        get_imd_season_factor(),                     # monsoon/summer/winter
        imd_api.forecast_disruption_prob(rider.pincode, days=7),
        get_normalized_claim_history(rider_id)       # negative = discount
    ]
    risk_multiplier = xgboost_premium_model.predict([features])[0]

    # STEP 3: Final Premium
    premium = round(tier['base'] * (1 + risk_multiplier), 2)

    # STEP 4: Coverage Limit (tier-capped)
    coverage_limit = tier['max_coverage']

    return {'premium': premium, 'coverage': coverage_limit, 'tier': tier['name']}
```

#### Ravi's Real Calculation (Week 12, Pre-Monsoon)

```
Hours Tier: 24 hrs → Pro Tier
Base Premium: ₹125 | Max Coverage: ₹6,000

Risk Adjustments:
├── Coastal store (VIZ-003): +₹25  (+20%)
├── Pre-monsoon season:      +₹12  (+10%)
├── 2 rain days forecast:    +₹6   (+5%)
└── Clean claim history:     −₹12  (−10%)

TOTAL PREMIUM: ₹156/week
COVERAGE CAP:  ₹6,000
LOSS RATIO CHECK: Vizag avg 6 rain events/month × ₹480 avg payout = ₹2,880
                  vs. ~640 active riders × ₹156 = ₹99,840 collected
                  → Pool loss ratio: ~35% ✅ Sustainable
```

#### Payout Examples (Tier-Based)

| Hours Lost | Hourly Rate | Tier | Gross Loss | Payout (80%) | Net (after premium) |
|---|---|---|---|---|---|
| 4 hrs | ₹150 | Pro | ₹600 | ₹480 | ₹324 |
| 12 hrs | ₹150 | Pro | ₹1,800 | ₹1,440 | ₹1,284 |
| 24 hrs | ₹150 | Elite | ₹3,600 | ₹2,880 | ₹2,715 |

> **80% payout rate** is intentional: the 20% co-pay prevents full moral hazard while still replacing the majority of lost income. Coverage caps per tier further protect the liquidity pool during catastrophic weather weeks.

---

### Scenario 2 — Wednesday Rain → INSTANT Payout

```
7:15 PM: Ravi books 6–10 PM slot → Mock Zepto API confirms booking
8:00 PM: IMD rain crosses 10mm/hr → Zepto platform halts VIZ-003 service
8:05 PM: T1 trigger fires → System queries all active slot-holders at VIZ-003
8:15 PM: 7-layer fraud check runs in parallel → Ravi passes (score: 0.12)
8:20 PM: ₹480 INSTANT UPI payout (4 hrs × ₹150 × 80%)
```

**Ravi receives money WHILE the rain continues** → No waiting until Friday.

---

### Scenario 3 — GPS Fraud Caught

```
Fraudster claims "rain loss" from VIZ-003 but:
  • GPS: 80 km/h movement at 8 PM   (not stationary near store) ❌
  • Zepto login: Next slot at 10 PM  (was working a later shift) ❌
  • Accelerometer: 0 Hz vibration    (stationary at home)        ❌
  • ML Isolation Forest score: 0.92                              ❌

Result: Claim BLOCKED + 16% future premium surcharge applied
```

---

### Scenario 4 — Store Transfer (Dynamic Re-pricing)

```
Ravi transfers to Hyderabad → New assignment: HYD-045 (inland store)
Sunday auto-recalculation: Coastal loading removed, inland loading applied
VIZ-003 premium: ₹156 → HYD-045 premium: ₹138 (12% reduction)

App notification: "Good news! Your new store zone is lower risk.
                  Premium reduced to ₹138/week. Coverage: ₹6,000."
```

---

### Scenario 5 — Platform Downtime (T5 Trigger)

```
2:00 PM: Zepto API health-check fails at HYD-045 (Priya's store)
2:15 PM: 15-minute timeout confirmed → T5 trigger fires
2:16 PM: All active slot-holders at HYD-045 queried
2:30 PM: Fraud check passes (Priya: score 0.08)
2:32 PM: ₹300 UPI payout (2 hrs × ₹150 × 80% partial slot loss)
3:45 PM: Zepto API restored → T5 trigger reset, coverage resumes normally
```

---

## 5. Architecture & Tech Stack 

### Platform Choice: Why Mobile over Web?

We are building a **Mobile Application (React Native)** instead of a web platform. Grocery delivery partners like Ravi rely 100% on their Android smartphones while on the move. A mobile app is mandatory for our use-case because:
- **Background GPS tracking** — continuous location validation to prevent fraud, even when app is minimised
- **Push notifications** — real-time weather alerts and payout confirmations the moment a trigger fires
- **1-click UPI** — instant premium auto-debit and payout via deep-linked UPI flow
- **Accelerometer / Gyroscope access** — native sensor APIs for L2 bike-vibration fraud detection

### System Architecture

```
┌─ React Native App (Android/iOS) ──────────────────────────┐
│  Rider Dashboard | Alerts | Coverage Status | Payout Feed  │
└────────────────────────┬───────────────────────────────────┘
                         ↓ WebSocket (real-time alerts)
                         ↓ REST (claims, premium, auth)
┌─ FastAPI Backend (Python) ─────────────────────────────────┐
│  Async 10k req/s  |  ML Inference  |  Trigger Engine       │
└────┬──────────────┬──────────────┬──────────────┬──────────┘
     │              │              │              │
PostgreSQL      PostgreSQL      Redis         ML Models
(Policies,     (GPS + time-    (IMD/AQI      (Isolation
 Riders,        indexed logs)   cache,        Forest +
 Claims,                        session)      XGBoost)
 Payouts)
     │
Razorpay Sandbox    IMD FREE API    AQI.in FREE    Mock APIs
(UPI payments)      (weather)       (pollution)    (Zepto, News)
```

### Detailed Tech Stack & Justification

#### 1. Frontend: React Native
**Why:** Captures 90%+ of the Indian gig-worker Android market. Enables offline-first experience with background location fetching — critical for verifying a worker's location against environmental disruptions. Native module bridge allows low-level access to `isFromMockProvider()` for GPS spoof detection.

#### 2. Backend: FastAPI (Python)
**Why:** Async-first framework handling 10,000+ concurrent requests during peak weather events (mass simultaneous claim triggers). Python-native ML integration means our Isolation Forest and XGBoost models run in the same process — no inter-service latency for fraud checks.

#### 3. Databases

- **PostgreSQL (Supabase free tier):** Primary relational database. ACID compliance is non-negotiable for the financial ledger — weekly premium deductions, policy state, payout records. Supabase provides managed Postgres with a generous free tier for hackathon scale.
- **PostgreSQL time-indexed GPS table:** GPS logs stored in a partitioned, time-indexed table within the same Postgres instance. No separate TimescaleDB service needed at hackathon scale — partitioning by `created_at` week gives equivalent query performance for our 1M mock trace dataset.
- **Redis (Upstash free tier):** In-memory cache for IMD weather state and Mock Zepto slot lookups. Eliminates repeated external API calls during a rain event when thousands of claims query the same weather state simultaneously.

#### 4. Authentication: Mock Aadhaar OTP via MockAPI

Since this is an **insurance platform**, Aadhaar-based KYC is the industry-standard identity verification method in India (per IRDAI guidelines for micro-insurance). For the hackathon prototype:

- **MockAPI** simulates the Aadhaar OTP flow: rider enters their Aadhaar number → MockAPI returns a simulated 6-digit OTP → verified on backend → JWT issued.
- In production, this would integrate with a licensed Authentication User Agency (AUA) such as NSDL or Digilocker — which insurance platforms are legally permitted to use under the UIDAI framework.
- This approach is **architecturally identical** to real Aadhaar Auth, making the prototype a faithful representation of the production system.

```python
# Mock Aadhaar OTP Flow (MockAPI)
POST /auth/aadhaar/send-otp
{ "aadhaar_last4": "7823", "rider_id": "RV-001" }
→ MockAPI simulates OTP delivery to registered mobile

POST /auth/aadhaar/verify-otp  
{ "otp": "482910", "rider_id": "RV-001" }
→ Returns JWT access token (15min) + refresh token (7 days)
```

#### 5. AI / Machine Learning

- **XGBoost Regressor:** Dynamic weekly premium calculation. Inputs: `[hours_tier, store_zone_risk, imd_forecast_prob, claim_history_score]`. Target: optimal premium in ₹. Trained on synthetic rider-weather data.
- **Isolation Forest:** Fraud anomaly detection across 7 sensor layers. Unsupervised — ideal for our scenario where labeled fraud examples are scarce during early platform rollout.
- **DBSCAN (sklearn):** Spatial clustering for Telegram syndicate detection. Groups simultaneous claims by GPS coordinates to detect coordinated fraud rings.

#### 6. APIs & Integrations

| API | Type | Purpose |
|---|---|---|
| IMD Weather API | Free | T1 (rain), T2 (heat) real-time triggers |
| AQI.in API | Free | T3 (pollution) trigger |
| Mock Zepto API (MockAPI) | Mock | Slot bookings, rider hours, platform downtime (T5) |
| Mock News API (MockAPI) | Mock | T4 curfew/shutdown zone alerts |
| MockAPI Aadhaar OTP | Mock | Rider KYC and authentication |
| Razorpay Sandbox | Sandbox | Weekly UPI premium auto-debit + instant payout |
| MaxMind GeoIP (free tier) | Free | L4 IP geolocation for fraud layer |

#### 7. Scale Performance Targets

| Operation | Latency Target | Capacity | Stack |
|---|---|---|---|
| Fraud Check + Payout | < 20 seconds | Real-time | FastAPI + Redis + Isolation Forest |
| Premium Calculation | < 100ms | 10k/min | FastAPI + Redis cache |
| GPS Log Write | < 50ms | 200k riders | PostgreSQL time-partitioned table |
| Weather State Read | < 5ms | Unlimited | Redis cache (TTL 60s) |

### Security

```
✅ Mock Aadhaar OTP + JWT (access: 15min, refresh: 7 days)
✅ HTTPS enforced (TLS 1.3)
✅ GPS data TTL: 90 days max (DPDP Act 2023 compliant)
✅ Honeypot API endpoints (fraud detection)
✅ Cloudflare DDoS protection (FREE tier)
✅ Rate limiting: 10 claims/rider/day (abuse prevention)
```

---

## 6. AI/ML Integration (Premium + Fraud)

### 🧠 XGBoost Premium Model

Core pricing engine calculates hyper-local weekly premiums aligned with the mandatory **Weekly pricing constraint**.

- **Inputs:** `[hours_tier, store_zone_risk, imd_7day_disruption_prob, claim_history_norm]`
- **Output:** Weekly premium in ₹ (R² = 0.85 target on synthetic validation set)
- **Training Data:** Synthetic rider profiles × historical IMD weather patterns (mock)
- **Update Cycle:** Retrained weekly with latest claim outcomes

#### Predictive Risk Scoring

- **IMD 7-day Forecast Layer:** Fetches rain/heat probability per pincode → feeds disruption probability into premium multiplier
- **Historical Zone Risk:** Maps dark store locations to historical disruption frequency → location loading factor
- **Hyper-local Example:** Ravi's coastal VIZ-003 store has 35% weekly disruption probability in pre-monsoon season → +20% premium loading vs. inland average

### 🔍 Isolation Forest Fraud Model

Unsupervised anomaly detection across 7 sensor layers → single fraud score 0–1.

- **Model:** `sklearn.ensemble.IsolationForest` (contamination=0.01)
- **Features:** `[L1_mock_gps, L2_vibration, L3_tower_match, L4_ip_distance, L5_trajectory, L6_device_novelty, L7_spatial_density]`
- **Retraining:** Weekly on TimescaleDB 30-day rolling window

#### Feature Engineering

```python
def extract_features(claim_data):
    features = np.array([
        claim_data.mock_gps_score,           # L1: 0.0=native GPS, 1.0=mock app
        claim_data.vibration_score,          # L2: 0.0=bike vibration, 1.0=stationary
        claim_data.tower_match_ratio,        # L3: 0–1 (2/3 match = 0.67)
        claim_data.ip_geo_distance_km,       # L4: km from store (0=local, 500=VPN)
        claim_data.trajectory_straightness,  # L5: 0=natural path, 1=GPS circle
        claim_data.device_novelty_score,     # L6: 0=known device, 1=new/mock
        claim_data.spatial_density           # L7: simultaneous claims per 10m radius
    ])
    return features.reshape(1, -1)
```

#### Real-time Fraud Decision

```python
# Correct sklearn IsolationForest scoring
model = IsolationForest(contamination=0.01, random_state=42)

# decision_function returns negative scores for anomalies
raw_score = model.decision_function(features)[0]

# Normalize to [0, 1]: higher = more fraudulent
fraud_score = 1 - (raw_score - model.offset_) / (
    model.decision_function(np.zeros((1, 7)))[0] - model.offset_
)
fraud_score = float(np.clip(fraud_score, 0.0, 1.0))
```

#### FastAPI Claim Endpoint

```python
@app.post("/verify_claim")
async def fraud_check(claim: ClaimRequest):
    features = extract_features(claim)
    fraud_score = score_isolation_forest(features)  # Returns 0–1 float

    if fraud_score < 0.6:
        payout_id = await razorpay.instant_payout(claim.upi_id, claim.payout_amount)
        return {"status": "APPROVED", "fraud_score": fraud_score, "payout_id": payout_id}
    elif fraud_score < 0.85:
        await queue_human_review(claim, fraud_score)
        return {"status": "REVIEW", "fraud_score": fraud_score,
                "message": "Verification in progress. Usually resolves in 2 minutes."}
    else:
        await apply_premium_surcharge(claim.rider_id, surcharge_pct=16)
        return {"status": "BLOCKED", "fraud_score": fraud_score, "reason": "syndicate_pattern"}
```

### 📈 Business Impact Metrics

| Metric | Target | Basis |
|---|---|---|
| Fraud reduction | 95% vs GPS-only | 7-layer multi-signal detection |
| Payout latency | < 20s (P99) | Parallel pipeline design |
| False positive rate | < 2% | Yellow-light human review buffer |
| Platform scale | 200k riders | FastAPI async + Redis |
| ML inference time | < 200ms | sklearn in-process (no microservice hop) |
| Cost per verification | ₹0.10 | Redis + FastAPI serverless compute |

---

## 7. Adversarial Defense & Anti-Spoofing Strategy

### 🚨 OUR RESPONSE: 500-PERSON TELEGRAM GPS SPOOFING SYNDICATE

Simple GPS verification is dead. Here is our production-ready, 7-layer anti-spoofing architecture — designed from Day 1, not bolted on as an afterthought.

### 1. THE DIFFERENTIATION: Genuine Ravi vs. Telegram Spoofer

| Layer | Signal | Genuine Ravi (VIZ-003) | Telegram Fraudster | Detection Logic |
|---|---|---|---|---|
| L1: GPS Authenticity | `isFromMockProvider()` | Native Android GPS — returns FALSE | GPS Joystick app — returns TRUE | Android API flag; impossible to bypass without root |
| L2: Bike Physics | Accelerometer FFT | 2–5 Hz engine vibration signature | 0 Hz — stationary bedroom phone | 60Hz sampling → FFT frequency analysis |
| L3: Cell Towers | `TelephonyManager` | Vizag towers [403xx series] | Hyderabad towers [404xx] — wrong city | 2/3 tower match required for store zone |
| L4: IP Geolocation | MaxMind GeoIP | 59.145.x.x — Vizag ISP | 103.120.x.x — VPN / different city | City-level bounds validation |
| L5: Motion Path | Trajectory analysis | Natural turn radius ~8m at junctions | Perfect GPS circles / straight lines | Trajectory straightness score < 0.95 |
| L6: Device Trust | IMEI + Build hash | Consistent device fingerprint 30 days | New device or mock app signature | Hash vs. 30-day baseline; new device → elevated scrutiny |
| L7: Syndicate Detection | DBSCAN spatial | 1–3 claims per slot, normal spread | 47 simultaneous claims within 10m radius | DBSCAN(eps=0.0001, min_samples=5) → cluster alert |

### 2. DATA POINTS BEYOND GPS (Syndicate Killers)

#### Real-Time Signals (within 20-second pipeline)

- 📡 **CELL TOWERS:** 3 nearest tower IDs must match the store's pre-mapped zone
- 🌐 **CLIENT IP:** MaxMind GeoIP validates city bounds; VPN/proxy IPs flagged
- 📱 **DEVICE FINGERPRINT:** IMEI + `Build.FINGERPRINT` hash checked against 30-day device history
- 🏍️ **ACCELEROMETER:** 60Hz sampling → FFT → 2–5 Hz bike vibration band confirms physical riding
- 🧭 **GYROSCOPE:** Turn radius analysis → realistic motorcycle cornering vs. GPS app circles
- ⏱️ **CLAIM TIMING:** Claims submitted < 2 min after trigger = Telegram coordination pattern
- 👥 **SPATIAL DENSITY:** > 5 devices within 10m radius at claim time = fraud cluster (DBSCAN L7)

#### Historical Baselines (30-day rolling context)

- 🏠 **HOME WIFI SSIDs:** Rider's home network pre-registered → auto-whitelist for pre-slot weather prep
- 📈 **VELOCITY PROFILE:** Historical riding speed 20–30 km/h → 80 km/h at claim time = anomaly
- 📱 **DEVICE HISTORY:** New device appearing mid-monsoon = suspicious; lower auto-approval threshold
- 📊 **CLAIM CADENCE:** First claim this week vs. 5th claim in 3 days = very different risk signals

#### Live Telegram Attack Walkthrough

```
8:02 PM Telegram: "ALL spoof VIZ-003 NOW, rain just started 🤑"

Layer-by-layer detection:
  L1: isFromMockProvider() → TRUE (GPS Joystick detected)         → score: 1.0
  L2: Accelerometer FFT → 0 Hz (no vibration, phone is on table)  → score: 0.9
  L3: Cell towers → 404xx series (Hyderabad, not Vizag)            → score: 0.8
  L7: DBSCAN → 47 simultaneous claims within 10m of VIZ-003        → score: 1.0

Combined Isolation Forest Score: 0.92
→ DECISION: BLOCKED — ₹0 PAID — 16% premium surcharge queued
```

### 3. UX BALANCE: Protecting Honest Workers Without Friction

| Decision | Fraud Score | Response Time | Message to Rider |
|---|---|---|---|
| 🟢 **GREEN — Auto-Approve** | 0.0 – 0.6 | < 20 seconds | "✅ Rain @ VIZ-003 verified. ₹480 sent to UPI!" |
| 🟡 **YELLOW — Soft Review** | 0.6 – 0.85 | 2 min (auto re-check) | "🔍 Verifying your location. Takes ~2 min in heavy rain." |
| 🔴 **RED — Block** | > 0.85 | Instant | "🚨 Claim could not be verified. Premium adjusted." |

#### Smart Edge Case Handling (Protecting Honest Ravi)

| Edge Case | Problem | Our Solution |
|---|---|---|
| Poor signal in heavy rain | GPS unreliable, L3/L4 weaker | Bypass L3/L4, rely on L2 (vibration) + L7 (density). Auto-retry after 90s. |
| Rider at home before shift | Home location looks like "not at store" | Home WiFi SSID pre-registered → L3 score set to 0.0 (whitelisted) |
| New rider (< 30 days) | No historical baseline yet | Lower auto-approve threshold: 0.7 instead of 0.85 until baseline builds |
| Offline mode (no internet) | Cannot send features to backend | Cache last 5 min of sensor readings → submit when connection restored, within 10-min grace window |
| Network drop during rain | Intermittent connectivity | 2-minute grace period → auto re-submit → if still unresolved, route to human review |

### 4. Technical Implementation Snippets

##### L1: GPS Mock Detection (React Native → Kotlin Bridge)
```kotlin
val locationManager = getSystemService(LOCATION_SERVICE) as LocationManager
val isMocked = location.isMock  // Android API 31+ (replaces deprecated isFromMockProvider)
val mock_score = if (isMocked) 1.0f else 0.0f
```

##### L2: Bike Vibration (react-native-sensors)
```javascript
accelerometer.subscribe(({ x, y, z }) => {
  const magnitude = Math.sqrt(x*x + y*y + z*z) - 9.8  // remove gravity
  vibrationBuffer.push(magnitude)
  if (vibrationBuffer.length >= 60) {  // 1 second at 60Hz
    const variance = computeVariance(vibrationBuffer)
    vibrationBuffer.shift()
    // 2–5Hz bike engine signature: variance > 0.5
    vibration_score = variance > 0.5 ? 0.0 : 0.9
  }
})
```

##### L3: Cell Tower Match (Android Java)
```java
List<CellInfo> cells = telephonyManager.getAllCellInfo();  // Requires ACCESS_FINE_LOCATION
Set<Integer> currentTowerIds = extractTowerIds(cells);
Set<Integer> storeTowerIds = getStoreTowers("VIZ-003");  // [40401, 40402, 40403]
double matchRatio = intersection(currentTowerIds, storeTowerIds).size() / 3.0;
double tower_score = matchRatio >= 0.67 ? 0.0 : (1.0 - matchRatio);
```

##### L7: DBSCAN Syndicate Detection (Python / FastAPI)
```python
from sklearn.cluster import DBSCAN
import numpy as np

def detect_syndicate(recent_claims, window_seconds=60):
    coords = np.array([[c.lat, c.lon] for c in recent_claims])
    # eps=0.0001 degrees ≈ 10 metres at Indian latitudes
    db = DBSCAN(eps=0.0001, min_samples=5).fit(coords)
    labels = db.labels_
    
    valid_clusters = labels[labels >= 0]
    if len(valid_clusters) == 0:
        return 0.0
    
    max_cluster_size = max(np.bincount(valid_clusters))
    # Score: 1.0 when cluster size >= 20, scales linearly
    return float(min(1.0, max_cluster_size / 20.0))
```

---

## 8. Financial Model & Sustainability 

### Loss Ratio Validation

A sustainable parametric platform must demonstrate that expected payouts are covered by premiums collected. Below is our viability check for the coastal Vizag market.

| Parameter | Value | Source |
|---|---|---|
| Active riders at VIZ-003 zone | ~640 riders | Estimated Zepto Vizag network |
| Average weekly premium (Pro Tier) | ₹156 | Our pricing model |
| Weekly premium pool | ₹99,840 | 640 × ₹156 |
| Vizag rain disruption events/month | ~6 events | IMD historical data |
| Average payout per event per rider | ₹480 | 4 hrs × ₹150 × 80% |
| % of riders active during event | ~30% | Slot-booking overlap estimate |
| Monthly expected payout | ₹55,296 | 6 events × 192 riders × ₹480 |
| Monthly premium collected | ₹3,19,488 | 4 weeks × ₹99,840 |
| **Estimated Loss Ratio** | **~17%** | Payouts / Premiums |

> **Interpretation:** An estimated 17% loss ratio leaves significant headroom for operating costs, reinsurance buffer, and catastrophic weather events. Even tripling the disruption frequency to 18 events/month would still yield a ~51% loss ratio — well within insurer viability thresholds (typically < 70% target for micro-insurance).

> **Catastrophic week safeguard:** If a cyclone simultaneously halts all 640 riders for 4+ days, maximum pool exposure = ₹2.46M. Platform maintains a 2-week premium reserve (₹1.99M) as liquidity buffer, supplemented by standard parametric reinsurance in production.

### Weekly Premium Model Summary

```
Premium Collection:  Every Sunday 8 AM — Razorpay auto-debit
Coverage Window:     Monday 00:00 → Sunday 23:59
Payout Trigger:      Real-time (within 20s of confirmed disruption)
Payout Channel:      Razorpay Instant Settlement → UPI VPA
Claim Cap:           1 payout per trigger event per rider (per day)
Max Weekly Payout:   Tier coverage ceiling (₹4k / ₹6k / ₹8k)
```

---

## 9. Development Plan (6 Weeks) 

### Phase 1 — Seed [March 4–20]: Ideation & Foundation ✅

**Theme:** Know Your Delivery Worker

| Deliverable | Status |
|---|---|
| README.md (this document) | ✅ Complete |
| GitHub Repository | ✅ https://github.com/PraneethKulukuri26/Necromancer-GuideWire |
| 2-Minute Pitch Video | ✅ https://youtu.be/9xus4h3XnXw |
| Live Prototype (minimal UI) | ✅ nicromancer-demo.vercel.app |

**Key decisions locked:**
- Persona: Zepto Q-Commerce, coastal Vizag + inland Hyderabad contrast
- Platform: React Native mobile
- 5 parametric triggers defined with thresholds
- 7-layer anti-spoofing architecture designed

---

### Phase 2 — Scale [March 21–April 4]: Automation & Protection

**Theme:** Protect Your Worker

**Deliverables:**
- 2-minute demo video
- Executable source code demonstrating:
  - Rider onboarding with Mock Aadhaar OTP KYC
  - Policy creation with weekly tier assignment
  - Dynamic XGBoost premium calculation API
  - Claims management with rule-based fraud thresholds (Isolation Forest in Phase 3)
  - 3–5 automated parametric triggers (IMD + AQI + Mock Zepto APIs)

**Task Ownership (3-person team):**

| Member | Phase 2 Focus |
|---|---|
| Kulukuri Praneeth (Lead) | FastAPI backend — auth, premium API, trigger engine |
| Dodda Sai Prudhvi Raja | React Native app — onboarding, dashboard, UPI flow |
| Vasani Roshan Anantha Sai | ML models — XGBoost premium, mock data generation, PostgreSQL schema |

**Phase 2 Scope Discipline:**
- Fraud detection in Phase 2 = rule-based thresholds (fast to build)
- Isolation Forest model = Phase 3 only
- React Native = core screens only; polish in Phase 3

---

### Phase 3 — Soar [April 5–17]: Scale & Optimise

**Theme:** Perfect for Your Worker

**Deliverables:**
- Advanced Isolation Forest fraud detection (trained on 1M synthetic GPS traces)
- DBSCAN syndicate detection live in claim pipeline
- Razorpay Sandbox instant payout integration
- Worker Dashboard: earnings protected, active coverage, claim history
- Admin/Insurer Dashboard: loss ratios, predictive disruption analytics, fraud map
- 5-minute walkthrough demo video (live rainstorm simulation → auto-payout)
- Final Pitch Deck (PDF)

**Mock Data Generation Plan (Week 1 of Phase 3):**
```python
# Synthetic GPS trace generator — unblocks all ML training
def generate_legitimate_trace(store_lat, store_lon, duration_mins):
    # Random walk around store with realistic 20–30 km/h speed
    # Add 2–5 Hz accelerometer vibration signature
    # Assign local tower IDs from store zone map
    ...

def generate_syndicate_attack(store_lat, store_lon, n_fraudsters=50):
    # n phones clustered within 10m radius
    # All show 0 Hz accelerometer (stationary)
    # All report same GPS coordinate with slight jitter
    ...
```

---

### APIs Confirmed for All Phases

| API | Phase | Role |
|---|---|---|
| IMD Weather (free) | P1–P3 | T1 rain + T2 heat triggers |
| AQI.in (free) | P1–P3 | T3 pollution trigger |
| MockAPI — Zepto | P1–P3 | Slot bookings, rider hours, T5 downtime |
| MockAPI — News | P2–P3 | T4 curfew/shutdown alerts |
| MockAPI — Aadhaar OTP | P2–P3 | Rider KYC / authentication |
| Razorpay Sandbox | P2–P3 | Weekly UPI premium + instant payout |
| MaxMind GeoIP (free tier) | P2–P3 | L4 IP fraud detection |

---

## 10. Team Necromancer

| Name | Role | Email |
|---|---|---|
| **Kulukuri Praneeth** | Team Leader / Backend | `2300090274csitelge@gmail.com` |
| **Dodda Sai Prudhvi Raja** | Frontend / React Native | `2300032835cseird@gmail.com` |
| **Vasani Roshan Anantha Sai** | ML / Data / Infra | `roshanananthasai.v@gmail.com` |

---

*Built for India's gig workers — zero manual claims, 7-layer fraud-proof, scales to 1M riders.*