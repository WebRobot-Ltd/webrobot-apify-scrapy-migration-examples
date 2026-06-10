# Example 05 — Apify Playwright + **Camoufox** → WebRobot

**Source:** `apify/actor-templates` → `templates/js-crawlee-playwright-camoufox` (Apache-2.0).
**Pattern:** same browser crawl as 04, but launched through **Camoufox** (anti-fingerprint Firefox) with **`geoip`** proxy.

> 🎯 **Strongest migration story:** this Apify template's whole point — Camoufox + geo-IP proxy — is **WebRobot's default runtime**. What Apify ships as a special template is what WebRobot does out of the box.

## Original (Apify + camoufox-js)
`src/main.js`:
```js
import { PlaywrightCrawler } from '@crawlee/playwright';
import { launchOptions as camoufoxLaunchOptions } from 'camoufox-js';
import { firefox } from 'playwright';

const crawler = new PlaywrightCrawler({
    proxyConfiguration,
    requestHandler: router,
    launchContext: {
        launcher: firefox,
        launchOptions: await camoufoxLaunchOptions({
            headless: true,
            proxy: await proxyConfiguration.newUrl(),
            geoip: true,                      // geo-located proxy
        }),
    },
});
await crawler.run(['https://apify.com']);
```
Plus the camoufox-js dep, proxy wiring, and the Apify platform.

## WebRobot equivalent — [`pipeline.yaml`](pipeline.yaml)
```yaml
pipeline:
  - stage: fetch                       # WebRobot's runtime IS Camoufox — no setup
    args: ["https://apify.com"]
  - stage: explore
    args: ["a[href*='apify.com']", { depth: 1 }]
  - stage: extract
    args:
      - - { selector: "title", method: text, as: title }
output: { format: parquet, mode: overwrite }
# geo-IP (= camoufox geoip:true): set per-pipeline, e.g. metadata.runtime
# DATAIMPULSE_PROXY_COUNTRY / __cr.<country> — built-in, no proxy plumbing.
```

## Mapping
| Apify camoufox template | WebRobot |
|---|---|
| `camoufox-js` + `firefox` launcher | **`fetch`** (Camoufox is the engine) |
| `geoip: true` + `proxyConfiguration.newUrl()` | built-in DataImpulse proxy + per-session geo (`__cr.<country>`) |
| anti-fingerprint setup | default |
| `enqueueLinks` / detail handler | `explore` + `extract` |

**Apify's "advanced" Camoufox template = WebRobot's zero-config default.**
