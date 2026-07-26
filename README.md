# Churn Risk Analysis — Weekly Contact List

I take subscription billing and usage data and turn it into one thing: a ranked list of exactly which customers to contact before they cancel. This project is built on the Telco Customer Churn dataset from Kaggle.

## Key Results

- **26.58% churn rate** across 7,032 customers (11 rows with zero tenure and missing billing data were dropped as new signups with no churn signal yet).
- Found a clear churn profile: **month-to-month contract, fiber optic internet, electronic check payment, short tenure, high monthly charges.**
- Built a risk score from those five factors and validated it against real outcomes: customers scoring 0 churned **1.2%** of the time historically; customers scoring 8 churned **73.1%** of the time.
- Applied the score to customers who haven't churned yet, producing a ranked, tiered contact list: **536 customers to call this week**, 1,570 this month, 3,057 to monitor.

## Repo Structure

```
churn-risk-analysis/
├── data/
│   ├── WA_Fn-UseC_-Telco-Customer-Churn.csv   (raw dataset)
│   └── active_customers_with_risk_score.csv   (scored, active customers only)
├── notebooks/
│    churn_eda.ipynb                      (full analysis, scoring, validation)
├── dashboard/
│   └── dashboard.pbix
├── screenshots/
|   ├── segment_view.png
│   └── risk_score_view.png
└── README.md
```

## Process & Methodology

1. **Cleaned the data.** `TotalCharges` loaded as text; converting it to numeric revealed 11 rows with missing values. All 11 had zero tenure and had not churned — brand-new signups with no billing history yet — so they were dropped rather than filled, since they carried no churn signal.
2. **Explored churn by segment.** Checked churn rate against gender, senior citizen status, contract type, internet service, and payment method. Gender showed almost no difference (26.9% vs 26.2%). Contract type showed the widest gap by far: 42.7% churn on month-to-month vs 2.8% on two-year contracts.
3. **Found the shape of two continuous features.** Binned tenure and monthly charges into ranges and checked churn rate per bin, instead of relying on averages alone. Tenure showed a real cliff — churn nearly halves between the 0–12 and 12–24 month groups. Monthly charges rose more gradually, with no sharp break, but crossed 32% churn at the $60+ mark.
4. **Built a risk score.** Assigned tiered points to tenure and contract (since both showed a cliff shape), and flat single points to internet service, payment method, senior citizen status, and monthly charges (since those showed flatter or more gradual risk). Max possible score: 8.
5. **Validated the score against real outcomes** before trusting it. Grouped all customers by score and checked actual historical churn rate per group — confirmed it climbed cleanly from 1.2% to 73.1% with no reversals.
6. **Built the contact list.** Filtered to active customers only (excluding anyone who already churned), applied the same score, and split into three tiers using the two biggest jumps in the validation data: 0–3 (monitor), 4–5 (call this month), 6–8 (call this week). Sorted by score descending, then tenure ascending, so newest and most urgent customers surface first — new customers sit on the steepest part of the churn curve, and saving them earlier protects more lifetime value.
7. **Built two dashboards in Power BI.** A segment-level view (churn by contract, gender, internet service, payment method) and a risk-score view (the tiered contact list itself, filterable by tier).

## Case Study

**The problem.** I had the Telco dataset and asked it straight questions: churn rate, and whether it was tied to contract type, gender, senior citizen status, revenue, tenure. Most of it confirmed the obvious. Contract type didn't — it was the widest gap of anything I checked.

**What I did and decided.** I dug into tenure, charges, and internet service for month-to-month customers and found a specific, repeatable profile: short tenure, higher charges, electronic check, fiber optic. I built a segment dashboard around it. Then I looked at that dashboard and saw the real gap — it told me which segments churn more, not who to call. A chart isn't a list. So I turned the same factors into a validated risk score and applied it to customers who hadn't churned yet, ranking them into action tiers instead of leaving them as a chart.

**What came of it.** The score held up against real outcomes — 1.2% churn at the bottom, 73.1% at the top. Applied to 5,163 active customers, it produced 536 names to call this week, each with a plain reason attached, not just a number.

**What I'd do differently next time:** build the scored, ranked list from the start, instead of stopping at the segment dashboard and only noticing the gap afterward.

## Tools

Python (pandas, NumPy) for cleaning, scoring, and validation · Jupyter Notebook for the analysis · Power BI Desktop for both dashboards · Git/GitHub for version control.

## How to Run This

1. Clone the repo: `git clone <repo-url>`
2. Install dependencies: `pip install pandas numpy jupyter`
3. Open `notebooks/churn_eda.ipynb` in Jupyter to see the full analysis and scoring logic.
4. Open `dashboard/dashboard.pbix` in Power BI Desktop to explore both dashboard views interactively.

## Limitations

This uses a static Kaggle dataset, not a live company's billing or usage data — there's no real-time refresh, and the churn labels are historical, not predictions. The risk score is a simple, explainable point system, not a machine learning model; it was chosen deliberately so every flagged customer comes with a plain-language reason, not a black-box probability. The five factors used are the ones this specific dataset showed strong signal on — a different subscription business would need its own validation pass before trusting the same weights.
