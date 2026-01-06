
# Chocolate Sales Analysis

A data analysis project and Power BI dashboard focused on exploring chocolate product sales, profitability, and customer behavior to surface actionable business insights.

## Table of Contents
- [Introduction](#introduction)
- [Dataset / Inputs](#dataset--inputs)
- [Tech Stack](#tech-stack)
- [Installation](#installation)
- [Usage](#usage)
- [Analysis & Business Questions](#analysis--business-questions)
- [Results & Outputs](#results--outputs)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## Introduction

This repository contains cleaned transaction and product data used to build a Power BI dashboard that summarizes sales performance, profitability, and trends for chocolate products. The analysis is intended for business analysts and data practitioners who need reproducible datasets and reference code snippets for further exploration.

## Dataset / Inputs

- Canonical cleaned CSVs are stored in the `clean data/` folder. Key files:
	- [clean data/Sales_Transactions.csv](clean%20data/Sales_Transactions.csv) — primary transaction table (~200k rows).
	- [clean data/Products_cleaned.csv](clean%20data/Products_cleaned.csv) — product master (sku, category, mrp, cost).
	- [clean data/Warehouses_cleaned.csv](clean%20data/Warehouses_cleaned.csv) — warehouse lookup used by `fulfilled_by_warehouse_id`.

Notes on the data:
- `Sales_Transactions.csv` contains mixed date formats, currency symbols (₹), comma-thousands separators, and occasional empty numeric fields.
- `product_sku` is the primary join key across files; `return_flag` is Y/N for returns.

## Tech Stack

- Power BI (dashboard & visuals)
- DAX (measures used inside Power BI)
- Python (pandas) — recommended for data cleaning, sampling, and reproducible exports

## Installation

1. Install Power BI Desktop to open the dashboard (if using the PBIX file).
2. For Python-based work, create a virtual environment and install pandas:

```bash
python -m venv .venv
source .venv/bin/activate   # macOS / Linux
.venv\Scripts\activate    # Windows PowerShell
pip install pandas pyarrow
```

## Usage

- Open the Power BI report (if provided) to interact with dashboards and slicers.
- Use Python for repeatable preprocessing or to create additional aggregates. Example: load a sample of transactions:

```py
import pandas as pd
tx = pd.read_csv('clean data/Sales_Transactions.csv', nrows=1000)
tx['total_amount'] = pd.to_numeric(tx['total_amount'].astype(str).str.replace(r'[₹,]', '', regex=True), errors='coerce')
print(tx.head())
```

## Analysis & Business Questions

Typical analyses and questions this project supports:
- What are total sales, profit, and profit percentage by product and category?
- Year-over-year and month-over-month trends for sales and volume.
- Top and bottom performing chocolate SKUs by revenue and margin.
- Region / warehouse level performance using `fulfilled_by_warehouse_id`.
- Impact of promotions (`promo_code`) on average selling price and discount amounts.

## Results & Outputs

- Human-readable dashboards and images are stored under `Report/`.
- Machine-ready aggregated datasets or exports should go to `Result/` (use timestamped Parquet filenames for reproducibility).

## Project Structure

Top-level layout:

- [clean data/](clean%20data) — canonical CSV inputs (transactions, products, warehouses, returns, inventory snapshots)
- [RAW_DATA/](RAW_DATA) — original/raw dumps (if present)
- [Report/](Report) — dashboard exports and narrative reports
- [Result/](Result) — processed datasets and parquet outputs
- LICENSE — project license
- README.md — this file

## Contributing

- Open an issue to discuss changes before sending pull requests.
- Keep data edits minimal and deterministic; include a short README or script when adding preprocessing code.
- Avoid adding heavy binary files to the repo; prefer `Result/` or external storage for large artifacts.

## License

See the repository `LICENSE` file for license details.

## Contact

For questions or suggestions, open an issue on this repository or contact the maintainers (placeholder-varunmathur138@gmail.com).


