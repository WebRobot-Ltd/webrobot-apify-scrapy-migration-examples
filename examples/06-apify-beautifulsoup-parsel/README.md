# Example 06 — Apify Python BeautifulSoup / Parsel (HTTP) → WebRobot

**Source:** `apify/actor-templates` → `templates/python-beautifulsoup` (Apache-2.0).
**Pattern:** plain HTTP fetch + parse HTML with BeautifulSoup; follow links.

**Covers equally** (Python/JS HTTP parsers → WebRobot `wget`): `python-crawlee-beautifulsoup`, `python-crawlee-parsel`, `js/ts-bootstrap-cheerio-crawler`.

## Original (Apify Python + BeautifulSoup)
```python
from bs4 import BeautifulSoup
from httpx import AsyncClient
from apify import Actor

async def main():
    async with Actor:
        actor_input = await Actor.get_input() or {}
        url = actor_input.get('url', 'https://apify.com')
        async with AsyncClient() as client:
            resp = await client.get(url)
        soup = BeautifulSoup(resp.content, 'html.parser')
        title = soup.title.string if soup.title else None
        links = [a['href'] for a in soup.find_all('a', href=True)]
        await Actor.push_data({'url': url, 'title': title, 'links': links})
```
(Parsel variant: `Selector(text=resp.text).css('title::text').get()` — same shape.)

## WebRobot equivalent — [`pipeline.yaml`](pipeline.yaml)
```yaml
pipeline:
  - stage: wget                        # BeautifulSoup/Parsel = pure HTTP, no browser
    args: ["https://apify.com"]
  - stage: extract
    args:
      - - { selector: "title", method: text, as: title }
        - { selector: "h1",    method: text, as: heading }
output: { format: parquet, mode: overwrite }
```

## Mapping
| BeautifulSoup / Parsel | WebRobot |
|---|---|
| `httpx.get(url)` / `requests` | `wget` |
| `soup.title.string` / `Selector.css('title::text')` | `extract` `method: text` |
| `soup.select('.x')` / `.css('.x')` | `flatSelect`/`extract` selectors |
| `a['href']` / `::attr(href)` | `method: href` |
| `find_all` loop over items | `flatSelect(segment, [fields])` |
| `Actor.push_data` | `output` (parquet/csv) |

**Python HTTP scraper → a `wget` + `extract` pipeline, no parsing code.**
