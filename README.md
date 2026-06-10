<div align="center">

# Migrate to WebRobot — Apify · Scrapy · Zyte

**Side-by-side proof that any competitor scraper has a shorter, declarative [WebRobot](https://webrobot.eu) equivalent — and a demo agent that migrates it for you.**

_WebRobot is the **first ETL built to be consumed by AI agents** — declarative pipelines an agent can author, run, and read back. This migration capability is the proof._

</div>

---

## 🤖 The demo agent migrates your scraper — automatically

The **WebRobot demo chatbot agent** — **[try the conversational demo → portal.webrobot.eu/chat.html](https://portal.webrobot.eu/chat.html)** *(live soon)* — ships with **built-in migration capabilities**. You don't port by hand:

> **You:** "Migrate this Apify actor to WebRobot: `github.com/apify/…`"
>
> **Agent:** reads the source straight from GitHub (via the GitHub MCP), recognizes the framework pattern (Crawlee `CheerioCrawler`, Scrapy `parse`/`CrawlSpider`, Zyte auto-extract…), generates the equivalent **WebRobot pipeline YAML**, **runs it live**, and shows you the **side-by-side**.

Paste an **Apify actor**, a **Scrapy spider**, or a **Zyte** project URL → get back a working WebRobot pipeline. No clone, no rewrite. This repo is the curated proof + the knowledge base behind that capability.

---

## 🔒 The real positioning: your own **private crawler fleet** (BYOC)

The closest thing to WebRobot isn't a hosted scraping marketplace — it's **running your own private crawlers**, but without building and babysitting the infrastructure. With **BYOC (Bring Your Own Cloud)** the whole pipeline runs on **your** account:

- **Your cloud, your machines** — Spark + Camoufox browser workers are provisioned on *your* Hetzner / AWS / etc., scaled elastically up and down per job.
- **Your data never leaves your infra** — results land in *your* storage; no third-party platform sees or holds the data.
- **No per-request / per-extraction fees** — unlike Apify compute units or Zyte API calls. You pay your own cloud, nothing per page or per extraction.
- **Your proxies, your models** — bring your own proxy pool and your own LLM (via `cloud_credentials`) for `iextract` / `webrobot_llm`. Nothing is sent to a vendor's model by default.
- **Declarative + managed, not DIY** — you get a private crawler's control and economics, but you author in **YAML** on a managed browser farm with built-in anti-block and captcha HITL — none of the Scrapy self-hosting toil.

> Migrating off Apify/Zyte isn't just shorter code — it's **bringing your crawling in-house** (data sovereignty + cloud economics) **without** rebuilding the platform. That's the migration these examples are really about.

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

## 📊 Beyond the scraper — a complete ETL + analytics stack, in one place

Apify / Scrapy / Zyte stop at **acquisition**: they hand you raw items, then *you* bolt on a warehouse, transforms, analytics and BI. WebRobot is a **complete ETL** — scraping is just stage one of the same pipeline:

- **Transform in-pipeline** — `sql_query`, `python_extensions` (row/dataframe UDFs), dedup, joins. No separate ETL tool.
- **Advanced analytics built in** — aggregate / score / enrich on Spark; LLM enrichment via `webrobot_llm`; RAG stages (`rag_ingest` / `rag_query`).
- **Vertical use cases, shipped** — price comparison, real-estate arbitrage, product catalogs, comment/article sentiment, odds/sure-bet, and a closed loop with a distributed trading engine.
- **One stack** — scrape → transform → analyze → serve, all on Spark / BYOC, with no glue between five different tools.
- **Agent-native** — the **first ETL designed to be consumed by AI agents**: declarative YAML + MCP tools mean an agent can author a pipeline, run it, and read the result back (exactly how the migration capability works). Competitors are code libraries/platforms a *human* drives.

> The migration isn't tool-for-tool. You replace the scraper **and** the ETL + analytics layer you'd otherwise build downstream of it.

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
