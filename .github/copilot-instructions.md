Purpose
-------
This repository is data-first (sales & inventory). These instructions orient an AI coding agent to the project's structure, common data-quality issues, and typical analysis patterns so you can be productive immediately.

Quick repo map
- Data sources: [clean data](clean%20data) (canonical CSVs), [RAW_DATA](RAW_DATA) (original dumps).
- Working outputs: [Report](Report) and [Result](Result).

Files to inspect first
- [clean data/Sales_Transactions.csv](clean%20data/Sales_Transactions.csv): primary transaction table (200k+ rows).
- [clean data/Products_cleaned.csv](clean%20data/Products_cleaned.csv): product master with categories, mrp, cost.
- [clean data/Warehouses_cleaned.csv](clean%20data/Warehouses_cleaned.csv): warehouse lookup used by `fulfilled_by_warehouse_id`.

Project-specific facts and gotchas
- This repo contains CSV data only (no code). Most work is data-processing and reporting.
- Expect inconsistent formats in `Sales_Transactions.csv`: mixed date formats (e.g. "01/22/2025", "13-08-2024 21:23:00", "26 Feb 2025"), currency symbols (₹), comma-thousands separators and empty values in numeric columns.
- Column patterns: `product_sku` joins across files, `return_flag` is Y/N for returned transactions, `promo_code` maps to discounts in `discount_amount`.
- Filenames: most cleaned files use *_cleaned.csv, but `Sales_Transactions.csv` is the principal transactions file and may not follow the suffix convention.

Recommended agent behavior / coding patterns
- Use Python + pandas for quick iterations. Prefer streaming/chunked reads for large files: `pd.read_csv(..., chunksize=100_000)`.
- Normalize numeric columns before aggregation. Example to clean prices:

```py
df['unit_price'] = (
    df['unit_price'].astype(str)
      .str.replace(r'[₹,]', '', regex=True)
      .str.strip()
)
df['unit_price'] = pd.to_numeric(df['unit_price'], errors='coerce')
```

- Robust date parsing: coerce invalids and keep original for inspection.

```py
df['transaction_date_parsed'] = (
    pd.to_datetime(df['transaction_date'], errors='coerce', infer_datetime_format=True)
)
```

- Joins and common aggregation flow:
  - Merge `Sales_Transactions.csv` (transactions) with `Products_cleaned.csv` on `product_sku`.
  - Use `fulfilled_by_warehouse_id` to join warehouses for location-level metrics.
  - Filter out `return_flag == 'Y'` when calculating net revenue unless doing return analysis.

Small reproducible example (revenue by category):

```py
tx = pd.read_csv('clean data/Sales_Transactions.csv', usecols=['product_sku','quantity','total_amount','return_flag'])
prod = pd.read_csv('clean data/Products_cleaned.csv', usecols=['product_sku','category'])
tx['total_amount'] = pd.to_numeric(tx['total_amount'].astype(str).str.replace(r'[₹,]', '', regex=True), errors='coerce')
tx = tx[tx['return_flag'] != 'Y']
out = tx.merge(prod, on='product_sku', how='left').groupby('category', observed=True)['total_amount'].sum()
```

Performance and memory
- Files can be large (Sales_Transactions >200k rows). Use `chunksize`, `dtype` hints, and `usecols` to limit memory.
- Avoid loading all CSVs unnecessarily; prefer streaming joins or preprocessing smaller dimension tables (`Products_*`, `Warehouses_*`) fully and streaming transactions.

Output conventions
- Write human-readable reports to `Report/` and machine-ready datasets to `Result/`.
- Use deterministic, timestamped filenames for outputs (e.g. `Result/transactions_2025-01-06.parquet`). Prefer Parquet for intermediate results when preserving schema and speed matters.

PR / edit guidance for agents
- Make minimal, isolated changes to repository (do not add heavy dependencies without asking).
- When adding scripts, include a short README snippet showing how to run the script and example input/output paths.

When you are unsure
- If a requested analysis lacks a clear expected output, ask for a one-line example (e.g. "expected table: category | revenue | units_sold").
- For ambiguous cleaning rules (e.g. what to do with zero or negative `quantity`), state your assumption and implement it as toggleable code (comment or parameter).

Contact / next steps
- After creating or updating a script, run it on a 1k-row sample and attach the sample output for review.

End of instructions
