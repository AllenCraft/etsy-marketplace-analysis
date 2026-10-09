# Data schema

The raw data is **not** in this repository. It was scraped from Etsy best-seller pages with Octoparse and contains seller and listing identifiers. This file documents the structure only, with no values.

## Raw input files (kept locally in `data/`, git-ignored)

| Local name expected by the notebook | Scrape | Columns used |
|---|---|---|
| `data/best_seller_jun2024.xlsx` | June 2024 best-seller list, sheet 1 | `Plain URL`, `Page_URL`, `status`, `Demand State`, `Price USD`, `Title`, `Reviews_ThisItem`, `Shop_Reviews`, `Favorites` |
| `data/best_seller_jul2024.xlsx` | July 2024 best-seller list, sheet 1 | `Original_URL`, `Demand`, `Price`, `Title`, `Reviews Item`, `Reviews Shop`, `Listen on`, `Favorites` |

URLs are used only to extract the numeric listing ID for de-duplication and to recover a title from the URL slug. Neither is kept after cleaning.

## Cleaned analysis table (one row per unique listing with a demand badge)

| Column | Type | Meaning |
|---|---|---|
| `source` | text | Which scrape the row comes from (`2024-06` or `2024-07`). Where a listing is in both, the July record is kept. |
| `price` | float, USD | Price parsed from text. For listings with variations, the starting price. Missing if not scraped. |
| `has_variants` | bool | The raw price ended in `+` (a starting price). |
| `item_reviews`, `shop_reviews`, `favorites` | integer (nullable) | Counts parsed from text or numbers. Placeholder zeros from failed scrapes are set to missing. |
| `demand_tier` | text | Etsy's demand badge, grouped: `In 20+ carts`, `Bought recently`, `In 1-19 carts`, `Views only`, `Low stock only`. |
| `high_demand` | 0/1 | 1 if the badge shows 20+ carts or a recent purchase count. |
| `niche` | text | Keyword-based niche tag from the title (or URL slug). See the rule list in the notebook. |
| `price_band` | text | `up to $2`, `$2-4`, `$4-6`, `$6-10`, `$10-20`, `over $20`. |
| `bundle_word` | bool | Title contains bundle, set, mega, pack or a count of prints. |

## `anonymised_sample.csv`

A random sample of 300 rows from the cleaned table (fixed seed), with everything that could identify a shop or listing removed or coarsened:

| Column | Notes |
|---|---|
| `source`, `niche`, `demand_tier`, `high_demand`, `has_variants`, `bundle_word`, `price_band` | As above. |
| `price_usd_rounded` | Price rounded to the nearest dollar. |
| `item_reviews_band`, `shop_reviews_band`, `favorites_band` | Counts grouped into `0`, `1-10`, `11-100`, `101-1k`, `1k-10k`, `10k+`. |

Not included: titles, URLs, shop names, listing IDs, exact review or favourite counts, dates.
