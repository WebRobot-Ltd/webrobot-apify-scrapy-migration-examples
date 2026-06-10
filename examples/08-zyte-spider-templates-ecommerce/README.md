# Example 08 — Zyte `zyte-spider-templates` (e-commerce auto-crawl) → WebRobot

**Source:** `zytedata/zyte-spider-templates` → `zyte_spider_templates/spiders/ecommerce.py`.
**Pattern:** point the template at a domain → it **auto-discovers product pages** (navigation heuristics) and **auto-extracts** `Product` items via Zyte API. No selectors written by hand.

## Original (Zyte spider template)
```python
from zyte_common_items import Product, ProductNavigation
from zyte_spider_templates.spiders.base import BaseSpider
# ...
class EcommerceSpider(BaseSpider):
    name = "ecommerce"
    # input: url (a domain / category page), crawl_strategy, extract_from, geolocation
    # navigation heuristics yield product + nav requests; pages are auto-extracted as Product
    def parse_navigation(self, response, navigation: ProductNavigation):
        for req in navigation.items:        # product detail requests
            yield req
        for req in navigation.subCategories or []:
            yield req
        if navigation.nextPage:
            yield navigation.nextPage

    def parse_product(self, response, product: Product):
        yield product                       # Zyte API auto-extracted the fields
```
Runs on Scrapy + Zyte API (paid auto-extract + navigation heuristics + smart proxy).

## WebRobot equivalent — [`pipeline.yaml`](pipeline.yaml)
```yaml
pipeline:
  - stage: wget                        # static start (books.toscrape); use `fetch` for JS shops
    args: ["https://books.toscrape.com/"]
  - stage: intelligentExplore          # = Zyte ProductNavigation: LLM-infers pagination/category/product links
    args:
      - "listing pages and product links"   # natural-language goal
      - 2                                    # depth (visitLimit)
  - stage: intelligentJoin             # follow inferred product links to detail pages
    args:
      - "product detail page"
      - { useWget: true }              # HTTP, no browser (static); default is Visit/browser
  - stage: iextract                    # = Zyte Product auto-extract (LLM-native)
    args:
      - prompt: >
          Extract the product: name, price (number), currency, sku,
          availability, description, brand, breadcrumbs.
        as: product
output: { format: parquet, mode: overwrite }
```

## Mapping
| Zyte spider template | WebRobot |
|---|---|
| `ProductNavigation` heuristics (auto-find products/nav/next) | `intelligentExplore` (LLM-guided link discovery) |
| follow nav → product detail | `intelligentJoin` |
| `Product` auto-extract (`parse_product`) | `iextract` (LLM-native) |
| `extract_from: browserHtml` (browser) | default of `intelligentExplore`/`intelligentJoin` (Visit) — or `fetch` |
| static / HTTP crawl | `useWget: true` on `intelligentJoin` (and a Wget traceAction on `intelligentExplore`) |
| `geolocation` param | per-pipeline geo (`__cr.<country>`) |
| Zyte API + Scrapy Cloud (paid, their compute) | WebRobot + **BYOC** (your cloud + your model) |

> **Acquisition is a parameter, not a stage.** `intelligentExplore` and `intelligentJoin` default to **Visit (browser)**; for a static site set `useWget: true` on `intelligentJoin` (and pass a Wget traceAction to `intelligentExplore`). Zyte's flagship "give it a domain, get products" template → WebRobot's **intelligent** pattern with no hand selectors, **no per-request extraction fee**, on your own cloud + model. For deterministic fields, pair `iextract` with explicit `extract` selectors (see example 07).
