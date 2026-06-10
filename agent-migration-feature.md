# Demo agent — built-in migration capability

How the WebRobot demo chatbot agent migrates a competitor scraper end-to-end, and what to wire up for it.

## User experience
> "Migrate this to WebRobot: `https://github.com/apify/actor-templates` (js-crawlee-cheerio)"
> "Port this Scrapy spider: `https://github.com/scrapy/quotesbot`"
> "Here's our Zyte project, give me the WebRobot version."

The agent replies with the equivalent **WebRobot pipeline YAML**, runs it live, and shows the side-by-side.

## Agent workflow
1. **Ingest** the GitHub URL (repo, or a path within it).
2. **Read source via the GitHub MCP** (no clone):
   - `search_code` / `get_file_contents` for `src/main.js` + `routes.js` (Apify), `*/spiders/*.py` + `settings.py` (Scrapy), Zyte API calls.
   - Optionally `git/trees` to enumerate.
3. **Classify** the pattern: HTTP (Cheerio/`requests`) vs browser (Playwright/Puppeteer/scrapy-playwright); single-page vs link-following vs list→detail; any login/search/form; any custom enrichment.
4. **Map** to a WebRobot pipeline using [`MAPPING.md`](MAPPING.md) (loaded in the agent context).
5. **Emit + run**: `generatePipeline` → `saveGeneratedPipeline` (execute=true) → `getExecutionStatus`/`getExecutionOutput`.
6. **Present** the side-by-side (original vs YAML) + the live output.

## Wiring: GitHub MCP in the agent profile
The agent mounts external MCP servers from each node's `mcp_servers:` list (see `agentic-runtime/actors/agent_sdk_common.py::_mount_external_mcp`). Add a GitHub MCP entry alongside the existing WebRobot MCP.

**Option A — hosted GitHub MCP (HTTP):**
```yaml
mcp_servers:
  - name: github
    type: http
    url: https://api.githubcopilot.com/mcp/
    headers:
      Authorization: "Bearer ${GITHUB_TOKEN}"   # injected from env / cloud_credentials, never hard-coded
```

**Option B — local server (stdio, npx):**
```yaml
mcp_servers:
  - name: github
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "${GITHUB_TOKEN}"
```

- For reading **public** competitor repos a **read-only** token suffices (fine-grained PAT, public-repo read; or contents:read).
- Confirm which transport `_mount_external_mcp` supports (http vs stdio) and add the GitHub tools to the node's `allowed`/`tools` if the profile gates tools.
- Keep the WebRobot MCP entry as-is; GitHub MCP is additive (read source) — the actual pipeline build/run still goes through the WebRobot MCP tools.

## Knowledge the agent needs
- [`MAPPING.md`](MAPPING.md) — the translation rules (load into the system prompt / as a skill).
- The existing data-engineer profile rules (selectors, `method: href`, trace for search/login, `python_extensions`).

## Status
- ✅ Confirmed: GitHub source is readable without clone (raw + git-trees API; GitHub MCP gives first-class tool access).
- ⏳ TODO: add the GitHub MCP entry to the agent profile `mcp_servers`, provision the read-only token, load `MAPPING.md` into the agent context, and validate with example 01.
