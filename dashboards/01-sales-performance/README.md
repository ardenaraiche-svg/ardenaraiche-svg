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
