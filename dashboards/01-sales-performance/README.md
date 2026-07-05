# Sales Performance Command Center

An interactive analytics dashboard that helps sales leaders identify **where revenue and margin are changing**, **what is driving performance**, and **which actions to prioritize this month**.

## 1. Business Question
How can sales leadership improve revenue growth and margin by region, segment, and product while controlling discount-related profit erosion?

## 2. Audience
- VP of Sales
- Regional Sales Managers
- Revenue Operations / Finance Partners

## 3. Data Source
- Source: Sample retail transactions dataset (compatible with Superstore-style schema)
- Grain: Order-line level
- Typical fields:
  - `order_date`
  - `region`
  - `segment`
  - `category`
  - `sub_category`
  - `sales`
  - `profit`
  - `discount`
  - `customer_id`

> Place your input CSV at: `data/raw/sales_data.csv`

## 4. KPI Definitions
- **Total Revenue** = `SUM(sales)`
- **Total Profit** = `SUM(profit)`
- **Profit Margin %** = `SUM(profit) / SUM(sales) * 100`
- **Average Order Value (AOV)** = `SUM(sales) / COUNTD(order_id)` *(or row count fallback)*
- **Discount Rate %** = `AVG(discount) * 100`
- **YoY Revenue Growth %** = `(Revenue_current_period - Revenue_prior_period) / Revenue_prior_period * 100`

## 5. Dashboard Views
1. **Executive Overview**
   - KPI cards (Revenue, Profit, Margin, Discount Rate)
   - Monthly revenue and profit trend
   - Regional performance comparison
2. **Product Deep Dive**
   - Category/Sub-category contribution
   - Profit vs discount relationship
   - Underperforming product flags
3. **Customer & Segment View**
   - Revenue by segment
   - Top customers by sales
   - Repeat activity indicators

## 6. Key Insights (Example Narrative)
1. High-discount sub-categories show strong revenue but below-target margin.
2. One region contributes disproportionate profit decline despite flat sales.
3. A small cohort of customers drives a large share of total revenue.

## 7. Decisions Enabled
- Adjust discount strategy for margin-negative SKUs.
- Rebalance sales focus toward high-margin regions and segments.
- Launch retention and upsell plans for top-value customer cohorts.

## 8. Business Impact (Example)
- Identified discount-heavy SKUs causing margin drag; pricing actions projected to recover margin by **3–5 percentage points** in flagged categories.
- Regional re-prioritization identified a path to improve quarterly profit mix.

## 9. Screenshots
Add screenshots after publishing your dashboard outputs:

- `assets/overview.png`
- `assets/product-deep-dive.png`
- `assets/customer-segment-view.png`

## 10. How to Reproduce
### Option A — Streamlit (included)
1. Create and activate a virtual environment
2. Install dependencies:
   ```bash
   pip install -r requirements.txt

```python name=dashboards/01-sales-performance/dashboard/app.py url=https://github.com/ardenaraiche-svg/ardenaraiche-svg/blob/main/dashboards/01-sales-performance/dashboard/app.py
import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import streamlit as st
from pathlib import Path

st.set_page_config(page_title="Sales Performance Command Center", layout="wide")

st.title("📊 Sales Performance Command Center")
st.caption("Revenue, Profit, Margin, Discount, and Performance Drivers")

# -----------------------------
# Data loading
# -----------------------------
BASE_DIR = Path(__file__).resolve().parents[1]
DEFAULT_DATA_PATH = BASE_DIR / "data" / "raw" / "sales_data.csv"


def _sample_data() -> pd.DataFrame:
    rng = np.random.default_rng(7)
    dates = pd.date_range("2025-01-01", periods=180, freq="D")
    regions = ["East", "West", "Central", "South"]
    segments = ["Consumer", "Corporate", "Home Office"]
    categories = ["Technology", "Office Supplies", "Furniture"]
    subcats = {
        "Technology": ["Phones", "Accessories", "Copiers"],
        "Office Supplies": ["Storage", "Binders", "Paper"],
        "Furniture": ["Chairs", "Tables", "Bookcases"],
    }

    rows = []
    for i in range(1800):
        d = dates[rng.integers(0, len(dates))]
        region = regions[rng.integers(0, len(regions))]
        seg = segments[rng.integers(0, len(segments))]
        cat = categories[rng.integers(0, len(categories))]
        sub = subcats[cat][rng.integers(0, len(subcats[cat]))]

        sales = float(np.clip(rng.normal(350, 180), 20, None))
        discount = float(np.clip(rng.normal(0.18, 0.12), 0, 0.6))
        margin_base = {"Technology": 0.24, "Office Supplies": 0.18, "Furniture": 0.14}[cat]
        margin = margin_base - (discount * 0.22) + rng.normal(0, 0.03)
        profit = sales * margin

        rows.append(
            {
                "order_id": f"ORD-{100000 + i}",
                "order_date": d,
                "region": region,
                "segment": seg,
                "category": cat,
                "sub_category": sub,
                "sales": round(sales, 2),
                "profit": round(float(profit), 2),
                "discount": round(discount, 4),
                "customer_id": f"CUST-{rng.integers(1000, 1500)}",
            }
        )
    return pd.DataFrame(rows)


@st.cache_data
def load_data(uploaded_file):
    if uploaded_file is not None:
        df = pd.read_csv(uploaded_file)
    elif DEFAULT_DATA_PATH.exists():
        df = pd.read_csv(DEFAULT_DATA_PATH)
    else:
        df = _sample_data()

    required = {"order_date", "region", "segment", "category", "sub_category", "sales", "profit", "discount"}
    missing = required - set(df.columns)
    if missing:
        st.error(f"Missing required columns: {sorted(missing)}")
        st.stop()

    df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")
    df = df.dropna(subset=["order_date"]).copy()
    df["sales"] = pd.to_numeric(df["sales"], errors="coerce").fillna(0)
    df["profit"] = pd.to_numeric(df["profit"], errors="coerce").fillna(0)
    df["discount"] = pd.to_numeric(df["discount"], errors="coerce").fillna(0)

    if "order_id" not in df.columns:
        df["order_id"] = np.arange(1, len(df) + 1).astype(str)

    return df


with st.sidebar:
    st.header("Filters")
    uploaded = st.file_uploader("Upload CSV (optional)", type=["csv"])

df = load_data(uploaded)

with st.sidebar:
    min_date, max_date = df["order_date"].min(), df["order_date"].max()
    date_range = st.date_input("Date range", value=(min_date.date(), max_date.date()))

    regions = st.multiselect("Region", sorted(df["region"].dropna().unique()), default=sorted(df["region"].dropna().unique()))
    segments = st.multiselect("Segment", sorted(df["segment"].dropna().unique()), default=sorted(df["segment"].dropna().unique()))
    categories = st.multiselect("Category", sorted(df["category"].dropna().unique()), default=sorted(df["category"].dropna().unique()))

if len(date_range) == 2:
    start_date, end_date = pd.to_datetime(date_range[0]), pd.to_datetime(date_range[1])
else:
    start_date, end_date = df["order_date"].min(), df["order_date"].max()

f = df[
    (df["order_date"] >= start_date)
    & (df["order_date"] <= end_date)
    & (df["region"].isin(regions))
    & (df["segment"].isin(segments))
    & (df["category"].isin(categories))
].copy()

if f.empty:
    st.warning("No data after filters. Adjust selections.")
    st.stop()

# KPIs
revenue = f["sales"].sum()
profit = f["profit"].sum()
margin_pct = (profit / revenue * 100) if revenue else 0
discount_pct = f["discount"].mean() * 100
aov = revenue / f["order_id"].nunique() if f["order_id"].nunique() else 0

k1, k2, k3, k4, k5 = st.columns(5)
k1.metric("Revenue", f"${revenue:,.0f}")
k2.metric("Profit", f"${profit:,.0f}")
k3.metric("Profit Margin", f"{margin_pct:.1f}%")
k4.metric("Avg Discount", f"{discount_pct:.1f}%")
k5.metric("AOV", f"${aov:,.0f}")

# Monthly trend
monthly = (
    f.assign(month=f["order_date"].dt.to_period("M").dt.to_timestamp())
    .groupby("month", as_index=False)[["sales", "profit"]]
    .sum()
)
fig_trend = go.Figure()
fig_trend.add_trace(go.Scatter(x=monthly["month"], y=monthly["sales"], mode="lines+markers", name="Revenue"))
fig_trend.add_trace(go.Scatter(x=monthly["month"], y=monthly["profit"], mode="lines+markers", name="Profit"))
fig_trend.update_layout(title="Monthly Revenue vs Profit", height=360, legend=dict(orientation="h"))

# Region performance
region_perf = f.groupby("region", as_index=False).agg(sales=("sales", "sum"), profit=("profit", "sum"))
fig_region = px.bar(region_perf, x="region", y="sales", color="profit", title="Revenue by Region (color = Profit)")
fig_region.update_layout(height=360)

c1, c2 = st.columns(2)
c1.plotly_chart(fig_trend, use_container_width=True)
c2.plotly_chart(fig_region, use_container_width=True)

# Product deep dive
st.subheader("Product Deep Dive")
prod = f.groupby(["category", "sub_category"], as_index=False).agg(sales=("sales", "sum"), profit=("profit", "sum"), discount=("discount", "mean"))
fig_prod = px.treemap(prod, path=["category", "sub_category"], values="sales", color="profit", title="Sales Contribution by Category/Sub-category")
fig_scatter = px.scatter(prod, x="discount", y="profit", size="sales", color="category", hover_name="sub_category", title="Discount vs Profit by Sub-category")

c3, c4 = st.columns(2)
c3.plotly_chart(fig_prod, use_container_width=True)
c4.plotly_chart(fig_scatter, use_container_width=True)

# Customer and segment view
st.subheader("Customer & Segment View")
seg = f.groupby("segment", as_index=False).agg(sales=("sales", "sum"), profit=("profit", "sum"))
fig_seg = px.bar(seg, x="segment", y="sales", color="profit", title="Sales by Segment")

cust = f.groupby("customer_id", as_index=False).agg(sales=("sales", "sum")).sort_values("sales", ascending=False).head(10)
fig_cust = px.bar(cust.sort_values("sales"), x="sales", y="customer_id", orientation="h", title="Top 10 Customers by Sales")

c5, c6 = st.columns(2)
c5.plotly_chart(fig_seg, use_container_width=True)
c6.plotly_chart(fig_cust, use_container_width=True)

# Insight callouts
st.subheader("Auto-generated Insight Starters")
worst_margin = prod.assign(margin=np.where(prod["sales"] > 0, prod["profit"] / prod["sales"], 0)).sort_values("margin").head(3)
top_region = region_perf.sort_values("profit", ascending=False).head(1)

a = worst_margin.iloc[0]
b = top_region.iloc[0]

st.markdown(
    f"""
- Lowest-margin sub-category in current filter: **{a['sub_category']}** ({a['category']})
- Top profit region: **{b['region']}** with **${b['profit']:,.0f}** profit
- Current average discount: **{discount_pct:.1f}%**; monitor high-discount categories for margin pressure
"""
)

st.caption("Tip: Replace sample data with your own CSV at data/raw/sales_data.csv for portfolio screenshots.")
streamlit>=1.37.0
pandas>=2.2.2
numpy>=2.0.0
plotly>=5.23.0
