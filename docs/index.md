# dynamic_characterization

This is a collection of dynamic characterization functions for life cycle inventories with temporal information. 

Here's an overview of what is currently included:

| impact category | metric | covered emissions | source
|-------|----------|----------|--|
| climate change | radiative forcing | CO2, CH4 |[bw_temporalis](https://github.com/brightway-lca/bw_temporalis/tree/main)|
| climate change | radiative forcing | 247 GHGs from [IPCC AR6 Ch.7](https://www.ipcc.ch/report/ar6/wg1/chapter/chapter-7/) |[bw_timex](https://github.com/brightway-lca/bw_timex/tree/main)|

## What do dynamic characterization functions do?

The functions are meant to work with a common input format of the dynamic inventory, collected in a pandas DataFrame that looks like this:

| date | amount | flow | activity |
|-------|-------|------|----------|
| 101   | 33    | 1    | 2        |
| 312   | 21    | 4    | 2        |

Each function takes one row of this dynamic inventory dataframe (i.e. one emission at one point in time) and transform it according to some metric. The output generated by applying a very simple function to both rows of the input dataframe could look like:

| date | amount | flow | activity |
|------|--------|------|----------|
| 101  | 33     | 1    | 2        |
| 102  | 31     | 1    | 2        |
| 103  | 31     | 1    | 2        |
| 312  | 21     | 4    | 2        |
| 313  | 20     | 4    | 2        |
| 314  | 19     | 4    | 2        |

## What do dynamic characterization functions look like?

Here's an example of what such a function could look like:

```python
def characterize_something(series, period: int = 100, cumulative=False) -> pd.DataFrame:
    date_beginning: np.datetime64 = series["date"].to_numpy()
    date_characterized: np.ndarray = date_beginning + np.arange(
        start=0, stop=period, dtype="timedelta64[Y]"
    ).astype("timedelta64[s]")

    decay_multipliers: list = np.array(
        [
            1.234 * (1 - np.exp(-year / 56.789))
            for year in range(period)
        ]
    )

    forcing = pd.Series(data=series.amount * decay_multipliers, dtype="float64")
    if not cumulative:
        forcing = forcing.diff(periods=1).fillna(0)

    return pd.DataFrame(
        {
            "date": pd.Series(data=date_characterized, dtype="datetime64[s]"),
            "amount": forcing,
            "flow": series.flow,
            "activity": series.activity,
        }
    )
```

```{toctree}
---
hidden:
maxdepth: 1
---
content/usage
content/api/index
Code of Conduct <content/codeofconduct>
Contributing <content/contributing>
content/license
Changelog <content/changelog>
```
