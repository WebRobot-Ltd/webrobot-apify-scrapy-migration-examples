<div align="center">

# Migrate to WebRobot — Apify · Scrapy · Zyte

**Side-by-side proof that any competitor scraper has a shorter, declarative [WebRobot](https://webrobot.eu) equivalent — and a demo agent that migrates it for you.**

</div>

---

## 🤖 The demo agent migrates your scraper — automatically

The **WebRobot demo chatbot agent** (try it at [portal.webrobot.eu](https://portal.webrobot.eu)) ships with **built-in migration capabilities**. You don't port by hand:

> **You:** "Migrate this Apify actor to WebRobot: `github.com/apify/…`"
>
> **Agent:** reads the source straight from GitHub (via the GitHub MCP), recognizes the framework pattern (Crawlee `CheerioCrawler`, Scrapy `parse`/`CrawlSpider`, Zyte auto-extract…), generates the equivalent **WebRobot pipeline YAML**, **runs it live**, and shows you the **side-by-side**.

Paste an **Apify actor**, a **Scrapy spider**, or a **Zyte** project URL → get back a working WebRobot pipeline. No clone, no rewrite. This repo is the curated proof + the knowledge base behind that capability.

---

## Why migrate

| | Apify | Scrapy | Zyte | **WebRobot** |
|---|---|---|---|---|
| **Authoring** | imperative JS + Crawlee boilerplate | Python spider + callbacks + settings | Scrapy + Zyte API calls | **declarative YAML (~10 lines)** |
| **Runtime** | Apify platform (locked-in) | self-hosted, you scale it | Scrapy Cloud (their compute) | **Spark-distributed, BYOC — your cloud** |
| **Anti-block** | Apify Proxy (paid add-on) | DIY | Zyte Smart Proxy / API (paid) | **built-in proxy + per-session geo + captcha HITL** |
| **Enrichment** | custom JS in pageFunction | item pipelines | Zyte API | **sandboxed `python_extensions` + `webrobot_llm`** |
| **AI extraction** | — | — | Zyte auto-extract | **intelligent `iextract` (LLM-native)** |

**Same result, a fraction of the code, on your own infrastructure, AI-native.**

---

## What we port from

- **Apify** — `apify/actor-templates`, `apify/crawlee` examples, the generic `*-scraper` actors, and open Store actors that publish a GitHub repo.
- **Scrapy** — `scrapy/quotesbot` and the large community spider base (`parse` callbacks, `CrawlSpider` + `Rule`/`LinkExtractor`).
- **Zyte** — Scrapy Cloud spiders (= Scrapy) and Zyte API auto-extract (article / product) usages.

> The agent reads upstream source **without cloning** — through the **GitHub MCP** (`get_file_contents`, `search_code`, `search_repositories`). For manual curation: tree via `api.github.com/repos/<org>/<repo>/git/trees/<branch>?recursive=1`, files via `raw.githubusercontent.com/...`.

---

## Repo layout

| Path | What |
|---|---|
| [`MAPPING.md`](MAPPING.md) | Construct-by-construct cheat-sheet: Crawlee / Scrapy / Zyte → WebRobot stages. The core artifact — also loaded into the agent's context. |
| [`agent-migration-feature.md`](agent-migration-feature.md) | How the demo agent performs migrations self-serve (GitHub MCP access + workflow + the profile `mcp_servers` entry). |
| [`examples/`](examples/) | One file per ported scraper: **original** competitor code + **equivalent** WebRobot YAML + notes. |

### Examples
Each folder has the original competitor snippet, the mapping README, and a real runnable [`pipeline.yaml`](examples/).

| # | Source | Pattern | Also covers |
|---|---|---|---|
| [01](examples/01-apify-cheerio-crawler/) | **Apify** CheerioCrawler | HTTP crawl + follow links + extract | js/ts-crawlee-cheerio |
| [02](examples/02-scrapy-quotesbot/) | **Scrapy** `quotesbot` | list of items + pagination | toscrape-css/xpath, python-scrapy |
| [03](examples/03-zyte-auto-extract/) | **Zyte** API auto-extract | ML product/article → `iextract` | zyte-api product/article |
| [04](examples/04-apify-playwright-browser/) | **Apify** PlaywrightCrawler | browser, list→detail | puppeteer, selenium, python-playwright |
| [05](examples/05-apify-playwright-camoufox/) | **Apify** Playwright + **Camoufox** | browser + geo-IP (= WebRobot's default runtime) | camoufox-js templates |
| [06](examples/06-apify-beautifulsoup-parsel/) | **Apify** Python BeautifulSoup/Parsel | HTTP parse | crawlee-beautifulsoup, parsel, bootstrap-cheerio |
| [07](examples/07-scrapy-booksbot-list-detail/) | **Scrapy** `booksbot` | **list → detail** (paginate, follow product, extract on detail) | e-commerce listings |
| [08](examples/08-zyte-spider-templates-ecommerce/) | **Zyte** `zyte-spider-templates` | auto-discover + auto-extract → `intelligentExplore` + `iextract` | zyte ecommerce/article templates |

---

## How a migration reads (preview)

**Apify CheerioCrawler** (main.js + routes.js, ~30 lines + Crawlee dep):
```js
const crawler = new CheerioCrawler({ proxyConfiguration, requestHandler: router });
router.addDefaultHandler(async ({ enqueueLinks, $, request, pushData }) => {
    await enqueueLinks();
    await pushData({ url: request.loadedUrl, title: $('title').text() });
});
await crawler.run(startUrls);
```

**WebRobot equivalent** (`pipeline.yaml`):
```yaml
pipeline:
  - stage: wget
    args: ["https://crawlee.dev"]
  - stage: wgetExplore          # = enqueueLinks() — follow links
    args: ["a", { depth: 1 }]
  - stage: extract
    args:
      - - { selector: "title", method: text, as: title }
output: { format: parquet, mode: overwrite }
```

See [`examples/`](examples/) for the full worked migrations.

---

## License

Ports and docs in this repo: **Apache-2.0**. Each example references its upstream under that project's own license (mostly Apache-2.0 / MIT / BSD) — verify before forking a specific actor (some Apify Store actors are closed/paid; Zyte API requires their service).
