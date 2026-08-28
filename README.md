## MIT805 AML Transaction Analysis

This project explores a large financial transaction dataset labelled for
possible money laundering. Part 1 covers data conditioning, data cleaning, and
exploratory data analysis (EDA) using Polars and Matplotlib.

### Project workflow

The notebooks form a sequential pipeline:

1. **Data conditioning** — `notebooks/part1_data_conditioning.ipynb` lazily
	reads `data/LI-Large_Trans.csv`, converts `Timestamp` to datetime, and
	writes `data/filtered_transactions.parquet`. Records on or after
	2022-11-06 are excluded because the later period does not retain the same
	mixture of laundering and normal transactions.
2. **Data cleaning** — `notebooks/part1_data_cleaning.ipynb` renames the
	account columns, checks missing values and categorical values, identifies
	exact duplicates among repeated-timestamp candidates, adds a `Transfer
	Type` feature, and writes
	`data/filtered_cleaned_transactions.parquet`.
3. **EDA** — `notebooks/part1_eda.ipynb` reads the cleaned Parquet file and
	examines dataset composition, transaction amounts, temporal behaviour,
	payment formats, currencies, transfer types, laundering rates, bank-level
	patterns, repeated account pairs, and reciprocal account-pair behaviour.

The data files, their schemas, row counts, source attribution, and processing
notes are documented in [`data/README.md`](data/README.md).

### Dataset used by the analysis

The final EDA input, `filtered_cleaned_transactions.parquet`, contains
176,060,109 transactions and 12 columns. It covers 2022-08-01 through
2022-11-05 and includes the following fields:

- transaction time, sending and receiving bank IDs;
- sending and receiving account IDs;
- amount paid and amount received;
- payment and receiving currencies;
- payment format;
- the supplied `Is Laundering` label; and
- the derived `Transfer Type` feature.

The label is strongly imbalanced: 96,938 rows are labelled laundering and
175,963,171 are labelled non-laundering in the cleaned analysis dataset.
Therefore, laundering rates and group comparisons should be interpreted with
the class imbalance in mind.

### Setup and execution

From the project root on macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Open the notebooks in VS Code or Jupyter and run them in the order listed
above. The notebooks use relative paths such as `../data/...`, so run them
from their normal location in the `notebooks` directory. The raw transaction
file is very large; ensure sufficient disk space and memory before rebuilding
the derived files.