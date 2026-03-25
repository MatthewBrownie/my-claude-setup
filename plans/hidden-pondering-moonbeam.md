# Plan: MTG Slang Scraper

## Context
User wants a scraper for https://mtg.fandom.com/wiki/List_of_Magic_slang that extracts term/definition pairs and saves them as both JSON and CSV. The site uses Cloudflare bot protection that blocks plain HTTP requests, but the Fandom MediaWiki API (`/api.php?action=parse`) bypasses it and returns raw wikitext.

## Wikitext structure (confirmed by API probe)
- Terms are `====level-4 headings====`, sometimes with wikilinks: `====[[Link|Term]]====`
- Definitions are the text block between consecutive `====` headings
- Letter groups are `===A===`, `===B===` etc. (level-3) — act as separators only
- Top sections (`==Current terms==`) are level-2
- Definitions may contain bullet lists (`* item`), wikilinks, templates, and `<c>card</c>` tags
- Page has sub-tabs (Dated, R&D slang, Card nicknames) — these are separate wiki pages; scrape main page only

## Implementation plan

### File: `scrape_slang.py` (project root)

**Functions:**

1. `fetch_wikitext(page: str) -> str`
   - GET `https://mtg.fandom.com/api.php` with `action=parse`, `prop=wikitext`, `format=json`
   - User-Agent: `MTGSlangScraper/1.0 (brownie_magic_tools)`
   - Returns raw wikitext string

2. `clean_wikitext(text: str) -> str`
   - Strip `{{templates}}` (color symbols like `{{W}}`, etc.)
   - Resolve `[[Link|Display]]` → `Display`, `[[Term]]` → `Term`
   - Strip `<c>card</c>` tags, keep inner text
   - Strip other HTML tags
   - Remove `''` and `'''` bold/italic markers
   - Collapse excess blank lines

3. `parse_terms(wikitext: str) -> list[dict]`
   - Iterate lines; track current term (from `====heading====`) and accumulate definition lines
   - Flush current term on each new `====` or `===`/`==` heading
   - Skip entries where cleaned definition is empty (redirect stubs)
   - Return list of `{"term": str, "definition": str}`

4. `save_json(terms, path)` — `json.dump` with `indent=2`, UTF-8
5. `save_csv(terms, path)` — `csv.DictWriter` with `term`/`definition` columns, UTF-8

6. `main()` — orchestrates fetch → parse → save, prints count of terms found

### Output files
- `slang.json` (project root)
- `slang.csv` (project root)

### Dependencies
- `requests` (already used in project)
- stdlib only otherwise (`re`, `json`, `csv`)

## Verification
Run `python scrape_slang.py` from the project root; expect:
- ~150+ terms printed count
- `slang.json` and `slang.csv` created at project root
- Spot-check: "Aggro", "Combo", "Control" entries exist with non-empty definitions
