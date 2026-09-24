# Smakly food data: changes to the OpenNutrition database

This repository is the **file of alterations** for the copy of the
[OpenNutrition](https://www.opennutrition.app) food database used by
[Smakly](https://smakly.app), published under ODbL 1.0 §4.6 b.

Smakly is a meal planner. It shows nutrition figures from OpenNutrition, and to
make them usable it adds things of its own: Polish names, a shop department per
food, an icon key, a handful of data-quality notes and two dozen corrected
units. Those additions make our copy a *Derivative Database*, and because the
app uses it publicly, the licence obliges us to offer either the whole database
or a file of every alteration. This is that file — or rather, ten of them, one
per kind of alteration, each keyed by the food's own upstream id so anyone with
the original dataset can apply them.

Nothing here is a copy of the source data. The nutrition figures, the English
names, the ingredients, the barcodes and the food types are OpenNutrition's and
stay there; download them from OpenNutrition, then apply these files on top.

## Attribution and licence

The source database is **OpenNutrition** (<https://www.opennutrition.app>),
released under the [Open Database License (ODbL) 1.0](https://opendatacommons.org/licenses/odbl/1-0/),
with the individual contents under a modified
[Database Contents License (DbCL) 1.0](https://opendatacommons.org/licenses/dbcl/1-0/).
Portions of the OpenNutrition dataset incorporate data from **Open Food Facts**
(<https://world.openfoodfacts.org>), © Open Food Facts contributors, also under
ODbL 1.0.

The files in this repository are likewise licensed under **ODbL 1.0**
(see [`LICENSE`](LICENSE)). OpenNutrition's contents licence is reproduced as
[`LICENSE-DbCL-OpenNutrition.txt`](LICENSE-DbCL-OpenNutrition.txt) exactly as it
ships with the dataset, because its attribution conditions travel with the data
you will merge these files into: if you display any of it, attribute
"OpenNutrition" with a link on every screen that shows it, not only in an
about page.

## Which version this is

These files describe our database as imported from
**`opennutrition-dataset-2025.1`**, loaded on **2026-01-21**, and are keyed to
the 326,759 food ids in that release. Every id in every file was checked to
exist in that dataset, and no id in our database is missing from it.

A newer upstream release renumbers nothing — `fd_…` ids are stable — but it will
contain foods these files say nothing about.

## Format

CSV, one file per kind of alteration: UTF-8, LF line endings, `,` separator,
`"` quoting (RFC 4180), a header row, and rows sorted by the first column. CSV
rather than JSONL because every alteration here is one flat row per food, which
is what `\copy`, `pandas.read_csv` and a spreadsheet all want; the single nested
value (Polish synonyms) is a JSON array inside its cell.

`open_nutrition_id` is OpenNutrition's own `id` column (`fd_…`). It is the only
key used anywhere here: our database's UUIDs are generated per environment and
would mean nothing to you.

| File | Rows | What it holds |
|---|---:|---|
| [`food_names_pl.csv`](food_names_pl.csv) | 9,135 | `open_nutrition_id, name_pl, alternate_names_pl`. The Polish name of a food and the search synonyms under which it can be found; `alternate_names_pl` is a JSON array of up to four lowercase entries (one food has none). Covers the `everyday` and `prepared` foods, the ones the app searches. |
| [`dictionary_names_pl.csv`](dictionary_names_pl.csv) | 898 | `dictionary, key, name_en, name_pl`. Polish names for the four vocabularies shown next to a food: `measure_units` (761, keyed by unit name), `nutrients` (97, keyed by technical name), `food_labels` (16, keyed by their English name) and `shopping_categories` (24, ours — see below). |
| [`food_shopping_categories.csv`](food_shopping_categories.csv) | 326,759 | `open_nutrition_id, shopping_category`. One of 24 shop departments per food, so a shopping list can be grouped the way a shop is laid out. Derived from the food's name, brand, ingredients and description. |
| [`shopping_categories.csv`](shopping_categories.csv) | 24 | `shopping_category, store_position, name_en, name_pl`. The departments themselves, in the order a shopper walks a shop. Our own taxonomy, not OpenNutrition's; published so the file above can be read. |
| [`food_data_flags.csv`](food_data_flags.csv) | 346 | `open_nutrition_id, problem`. Foods whose upstream figures we do not trust and keep out of our own search. `impossible_energy_density` (345): above 900 kcal per 100 units, which pure fat cannot reach — upstream divided a correct per-serving figure by a wrong serving weight. `junk_named_measure` (1): a named measure carries a unit that is neither a mass nor a volume, so its multiplier means nothing. These are annotations; the rows keep the values the source gave them. |
| [`food_data_repairs.csv`](food_data_repairs.csv) | 40 | `open_nutrition_id, problem, field, value_before, value_after, applied_by, applied_on`. **The only place where we overwrite a value the source gave us.** Forty foods carry a reference unit of `mg` or `iu`, neither of which is a mass or a volume, while their figures are ordinary per-100 g/ml ones. We relabel the unit — 35 to `g`, 5 to `ml` — deciding which from the size of each food's own named measures (a `cup` of 236 is a cup, a `cup` of 38 is a weight per cup). Only the label changes: amounts, calories, serving sizes and multipliers are untouched. |
| [`food_icons.csv`](food_icons.csv) | 1,225 | `open_nutrition_id, icon_key`. Which of our 188 ingredient icons a food is drawn with. The key only — the artwork is ours and is not part of this database (see below). |
| [`food_type_visibility.csv`](food_type_visibility.csv) | 4 | `food_type, foods, offered_in_search_by_default, note`. Which of the four upstream food types our search offers when the caller asks for no type in particular. `grocery` and `restaurant` — 97% of the catalogue, all North-American — are left out. No row is changed or removed by this: it is a condition in a query, recorded here because it is the honest answer to "what does your copy show?". |
| [`removed_foods.csv`](removed_foods.csv) | 0 | `open_nutrition_id, action, merged_into_open_nutrition_id, reason`. Foods we deleted or merged into another. **Empty:** we have deleted and merged nothing. Every one of the 326,759 upstream foods is present in our database, which the export verifies against the dataset file rather than asserting. |
| [`added_foods.csv`](added_foods.csv) | 0 | `smakly_food_id, name_en, food_type, calories_per_100, reference_unit, serving_amount`. Generic foods of our own, with no upstream counterpart. **Empty:** we have added none yet. When we do, they will appear here in full, as ODbL §4.6 b requires of additional Contents. |

## Applying the changes

Start from the OpenNutrition dataset (`opennutrition_foods.tsv`, column `id`),
then left-join each file on `open_nutrition_id`. Nothing here needs to be
applied in any particular order, and none of these files removes or rewrites a
row of the source except `food_data_repairs.csv`, which names the field it
changes.

```python
import json
import pandas as pd

foods = pd.read_csv("opennutrition_foods.tsv", sep="\t")            # the source
pl     = pd.read_csv("food_names_pl.csv")
cats   = pd.read_csv("food_shopping_categories.csv")

pl["alternate_names_pl"] = pl["alternate_names_pl"].map(json.loads)

derived = (foods
           .merge(pl,   left_on="id", right_on="open_nutrition_id", how="left")
           .merge(cats, left_on="id", right_on="open_nutrition_id", how="left"))
```

In SQL, the same thing: load each CSV into a temp table and update or join on
the upstream id. `food_data_repairs.csv` is applied by setting `field` to
`value_after` for the food, after checking that it still holds `value_before` —
if it does not, upstream has changed the row and the repair should be
reconsidered rather than forced.

## What is deliberately not here

- **How the changes were made.** §4.6 b asks for the alterations *or* the method
  of making them; we publish the alterations. The tooling that produced them —
  our scripts, rules and internal evaluation data — stays private.
- **The icon artwork.** The images are our own drawings, not data taken from
  OpenNutrition and not part of any database. `food_icons.csv` gives the key, so
  the grouping — which foods we consider to look alike — is fully published;
  the pictures are not.
- **Fields our search index computes for itself**: popularity and ranking
  scores, index-internal ordering. They describe a position in our index rather
  than a food, they are recomputed from our own usage, and they say nothing
  about the source data.
- **Anything about our users.** Recipes, meal plans, shopping lists and search
  logs are databases of their own and independent of this one.

## Refresh

Regenerated from our production catalogue whenever a batch of alterations lands
— new translations, a fresh pass over the shopping departments, a data repair —
and at least once per upstream dataset import. Every release is tagged with the
date of the import it reflects; a later revision of the same import adds a
suffix, so `2026-01-21.1` is a second pass over the data imported on
2026-01-21. The files are produced by a script, not
by hand, so a diff between two releases is exactly what changed.

Corrections and questions: open an issue, or write to the address on
<https://smakly.app>. Corrections to the source data itself belong with
OpenNutrition (`opensource@opennutrition.app`), and we would rather send a fix
upstream than carry it here.

---

## Po polsku (skrót)

To repozytorium zawiera **wykaz zmian**, jakie Smakly wprowadza w bazie
produktów spożywczych [OpenNutrition](https://www.opennutrition.app)
(licencja ODbL 1.0, zawiera dane [Open Food Facts](https://world.openfoodfacts.org)).
Licencja wymaga, by przy publicznym użyciu zmienionej bazy udostępnić albo całą
bazę, albo plik ze wszystkimi zmianami (§4.6 b) — i to drugie robimy tutaj.

Same dane źródłowe (wartości odżywcze, angielskie nazwy, składniki, kody
kreskowe) pozostają u OpenNutrition; tutaj są wyłącznie **nasze zmiany**,
kluczowane identyfikatorem produktu z OpenNutrition (`fd_…`):

- `food_names_pl.csv` — polskie nazwy i synonimy wyszukiwania (9 135 produktów),
- `dictionary_names_pl.csv` — polskie nazwy jednostek, składników odżywczych,
  etykiet i działów sklepowych (898),
- `food_shopping_categories.csv` — dział sklepowy każdego produktu (326 759),
- `shopping_categories.csv` — lista 24 działów w kolejności obchodzenia sklepu,
- `food_data_flags.csv` — produkty z błędnymi danymi źródłowymi, których nie
  pokazujemy (346),
- `food_data_repairs.csv` — jedyne poprawki wartości źródłowych: 40 błędnych
  jednostek odniesienia (`mg`/`iu` → `g`/`ml`),
- `food_icons.csv` — przypisanie ikony do produktu (1 225; same klucze, bez
  grafik, które są nasze),
- `food_type_visibility.csv` — które typy produktów pokazuje wyszukiwarka,
- `removed_foods.csv`, `added_foods.csv` — puste: nie usunęliśmy, nie scaliliśmy
  ani nie dodaliśmy żadnego produktu.

Wszystko na licencji **ODbL 1.0** (plik `LICENSE`). Nie publikujemy narzędzi,
którymi zmiany powstały, grafik ikon ani pól technicznych naszego indeksu
wyszukiwania.
