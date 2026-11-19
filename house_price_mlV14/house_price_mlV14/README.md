# House price prediction: Bromley, Croydon, Greenwich

Can a machine-learning model predict what a home sells for better than just looking up the typical local price?

I tested that on HM Land Registry sales in three London boroughs: two median baselines against Linear
Regression and Random Forest (scikit-learn), with and without postcode detail. The models learn from 2024
sales, get tuned on January-June 2025, and are tested once on July-December 2025.

Most of it is in `house_prices.ipynb`. The second notebook, `floor_area_experiment.ipynb`, asks one follow-up
question: does knowing a home's floor area make the predictions better? (Short answer: yes, by a lot. Numbers below.)

**You don't need to download anything just to look.** GitHub shows both notebooks with their results, and
every table is in `results/`. The steps below are only for running it yourself, and even then the notebooks
fetch their own data. Links are in [Getting the data](#getting-the-data) in case a download fails.

## Run it

Python 3.12 or newer, and an internet connection for the first run. Same steps on Linux, macOS and Windows,
only the path to the environment changes.

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

The last line runs the whole notebook and saves it with its results. If you'd rather watch, open it with
`jupyter lab house_prices.ipynb` (same `.venv` path in front) and Run All.

The first run downloads the two national Land Registry files (100 MB+ each) into `data/raw/`, so give it a few
minutes. After that it's a minute or two. Everything gets written to `results/`.

Tested with Python 3.12 and 3.13 on Linux, with the library versions in `requirements.txt`.

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

Location does most of the work. Adding the postcode district cuts the error by £12,000-£14,000, while the choice of
model moves it by a few thousand at most, and on the basic features neither model really beats the plain
borough-and-type median. Section 11 of the notebook has six follow-ups I added afterwards (a more local
baseline, postcode sector, raw-price target, gradient boosting), with results in `results/extra_experiments.csv`.

The Land Registry adds late registrations every month, so a fresh download can have a few more 2025 sales
and give slightly different numbers.

## The floor-area experiment

The models above know a home's type and postcode but not its size, so a small detached house and a big one get
the same guess. That seemed like the most obvious gap, so `floor_area_experiment.ipynb` tests it with one comparison:

- **Model A**: property type, freehold or leasehold, postcode district, month of sale.
- **Model B**: the same, plus total floor area in m².

Both Random Forests, tuned the same way, tested on the same sales. The floor areas come from University College
London's linked dataset, which already matches Land Registry sales to Energy Performance Certificates. That file
stops at 31 October 2024, so this experiment learns from 2021-2022, tunes on 2023 and tests on January-October
2024. Don't compare its numbers with the R1-R7 table above. Model A against Model B is the comparison that counts.

Run it the same way as the main notebook, with `floor_area_experiment.ipynb` in place of `house_prices.ipynb`.
The first run downloads the UCL file (1.24 GB) into `data/ucl/` and cuts it down to a small extract, which takes a
while. Run `house_prices.ipynb` first if you want the coverage check in section 9.

For a while that download didn't work. The London Datastore turns away Python's default
client name with a 403, and I didn't notice because I'd always fetched the zip by hand first. It only came out
when I ran the notebook in an empty folder. Fixed now, and the empty-folder run went through on 18 November 2026.
If it fails anyway, fetch `hpm_la_2024.zip` by hand from the
[London Datastore page](https://data.london.gov.uk/dataset/house-price-per-square-metre-in-england-and-wales)
and put it in `data/ucl/`.

The notebook saves the size and SHA-256 fingerprint of the UCL zip and the extract in
`results/floor_area_inputs.csv`. If your zip's fingerprint is different, UCL have changed the file and your numbers
may differ too.

### What it found

First run 17 November 2026, on 41,859 matched sales: 27,686 to learn from, 8,563 to tune on, 5,610 to test on.
Re-run from an empty folder on 18 November, with the notebook doing its own download: every table came out
byte for byte the same.

| On the 5,610 test sales, Jan-Oct 2024 | Without floor area (A) | With floor area (B) |
|---|---|---|
| Average error (MAE) | £118,044 | £80,227 |
| R² | 0.47 | 0.78 |
| Average percentage error | 24.1% | 16.8% |

- Floor area cuts the average error by about a third. Redrawing the test sales 1,000 times puts the gain between
  £34,900 and £40,715.
- It helps every property type, most of all where the main notebook struggled: detached houses go from £248,437
  to £160,683, and the most expensive tenth of homes from £371,300 to £227,602.
- It holds when both models use the same settings (a gain of £34,879) and on the cleanest matches alone (£37,639).
- Only 62% of the January-October 2024 sales in the main notebook have a floor area in the UCL file, and flats are
  under-represented among them.
- Even with floor area, detached houses are still £160,683 out on average, so this isn't a valuation tool.

Two things about the data surprised me. Four of the five cleaning rules remove nothing at all, because the file is
that tidy. And the one rule that does remove something, floor area between 10 and 1,000 m², takes out three
£5m+ Bromley houses that are probably real, along with the obvious typos. Section 3 of the notebook shows them.

### What homes of each size sold for

Section 10 of the same notebook shows the typical (median) price by floor area, by number of habitable rooms and by
borough, from the matched sales of 2021 to October 2024.

| Detached houses, habitable rooms | 3 | 5 | 7 | 9 or more |
|---|---|---|---|---|
| Typical price | £525,000 | £670,000 | £875,000 | £1,305,000 |
| Typical floor area | 75 m² | 109 m² | 152 m² | 243 m² |

- Habitable rooms are bedrooms plus living and dining rooms, so a three-bed house is usually four or five.
- Bigger homes cost more in total but less per m². A detached house of 50-74 m² went for about £7,170 per m², one
  of 300 m² or more for about £5,128.
- Croydon is the cheapest of the three per m² for every property type. Greenwich flats are the dearest group of
  all, at about £6,531 per m².
- These are past prices, one thing at a time. Bigger homes also tend to be in pricier streets, which is why you
  need the model to separate size from location.

Results go to `results/floor_area_*.csv` and `results/price_*.csv`.

## Files

- `house_prices.ipynb`: the main project (load, clean, features, split, baselines, models, tuning, test,
  error analysis, follow-up experiments)
- `floor_area_experiment.ipynb`: the floor-area experiment above
- `data/ucl/`: the UCL download and the extract cut from it; created on the first run, not in the repository
- `requirements.txt`: the library versions it was run with
- `results/`: every table (.csv), the figures (fig_*.png) and the trained tuned Random Forest
  (`model_r7_tuned_rf.joblib`, load it with `joblib.load`)
- `data/README.md`: where the data comes from and what the columns mean
- `data/processed/three_boroughs_2024_2025.csv`: the cleaned subset the experiments use: 22,170 standard
  sales in the three boroughs. The notebook writes it. It's here so the results can be checked without the
  national files, but a run always starts from `data/raw/`.
- `data/raw/`: created on the first run; not in the repository

## Getting the data

Both notebooks download what they need on their first run. If a download fails, these are the sources:

| Data | Used by | Where to get it | Put it in |
|---|---|---|---|
| HM Land Registry Price Paid Data, 2024 and 2025 yearly files | `house_prices.ipynb` | [pp-2024.csv](https://price-paid-data.publicdata.landregistry.gov.uk/pp-2024.csv) and [pp-2025.csv](https://price-paid-data.publicdata.landregistry.gov.uk/pp-2025.csv), from the [GOV.UK yearly-file page](https://www.gov.uk/government/statistical-data-sets/price-paid-data-yearly-file) | `data/raw/` |
| UCL House price per square metre dataset, 2024 file | `floor_area_experiment.ipynb` | `hpm_la_2024.zip` (1.24 GB) from the [London Datastore](https://data.london.gov.uk/dataset/house-price-per-square-metre-in-england-and-wales) | `data/ucl/` |

The full UCL collection, including the original files it was built from, is also on the
[UK Data Service](https://doi.org/10.5255/UKDA-SN-857911). The notebook uses the London Datastore file.

## Data licence

Contains HM Land Registry data © Crown copyright and database right 2021. This data is licensed under the Open Government Licence v3.0.

The data includes address details (postcodes) that Royal Mail and Ordnance Survey allow for personal or
non-commercial use, or for showing residential price information. Check the conditions on the
[Price Paid Data page](https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads)
before using it any other way.

The floor-area experiment uses the UCL dataset listed under References. Its floor areas and room counts come
from Domestic Energy Performance Certificates published by the Ministry of Housing, Communities and Local
Government. The licence of the copy used here isn't clearly stated:

- The London Datastore copy, which the notebook downloads, shows no licence, and says its source data was
  downloaded for non-commercial purposes.
- The UK Data Service record shows a CC BY 4.0 badge on its two download bundles, but only as an image.
  Its DataCite metadata lists no licence.
- The 2021 method paper is CC BY 4.0, but that covers the paper, not the data.

The file also contains the Land Registry postcodes. So this repository doesn't store the UCL file or the
extract cut from it, only summary tables made from them (`results/floor_area_*.csv` and
`results/price_*.csv`), credited to the authors below.

## References

- Chi, B., Dennett, A., Oléron-Evans, T. and Morphet, R. (2021) A new attribute-linked residential property
  price dataset for England and Wales, 2011–2019. *UCL Open: Environment*, 2021(2), Article 07.
  https://doi.org/10.14324/111.444/ucloe.000019
- Chi, B., Dennett, A., Oléron-Evans, T. and Morphet, R. (2025) *House Price Per Square Metre in England and
  Wales, 1995-2024* [data collection]. Colchester, Essex: UK Data Service. https://doi.org/10.5255/UKDA-SN-857911
- HM Land Registry, *Price Paid Data*, yearly files for 2024 and 2025, downloaded 15 November 2026.
  https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads

The papers behind the method choices (Breiman, Rosen, Pedregosa et al., Strobl et al., Mathotaarachchi et al.,
Mora-García et al. and Ho et al.) are listed at the end of `house_prices.ipynb`.
