# Example 03 — Zyte API auto-extract → WebRobot

**Source pattern:** Zyte API automatic extraction (`product` / `article`) — e.g. `python-zyte-api` or `scrapy-zyte-api` usage. Representative, since Zyte API is a paid service (no forkable actor).
**Pattern:** fetch a page through Zyte's smart proxy + let Zyte's ML return structured fields.

## Original (Zyte API)
```python
from zyte_api import ZyteAPI

client = ZyteAPI(api_key="ZYTE_API_KEY")

# product auto-extract (ML-based) + JS render + smart-proxy, all server-side
resp = client.get({
    "url": "https://example.com/product/123",
    "product": True,
    "productOptions": {"extractFrom": "browserHtml"},
})
product = resp["product"]      # {name, price, currency, sku, availability, ...}
```
Runs on Zyte's compute; proxy + browser + extraction are their paid API.

## WebRobot equivalent
`pipeline.yaml` — `iextract` is LLM-native extraction; `fetch` gives the browser render; proxy/geo are built in:
```yaml
pipeline:
  - stage: fetch                       # browserHtml render (Camoufox); built-in proxy + geo
    args: ["https://example.com/product/123"]
  - stage: iextract                    # = Zyte product auto-extract (LLM-native)
    args:
      - prompt: >
          Extract the product as fields: name, price (number),
          currency, sku, availability, brand.
        as: product
output:
  format: parquet
  mode: overwrite
```
For many product URLs, drive it from a dataset (`load_csv` + `$col` sweep) instead of one URL.

## Side-by-side notes
| Zyte | WebRobot |
|---|---|
| Zyte API `"product": True` (ML auto-extract) | `iextract` (LLM-native) |
| Zyte API `"article": True` | `iextract` (article) + sentiment via `webrobot_llm` |
| `extractFrom: browserHtml` (JS render) | `fetch` (browser / Camoufox) |
| Zyte Smart Proxy Manager (paid) | built-in proxy + per-session geo (`__cr.<country>`) |
| `ZyteAPI(api_key=…)` + their compute | your `cloud_credentials` LLM key + **BYOC Spark** |
| Scrapy Cloud orchestration | Spark-distributed pipeline |

**A paid per-request extraction API → an `iextract` stage on your own cloud, with your own model.**

> If the field set must be exact/deterministic (no LLM variance), pair `iextract` with a deterministic `extract`/`flatSelect` for the known selectors and use `iextract` only for the fuzzy fields.
