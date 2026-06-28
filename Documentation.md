# UK Railway Ticket Sales Analysis
### Project Documentation

**Team Innovators — DEPI Capstone Project**
Business Understanding · Data Cleaning · EDA · Hypothesis Testing · Power BI

01. Mohamed Elsayed
02. Abdulrahman Abdulrazek
03. Gaber Alaa
04. Ahmed Fawzy

*Consolidated and quality-reviewed documentation covering the full project lifecycle, from raw data to dashboard.*

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Business Understanding](#2-business-understanding)
3. [Data Understanding](#3-data-understanding)
4. [Data Cleaning & Quality Assurance](#4-data-cleaning--quality-assurance)
5. [Exploratory Data Analysis — Key Findings](#5-exploratory-data-analysis--key-findings)
6. [Hypothesis Testing](#6-hypothesis-testing)
7. [Power BI Architecture](#7-power-bi-architecture)
8. [Assumptions and Limitations](#8-assumptions-and-limitations)
9. [Summary of Recommendations](#9-summary-of-recommendations)

---

## 1. Executive Summary

This document consolidates the full UK Railway Ticket Sales Analysis project — business understanding, data cleaning, exploratory data analysis, hypothesis testing, and the Power BI architecture — into a single, verified reference. All figures in this report were re-computed directly against the final cleaned dataset (31,653 transactions, January–April 2024) to ensure internal consistency across every section.

**Three headline findings anchor the analysis:**

- **Discounting is the single largest drag on revenue.** Tickets sell, on average, for 40.8% below their pre-discount value — far larger than the 5.2% lost to refunds.
- **Refund behavior is driven by perceived fault, not inconvenience.** Delays attributed to internal causes (Technical Issues) trigger refunds at 54.9%, versus 9.7% for Weather — a difference confirmed statistically significant.
- **Two specific routes** (Liverpool–London Euston, Manchester–London Euston) carry disruption rates of 70–80%, roughly six times the network average — but this is **not** a peak-hour effect; the same routes are equally disrupted at all hours of the day.

A data-quality audit prior to analysis identified and corrected three issues in the original cleaned dataset that would otherwise have produced incorrect downstream metrics: a stray text artifact in 1,880 cells, invalid negative journey durations for 961 overnight services, and a duplicate station name affecting 332 transactions. These are documented in full in Section 4.2.

---

## 2. Business Understanding

### 2.1 Business Problem Statements

The project addresses three problem statements (PS), each owned by a different business function:

1. **PS01 — Service & Refunds:** Analyze service delays to identify the main reasons customers request ticket refunds, in order to reduce financial losses caused by delays.
2. **PS02 — Pricing & Discounts:** Analyze ticket pricing and purchase patterns to evaluate the impact of the advance-purchase discount on revenue, given that a large share of customers buy in advance.
3. **PS03 — Demand & Capacity:** Analyze ticket purchases by station and time to identify peak demand periods, in order to reduce crowding and inform ticket pricing.

### 2.2 Business Process

A customer selects a route (departure to arrival station) and a ticket type (Advance, Off-Peak, or Anytime), in either Standard or First Class, and completes the purchase online or at a station. The system records the transaction and schedules the journey. On the journey date, actual arrival time is compared against the scheduled time to determine Journey Status (On Time / Delayed / Cancelled). If a disruption occurs, the customer may submit a Refund Request through customer service.

### 2.3 Stakeholders

| Stakeholder | Interest | Type |
|---|---|---|
| Sales Management | Accurate pricing strategies to maximize revenue | Internal |
| Operations Management | Reducing operational pressure and optimizing workflow | Internal |
| Marketing Team | Fine-tuning discount structures and promotions | Internal |
| Customer Service | Reducing refund requests; improving satisfaction | Internal |
| Government | Compliance with regulation, taxation, legal frameworks | External |
| Passengers | Reliable service, fair pricing, smooth experience | External |

### 2.4 Business Questions & KPIs by Function

#### Sales Management

**KPIs:**
- Gross Sales, Net Revenue (after refunds), Average Ticket Price
- Discount Rate % (revenue-weighted), Price Realization % (Price / Price before discount)
- Total Loss from Refund, Total Loss from Discount

**Key questions:**
- Which Purchase Type and Ticket Type generate the most revenue, and how does this compare in average ticket value?
- What share of total revenue comes from the top 5 routes? (Concentration risk)
- Which Railcard × Ticket Type combination drives the highest discount leakage relative to its revenue share?
- How does refund activity trend over time, and which routes drive it?

#### Operations

**KPIs (tracked per day):**
- Refund Request Rate %, Delay Rate %, On-Time Rate %, Cancellation Rate %
- Average Delay (minutes), Total Journeys, Peak Departure Hour

**Key questions:**
- Which reasons for delay drive the most refund requests, and is the relationship statistically significant?
- Which specific routes carry disproportionate disruption, and is it time-of-day driven or route-specific?
- What is the expected (corrected) journey time for each line?
- What is the typical lead time between ticket purchase and journey, by Ticket Type?

#### Marketing & Passengers

**KPIs:**
- Total Transactions, Railcard Penetration %, Disabled Railcard %
- Online Purchase Ratio, Average Ticket Price by segment

**Key questions:**
- What is the distribution of Railcard types among passengers, and does it affect ticket class choice?
- How does payment method vary by purchase channel (Online vs. Station)?
- Is there a relationship between payment method and refund request behavior?

---

## 3. Data Understanding

### 3.1 Data Sources

| Data Source | Description | Key Columns |
|---|---|---|
| Railway Transactions | Individual ticket purchase records | Date/Time of Purchase, Purchase Type, Price, Refund Request |
| Stations | Departure and arrival station information | Departure Station, Arrival Destination |
| Journey | Scheduled and actual journey timing | Date of Journey, Departure/Arrival Time, Actual Arrival Time |
| Ticket | Class, type, and pricing information | Ticket Class, Ticket Type |
| Passenger | Railcard and payment method details | Railcard, Payment Method |
| Delay | Journey status and reasons for delay | Reason for Delay, Journey Status |

### 3.2 Data Dictionary (Source / Pre-Cleaning)

The table below reflects the dataset as received, before cleaning. Two known issues in the dictionary's own wording are flagged below; these are documentation issues in the original source, not cleaning errors.

| Field | Description | Example |
|---|---|---|
| Transaction ID | Unique identifier per purchase | `da8a6ba8-b3dc-4677-b176` |
| Date / Time of Purchase | When the ticket was bought | `2023-12-08` / `12:41:11` |
| Purchase Type | Online or Station | `Online`, `Station` |
| Payment Method | Contactless, Credit Card, Debit Card | `Contactless` |
| Railcard | Adult/Senior/Disabled holder, or none. 1/3 off ticket price | `Adult`, `None` |
| Ticket Class | Standard or First | `First Class`* |
| Ticket Type | Advance (1/2 off, ≥1 day ahead), Off-Peak (1/4 off, outside 6–8am/4–6pm weekdays), Anytime (full price) | `Advance` |
| Price | Final cost charged | `43` |
| Departure Station / Arrival Destination | Boarding / exit stations | `London Paddington` |
| Date / Departure / Arrival Time | Scheduled journey timing (arrival may fall the next day) | `2024-01-01` / `11:00:00` / `13:30:00` |
| Actual Arrival Time | Actual arrival (may fall the next day) | `13:30:00` |
| Journey Status | On Time, Delayed, or Cancelled | `On Time` |
| Reason for Delay | Cause of delay or cancellation | `Signal Failure` |
| Refund Request | Whether a refund was requested after disruption | `Yes` / `No` |

*\* The dictionary's field description says "Standard or First," but the example column — and the data itself — use "First Class." This is a wording inconsistency within the source dictionary, confirmed against the raw data.*

---

## 4. Data Cleaning & Quality Assurance

Cleaning was performed in Python (pandas) and is fully reproducible from `Data_Cleaning.ipynb`. This section documents (4.1) routine cleaning decisions, (4.2) three critical issues discovered through direct verification against the data — not visible from a casual review — and (4.3) the engineered columns added for analysis.

### 4.1 Standard Cleaning Steps

- **Railcard:** 20,918 of 31,653 records (66.1%) had no Railcard value. Per the data dictionary, a blank here means "no card," not a data error, so these were recoded as "No Railcard."
- **Reason for Delay:** missing only for On Time journeys (logically expected, since there is no delay to explain) and filled as "No Delay." Cancelled and Delayed rows always carried a real reason — confirmed with zero exceptions.
- **Inconsistent raw labels** were standardized to one canonical value each: "Signal failure" / "Signal Failure" → Signal Failure; "Staffing" → Staff Shortage; "Weather" → Weather Conditions.
- **Actual Arrival Time** is missing by definition for all 1,880 Cancelled journeys (the train never ran) — not a data quality issue, and excluded from delay-time calculations for that status.

### 4.2 Critical Issues Found and Corrected

The three issues below were not visible from summary statistics or a glance at the file. Each was confirmed by direct, row-level verification against the dataset before being corrected.

> #### Issue 1 — Literal "NaT" text in Actual Arrival Time
> **Found in 1,880 rows (all Cancelled journeys)**
>
> A formatting step converted all time columns to text using Python's `str.replace("0 days ", "")`. For the 1,880 missing (Cancelled) values, this produced the literal text string "NaT" instead of a blank cell.
>
> **Risk:** pandas' default missing-value list does **not** include the string "NaT", so re-loading this file in pandas, Excel, or Power BI would silently treat these 1,880 cells as real text data rather than nulls — breaking any downstream filter or aggregation on this column.
>
> **Fix:** replace the literal string "NaT" with a true empty value before final export.

> #### Issue 2 — Negative Journey Duration for overnight services
> **Found in 961 rows (3.04% of all transactions)**
>
> Journey Duration was computed as Arrival Time minus Departure Time using time-of-day values only, with no allowance for the journey crossing midnight — even though the data dictionary explicitly states arrival can fall on the day after departure.
>
> **Verification:** adding exactly 1,440 minutes (24 hours) to every negative value produced durations that matched, to the minute, the normal duration of the same route on its non-overnight departures (e.g. London Kings Cross → York: corrected to 110 minutes, identical to the route's standard duration). This confirms a day-rollover bug, not a data error.
>
> **Fix:** where Journey Duration < 0, add 1,440 minutes.

> #### Issue 3 — Duplicate station name ("Edinburgh" / "Edinburgh Waverley")
> **Found in 332 rows**
>
> "Edinburgh" and "Edinburgh Waverley" appear as two distinct values in Arrival Destination, despite referring to the same physical station (confirmed via satellite map — identical building, platforms, and location).
>
> **Verification:** for the same origin (e.g. York), both labels produced an identical journey duration (150 minutes), confirming they represent one route recorded under two names.
>
> **Risk:** any Route-level aggregation (Top N lines, revenue or disruption by route) will undercount this destination unless the two labels are merged first.
>
> **Fix:** standardize all destination values to "Edinburgh Waverley."

### 4.3 Engineered Features

The following columns were derived for analysis. Each formula was validated against all 31,653 rows with zero arithmetic exceptions.

| Column | Formula / Logic |
|---|---|
| Journey Duration | Arrival Time − Departure Time, corrected for day-rollover (Issue 2) |
| Delay in Minutes | Actual Arrival Time − Arrival Time (null for Cancelled journeys) |
| Route | Departure Station + " to " + Arrival Destination (after Issue 3 correction) |
| Lead Time | Date of Journey − Date of Purchase, in days |
| Ticket Type Discount | Anytime = 0%, Off-Peak = 25%, Advance = 50% |
| Railcard Discount | 1/3 (33.3%) for Adult/Senior/Disabled; 0% for No Railcard |
| Price before discount | Price ÷ [(1 − Ticket Type Discount) × (1 − Railcard Discount)] — discounts assumed to combine multiplicatively |
| Total Discount % | 1 − [(1 − Ticket Type Discount) × (1 − Railcard Discount)] |
| Price After Refund | Price if Refund Request = No, else 0 (used to compute Net Revenue) |
| Is Peak Departure | True if weekday AND departure hour in 6–8am or 4–6pm (matches the dictionary's exact wording: "weekdays between…") |

---

## 5. Exploratory Data Analysis — Key Findings

All figures below were computed directly from the corrected dataset. Where an earlier draft of the analysis stated a different number or explanation, the correction is noted explicitly.

### 5.1 Sales & Revenue

| Metric | Value |
|---|---|
| Gross Sales | £741,921 |
| Refunded Amount | £38,702 (5.2% of gross) |
| Net Revenue | £703,219 |
| Revenue-weighted Discount Rate | 40.8% (1 − Price / Price-before-discount, summed) |
| Top 5 routes' share of revenue | 64.1% (out of 65 routes) |
| Standard / First Class revenue split | £592,522 (79.9%) / £149,399 (20.1%) |

*Correction note: an earlier draft reported total revenue as £772,487 with a 79.2% / 20.8% Standard / First Class split. Recomputing directly from the Price column gives £741,921 total — the figure also shown on the donut chart in that same draft, confirming £741,921 (not £772,487) is correct.*

**Average price and discount by Ticket Type:**

| Ticket Type | Avg. Price | Avg. Discount | Volume |
|---|---|---|---|
| Advance | £17.61 | 55.6% | 17,561 (55.5%) |
| Off-Peak | £25.52 | 33.1% | 8,752 (27.6%) |
| Anytime | £39.20 | 12.3% | 5,340 (16.9%) |

**Finding:** more than half of all tickets sold (55.5%) are Advance tickets, averaging a 55.6% discount. Combined with the 40.8% revenue-weighted discount rate above, discounting — not refunds — is the larger of the two value leaks identified in PS01/PS02, by roughly 8x.

**Recommendation:** Treat discount policy, particularly Advance-ticket allocation on top-revenue routes, as the primary lever for improving net revenue — ahead of refund-reduction initiatives.

### 5.2 Operations

On-time performance: 86.8% of journeys arrive on time; 7.2% are delayed and 5.9% are cancelled.

**Delay duration and refund rate by cause:**

| Reason for Delay | Avg. Delay (min) | Refund Request Rate |
|---|---|---|
| Technical Issue | 24.9 | 54.9% |
| Traffic | 32.3 | 38.5% |
| Staff Shortage | 51.2 | 32.3% |
| Signal Failure | 51.8 | 22.2% |
| Weather Conditions | 43.8 | 9.7% |

**Finding:** refund likelihood does not track delay duration. Technical Issue causes the shortest average delay (24.9 min) but the highest refund rate (54.9%); Weather Conditions causes a longer delay (43.8 min) but the lowest refund rate (9.7%). Passengers appear to respond to perceived fault (was this the company's fault?) rather than to the length of the inconvenience. This pattern is tested formally in Section 6.

#### Route-level disruption: a route-specific issue, not a peak-hour effect

| Route | Trips | Disruption Rate |
|---|---|---|
| Liverpool Lime Street → London Euston | 1,097 | 80.1% |
| Manchester Piccadilly → London Euston | 345 | 70.4% |
| Birmingham New Street → Manchester Piccadilly | 224 | 50.9% |
| **Network average (all routes)** | 31,653 | 13.2% |

> **Correction: the "peak-hour" explanation is not supported by the data**
>
> An earlier draft attributed this disruption concentration to the 9:00am and 4:00pm commuter peaks, recommending budget reallocation specifically for those hours.
>
> Direct test on the Liverpool–Euston route: disruption rate is 79.4% during the dictionary's defined peak windows (weekday 6–8am / 4–6pm) versus 77.6% outside them — a negligible difference. Hour-by-hour, the same route shows 7.1% disruption at 9am but 100% at 4pm and 8am — an erratic pattern inconsistent with a true peak-hour cause.
>
> **Conclusion:** these two routes carry a severe, structural disruption problem present at all hours, not a congestion effect. The original recommendation to invest in infrastructure/signaling on this corridor still holds — only the stated cause changes.

Network-wide peak-hour test: a formal test across the full network (Section 6.2) confirms peak-hour departures show no meaningful difference in disruption likelihood compared to off-peak departures — reinforcing that the Liverpool/Manchester–Euston pattern is route-specific, not a general peak-hour phenomenon.

### 5.3 Marketing & Passenger Behavior

**Railcard distribution:**

| Railcard | Transactions | Share |
|---|---|---|
| No Railcard | 20,918 | 66.1% |
| Adult | 4,846 | 15.3% |
| Disabled | 3,089 | 9.8% |
| Senior | 2,800 | 8.8% |

**Finding:** Railcard status has no meaningful effect on Ticket Class choice. Holders and non-holders both purchase First Class at ~9.5–10% of the time (formally tested in Section 6.1, p = 0.155 — not statistically significant).

**Payment method by purchase channel:**

| Purchase Type | Contactless | Credit Card | Debit Card |
|---|---|---|---|
| Online | 34.5% | 62.0% | 3.5% |
| Station | 33.8% | 58.3% | 7.9% |

**Finding:** Credit Card dominates both channels, but Debit Card usage is more than twice as common at stations (7.9%) as online (3.5%) — consistent with in-person buyers more often using everyday debit payments, while online buyers skew toward credit.

---

## 6. Hypothesis Testing

Two relationships raised in the EDA were tested formally with a chi-square test of independence, rather than judged from percentage differences alone. This distinguishes a real, statistically supported effect from a difference that is plausible-looking but not significant once sample size is accounted for.

### 6.1 Railcard status and Ticket Class — no real relationship

H₀: Ticket Class choice is independent of Railcard status. H₁: Railcard holders show a distinct Ticket Class preference.

| Statistic | Value |
|---|---|
| χ² | 2.02 |
| p-value | 0.155 |
| Cramér's V (effect size) | 0.008 (near zero) |
| Conclusion | Fail to reject H₀. No statistically significant relationship. |

With n = 31,653, even a trivial difference (9.49% vs. 10.00% First Class share) can look meaningful by eye. The test confirms it is not: Railcard status should not be used as a segmentation variable for Ticket Class marketing.

### 6.2 Peak-hour departure and journey disruption — statistically detectable, practically negligible

H₀: Journey Status is independent of whether departure falls in a peak window. H₁: Peak departures are more likely to be disrupted.

| Statistic | Value |
|---|---|
| χ² | 9.76 |
| p-value | 0.0076 |
| Cramér's V (effect size) | 0.018 (near zero) |
| Conclusion | Statistically significant, but the effect size is negligible — not practically useful. |

This is the network-wide counterpart to the route-level finding in Section 5.2: at n = 31,653, the test detects the tiny true difference between peak and off-peak disruption rates (87.1% vs. 86.7% on-time) as statistically real, but an effect this small has no operational value. This directly contradicts the "peak-hour infrastructure collapse" narrative in earlier drafts, and supports treating Liverpool–Euston / Manchester–Euston as a route-specific issue (Section 5.2), not a time-of-day one.

### 6.3 Reason for Delay and Refund Request — confirmed significant

H₀: The likelihood of a refund request depends only on delay duration, regardless of cause. H₁: Refund likelihood is driven by perceived fault (liability), independent of duration.

| Statistic | Value |
|---|---|
| χ² | 533.8 |
| p-value | < 0.001 (3.3 × 10⁻¹¹⁴) |
| Cramér's V (effect size) | 0.358 (moderate — not negligible) |
| Conclusion | Reject H₀. The relationship is both statistically significant and practically meaningful. |

Unlike Sections 6.1 and 6.2, this relationship holds up to formal testing with a real effect size, not just a large sample size. **This is the one finding in the dataset that justifies a behavioral, not just descriptive, recommendation: refund policy and customer-service response should differentiate by cause of disruption (internal fault vs. external/uncontrollable), since passengers themselves already do.**

---

## 7. Power BI Architecture

The cleaned dataset (post Section 4 corrections) is loaded directly into Power BI from the project's GitHub repository. Power Query transformations were intentionally minimized, since cleaning was already completed in Python.

### 7.1 Star Schema Design

A star schema was chosen for simpler report development, faster DAX evaluation, reduced duplication, and predictable filtering behavior.

**Fact table — `f_transactions`**
Transaction ID, Route ID, Attributes ID, Purchase Date/Time, Journey Date, Departure/Arrival/Actual Arrival Time, Price, Price Before Discount, Price After Refund, Journey Duration, Lead Time, Delay Minutes, Discounts.

**Dimension tables**
- `d_routes` — Route ID, Departure Station, Arrival Destination, Route Name (built after the Edinburgh/Edinburgh Waverley merge in Section 4.2, Issue 3).
- `d_attributes` — Purchase Type, Payment Method, Railcard, Ticket Class, Ticket Type, Journey Status, Refund Request, Reason for Delay. Originally separate lookup tables; merged into one dimension since the individual tables added relationship overhead without analytical benefit.
- `d_date_purchase` and `d_date_journey` — two independent calendar tables, since Purchase Date and Journey Date represent different business events and a single calendar would create ambiguous filtering.
- `d_time_purchase` and `d_time_journey` — two independent time-of-day tables, supporting purchase-hour analysis and departure/peak-hour analysis respectively without relationship conflicts.

**Relationships**

| From (Fact) | To (Dimension) |
|---|---|
| Attributes ID | d_attributes |
| Route ID | d_routes |
| Purchase Date | d_date_purchase |
| Journey Date | d_date_journey |
| Purchase Time | d_time_purchase |
| Departure Time | d_time_journey |

**Measure tables**
- *Revenue Measures* — Total Revenue, Total Sales, Refund Amount, Discount Loss, Average Ticket Price, Average Discount.
- *Operational Measures* — Delay Rate, Incident Rate, Average Delay, Average Journey Duration, Cancel Rate, Refund Rate, On-Time Rate.
- *Marketing Measures* — Purchase Online Ratio, Railcard Holder Ratio, Station Purchase Ratio, Total Online Purchases, Passenger Counts.

*Keeping measures in dedicated tables, separate from the data model, improves readability and keeps business logic auditable independent of the schema.*

### 7.2 Dashboard Design

Five report pages share a consistent left-side navigation panel and common slicers (Departure Station, Arrival Station, Date Range).

| Dashboard | Purpose | Key Visuals |
|---|---|---|
| Executive Summary | High-level performance overview for leadership | KPI cards, Journey Trend by Hour, Revenue Trend, Top Performing Routes |
| Revenue | Financial performance analysis | Revenue Trend, Revenue by Ticket Class / Type, Top Performing Routes |
| Operational Overview | Operational efficiency monitoring | On-Time vs. Incident Rate by route, Refund Requests by Delay Reason |
| Marketing | Passenger purchasing behavior | Revenue by Ticket Type, Railcard Distribution, Revenue Trend |
| Operational Deep Dive | Reserved for detailed operational drill-down | (extension page) |

Interactive features: cross-filtering, slicers, dynamic KPI cards, navigation buttons, conditional formatting, and time intelligence.

---

## 8. Assumptions and Limitations

### 8.1 Data Cleaning Transformations

- Missing Railcard values (raw "none") were recoded as "No Railcard." The dictionary's example value of "None" describes the raw, pre-cleaning state.
- Missing Reason for Delay values occur only for On Time journeys and were filled as "No Delay."
- Inconsistent raw labels in Reason for Delay were standardized (see Section 4.1).
- Actual Arrival Time is missing by definition for Cancelled journeys, and excluded from delay-time calculations for that status, not treated as a data quality issue.

### 8.2 Known Data Characteristics

- This dataset is a mock/simulated dataset (publicly distributed for analytics training and competitions), not live transactional data from an operating rail company. Discount and ticket-type rules were modeled closely on real UK National Rail fare structures, but individual transaction-level patterns should be read as illustrative rather than literal passenger behavior.
- Two routes (Liverpool–London Euston, Manchester–London Euston) show disruption rates far above the network average at all hours of day — most consistent with a fixed, route-level data characteristic rather than a real-world operational cause.
- "Ticket Class" is recorded as "First Class" (not "First") in the source data, differing from the dictionary's field description text.

### 8.3 Business Logic Assumptions

- Refund Request = Yes is treated as a full, honored refund of Price. The data has no separate "Approved / Rejected" refund status, so "Total Refunded Amount" assumes every request was paid in full.
- Price reflects the amount actually charged (post-discount). Price before discount and Total Discount % are reverse-derived assuming Ticket Type discount and Railcard discount combine multiplicatively (e.g. Advance + Railcard ≈ 66.7% off), since the business has not confirmed whether discounts stack multiplicatively or additively.
- "Peak hours" are defined exactly as the dictionary states — weekdays, 6–8am and 4–6pm. Under this definition, zero Off-Peak tickets in the dataset were used in violation of the stated rule; all apparent violations (under a time-only definition) fell entirely on weekends, when the restriction does not apply.

---

## 9. Summary of Recommendations

1. **Prioritize discount policy over refund policy.** The revenue-weighted discount rate (40.8%) is roughly 8x larger than the refund leakage rate (5.2%). Reviewing Advance-ticket allocation, especially on the top revenue routes, has substantially more financial upside than refund-reduction efforts alone.
2. **Differentiate refund/customer-service response by cause, not duration.** Technical Issue delays trigger refunds at nearly 6x the rate of Weather delays despite being shorter on average — a statistically confirmed effect (Section 6.3). Low-cost, immediate goodwill gestures for internally-caused disruptions are likely to be more cost-effective than blanket policies.
3. **Treat the Liverpool–Euston and Manchester–Euston corridors as a structural, not time-of-day, problem.** Infrastructure investment is justified by the data, but should not be scheduled around "peak hours" — disruption on these routes is consistent across the full day.
4. **Do not use Railcard status as a Ticket Class marketing segment.** The relationship is not statistically significant (Section 6.1); resources aimed at this segmentation are unlikely to move outcomes.
5. **Monitor revenue concentration risk.** The top 5 routes generate 64.1% of total revenue — any operational issue on these specific lines has outsized financial impact and should be prioritized accordingly in service-quality monitoring.
