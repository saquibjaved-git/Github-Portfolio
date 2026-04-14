# PRD: Multi-Restaurant Cart

## 1. Problem Statement & Value Proposition

### Problem
- Users abandon cart when items unavailable from single restaurant
- Average delivery time 45+ mins disincentivizes small orders
- Competitive disadvantage vs. aggregators supporting multi-vendor carts
- Geographic fragmentation reduces order frequency in underserved areas

### Value Proposition
- **AOV Growth**: 30-40% increase through bundled multi-restaurant orders
- **Reduced Abandonment**: Complete cart fulfillment across nearby vendors
- **Fleet Efficiency**: Optimized routing reduces per-order delivery cost by 15-20%
- **Market Expansion**: Enable ordering in zones with sparse restaurant density

---

## 2. User Stories

### Group Ordering
- **US-01**: As a group coordinator, I want to add items from 2-3 restaurants to one cart so that I split costs and single delivery trip consolidates orders
- **US-02**: As a group member, I want to see itemized breakdown by restaurant and contributor so that settlement is transparent
- **US-03**: As an organizer, I want to modify quantities/remove items from others' selections before checkout so that I control final cart

### Convenience & Discovery
- **US-04**: As a busy user, I want the app to auto-suggest complementary restaurants (dessert after main) based on cart contents so that I save browsing time
- **US-05**: As a user in low-density area, I want to search "pizza + dessert" and see available multi-restaurant combos so that I don't abandon cart due to unavailability
- **US-06**: As a cost-conscious user, I want to compare delivery routes showing total time/cost for single vs. multi-restaurant orders so that I make informed choice

### Operational
- **US-07**: As a restaurant owner, I want visibility into multi-restaurant order breakdowns so that I manage prep time coordination
- **US-08**: As a delivery partner, I want optimized route with clear pickup sequence so that I complete multi-stop delivery efficiently

---

## 3. Functional Specifications

### 3.1 Discovery & Eligibility Logic

| Component | Logic | Constraints |
|-----------|-------|-------------|
| **Restaurant B Selection** | Suggests restaurants within 2km radius OR on existing delivery route from Restaurant A | Distance measured via routing engine; override requires explicit user action |
| **Eligibility Gate** | Restaurants B & C must share ≥1 delivery zone | Prevents fragmented logistics |
| **Cart Composition** | Max 3 restaurants per cart | Prevents rider overload; sunset rule enforced at checkout |
| **Inventory Sync** | Real-time availability checks on secondary restaurants during 5-sec confirmation window | Fallback: auto-remove if item goes OOS |
| **Geo-Fencing** | Multi-restaurant option visible only if 2+ eligible restaurants detected within 2km buffer | Hidden otherwise; UX fallback to single-restaurant |

### 3.2 Cart UI/UX

| Element | Specification |
|---------|---------------|
| **Restaurant Sections** | Collapsible cards per restaurant; sub-total + prep time per section |
| **Distance Badge** | Shows "2.4km" or "On Route" for secondary restaurants; red flag if >2km or NOT on route (requires opt-in) |
| **Delivery Route Preview** | Interactive map shows: Restaurant A → Restaurant B → User; ETA on each leg |
| **Prep Time Display** | Max(Prep_A, Prep_B) + 5 min buffer; sequential prep shown with visual timeline |
| **Multi-Taxation UI** | Taxes/fees grouped by restaurant; clear GST breakdowns per vendor; no "hidden fees" allowed |
| **Itemized Checkout** | Name of each restaurant, items, quantity, restaurant subtotal → Grand Total |

### 3.3 Multi-Taxation Engine

| Tax Type | Handling |
|----------|----------|
| **GST (18% / 5% / 12%)** | Applied per restaurant based on food category; not cumulative across restaurants |
| **Platform Fee** | Calculated as % of mega-order value OR flat fee per restaurant; capped at max(Restaurant A fee, Restaurant B fee) |
| **Delivery Fee** | Single fee covering multi-restaurant route; if Individual A+B > Combined, display savings |
| **Packaging Charges** | Per-restaurant, aggregated in "Bags & Packaging" line |
| **Restaurant Discount** | Applied per restaurant independently; cumulative discounts calculated pre-delivery fee |

---

## 4. Logistics Engine

### 4.1 Rider Batching

| Phase | Rule |
|-------|------|
| **Assignment** | Match multi-restaurant order to rider with capacity for 2+ stops within service window |
| **Sequencing** | Optimize: Restaurant A (longer prep) → wait → Restaurant B (shorter prep) → User; minimize idle time |
| **Conflict Resolution** | If Restaurant prep times differ by >15 min, offer user two options: (a) wait for second restaurant, (b) split into 2 deliveries |
| **Capacity Check** | Rider vehicle capacity ≥ sum of bag counts from both restaurants; reject multi-restaurant if single-rider capacity insufficient |

### 4.2 ETA Calculation

| Component | Formula | Parameters |
|-----------|---------|------------|
| **Pickup ETA to A** | Base ETA + assignment buffer (60 sec) | Current rider location → Restaurant A |
| **Pickup ETA to B** | Pickup_A_ETA + Prep_A + Drive(A→B) + Buffer(60 sec) | Drive distance via routing API; buffer accounts for ordering/packing |
| **Delivery ETA** | Pickup_B_ETA + Prep_B + Drive(B→User) | Assumes sequential pickup |
| **Total Order Time** | Checkout → Delivery_ETA | Shown to user at cart acceptance |
| **Variance Buffer** | +3 min per restaurant (prep uncertainty) + 2 min per km driven | Communicated as "Delivery by HH:MM" with ±5 min confidence band |

---

## 5. Edge Cases & Mitigation

| Edge Case | Scenario | Resolution |
|-----------|----------|-----------|
| **Restaurant OOS During Confirmation** | Item becomes unavailable at secondary restaurant in 5-sec window | Auto-remove item; show notification; calculate revised totals; keep cart live |
| **Prep Time Conflict (>15 min delta)** | Restaurant A prep = 30 min, Restaurant B prep = 10 min | Explicitly offer user: (1) wait bundled, (2) pre-deliver A, (3) split orders. Default = bundled if <25 min wait. |
| **Rider No-Show / Cancellation** | Assigned rider cancels after 2 stops locked | Reassign to new rider; if reassignment delays ETA >20 min, offer ₹100 credit or order cancellation |
| **Partial Cancellation** | User removes Restaurant B items post-acceptance but pre-pickup | System recalculates delivery route; refund delivery fee differential if savings >₹30 |
| **Secondary Restaurant Cancellation** | Restaurant B decline order mid-confirmation | Revert to single-restaurant mode; notify user, keep Restaurant A + auto-retry alternatives |
| **Route Eligibility Changed** | Restaurant B moves >2km OR goes outside zone | Flag as "No longer eligible"; block checkout; offer single-restaurant fallback |
| **Delivery Zone Mismatch** | Restaurant A in Zone 1, Restaurant B in Zone 2 (different delivery partners) | Prevent at eligibility gate; show error "Cannot combine restaurants from different zones" |
| **High Load Surge Pricing** | Multi-restaurant order triggers 3x surge; user rejects | Allow un-bundling: split into 2 single orders with regular pricing; document user preference for future bundled discounts |

---

## 6. Key Performance Indicators (KPIs)

### Primary Metrics

| KPI | Target | Measurement | Tracking Cadence |
|-----|--------|-------------|------------------|
| **Average Order Value (AOV) Growth** | +35% for multi-restaurant orders vs. baseline single-restaurant | (Multi-Restaurant Total Revenue) / (Multi-Restaurant Order Count) | Daily |
| **Multi-Restaurant Adoption Rate** | 25% of orders use multi-restaurant cart within 6 months | (Orders with 2+ restaurants) / (Total orders) | Weekly |
| **Order Completion Rate** | >92% without partial cancellations | (Completed Orders) / (Initiated Orders) | Daily |
| **Average Delivery Time (Multi-Restaurant)** | <42 min end-to-end | Checkout → Delivery timestamp | Daily |
| **Delivery Time Delta (vs. Single-Restaurant)** | ≤+8 min premium | (Multi-Restaurant ETA) - (Single-Restaurant ETA) | Weekly |

### Secondary Metrics

| KPI | Target | Measurement | Tracking Cadence |
|-----|--------|-------------|------------------|
| **Customer Satisfaction (Multi-Restaurant)** | ≥4.6/5.0 stars | Order-level feedback; exclude single-restaurant orders | Weekly |
| **Cart Abandonment Rate (Multi-Restaurant)** | <15% | (Abandoned Multi-Carts) / (Initiated Multi-Carts) | Daily |
| **Prep Time Accuracy** | ±5 min variance (80th percentile) | |Actual prep vs. promised| ≤ 5 min for 80% of orders | Daily |
| **Rider Utilization** | 1.8+ orders per rider (multi-stops) | (Multi-Stop Deliveries) / (Active Riders) | Weekly |
| **Restaurant Participation** | 60% of restaurants in eligible zones opt-in | (Restaurants with multi-cart enabled) / (Eligible restaurants) | Bi-weekly |

### Financial Metrics

| KPI | Target | Measurement | Tracking Cadence |
|-----|--------|-------------|------------------|
| **Delivery Cost Efficiency Gain** | -18% cost per order vs. 2 separate deliveries | (Combined Multi-Delivery Cost) vs. (2× Single-Delivery Cost) | Weekly |
| **Incremental Revenue (Feature)** | +₹12 Cr revenue over 6 months | (Multi-Restaurant GMV) - (Cannibalized single-restaurant orders) | Monthly |
| **Rider Margin Impact** | +12% gross margin per multi-stop delivery | (Revenue per delivery) / (Cost per delivery) excluding cannibalization | Monthly |

### Go/No-Go Criteria (Week 8 Review)

| Metric | Pass Threshold | Action if Failed |
|--------|----------------|------------------|
| Adoption Rate | ≥12% | Extend beta; audit UI/incentives |
| AOV Growth | ≥+20% | Proceed to full rollout |
| Delivery Time Delta | ≤+12 min | Optimize batching logic; proceed with monitoring |
| Completion Rate | ≥88% | Fix cancellation root causes; pause if <85% |

---

## 7. Success Criteria & Rollout

### Phase 1: Closed Beta (Weeks 1-4)
- 500 users; 2-3 metros (e.g., Bangalore, Delhi)
- Target: 5% multi-restaurant order penetration

### Phase 2: Open Beta (Weeks 5-8)
- 50K users; expand to 8 metros
- Target: 15% penetration; validate KPIs

### Phase 3: Full Rollout (Week 9+)
- All metros; geo-targeting by eligibility
- Target: 25%+ penetration; sustain <42 min delivery time
