# Data

HM Land Registry Price Paid Data, yearly files for 2024 and 2025 (England and Wales, over 100 MB each).

They're too big for the repository, so the notebook downloads them into `data/raw/` the first time it runs.
There's nothing to do by hand.

If you'd rather fetch them yourself (say the notebook's machine has no internet), save the two files from the
official yearly-file page into `data/raw/` under these names:

- `data/raw/pp-2024.csv` from https://price-paid-data.publicdata.landregistry.gov.uk/pp-2024.csv
- `data/raw/pp-2025.csv` from https://price-paid-data.publicdata.landregistry.gov.uk/pp-2025.csv

Yearly-file page: https://www.gov.uk/government/statistical-data-sets/price-paid-data-yearly-file

Info page: https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads

`data/processed/three_boroughs_2024_2025.csv` (about 2 MB) is the cleaned subset the models use: Bromley,
Croydon and Greenwich, category A, types D/S/T/F. The notebook writes it, and it's shared so the results
can be checked without the raw files.

## What's in the raw files

- No header row. Columns in order: transaction id, price, date, postcode, property type (D/S/T/F/O), old/new
  (Y = new build), duration (F = freehold, L = leasehold), PAON, SAON, street, locality, town/city, district,
  county, PPD category (A = standard sale, B = additional e.g. repossessions / buy-to-let / non-private buyers),
  then record status.
- The GOV.UK guide says record status is only in the monthly files. The yearly files have it anyway.
  I checked the first line of both official files on 18 November 2026: 16 fields, the last one "A".
  The notebook takes 15 or 16 columns in case that changes, and doesn't use record status either way.
- Downloaded 15 Nov 2026. Land Registry updates these files monthly, so a later download can have a few more 2025 rows.

Contains HM Land Registry data © Crown copyright and database right 2021. This data is licensed under the Open Government Licence v3.0.

## Floor areas (used by `floor_area_experiment.ipynb` only)

*House Price Per Square Metre in England and Wales, 1995-2024*, by Chi, Dennett, Oléron-Evans and Morphet
at University College London. It's Land Registry sales already matched to the floor area on each home's
Energy Performance Certificate, up to 31 October 2024.

- The notebook downloads `hpm_la_2024.zip` (1.24 GB) into `data/ucl/` on its first run. The London Datastore
  gives Python's default client name a 403, so the notebook sends its own. To fetch it by hand, use
  https://data.london.gov.uk/dataset/house-price-per-square-metre-in-england-and-wales
- It then keeps only the three boroughs from 2021 onwards and saves `data/ucl/three_boroughs_2021_2024.csv`.
  Later runs start from that small file.
- Columns used: `transactionid`, `year`, `dateoftransfer`, `price`, `propertytype`, `duration`, `postcode`,
  the local authority code, `tfarea` (total floor area, m²), `numberrooms` (habitable rooms, not bedrooms)
  and `classt` (11 = the sale matched one certificate, 12 = it matched more than one).
- `priceper` (price per m²) is in the file but never goes into a model, since it's worked out from the sale
  price. Section 10 works out price per m² itself, only to describe the sales.
- Quirks I found: the field guide shows dates like 11/3/2014 but this file uses 2014-03-11, the local
  authority column is `lad23cd`, two sales list 51 habitable rooms, and some flats have a floor area of 0 m².
- Nothing in `data/ucl/` is stored in the repository. The licence of this copy isn't clearly stated: the
  London Datastore page shows none and says its source data was downloaded for non-commercial purposes, and
  the UK Data Service record shows a CC BY 4.0 badge on its bundles but no licence in its DataCite metadata.
  So only summary tables made from it are shared.
- The size and SHA-256 fingerprint of the zip and the extract used are saved in `results/floor_area_inputs.csv`.

Dataset record: https://doi.org/10.5255/UKDA-SN-857911

How the matching was done (the method paper): Chi, B., Dennett, A., Oléron-Evans, T. and Morphet, R. (2021)
A new attribute-linked residential property price dataset for England and Wales, 2011–2019. *UCL Open:
Environment*, 2021(2), Article 07. https://doi.org/10.14324/111.444/ucloe.000019
