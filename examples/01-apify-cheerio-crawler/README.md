# Example 01 — Apify CheerioCrawler → WebRobot

**Source:** `apify/actor-templates` → `templates/js-crawlee-cheerio` (Apache-2.0).
**Pattern:** HTTP crawl, follow links (`enqueueLinks`), extract `title` per page.

## Original (Apify / Crawlee)

`src/main.js` (trimmed):
```js
import { CheerioCrawler } from '@crawlee/cheerio';
import { Actor } from 'apify';
import { router } from './routes.js';

await Actor.init();
const { startUrls = ['https://crawlee.dev'], maxRequestsPerCrawl = 100 } = (await Actor.getInput()) ?? {};
const proxyConfiguration = await Actor.createProxyConfiguration({ checkAccess: true });

const crawler = new CheerioCrawler({ proxyConfiguration, maxRequestsPerCrawl, requestHandler: router });
await crawler.run(startUrls);
await Actor.exit();
```

`src/routes.js`:
```js
import { createCheerioRouter } from '@crawlee/cheerio';
export const router = createCheerioRouter();

router.addDefaultHandler(async ({ enqueueLinks, request, $, log, pushData }) => {
    await enqueueLinks();                              // follow links
    const title = $('title').text();                  // extract
    await pushData({ url: request.loadedUrl, title }); // store
});
```
Plus `.actor/input_schema.json` (startUrls, maxRequestsPerCrawl), a `Dockerfile`, `package.json`, the Crawlee + Apify SDK deps, and the Apify platform to run on.

## WebRobot equivalent

`pipeline.yaml`:
```yaml
pipeline:
  - stage: wget                      # CheerioCrawler = pure HTTP
    args: ["https://crawlee.dev"]
  - stage: wgetExplore               # = enqueueLinks(): follow links
    args:
      - "a"                          # link selector
      - { depth: 1 }                 # ~ maxRequestsPerCrawl / crawl depth
  - stage: extract
    args:
      - - { selector: "title", method: text, as: title }
                                     # url is captured automatically
output:
  format: parquet
  mode: overwrite
```

## Side-by-side notes
| Crawlee | WebRobot |
|---|---|
| `CheerioCrawler` | `wget` |
| `enqueueLinks()` | `wgetExplore` (`a`, `depth`) |
| `$('title').text()` | `extract` field `method: text` |
| `request.loadedUrl` in `pushData` | captured automatically |
| `proxyConfiguration` | built-in proxy (no setup) |
| `startUrls` input | `wget` arg / `metadata.params` for a sweep |
| `pushData` → Apify Dataset | `output` → WebRobot dataset (parquet) |
| Actor.init/exit + Dockerfile + deps + Apify platform | — none — |

**~30 lines of JS + Crawlee dependency + Apify platform → ~10 lines of declarative YAML, distributed on Spark, runs on your own cloud.**

> Tip: to extract more than `title`, add fields to `extract` (e.g. `{ selector: "h1", method: text, as: heading }`, `{ selector: "a.next", method: href, as: next_url }`). For list→detail pages, use `flatSelect` for the rows then `join`/`explore` into the detail page.
