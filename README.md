## MIT805 AML Transaction Analysis

This project analyses a large financial transaction dataset labelled for
possible money laundering. The workflow is split across two parts:

- **Part 1**: Polars-based conditioning, cleaning, and exploratory data
  analysis (EDA) using Python and Matplotlib.
- **Part 2**: PySpark-based distributed analytics and MapReduce-style
  processing for larger-scale grouped and reduced transaction analysis.

### Project workflow

The notebooks form a sequential pipeline for the full analysis:

1. **Data conditioning** — `notebooks/part1_data_conditioning.ipynb` lazily
   reads `data/LI-Large_Trans.csv`, converts `Timestamp` to datetime, and
   writes `data/filtered_transactions.parquet`. Records on or after
   2022-11-06 are excluded because the later period does not retain the same
   mixture of laundering and normal transactions.
2. **Data cleaning** — `notebooks/part1_data_cleaning.ipynb` renames the
   account columns, checks missing values and categorical values, identifies
   exact duplicates among repeated-timestamp candidates, adds a `Transfer Type`
   feature, and writes `data/filtered_cleaned_transactions.parquet`.
3. **EDA** — `notebooks/part1_eda.ipynb` reads the cleaned Parquet file and
   examines dataset composition, transaction amounts, temporal behaviour,
   payment formats, currencies, transfer types, laundering rates, bank-level
   patterns, repeated account pairs, and reciprocal account-pair behaviour.
4. **PySpark MapReduce analysis** — `notebooks/part2_pyspark_mapreduce.ipynb`
   loads the cleaned dataset, recreates the `Transfer Type` mapping in Spark,
   runs grouped aggregations and explicit RDD-based MapReduce examples, and
   studies how Spark shuffles and reduces data for AML analysis.

The data files, their schemas, row counts, source attribution, and processing
notes are documented in [`data/README.md`](data/README.md). The LaTeX reports are in `report/`.

### Dataset used by the analysis

The primary dataset for both the Part 1 and Part 2 notebooks is
`data/filtered_cleaned_transactions.parquet`. It contains 176,060,109
transactions and 12 columns. It covers 2022-08-01 through 2022-11-05 and
includes the following fields:

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

From the project root:

macOS / Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Windows (PowerShell):

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Open the notebooks in VS Code or Jupyter and run them in the order listed
above. The Part 1 notebooks use relative paths such as `../data/...`, so run them
from their normal location in the `notebooks` directory. The raw transaction
file is very large; ensure sufficient disk space and memory before rebuilding
the derived files. For the Part 2 notebook, a cloud notebook environment with PySpark is required to execute the distributed
MapReduce examples. For this project, Colab was used. As a result, `DATA_PATH` should be specified
to where your cleaned data (parquet file) would be located on your Google Drive.