# House price prediction: Bromley, Croydon, Greenwich

Can a machine-learning model predict what a home sells for better than looking up the typical local price?

This project tests that on HM Land Registry sales in three London boroughs. It compares two median baselines,
Linear Regression and Random Forest (scikit-learn), with and without postcode detail. The models learn from
2024 sales, are tuned on January-June 2025, and are tested once on July-December 2025.

Everything is in one notebook, `house_prices.ipynb`.

## Run it

You need Python 3.12 or newer and an internet connection for the first run. The steps are the same on
Linux, macOS and Windows; only the path to the environment differs.

Download or clone this folder, open a terminal inside it, then:

**Linux / macOS**
```
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/jupyter nbconvert --to notebook --execute --inplace house_prices.ipynb
```

**Windows**
```
py -m venv .venv
.venv\Scripts\pip install -r requirements.txt
.venv\Scripts\jupyter nbconvert --to notebook --execute --inplace house_prices.ipynb
```

The last line runs the whole notebook and saves it with its results. If you'd rather watch it run, open it
with `jupyter lab house_prices.ipynb` (same `.venv` path in front) and choose Run All.

The first run downloads the two national Land Registry files (over 100 MB each) into `data/raw/`, so give it
a few minutes. After that a run takes a minute or two, depending on the computer. All tables and figures
are written to `results/`.

Tested with Python 3.12 and 3.13 on Linux, using the library versions in `requirements.txt`.

## What it found

Average error (MAE) on the 5,605 test sales, July-December 2025, using data downloaded on 15 November 2026.
The median test sale was £470,000.

| Run | Method | Average error |
|---|---|---|
| R1 | Overall median price | £189,807 |
| R2 | Median for the same borough and property type | £138,523 |
| R3 | Linear Regression, basic features | £138,613 |
| R4 | Random Forest, basic features | £138,105 |
| R5 | Linear Regression + postcode district | £124,468 |
| R6 | Random Forest + postcode district | £126,351 |
| R7 | Tuned Random Forest + postcode district | £121,839 |

Location is what matters: adding the postcode district cuts the error by £12,000-£14,000, while the choice
of model moves it by a few thousand at most. Section 11 of the notebook has six follow-up experiments
(a more local baseline, postcode sector, raw-price target, gradient boosting); their results are in
`results/extra_experiments.csv`.

The Land Registry adds late registrations to its files every month, so a fresh download can contain a few
more 2025 sales and give slightly different numbers.

## Files

- `house_prices.ipynb` - the whole project: load, clean, features, split, baselines, models, tuning, test,
  error analysis, follow-up experiments
- `requirements.txt` - the library versions it was run with
- `results/` - every table (.csv), the figures (fig_*.png) and the trained tuned Random Forest
  (`model_r7_tuned_rf.joblib`, load it with `joblib.load`)
- `data/README.md` - where the data comes from and what the columns mean
- `data/processed/three_boroughs_2024_2025.csv` - the cleaned subset the experiments use: 22,170 standard
  sales in the three boroughs. The notebook writes this file. It is here so the results can be checked
  without the national files, but a run always starts from `data/raw/`.
- `data/raw/` - created on the first run; not stored in the repository

## Data licence

Contains HM Land Registry data © Crown copyright and database right 2021. This data is licensed under the Open Government Licence v3.0.

The data includes address details (postcodes) that Royal Mail and Ordnance Survey allow for personal or
non-commercial use, or for showing residential price information. Check the conditions on the
[Price Paid Data page](https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads)
before using it any other way.
