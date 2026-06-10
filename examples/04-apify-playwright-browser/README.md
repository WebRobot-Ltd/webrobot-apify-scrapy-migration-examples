# Example 04 — Apify PlaywrightCrawler (browser) → WebRobot

**Source:** `apify/actor-templates` → `templates/js-crawlee-playwright-chrome` (Apache-2.0).
**Pattern:** browser (JS-rendered) crawl, `enqueueLinks` list→detail, `page.title()` per detail page.

**Covers equally** (all browser crawlers → WebRobot `fetch`): `*-crawlee-playwright-chrome`, `*-crawlee-puppeteer-chrome`, `python-crawlee-playwright`, `python-playwright`, `python-selenium`.

## Original (Apify / Crawlee Playwright)
`src/main.js`:
```js
import { PlaywrightCrawler } from '@crawlee/playwright';
const crawler = new PlaywrightCrawler({ proxyConfiguration, requestHandler: router });
await crawler.run(['https://apify.com']);
```
`src/routes.js`:
```js
router.addDefaultHandler(async ({ enqueueLinks }) => {
    await enqueueLinks({ globs: ['https://apify.com/*'], label: 'detail' });
});
router.addHandler('detail', async ({ request, page, pushData }) => {
    await pushData({ url: request.loadedUrl, title: await page.title() });
});
```

## WebRobot equivalent — [`pipeline.yaml`](pipeline.yaml)
```yaml
pipeline:
  - stage: fetch                       # PlaywrightCrawler = browser render (Camoufox)
    args: ["https://apify.com"]
  - stage: visitExplore                # = enqueueLinks({globs}): browser link-follow (no trace needed)
    args:
      - "a[href*='apify.com']"
      - { depth: 1 }
  - stage: extract                     # = detail handler: page.title()
    args:
      - - { selector: "title", method: text, as: title }
output: { format: parquet, mode: overwrite }
```

## Mapping
| Crawlee Playwright | WebRobot |
|---|---|
| `PlaywrightCrawler` / Puppeteer / Selenium | `fetch` (browser, Camoufox) |
| `enqueueLinks({globs,label})` | `visitExplore` (browser link-follow; no trace) |
| `addHandler('detail', … page.title())` | `extract` on the followed pages |
| `page.click/fill` before extracting | `fetch`/`explore` with a `trace` (click/fill/hover actions) |

> **Stage note:** browser link-following = **`visitExplore`** (just visits each followed link). Use **`explore`** only when you need to *replay actions* per page (it requires a `trace`). HTTP link-following (no browser) = `wgetExplore`.
| `proxyConfiguration` | built-in proxy + geo |

**A Playwright actor + Crawlee runtime → a `fetch` stage on WebRobot's managed browser farm.**
