# FDA NDC Directory

Turns the openFDA NDC bulk file into separate tables, one grain each, so a package code can be looked up without fanning the product row out.

Source: [openFDA NDC Directory](https://open.fda.gov/apis/drug/ndc/download/). One zip, about 138,000 products and about 256,000 packages. The API page loop stops near 26,000, so the notebook uses the download file.

## Grain

| File | One row per | Joins on |
| --- | --- | --- |
| `product.csv` | product NDC (labeler + product code) | |
| `package.csv` | 11-digit package NDC | `product_ndc` |
| `ingredient.csv` | product + ingredient + strength | `product_ndc` |
| `drug_class.csv` | product + class | `product_ndc` |
| `route.csv` | product + route | `product_ndc` |
| `ndc_openfda_lookup.csv` | product + openFDA field + value | `product_ndc` |

`product_ndc` is the hyphenated code (`71335-0890`). `ndc11` is built, not copied: labeler padded to 5, product code padded to 4, package code padded to 2.

A product can have many packages, ingredients, classes, and routes. Those stay in their own files. Joining them all back onto package before a filter fans the row.

## What the notebook does

1. Downloads `drug-ndc-0001-of-0001.json.zip` if it is not already on disk.
2. Writes `ndc_source_before_processing.csv`. Nested lists stay as text. Grain is still one product.
3. Builds the six frames and writes them beside the notebook.
4. Looks up one package NDC and returns product, package, ingredient, class, route, and openFDA.
5. Builds `ndc_site_extract.csv`, one row per package, with class, route, and ingredient collapsed into cells. Same shape as the FDA site search page.
6. Joins active human prescription products to the WHO ATC-DDD file on a cleaned generic name. Writes the match, the fallout, and a class rollup.

## ATC join

Second file, not part of openFDA. [WHO ATC-DDD](https://atcddd.fhi.no/atc_ddd_index/), scraped 2026-04-25. Level is the length of the code: 1 anatomical, 3 therapeutic, 5 chemical, 7 substance.

There is no NDC on the WHO file. The join key is the generic name after salt and dose form are stripped, with a few US-to-INN spellings (`acetaminophen` to `paracetamol`). Combos, gene therapies, and names WHO does not list fall out. That fallout is expected.

| File | What it is |
| --- | --- |
| `ndc_matched.csv` | Products that found an ATC code |
| `ndc_fallout.csv` | Products that did not, with a reason |
| `ndc_class_roll.csv` | Product counts by anatomical and therapeutic class |

This is not a Medi-Span or FDB crosswalk. Same ingredient can sit in more than one ATC class. The rollup is a study view, not spend.

## Run

Open `Normalizing FDA Drug Data.ipynb` and run the cells in order. Outputs land in the working folder.

Lookup takes a 10- or 11-digit code, with or without hyphens.

```python
hit = lookup("71335-0890-7")
```

## Not in this repo

The zip and the CSVs are local. They are large, and openFDA is the source of record. Clone the notebook and run the fetch cell.

## Disclaimer

For study and personal learning only. Not for clinical use, dispensing, billing, pricing, purchasing, or any other decision about a drug or a patient.

The tables are a reshape of the public openFDA NDC download. They are not a complete, current, or certified drug file. openFDA and the FDA label are the source. Packaging, strength, class, route, and marketing status can be missing or stale. Do not rely on this project in place of Medi-Span, FDB, or the label.

