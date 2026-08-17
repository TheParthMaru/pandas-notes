# Pandas notes

Personal notes from learning pandas. The working notebook is [`pandas-notes.ipynb`](pandas-notes.ipynb). Sample coffee sales live in `sample_data/`; Olympics datasets live in `data/`.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Then open `pandas-notes.ipynb` and select the `.venv` kernel.

## Cheatsheet

### Import and DataFrame

```python
import pandas as pd

df = pd.DataFrame(
    [[1, 2, 3], [4, 5, 6], [7, 8, 9]],
    columns=["A", "B", "C"],
    index=["X", "Y", "Z"],
)
```

A DataFrame is pandas' table: rows, columns, and extra methods for working with that data.

### Inspect

```python
df.head()          # first 5 rows (pass n for a different count)
df.tail(1)         # last n rows
df.columns         # column labels
df.columns.to_list()
df.index           # row labels
df.index.to_list()
df.info()          # dtypes, non-null counts
df.describe()      # summary stats
df.shape           # (rows, columns)
df.size            # total cells (rows * columns)
df.sample(3)       # random rows
```

Prefer `to_list()` in pandas. `tolist()` is the NumPy name; they do the same thing.

`print(df)` dumps text. In a notebook, `display(df)` is easier to read.

### Load and export

```python
coffee = pd.read_csv("sample_data/coffee.csv")
results = pd.read_parquet("data/results.parquet")   # needs pyarrow
olympics = pd.read_excel("data/olympics-data.xlsx")  # needs openpyxl
results_sheet = pd.read_excel("data/olympics-data.xlsx", sheet_name="results")

coffee.to_excel("sample_data/coffee.xlsx")           # path must end in .xlsx
```

| Format   | Notes                                      |
| -------- | ------------------------------------------ |
| CSV      | Easy to inspect; safest default            |
| Parquet  | Smallest; needs `pyarrow`                  |
| Feather  | Smaller than CSV                           |
| Excel    | Slowest to read; use `sheet_name` for tabs |

### Select: `loc` vs `iloc`

- `loc` — labels (index names, column names). Slice **includes** the end.
- `iloc` — integer positions. Slice **excludes** the end (like Python lists).

```python
coffee.loc[4]                              # row with index label 4
coffee.loc[[0, 2, 3]]                      # specific rows
coffee.loc[0:5]                            # rows 0 through 5 inclusive
coffee.loc[0:5, ["Day", "Units Sold"]]     # rows + named columns

coffee.iloc[0]                             # first row
coffee.iloc[0:5, 2]                        # first 5 rows, 3rd column
coffee.iloc[0:5, [0, 2]]                   # first 5 rows, columns 0 and 2
```

`coffee.loc[4]` is a Series. `coffee.loc[[4]]` keeps a one-row DataFrame.

### Single cells: `at` / `iat`

Faster than `loc`/`iloc` for one value. No slicing.

```python
coffee.at[0, "Units Sold"]   # label
coffee.iat[0, 0]             # position
```

### Update and grab columns

```python
coffee.loc[1, "Units Sold"] = 69   # one cell; slice to update many
coffee.Day                         # column as Series (no spaces)
coffee["Units Sold"]               # works with spaces
```

### Sort

```python
coffee.sort_values("Units Sold")                          # ascending
coffee.sort_values("Units Sold", ascending=False)         # descending
coffee.sort_index(ascending=False)

# Multi-level: Units Sold first, then Coffee Type if values tie
coffee.sort_values(["Units Sold", "Coffee Type"], ascending=False)
coffee.sort_values(["Units Sold", "Coffee Type"], ascending=[False, True])
```

`sort_values` / `sort_index` return a new DataFrame unless you pass `inplace=True`.

### Filter

```python
bios = pd.read_csv("data/bios.csv")

# Boolean mask
bios.loc[bios["height_cm"] > 200, ["name", "height_cm"]]
bios[bios["height_cm"] > 200][["name", "height_cm"]]

# Multiple conditions — wrap each in parentheses, use & / |
bios.loc[(bios["height_cm"] > 200) & (bios["born_country"] == "GBR")]

# String contains (regex). case=False to ignore case; | for OR
bios[bios["name"].str.contains("Keith")][["name", "born_country"]]
bios[bios["name"].str.contains("keith|patrick", case=False)]

# Membership
bios[bios["born_country"].isin(["USA", "FRA", "GBR"])][["name", "born_country"]]

# Query string (less flexible than the mask style above)
bios.query("born_country == 'USA' and born_city == 'Seattle'")
```
