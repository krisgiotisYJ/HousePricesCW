# Data

HM Land Registry Price Paid Data, yearly files for 2024 and 2025 (England and Wales, over 100 MB each).

They're too big to store in the repository, so the notebook downloads them into `data/raw/` the first time
it runs. Nothing needs doing by hand.

If you'd rather fetch them yourself (for example on a machine with no internet access for the notebook),
save the two files from the official yearly-file page into `data/raw/` under these names:

- `data/raw/pp-2024.csv` from https://price-paid-data.publicdata.landregistry.gov.uk/pp-2024.csv
- `data/raw/pp-2025.csv` from https://price-paid-data.publicdata.landregistry.gov.uk/pp-2025.csv

Yearly-file page: https://www.gov.uk/government/statistical-data-sets/price-paid-data-yearly-file

Info page: https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads

`data/processed/three_boroughs_2024_2025.csv` (about 2 MB) is the cleaned subset the models use: Bromley,
Croydon and Greenwich, category A, types D/S/T/F. The notebook writes it, and it's shared so the results
can be checked without the raw files.

## What's in the raw files

- No header row. 16 columns: transaction id, price, date, postcode, property type (D/S/T/F/O), old/new (Y = new build),
  duration (F = freehold, L = leasehold), PAON, SAON, street, locality, town/city, district, county,
  PPD category (A = standard sale, B = additional e.g. repossessions / buy-to-let / non-private buyers), record status.
- Downloaded 15 Nov 2026. Land Registry updates these files monthly, so a later download can have a few more 2025 rows.

Contains HM Land Registry data © Crown copyright and database right 2021. This data is licensed under the Open Government Licence v3.0.
