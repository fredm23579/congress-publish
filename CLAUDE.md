# CLAUDE.md — Congress Publisher

AI-assistant context for the **congress-publish** repository.

---

## Project Purpose

Congress Publisher is a Python 3 CLI tool that downloads U.S. Congressional
bill and amendment data from official government APIs and publishes it to a
GitHub repository (`gh-pages` branch). It supports manual one-shot runs as
well as automated self-updating via built-in scheduling.

---

## Repository Layout

```
congress-publish/
├── publish            # Main executable (Python 3, no .py extension)
├── requirements.txt   # Runtime dependencies (pip)
├── config.yml.example # Configuration template
├── config.yml         # !! GITIGNORED — contains secrets; copy from example
├── CLAUDE.md          # This file
└── README.md          # User-facing documentation
```

---

## Tech Stack

| Concern             | Library / Tool              |
|---------------------|-----------------------------|
| CLI argument parsing | `argparse` (stdlib)         |
| HTTP requests        | `requests >= 2.32`          |
| YAML config          | `PyYAML >= 6.0`             |
| Git operations       | `GitPython >= 3.1`          |
| Scheduling           | `schedule >= 1.2`           |
| Data source (primary)| Congress.gov API v3         |
| Data source (supplemental) | GovInfo.gov API       |

---

## Development Setup

```bash
# Python 3.8+ required
python3 -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate

pip install -r requirements.txt

cp config.yml.example config.yml
# Edit config.yml — fill in github.token, github.repo, repo (local path)
```

---

## Running the Tool

```bash
# Update current congress (119th as of 2025–2026)
./publish --current

# Update specific congresses
./publish --congresses=119
./publish --congresses=117,118,119

# Bills only (no amendments)
./publish --current --no-amendments

# Dry run — preview without writing or pushing
./publish --current --dry-run --verbose

# Self-updating: run once then repeat on a schedule
./publish --current --schedule=daily
./publish --current --schedule=6h
./publish --current --schedule=30m
```

---

## Current Congress Calculation

```python
# Congress starts in odd years; 1st Congress began 1789
year = datetime.now().year
if year % 2 == 0:
    year -= 1
congress = (year - 1789) // 2 + 1
# 2025–2026 → 119th Congress
```

---

## Key Functions (publish)

| Function | Description |
|---|---|
| `main()` | CLI entry point; parses args, loads config, calls `run_update()` |
| `run_update()` | Iterates congresses, calls `update_congress()` for each |
| `update_congress()` | Fetches data, writes files, calls `publish_to_git()` |
| `fetch_bills_congress_api()` | Paginated GET from Congress.gov `/bill/{congress}` |
| `fetch_amendments_congress_api()` | Paginated GET from Congress.gov `/amendment/{congress}` |
| `publish_to_git()` | `git init/add/commit/push` to `gh-pages` via GitPython |
| `start_scheduler()` | Wraps `schedule` library for recurring runs |
| `current_congress_number()` | Calculates current Congress number from year |
| `ordinal()` | `119` → `"119th"` |
| `load_config()` | Loads and validates `config.yml` |

---

## Output Data Structure

After a successful run, the local repo directory looks like:

```
<config.repo>/
└── 119/
    ├── metadata.json     # Bill/amendment counts and fetch timestamp
    ├── bills.json        # Full paginated bill list
    ├── amendments.json   # Full paginated amendment list
    └── bills/
        ├── hr1.json
        ├── s42.json
        └── …
```

All files are committed to the `gh-pages` branch and pushed to GitHub.

---

## Configuration Fields

| Field | Required | Description |
|---|---|---|
| `github.token` | Recommended | GitHub PAT with `repo` scope |
| `github.repo` | Yes | Remote repo URL (SSH or HTTPS) |
| `repo` | Yes | Local directory for writing data |
| `congress_api_key` | Recommended | Congress.gov API key (avoids rate limits) |
| `govinfo_api_key` | Optional | GovInfo.gov API key |

---

## API Registration Links

- **Congress.gov API** (primary): https://api.congress.gov/sign-up/
- **GovInfo API** (supplemental): https://api.data.gov/signup/

Both are free and provided by the U.S. federal government.

---

## Common Tasks for AI Assistants

- **Add a new data source**: implement a `fetch_*()` function following the
  pattern of `fetch_bills_congress_api()`, then call it in `update_congress()`.
- **Change output format**: modify the `json.dump()` calls in `update_congress()`.
- **Add a new CLI flag**: add `parser.add_argument(...)` in `build_parser()`.
- **Change schedule granularity**: edit the `start_scheduler()` function.
- **Fix ordinal edge cases**: see the `ordinal()` function; 11th/12th/13th use
  the `% 100 in (11, 12, 13)` check.

---

## Notes

- `config.yml` is in `.gitignore` — never commit it (contains credentials).
- The `gh-pages` branch is the published branch on the target GitHub repo.
- `--dry-run` is safe to use; it skips all filesystem writes and git operations.
- Rate limiting: 0.5 s sleep between paginated API requests (`RATE_LIMIT_SLEEP`).
- Congress.gov API returns max 250 results per page (`BILLS_PER_PAGE`).
