# Example 02 — Scrapy `quotesbot` → WebRobot

**Source:** `scrapy/quotesbot` → `quotesbot/spiders/toscrape-css.py` (BSD).
**Pattern:** list of items per page (`div.quote`) + pagination (`li.next`), the canonical Scrapy tutorial spider.

## Original (Scrapy)
```python
import scrapy

class ToScrapeCSSSpider(scrapy.Spider):
    name = "toscrape-css"
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        for quote in response.css("div.quote"):
            yield {
                'text':   quote.css("span.text::text").extract_first(),
                'author': quote.css("small.author::text").extract_first(),
                'tags':   quote.css("div.tags > a.tag::text").extract(),
            }
        next_page_url = response.css("li.next > a::attr(href)").extract_first()
        if next_page_url is not None:
            yield scrapy.Request(response.urljoin(next_page_url))
```
Plus `scrapy.cfg`, `settings.py`, the project scaffold, and you run/scale Scrapy yourself.

## WebRobot equivalent
`pipeline.yaml`:
```yaml
pipeline:
  - stage: wget
    args: ["http://quotes.toscrape.com/"]
  - stage: wgetExplore                 # = yield Request(next_page): pagination
    args:
      - "li.next > a"                   # the "next" link selector
      - { depth: 10 }                   # how many pages to follow
  - stage: flatSelect                   # = `for quote in response.css("div.quote")`
    args:
      - "div.quote"                     # one output row per quote
      - - { selector: "span.text",   method: text, as: text }
        - { selector: "small.author", method: text, as: author }
        - { selector: "div.tags > a.tag", method: text, as: tags }
output:
  format: parquet
  mode: overwrite
```

## Side-by-side notes
| Scrapy | WebRobot |
|---|---|
| `scrapy.Spider` + `start_urls` | `wget` |
| `for quote in response.css("div.quote")` | `flatSelect("div.quote", …)` (row per match) |
| `quote.css("span.text::text").extract_first()` | field `method: text` |
| `::attr(href)` | `method: href` |
| `li.next > a` + `yield Request(urljoin(...))` | `wgetExplore("li.next > a", depth)` |
| `yield {item}` | `output` (parquet/csv) |
| `settings.py` / project scaffold / self-host | — (declarative, Spark-distributed, BYOC) |

**~20 lines of spider + Scrapy project → ~12 lines of YAML.**

> Nuance — **multi-value `tags`**: Scrapy's `.extract()` returns a list. A single `flatSelect` field grabs one value; to collect all tags per quote into an array, either nest a sub-select or post-process with a `python_extensions` `row_transform` (`webrobot_llm` not needed — pure string collect). Documented so the port stays honest.
