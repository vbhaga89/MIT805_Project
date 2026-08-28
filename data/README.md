## Data

This directory contains the transaction and account data used by the MIT805
project notebooks, together with the intermediate datasets produced by the
conditioning and cleaning steps.

### Source data

The transaction data is from IBM's anti-money-laundering (AML) transaction
dataset:

- `LI-Large_Trans.csv` — 176,066,557 transaction records.

### Transaction data

`LI-Large_Trans.csv` has 11 columns:

| Column | Description |
| --- | --- |
| `Timestamp` | Transaction date and time, stored as `YYYY/MM/DD HH:MM` text in the raw CSV |
| `From Bank` | Sending bank identifier |
| `Account` | Sending account identifier in the raw file |
| `To Bank` | Receiving bank identifier |
| `Account_duplicated_0` | Receiving account identifier in the raw file |
| `Amount Received` | Amount received |
| `Receiving Currency` | Currency received |
| `Amount Paid` | Amount paid |
| `Payment Currency` | Currency paid |
| `Payment Format` | Payment method or format |
| `Is Laundering` | Target label: `0` for non-laundering and `1` for laundering |

The raw transaction file contains no missing values. Its timestamp range is
2022-08-01 00:00 to 2023-01-12 10:18. The label distribution is highly
imbalanced: 100,604 laundering records and 175,965,953 non-laundering records.

### Derived datasets

The notebooks create the following compressed Parquet files:

#### `filtered_transactions.parquet`

Created by `notebooks/part1_data_conditioning.ipynb`.

- 176,060,273 rows and 11 columns.
- `Timestamp` converted from text to a Polars datetime.
- Includes records from 2022-08-01 through 2022-11-05.
- Rows with timestamps on or after 2022-11-06 are excluded so that the
	analysis retains a meaningful mixture of laundering and normal activity.

#### `filtered_cleaned_transactions.parquet`

Created by `notebooks/part1_data_cleaning.ipynb` from the filtered Parquet
file.

- 176,060,109 rows and 12 columns.
- Renames `Account` to `From Account` and `Account_duplicated_0` to `To Account`.
- Adds `Transfer Type`, classified as `Self-Transfer`, `Cross-Bank Transfer`,
	or `Standard Transfer`.
- Checks for missing values, validates categorical columns, and removes exact
	duplicate rows from repeated-timestamp candidates while keeping the first
	occurrence.
- Contains no missing values in the generated file.

### Rebuilding the files

Run the notebooks in this order from the project root:

1. `part1_data_conditioning.ipynb`
2. `part1_data_cleaning.ipynb`
3. `part1_eda.ipynb`

The notebooks use relative paths and expect the raw CSV files to remain in
this directory. The raw transaction CSV is approximately 176 million rows, so
execution requires substantial memory, storage, and time. Polars lazy scans
and streaming Parquet writes are used to reduce the processing overhead.

### Data-use notes

- The filtering cutoff is a project analysis choice, not a claim that later
records are invalid. 
- The laundering label is treated as the provided target
variable