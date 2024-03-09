![Polars](https://raw.githubusercontent.com/pola-rs/polars-static/master/logos/polars_github_logo_rect_dark_name.svg)
# Polars: The Blazingly Fast DataFrames Library (A quick guide ⚡)
Polars is a [DataFrame](https://en.wikipedia.org/wiki/Dataframe "A DataFrame is a 2-dimensional data structure that is useful for data manipulation and analysis.") implementation that focuses on maximizing performance
and utilizing your machine's resources efficiently. It has an intuitive & expressive API (Syntax) that'll empower you to write code which is both readable and performant.

It supports a wide range of data formats (read/write operations) including:
- Text: CSV & JSON
- Binary: [Parquet](https://en.wikipedia.org/wiki/Apache_Parquet "Parquet is a column-oriented data storage format"), [AVRO](https://en.wikipedia.org/wiki/Apache_Avro "Avro is a row-based data serialization format/system"), Excel etc.
- [IPC](https://en.wikipedia.org/wiki/Inter-process_communication "inter-process communication (IPC), are the mechanisms provided by an operating system for processes to manage shared data."): [Feather](https://arrow.apache.org/docs/python/feather.html "Feather is a portable file format for storing Arrow tables or data frames (from languages like Python or R) that utilizes the Arrow IPC format internally."), [Arrow](https://arrow.apache.org/docs/format/Columnar.html "The “Arrow Columnar Format” includes a language-agnostic in-memory data structure specification, a protocol for serialization and generic data transport. Arrow defines an inter-process communication (IPC) mechanism to transfer a collection of Arrow columnar arrays (called a “record batch”)")
- Databases: MySQL, Postgres, SQL Server, Sqlite, Redshift & Oracle
- Cloud storage: S3, Azure Blob & Azure File

## Polars Lazy API
With the lazy API Polars doesn't run each query line-by-line but instead processes the full query end-to-end.
To get the most out of Polars it is important that you use the lazy API

## Some benefits of Polars Lazy API (LazyDataFrame)
- The lazy API allows Polars to apply automatic query [optimization](https://docs.pola.rs/user-guide/lazy/optimizations/) with the query optimizer.
- The lazy API allows you to work with larger than memory datasets using streaming.
- The lazy API can catch shema errors before processing the data


Note: The collect() method starts the execution and collects the result into a dataframe. Essentially, this method instructs Polars to eagerly execute the query ("collect" the results of the query).

## Using the LazyAPI
We can use the Lazy API starting from either a file or and existing DataFrame

### Using the lazy API from a file
We can use the lazy API starting from a file by calling the files respective ```pl.scan_*``` function

The ```pl.scan_*``` function is available for a number of file types including CSV, IPC, Parquet and JSON.

### Using the lazy API from a DataFrame
An alternative way to access the lazy API is to call .lazy on a DataFrame that has already been created in memory e.g.
```python
q3 = pl.DataFrame({"foo": ["a", "b", "c"], "bar": [0, 1, 2]}).lazy()
```


## Installation
```python
pip install polars
```
Or in Jupyter Notebook
```python
!pip install polars
```

## Check version
```python
import polars as pl
print(pl.__version__)
```

## Reading and writing data (CSV):
```python
import polars as pl
# read from file
df = pl.read_csv('data.csv', try_parse_dates=True)
print(df)

# Or from a BytesIO object
from io import BytesIO
data = BytesIO(
    b"ID,Name,School,Track,Circle\n"
    b"1,Morgan,Data,DE,31\n2,Precious,Data,DE,31\n"
    b"3,Ezekiel,Data,DE,31\n4,Gift,Data,DE,31\n"
    b"5,Fabulouz,Data,DE,31\n6,Uchenna,Data,DE,31\n"
    b"7,Angela,Data,DE,31\n8,Faisal,Data,DE,31\n"
    b"9,Olanrewaju,Data,DE,31\n10,Rshomide4,Data,DE,31\n"
)

df = pl.read_csv(data, try_parse_dates=True)
print(df)

# write to CSV file
df.write_csv('output.csv')

```

## Reading data (Parquet):
```python
loan_risk = pl.scan_parquet("https://tinyurl.com/5eduaare")  # Lazily read remote parquet file
print(loan_risk.collect().shape)  # print shapes
print(loan_risk..collect().head(10))  # print the first 10 rows
```

## Filtering
```python
non_debtors = loan_risk.filter(  # non-debtors
    pl.col('paid_amnt').eq(pl.col('funded_amnt'))
).collect()

print(non_debtors)

paid_1h_3h = loan_risk.filter( # those who have made between 100.0 and 300.0 in loan repayments
    pl.col('paid_amnt').is_between(100.0, 300.0)
).collect()

print(paid_1h_3h)

ca_debtors = loan_risk.filter( # debtors from California
    (pl.col('paid_amnt') < pl.col('funded_amnt')) & (pl.col('addr_state') == 'CA')
).collect()

print(ca_debtors)
```

## Creating new columns
```python
new_df = loan_risk.with_columns([
    pl.col('funded_amnt').mean().alias('avg_loan_amnt'),
    pl.col('paid_amnt').mean().alias('avg_paid_amnt')
]).collect()

print(new_df)
```

## Groupby
```python
# aggregates by state
groupby = loan_risk.group_by('addr_state', maintain_order=True).agg([
    pl.col('funded_amnt').mean().alias('avg_funded_amnt_by_state'),
    pl.col('funded_amnt').max().alias('max_funded_amnt_by_state'),
    pl.col('funded_amnt').min().alias('min_funded_amnt_by_state')
]).collect()

print(groupby)
```

## Combining dataframes
```python
import numpy as np
from datetime import datetime, timedelta

df = pl.LazyFrame({
    "a": np.arange(0, 8),
    "b": np.random.rand(8),
    "c": [datetime(2023, 3, 9) + timedelta(days=idx) for idx in range(8)],
    "d": [1, 2.0, np.NaN, np.NaN, 0, -5, -42, None]
}).collect()

df2 = pl.LazyFrame({
    "x": np.arange(0, 8),
    "y": ['A', 'A', 'A', 'B', 'B', 'C', 'X', 'X']
}).collect()

joined_df = df.join(df2, left_on='a', right_on='x')  # ~ join on 'a' == 'x'

print(joined_df)
```

## SQLContext
```python
# Create an SQLContext from the "loan_risk" LazyFrame.
result = pl.SQLContext(frame=loan_risk).execute(
    "SELECT funded_amnt FROM frame WHERE funded_amnt >= 3000"
).collect()

print(result)
```

## Benchmarks
- [DuckDB Labs](https://duckdblabs.github.io/db-benchmark/)
- [TPCH](https://pola.rs/posts/benchmarks/)