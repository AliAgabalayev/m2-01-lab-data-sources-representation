![logo_ironhack_blue 7](https://user-images.githubusercontent.com/23629340/40541063-a07a0a8a-601a-11e8-91b5-2f13e4e6b441.png)

# Lab | Data Sources, Structure, and Representation

## Overview

In this lab you will work hands-on with real data in different formats, from different sources, and with different levels of structure. You will inspect raw files, reason about their origins and schema assumptions, compare row-oriented and column-oriented representations, measure the concrete differences between text and binary formats, and make deliberate choices about how to represent data for downstream use. The goal is to build practical intuition for the decisions data engineers and ML practitioners face every day when data first arrives in their systems.

This is not a "follow the recipe" lab. At every step you will need to think about why a format or representation was chosen, what trade-offs it introduces, and how a different choice would affect performance, storage, or usability. By the end, you will have a notebook that demonstrates clear reasoning about data sources, formats, layouts, and structure — the foundational skills for everything that follows in data engineering.

## Learning Goals

By the end of this lab, you should be able to:

- Identify different data source types (user input, system-generated, internal) and describe their characteristics
- Work with JSON, CSV, and Parquet formats programmatically and explain each format's trade-offs
- Demonstrate the performance difference between row-major and column-major access patterns
- Measure and compare file sizes between text and binary formats
- Distinguish structured from unstructured data and reason about where schema responsibility lies
- Make and justify representation choices for a given downstream use case

## Setup and Context

You will work in a Jupyter Notebook throughout this lab. All data will be generated or created inline — there are no external datasets to download. This is intentional: by building data yourself, you develop a deeper understanding of format mechanics than you would by loading a pre-made file.

## Requirements

- Fork this repository to your own GitHub account.
- Clone your fork to your machine.
- Make sure you can open and run Jupyter notebooks (for example via JupyterLab or VS Code).
- You will need `pandas`, `numpy`, and `pyarrow` (for Parquet support). Install them if needed:

```bash
pip install pandas numpy pyarrow
```

## Getting Started

- Create a new notebook and name it `m2-01-data-sources-representation-lab.ipynb`.
- Complete all tasks in order within this notebook.
- Use markdown cells between tasks to document your reasoning — short explanations of what you observe and why.
- Before you submit, restart your kernel and run the notebook **top to bottom** to make sure it executes cleanly.

## Tasks

### Task 1: Identify and Classify Data Sources

In this task you will create realistic sample data representing different source types and reason about their characteristics.

**Step 1.** Create a Python dictionary representing a single user input event — the kind of data that comes from a web form or mobile app. Include at least five fields such as `username`, `email`, `age`, `bio`, and `signup_source`. Intentionally introduce realistic data quality issues: make the `age` field a string instead of an integer, include extra whitespace in the `email`, and make the `bio` field excessively long (over 500 characters). Print the dictionary and write a markdown cell explaining what makes user input data challenging to work with.

**Step 2.** Create a list of at least 10 dictionaries representing system-generated log entries. Each log entry should have fields like `timestamp`, `service_name`, `level` (e.g., INFO, WARNING, ERROR), `message`, `response_time_ms`, and `request_id`. Make the data realistic: most entries should be INFO level, a couple should be WARNING, and one should be ERROR. Write a markdown cell comparing these log entries to the user input from Step 1 — what is different about their reliability, format consistency, and processing urgency?

**Step 3.** Create a list of at least 8 dictionaries representing records from an internal product database. Each record should have fields like `product_id`, `name`, `category`, `price`, `stock_quantity`, and `supplier_id`. All records should follow a consistent schema. Write a markdown cell explaining what distinguishes internal database data from the other two sources, and describe a scenario where an ML system would need to combine data from all three sources to make a decision.

### Task 2: Working with JSON — Flexibility and Limitations

This task explores JSON as a data format, focusing on its flexibility and pain points.

**Step 1.** Take the user input dictionary from Task 1 and serialize it to a JSON string using `json.dumps()`. Then deserialize it back with `json.loads()`. Verify that the round-trip preserves all values by comparing the original dictionary to the deserialized one. Print both and note any differences.

**Step 2.** Create two JSON documents representing the same real-world entity (for example, a product listing) but with different schemas. The first document should be highly structured with separate fields for every attribute. The second should store the same information as a single unstructured text blob. For example:

```python
# Structured
structured_product = {
    "name": "Wireless Headphones",
    "brand": "SoundMax",
    "price_usd": 79.99,
    "weight_grams": 250,
    "colors": ["black", "white", "navy"]
}

# Unstructured
unstructured_product = {
    "description": "SoundMax Wireless Headphones, $79.99, 250g, available in black, white, and navy"
}
```

Write code that extracts the price from both representations. For the structured version, this should be trivial. For the unstructured version, write a small parser (using string methods or a regular expression) that attempts to find the price. In a markdown cell, discuss which representation would be better for (a) displaying to a human, (b) filtering products by price in an automated system, and (c) storing in a system where the product schema changes frequently.

**Step 3.** Create a list of 5 JSON documents where each document has a slightly different schema — for instance, some documents have a `discount` field and others do not, some have `ratings` as a list and others as a single number. Write a function `normalize_records(records)` that takes this list and returns a list of dictionaries with a consistent schema (filling in default values for missing fields). Print the normalized output and discuss in a markdown cell how this illustrates the schema responsibility shift between structured and unstructured data.

### Task 3: CSV vs Parquet — Row-Major Meets Column-Major

This is the central task of the lab. You will generate a realistic dataset and use it to concretely demonstrate the differences between CSV and Parquet.

**Step 1.** Generate a pandas DataFrame with at least 200,000 rows and at least 15 columns. Design the columns to represent a realistic dataset — for example, e-commerce transactions with fields like `transaction_id`, `user_id`, `timestamp`, `product_category`, `amount`, `currency`, `payment_method`, `shipping_country`, `discount_pct`, `is_returned`, and several numerical feature columns. Use `numpy.random` to generate realistic distributions (e.g., exponential for amounts, categorical choices for countries and payment methods).

```python
import pandas as pd
import numpy as np

np.random.seed(42)
n = 200_000

transactions = pd.DataFrame({
    "transaction_id": range(n),
    "user_id": np.random.randint(1, 20000, n),
    # ... add at least 13 more columns
})
```

**Step 2.** Save this DataFrame as both CSV and Parquet:

```python
transactions.to_csv("transactions.csv", index=False)
transactions.to_parquet("transactions.parquet", index=False)
```

Report the file sizes of both files using `os.path.getsize()`. Compute the ratio and write a markdown cell interpreting the result: why is one smaller? What accounts for the difference?

**Step 3.** Read back only 3 columns from each format and time the operation:

```python
import time

# CSV: must read everything, then select
start = time.time()
df_csv = pd.read_csv("transactions.csv", usecols=["amount", "currency", "is_returned"])
csv_time = time.time() - start

# Parquet: reads only the requested columns from disk
start = time.time()
df_pq = pd.read_parquet("transactions.parquet", columns=["amount", "currency", "is_returned"])
pq_time = time.time() - start

print(f"CSV read time:     {csv_time:.4f}s")
print(f"Parquet read time: {pq_time:.4f}s")
print(f"Speedup:           {csv_time / pq_time:.1f}x")
```

Run this cell multiple times and report the typical speedup. In a markdown cell, explain why Parquet is faster for this specific operation, connecting your explanation to the concept of column-major layout.

**Step 4.** Now time a different operation: reading the entire file:

```python
start = time.time()
full_csv = pd.read_csv("transactions.csv")
csv_full_time = time.time() - start

start = time.time()
full_pq = pd.read_parquet("transactions.parquet")
pq_full_time = time.time() - start

print(f"CSV full read:     {csv_full_time:.4f}s")
print(f"Parquet full read: {pq_full_time:.4f}s")
```

In a markdown cell, discuss whether Parquet's advantage is larger or smaller when reading all columns versus a few columns, and explain why.

### Task 4: Row Access vs Column Access — NumPy and pandas

This task makes the row-major vs column-major performance difference concrete and measurable.

**Step 1.** Create a pandas DataFrame with 10,000 rows and 50 columns of random floats:

```python
df = pd.DataFrame(np.random.randn(10000, 50))
```

Time two operations:

a) Iterate over all rows using `df.iloc[i]` in a loop and compute the sum of each row.
b) Iterate over all columns using `df[col]` and compute the sum of each column.

Report the times and the ratio. Explain in a markdown cell why column iteration is faster in pandas.

**Step 2.** Convert the same DataFrame to a NumPy array using `df.values`. Repeat the row iteration from Step 1 on the NumPy array. Compare the timing to the pandas row iteration. Compute and report the speedup factor.

In a markdown cell, explain how the memory layout difference between pandas (column-major) and NumPy (row-major by default) accounts for the performance difference.

**Step 3.** Demonstrate the best approach: use vectorized operations to compute row sums and column sums without any explicit Python loops:

```python
row_sums = df.sum(axis=1)
col_sums = df.sum(axis=0)
```

Time these vectorized operations and compare them to the loop-based approaches from Steps 1 and 2. In a markdown cell, summarize the performance hierarchy: vectorized > matching-layout loop > mismatched-layout loop. Explain why this hierarchy matters for real-world data processing.

### Task 5: Text vs Binary — Measuring the Difference

**Step 1.** Using the transactions DataFrame from Task 3, save it in three additional formats:

a) Parquet with snappy compression (default)
b) Parquet with gzip compression
c) JSON (using `df.to_json("transactions.json", orient="records", lines=True)`)

Report the file size of all four versions (CSV, JSON, Parquet-snappy, Parquet-gzip) in a summary table. Compute each size as a percentage of the CSV size.

**Step 2.** Write a small experiment to demonstrate precision loss in CSV. Create a DataFrame with a single column of 10 float values with high precision (e.g., `np.random.random(10) * np.pi`). Save to CSV and read back. Compare the original and round-tripped values — are they exactly equal? Use `np.allclose()` and also check exact equality with `==`. Do the same with Parquet. In a markdown cell, explain why binary formats preserve precision better than text formats.

**Step 3.** Write a summary markdown cell that gives practical guidance: when would you choose CSV, when JSON, and when Parquet? Address at least three different scenarios (e.g., sharing a small dataset with a non-technical colleague, storing training data for an ML pipeline, receiving data from a web API).

### Task 6: Structured vs Unstructured — Schema Decisions

**Step 1.** Create a raw text string representing 15 lines of semi-structured log data. Most lines should follow a pattern like `"2025-03-15 14:30:01 | INFO | user_service | User 4521 logged in"`, but include at least 3 lines that deviate: one with a different delimiter, one with a missing field, and one that is a multi-line stack trace or error message.

Write a parser function that attempts to extract structured records (timestamp, level, service, message) from each line. The function should handle the deviating lines gracefully — either by returning partial records or by flagging them as unparseable. Print the results and count how many lines were successfully parsed versus flagged.

**Step 2.** Take the successfully parsed records and create a pandas DataFrame. Then take the original raw text and store it as a list of strings. In a markdown cell, discuss:

- Which representation (DataFrame vs raw strings) makes it easier to answer "How many ERROR events occurred?"
- Which representation handles unexpected formats more gracefully?
- Where does the schema responsibility live in each case?

**Step 3.** Write a final reflection paragraph (in a markdown cell) connecting this task back to the data warehouse vs data lake concept. Explain how a real organization might use a data lake to store the raw logs and a data warehouse to store the parsed, structured version — and why having both is valuable.

## Common Pitfalls and Debugging Notes

- **Parquet requires `pyarrow`**: If you get an import error when calling `to_parquet()` or `read_parquet()`, install it with `pip install pyarrow`.
- **Timing variability**: The first run of a read operation may be slower due to disk caching. Run timing cells multiple times and report typical values, not outliers.
- **File cleanup**: The lab creates several files on disk. You may want to add a cleanup cell at the end that removes them with `os.remove()`.
- **Precision comparison**: When comparing floats, remember that `==` checks exact bit-level equality. Use `np.allclose()` for approximate comparison and `==` to highlight precision loss.
- **Large DataFrames in notebooks**: If your notebook becomes slow, avoid printing entire large DataFrames. Use `.head()`, `.shape`, or `.describe()` instead.

## Submission

### What to submit

Submit the following file:

- A notebook file `m2-01-data-sources-representation-lab.ipynb` containing all six tasks with code, outputs, and markdown explanations

### Definition of done (checklist)

Before you submit, make sure:

- [ ] The notebook runs **top to bottom** without errors after a kernel restart.
- [ ] All six tasks are completed with both code and markdown explanations.
- [ ] Task 1 creates realistic data for three different source types with quality analysis.
- [ ] Task 2 demonstrates JSON serialization, schema flexibility, and normalization.
- [ ] Task 3 compares CSV and Parquet with measured file sizes and read times.
- [ ] Task 4 demonstrates row vs column access performance across pandas and NumPy.
- [ ] Task 5 compares file sizes across four formats and demonstrates precision differences.
- [ ] Task 6 parses semi-structured data and discusses schema responsibility.
- [ ] Markdown cells provide thoughtful explanations (not just code outputs).
- [ ] Temporary files are cleaned up or noted for cleanup.

### How to submit (Git workflow)

When you are done, make sure all changes are saved, then run:

```bash
git add .
git commit -m "Solved m2-01 lab"
git push -u origin HEAD
```

- Make a pull request.
- Paste the link to your pull request in the Student Portal.

## Evaluation Criteria

Your work will be evaluated on three dimensions:

**Correctness.** Your code produces accurate results — file sizes are reported correctly, timing comparisons are meaningful, parsers handle edge cases, and format conversions are verified.

**Reasoning.** Your markdown explanations demonstrate genuine understanding of why formats differ, how memory layouts affect performance, and where schema responsibility lies. We are looking for explanations that connect observations to underlying concepts, not just descriptions of what happened.

**Professionalism.** Your notebook is clean, well-organized, and runs reproducibly from top to bottom. Code is readable, variables have meaningful names, and the narrative flows logically from one task to the next.
