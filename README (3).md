# Automated Web Scraping & Data Extraction Pipeline

An ETL pipeline built with **Requests + BeautifulSoup** that scrapes product listings from [books.toscrape.com](https://books.toscrape.com) (a public sandbox site built for scraping practice), transforms the raw HTML into structured data, saves it as **CSV and JSON**, and generates an **automated summary statistics report** on every run.

## Pipeline

```
Extract            Transform              Load / Report
Fetcher  ───►  parse_listing_page  ───►  books.csv, books.json
(rate limit,     (HTML → Book records,    summary.json
 headers,         de-duplication,         summary_report.txt
 retries,         bad items skipped)
 robots.txt)
```

## Scraping best practices implemented

| Practice | Implementation (`scraper/fetcher.py`) |
|----------|----------------------------------------|
| Rate limiting | Minimum delay between every request (`--delay`, default 1 s) |
| Custom headers | Descriptive `User-Agent`, `Accept`, `Accept-Language` |
| Retry logic | Retries on timeouts, connection errors and HTTP 429/500/502/503/504 with exponential backoff; honours `Retry-After` |
| Fail fast | Non-transient errors (e.g. 404) are not retried |
| Politeness | Checks `robots.txt` before fetching (disable only with `--ignore-robots`) |
| Resilience | A malformed product block is skipped and counted, not fatal; a failed page keeps partial results; pagination loops are guarded |
| Timeouts | Every request has a timeout |

**Please edit the `User-Agent` string in `scraper/fetcher.py` to include your own contact/repo URL.**

## Installation

Requires Python 3.9+.

```bash
git clone <your-repo-url> web-scraper-pipeline
cd web-scraper-pipeline
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Usage

```bash
python -m scraper                          # scrape the whole catalogue (50 pages, ~1000 books)
python -m scraper --max-pages 5 -v         # first 5 pages with progress logging
python -m scraper --delay 2 --retries 5    # slower and more patient
python -m scraper --output-dir results
python -m scraper --help
```

| Option | Default | Meaning |
|--------|---------|---------|
| `--start-url` | page 1 of the catalogue | first listing page |
| `--max-pages` | 50 | page limit |
| `--delay` | 1.0 | seconds between requests |
| `--retries` | 3 | retries per page on transient errors |
| `--timeout` | 10 | request timeout (s) |
| `--output-dir` | `output` | where files are written |
| `--ignore-robots` | off | skip robots.txt check |

Exit codes: `0` success, `1` no records extracted, `2` invalid arguments.

### Output files

Real run writes to `output/`:

- `books.csv` – columns: `title, price_gbp, rating, in_stock, url`
- `books.json` – same records as JSON
- `summary.json` / `summary_report.txt` – automated statistics (count, price min/max/mean/median/stdev, rating distribution, average price per rating, stock %, top-5 most expensive and cheapest)

### About the sample output

`output/sample_*` files were generated **offline** by `examples/generate_sample_output.py` from a *simulated* copy of the site's markup (100 synthetic books). They demonstrate the output format only; the titles and prices are not real. Running `python -m scraper` on a machine with internet access produces the real `output/books.csv`.

## Tests

```bash
pip install -r requirements-dev.txt
pytest
```

30 tests (98% coverage) run fully offline against a fake HTTP session and generated HTML. They cover parsing, price/rating cleaning, malformed items, pagination, de-duplication, rate-limit timing, retry/backoff, `Retry-After`, robots.txt handling, partial failure, report statistics and the CLI.

## Project layout

```
web-scraper-pipeline/
├── requirements.txt / requirements-dev.txt
├── scraper/        fetcher.py  parser.py  pipeline.py  report.py  cli.py
├── tests/          test_scraper.py  fakesite.py
├── examples/       generate_sample_output.py
├── output/         sample_books.csv  sample_books.json  sample_summary_report.txt ...
└── docs/           README.pdf  test-report.pdf
```

## Responsible use

books.toscrape.com explicitly exists for scraping practice. If you adapt this tool to another site, check its terms of service and `robots.txt`, keep the delay generous, and do not collect personal data.

## License

MIT
