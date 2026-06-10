# Example 07 — Scrapy `booksbot` (list → detail) → WebRobot

**Source:** `scrapy/booksbot` → `books/spiders/books.py` (educational, scrapes books.toscrape.com).
**Pattern:** **list → detail** — paginate a listing, follow each product link to its detail page, extract product fields there. The classic e-commerce shape.

## Original (Scrapy)
```python
class BooksSpider(scrapy.Spider):
    name = "books"
    start_urls = ['http://books.toscrape.com/']

    def parse(self, response):
        # follow each product to its detail page
        for book_url in response.css("article.product_pod > h3 > a ::attr(href)").extract():
            yield scrapy.Request(response.urljoin(book_url), callback=self.parse_book_page)
        # pagination
        next_page = response.css("li.next > a ::attr(href)").extract_first()
        if next_page:
            yield scrapy.Request(response.urljoin(next_page), callback=self.parse)

    def parse_book_page(self, response):
        product = response.css("div.product_main")
        yield {
            "title":       product.css("h1 ::text").extract_first(),
            "category":    response.xpath("//ul[@class='breadcrumb']/li[@class='active']/preceding-sibling::li[1]/a/text()").extract_first(),
            "description": response.xpath("//div[@id='product_description']/following-sibling::p/text()").extract_first(),
            "price":       response.css('p.price_color ::text').extract_first(),
        }
```

## WebRobot equivalent — [`pipeline.yaml`](pipeline.yaml)
```yaml
pipeline:
  - stage: wget
    args: ["http://books.toscrape.com/"]
  - stage: wgetExplore                 # pagination: all listing pages (li.next)
    args: ["li.next > a", { depth: 50 }]
  - stage: wgetExplore                 # follow each product link → its detail page
    args: ["article.product_pod > h3 > a", { depth: 1 }]
  - stage: extract                     # = parse_book_page: detail fields
    args:
      - - { selector: "div.product_main h1", method: text, as: title }
        - { selector: "p.price_color",       method: text, as: price }
        - { selector: "#product_description ~ p", method: text, as: description }
        - { selector: "ul.breadcrumb li:nth-last-child(2) a", method: text, as: category }
output: { format: parquet, mode: overwrite }
```

## Mapping
| Scrapy booksbot | WebRobot |
|---|---|
| `parse`: `yield Request(book_url, parse_book_page)` (follow to detail) | second `wgetExplore` on the product-link selector |
| `parse`: `li.next` recursion (pagination) | first `wgetExplore` on `li.next > a` |
| `parse_book_page`: `css/xpath` fields | `extract` fields on the detail page |
| `response.urljoin(...)` | handled automatically (absolute follow) |
| two callbacks (`parse` + `parse_book_page`) | two `wgetExplore` stages + one `extract` |

> **list→detail** is the e-commerce workhorse. Static site → two `wgetExplore` (paginate, then follow product links) + `extract`. For a JS-rendered shop use `fetch` + `visitExplore`. Scrapy's `xpath` breadcrumb/description map to CSS approximations here (`:nth-last-child`, sibling `~`); use exact xpath where the site needs it.
