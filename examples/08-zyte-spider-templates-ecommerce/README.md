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
  - stage: fetch                       # browser render + built-in proxy/geo
    args: ["https://books.toscrape.com/"]
  - stage: intelligentExplore          # = Zyte ProductNavigation heuristics: auto-find product/category/next
    args:
      - { goal: "product detail pages", depth: 2 }
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
| `Product` auto-extract (`parse_product`) | `iextract` (LLM-native) |
| `extract_from: browserHtml` | `fetch` (Camoufox) |
| `geolocation` param | per-pipeline geo (`__cr.<country>`) |
| Zyte API + Scrapy Cloud (paid, their compute) | WebRobot + **BYOC** (your cloud + your model) |
| `crawl_strategy` (full / navigation / pagination) | `intelligentExplore` `goal` + `depth` |

> Zyte's flagship "give it a domain, get products" template → WebRobot's **intelligent** pattern (`intelligentExplore` + `iextract`): no hand selectors, runs on your own cloud and model. Same idea, no per-request extraction fee. For deterministic fields, pair `iextract` with explicit `extract` selectors (see example 07).
