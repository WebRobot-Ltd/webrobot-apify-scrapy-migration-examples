# Mapping cheat-sheet — Crawlee / Scrapy / Zyte → WebRobot

The construct-by-construct translation table behind every example. Also loaded into the demo agent's context so it can migrate autonomously.

## WebRobot target stages (quick reference)
- **`wget(url)`** — pure HTTP, static pages / APIs (no browser).
- **`fetch(url, trace?)`** — browser (Camoufox); `trace` = actions: `visit`, `fill`, `click`, `hover`, `select`. Use for JS-rendered pages, search, login, age-gate.
- **link-following / pagination** — pick by mode (takes a link selector + `depth`):
  - **`wgetExplore`** — **static / HTTP** (no browser).
  - **`visitExplore`** — **browser** (Camoufox), just visits each followed link — **no trace required**.
  - **`explore`** — browser, **requires a `trace`** — use only when you must *replay actions* per page (click "load more", paginate via a button, etc.).
- **`join` / `wgetJoin` / `visitJoin`** — fan out: per row, fetch a detail URL and merge.
- **`flatSelect(segment, [fields])`** — repeating rows (list items) → one output row each.
- **`extract([fields])`** — fields from the current page.
- **field** = `{ selector, method, as }`; `method` ∈ `text` · `html` · `href` (absolute URL) · `src` · `attr:NAME`.
- **`sql_query`** — SQL over the dataframe.
- **`python_extensions`** — `row_transform` (per-row UDF) · `dataframe_transform` (driver, Py4J) · `sql_query`; helper **`webrobot_llm(prompt, …)`** for safe LLM calls in the sandbox.
- **`iextract`** — LLM-native intelligent extraction (article/product/fields).
- **`load_csv` / `metadata.params`** — input dataset / `$col` sweep over many URLs/queries.
- **`output`** — parquet/csv → WebRobot dataset.

---

## Apify / Crawlee → WebRobot
| Crawlee / Apify | WebRobot |
|---|---|
| `CheerioCrawler` (HTTP) | `wget` |
| `PlaywrightCrawler` / `PuppeteerCrawler` / `web-scraper` (browser) | `fetch(url, trace)` |
| `enqueueLinks()` — Cheerio (HTTP) | `wgetExplore` (+`depth`) |
| `enqueueLinks()` — Playwright/Puppeteer (browser) | `visitExplore` (+`depth`); `explore` only if replaying actions |
| router `addHandler('detail')` (list→detail) | list `flatSelect` → `join`/`explore` → detail `extract` |
| `$('sel').text()` | `method: text` |
| `$('sel').attr('href')` | `method: href` (absolute) |
| `$('sel').attr('X')` | `method: attr:X` |
| custom JS in `pageFunction` / data shaping | `python_extensions` (`row_transform` / `dataframe_transform`) |
| LLM call inside `pageFunction` | `webrobot_llm(…)` |
| `proxyConfiguration` (Apify Proxy) | built-in proxy + per-session geo (`__cr.<country>`) |
| `INPUT_SCHEMA.startUrls` | `metadata.params` / `load_csv` (`$col` sweep) |
| `pushData({…})` → Apify Dataset | `output` (parquet/csv) → dataset |
| `Actor.init()/exit()`, platform glue | — (declarative, Spark-native) |

## Scrapy → WebRobot
| Scrapy | WebRobot |
|---|---|
| `start_urls` / `start_requests` | `wget`/`fetch` + `metadata.params` |
| `def parse(self, response)` | the pipeline stages (`flatSelect`/`extract`) |
| `response.css('sel::text').get()` | `method: text` |
| `response.css('a::attr(href)').get()` | `method: href` |
| `response.css('sel::attr(X)')` | `method: attr:X` |
| `response.xpath('…')` | `selector` (prefer CSS; xpath where supported) |
| `response.follow(next)` / `yield Request(url)` | `wgetExplore` (HTTP); `visitExplore` if scrapy-playwright |
| `CrawlSpider` `Rule(LinkExtractor(...))` | `wgetExplore` with the link selector |
| `yield {item}` / `Item` / `ItemLoader` | `output` + `python_extensions` (`row_transform`) |
| `FormRequest` (login/search) | `fetch` with `trace` (fill + submit actions) |
| item pipelines (clean/dedup/store) | `sql_query` / `python_extensions` / `rag_dedup` |
| `scrapy-playwright` / Splash (JS) | `fetch` (browser) |
| `settings`: DOWNLOAD_DELAY / AUTOTHROTTLE / proxies | runtime config + built-in proxy |

## Zyte → WebRobot
| Zyte | WebRobot |
|---|---|
| Scrapy Cloud spider | *(same as Scrapy above)* |
| Zyte Smart Proxy Manager | built-in proxy + geo |
| Zyte API auto-extract: **product** | `iextract` / product-extraction pattern |
| Zyte API auto-extract: **article** | `iextract` (article) + sentiment via `webrobot_llm` |
| Zyte API `browserHtml` (JS render) | `fetch` (browser / Camoufox) |
| Scrapy Cloud (their compute) | BYOC Spark (your cloud) |

---

## When you genuinely need procedural code
Crawlee `pageFunction` logic or Scrapy item-pipeline logic that's truly imperative → **`python_extensions`**:
- **`row_transform`** — per-row UDF (no DB; `webrobot_llm` available).
- **`dataframe_transform`** — driver-side, can `import pyspark.sql`.
- **`sql_query`** — declarative SQL.
- **`webrobot_llm(prompt, …)`** — sandbox-safe LLM call (key injected from `cloud_credentials` at submit time).
